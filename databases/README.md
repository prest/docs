---
description: >-
  Which SQL databases pREST supports today — Native PostgreSQL and MySQL 8,
  certified Postgres-compatible engines, and the remaining roadmap.
---

# Databases

This page lists which SQL databases pREST supports today and what is still on the roadmap — the canonical **support labels**.

PostgreSQL is **native**. MySQL 8.0.19+ is a separate **native** dialect from v2.5.0. Postgres-compatible engines are **certified** or documented with caveats on the PostgreSQL adapter. SQLite and SQL Server are **Roadmap**.

*Last updated: October 8, 2026*

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

---

## MySQL (available)

| Engine | Label | Page |
|--------|-------|------|
| MySQL 8.0.19+ | Native | [mysql.md](mysql.md) |

MariaDB, TiDB, and Aurora MySQL stay on the [roadmap](roadmap.md). MCP on MySQL is [v2.5.1+](../releases/v2.5.1.md).

Deep how-tos also live under [Integrations](../integrations/README.md).

---

## Roadmap families

| Engine | Label | Page |
|--------|-------|------|
| MariaDB / TiDB / Aurora MySQL | Roadmap (Phase 2) | [roadmap.md](roadmap.md) |
| SQLite | Roadmap (Phase 3) | [sqlite.md](sqlite.md) |
| SQL Server / Azure SQL | Roadmap (Phase 4) | [sql-server.md](sql-server.md) |
| Oracle | Roadmap (Phase 5) | See [roadmap.md](roadmap.md) |
| Analytical (DuckDB, ClickHouse, Snowflake, BigQuery) | Separate read-only track | See [roadmap.md](roadmap.md) |

Full phases and architecture notes: [Database roadmap](roadmap.md).

---

## FAQ

### What databases does pREST support?

Today: **PostgreSQL** (native), **MySQL 8.0.19+** (native dialect, v2.5.0), and engines that use the PostgreSQL wire protocol, with per-engine matrices. SQLite and SQL Server are planned — they are not installable yet.

### Is Aurora / Neon / Supabase supported?

Yes, as **Hosted PostgreSQL**. Use the same connection settings as PostgreSQL. See [Aurora PostgreSQL](aurora-postgresql.md).

### When will MariaDB or SQLite work?

MariaDB, TiDB, and Aurora MySQL stay on Phase 2 of the [roadmap](roadmap.md). SQLite is Phase 3. MySQL 8 itself ships in [v2.5.0](../releases/v2.5.0.md) — see [MySQL](mysql.md).

---

## Related

- [Homepage](../README.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Integrations](../integrations/README.md)
