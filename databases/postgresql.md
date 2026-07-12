---
description: >-
  Expose PostgreSQL as a REST and MCP API with pREST — native adapter,
  auto CRUD, auth, ACL, and read-only MCP at /_mcp.
---

# PostgreSQL

**pREST** turns a PostgreSQL database into an instant REST API and (from v2.1.0) a read-only MCP endpoint. PostgreSQL is the **Native** adapter — the first and primary dialect implementation.

*Last updated: July 11, 2026* · **Label:** Native

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported |
| Schema discovery | supported |
| CRUD | supported |
| Filtering / ordering / pagination | supported |
| Transactions | supported |
| Scripts (`/_QUERIES`) | supported |
| MCP (`/_mcp`) | supported (v2.1.0+) |
| ACL / permissions | supported |

---

## Connection

```sh
export PREST_VERSION=2
export PREST_PG_URL='postgres://user:pass@localhost:5432/mydb?sslmode=disable'
# or discrete PREST_PG_HOST / PREST_PG_USER / PREST_PG_PASS / PREST_PG_DATABASE
```

TOML `[pg]` and multi-database `[[databases]]` are documented in [Configuring pREST](../get-started/configuring-prest.md) and [Multi-database](../get-started/multi-database.md).

---

## Quick CRUD example

```http
GET /mydb/public/users
POST /mydb/public/users
Content-Type: application/json

{"name": "ada"}
```

See [API Reference](../api-reference/README.md) and [Parameters](../api-reference/parameters.md).

---

## MCP

```http
GET /_mcp
POST /_mcp
```

Guide: [MCP over HTTP](../get-started/mcp-over-http.md). AI client setup: [docs.prestd.com/ai](https://docs.prestd.com/ai).

---

## Known limitations

- Requires PostgreSQL **9.5+** (project baseline).
- SQL dialect features beyond what the adapter generates (exotic types, some extensions) may need [custom queries](../api-reference/custom-queries.md).
- Other SQL engines are not this adapter — see [roadmap](roadmap.md).

---

## Related

- [Databases](README.md)
- [Get pREST](../get-prest/README.md)
- [Deploying with Docker](../deployment/deploying-with-docker.md)
- [Permissions](../get-started/permissions.md)
