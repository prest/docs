---
description: >-
  Use Amazon Aurora PostgreSQL with pREST — same native PostgreSQL adapter,
  REST CRUD and MCP over the PostgreSQL wire protocol.
---

# Amazon Aurora PostgreSQL

**pREST** works with **Amazon Aurora PostgreSQL** as **Hosted PostgreSQL**: the same **Native** PostgreSQL adapter, pointed at your Aurora cluster endpoint.

*Last updated: July 11, 2026* · **Label:** Hosted PostgreSQL

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported (Aurora PG endpoint + SSL as required) |
| Schema discovery | supported |
| CRUD | supported |
| Filtering / ordering / pagination | supported |
| Transactions | supported |
| Scripts (`/_QUERIES`) | supported |
| MCP (`/_mcp`) | supported (v2.1.0+) |
| ACL / permissions | supported |

Same matrix as [PostgreSQL](postgresql.md). Aurora MySQL is **not** supported until the MySQL adapter ships — see [MySQL roadmap](mysql.md).

---

## Connection

Use the Aurora **PostgreSQL** writer/reader endpoint with TLS appropriate for your VPC:

```sh
export PREST_VERSION=2
export PREST_PG_URL='postgres://user:pass@my-cluster.cluster-xxxx.region.rds.amazonaws.com:5432/mydb?sslmode=require'
```

Or set `PREST_PG_HOST`, `PREST_PG_PORT`, `PREST_PG_USER`, `PREST_PG_PASS`, `PREST_PG_DATABASE`, and `PREST_PG_SSL_MODE`.

For multi-tenant / multi-cluster setups, register aliases under `[[databases]]` or `DATABASE_ALIAS_N` / `DATABASE_URL_N` — [Multi-database](../get-started/multi-database.md).

---

## CRUD and MCP

Identical to native PostgreSQL:

- REST: `/{database}/{schema}/{table}` — [API Reference](../api-reference/README.md)
- MCP: `/_mcp` — [MCP over HTTP](../get-started/mcp-over-http.md)

---

## Known limitations

- Use the **PostgreSQL-compatible** Aurora engine, not Aurora MySQL.
- Network access (security groups, private subnets, SSL) is your responsibility.
- Aurora-specific extensions or version skew may differ from community PostgreSQL; validate critical queries in staging.
- Reader endpoints are fine for read-heavy REST/MCP traffic; send writes to the writer endpoint.

---

## Related

- [PostgreSQL](postgresql.md)
- [Databases](README.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Database roadmap](roadmap.md)
