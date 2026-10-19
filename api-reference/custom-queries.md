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

When the database prefix is omitted, the default database (`pg.database`) is used. Without a registry, the prefix must be `pg.database` or a name listed in `pg.allowed_databases` (v2.5.2); any other name is **400** `database not registered: <name>`.

{% hint style="warning" %}
**Interpolated values are screened.** A value written into the SQL text — `'{{.field1}}'` above — becomes part of the statement, so pREST rejects values carrying quotes, `--`, `::`, or (for multi-word values) a SQL keyword. Since **v2.4.2** ([#1023](https://github.com/prest/prest/pull/1023)), a rejected value **fails the request with `400`** rather than being silently replaced by an empty string.

For anything user-supplied, [bind the value](#binding-values-sqlval-sqllist-ident-v242) with `{{sqlVal "field1"}}` instead — bound values skip the screen entirely and can never be parsed as SQL.
{% endhint %}

**To activate filesystem scripts**, set a location in `prest.toml`:

```toml
[queries]
location = "/path/to/queries/"
```

Default storage is the filesystem (`queries.storage = "filesystem"`). Database-backed storage is available in **v2.2.0+** — see [below](#database-backed-storage-v220).

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

Header values go through the same screen as query parameters. A rejected header is blanked and logged at `WARN` rather than failing the request — an ordinary `User-Agent` contains `(` and `;`, which the character allow-list refuses, so erroring would reject nearly every browser request.

The keys `header`, `_param`, and `_header` are reserved for template data. Query parameters using those names are ignored.

#### Credential headers (v2.4.2)

Since **v2.4.2** ([#1023](https://github.com/prest/prest/pull/1023)), credential-bearing headers are withheld from templates entirely and render as an empty string:

`Authorization` · `Proxy-Authorization` · `Cookie` · `X-Api-Key` · `X-Auth-Token` · `X-Access-Token`

They are withheld from the bound form too, so `{{sqlVal "header.Authorization"}}` is also empty — blanking them is about secrecy, and binding must not become a way around it. A bearer token is plain base64url text that passed the value screen untouched, so a template referencing it interpolated the caller's credential into SQL that was then logged.

The request still succeeds; only the value is gone. A script that scoped rows by the caller's token now matches nothing — move that logic to [Permissions](../get-started/permissions.md), or pass a non-credential header such as `X-UserId`.

### Binding values (`sqlVal`, `sqlList`, `ident`) (v2.4.2)

Bind a value instead of interpolating it and the screen does not apply at all. A bound value travels to PostgreSQL out of band as a query parameter, where it can never be parsed as SQL:

```sql
-- interpolated: screened, and rejected for values like 'compra do mes'
SELECT * FROM articles WHERE slug = '{{.slug}}'

-- bound: the caller's value arrives verbatim, whatever it contains
SELECT * FROM articles WHERE slug = {{sqlVal "slug"}}
```

| Helper | Use for | Renders |
|--------|---------|---------|
| `{{sqlVal "key"}}` | A single value | `$1` |
| `{{sqlList "key"}}` | A repeated query parameter (`?tag=a&tag=b`) | `($1,$2)` |
| `{{ident "key"}}` | A table or column name, which cannot be bound | `"public"."users"` |

`sqlVal` and `sqlList` also reach headers, using a `header.` prefix:

```sql
SELECT * FROM tenants WHERE app = {{sqlVal "header.X-Application"}}
```

These helpers have existed since v2.0.0, but before **v2.4.2** they bound the *screened* value — so a rejected value bound as `""`. Since v2.4.2 ([#1023](https://github.com/prest/prest/pull/1023)) they resolve the raw value, which is what makes binding a complete alternative to the screen.

{% hint style="info" %}
Prefer binding for anything user-supplied — search phrases especially. A phrase containing a common word such as `do`, `as`, or `or` is exactly what the interpolation screen refuses.
{% endhint %}

**When a value is rejected**, the request fails with `400` and a message naming the parameter. The value itself is never echoed back:

```json
{
  "error": "invalid value for parameter slug: it contains SQL syntax that cannot be interpolated safely; use the sqlVal template helper to bind free-form values"
}
```

This applies to anything that renders the value into SQL text — `{{.slug}}`, `{{inFormat "slug"}}`, `{{unEscape .slug}}`. `sqlVal` and `sqlList` are exempt.

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

#### unEscape

URL-decodes a string (percent-encoding and `+`). The result is written into the SQL text, so it is subject to the [value screen](#binding-values-sqlval-sqllist-ident-v242) — bind with `sqlVal` when the value comes from the caller.

```sql
SELECT * FROM table WHERE path = '{{unEscape .path}}'
```

### Database-backed storage (v2.2.0+)

{% hint style="info" %}
Available in **pREST v2.2.0+** ([#980](https://github.com/prest/prest/pull/980)). See [v2.2.0 release notes](../releases/v2.2.0.md).
{% endhint %}

Set `queries.storage = "database"` to store scripts in the `prest_queries` table instead of (or in addition to importing from) `.sql` files. Execution URLs stay the same (`/_QUERIES/{location}/{script}`).

{% hint style="danger" %}
**The queries table is reserved from v2.5.2.** Up to v2.5.1, the generic table routes could write `prest_queries`, so with the default `[access] restrict = false` anyone could store SQL and then run it through `/_QUERIES` ([GHSA-2x9q-r7j9-g85f](https://github.com/prest/prest/security/advisories/GHSA-2x9q-r7j9-g85f), critical). From v2.5.2 the table (and your configured `queries.table`) is unreachable through CRUD, `/batch`, `/show`, `_join`, and MCP; the [registry API](#registry-api) is the only way to write it.

Upgrading from v2.5.1 or earlier? The fix does not remove rows already stored — [audit the table](../releases/v2.5.2.md#upgrading-from-v251).
{% endhint %}

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
| `schema` / `table` | Where rows live (default `public.prest_queries`). The table is [reserved](../get-started/permissions.md#reserved-tables-v252) from v2.5.2 |
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

## Troubleshooting

| Response | Cause |
|---|---|
| `400` — `invalid value for parameter <name>: …` | An interpolated value was rejected by the screen. [Bind it](#binding-values-sqlval-sqllist-ident-v242) with `sqlVal`. |
| `400` — `database not registered: <name>` | No registry, and `<name>` is neither `pg.database` nor in `pg.allowed_databases` (v2.5.2). |
| `400` — `invalid identifier in path` | The database, folder, or script name contains characters outside the allow-list, including `.` — so `get_all.read` cannot be requested directly. |
| `400` — `invalid script path: <folder>/<script>` | The resolved `.sql` file lies outside the queries directory. Since **v2.4.2** ([#1023](https://github.com/prest/prest/pull/1023)), `..` segments and symlinks escaping the tree are rejected both lexically and after symlink resolution. |
| `400` — `could not parse script <folder>/<script>, check your prest logs` | A template parse or render error. Since **v2.4.2** the detail is logged rather than returned, so read the `prestd` logs. |
| Empty value where a header was expected | The header is a [credential header](#credential-headers-v242) and is withheld from templates. |

{% hint style="info" %}
Since **v2.4.2**, SQL generated from custom query scripts is **not written to logs at any level** — the statement is caller-influenced. Use PostgreSQL's own statement logging when you need to see it. See [Configuring pREST — Logging](../get-started/configuring-prest.md#logging).
{% endhint %}

## Related

- [Configuring pREST](../get-started/configuring-prest.md)
- [Multi-database](../get-started/multi-database.md)
- [Permissions](../get-started/permissions.md)
- [v2.5.2 release notes](../releases/v2.5.2.md) · [v2.4.2 release notes](../releases/v2.4.2.md) · [v2.2.0 release notes](../releases/v2.2.0.md)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [SQL](../prestd/acronyms.md#sql)
