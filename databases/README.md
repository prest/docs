---
description: >-
  Which SQL databases pREST supports today — Native PostgreSQL, certified
  Postgres-compatible engines, and the multi-database roadmap.
---

# Databases

This page lists which SQL databases pREST supports today and what is still on the roadmap — the canonical **support labels**.

PostgreSQL is **native**. Postgres-compatible engines are **certified** or documented with caveats on that adapter. MySQL, SQLite, and SQL Server are **Roadmap** until dedicated adapters ship.

*Last updated: July 15, 2026*

---

## Support labels

| Label | Meaning |
|-------|---------|
| **Native** | First-class dialect adapter |
| **Certified** | Runs on the native adapter with a published features matrix |
| **Compatible with caveats** | Connects via PostgreSQL wire protocol; known gaps documented |
| **Hosted PostgreSQL** | Managed PostgreSQL — same native adapter |
| **Roadmap** | Planned — not available yet |
| **Experimental** | Incomplete; use only with explicit expectations |

---

## PostgreSQL family (available)

| Engine | Label | Page |
|--------|-------|------|
| PostgreSQL | Native | [postgresql.md](postgresql.md) |
| Amazon Aurora PostgreSQL | Hosted PostgreSQL | [aurora-postgresql.md](aurora-postgresql.md) |
| Neon, Supabase, AlloyDB | Hosted PostgreSQL | Same adapter as PostgreSQL — use standard `DATABASE_URL` / `pg.*` |
| CockroachDB | Certified (PG wire) | [cockroachdb.md](cockroachdb.md) |
| YugabyteDB (YSQL) | Certified (PG wire) | [yugabytedb.md](yugabytedb.md) |
| TimescaleDB | Compatible with caveats | [timescaledb.md](timescaledb.md) |
| Amazon Redshift | Compatible with caveats | [amazon-redshift.md](amazon-redshift.md) |

Deep how-tos also live under [Integrations](../integrations/README.md).

---

## Roadmap families

| Engine | Label | Page |
|--------|-------|------|
| MySQL / MariaDB / TiDB / Aurora MySQL | Roadmap (Phase 2) | [mysql.md](mysql.md) |
| SQLite | Roadmap (Phase 3) | [sqlite.md](sqlite.md) |
| SQL Server / Azure SQL | Roadmap (Phase 4) | [sql-server.md](sql-server.md) |
| Oracle | Roadmap (Phase 5) | See [roadmap.md](roadmap.md) |
| Analytical (DuckDB, ClickHouse, Snowflake, BigQuery) | Separate read-only track | See [roadmap.md](roadmap.md) |

Full phases and architecture notes: [Database roadmap](roadmap.md).

---

## FAQ

### What databases does pREST support?

Today: **PostgreSQL** (native) and engines that use the PostgreSQL wire protocol, with per-engine matrices. Other SQL families are planned — they are not installable yet.

### Is Aurora / Neon / Supabase supported?

Yes, as **Hosted PostgreSQL**. Use the same connection settings as PostgreSQL. See [Aurora PostgreSQL](aurora-postgresql.md).

### When will MySQL or SQLite work?

They are Phase 2 and Phase 3 on the [roadmap](roadmap.md). Do not expect a MySQL or SQLite connection string to work with current releases.

---

## Related

- [Homepage](../README.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Integrations](../integrations/README.md)
