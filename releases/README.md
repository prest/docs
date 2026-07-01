# Releases

pREST v2 is currently in **release candidate** status. The latest RC is [v2.0.0-rc6](v2.0.0-rc6.md). Release candidates are intended for testing and feedback — they are not recommended for production workloads yet.

For stable v1 releases, see [GitHub Releases](https://github.com/prest/prest/releases/latest).

## v2 Release Candidate changelog (rc1 – rc6)

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
| rc6 | Server refuses to start when JWT is enabled without verification material (unless debug mode is on) |

### Fixes

| Version | Change |
| ------- | ------ |
| rc2 | Default cache storage path set when caching is disabled ([#918](https://github.com/prest/prest/pull/918)) |
| rc6 | `_select` field names are whitespace-trimmed after comma splitting ([#941](https://github.com/prest/prest/pull/941)) |
| rc6 | Identifier formatting fix ([#955](https://github.com/prest/prest/pull/955)) |

## Upgrading

If you are moving from v1 to v2, see the [Upgrading to v2](../get-started/upgrading-to-v2.md) guide.
