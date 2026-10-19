---
description: >-
  Query Guard rejects expensive queries before PostgreSQL runs them — inspect the
  execution plan, refuse sequential scans, cost and row ceilings, and require
  indexed access on databases exposed to third parties.
---

# Query Guard

Query Guard inspects the **execution plan** of a generated query and refuses it when the plan violates a policy you configure. It is opt-in and disabled by default.

{% hint style="warning" %}
Available on unreleased [`prest/prest` `main`](https://github.com/prest/prest/tree/main). Not in a release tag yet — see [Changes since v2.1.0](../releases/main-since-v2.1.0.md).
{% endhint %}

## Why

[RBAC](permissions.md) answers *who* may read a table. It says nothing about *how expensive* the read is.

When you expose _**prestd**_ to third parties — vendors, BI tools, [AI agents](../ai/README.md) — a client filtering an unindexed column over a table with millions of rows produces repeated sequential scans and real database load. Without Query Guard the only defence is hand-writing [custom queries](../api-reference/custom-queries.md) for every endpoint.

## How it works

1. _**prestd**_ generates the SQL for the request as usual.
2. Before executing it, the query is planned with `EXPLAIN (FORMAT JSON)`.
3. The plan is matched against the configured rules.
4. A violation answers `422 Unprocessable Entity`; anything else runs normally.

`EXPLAIN ANALYZE` is never used, so the statement is planned but **not executed** during the check.

> Accepted queries pay one extra planning round trip. That is the cost of the protection — enable it where database load matters more than latency.

## Rejection response

```json
{
  "error": "query rejected by Query Guard",
  "reason": "Sequential Scan detected on table 'orders'.",
  "rule": "reject_seq_scan"
}
```

`422` is used on purpose: the request is well formed and authorized (that would be `400` and `403`), but its execution plan is not acceptable.

## Configuration

### TOML

```toml
[query_guard]
enabled = true                    # master switch, default false

reject_seq_scan = true            # refuse full table scans
reject_parallel_seq_scan = true   # refuse parallel sequential scans
max_cost = 50000                  # planner total cost ceiling (0 = no limit)
max_rows = 100000                 # estimated rows ceiling (0 = no limit)
require_index_usage = true        # plan must read through an index
max_joins = 3                     # join nodes ceiling (0 = no limit)
allow_tables = ["lookup", "countries"]
```

### Environment variables

| var | default | description |
| --- | ------- | ----------- |
| `PREST_QUERY_GUARD_ENABLED` | `false` | master switch |
| `PREST_QUERY_GUARD_REJECT_SEQ_SCAN` | `false` | refuse plans containing a sequential scan |
| `PREST_QUERY_GUARD_REJECT_PARALLEL_SEQ_SCAN` | `false` | refuse parallel sequential scans |
| `PREST_QUERY_GUARD_MAX_COST` | `0` | ceiling on the estimated total cost (`0` disables) |
| `PREST_QUERY_GUARD_MAX_ROWS` | `0` | ceiling on the estimated row count (`0` disables) |
| `PREST_QUERY_GUARD_REQUIRE_INDEX_USAGE` | `false` | plan must read at least one relation through an index |
| `PREST_QUERY_GUARD_MAX_JOINS` | `0` | ceiling on join nodes in the plan (`0` disables) |

A negative limit is treated as "no limit" instead of rejecting every query.

## Rules

| Rule | Refuses |
| ---- | ------- |
| `reject_seq_scan` | Any sequential scan, parallel ones included |
| `reject_parallel_seq_scan` | Parallel sequential scans specifically |
| `max_cost` | Plans whose root estimated cost exceeds the ceiling |
| `max_rows` | Plans whose root estimated row count exceeds the ceiling |
| `require_index_usage` | Plans that read no relation through an index |
| `max_joins` | Plans with more join nodes than allowed |

Scan rules are evaluated before cost rules, so the reported `reason` points at the actual cause.

### allow_tables

Relations listed in `allow_tables` are exempt from the **scan rules** (`reject_seq_scan`, `reject_parallel_seq_scan`, `require_index_usage`). Use it for small lookup tables where a full scan is genuinely cheaper than an index:

```toml
allow_tables = ["countries", "currencies", "status_codes"]
```

Cost and row ceilings still apply — they are properties of the whole query, not of one table.

## Per-database policies

With [multi-database](multi-database.md) deployments, each alias can relax or tighten the global policy. An override **starts from the global settings and replaces only the keys it declares**:

```toml
[query_guard]
enabled = true
reject_seq_scan = true
max_cost = 50000

# analytics keeps max_rows/require_index_usage from above,
# but allows sequential scans and a higher cost ceiling
[query_guard.databases.analytics]
reject_seq_scan = false
max_cost = 200000
```

Alias names are matched case-insensitively.

## What is checked

| Path | Guarded |
| ---- | ------- |
| `GET /{database}/{schema}/{table}` (table reads) | ✅ |
| MCP `select_table` tool ([MCP over HTTP](mcp-over-http.md)) | ✅ |
| `POST` / `PUT` / `PATCH` / `DELETE` on tables | ❌ — writes are not planned |
| `/auth` | ❌ |
| Catalog and listing endpoints (`/databases`, `/schemas`, `/tables`) | ❌ |
| [Custom queries](../api-reference/custom-queries.md) (`/_QUERIES`) | ❌ |

Only SQL generated from a client request is checked. Auth, catalog and custom-query SQL is written by _**prestd**_ or by you, not by API clients — checking it would, for example, break login on a `prest_users` table without an index.

## Requirements and startup behavior

Query Guard needs an adapter that can produce execution plans. PostgreSQL and its wire-compatible variants ([TimescaleDB](../databases/timescaledb.md), [Aurora](../databases/aurora-postgresql.md), [YugabyteDB](../databases/yugabytedb.md), [CockroachDB](../databases/cockroachdb.md), [Redshift](../databases/amazon-redshift.md)) qualify.

{% hint style="info" %}
If Query Guard is enabled and the adapter cannot plan queries, _**prestd**_ **fails to start**. This is deliberate: it is safer than serving an installation that believes it is protected. It is the one exception to [configuration resilience](configuring-prest.md#configuration-resilience).
{% endhint %}

When planning itself fails — for example the relation does not exist — the query is blocked and reported as `400`, not `422`: that is a database error, not a policy violation.

## Observability

Rejections are logged at `warn` level with the rule, the reason, the plan estimates and a compact plan shape:

```
level=WARN msg="query rejected by Query Guard" database=analytics rule=reject_seq_scan
  reason="Sequential Scan detected on table 'orders'." estimated_cost=18342.5
  estimated_rows=981230 plan="Seq Scan(orders)"
```

The SQL text is **not** logged: it carries user-supplied values. Set [`PREST_LOG_LEVEL`](configuring-prest.md#logging) to `warn` or lower to capture these.

## Tuning the policy

Start permissive and tighten:

1. Enable with `max_cost` only, set generously, and watch the rejection logs.
2. Lower `max_cost` until it matches what your database can actually absorb.
3. Add `reject_seq_scan` once you know which tables legitimately need full scans, and list those in `allow_tables`.
4. Add `require_index_usage` last — it is the strictest rule.

To find the cost of a query your clients run today, ask PostgreSQL directly:

```sql
EXPLAIN (FORMAT JSON) SELECT "id", "name" FROM "public"."orders" WHERE "status" = 'open';
```

The `Total Cost` of the top node is the value `max_cost` compares against.

## Related

- [Configuring pREST](configuring-prest.md)
- [Permissions](permissions.md)
- [Multi-database](multi-database.md)
- [MCP over HTTP](mcp-over-http.md)
- [Read-only PostgreSQL for AI](../ai/read-only-postgres.md)
- [Custom Queries](../api-reference/custom-queries.md)
