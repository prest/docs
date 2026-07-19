---
description: >-
  Key features of pREST — instant REST and MCP APIs for SQL databases,
  PostgreSQL-first native adapter, auth, ACL, multi-database, and plugins.
---

# pREST key features

**pREST** is a open-source software that generates HTTP APIs from your schema so teams ship data products without writing boilerplate CRUD services — instant REST and MCP APIs for SQL databases. **PostgreSQL is the first native adapter**; Postgres-compatible engines are documented under [Databases](../databases/README.md).

*Last updated: July 18, 2026*

---

## Feature summary

| Feature | Why it matters |
|---------|----------------|
| Automatic REST from schema | Tables become endpoints without hand-written controllers |
| Full CRUD over HTTP | Familiar verbs map to SQL operations on the active dialect |
| MCP over HTTP | AI agents discover and read data via `/_mcp` (v2.1.0+) |
| pREST Studio | Embedded UI at `/_studio/` — catalog, REST, and MCP explorers (v2.2.0+) |
| Auth & permissions | JWT/auth stack and table-level ACL |
| Multi-database | Route by alias across clusters; Timescale adapter auto-detect on `main` ([#999](https://github.com/prest/prest/pull/999)) |
| Custom SQL routes | Templated `/_QUERIES` for curated operations |
| Plugins | Extend with middleware and custom endpoints |
| Postgres-family reach | Native PG plus certified/compatible wire engines |

---

## Automatic API generation

**REST (Representational State Transfer)** is an architectural style for building APIs over HTTP. pREST exposes your SQL tables as REST resources at `/{database}/{schema}/{table}`, mapping standard verbs to CRUD on the same server as MCP.

pREST introspects the connected database catalog and exposes those resources automatically. You model data in SQL; pREST serves the API. See [API Reference](../api-reference/README.md).

---

## CRUD over HTTP

Standard HTTP methods map to insert/select/update/delete on the **current native dialect (PostgreSQL)**. Filtering, ordering, and pagination use query parameters — [Parameters](../api-reference/parameters.md).

---

## MCP for AI agents

**Artificial intelligence (AI)** here means applications and agents that use models to reason and act on tools and data. **MCP (Model Context Protocol)** is an open standard for connecting AI apps and agents to tools and data. pREST exposes a read-only MCP endpoint at `/_mcp` so clients can discover schemas and query tables through the same server as the REST API.

From v2.1.0, the same process exposes read-only MCP tools — list schemas/tables, describe columns, select rows (max 100). Guide: [MCP over HTTP](../get-started/mcp-over-http.md).

---

## pREST Studio

From v2.2.0, open `/_studio/` for an embedded catalog / REST / MCP explorer UI (read-only). Guide: [pREST Studio](../get-started/prest-studio.md).

---

## Authentication and authorization

Protect routes with JWT/auth configuration and restrict tables via `access.tables` / per-user rules — [Auth](../api-reference/auth.md), [Permissions](../get-started/permissions.md).

---

## Multi-database and SQL platform direction

- **Today:** PostgreSQL native adapter; Timescale wire + E2E; hosted PG and PG-wire engines per [Databases](../databases/README.md). Multi-adapter routing (Postgres + Timescale) on prest `main` ([#999](https://github.com/prest/prest/pull/999)) — see [Multi-database](../get-started/multi-database.md).  
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

It started with PostgreSQL and that remains the **native** adapter. The product direction is instant REST and MCP APIs for SQL databases — see the [roadmap](../databases/roadmap.md).

### Can I use pREST with YugabyteDB or CockroachDB?

Yes, via the PostgreSQL wire path with documented labels and matrices — [YugabyteDB](../databases/yugabytedb.md), [CockroachDB](../databases/cockroachdb.md).

---

## Related

- [Homepage](../README.md)
- [Acronyms](acronyms.md) ([REST](acronyms.md#rest), [MCP](acronyms.md#mcp), [AI](acronyms.md#ai))
- [Databases](../databases/README.md)
- [Get Started](../get-started/README.md)
- [Who uses pREST](who-uses-prest.md)
