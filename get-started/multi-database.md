# Multi-database

pREST uses the first URL path segment as the **database selector** for CRUD, catalog, and optional script routes. Two modes are supported:

| Mode | When | `{database}` in URL | Connection target |
|------|------|---------------------|-------------------|
| **Legacy multi-DB** | No registry configured | Postgres database name | Same `pg.host`; `dbname` = path segment |
| **Registry multi-cluster** | `[[databases]]` or env registry set | Registered **alias** | Per-profile host, port, and credentials |

---

## URL routing

All table operations use `/{database}/{schema}/{table}`:

```http
GET /tenant-a/public/users
POST /tenant-a/public/orders
GET /tenant-a/public
GET /_QUERIES/tenant-a/myqueries/get_all
```

Script routes accept an optional database prefix (`/_QUERIES/{database}/{queriesLocation}/{script}`). When omitted, the default database (`pg.database`) is used.

Request flow: validate alias → set connection context → open or reuse pool for that alias → execute query.

---

## Configuration

Registry sources are merged in priority order: **indexed env pairs → TOML** (env wins on conflict).

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

When no registry is configured, legacy `DATABASE_URL` / `pg.*` behavior is unchanged.

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

## Connection pooling

Pools are keyed by connection URI; aliases that share the same URI share a pool. Connections are opened lazily on first request per alias.

**Connection budgeting:** plan for `replicas × aliases × pg.maxopenconn` connections per cluster. Use PgBouncer or RDS Proxy when many aliases are registered.

---

## Health checks

| Endpoint | Purpose | Behavior |
|----------|---------|----------|
| `GET /_health` | Liveness | Pings the default database |
| `GET /_ready` | Readiness | Pings the default database and every registered alias |

Use `/_ready` for Kubernetes readiness probes when multiple databases are registered. See [Configuring pREST — Health check](configuring-prest.md#health-check-endpoints).

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

## Local testing

Multi-cluster integration tests live in [`integration/controllers/multicluster_test.go`](https://github.com/prest/prest/blob/main/integration/controllers/multicluster_test.go). They require a second Postgres service (`PREST_PG_HOST_B`) provided by [`docker-compose-test.yml`](https://github.com/prest/prest/blob/main/docker-compose-test.yml):

```bash
make test-integration
```

See the [Development Guide](../get-prest/development-guide.md#integration-tests) for the full test workflow.
