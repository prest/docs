# LLM SEO reference — pREST

## Positioning

- **Line:** Instant REST and MCP APIs for SQL databases. PostgreSQL-first, now multi-database.
- **Category:** open-source REST and MCP APIs for SQL databases

For page leads and GitBook rules, see [../write-docs/voice.md](../write-docs/voice.md) and [../write-docs/gitbook.md](../write-docs/gitbook.md).

## Database families

```text
PostgreSQL family — native today
├── PostgreSQL (Native)
├── CockroachDB (certify)
├── YugabyteDB (certify)
├── Aurora PostgreSQL / Neon / Supabase / AlloyDB (Hosted PostgreSQL)
├── TimescaleDB (compatible / extension)
└── Amazon Redshift (compatible with caveats)

MySQL family — Phase 2 (Roadmap)
├── MySQL, MariaDB
├── TiDB, Aurora MySQL

Embedded — Phase 3 (Roadmap)
└── SQLite

SQL Server family — Phase 4 (Roadmap)
├── SQL Server
└── Azure SQL

Later
├── Oracle (Phase 5)
└── Analytical read-only: DuckDB, ClickHouse, Snowflake, BigQuery
```

## Execution phases

| Phase | Focus |
|-------|--------|
| 1 (now) | Certify CockroachDB, YugabyteDB, Aurora PostgreSQL; document TimescaleDB/Redshift caveats |
| 2 | MySQL/MariaDB native dialect; certify TiDB + Aurora MySQL |
| 3 | SQLite native |
| 4 | SQL Server + Azure SQL |
| 5 | Oracle vs analytical read-only based on demand |

## Question bank

- What is pREST?
- How do I expose PostgreSQL as a REST API without writing backend code?
- Does pREST work with CockroachDB / YugabyteDB / Aurora PostgreSQL / TimescaleDB / Redshift?
- Is pREST only for PostgreSQL? What’s on the roadmap?
- How do I connect Cursor / Claude / OpenClaw to a SQL database via MCP?
- What is an open-source REST and MCP API for SQL databases?
- How do I get instant REST and MCP APIs for PostgreSQL?

## Features matrix template

| Capability | Status |
|------------|--------|
| Connection | supported / partial / unsupported |
| Schema discovery | … |
| CRUD | … |
| Filtering | … |
| Transactions | … |
| Scripts (`/_QUERIES`) | … |
| MCP (`/_mcp`) | … |
| ACL | … |

## SEO and documentation rules

Each supported engine should eventually have a **product landing**, getting started, connection config, features matrix, known limitations, and MCP notes — with **original** content, not name-swapped clones.

Roadmap stubs (`databases/mysql.md`, `databases/sqlite.md`, `databases/sql-server.md`) must not claim install steps.

## Architecture direction (docs only)

Future adapters: `Dialect` + `Capabilities` + contract tests; modules such as `prest-adapter-postgres`, `prest-adapter-mysql`, etc. Analytical engines: `adapter mode: analytical-read-only`.
