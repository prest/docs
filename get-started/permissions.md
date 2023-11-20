# Permissions

#### Restrict mode

In the[ prest.toml](#user-content-fn-1)[^1] you can configure read/write/delete permissions of each table.

```toml
[access]
restrict = true  # can access only the tables listed below
```

`restrict = false`: (default) The pREST will serve in public mode. You can write/read/delete every data without configuring permissions.

`restrict = true`: you need to configure the permissions of all tables.

#### Ignore table

If you need to ignore restricted access mode for some tables you can use the `ignore_table` option, it receives a string list with the names of the tables to be _"ignored"_, by **default,** is an empty list `[]`.

```toml
[access]
restrict = true
ignore_table = ["news"]
```

#### Table permissions

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
| permissions | Table permissions. Options: `read`, `write` and `delete` |
| fields      | Fields permitted for operations                          |

## Example

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

[^1]: 
