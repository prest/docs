---
description: >-
  Turn on pREST's opt-in [guard] middleware for per-client rate limits, attack
  payload inspection, IP allow and block lists, auto-bans and shared Redis state.
---

# Request security

Turn on `[guard]` to rate-limit each client, refuse requests that carry attack payloads, and allow or block client IP addresses before pREST runs auth or queries the database. The guard is off by default, and while it is off pREST behaves exactly as before.

{% hint style="warning" %}
**Requires prest `main` (unreleased)** — [#1044](https://github.com/prest/prest/pull/1044).
{% endhint %}

The guard runs inside pREST on the [guard-core](https://github.com/rennf93/guard-core-go) engine. It complements a reverse proxy or gateway in front of pREST; it does not replace one.

---

## Quick start

```toml
[guard]
enabled = true
rate_limit = 100          # requests per client IP per window; 0 turns rate limiting off
rate_limit_window = 60    # seconds
exclude_paths = ["/_health", "/_ready"]
```

Or with environment variables:

```shell
PREST_GUARD_ENABLED=true
PREST_GUARD_RATE_LIMIT=100
PREST_GUARD_EXCLUDE_PATHS=/_health,/_ready
```

`enabled = true` on its own inspects every request for attack payloads but does not rate-limit, because `rate_limit` defaults to `0`. To see what the guard would refuse before it refuses anything, start with `passive = true` (see [Passive mode](#passive-mode)).

---

## Settings

| TOML key | Environment variable | Default | Description |
| -------- | -------------------- | ------- | ----------- |
| `guard.enabled` | `PREST_GUARD_ENABLED` | `false` | Turn the guard on |
| `guard.passive` | `PREST_GUARD_PASSIVE` | `false` | Log what would be refused, refuse nothing. See [Passive mode](#passive-mode) |
| `guard.rate_limit` | `PREST_GUARD_RATE_LIMIT` | `0` | Requests each client IP may send per window. `0` turns rate limiting off. See [Rate limits](#rate-limits) |
| `guard.rate_limit_window` | `PREST_GUARD_RATE_LIMIT_WINDOW` | `60` | Window length in seconds |
| `guard.penetration_detection` | `PREST_GUARD_PENETRATION_DETECTION` | `true` | Inspect requests for attack payloads. See [Payload inspection](#payload-inspection) |
| `guard.max_body_bytes` | `PREST_GUARD_MAX_BODY_BYTES` | `8192` | How many leading bytes of a request body are inspected |
| `guard.blacklist` | `PREST_GUARD_BLACKLIST` | `[]` | IP addresses and CIDR ranges to refuse. See [IP allow and block lists](#ip-allow-and-block-lists) |
| `guard.whitelist` | `PREST_GUARD_WHITELIST` | `[]` | When not empty, the **only** IP addresses and ranges let in |
| `guard.exclude_paths` | `PREST_GUARD_EXCLUDE_PATHS` | `[]` | Paths that skip every guard check. See [Excluded paths](#excluded-paths) |
| `guard.trusted_proxies` | `PREST_GUARD_TRUSTED_PROXIES` | `[]` | Proxies whose `X-Forwarded-For` header names the client. See [Clients behind a proxy](#clients-behind-a-proxy) |
| `guard.auto_ban` | `PREST_GUARD_AUTO_BAN` | `false` | Ban a client for an hour after repeated attack payloads. See [Auto-ban](#auto-ban) |
| `guard.block_cloud_providers` | `PREST_GUARD_BLOCK_CLOUD_PROVIDERS` | `[]` | Refuse clients in these providers' published address ranges. See [Cloud provider ranges](#cloud-provider-ranges) |
| `guard.redis_url` | `PREST_GUARD_REDIS_URL` | | Share rate-limit and ban state through Redis. See [Several instances](#several-instances) |
| `guard.redis_prefix` | `PREST_GUARD_REDIS_PREFIX` | `prest_guard` | Prefix of the guard's Redis keys |
| `guard.redis_fail_open` | `PREST_GUARD_REDIS_FAIL_OPEN` | `true` | Keep serving on per-instance state when Redis does not answer. `false` refuses requests instead |

A `guard` object in [`PREST_CONFIG_JSON`](configuring-prest.md#configuration-as-json) is applied too. Every value is checked strictly: one that cannot be used makes every request answer `500`, as described in [Invalid configuration](#invalid-configuration).

### Environment variable lists

List variables are **comma-separated**, and spaces around entries are ignored:

```shell
PREST_GUARD_BLACKLIST=203.0.113.0/24,198.51.100.7
PREST_GUARD_EXCLUDE_PATHS="/_health, /_ready"
```

This is the opposite of `PREST_CORS_*` and `PREST_JWT_WHITELIST`, which split on spaces. A space-separated guard list is read as one entry. In `blacklist`, `whitelist` or `trusted_proxies` that entry is not an IP address, so every request gets `500`. In `exclude_paths` it matches no path. A variable replaces the whole list from `prest.toml`; it does not add to it.

---

## Where the guard runs

1. [`http.allowed_hosts`](configuring-prest.md#allowed-hosts) refuses requests for other hosts.
2. [CORS](cors-support.md) answers preflight requests.
3. The guard checks the request.
4. JWT and auth run, then the route.

The guard therefore covers `/auth`, `/_QUERIES`, `/_mcp`, `/_health`, `/_ready` and Studio as well as table routes, unless a path is [excluded](#excluded-paths).

## What a refused request gets

| Reason | Status | `error` | `check` in the log |
| ------ | ------ | ------- | ------------------ |
| Over the rate limit | `429`, with `Retry-After` set to `rate_limit_window` | `Too many requests` | `rate_limit` |
| Attack payload found | `400` | `Suspicious activity detected` | `suspicious_activity` |
| Client on `blacklist`, or not on a non-empty `whitelist` | `403` | `Forbidden` | `ip_security` |
| Client banned by [auto-ban](#auto-ban) | `403` | `IP address banned` | `ip_security` |
| Client in a blocked cloud range | `403` | `Cloud provider IP not allowed` | `cloud_provider` |

The body is pREST's usual error, `{"error": "Too many requests"}`, and follows `_renderer` like any other error. The guard's own `500` answers are covered in [Invalid configuration](#invalid-configuration) and [Several instances](#several-instances). The response keeps the CORS headers and, when configured, the [request ID](configuring-prest.md#request-ids), so a browser app can read it.

Each refusal is logged as a warning:

```json
{"time":"2026-11-27T10:15:02Z","level":"WARN","msg":"guard request blocked","passive":false,"check":"rate_limit","reason":"Rate limit exceeded for IP: 198.51.100.4 (101 requests in 60s window)","trigger":"","client_ip":"198.51.100.4","method":"GET","path":"/db/public/orders"}
```

For a payload, `reason` names the part of the request that matched, such as `Query param 'name'`, `Header 'cookie'` or `Request body field 'note'`, and the pattern it matched.

## Response headers

With the guard on, every response it checks, served or refused, also carries these headers:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
X-Permitted-Cross-Domain-Policies: none
X-Download-Options: noopen
```

pREST has no setting to change or remove them. Responses on [excluded paths](#excluded-paths) do not get them.

{% hint style="warning" %}
`Strict-Transport-Security` tells a browser that reached pREST over HTTPS to use only HTTPS for that host **and all its subdomains** for a year. Browsers ignore it on plain HTTP. If a subdomain of the API host still serves plain HTTP, strip the header at your proxy before you turn the guard on.
{% endhint %}

---

## Rate limits

`rate_limit` caps how many requests one client IP may send in any `rate_limit_window` seconds, across all routes together. Every request over the limit gets `429`.

* The window slides: pREST counts the client's requests of the last `rate_limit_window` seconds. Refused requests count too, so a client that keeps retrying stays refused until it slows down.
* The guard counts requests before any token is checked, so unauthenticated requests and `POST /auth` logins count too. This also slows down password guessing.
* CORS preflights do not count: the CORS layer answers them before the guard runs.
* Each instance counts on its own unless `redis_url` is set. See [Several instances](#several-instances).
* Behind a load balancer, set [`trusted_proxies`](#clients-behind-a-proxy), or every client shares the balancer's address and one limit.
* Health probes count like any other client. Put `/_health` and `/_ready` in `exclude_paths` so a probe never gets `429`.

## Payload inspection

With `penetration_detection = true` (the default), the guard looks for attack patterns, such as SQL injection, cross-site scripting, path traversal and command injection, in the URL path, the query parameters, the request headers (cookies included) and the first `max_body_bytes` bytes of the body. A match gets `400`.

pREST's own query syntax passes, for example `id=$in.1,2`, `name=$ilike.%25smith%25`, `_order=-id` and `_count=*`.

{% hint style="warning" %}
**Legitimate data can look like an attack.** A write whose body holds SQL text such as `SELECT * FROM t WHERE a = 1`, an HTML snippet with a `<script>` tag, or `; DROP TABLE` is refused with `400`, and so is a filter value such as `union select` or `../../`. If your data contains such text, run in [passive mode](#passive-mode) first and read the log. Then exclude the affected paths, or set `penetration_detection = false` to keep only rate limits and IP rules.
{% endhint %}

* Body content past `max_body_bytes` is not inspected, and the request is still served. A larger value inspects more of each body at a higher CPU cost per request.
* Clients on a non-empty `whitelist` are not inspected.
* With [`auto_ban`](#auto-ban), repeated matches ban the client.

## IP allow and block lists

```toml
[guard]
enabled = true
blacklist = ["203.0.113.0/24", "198.51.100.7"]
```

* `blacklist` refuses the listed addresses and ranges with `403`.
* `whitelist` is an **allowlist**. As soon as it has an entry, every client that is not on it gets `403`. Listed clients skip rate limits and payload inspection.
* Entries are IP addresses or CIDR ranges. Any other entry, such as a host name, makes every request answer `500`. See [Invalid configuration](#invalid-configuration).
* Both lists match the client address pREST resolved, which is the forwarded address behind a [trusted proxy](#clients-behind-a-proxy).

{% hint style="danger" %}
Do not use `whitelist` to exempt a few addresses from rate limits. It locks out every client that is not listed, load balancer health checks included, unless their paths are in `exclude_paths`.
{% endhint %}

## Clients behind a proxy

Without `trusted_proxies`, the client is the address of the TCP connection. Behind a load balancer or reverse proxy that is the proxy, for every request. All clients then share one rate limit, the IP lists cannot tell them apart, and an auto-ban locks out everyone.

List the proxies' addresses or ranges in `trusted_proxies`:

```toml
[guard]
enabled = true
trusted_proxies = ["10.0.0.0/8"]
```

When the connection comes from a listed proxy, pREST reads `X-Forwarded-For` from the right and skips addresses that are themselves trusted proxies. The first address that is not one is the client. If every address is a trusted proxy, the leftmost one is used.

| Connection from | `X-Forwarded-For` | Client |
| --------------- | ----------------- | ------ |
| `10.0.0.5` (trusted) | `198.51.100.4` | `198.51.100.4` |
| `10.0.0.5` (trusted) | `203.0.113.66, 198.51.100.4` | `198.51.100.4`. The client sent the first entry itself |
| `10.0.0.5` (trusted) | `198.51.100.4, 10.0.0.9` | `198.51.100.4`. `10.0.0.9` is a trusted proxy too |
| `192.0.2.10` (not trusted) | `198.51.100.4` | `192.0.2.10`. The header is ignored |

* Several `X-Forwarded-For` header lines are read as one list.
* An entry may carry the port some proxies append, such as `198.51.100.4:51234` or `[2001:db8::1]:51234`.
* If an entry pREST reaches on the way is not an IP address, it ignores the header and uses the connection's address.
* `X-Real-IP` and `Forwarded` are not read.
* `trusted_proxies` entries must be IP addresses or CIDR ranges. A host name makes every request answer `500`.

## Passive mode

`passive = true` runs every check but refuses nothing. A request that would have been refused is served and logged with `"passive":true`:

```json
{"time":"2026-11-27T10:15:02Z","level":"WARN","msg":"guard request blocked","passive":true,"check":"suspicious_activity","reason":"Suspicious activity detected: 198.51.100.4","trigger":"Query param 'name': Value matched pattern '…'","client_ip":"198.51.100.4","method":"GET","path":"/db/public/orders"}
```

* Passive mode covers rate limits, payload inspection, the IP lists and cloud ranges.
* It never bans, even with `auto_ban = true`.
* An [invalid configuration](#invalid-configuration) still answers `500`.

## Auto-ban

`auto_ban = true` bans a client that keeps sending attack payloads. pREST counts the requests flagged in each attack category for each client. The request that brings a count to 10 gets `403` `IP address banned`, and so does every request from that client for the next hour, whatever it contains. Other clients are not affected.

* The threshold (10) and the ban length (one hour) are fixed.
* Requests refused by the rate limit do not count toward a ban.
* The counts are kept in each instance's memory until prestd restarts. Bans are kept in memory too, or in Redis when `redis_url` is set.
* Loopback addresses (`127.0.0.0/8` and `::1`) are never banned.

{% hint style="warning" %}
Behind a load balancer, set `trusted_proxies` before you turn on `auto_ban`. Otherwise the ban lands on the balancer's address and locks out every client at once. That is why `auto_ban` is off by default.
{% endhint %}

## Cloud provider ranges

```toml
[guard]
enabled = true
block_cloud_providers = ["AWS", "GCP", "Azure"]
```

A client whose address is in one of these providers' published ranges gets `403`. The accepted names are `AWS`, `GCP`, `Azure`, `DigitalOcean`, `Linode` and `Vultr`, written exactly so. Any other name makes every request answer `500`.

pREST downloads the ranges from each provider when it starts and refreshes them every hour, so it needs outbound HTTPS access to the providers' sites. If a download fails, pREST logs it and lets that provider's addresses through until a later refresh succeeds.

## Excluded paths

`exclude_paths` lists paths that skip every guard check, the IP lists and rate limits included. An entry covers that path and everything below it: `/_health` covers `/_health` and `/_health/db`, but not `/_healthz`. The query string is not part of the match.

```toml
[guard]
enabled = true
exclude_paths = ["/_health", "/_ready"]
```

While `exclude_paths` is empty, the guard engine's own defaults apply instead: `/docs`, `/redoc`, `/openapi.json`, `/openapi.yaml`, `/favicon.ico` and `/static`, with everything below them, skip payload inspection, though rate limits and the IP lists still apply. pREST serves none of these paths, so this only matters for a database named `docs` or `static`. Setting `exclude_paths` replaces the defaults.

## Several instances

Without `redis_url`, each instance keeps its own rate-limit counts and bans in memory, and a restart clears them. A client whose requests are spread over several instances can then send `rate_limit` requests to each one. Set `redis_url` to share the state:

```toml
[guard]
enabled = true
rate_limit = 100
redis_url = "redis://redis:6379/0"
```

* Counts and bans then survive a restart of prestd.
* Keys start with `redis_prefix` exactly as written, with no separator added: the default gives keys such as `prest_guardrate_limit:rate:198.51.100.4`. Give each deployment that shares a Redis database its own prefix.

`redis_fail_open` decides what happens when Redis does not answer:

| `redis_fail_open` | Redis unreachable when prestd starts | Redis stops answering later |
| ----------------- | ------------------------------------ | --------------------------- |
| `true` (default) | Logs `Redis unavailable during initialization, failing open` and serves on per-instance state | Requests are served on per-instance state |
| `false` | Every request gets `500` `guard configuration invalid` until prestd is restarted with Redis reachable | Requests get `500` `Security check failed` until Redis answers again |

While Redis is not answering, each request first waits for the Redis timeouts, which can take several seconds.

## Invalid configuration

The guard fails closed. If a `[guard]` value cannot be used, prestd still starts, but every request gets `500` `{"error":"guard configuration invalid"}` instead of running without the protection you asked for. The cause is only in the log, written with every request:

```json
{"time":"2026-11-27T10:15:02Z","level":"ERROR","msg":"guard config invalid, failing closed","err":"guard config invalid: guard.rate_limit: \"100/min\" is not a valid integer"}
```

A value is invalid when:

* `enabled`, `passive`, `penetration_detection`, `auto_ban` or `redis_fail_open` is not a boolean, such as `yes` or `on`. `true`, `false`, `1` and `0` work.
* `rate_limit`, `rate_limit_window` or `max_body_bytes` is not a whole number, such as `100/min` or `2.5`, `rate_limit` is negative, or `rate_limit_window` or `max_body_bytes` is `0` or less.
* A `blacklist`, `whitelist` or `trusted_proxies` entry is not an IP address or CIDR range.
* A `block_cloud_providers` name is unknown.
* Redis is unreachable when prestd starts and `redis_fail_open = false`.

{% hint style="warning" %}
The first two cases apply even while the guard is off. A boolean or number in `[guard]` that cannot be read makes every request answer `500`, even with `enabled` unset or `false`.
{% endhint %}

---

## Troubleshooting

| Symptom | Cause | Fix |
| ------- | ----- | --- |
| Every request answers `500` `{"error":"guard configuration invalid"}` | A `[guard]` value is invalid, or Redis was unreachable at startup with `redis_fail_open = false` | Read the `err` field of the `guard config invalid, failing closed` log line. See [Invalid configuration](#invalid-configuration) |
| Every client gets `403` `Forbidden` | `whitelist` has entries, so only those addresses are let in | Empty `whitelist`, or list every client range. See [IP allow and block lists](#ip-allow-and-block-lists) |
| All clients get `429` together, or one ban locks everyone out | pREST is behind a proxy and sees only the proxy's address | Set `trusted_proxies`. See [Clients behind a proxy](#clients-behind-a-proxy) |
| Load balancer or Kubernetes probes fail with `429` or `403` | Probes go through the guard like any client | Add `/_health` and `/_ready` to `exclude_paths` |
| A normal-looking write gets `400` `Suspicious activity detected` | Payload inspection matched text in the request | Read `reason` in the log. See [Payload inspection](#payload-inspection) |
| `PREST_GUARD_EXCLUDE_PATHS` has no effect | The paths were separated by spaces | Separate them with commas. See [Environment variable lists](#environment-variable-lists) |
| The log shows `The connecting IP (…) already appears inside its own X-Forwarded-For chain` | Expected with `trusted_proxies`: pREST resolves the client before the guard engine sees the request | Nothing. The engine logs it once |

---

## Related

* [Configuring pREST](configuring-prest.md)
* [CORS Support](cors-support.md)
* [Allowed hosts](configuring-prest.md#allowed-hosts)
* [Auth](../api-reference/auth.md)
* [Deploying with Docker](../deployment/deploying-with-docker.md)
* [Changes since v2.6.0 (unreleased)](../releases/main-since-v2.6.0.md)
* [Acronyms — HTTP](../prestd/acronyms.md#http) · [JWT](../prestd/acronyms.md#jwt)
