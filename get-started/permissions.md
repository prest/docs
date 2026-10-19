---
description: How to manage tables read/writes
---

# Permissions

This guide will allow you to understand how `pREST` to understand and manage your table's permission management and how you can tailor it to your needs by using the  `prest.toml` file.&#x20;

## Restrict mode

The `prest.toml` file allows you to configure each table's read/write/delete permissions.

```toml
[access]
restrict = true  # can access only the tables listed below
```

`restrict = false`: (default) The pREST will serve in public mode. You can write/read/delete every data without configuring permissions.

`restrict = true`: you need to configure the permissions of all tables.

The same rules apply to `/show/{database}/{schema}/{table}` and the catalog routes from v2.5.2: with `restrict = true`, a table not granted `read` is **401** there as it is on CRUD ([GHSA-fhf5-m388-4w4p](https://github.com/prest/prest/security/advisories/GHSA-fhf5-m388-4w4p)).

## Ignore table

If you need to ignore restricted access mode for some tables, you can use the `ignore_table` option, it receives a string list with the names of the tables to be _"ignored"_, by **default,** is an empty list `[]`.

```toml
[access]
restrict = true
ignore_table = ["news"]
```

## Reserved tables (v2.5.2)

pREST's own control-plane tables are never reachable through data routes, whatever `restrict`, `ignore_table`, or per-user rules say — and even when auth or database-backed queries are turned off ([GHSA-2x9q-r7j9-g85f](https://github.com/prest/prest/security/advisories/GHSA-2x9q-r7j9-g85f)). The names are matched case-insensitively, in any schema:

- `prest_queries` — stored [custom queries](../api-reference/custom-queries.md#database-backed-storage-v220)
- `prest_users` — [auth](../api-reference/auth.md) users
- the configured `queries.table`
- the configured `auth.table` (`PREST_AUTH_TABLE`)

| Request | Response |
|---------|----------|
| Any method on `/{database}/{schema}/{reserved}`, `/batch/...`, or `/show/...` | **401** `{"error":"authorization required"}`, even with an admin token |
| `_join` that targets a reserved table | **400** `{"error":"table is reserved for pREST internal use"}` |
| MCP `select_table` / `describe_table` | JSON-RPC permission error |

Manage stored queries through the [`/_QUERIES/registry` API](../api-reference/custom-queries.md#registry-api) and users with SQL or `prestd migrate`. There is no opt-out: if an application table has one of these names, rename it.

## Table permissions

Example:

```toml
[[access.tables]]
name = "test"
permissions = ["read", "write", "delete"]
fields = ["id", "name"]
```

Multiple configurations for the same table:

```toml
[access]
restrict = true  # can access only the tables listed below

[[access.tables]]
name = "test"
permissions = ["read"]
fields = ["id", "name"]
[[access.tables]]
name = "test"
permissions = ["write"]
fields = ["name"]
```

| attribute   | description                                              |
| ----------- | -------------------------------------------------------- |
| name        | Table name                                               |
| database    | Optional. Database alias when using a registry ([#973](https://github.com/prest/prest/pull/973)) |
| schema      | Optional. Schema name (default matching applies when omitted) |
| permissions | Table permissions. Options: `read`, `write` and `delete` |
| fields      | Exposed fields permitted for operations                  |

### Per-database permissions

When a [database registry](multi-database.md) is active, scope permissions to a specific alias and schema:

```toml
[[access.tables]]
database = "tenant-a"
schema = "public"
name = "users"
permissions = ["read"]
fields = ["id", "name"]
```

Permissions are matched against **alias + schema + table name** when the registry is configured. When `database` is omitted, the rule applies to all aliases.

## JOINs and restrict (v2.5.0)

From v2.5.0 ([#1032](https://github.com/prest/prest/pull/1032), [#1033](https://github.com/prest/prest/pull/1033), fixes [#364](https://github.com/prest/prest/issues/364)), a [`_join`](../api-reference/advanced-queries.md#join) is checked table by table. With `restrict = true`:

- **The queried table** must be readable, or the request is **401** `{"error":"authorization required"}`.
- **Each joined table** goes through the same `[[access.tables]]` and `[[access.users.tables]]` rules for the operation. A readable table adds its permitted `fields`, qualified as `table.column`, or `table.*` when its fields are `["*"]` or it is listed in `ignore_table`.
- **A joined table you cannot read adds no columns, but the JOIN still runs** — it still filters which rows come back.
- **Without `_select`**, the response carries the queried table's permitted fields plus every permitted joined table's fields.
- **With `_select`**, a bare name means the queried table; a joined column must be written `table.column` (or `table.*`). Entries you are not permitted to read are dropped silently. If none remain, the request is **400** `you don't have permission for this action, please check the permitted fields for this table`.

With `restrict = false`, `_select` passes through unchanged. In both modes, joining the queried table itself or the same table twice is **400**, and joining a [reserved table](#reserved-tables-v252) is **400**.

Example:

```toml
[access]
restrict = true

[[access.tables]]
name = "friends"
permissions = ["read"]
fields = ["id", "userid"]

[[access.tables]]
name = "users"
permissions = ["read"]
fields = ["id", "name"]

# no entry for "salaries"
```

```
GET /prest/public/friends?_join=inner:users:friends.userid:$eq:users.id&_join=inner:salaries:users.id:$eq:salaries.user_id
```

Returns `friends.id`, `friends.userid`, `users.id`, and `users.name` — only for friends whose user has a `salaries` row. No `salaries` column is returned.

## User-level permissions

v2 supports per-user table permissions via `[[access.users]]`. The authenticated user's identity (from the JWT `sub` claim or username) is matched against `access.users.name`.

User-level table permissions restrict or extend the global `access.tables` rules for that specific user.

Example:

```toml
[access]
restrict = true

[[access.tables]]
name = "read_table"
permissions = ["read"]
fields = ["id", "name", "age", "gender"]

[[access.users]]
name = "foo_read"
[[access.users.tables]]
name = "read_table"
permissions = ["read"]
fields = ["id", "name"]
```

In this example, user `foo_read` can only read the `id` and `name` fields on `read_table`, even if the global table permission allows more fields.

For a comprehensive example with multiple users and permission combinations, see [testdata/prest.toml](https://github.com/prest/prest/blob/main/testdata/prest.toml).

MCP (`/_mcp`, v2.1.0+) inherits the same ACL rules as REST table routes — read tools only see tables and fields your access config allows. See [MCP over HTTP](mcp-over-http.md).

## Example configuration

Configuration example: [prest.toml](https://github.com/prest/prest/blob/main/testdata/prest.toml)

```toml
[auth]
table = "prest_users"  # reserved: not reachable through REST, _join, or MCP (v2.5.2)
username = "username"
password = "password"
metadata = ["first_name", "last_name", "last_login"]

[http]
port = 3000

[cache]
enabled = true

    [[cache.endpoints]]
    endpoint = "/prest/public/test"
    time = 5

[access]
restrict = true  # can access only the tables listed below

    [[access.tables]]
    name = "Reply"
    permissions = ["read", "write", "delete"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "test"
    permissions = ["read", "write", "delete"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "testarray"
    permissions = ["read", "write", "delete"]
    fields = ["id", "data"]

    [[access.tables]]
    name = "test2"
    permissions = ["read", "write", "delete"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "test3"
    permissions = ["read", "write", "delete"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "test4"
    permissions = ["read", "write", "delete"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "test5"
    permissions = ["read", "write", "delete"]
    fields = ["*"]

    [[access.tables]]
    name = "test_readonly_access"
    permissions = ["read"]
    fields = ["id", "name"]

    [[access.tables]]
    name = "test_write_and_delete_access"
    permissions = ["write", "delete"]

    [[access.tables]]
    name = "test_list_only_id"
    permissions = ["read"]
    fields = ["id"]

    [[access.tables]]
    name = "test6"
    permissions = ["read", "write", "delete"]
    fields = ["nuveo", "name"]

    [[access.tables]]
    name = "view_test"
    permissions = ["read"]
    fields = ["player"]

    [[access.tables]]
    name = "test_group_by_table"
    permissions = ["read"]
    fields = ["id", "name", "age", "salary"]
```

## Responses

| Situation | Response |
|-----------|----------|
| Table not granted for the operation (`restrict = true`) | **401** `{"error":"authorization required"}` |
| Reserved table on CRUD, `/batch`, or `/show` (v2.5.2) | **401** `{"error":"authorization required"}` |
| `_join` to a reserved table (v2.5.2) | **400** `{"error":"table is reserved for pREST internal use"}` |
| `_select` with no permitted column left | **400** `you don't have permission for this action, please check the permitted fields for this table` |
| Same table joined twice, or joined to itself | **400** `table "<name>" is joined more than once: invalid join clause` |

## Related

- [Advanced queries — JOIN](../api-reference/advanced-queries.md#join)
- [v2.5.2 release notes](../releases/v2.5.2.md)
- [Multi-database](multi-database.md)
- [MCP over HTTP](mcp-over-http.md)
- [Auth](../api-reference/auth.md)
- [Acronyms](../prestd/acronyms.md) · [ACL](../prestd/acronyms.md#acl) · [MCP](../prestd/acronyms.md#mcp)
