---
description: >-
  Changes on prest/prest main since v2.1.0 — database-backed custom queries
  (#980), TimescaleDB E2E (#988), sample TOML (#978). Not in a release tag yet.
---

# Changes since v2.1.0

{% hint style="warning" %}
These commits are on [`prest/prest` `main`](https://github.com/prest/prest/tree/main) and are **not included** in the [v2.1.0](v2.1.0.md) release tag. Do not treat them as installable until a newer tag ships. Compare: [v2.1.0...main](https://github.com/prest/prest/compare/v2.1.0...main).
{% endhint %}

| Commit | PR | Summary |
|--------|-----|---------|
| `8313b87` | [#980](https://github.com/prest/prest/pull/980) | Database-backed custom query storage, `/_QUERIES/registry`, query ACL |
| `35eeedd` | [#988](https://github.com/prest/prest/pull/988) | TimescaleDB E2E certification on the native PostgreSQL adapter |
| `5fb0290` | [#978](https://github.com/prest/prest/pull/978) | Fully documented [`samples/prest.sample.toml`](https://github.com/prest/prest/blob/main/samples/prest.sample.toml) |
| `e8cdcac` | [#979](https://github.com/prest/prest/pull/979) | Postgres / Timescale test images bumped toward Postgres 18 |

Also on this range: coverage reporting for fork PRs ([#984](https://github.com/prest/prest/pull/984)) and README/docs updates ([#986](https://github.com/prest/prest/pull/986)).

---

## #980 — Database-backed custom queries

Adds `queries.storage = "database"` as an alternative to filesystem `.sql` scripts (default remains `filesystem`).

User-facing surface:

- Table `prest_queries` (configurable `schema` / `table`)
- Admin API `/_QUERIES/registry` when `register_enabled = true`
- Query ACL: `queries.restrict`, `[[queries.scripts]]`, `[[queries.users]]`
- Startup migrate/import from filesystem; CLI `prestd migrate up|down queries`

See [Custom Queries](../api-reference/custom-queries.md#database-backed-storage-main).

---

## #988 — TimescaleDB E2E

Certifies TimescaleDB through the existing PostgreSQL adapter (no separate adapter). Adds `make test-integration-timescaledb`, Compose under `integration/timescaledb/`, and hypertable / catalog caveats.

Still labeled **Compatible with caveats** in docs. See [TimescaleDB](../databases/timescaledb.md) and upstream [DIFFERENCES.md](https://github.com/prest/prest/blob/main/integration/timescaledb/DIFFERENCES.md).

---

## #978 — Sample configuration

[`samples/prest.sample.toml`](https://github.com/prest/prest/blob/main/samples/prest.sample.toml) documents every accepted key (including `[https]` vs `[pg.ssl]` and the new `[queries]` options). Prefer it as the config reference alongside [Configuring pREST](../get-started/configuring-prest.md).

## Related

- [v2.1.0 release notes](v2.1.0.md)
- [Releases](README.md)
- [Custom Queries](../api-reference/custom-queries.md)
- [TimescaleDB](../databases/timescaledb.md)
