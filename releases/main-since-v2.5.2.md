---
description: >-
  Changes merged after v2.5.2 and released in v2.5.3 — three security fixes
  (unfiltered writes, write fields, read-column ACL) and dependency updates.
---

# Changes since v2.5.2 (in v2.5.3)

**Released as [v2.5.3](v2.5.3.md).** The commits below were merged after [v2.5.2](v2.5.2.md) and ship in that tag. Compare: [v2.5.2...v2.5.3](https://github.com/prest/prest/compare/v2.5.2...v2.5.3).

| Commit | PR | Summary |
|--------|----|---------|
| `0e2e64b` | — | Refuse `DELETE`/`PUT`/`PATCH` without a filter; `access.allow_unscoped_writes` ([GHSA-r867-ghgv-mrp9](https://github.com/prest/prest/security/advisories/GHSA-r867-ghgv-mrp9)) |
| `9804dd7` | [#1072](https://github.com/prest/prest/pull/1072) | Write-field allowlist; write responses show only readable columns ([GHSA-h25q-9rmj-646h](https://github.com/prest/prest/security/advisories/GHSA-h25q-9rmj-646h)) |
| `6919f25` | [#1083](https://github.com/prest/prest/pull/1083) | Field ACL on `_count` and every column a read references ([GHSA-48r9-g6f9-x3gm](https://github.com/prest/prest/security/advisories/GHSA-48r9-g6f9-x3gm)) |
| `dc80f1e` | [#1074](https://github.com/prest/prest/pull/1074) | `github.com/go-jose/go-jose/v4` 4.1.6 |
| `3cc42b0` | [#1077](https://github.com/prest/prest/pull/1077) | Studio: `@tanstack/react-table` 9.3.1 |
| `777b717` | [#1078](https://github.com/prest/prest/pull/1078) | Studio: `react-hook-form` 7.90.0 |

Details: [v2.5.3 release notes](v2.5.3.md).

---

## Related

- [v2.5.3 release notes](v2.5.3.md)
- [v2.5.2 release notes](v2.5.2.md)
- [Releases](README.md)
