---
description: >-
  TimescaleDB with pREST — PostgreSQL extension for time-series data, REST and
  MCP via the native PostgreSQL adapter, with documented caveats.
---

# TimescaleDB

**TimescaleDB** is a PostgreSQL extension for time-series workloads. **pREST** treats it as **Compatible with caveats**: you connect like PostgreSQL and use hypertables as tables, but warehouse/time-series specifics may need custom SQL.

*Last updated: July 15, 2026* · **Label:** Compatible with caveats

{% hint style="info" %}
On prest `main`, TimescaleDB has first-class E2E coverage through the native PostgreSQL adapter ([#988](https://github.com/prest/prest/pull/988)) — still **Compatible with caveats**, not a separate adapter. See [Changes since v2.1.0](../releases/main-since-v2.1.0.md) and upstream [DIFFERENCES.md](https://github.com/prest/prest/blob/main/integration/timescaledb/DIFFERENCES.md).
{% endhint %}

Full Compose example: [Integrations — TimescaleDB](../integrations/timescaledb.md). Prefer `timescale/timescaledb:latest-pg18` (or another Timescale-supported PG major) and `CREATE EXTENSION IF NOT EXISTS timescaledb`.

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported (Postgres protocol) |
| Schema discovery | supported — hypertables and `_timescaledb_*` catalogs may appear |
| CRUD | supported on underlying relations; hypertable semantics apply |
| Filtering / ordering / pagination | supported |
| Transactions | supported (PostgreSQL) |
| Scripts (`/_QUERIES`) | supported — preferred for Timescale-specific SQL |
| MCP (`/_mcp`) | supported (v2.1.0+) for readable relations |
| ACL / permissions | supported — you may need allowlists for Timescale system schemas |

---

## Connection

Same as PostgreSQL — point `PREST_PG_*` / `PREST_PG_URL` at a TimescaleDB (or Timescale Cloud) instance. See the [integration Compose file](../integrations/timescaledb.md) for a local stack.

```sh
export PREST_VERSION=2
export PREST_PG_HOST=timescaledb
export PREST_PG_USER=prest
export PREST_PG_PASS=prest
export PREST_PG_DATABASE=prest
export PREST_PG_SSL_MODE=disable
```

---

## MCP and REST

- REST CRUD: [API Reference](../api-reference/README.md)  
- Time-bucket / continuous-aggregate queries: prefer [custom queries](../api-reference/custom-queries.md)  
- MCP: [MCP over HTTP](../get-started/mcp-over-http.md)

---

## Known limitations

- Timescale-specific functions (`time_bucket`, etc.) are not first-class REST operators — use `/_QUERIES` templates.
- Compression, retention policies, and continuous aggregates are database features outside pREST’s generic CRUD model.
- Catalog listing may include hypertables, chunk children, and `_timescaledb_*` schemas — filter with [access](../get-started/permissions.md) / expose settings as needed.
- Use a PostgreSQL version Timescale supports for your image/cloud tier (docs examples use PG 18 images).

---

## Related

- [Integrations — TimescaleDB](../integrations/timescaledb.md)
- [PostgreSQL](postgresql.md)
- [Databases](README.md)
- [Changes since v2.1.0](../releases/main-since-v2.1.0.md)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [MCP](../prestd/acronyms.md#mcp)
