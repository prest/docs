---
description: >-
  Changes merged to prest main after v2.5.3 — not in a tagged release yet.
  Updated as commits land.
---

# Changes since v2.5.3 (unreleased)

[v2.5.3](v2.5.3.md) is released and is the current stable tag. This page tracks work merged to `main` **after** it, ahead of the next release. None of it is in `prest/prest:v2.5.3`. Compare: [v2.5.3...main](https://github.com/prest/prest/compare/v2.5.3...main).

| Commit | PR | Summary |
|--------|----|---------|
| `5597ba9` | [#1093](https://github.com/prest/prest/pull/1093) | Only a leading `$op` is read as a filter operator |
| `d205843` | [#1094](https://github.com/prest/prest/pull/1094) | `$contains` filter for array columns — [Array containment](../api-reference/parameters.md#array-containment) |
| `bbb1959` | [#1096](https://github.com/prest/prest/pull/1096) | Dependency updates |
| `24e6210` | [#1097](https://github.com/prest/prest/pull/1097) | `auth.token_ttl` sets the lifetime of `/auth` tokens — [Auth](../get-started/configuring-prest.md#auth) |
| `d105f35` | [#1103](https://github.com/prest/prest/pull/1103) | `UPDATE`/`DELETE ... RETURNING` errors are returned instead of panicking |
| `b5d8373` | [#1104](https://github.com/prest/prest/pull/1104) | `rows` and `columns` renderers — [Rows and columns](../api-reference/parameters.md#rows-and-columns) |
| `9b320aa` | [#1102](https://github.com/prest/prest/pull/1102) | `prestd auth create-user` command |
| `5a8ee66` | [#1105](https://github.com/prest/prest/pull/1105) | `msgpack` renderer |
| `e8031fb` | [#1106](https://github.com/prest/prest/pull/1106) | `[pg] timezone` session time zone |

---

## Related

- [v2.5.3 release notes](v2.5.3.md)
- [Changes since v2.5.2 (in v2.5.3)](main-since-v2.5.2.md)
- [Releases](README.md)
