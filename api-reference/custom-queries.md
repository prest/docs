# Custom Queries

If you need advanced SQL, write script templates and call them over REST. Scripts accept values from the URL (and headers) and run with the same auth stack as other routes.

_**awesome\_folder/example\_of\_powerful.read.sql**_**:**

```sql
SELECT * FROM table WHERE name = "{{.field1}}" OR name = "{{.field2}}"
```

**Get result:**

```
GET /_QUERIES/awesome_folder/example_of_powerful?field1=foo&field2=bar
```

With a [database registry](../get-started/multi-database.md), prefix the database alias:

```
GET /_QUERIES/tenant-a/awesome_folder/example_of_powerful?field1=foo&field2=bar
```

When the database prefix is omitted, the default database (`pg.database`) is used.

> **Security (v2.0.0+):** template query parameters are sanitized ([#972](https://github.com/prest/prest/pull/972)). Pass well-formed values; unsafe inputs are rejected.

**To activate filesystem scripts**, set a location in `prest.toml`:

```toml
[queries]
location = "/path/to/queries/"
```

Default storage is the filesystem (`queries.storage = "filesystem"`). Database-backed storage is available on unreleased `main` — see [below](#database-backed-storage-main).

### Scripts templates rules

In your scripts, the fields to replace have to look like: _field1 or field2 are examples._

```sql
SELECT * FROM table WHERE name = "{{.field1}}" OR name = "{{.field2}}"
```

Script file must have a suffix based on http verb:

| HTTP Verb  | Suffix      |
| ---------- | ----------- |
| GET        | .read.sql   |
| POST       | .write.sql  |
| PUT, PATCH | .update.sql |
| DELETE     | .delete.sql |

In `queries.location` you need to have a folder for your scripts:

```shell
queries/
└── foo
    └── some_get.read.sql
    └── some_create.write.sql
    └── some_update.update.sql
    └── some_delete.delete.sql
└── bar
    └── some_get.read.sql
    └── some_create.write.sql
    └── some_update.update.sql
    └── some_delete.delete.sql

URLs to foo folder:

GET    /_QUERIES/foo/some_get?field1=bar
POST   /_QUERIES/foo/some_create?field1=bar
PUT    /_QUERIES/foo/some_update?field1=bar
PATCH  /_QUERIES/foo/some_update?field1=bar
DELETE /_QUERIES/foo/some_delete?field1=bar


URLs to bar folder:

GET    /_QUERIES/bar/some_get?field1=foo
POST   /_QUERIES/bar/some_create?field1=foo
PUT    /_QUERIES/bar/some_update?field1=foo
PATCH  /_QUERIES/bar/some_update?field1=foo
DELETE /_QUERIES/bar/some_delete?field1=foo
```

### Template data

You can access the query parameters of the incoming HTTP request using the `.` notation.

For instance, the following request:

```
GET    /_QUERIES/bar/some_get?field1=foo&field2=bar
```

Makes available the fields `field1` and `field2` in the script:

```sql
{{.field1}}
{{.field2}}
```

You can also access the query headers of the incoming HTTP requests using the `.header` notation.

For instance, the following request:

```
GET    /_QUERIES/bar/some_get
X-UserId: am9obi5kb2VAYW5vbnltb3VzLmNvbQ
X-Application: prest
```

makes available the headers `X-UserId` and `X-Application` in the script:

```
{{index .header "X-UserId"}}
{{index .header "X-Application"}}
```

### Template functions

#### isSet

Return true if the param is set.

```sql
SELECT * FROM table
{{if isSet "field1"}}
WHERE name = "{{.field1}}"
{{end}}
```

#### defaultOrValue

Return param value or default value.

```sql
SELECT * FROM table WHERE name = '{{defaultOrValue "field1" "gopher"}}'
```

#### inFormat

If you need to format data for usage on a `IN ('option1', 'option2')` statement. You can use this with the field inside the format. Whenever passing multiple arguments to `inFormat` function, you must use multiple `field1` instances on the URL.

For example: I want to query the field `name` with options `Mary` and `John`.

```sql
-- URL will be equal to /_QUERIES/custom_query/query?name=Mary&name=John

SELECT * FROM names WHERE name IN {{inFormat "name"}}
```

**Future updates** will include the support of multiple strings split by `,` on the same instance of the field.

#### split

Splits a string into substrings separated by a delimiter

```sql
SELECT * FROM table WHERE
name IN ({{ range $index,$part := split 'test1,test2,test3' `,` }}{{if gt $index 0 }},{{end}}'{{$part}}'{{ end }})
```

#### limitOffset

Assemble `limit offset()` string with validation for non-allowed characters _parameters must be integer values_

```sql
SELECT * FROM table {{limitOffset "1" "10"}}
```

**generating the query:**

```sql
SELECT * FROM table LIMIT 10 OFFSET(1 - 1) * 10
```

_We recommend using the default pREST variables `_page` and `_page_size`:_

```sql
{{limitOffset ._page ._page_size}}
```

### Database-backed storage (main)

{% hint style="warning" %}
**Requires prest `main` (unreleased)** — [#980](https://github.com/prest/prest/pull/980). Not in [v2.1.0](../releases/v2.1.0.md). See [Changes since v2.1.0](../releases/main-since-v2.1.0.md).
{% endhint %}

Set `queries.storage = "database"` to store scripts in the `prest_queries` table instead of (or in addition to importing from) `.sql` files. Execution URLs stay the same (`/_QUERIES/{location}/{script}`).

```toml
[auth]
enabled = true
migrate_on_startup = true

[jwt]
key = "your-secret"
algo = "HS256"

[queries]
storage = "database"          # default: filesystem
schema = "public"
table = "prest_queries"
register_enabled = true
register_admins = ["admin@example.com"]
restrict = true
migrate_on_startup = true     # default true when storage = "database"
import_on_startup = true      # default true when storage = "database"
import_policy = "update"      # skip | update | error
location = "./queries"        # filesystem import source (also PREST_QUERIES_LOCATION)

[[queries.scripts]]
location = "fulltable"
name = "get_all"
permissions = ["read"]

[[queries.users]]
name = "app_user@example.com"
[[queries.users.scripts]]
location = "fulltable"
name = "get_all"
permissions = ["read"]
```

| Key | Meaning |
|-----|---------|
| `storage` | `filesystem` (default) or `database` |
| `schema` / `table` | Where rows live (default `public.prest_queries`) |
| `migrate_on_startup` | Create the table on API boot when using database storage |
| `import_on_startup` | Import `.sql` files from `location` into the table |
| `import_policy` | `skip`, `update`, or `error` when a script already exists |
| `restrict` | Require auth + script ACL for `/_QUERIES` execution |
| `register_enabled` | Enable admin CRUD on `/_QUERIES/registry` |
| `register_admins` | Usernames allowed to use the registry API |

**Fail-closed:** `register_enabled` is auto-disabled without `auth.enabled`, `jwt.key`, and a non-empty `register_admins`. `restrict` is auto-disabled without `auth.enabled`.

#### CLI migrate

```sh
prestd migrate up queries
prestd migrate down queries
```

#### Registry API

When `register_enabled = true`, admins listed in `register_admins` can manage stored scripts:

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/_QUERIES/registry` | List (`?database=` / `?location=` filters) |
| `POST` | `/_QUERIES/registry` | Create / upsert |
| `GET` | `/_QUERIES/registry/{location}/{name}` | Get one |
| `PUT` | `/_QUERIES/registry/{location}/{name}` | Update |
| `DELETE` | `/_QUERIES/registry/{location}/{name}` | Delete |

Optional `{database}` path segment or query param scopes multi-database aliases.

Create/update JSON body fields include `database`, `location`, `name`, `read_sql`, `write_sql`, `update_sql`, `delete_sql`, and `description` (body size capped at 1 MiB).

Full key list: [`samples/prest.sample.toml`](https://github.com/prest/prest/blob/main/samples/prest.sample.toml). Fixture: [`testdata/prest_queries.toml`](https://github.com/prest/prest/blob/main/testdata/prest_queries.toml).

### Ready-made queries

_consultations ready to use prest_

* [Opps CMS](https://github.com/opps/prest-queries)

## Related

- [Configuring pREST](../get-started/configuring-prest.md)
- [Multi-database](../get-started/multi-database.md)
- [Changes since v2.1.0](../releases/main-since-v2.1.0.md)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [SQL](../prestd/acronyms.md#sql)
