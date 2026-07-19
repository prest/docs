---
description: >-
  Run multiple SQL databases in one pREST instance — alias registry, Postgres and
  Timescale adapters, routing, pooling, and MCP.
---

# Multi-database

Route CRUD, catalog, scripts, and MCP tools across **one or more** SQL databases from a single pREST process. The first URL path segment selects the database (legacy name or registered **alias**).

{% hint style="warning" %}
**Multi-adapter (Postgres + Timescale auto-detect)** requires prest **`main`** ([#999](https://github.com/prest/prest/pull/999)) — not in [v2.2.0](../releases/v2.2.0.md). Registry multi-cluster on the **PostgreSQL** adapter shipped earlier in [v2.0.0](../releases/v2.0.0.md). MySQL / SQLite adapters are **roadmap**, not installable.
{% endhint %}

| Mode | When | `{database}` in URL | Connection target |
|------|------|---------------------|-------------------|
| **Legacy multi-DB** | No registry configured | Postgres database name | Same `pg.host`; `dbname` = path segment |
| **Registry multi-cluster** | `[[databases]]` or env registry set | Registered **alias** | Per-profile host, port, and credentials |
| **Multi-adapter** (`main` / #999) | Registry + Timescale detection | Registered **alias** | Postgres or Timescale adapter per alias |

```mermaid
flowchart LR
  req["GET /alias/schema/table"] --> mw[Adapter selector]
  mw --> reg[Adapter registry]
  reg --> pg[Postgres adapter]
  reg --> ts[TimescaleDB adapter]
  pg --> db1[(app_db)]
  ts --> db2[(metrics)]
```

---

## URL routing

All table operations use `/{database}/{schema}/{table}`:

```http
GET /tenant-a/public/users
POST /tenant-a/public/orders
GET /tenant-a/public
GET /_QUERIES/tenant-a/myqueries/get_all
```

With Postgres + Timescale aliases (`main` / #999):

```http
GET /postgres/public/users
GET /timescaledb/public/metrics
```

Script routes accept an optional database prefix (`/_QUERIES/{database}/{queriesLocation}/{script}`). When omitted, the default database (`pg.database`) is used.

Request flow: validate alias → select adapter (registry) → open or reuse pool → execute query.

---

## Configuration

Registry sources are merged in priority order: **indexed env pairs → TOML** (env wins on conflict).

Full sample (sanitized for public docs): [examples/multi-database-config.toml](examples/multi-database-config.toml).

### Environment variables (production / Kubernetes)

Register databases with contiguous 1-based index pairs:

```sh
DATABASE_ALIAS_1=tenant-a
DATABASE_URL_1=postgres://user:pass@cluster-a.example.com:5432/app_a?sslmode=require
DATABASE_ALIAS_2=tenant-b
DATABASE_URL_2=postgres://user:pass@cluster-b.example.com:5432/app_b?sslmode=require
```

`PREST_DATABASE_ALIAS_N` and `PREST_DATABASE_URL_N` are accepted as aliases of the above keys.

See [`install-manifests/kubernetes/deployment.yaml`](https://github.com/prest/prest/blob/main/install-manifests/kubernetes/deployment.yaml) for a multi-secret example with liveness/readiness probes.

### TOML (local development)

`pg.*` remains the default/fallback profile; registry entries override host, port, and credentials per alias:

```toml
[pg]
database = "prest-test"
single = false

[[databases]]
alias = "postgres"
host = "localhost"
port = 5432
database = "app_db"
user = "prest"
pass = "prest"
maxopenconn = 10
maxidleconn = 2

[[databases]]
alias = "timescaledb"
host = "localhost"
port = 5432
database = "metrics"
user = "prest"
pass = "prest"
```

On `main` / #999, startup **auto-detects** Timescale (extension present) vs Postgres for each alias. Unreachable aliases log a warning and are skipped; other aliases keep serving.

When no registry is configured, legacy `DATABASE_URL` / `pg.*` behavior is unchanged (single-adapter auto-detect on `main`).

### Per-database SSL

```toml
[[databases]]
alias = "secure_db"
host = "secure.internal"
user = "prest"
pass = "password"
database = "app"

[databases.ssl]
mode = "require"
cert = "/etc/prest/certs/client.crt"
key = "/etc/prest/certs/client.key"
rootcert = "/etc/prest/certs/ca.crt"
```

### Environment variable reference

| Variable | Description |
|----------|-------------|
| `DATABASE_ALIAS_N` | Alias for the Nth registered database (1-based index) |
| `DATABASE_URL_N` | Connection URL for the Nth registered database |
| `PREST_DATABASE_ALIAS_N` | Accepted alias for `DATABASE_ALIAS_N` |
| `PREST_DATABASE_URL_N` | Accepted alias for `DATABASE_URL_N` |

---

## Alias vs physical database name

- URLs and access rules use the **alias** (e.g. `tenant-a`).
- Connection pools use the profile's `database`, `host`, and credentials (e.g. `app_a` on `cluster-a.example.com`).
- When alias equals the physical database name (legacy mode), behavior matches pre-registry pREST.

---

## `pg.single`

Set `pg.single = false` to allow routing to multiple databases or aliases. When `true` and a registry is active, only the default database alias is accepted.

```toml
[pg]
single = false
```

Or via environment variable: `PREST_PG_SINGLE=false`.

---

## Timescale operators (`main` / #999)

When an alias is attached to the **TimescaleDB adapter**:

| Query param | Purpose |
|-------------|---------|
| `_time_bucket=1h` | `GROUP BY time_bucket('1 hour', time)` |
| `_time_bucket=1h,created_at` | Same with a custom time column |
| `_include_system_schemas=true` | Include `_timescaledb_*` schemas in listings |

Supported intervals: `5m`, `15m`, `1h`, `6h`, `1d`, `7d`, `30d`, `1y`.

```bash
curl "http://localhost:3000/timescaledb/public/metrics?_time_bucket=1h"
```

Continuous aggregates appear as queryable relations; create/manage them with SQL or [custom queries](../api-reference/custom-queries.md). See [TimescaleDB](../databases/timescaledb.md).

---

## Connection pooling

Pools are keyed by connection URI; aliases that share the same URI share a pool. Connections are opened lazily on first request per alias (or at startup when adapters register on `main` / #999).

Configure `maxopenconn` / `maxidleconn` per `[[databases]]` entry. **Budget:** `replicas × aliases × maxopenconn` against each cluster. Use PgBouncer or RDS Proxy when many aliases are registered.

---

## Health checks

| Endpoint | Purpose | Behavior |
|----------|---------|----------|
| `GET /_health` | Liveness | Pings the default database |
| `GET /_ready` | Readiness | Pings the default database and every registered alias |

Use `/_ready` for Kubernetes readiness probes when multiple databases are registered. See [Configuring pREST — Health check](configuring-prest.md#health-check-endpoints).

Unknown or unregistered aliases return **404**.

---

## Access control

`access.tables` entries support optional `database` and `schema` fields for per-alias permissions:

```toml
[[access.tables]]
database = "tenant-a"
schema = "public"
name = "users"
permissions = ["read"]
```

When a database registry is active, permissions are matched against alias + schema + table name. See [Permissions](permissions.md#table-permissions).

---

## MCP and aliases

With [MCP over HTTP](mcp-over-http.md) (v2.1.0+), registered aliases appear in MCP tools the same way they do in REST paths:

| Tool | Multi-database behavior |
|------|-------------------------|
| `prest.list_databases` | Lists registered aliases (and the default database) |
| `prest.select.{database}.{schema}.{table}` | `{database}` is the alias when a registry is configured |
| `prest.select_table` / catalog tools | Accept a database/alias argument where applicable |

Auth and ACL still apply — MCP is read-only but otherwise shares the HTTP stack.

---

## Migrating from single to multi-database

1. Keep `[pg]` for local/default compatibility.
2. Add `[[databases]]` (or `DATABASE_ALIAS_N` / `DATABASE_URL_N`) with clear aliases (`primary`, `metrics`).
3. Point clients at `/{alias}/...` instead of the physical database name when using the registry.
4. On `main` / #999, confirm Timescale aliases get `_time_bucket` if you need time-series grouping.

Single-database deployments without `[[databases]]` keep working.

---

## Local testing

Multi-cluster integration tests live in [`integration/controllers/multicluster_test.go`](https://github.com/prest/prest/blob/main/integration/controllers/multicluster_test.go). They require a second Postgres service (`PREST_PG_HOST_B`) provided by [`docker-compose-test.yml`](https://github.com/prest/prest/blob/main/docker-compose-test.yml):

```bash
make test-integration
```

Timescale E2E: `make test-integration-timescaledb`. See the [Development Guide](../get-prest/development-guide.md#integration-tests).

## Related

- [examples/multi-database-config.toml](examples/multi-database-config.toml)
- [Configuring pREST](configuring-prest.md)
- [TimescaleDB](../databases/timescaledb.md)
- [MCP over HTTP](mcp-over-http.md)
- [Permissions](permissions.md)
- [PR #999](https://github.com/prest/prest/pull/999)
- [Acronyms](../prestd/acronyms.md) · [MCP](../prestd/acronyms.md#mcp) · [SQL](../prestd/acronyms.md#sql)
