---
description: >-
  TimescaleDB with pREST — hypertables over the Postgres wire, auto-detect
  adapter on main (#999), _time_bucket, REST and MCP.
---

# TimescaleDB

**TimescaleDB** is a PostgreSQL extension for time-series workloads. **pREST** treats it as **Compatible with caveats**: connect like PostgreSQL, use hypertables as tables, and use Timescale-aware operators when the Timescale adapter is selected.

*Last updated: July 18, 2026* · **Label:** Compatible with caveats

{% hint style="info" %}
**v2.2.0+:** E2E coverage on the native PostgreSQL adapter ([#988](https://github.com/prest/prest/pull/988)). See [v2.2.0](../releases/v2.2.0.md).

**`main` / [#999](https://github.com/prest/prest/pull/999):** pREST can **auto-detect** Timescale (extension check) and select a Timescale adapter with `_time_bucket` and system-schema filtering. Still not a separate dialect family — wire-compatible with PostgreSQL. MySQL/SQLite multi-adapter is roadmap.
{% endhint %}

Full Compose example: [Integrations — TimescaleDB](../integrations/timescaledb.md). Prefer `timescale/timescaledb:latest-pg18` and `CREATE EXTENSION IF NOT EXISTS timescaledb`.

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported (Postgres protocol) |
| Schema discovery | supported — hypertables; `_timescaledb_*` hidden by default on Timescale adapter (`main`) |
| CRUD | supported on underlying relations; hypertable semantics apply |
| Filtering / ordering / pagination | supported |
| `_time_bucket` | **`main` / #999** — Timescale adapter only (`5m`…`1y`) |
| Transactions | supported (PostgreSQL) |
| Scripts (`/_QUERIES`) | supported — preferred for policies, continuous-aggregate DDL |
| MCP (`/_mcp`) | supported (v2.1.0+) for readable relations |
| ACL / permissions | supported |

---

## Connection

Same as PostgreSQL — point `PREST_PG_*` / `PREST_PG_URL` at a TimescaleDB (or Timescale Cloud) instance, or register an alias in [Multi-database](../get-started/multi-database.md).

```sh
export PREST_VERSION=2
export PREST_PG_HOST=timescaledb
export PREST_PG_USER=prest
export PREST_PG_PASS=prest
export PREST_PG_DATABASE=prest
export PREST_PG_SSL_MODE=disable
```

On `main` / #999, startup tries the Timescale adapter first, then falls back to Postgres.

---

## Time bucketing (`main` / #999)

```bash
curl "http://localhost:3000/prest/public/readings?_time_bucket=1h"
curl "http://localhost:3000/prest/public/readings?_time_bucket=1h,measured_at"
```

Intervals: `5m`, `15m`, `1h`, `6h`, `1d`, `7d`, `30d`, `1y`. Include Timescale system schemas with `_include_system_schemas=true`.

---

## MCP and REST

- REST CRUD: [API Reference](../api-reference/README.md)  
- Continuous aggregates / policies: [custom queries](../api-reference/custom-queries.md) or SQL  
- Multi-alias setups: [Multi-database](../get-started/multi-database.md)  
- MCP: [MCP over HTTP](../get-started/mcp-over-http.md)

---

## Known limitations

- Stock Postgres adapter (v2.2.0 and earlier paths) does not implement `_time_bucket` — use `/_QUERIES` or upgrade to a build with #999.
- Compression, retention policies, and continuous-aggregate **DDL** stay outside generic CRUD.
- Catalog listing may include hypertables and chunk children; use ACL / expose settings as needed.
- Use a PostgreSQL version Timescale supports for your image/cloud tier (docs examples use PG 18 images).

---

## Related

- [Integrations — TimescaleDB](../integrations/timescaledb.md)
- [Multi-database](../get-started/multi-database.md)
- [PostgreSQL](postgresql.md)
- [Databases](README.md)
- [v2.2.0 release notes](../releases/v2.2.0.md)
- [PR #999](https://github.com/prest/prest/pull/999)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [MCP](../prestd/acronyms.md#mcp)
