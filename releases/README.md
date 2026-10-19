# Releases

**Latest v2 release:** [v2.5.2](v2.5.2.md) — security release ([#1070](https://github.com/prest/prest/pull/1070)): stored-query SQL execution, unauthenticated catalog, and plugin path traversal fixed; `pg.single` replaced by `pg.allowed_databases` ([#1068](https://github.com/prest/prest/pull/1068)).

For stable v1 releases, see [GitHub Releases](https://github.com/prest/prest/releases/latest).

## Unreleased (main)

Nothing merged after v2.5.2 yet. See [Changes since v2.5.2](main-since-v2.5.2.md).

## v2.5.2 highlights

| Area | Change |
|------|--------|
| Security | Reserved control-plane tables (`prest_queries`, `prest_users`) blocked on REST, `_join`, and MCP — critical, CVSS 9.0 ([GHSA-2x9q-r7j9-g85f](https://github.com/prest/prest/security/advisories/GHSA-2x9q-r7j9-g85f)) |
| Security | Catalog routes and `/show` behind auth and table ACL — high, CVSS 7.5 ([GHSA-fhf5-m388-4w4p](https://github.com/prest/prest/security/advisories/GHSA-fhf5-m388-4w4p)) |
| Security | `/_PLUGIN` name confinement and auth — high, CVSS 8.1 ([GHSA-fmr6-9j48-xrqp](https://github.com/prest/prest/security/advisories/GHSA-fmr6-9j48-xrqp)) |
| Multi-database | `pg.single` removed; `pg.allowed_databases`; unreachable aliases answer 503 instead of stopping startup ([#1068](https://github.com/prest/prest/pull/1068)) |

See [v2.5.2 release notes](v2.5.2.md) and [Changes since v2.5.1](main-since-v2.5.1.md). **Upgrade from v2.5.1 or earlier now**, and audit `prest_queries` if you used database-backed queries without auth.

## v2.5.1 highlights

| Area | Change |
|------|--------|
| MCP | Read-only `/_mcp` on the MySQL adapter ([#1056](https://github.com/prest/prest/pull/1056)) |
| Registry | A `[[databases]]` alias serves only its own database as `{schema}` |
| JSON | Integers above 2^53 are written exactly; large MySQL integers scan as JSON numbers |
| Errors | Connection failure is **503** `database unavailable` with no driver text |
| Images | Docker tags are linux/amd64 and linux/arm64 |

See [v2.5.1 release notes](v2.5.1.md) and [MySQL](../databases/mysql.md).

## v2.5.0 highlights

| Area | Change |
|------|--------|
| MySQL | Native dialect adapter for MySQL 8.0.19+ when `engine = "mysql"` ([#1048](https://github.com/prest/prest/pull/1048)) |
| JOIN ACL | Restricted joins return only permitted columns; bad joins are rejected ([#1032](https://github.com/prest/prest/pull/1032), [#1033](https://github.com/prest/prest/pull/1033)) |
| Plugins | Middleware `.so` builds share the server module; missing plugins no-op ([#1039](https://github.com/prest/prest/pull/1039)) |
| Dependencies | OpenTelemetry 1.47, Studio table library 8 → 9 ([#1055](https://github.com/prest/prest/pull/1055)) |

See [v2.5.0 release notes](v2.5.0.md) and [MySQL](../databases/mysql.md). MariaDB is not supported.

## v2.4.2 highlights

| Area | Change |
|------|--------|
| JWT | HMAC `jwt.key` minimums (32/48/64 bytes); short keys are discarded and **auth disables itself** ([#1017](https://github.com/prest/prest/pull/1017)) |
| JWT | `jwt.algo` enforced as the sole permitted signature algorithm ([#1017](https://github.com/prest/prest/pull/1017)) |
| Custom queries | Bind values with `sqlVal` / `sqlList` / `ident`; rejected interpolated values now fail with `400` ([#1023](https://github.com/prest/prest/pull/1023)) |
| Custom queries | Credential headers withheld from templates; script path traversal rejected ([#1023](https://github.com/prest/prest/pull/1023)) |
| Logging | Script SQL no longer logged; CRUD parameter values replaced by a count ([#1023](https://github.com/prest/prest/pull/1023)) |

See [v2.4.2 release notes](v2.4.2.md). **Check `jwt.key` length before upgrading** — a short key leaves the API unauthenticated rather than refusing to start.

## v2.4.1 highlights

| Area | Change |
|------|--------|
| MCP | `/_mcp` honours `[expose]`, closing a catalog-discovery bypass ([#1016](https://github.com/prest/prest/pull/1016)) |
| Custom queries | SQL-keyword screen on interpolated script values ([#1016](https://github.com/prest/prest/pull/1016)) — too broad, relaxed in [v2.4.2](v2.4.2.md) |

See [v2.4.1 release notes](v2.4.1.md) and [Changes since v2.4.0](main-since-v2.4.0.md). Upgrade past v2.4.1 to v2.4.2.

## v2.4.0 highlights

| Area | Change |
|------|--------|
| Observability | Opt-in OpenTelemetry traces/metrics/logs, local SigNoz dev stack ([#1003](https://github.com/prest/prest/pull/1003)) |
| Vector search | pgvector KNN ordering (`_korder`) and distance filtering (`:vecdist`) ([#1011](https://github.com/prest/prest/pull/1011)) |
| pREST Studio | Dependency upgrade — auth-dialog and tool-invocation fixes ([#1004](https://github.com/prest/prest/pull/1004)) |

See [v2.4.0 release notes](v2.4.0.md) and [Changes since v2.3.0](main-since-v2.3.0.md).

{% hint style="info" %}
`_korder` / `:vecdist` and the `[otel]` section landed in v2.4.0 and are unchanged in v2.4.1 and v2.4.2.
{% endhint %}

## v2.3.0 highlights

| Area | Change |
|------|--------|
| Security | Unauthenticated `_select` SQL-injection fixed — CVSS 9.8 ([GHSA-qvx3-q8vx-9q3c](https://github.com/prest/prest/security/advisories/GHSA-qvx3-q8vx-9q3c), [#1002](https://github.com/prest/prest/pull/1002)) |
| Multi-adapter | Adapter registry, automatic Postgres/TimescaleDB detection, path-based routing ([#999](https://github.com/prest/prest/pull/999)) |
| JWKS hardening | `jwx/v3` — non-2xx rejection, 1 MiB body cap, URL redaction in logs ([#1002](https://github.com/prest/prest/pull/1002)) |

See [v2.3.0 release notes](v2.3.0.md). Upgrade from v2.2.0 as soon as possible for the security fix.

## v2.2.0 highlights

| Area | Change |
|------|--------|
| pREST Studio | Embedded UI at `/_studio/` — Data / REST / MCP explorers ([#990](https://github.com/prest/prest/pull/990)) |
| Custom queries | Optional database storage, registry API, query ACL ([#980](https://github.com/prest/prest/pull/980)) |
| TimescaleDB | E2E certification on the native PostgreSQL adapter ([#988](https://github.com/prest/prest/pull/988)) |
| Config sample | Fully documented [`prest.sample.toml`](https://github.com/prest/prest/blob/v2.2.0/samples/prest.sample.toml) ([#978](https://github.com/prest/prest/pull/978)) |

See [v2.2.0 release notes](v2.2.0.md), [Changes since v2.1.0](main-since-v2.1.0.md), and [pREST Studio](../get-started/prest-studio.md).

## v2.1.0 highlights

| Area | Change |
|------|--------|
| MCP over HTTP | Read-only `/_mcp` endpoint with JSON-RPC `initialize`, `tools/list`, and `tools/call` ([#977](https://github.com/prest/prest/pull/977)) |
| Schema-aware tools | Per-table `prest.select.{database}.{schema}.{table}` tools with typed input schemas from catalog metadata |
| Safety | Read-only by design; inherits auth, ACL, and identifier validation from the existing HTTP stack |

See [v2.1.0 release notes](v2.1.0.md), the [MCP over HTTP guide](../get-started/mcp-over-http.md), and [AI and MCP](../ai/README.md) (Cursor / Claude Desktop / adapter install) for usage and upgrade notes.

## v2.0.0 highlights

| Area | Change |
|------|--------|
| Multi-database | `[[databases]]` registry, alias routing, `/_ready` ([#973](https://github.com/prest/prest/pull/973)) |
| Config resilience | Graceful fallbacks; startup never blocked by bad config ([#974](https://github.com/prest/prest/pull/974)) |
| JWT | Auto-disable when misconfigured; `jwt.default` defaults to `false` ([#974](https://github.com/prest/prest/pull/974)) |
| Security | Template sanitization, credential redaction in logs ([#972](https://github.com/prest/prest/pull/972)) |
| OR filtering | `_or` query parameter ([#958](https://github.com/prest/prest/pull/958)) |
| Permissions | Per-user table permissions ([#912](https://github.com/prest/prest/pull/912)) |

See [v2.0.0 release notes](v2.0.0.md) and [Changes since rc6](main-since-rc6.md) for full details.

## Release candidate history (rc1 – rc6)

The v2 release candidates shipped the following before v2.0.0 was tagged.

### Features

| Version | Change |
| ------- | ------ |
| rc1 | Per-user table permissions via `[[access.users]]` ([#912](https://github.com/prest/prest/pull/912)) |
| rc6 | OR clause filtering with the `_or` query parameter ([#958](https://github.com/prest/prest/pull/958)) |
| rc6 | Structured JSON logging via Go `slog` ([#950](https://github.com/prest/prest/pull/950)) |
| rc6 | Docker images built with GoReleaser ([#953](https://github.com/prest/prest/pull/953)) |

### Security

| Version | Change |
| ------- | ------ |
| rc3 | `_returning` parameter hardened against SQL injection ([#935](https://github.com/prest/prest/pull/935)) |
| rc4 | Unified identifier validation across templates, groupby, and path params ([#938](https://github.com/prest/prest/pull/938), [GHSA-p46v-f2x8-qp98](https://github.com/prest/prest/security/advisories/GHSA-p46v-f2x8-qp98)) |
| rc5 | `tsquery` operator hardened against SQL injection ([#940](https://github.com/prest/prest/pull/940)) |
| rc6 | JWT auth bypass fixed when default enforcement runs without a key ([#960](https://github.com/prest/prest/pull/960), [GHSA-fj7v-859r-2fm4](https://github.com/prest/prest/security/advisories/GHSA-fj7v-859r-2fm4)) |

### Config and breaking changes

| Version | Change |
| ------- | ------ |
| rc2 | Deprecated `PREST_SSL_*` environment variables and `[ssl]` TOML block removed — use `PREST_PG_SSL_*` / `[pg.ssl]` ([#919](https://github.com/prest/prest/pull/919)) |
| rc2 | Default `pg.ssl.mode` is `disable` when no config file is found (v1 used `require`) |
| rc6 | Server refuses to start when JWT is enabled without verification material (unless debug mode is on)[^jwt-v200] |

[^jwt-v200]: **Superseded in v2.0.0** ([#974](https://github.com/prest/prest/pull/974)): missing JWT verification material now auto-disables JWT/auth with a warning instead of aborting startup. See [v2.0.0 release notes](v2.0.0.md).

### Fixes

| Version | Change |
| ------- | ------ |
| rc2 | Default cache storage path set when caching is disabled ([#918](https://github.com/prest/prest/pull/918)) |
| rc6 | `_select` field names are whitespace-trimmed after comma splitting ([#941](https://github.com/prest/prest/pull/941)) |
| rc6 | Identifier formatting fix ([#955](https://github.com/prest/prest/pull/955)) |

## Upgrading

If you are moving from v1 to v2, see the [Upgrading to v2](../get-started/upgrading-to-v2.md) guide.
