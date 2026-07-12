---
description: >-
  CockroachDB REST and MCP via pREST — PostgreSQL wire-compatible, with a
  documented support matrix and known dialect differences.
---

# CockroachDB

**pREST** can expose **CockroachDB** through the **Native** PostgreSQL adapter because CockroachDB implements the PostgreSQL wire protocol. Treat support as **Certified (PG wire)** with the matrix below — not identical to community PostgreSQL in every dialect detail.

*Last updated: July 11, 2026* · **Label:** Certified (PostgreSQL wire)

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported (PG connection string / port) |
| Schema discovery | supported (validate against your CRDB version) |
| CRUD | supported for standard table operations |
| Filtering / ordering / pagination | supported |
| Transactions | supported with CockroachDB transaction semantics |
| Scripts (`/_QUERIES`) | supported for SQL accepted by CockroachDB |
| MCP (`/_mcp`) | supported (v2.1.0+) when catalog/query paths succeed |
| ACL / permissions | supported (pREST ACL layer) |

---

## Connection

Default CockroachDB SQL port is often **26257**:

```sh
export PREST_VERSION=2
export PREST_PG_URL='postgres://root@localhost:26257/defaultdb?sslmode=disable'
```

For secure clusters, use TLS settings required by your deployment (`sslmode=verify-full`, client certs, etc.).

Docker smoke example (illustrative):

```sh
docker run -d --name crdb -p 26257:26257 cockroachdb/cockroach:latest start-single-node --insecure
docker run -d -p 3000:3000 \
  -e PREST_VERSION=2 \
  -e PREST_PG_URL='postgres://root@host.docker.internal:26257/defaultdb?sslmode=disable' \
  -e PREST_DEBUG=true \
  prest/prest:v2.1.0
```

Adjust host networking for your OS/Docker setup.

---

## CRUD example

```http
GET /defaultdb/public/users
```

Path segments follow pREST’s `/{database}/{schema}/{table}` convention — [API Reference](../api-reference/README.md).

---

## MCP

Point MCP clients at the same `prestd` `/_mcp` endpoint — [MCP over HTTP](../get-started/mcp-over-http.md). Prefer a read-only DB role for agent traffic.

---

## Known limitations / incompatibilities

- Not every PostgreSQL extension or catalog quirk exists in CockroachDB.
- Some PostgreSQL-specific SQL (types, functions, `RETURNING` edge cases, advanced DDL) may fail; use [custom queries](../api-reference/custom-queries.md) only when CockroachDB accepts the SQL.
- Serial/identity and migration patterns differ from PostgreSQL — design schemas with CockroachDB docs in mind.
- Run your own integration checks before production; Phase 1 certification means **documented** use of the PG adapter, not a claim of 100% PostgreSQL parity.

---

## Related

- [Databases](README.md)
- [PostgreSQL](postgresql.md)
- [YugabyteDB](yugabytedb.md)
- [Roadmap](roadmap.md)
