---
description: >-
  pREST — open-source instant REST and Model Context Protocol (MCP) APIs for SQL databases.
  PostgreSQL-first, now multi-database.
---

# pREST

**pREST** is open-source software that gives you instant REST and [Model Context Protocol (MCP)](get-started/mcp-over-http.md) APIs for SQL databases. Point it at a database and get production-ready HTTP APIs — CRUD, custom SQL routes, auth, ACL, and (from v2.1.0) a read-only MCP endpoint — without hand-writing a backend.

**PostgreSQL is the first native adapter.** Postgres-compatible engines can be certified on that adapter. MySQL, SQLite, and SQL Server are on the [roadmap](databases/roadmap.md).

*Last updated: July 11, 2026*

---

## What you get

| Capability | Detail |
|------------|--------|
| Instant REST | Auto CRUD from schema: `GET/POST/PUT/PATCH/DELETE /{db}/{schema}/{table}` |
| MCP over HTTP | Read-only `/_mcp` for AI agents and IDEs ([guide](get-started/mcp-over-http.md)) |
| Auth & ACL | JWT/auth stack and table-level permissions |
| Multi-database | Alias registry across clusters ([guide](get-started/multi-database.md)) |
| Custom SQL | Templated `/_QUERIES` scripts |
| Plugins | Middleware and endpoint extensions |

---

## Which databases work today?

| Status | Engines |
|--------|---------|
| **Native** | [PostgreSQL](databases/postgresql.md) |
| **Hosted PostgreSQL** | [Aurora PostgreSQL](databases/aurora-postgresql.md), Neon, Supabase, AlloyDB |
| **Certified / certifying** | [CockroachDB](databases/cockroachdb.md), [YugabyteDB](databases/yugabytedb.md) |
| **Compatible with caveats** | [TimescaleDB](databases/timescaledb.md), [Amazon Redshift](databases/amazon-redshift.md) |
| **Roadmap** | [MySQL](databases/mysql.md), [SQLite](databases/sqlite.md), [SQL Server](databases/sql-server.md) |

Full matrix and labels: [Databases](databases/README.md). Phases: [Database roadmap](databases/roadmap.md).

---

## Latest release

**[v2.1.0](releases/v2.1.0.md)** — native MCP over HTTP (`/_mcp`), schema-aware read-only tools, auth/ACL inheritance.

- Docker: `prest/prest:v2.1.0`
- Go: `go install github.com/prest/prest/v2/cmd/prestd@v2.1.0`

See [Releases](releases/README.md) and [Upgrading to v2](get-started/upgrading-to-v2.md).

---

## Get started

1. [Get pREST](get-prest/README.md) — Docker, Homebrew, or Go
2. [Configuring pREST](get-started/configuring-prest.md)
3. [API Reference](api-reference/README.md)
4. Optional: [MCP over HTTP](get-started/mcp-over-http.md) for AI clients
5. Optional: [AI and MCP](ai/README.md) — Cursor, Claude Desktop, stdio adapter

---

## FAQ

### What is pREST?

pREST is open-source software for instant REST and MCP APIs on SQL databases. It turns HTTP requests into safe, parameterized database operations using your existing schema.

### Is pREST only for PostgreSQL?

PostgreSQL is the **native** adapter today. Engines that speak the PostgreSQL wire protocol (for example YugabyteDB, CockroachDB, Aurora PostgreSQL) can use that adapter with documented support levels. Other SQL families are planned — see the [roadmap](databases/roadmap.md).

### Does pREST support MCP for AI agents?

**MCP (Model Context Protocol)** is an open standard for connecting AI apps and agents to tools and data. pREST exposes a read-only MCP endpoint at `/_mcp` so clients can discover schemas and query tables through the same server as the REST API.

Yes, from **v2.1.0**. Stdio clients use the [pREST MCP Adapter](ai/install-prest-mcp.md) (`brew install prest/tap/prest-mcp`). Full guide: [MCP over HTTP](get-started/mcp-over-http.md) · [AI and MCP](ai/README.md).

### How do I start quickly?

Use Docker or Homebrew from [Get pREST](get-prest/README.md), set `PREST_PG_URL` (or `pg.*` / `DATABASE_URL`), and call `http://localhost:3000/{database}/{schema}/{table}`.

---

## Related documentation

- [Key features](prestd/prestd-key-features.md)
- [Databases](databases/README.md)
- [Database roadmap](databases/roadmap.md)
- [AI and MCP](ai/README.md)
- [Who uses pREST](prestd/who-uses-prest.md)
- [Contributing](prestd/contributing-to-prestd.md)

Questions? [GitHub Discussions](https://github.com/prest/prest/discussions) or [Discord](https://discord.gg/aB8mwvVEhC).
