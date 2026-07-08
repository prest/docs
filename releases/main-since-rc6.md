# Changes since v2.0.0-rc6

**Released as [v2.0.0](v2.0.0.md)** (tag `8784a25`). The commits below were merged after [v2.0.0-rc6](v2.0.0-rc6.md) and are included in the v2.0.0 release.

| Commit | PR | Date | Summary |
|--------|-----|------|---------|
| `c54b7dd` | [#972](https://github.com/prest/prest/pull/972) | Jul 4, 2026 | PrestConf refactor, Postgres adapter/connection pooling, credential redaction in logs, template param sanitization |
| `31a9854` | [#974](https://github.com/prest/prest/pull/974) | Jul 4, 2026 | Config resilience (graceful fallbacks), JWT auto-disable instead of startup abort |
| `8784a25` | [#973](https://github.com/prest/prest/pull/973) | Jul 7, 2026 | Multi-database registry, alias-aware routing, `/_ready`, per-alias permissions |

---

## #972 — Adapter and connection management

Mostly internal, with these user-facing changes:

- **Postgres adapter refactor** with per-request connection context (foundation for multi-database routing and health checks).
- **Credential redaction in error logs** via the `logsafe` package — database passwords and connection strings are not emitted in log output.
- **Custom query template parameter sanitization** — template query parameters are validated; unsafe inputs are rejected. Pass well-formed values to `/_QUERIES` scripts.
- **Context-aware health checks** — health endpoints use request-scoped connection context.

---

## #974 — Configuration resilience

Startup is never blocked by bad configuration. Invalid or missing settings produce warnings and safe fallbacks instead of fatal exits.

### JWT auto-disable (`ensureJWTConfig`)

In v2.0.0, missing JWT verification material no longer aborts startup:

- When `jwt.default = true` and `debug = false`, but no `jwt.key`, `jwt.jwks`, or `jwt.wellknownurl` is configured → JWT middleware is **auto-disabled** with an error log. The server continues to start.
- When `auth.enabled = true` but `jwt.key` is missing → auth is **auto-disabled** with an error log.

This is the safe alternative to the rc6 fail-closed behavior, which still prevents authentication bypass via an empty HMAC key ([GHSA-fj7v-859r-2fm4](https://github.com/prest/prest/security/advisories/GHSA-fj7v-859r-2fm4)).

> **rc6 tagged binary:** the [v2.0.0-rc6](v2.0.0-rc6.md) release still **refuses to start** without verification material. See the rc6 page for that behavior.

### `jwt.default` default changed to `false`

The Viper default for `jwt.default` is now **`false`** in v2.0.0 (rc6 implied `true`). JWT middleware is off unless you explicitly enable it.

### Other resilience fallbacks

| Condition | Behavior |
|-----------|----------|
| Missing, unreadable, or malformed TOML | Warnings; Viper defaults applied |
| Invalid `access.tables`, `cache.endpoints`, or `databases` keys | Zero values + warning |
| Queries path unavailable | Fallback to `~/queries`, then disable queries feature |
| Cache storage unavailable | Fallback to `./`, then disable cache |
| Invalid registry entries (duplicate alias, missing URL, invalid alias) | Entry skipped with warning |

### Structured logging

`setupLogger` runs on **every** startup — structured JSON logging via Go `slog` is always configured, not only when cache is enabled. Use `PREST_LOG_LEVEL` to control verbosity.

---

## #973 — Multi-database

Full multi-database support via a `[[databases]]` registry and environment variable pairs.

### URL routing

All CRUD uses `/{database}/{schema}/{table}`:

```http
GET /tenant-a/public/users
POST /tenant-a/public/orders
GET /_QUERIES/tenant-a/myfolder/my_query?field1=foo
```

The `{database}` segment is either a Postgres database name (legacy mode) or a registered **alias** (registry mode).

### Configuration

**Environment variables** (Kubernetes / production):

```sh
DATABASE_ALIAS_1=tenant-a
DATABASE_URL_1=postgres://user:pass@cluster-a.example.com:5432/app_a?sslmode=require
DATABASE_ALIAS_2=tenant-b
DATABASE_URL_2=postgres://user:pass@cluster-b.example.com:5432/app_b?sslmode=require
```

`PREST_DATABASE_ALIAS_N` and `PREST_DATABASE_URL_N` are accepted aliases. Env pairs win over TOML on conflict.

**TOML** (local development):

```toml
[pg]
database = "prest-test"
single = false

[[databases]]
alias = "prest-test"
host = "postgres"
port = 5432
database = "prest-test"
user = "postgres"
pass = "postgres"
ssl.mode = "disable"

[[databases]]
alias = "secondary-db"
host = "postgres-b"
port = 5432
database = "secondary-cluster"
user = "postgres"
pass = "postgres"
ssl.mode = "disable"
```

See the [Multi-database guide](../get-started/multi-database.md) for full details.

### `GET /_ready` readiness endpoint

| Endpoint | Purpose | Behavior |
|----------|---------|----------|
| `GET /_health` | Liveness | Pings the default database |
| `GET /_ready` | Readiness | Pings the default database and every registered alias |

Use `/_ready` for Kubernetes readiness probes. See the [Kubernetes deployment manifest](https://github.com/prest/prest/blob/main/install-manifests/kubernetes/deployment.yaml) in the prest repo for a multi-secret example.

### Per-alias permissions

`access.tables` entries support optional `database` and `schema` fields for per-alias access control:

```toml
[[access.tables]]
database = "tenant-a"
schema = "public"
name = "users"
permissions = ["read"]
```

See [Permissions](../get-started/permissions.md#table-permissions).

---

## Documentation links

- [Multi-database guide](../get-started/multi-database.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Upgrading to v2](../get-started/upgrading-to-v2.md)
- [v2.0.0 release notes](v2.0.0.md)
- [v2.0.0-rc6 release notes](v2.0.0-rc6.md)
