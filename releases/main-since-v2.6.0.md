---
description: >-
  Changes merged to prest main after v2.6.0 — not in a tagged release yet.
  Updated as commits land.
---

# Changes since v2.6.0 (unreleased)

[v2.6.0](v2.6.0.md) is released and is the current stable tag. This page tracks work merged to `main` **after** it, ahead of the next release. None of it is in `prest/prest:v2.6.0`. Compare: [v2.6.0...main](https://github.com/prest/prest/compare/v2.6.0...main).

| Commit | PR | Summary |
|--------|----|---------|
| `6ffd49e` | [#1122](https://github.com/prest/prest/pull/1122) | Studio fixes found by a new end-to-end test harness, such as the Copy URL, MCP form arguments, large-number precision, and sequences listed as tables |
| `0039e5b` | [#1113](https://github.com/prest/prest/pull/1113) | `csv` renderer — [CSV](../api-reference/parameters.md#csv) |
| `621edc5` | [#1044](https://github.com/prest/prest/pull/1044) | Opt-in `[guard]` request security middleware: per-client rate limits, payload inspection, IP allow and block lists, auto-ban, cloud range blocking and shared Redis state — [Request security](../get-started/request-security.md) |

---

## Related

- [v2.6.0 release notes](v2.6.0.md)
- [Changes since v2.5.3 (in v2.6.0)](main-since-v2.5.3.md)
- [Releases](README.md)
