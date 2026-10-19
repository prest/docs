---
description: >-
  Run multiple SQL databases in one pREST instance — alias registry, Postgres,
  Timescale, and MySQL adapters, pg.allowed_databases, routing, pooling, and MCP.
---

# Multi-database

Route CRUD, catalog, scripts, and MCP tools across **one or more** SQL databases from a single pREST process. The first URL path segment selects the database (a served database name or a registered **alias**).

{% hint style="info" %}
**Multi-adapter (Postgres + Timescale auto-detect)** shipped in [**v2.3.0**](../releases/v2.3.0.md) ([#999](https://github.com/prest/prest/pull/999)). Registry multi-cluster on the **PostgreSQL** adapter shipped earlier in [v2.0.0](../releases/v2.0.0.md). A **MySQL 8** dialect (`engine = "mysql"`) shipped in [v2.5.0](../releases/v2.5.0.md). SQLite remains **roadmap**.
{% endhint %}

| Mode | When | `{database}` in URL | Connection target |
|------|------|---------------------|-------------------|
| **No-registry** (v2.5.2) | No registry configured | `pg.database` or a name in `pg.allowed_databases` | Same `[pg]` host and credentials; any other name is **400** |
| **Registry multi-cluster** | `[[databases]]` or env registry set | Registered **alias** | Per-profile host, port, and credentials |
| **Multi-adapter** (v2.3.0 / #999) | Registry + Timescale detection | Registered **alias** | Postgres or Timescale adapter per alias |
| **Mixed engines** (v2.5.0+) | Registry entry with `engine = "mysql"` or a `mysql://` URL | Registered **alias** | MySQL adapter for that alias, PostgreSQL/Timescale for the rest |

```mermaid
flowchart LR
  req["GET /alias/schema/table"] --> mw[Adapter selector]
  mw --> reg[Adapter registry]
  reg --> pg[Postgres adapter]
  reg --> ts[TimescaleDB adapter]
  reg --> my[MySQL adapter]
  pg --> db1[(app_db)]
  ts --> db2[(metrics)]
  my --> db3[(shop)]
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

With Postgres + Timescale aliases (v2.3.0 / #999):

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

`PREST_DATABASE_ALIAS_N` and `PREST_DATABASE_URL_N` are accepted as aliases of the above keys. A `DATABASE_URL_N` that starts with `mysql://` registers a MySQL alias. From v2.5.2, env entries inherit the global `maxopenconn` / `maxidleconn`.

See [`install-manifests/kubernetes/deployment.yaml`](https://github.com/prest/prest/blob/main/install-manifests/kubernetes/deployment.yaml) for a multi-secret example with liveness/readiness probes.

### TOML (local development)

`pg.*` remains the default/fallback profile; registry entries override host, port, and credentials per alias:

```toml
[pg]
database = "prest-test"

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

[[databases]]
alias = "shop"
engine = "mysql"
url = "mysql://prest:prest@localhost:3306/shop"
```

Each entry sets its own `engine`, or inherits the root `engine` when empty. A `mysql://` URL with no `engine` is MySQL; other URLs are not probed. Since v2.3.0 (#999), startup **auto-detects** Timescale (extension present) vs Postgres for each PostgreSQL alias.

**Unreachable aliases (v2.5.2).** An entry that cannot connect at startup (network or DNS error) is registered lazily: it answers **503** `{"error":"database unavailable"}` and reconnects on first use, while the other aliases keep serving. Startup fails only when **every** entry is unreachable or the config is invalid. The default adapter is the first reachable alias. Timescale detection and `migrate_on_startup` steps skip a lazy alias until the next restart. Before v2.5.2, any unreachable entry stopped startup.

Without a registry, see [No-registry mode](#no-registry-mode-v252).

### Per-database SSL

For a MySQL entry only `mode` applies (`disable`, `require`, `skip-verify`); `cert`, `key`, or `rootcert` fail startup. See [MySQL](../databases/mysql.md#connection).

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
- Without a registry, the path segment is the physical database name (`pg.database` or a `pg.allowed_databases` entry).
- On MySQL, `{schema}` is a database: with a registry it must be the alias's own database (v2.5.1); without one it must equal `{database}` (v2.5.2). Anything else is **404**.

---

## No-registry mode (v2.5.2)

Without a `[[databases]]` registry, pREST serves `pg.database` plus the databases listed in `pg.allowed_databases`, all on the `[pg]` host with the `[pg]` credentials. PostgreSQL and MySQL both support it.

```toml
[pg]
database = "prest"
allowed_databases = ["reporting", "archive"]
```

```sh
PREST_PG_DATABASE=prest
PREST_PG_ALLOWED_DATABASES=reporting,archive
# or, read first when both are set:
PREST_DB_ALLOWED_DATABASES=reporting,archive
```

Requests for any other database are **400** `database not registered: <name>`. On MySQL, `{schema}` must equal `{database}`, so `/reporting/reporting/orders` works and `/reporting/archive/orders` is **404** `schema not served by this database`.

### Migrating from `pg.single`

`pg.single` and `PREST_PG_SINGLE` were removed in v2.5.2. They are ignored and startup logs `config key ignored: pg.single was removed; use [[databases]] or pg.allowed_databases`.

- **Allow-listed names are always served.** Before, `pg.single = true` (the default) only served `pg.database`.
- **A registry is no longer limited to `pg.database`.** Before, a registry with `pg.single = true` accepted only the default alias. Now it serves all of its aliases.
- **Arbitrary database names stop working.** Before, `pg.single = false` without a registry served any database on the `[pg]` host named in the path. Now list each one in `pg.allowed_databases`, or use a registry.
- **`/databases` and MCP `prest.list_databases` list only served databases.**
- **On MySQL without a registry,** `/{database}/{other-schema}/...` is **404**. Before, any database the MySQL user could see was reachable through `pg.database`.

---

## Timescale operators (v2.3.0)

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

Pools are keyed by connection URI; aliases that share the same URI share a pool. Connections are opened lazily on first request per alias (or at startup when adapters register, since v2.3.0 / #999). From v2.5.2, PostgreSQL keeps at most **64** distinct pools per process — a request that needs another gets **503** — and MySQL opens one pool per served database.

Configure `maxopenconn` / `maxidleconn` per `[[databases]]` entry. **Budget:** `replicas × aliases × maxopenconn` against each cluster. Use PgBouncer or RDS Proxy when many aliases are registered.

---

## Health checks

| Endpoint | Purpose | Behavior |
|----------|---------|----------|
| `GET /_health` | Liveness | Pings the default database |
| `GET /_ready` | Readiness | Pings the default database and every registered alias; **503** while any alias, including a lazily registered one, is down |

Use `/_ready` for Kubernetes readiness probes when multiple databases are registered. See [Configuring pREST — Health check](configuring-prest.md#health-check-endpoints).

Unknown or unregistered names are **400** `database not registered: <name>`. A registered alias whose database is down is **503** `database unavailable` (no driver text).

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
| `prest.list_databases` | Lists served databases only: registry aliases, or `pg.database` plus `pg.allowed_databases` (v2.5.2). `/databases` returns the same set |
| `prest.select.{database}.{schema}.{table}` | `{database}` is the alias when a registry is configured |
| `prest.select_table` / catalog tools | Accept a database/alias argument where applicable |

Auth and ACL still apply — MCP is read-only but otherwise shares the HTTP stack.

---

## Migrating from single to multi-database

1. Keep `[pg]` for local/default compatibility.
2. Add `[[databases]]` (or `DATABASE_ALIAS_N` / `DATABASE_URL_N`) with clear aliases (`primary`, `metrics`).
3. Point clients at `/{alias}/...` instead of the physical database name when using the registry.
4. Since v2.3.0 (#999), confirm Timescale aliases get `_time_bucket` if you need time-series grouping.

Single-database deployments without `[[databases]]` keep working. To serve a few more databases on the same host without a registry, use [`pg.allowed_databases`](#no-registry-mode-v252).

---

## Local testing

Multi-cluster integration tests live in [`integration/postgres/controllers/multicluster_test.go`](https://github.com/prest/prest/blob/v2.5.2/integration/postgres/controllers/multicluster_test.go). They require a second Postgres service (`PREST_PG_HOST_B`) provided by [`integration/postgres/docker-compose.yml`](https://github.com/prest/prest/blob/v2.5.2/integration/postgres/docker-compose.yml):

```bash
make test-integration
```

Timescale E2E: `make test-integration-timescaledb`. MySQL E2E, including a mixed registry and a lazy unreachable alias: `make test-integration-mysql`. See the [Development Guide](../get-prest/development-guide.md#integration-tests).

## Related

- [examples/multi-database-config.toml](examples/multi-database-config.toml)
- [Configuring pREST](configuring-prest.md)
- [TimescaleDB](../databases/timescaledb.md)
- [MySQL](../databases/mysql.md)
- [v2.5.2 release notes](../releases/v2.5.2.md)
- [MCP over HTTP](mcp-over-http.md)
- [Permissions](permissions.md)
- [PR #999](https://github.com/prest/prest/pull/999)
- [Acronyms](../prestd/acronyms.md) · [MCP](../prestd/acronyms.md#mcp) · [SQL](../prestd/acronyms.md#sql)
