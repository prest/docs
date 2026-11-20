---
description: >-
  Changes merged after v2.5.3 and released in v2.6.0 — request IDs,
  pg.application_name, MCP protocolVersion, JSON config, allowed hosts and more.
---

# Changes since v2.5.3 (in v2.6.0)

**Released as [v2.6.0](v2.6.0.md).** The commits below were merged after [v2.5.3](v2.5.3.md) and ship in that tag. Compare: [v2.5.3...v2.6.0](https://github.com/prest/prest/compare/v2.5.3...v2.6.0).

| Commit | PR | Summary |
|--------|----|---------|
| `5597ba9` | [#1093](https://github.com/prest/prest/pull/1093) | Only a leading `$op` is read as a filter operator |
| `d205843` | [#1094](https://github.com/prest/prest/pull/1094) | `$contains` filter for array columns — [Array containment](../api-reference/parameters.md#array-containment) |
| `bbb1959` | [#1096](https://github.com/prest/prest/pull/1096) | Dependency updates |
| `24e6210` | [#1097](https://github.com/prest/prest/pull/1097) | `auth.token_ttl` sets the lifetime of `/auth` tokens — [Auth](../get-started/configuring-prest.md#auth) |
| `d105f35` | [#1103](https://github.com/prest/prest/pull/1103) | `UPDATE`/`DELETE ... RETURNING` errors are returned instead of panicking |
| `b5d8373` | [#1104](https://github.com/prest/prest/pull/1104) | `rows` and `columns` renderers — [Rows and columns](../api-reference/parameters.md#rows-and-columns) |
| `9b320aa` | [#1102](https://github.com/prest/prest/pull/1102) | `prestd auth create-user` command — [Creating users](../api-reference/auth.md#creating-users-prestd-auth-create-user) |
| `5a8ee66` | [#1105](https://github.com/prest/prest/pull/1105) | `msgpack` renderer — [MessagePack](../api-reference/parameters.md#messagepack) |
| `e8031fb` | [#1106](https://github.com/prest/prest/pull/1106) | `[pg] timezone` session time zone — [Session time zone](../get-started/configuring-prest.md#session-time-zone) |
| `1c86994` | [#1107](https://github.com/prest/prest/pull/1107) | `http.allowed_hosts` rejects requests for other `Host` headers with 400 — [Allowed hosts](../get-started/configuring-prest.md#allowed-hosts) |
| `9a2d4a0` | [#1100](https://github.com/prest/prest/pull/1100) | Application config from `PREST_CONFIG_JSON`, and `prestd config export-json` — [Configuration as JSON](../get-started/configuring-prest.md#configuration-as-json) |
| `b37aaec` | [#1114](https://github.com/prest/prest/pull/1114) | `http.request_id_header` request IDs on responses and access logs — [Request IDs](../get-started/configuring-prest.md#request-ids) |
| `b90aa41` | [#1118](https://github.com/prest/prest/pull/1118) | Response body write errors are logged instead of discarded |
| `70ba8d0` | [#1117](https://github.com/prest/prest/pull/1117) | MCP `initialize` negotiates `protocolVersion` and accepts unknown request fields — [Protocol version](../get-started/mcp-over-http.md#protocol-version) |
| `749ccb5` | [#1091](https://github.com/prest/prest/pull/1091) | `pg.application_name` names pooled Postgres connections — [Application name](../get-started/configuring-prest.md#application-name) |
| `72d59b3` | [#1119](https://github.com/prest/prest/pull/1119) | Go modules upgraded, `go.mod` declares Go 1.27, and `make deps-upgrade` — [Upgrading dependencies](../get-prest/development-guide.md#upgrading-dependencies) |
| `508d79e` | [#1120](https://github.com/prest/prest/pull/1120) | Studio dependency updates |

Details: [v2.6.0 release notes](v2.6.0.md).

---

## Related

- [v2.6.0 release notes](v2.6.0.md)
- [v2.5.3 release notes](v2.5.3.md)
- [Changes since v2.6.0 (unreleased)](main-since-v2.6.0.md)
- [Releases](README.md)
