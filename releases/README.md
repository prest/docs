# Releases

**Latest v2 release:** [v2.1.0](v2.1.0.md) — native MCP over HTTP on `/_mcp`, with schema-aware read-only tools and auth/ACL integration ([#977](https://github.com/prest/prest/pull/977)).

**On `main` (unreleased):** [Changes since v2.1.0](main-since-v2.1.0.md) — database-backed custom queries ([#980](https://github.com/prest/prest/pull/980)), TimescaleDB E2E ([#988](https://github.com/prest/prest/pull/988)).

For stable v1 releases, see [GitHub Releases](https://github.com/prest/prest/releases/latest).

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
