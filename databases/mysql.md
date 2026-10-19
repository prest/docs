---
description: >-
  Use MySQL 8.0.19+ with pREST — native dialect from v2.5.0, MCP from v2.5.1.
  REST CRUD, alias schema scope, and documented gaps. MariaDB is not this adapter.
---

# MySQL

**pREST** v2.5.0+ exposes **MySQL 8.0.19+** through a native dialect adapter. Set `engine = "mysql"`. An empty `engine` stays PostgreSQL.

*Last updated: October 18, 2026* · **Label:** Native (MySQL 8.0.19+)

MariaDB, TiDB, and Amazon Aurora MySQL are **not** this adapter. MCP (`/_mcp`) is supported from **v2.5.1**, read-only.

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | `engine = "mysql"`, or a `[[databases]]` URL starting with `mysql://` and no `engine` |
| Schema discovery | supported (`information_schema`; system schemas hidden) |
| CRUD | supported, with write differences below |
| Filtering / ordering / pagination | supported (`LIMIT` / `OFFSET`) |
| Scripts (`/_QUERIES`) | supported — `?` placeholders and backtick identifiers |
| ACL / permissions | same TOML allowlist as PostgreSQL |
| MCP (`/_mcp`) | supported (v2.5.1+), read-only, backticks / `?` |
| pgvector / ltree / `tsquery` | unsupported operators |

---

## Connection

```toml
engine = "mysql"

[pg]
url = "mysql://user:pass@localhost:3306/mydb"
```

Or environment variables. `PREST_DB_*` is read first, then legacy `PREST_PG_*`:

```sh
export PREST_VERSION=2
export PREST_ENGINE=mysql
export PREST_DB_URL='mysql://user:pass@localhost:3306/mydb'
```

`engine = "mysql"` (or `PREST_ENGINE=mysql`) is required: a root `mysql://` URL without it is ignored and pREST falls back to PostgreSQL defaults. TLS can also be set in the URL with `?tls=false`, `?tls=true`, or `?tls=skip-verify`. Port defaults to **3306** when unset. Postgres defaults (user `postgres`, database `prest`, port 5432) are not applied. A missing user or database fails startup.

TLS: `disable`, `require`, or `skip-verify`. A client cert, key, or root certificate fails startup in this release.

`[[databases]]` entries can set `engine = "mysql"` per alias. See [Multi-database](../get-started/multi-database.md).

---

## Paths

Routes stay `/{database}/{schema}/{table}`.

- `{database}` selects the pool: a registry alias, or — without a registry — `pg.database` or a name in `pg.allowed_databases` (v2.5.2). Any other name is **400** `database not registered`.
- `{schema}` is the MySQL database. Without a registry it must equal `{database}` (v2.5.2); otherwise the request is **404** `schema not served by this database`.
- `/databases` and `/schemas` list only served databases.
- Hidden schemas: `mysql`, `information_schema`, `performance_schema`, `sys`.

---

## Registry

From **v2.5.1** ([#1056](https://github.com/prest/prest/pull/1056)):

- A `[[databases]]` alias serves only the database named in its `url` or `database`. Any other `{schema}` is **404** on CRUD, `/show`, and `/{alias}/{schema}` table listing.
- In registry mode `/databases` returns `[{"datname": alias, "name": alias, "physical_name": database}]` instead of every database on the server.
- From **v2.5.2**, `/schemas` in registry mode lists only the MySQL entries on the same host and port, not every database the login can see.
- From **v2.5.2**, an entry that is unreachable at startup answers **503** `database unavailable` until it connects, instead of stopping startup.
- Without a `[[databases]]` block, see [Paths](#paths): only `pg.database` and `pg.allowed_databases` are served. Before v2.5.2 the single connection served every granted database as `{schema}`.
- A `[[databases]]` entry whose `url` starts with `mysql://` and has **no** `engine` is MySQL. Any other URL is not probed. Empty root `engine` stays PostgreSQL.

---

## Known limitations

- **MySQL 8.0.19+.** The upsert form needs the row alias added in 8.0.19. MariaDB is out of scope.
- **No SQL `RETURNING`.** Insert is followed by `SELECT *` in the same transaction. `UPDATE`/`DELETE` without `_returning` return `{"rows_affected": n}`. With `_returning`, pREST emulates the row image inside a `REPEATABLE READ` transaction.
- **`Prest-Batch-Method: copy`** is a transactional multi-row `INSERT`, not `LOAD DATA`. Batch insert uses the union of keys. A missing key is SQL `DEFAULT`.
- **No `FULL` join.** `INNER`, `LEFT`, `RIGHT`, and `CROSS` are generated.
- **`$ilike` is `LIKE`.** Case follows the column collation. pREST does not wrap the column in `LOWER()`.
- **`$all`, ltree, `$tsquery`, and `vecdist` error** with `unsupported operator`.
- **Scripts** use `?` placeholders and backtick `ident`, not Postgres `$n` and double quotes.
- **Connection failure** (down, refused, timed out) is **503** `{"error":"database unavailable"}` with no driver text.
- **MySQL 1146, 1142, 1044, and 1049** are **404**. For 1142, 1044, and 1049 the server text is dropped because it names the account.
- **`DATETIME` has no zone** (`YYYY-MM-DDTHH:MM:SS`). **`TIMESTAMP` is UTC** with `+00:00`. `DATE` is `YYYY-MM-DD`.
- **`DECIMAL` / `NUMERIC` stay strings**, so `SUM` / `AVG` of an integer column is a string.

Dialect matrix in the server repo: [`integration/mysql/DIFFERENCES.md`](https://github.com/prest/prest/blob/v2.5.2/integration/mysql/DIFFERENCES.md).

---

## Related

- [v2.5.2 release notes](../releases/v2.5.2.md)
- [v2.5.1 release notes](../releases/v2.5.1.md)
- [v2.5.0 release notes](../releases/v2.5.0.md)
- [Multi-database](../get-started/multi-database.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Databases](README.md)
- [Database roadmap](roadmap.md)
- [API Reference](../api-reference/README.md)
- [Acronyms](../prestd/acronyms.md) ([MCP](../prestd/acronyms.md#mcp), [REST](../prestd/acronyms.md#rest))
