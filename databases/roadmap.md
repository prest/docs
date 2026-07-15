---
description: >-
  pREST multi-database roadmap — PostgreSQL family first, then MySQL, SQLite,
  SQL Server; analytical databases on a separate read-only track.
---

# Database roadmap

This page describes **planned** SQL adapters for pREST. Nothing here is installable until a release ships it. PostgreSQL is native today — see [Databases](README.md).

*Last updated: July 15, 2026*

---

## Why this order?

Developer usage (for example Stack Overflow surveys) weights **PostgreSQL, MySQL, SQLite, and SQL Server** heavily for application builders. Broader market-visibility rankings put Oracle and warehouses higher, but they are a poorer early fit for open-source adoption per engineering hour.

pREST therefore prioritizes:

1. Certify the **PostgreSQL-compatible** family on the existing adapter  
2. **MySQL family** (high SEO + open-source fit)  
3. **SQLite** (local/file DX)  
4. **SQL Server / Azure SQL** (enterprise)  
5. **Oracle** or **analytical read-only** based on demand  

---

## Adapter families (target)

```text
PostgreSQL family — native today
├── PostgreSQL
├── CockroachDB (certify)
├── YugabyteDB (certify)
└── Aurora PostgreSQL (certify)

MySQL family — Phase 2
├── MySQL, MariaDB (dialect profiles)
├── TiDB, Aurora MySQL (certify after MySQL adapter)

Embedded family — Phase 3
└── SQLite

SQL Server family — Phase 4
├── SQL Server
└── Azure SQL

Later / separate
├── Oracle (Phase 5)
└── Analytical read-only: DuckDB, ClickHouse, Snowflake, BigQuery
```

---

## Phases

| Phase | Focus | Status |
|-------|--------|--------|
| **1** | Certify CockroachDB, YugabyteDB, Aurora PostgreSQL; TimescaleDB/Redshift caveats + Timescale E2E on native PG adapter ([#988](https://github.com/prest/prest/pull/988) on `main`) | **In progress** |
| **2** | MySQL/MariaDB native dialect; certify TiDB + Aurora MySQL | Roadmap |
| **3** | SQLite native — one file → REST/MCP | Roadmap |
| **4** | SQL Server + Azure SQL | Roadmap |
| **5** | Oracle vs analytical read-only track | Undecided |

Current support labels: [Databases](README.md).

---

## Analytical databases (separate track)

Snowflake, Databricks, BigQuery, ClickHouse, and DuckDB speak SQL but are a poor fit for generic transactional CRUD (`POST`/`PATCH`, row updates, transactions).

Planned direction:

```text
adapter mode: transactional
adapter mode: analytical-read-only
```

Analytical mode would emphasize schema exploration, controlled queries, scripts, and MCP — not forced row-level CRUD.

---

## Architecture direction

Future adapters should isolate **SQL generation and schema behavior**, not only connection drivers. Design direction (not yet implemented in this docs pass):

- Explicit `Dialect` / `Capabilities` interfaces  
- Shared contract-test suite (discovery, CRUD, pagination, JSON, MCP, ACL)  
- Separate modules where practical (`prest-adapter-postgres`, `prest-adapter-mysql`, …)  

---

## Related

- [Databases](README.md)
- [Homepage](../README.md)
- [Key features](../prestd/prestd-key-features.md)
