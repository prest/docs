---
description: >-
  Key features of pREST — instant REST and MCP APIs for SQL databases,
  PostgreSQL-first native adapter, auth, ACL, multi-database, and plugins.
---

# pREST key features

**pREST** is an open-source REST and MCP gateway for SQL databases. It generates HTTP APIs from your schema so teams ship data products without writing boilerplate CRUD services. **PostgreSQL is the first native adapter**; Postgres-compatible engines are documented under [Databases](../databases/README.md).

*Last updated: July 11, 2026*

---

## Feature summary

| Feature | Why it matters |
|---------|----------------|
| Automatic REST from schema | Tables become endpoints without hand-written controllers |
| Full CRUD over HTTP | Familiar verbs map to SQL operations on the active dialect |
| MCP over HTTP | AI agents discover and read data via `/_mcp` (v2.1.0+) |
| Auth & permissions | JWT/auth stack and table-level ACL |
| Multi-database | Route by alias across clusters |
| Custom SQL routes | Templated `/_QUERIES` for curated operations |
| Plugins | Extend with middleware and custom endpoints |
| Postgres-family reach | Native PG plus certified/compatible wire engines |

---

## Automatic API generation

pREST introspects the connected database catalog and exposes resources under `/{database}/{schema}/{table}`. You model data in SQL; pREST serves the API. See [API Reference](../api-reference/README.md).

---

## CRUD over HTTP

Standard HTTP methods map to insert/select/update/delete on the **current native dialect (PostgreSQL)**. Filtering, ordering, and pagination use query parameters — [Parameters](../api-reference/parameters.md).

---

## MCP for AI agents

From v2.1.0, the same process exposes read-only MCP tools at `/_mcp` — list schemas/tables, describe columns, select rows (max 100). Guide: [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Authentication and authorization

Protect routes with JWT/auth configuration and restrict tables via `access.tables` / per-user rules — [Auth](../api-reference/auth.md), [Permissions](../get-started/permissions.md).

---

## Multi-database and SQL platform direction

- **Today:** PostgreSQL native adapter; hosted PG and PG-wire engines per [Databases](../databases/README.md).  
- **Next:** MySQL family, SQLite, SQL Server — [roadmap](../databases/roadmap.md).  

pREST is positioned as a **multi-database** API platform; adapters beyond PostgreSQL are explicit roadmap work, not silent claims.

---

## Custom queries and plugins

- [Custom queries](../api-reference/custom-queries.md)  
- [Middleware plugins](../plugins/middleware-plugin.md)  
- [Endpoint plugins](../plugins/endpoint-plugin.md)  

---

## FAQ

### Is pREST only a PostgreSQL tool?

It started with PostgreSQL and that remains the **native** adapter. The product direction is an open-source REST and MCP gateway for SQL databases — see the [roadmap](../databases/roadmap.md).

### Can I use pREST with YugabyteDB or CockroachDB?

Yes, via the PostgreSQL wire path with documented labels and matrices — [YugabyteDB](../databases/yugabytedb.md), [CockroachDB](../databases/cockroachdb.md).

---

## Related

- [Homepage](../README.md)
- [Databases](../databases/README.md)
- [Get Started](../get-started/README.md)
- [Who uses pREST](who-uses-prest.md)
