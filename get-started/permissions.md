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

## Ignore table

If you need to ignore restricted access mode for some tables, you can use the `ignore_table` option, it receives a string list with the names of the tables to be _"ignored"_, by **default,** is an empty list `[]`.

```toml
[access]
restrict = true
ignore_table = ["news"]
```

## Table permissions

Example:

```toml
[[access.tables]]
name = "test"
permissions = ["read", "write", "delete"]
fields = ["id", "name"]
```

When a [database registry](multi-database.md) is active, use optional `database` and `schema` fields to scope permissions to a specific alias:

```toml
[[access.tables]]
database = "tenant-a"
schema = "public"
name = "users"
permissions = ["read"]
fields = ["id", "name"]
```

Permissions are matched against **alias + schema + table name** when the registry is configured. When `database` is omitted, the rule applies to all aliases (legacy behavior).

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

Permissions are matched against alias + schema + table name.

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
table = "prest_users"
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

## Related

- [Multi-database](multi-database.md)
- [MCP over HTTP](mcp-over-http.md)
- [Auth](../api-reference/auth.md)
- [Acronyms](../prestd/acronyms.md) · [ACL](../prestd/acronyms.md#acl) · [MCP](../prestd/acronyms.md#mcp)
