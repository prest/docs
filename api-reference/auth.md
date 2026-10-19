# Auth

_**prestd**_ has support in **JWT Token** generation based on two fields (example user and password), being possible to use an existing table from your database to login configuring some parameters in the configuration file (or environment variable), _by default this feature is_ **disabled**.

* Bearer - [RFC 6750](https://tools.ietf.org/html/rfc6750), bearer tokens to access OAuth 2.0-protected resources
* Basic - [RFC 7617](https://tools.ietf.org/html/rfc7617), base64-encoded credentials. More information below

> understand more about _http authentication_ [see this documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

---

## JWT verification (v2 defaults)

JWT middleware is **disabled by default** in v2+ (`jwt.default = false`). Set `jwt.default = true` to require a valid Bearer token on all endpoints except those in the JWT whitelist. See [Configuring pREST — JWT](../get-started/configuring-prest.md#jwt) for full configuration.

MCP (`/_mcp`, v2.1.0+) inherits the same JWT and auth stack as REST routes. When auth is enabled, send credentials on every `GET` and `POST` to `/_mcp`. Stdio clients that use the [pREST MCP Adapter](../get-started/prest-mcp-adapter.md) can pass a bearer token via `PREST_MCP_TOKEN`.

### Routes that require auth (v2.5.2)

With `auth.enabled = true`, every route except `/auth`, `/_health`, `/_ready`, and `/_studio` requires a bearer token. Before v2.5.2 the catalog routes and `/_PLUGIN` answered without one ([GHSA-fhf5-m388-4w4p](https://github.com/prest/prest/security/advisories/GHSA-fhf5-m388-4w4p), [GHSA-fmr6-9j48-xrqp](https://github.com/prest/prest/security/advisories/GHSA-fmr6-9j48-xrqp)).

| Route | With `auth.enabled = true` |
|-------|----------------------------|
| `/{database}/{schema}/{table}`, `/batch/{database}/{schema}/{table}` | Token + table ACL |
| `/databases`, `/schemas`, `/tables`, `/{database}/{schema}`, `/show/{database}/{schema}/{table}` | Token + table ACL (v2.5.2) |
| `/_QUERIES/{location}/{script}` | Token (also required when `queries.restrict = true`) + script ACL |
| `/_QUERIES/registry/...` | Token + `queries.register_admins` |
| `/_mcp` | Token |
| `/_PLUGIN/{file}/{func}` | Token (v2.5.2) |
| `/auth`, `/_health`, `/_ready`, `/_studio` | Public |

A request with no token gets **401** `{"error":"authorization token is empty"}`.

With `auth.enabled = false`, none of these routes require a token. `/_mcp` and `/_PLUGIN` are then open to any caller; CRUD, catalog, and `/show` still follow [`[access]`](../get-started/permissions.md). To require a token on every route regardless, set `jwt.default = true` (below).

### Required verification material

When `jwt.default = true` and debug mode is off, you should configure one of the following:

| Setting | Environment variable | Purpose |
|---------|---------------------|---------|
| `jwt.key` | `PREST_JWT_KEY` | Shared secret for HMAC algorithms — minimum length applies, see [below](#hmac-key-requirements-v242) |
| `jwt.jwks` | `PREST_JWT_JWKS` | JSON Web Key Set for asymmetric verification |
| `jwt.wellknownurl` | `PREST_JWT_WELLKNOWNURL` | OpenID Connect well-known URL to fetch JWKS |

**In v2+ ([#974](https://github.com/prest/prest/pull/974)):** if JWT is enabled but no verification material is configured, JWT middleware is **auto-disabled** with an error log — the server continues to start. When `auth.enabled = true` without `jwt.key`, auth is also auto-disabled.

> **v2.0.0-rc6 tagged binary:** the rc6 release **refuses to start** in the same situations. See [v2.0.0-rc6](../releases/v2.0.0-rc6.md#jwt-fail-closed-startup-960).

### HMAC key requirements (v2.4.2)

Since **v2.4.2** ([#1017](https://github.com/prest/prest/pull/1017)), pREST validates `jwt.key` against the RFC 7518 minimum for the configured HMAC algorithm at config load. The underlying library (`go-jose/v4`) enforces these sizes itself; checking at startup surfaces the problem in logs instead of at request time.

| `jwt.algo` | Minimum `jwt.key` length |
|---|---|
| `HS256` — also the default when `jwt.algo` is unset | **32 bytes** |
| `HS384` | **48 bytes** |
| `HS512` | **64 bytes** |
| `RS*`, `ES*`, `PS*`, `EdDSA` | Not checked — `jwt.key` is not used as a MAC key |

The check is on the byte length of the raw string, so a 32-character ASCII secret satisfies HS256. Measure yours with `printf '%s' "$PREST_JWT_KEY" | wc -c`.

{% hint style="danger" %}
**An undersized key fails open, not closed.** pREST starts normally, discards the key, and disables the features that need it. `POST /auth` is no longer registered (clients get **404**, not 401) and every route wrapped by the auth middleware **passes through unauthenticated**. With `jwt.default = true` and no JWKS, the JWT middleware is removed from the stack entirely.

Rotate short secrets **before** upgrading, and check startup logs for:

```
level=ERROR msg="jwt.key too short for HMAC algorithm" algo=HS256 got=6 want=32
level=ERROR msg="auth disabled: jwt.key is empty"
```
{% endhint %}

A configured `jwt.jwks` or `jwt.wellknownurl` is unaffected — verification continues against the JWKS even when an undersized HMAC key is discarded.

### Signature algorithm (`jwt.algo`, v2.4.2)

`jwt.algo` was accepted but discarded in earlier v2 releases — tokens were parsed without restricting the permitted signature algorithm. Since **v2.4.2** it is passed to the parser as the single allowed algorithm, which structurally prevents algorithm-confusion attacks.

- A token whose `alg` header does not match `jwt.algo` is rejected with **401** and `{"error": "failed JWT token parser"}`.
- The value is matched **case-sensitively** against `EdDSA`, `HS256`, `HS384`, `HS512`, `RS256`, `RS384`, `RS512`, `ES256`, `ES384`, `ES512`, `PS256`, `PS384`, `PS512`. Anything else — including `hs256` in lowercase or an explicit `algo = ""` — makes every request return **HTTP 500** with `unsupported JWT signature algorithm`.

Leave `jwt.algo` unset to get the `HS256` default.

### JWKS fetch hardening (v2.3.0)

When you configure `jwt.wellknownurl` / `PREST_JWT_WELLKNOWNURL`, pREST fetches the JWKS from the identity provider. Since **v2.3.0** ([#1002](https://github.com/prest/prest/pull/1002)), that fetch (now on `jwx/v3`):

- **Rejects non-2xx responses** instead of attempting to parse an error page.
- **Caps the response body at 1 MiB** to bound memory.
- **Redacts the URL in logs** — userinfo, query string, and fragment are dropped.

Key-matching semantics (kid match, single-key with empty kid, empty-HMAC-key bypass guard) are unchanged, and there are **no config or environment changes**: `jwt.jwks` and `jwt.wellknownurl` work exactly as before.

Debug mode (`PREST_DEBUG=true` or `debug = true` in TOML) bypasses JWT enforcement at runtime.

To disable JWT entirely, leave `jwt.default = false` (the default in v2+).

### Whitelist

Endpoints matching the whitelist regex do not require a JWT. The v2 default whitelist is `^\/auth$` (only the `/auth` endpoint). Configure additional patterns in TOML or via `PREST_JWT_WHITELIST`:

```toml
[jwt]
whitelist = ["\\/auth", "\\/ping", "\\/ping\\/.*"]
```

From v2.5.2 the whitelist is also how you re-expose a catalog route on purpose, for example a public schema browser:

```toml
[jwt]
whitelist = ["^\\/auth$", "^\\/tables$"]
```

{% hint style="warning" %}
Whitelisting `/databases`, `/schemas`, `/tables`, or `/show/...` lets anonymous callers map your schema again — the exposure fixed by [GHSA-fhf5-m388-4w4p](https://github.com/prest/prest/security/advisories/GHSA-fhf5-m388-4w4p). Table ACL from `[access]` still applies to whitelisted routes.
{% endhint %}

### Upgrading from v1

v1 used `[/auth]` as the default whitelist and did not enforce JWT key configuration at startup. See [Upgrading to v2](../get-started/upgrading-to-v2.md) for migration steps.

---

## Token generation (`/auth` endpoint)

When `auth.enabled = true`, pREST exposes a `/auth` endpoint that validates credentials against a database table and returns a signed JWT.

### Bearer

```sh
curl -i -X POST http://127.0.0.1:3000/auth -H "Content-Type: application/json" -d '{"username": "<username>", "password": "<password>"}'
```

### Basic

```sh
curl -i -X POST http://127.0.0.1:3000/auth --user "<username>:<password>"
```

## Related

- [Configuring pREST — JWT](../get-started/configuring-prest.md#jwt)
- [v2.4.2 release notes](../releases/v2.4.2.md)
- [v2.5.2 release notes](../releases/v2.5.2.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Install pREST MCP Adapter](../ai/install-prest-mcp.md)
- [Permissions](../get-started/permissions.md)
- [Acronyms](../prestd/acronyms.md) · [JWT](../prestd/acronyms.md#jwt) · [MCP](../prestd/acronyms.md#mcp)
