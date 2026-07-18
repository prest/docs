---
description: >-
  Changes since v2.1.0 included in v2.2.0 — Studio, database-backed queries,
  TimescaleDB E2E, sample TOML, and related packaging work.
---

# Changes since v2.1.0 (in v2.2.0)

**Released as [v2.2.0](v2.2.0.md).** The commits below were merged after [v2.1.0](v2.1.0.md) and are included in the v2.2.0 release. Compare: [v2.1.0...v2.2.0](https://github.com/prest/prest/compare/v2.1.0...v2.2.0).

| Commit | PR | Summary |
|--------|-----|---------|
| `441dc3d` | [#990](https://github.com/prest/prest/pull/990) | pREST Studio embedded UI at `/_studio/` |
| `8313b87` | [#980](https://github.com/prest/prest/pull/980) | Database-backed custom query storage, `/_QUERIES/registry`, query ACL |
| `35eeedd` | [#988](https://github.com/prest/prest/pull/988) | TimescaleDB E2E certification on the native PostgreSQL adapter |
| `5fb0290` | [#978](https://github.com/prest/prest/pull/978) | Fully documented [`samples/prest.sample.toml`](https://github.com/prest/prest/blob/v2.2.0/samples/prest.sample.toml) |
| `e8cdcac` | [#979](https://github.com/prest/prest/pull/979) | Postgres / Timescale test images bumped toward Postgres 18 |
| `e158f89` | [#998](https://github.com/prest/prest/pull/998) | Dockerfile / GoReleaser configuration for Studio embed |

Also on this range: coverage reporting for fork PRs ([#984](https://github.com/prest/prest/pull/984)), README/docs updates ([#986](https://github.com/prest/prest/pull/986)), and a small CI fixup ([#989](https://github.com/prest/prest/pull/989)).

---

## #990 — pREST Studio

Embedded admin and explorer UI at `/_studio/` (enabled by default). Same-origin REST and MCP client; disable with `PREST_STUDIO_ENABLED=false`. See [pREST Studio](../get-started/prest-studio.md) and [v2.2.0](v2.2.0.md).

---

## #980 — Database-backed custom queries

Adds `queries.storage = "database"` as an alternative to filesystem `.sql` scripts (default remains `filesystem`).

User-facing surface:

- Table `prest_queries` (configurable `schema` / `table`)
- Admin API `/_QUERIES/registry` when `register_enabled = true`
- Query ACL: `queries.restrict`, `[[queries.scripts]]`, `[[queries.users]]`
- Startup migrate/import from filesystem; CLI `prestd migrate up|down queries`

See [Custom Queries](../api-reference/custom-queries.md#database-backed-storage-v220).

---

## #988 — TimescaleDB E2E

Certifies TimescaleDB through the existing PostgreSQL adapter (no separate adapter). Adds `make test-integration-timescaledb`, Compose under `integration/timescaledb/`, and hypertable / catalog caveats.

Still labeled **Compatible with caveats** in docs. See [TimescaleDB](../databases/timescaledb.md) and upstream [DIFFERENCES.md](https://github.com/prest/prest/blob/v2.2.0/integration/timescaledb/DIFFERENCES.md).

---

## #978 — Sample configuration

[`samples/prest.sample.toml`](https://github.com/prest/prest/blob/v2.2.0/samples/prest.sample.toml) documents every accepted key (including `[https]` vs `[pg.ssl]`, `[studio]`, and `[queries]`). Prefer it as the config reference alongside [Configuring pREST](../get-started/configuring-prest.md).

## Related

- [v2.2.0 release notes](v2.2.0.md)
- [v2.1.0 release notes](v2.1.0.md)
- [Releases](README.md)
- [pREST Studio](../get-started/prest-studio.md)
- [Custom Queries](../api-reference/custom-queries.md)
- [TimescaleDB](../databases/timescaledb.md)
