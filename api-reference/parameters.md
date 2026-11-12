# Parameters

_**prestd**_ uses query string to apply filtering, sorting, paginating, and etc to api queries.

### Filters

HTTP method `GET`

| query string                             | Description                                                                                                                                                                                |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `_page={set page number}`                | the api return is paged, this parameter sets which page you want                                                                                                                           |
| `_page_size={number to return by pages}` | delimits the number of records per page, default `10`. Every time you specify a page size, you must include the page you are accessing.                                                    |
| `?_select={field name 1},{field name 2}` | Limit fields list on result. Comma-separated field names are whitespace-trimmed (`id, name` is equivalent to `id,name`). Values are validated — see [`_select` field validation](#_select-field-validation-v230). |
| `?_count={field name}`                   | Count per field - `*` representation all fields                                                                                                                                            |
| `?_count_first=true`                     | Query string `_count` returns a list, passing this parameter will return the first record as a non-list object, **by default** this parameter is set to `false` (_return list non-object_) |
| `?_renderer={format}`                    | Set the response format: `json` (by default), `xml`, `rows` or `columns`. See [Response formats](#response-formats-_renderer) below.                                                     |
| `?_distinct=true`                        | `DISTINCT` clause with SELECT                                                                                                                                                              |
| `?_order={FIELD}`                        | `ORDER BY` in sql query. For `DESC` order, use the prefix `-`. For _multiple_ orders, the fields are separated by comma `fieldname01,-fieldname02,fieldname03`                             |
| `?_groupby={FIELD}`                      | `GROUP BY` in sql query, The grouper is more complicated, a topic has been created to describe how to use                                                                                  |
| `?{FIELD NAME}={VALUE}`                  | Filter by field, you can set as many query parameters as needed                                                                                                                            |
| `?_or={CONDITION}\|\|{CONDITION}`        | OR clause filtering — combine alternatives with `\|\|`. See [OR clause filtering](#or-clause-filtering) below. |

#### Functions support

Used to perform data **aggregation**(**grouping** and **selection**)

| name     | Use in request   |
| -------- | ---------------- |
| SUM      | `sum:field`      |
| AVG      | `avg:field`      |
| MAX      | `max:field`      |
| MIN      | `min:field`      |
| STDDEV   | `stddev:field`   |
| VARIANCE | `variance:field` |

**`SELECT` with function:**

```
/{DATABASE}/{SCHEMA}/{TABLE}?_select=fieldname00,sum:fieldname01&_groupby=fieldname01
```

**Naming the result of a function:**

An aggregate's column is named after its function: `sum:salary` comes back as `sum`. Add a third part to name it yourself, `function:field:alias`, which pREST turns into `SUM("salary") AS "alias"`. Name aggregates whenever a request has two of the same function, as below. Without aliases both would be called `sum`: with the default `PREST_JSON_AGG_TYPE=jsonb_agg` only the last one stays in the response, and with `json_agg` both come back under the same key.

```http
GET /db/public/employees?_select=age,sum:salary:total_salary,sum:age:total_age&_groupby=age
```

```json
[{"age": 19, "total_age": 38, "total_salary": 7997}, {"age": 20, "total_age": 40, "total_salary": 1350}]
```

- The alias must be a plain identifier: a letter or `_`, then letters, digits or `_`, at most 63 characters, no dot. Anything else is **400** `invalid identifier`.
- `_order` can sort by the alias: `_order=-total_salary`.
- Aliases also work without `_groupby` (`_select=sum:salary:total,max:age:oldest` returns one row) and on MySQL.

**`GROUP BY` with function:**

```
/{DATABASE}/{SCHEMA}/{TABLE}?_groupby=fieldname->>having:GROUPFUNC:FIELDNAME:CONDITION:VALUE-CONDITION
/{DATABASE}/{SCHEMA}/{TABLE}?_select=fieldname00,sum:fieldname01&_groupby=fieldname01->>having:sum:fieldname01:$gt:500
```
### `_select` field validation (v2.3.0)

Since **v2.3.0** ([#1002](https://github.com/prest/prest/pull/1002), [GHSA-qvx3-q8vx-9q3c](https://github.com/prest/prest/security/advisories/GHSA-qvx3-q8vx-9q3c)), `_select` and `_count` values pass through a single validation gate that closes an unauthenticated SQL-injection. Each comma-separated field must be one of:

| Form | Example |
|------|---------|
| Wildcard | `*` |
| Identifier, optionally dotted | `id`, `public.users.name` |
| Joined column (with `_join`) | `users.name` |
| Table wildcard (v2.5.0, with `_join`) | `users.*` |
| Colon-syntax aggregate, optionally aliased | `sum:salary`, `avg:rating`, `sum:salary:total` |
| Pre-quoted aggregate | `SUM("salary") AS "total"` |

Aggregates are limited to `SUM`, `AVG`, `MAX`, `MIN`, `STDDEV`, `VARIANCE`. Anything else — subselects, `pg_*` probing, or extra parentheses — returns **`400` `ErrInvalidIdentifier`**. `_count` field names are quoted in the generated SQL (`, celphone` → `, "celphone"`).

With [`_join`](advanced-queries.md#join), a bare name refers to the queried table; qualify joined columns as `table.column`. Under `[access] restrict = true`, entries you may not read are dropped, and if none remain the request is **400** — see [Permissions — JOINs and restrict](../get-started/permissions.md#joins-and-restrict-v250).

For projections that need arbitrary SQL expressions, use a [custom query](custom-queries.md) instead.

### pgvector KNN ordering and distance filtering (v2.4.0)

Since **v2.4.0** ([#1011](https://github.com/prest/prest/pull/1011)), two query-parameter forms are available for `vector`-typed columns (requires the `pgvector` extension on the target database):

| Parameter | Form | Example | Effect |
|---|---|---|---|
| `_korder` | `<column>:<metric>:<vector>` | `_korder=embedding:l2:[1,0,0]` | Orders by nearest-neighbor distance (KNN); composes with `_order` as an additional sort term |
| `<column>:vecdist` | `<metric>:<comparison>:<vector>:<threshold>` | `embedding:vecdist=l2:lt:[1,0,0]:0.5` | Filters rows by distance threshold |

Metrics are restricted to a fixed whitelist: `l2`/`euclidean`, `cosine`/`cos`, `ip`/`inner`/`dot`, `l1`/`manhattan`. `:vecdist` comparisons are restricted to `=`, `!=`, `<`, `<=`, `>`, `>=` (non-scalar comparisons like `like` are rejected). The column goes through identifier validation, the vector literal round-trips through `ParseFloat`/`FormatFloat`, and the threshold is passed as a bound parameter — malformed metrics, non-numeric vector elements, oversized vectors (>16000 dims, pgvector's own limit), and dimension mismatches all return `400` rather than reaching the database unsafely.

```http
GET /db/public/docs?_korder=embedding:cosine:[0.1,0.2,0.3]&_page_size=5
GET /db/public/docs?embedding:vecdist=l2:lt:[0.1,0.2,0.3]:0.5
```

### Operators Reference Guide

The following operators are used for filtering data in queries. Each operator defines a specific matching condition that determines which records are included in the result set.

---

| Operator         | Description                                                        | Example Usage                                           |
| ---------------- | ------------------------------------------------------------------ | ------------------------------------------------------- |
| `$eq`            | Matches values that are equal to a specified value.                | `status=$eq.active`                                     |
| `$gt`            | Matches values greater than a specified value.                     | `age=$gt.25`                                            |
| `$gte`           | Matches values greater than or equal to a specified value.         | `salary=$gte.50000`                                     |
| `$lt`            | Matches values less than a specified value.                        | `experience=$lt.5`                                      |
| `$lte`           | Matches values less than or equal to a specified value.            | `rating=$lte.4.5`                                       |
| `$ne`            | Matches values that are not equal to a specified value.            | `status=$ne.closed`                                     |
| `$in`            | Matches any of the values specified in an array.                   | `role=$in.admin,editor,viewer`                          |
| `$nin`           | Matches none of the values specified in an array.                  | `department=$nin.hr,finance`                            |
| `$contains`      | Matches array columns that hold every listed element. See [Array containment](#array-containment). | `tags=$contains.go,sql`                                 |
| `$null`          | Matches if the field value is null.                                | `remarks=$null`                                         |
| `$notnull`       | Matches if the field value is not null.                            | `remarks=$notnull`                                      |
| `$true`          | Matches if the field value is true.                                | `is_verified=$true`                                     |
| `$nottrue`       | Matches if the field value is not true.                            | `is_verified=$nottrue`                                  |
| `$false`         | Matches if the field value is false.                               | `is_active=$false`                                      |
| `$notfalse`      | Matches if the field value is not false.                           | `is_active=$notfalse`                                   |
| `$like`          | Matches the entire string (case-sensitive).                        | `name=$like.John%`                                      |
| `$ilike`         | Matches the entire string, case-insensitive.                       | `city=$ilike.mumbai%`                                   |
| `$nlike`         | Excludes matches that cover the entire string.                     | `email=$nlike.%@test.com`                               |
| `$nilike`        | Excludes matches, case-insensitive.                                | `email=$nilike.%@gmail.com`                             |
| `$ltreelanc`     | Checks if left argument is an ancestor of right (or equal).        | `category_path=$ltreelanc.electronics`                  |
| `$ltreerdesc`    | Checks if left argument is a descendant of right (or equal).       | `category_path=$ltreerdesc.electronics.mobiles`         |
| `$ltreematch`    | Checks if ltree matches lquery.                                    | `tags=$ltreematch.tech.*`                               |
| `$ltreematchtxt` | Checks if ltree matches ltxtquery.                                 | `tags=$ltreematchtxt.smartphone & android`              |

---

### Array containment

{% hint style="warning" %}
**Requires prest `main` (unreleased)** — [#1094](https://github.com/prest/prest/pull/1094).
{% endhint %}

`$contains` selects rows whose array column holds every listed element. `$in` works the other way round: it compares a scalar column against a list.

```http
GET /db/public/users?blocked_user_ids=$contains.3
GET /db/public/users?blocked_user_ids=$contains.1,3
```

The first request matches users whose `blocked_user_ids` holds `3`, the second those holding both `1` and `3`.

- Separate elements with a comma, as with `$in`. pREST binds them as one array parameter and generates `"column" @> $1`, so PostgreSQL types the parameter as the column's own array type (`int[]`, `text[]`, `varchar(n)[]`, `uuid[]`, …) and can use a GIN index on the column.
- `$contains` also works inside `_or`.
- On a column that is not an array the request fails with **400**. On a [jsonb field](advanced-queries.md#jsonb-support) (`FIELD->>JSONFIELD:jsonb`) it fails with `invalid operator`, because `->>` yields text.
- MySQL answers `unsupported operator` — see [MySQL](../databases/mysql.md).

---

### OR clause filtering

Use `_or` to combine filter conditions with OR logic without writing a custom SQL query. Available since v2.0.0-rc6, included in [v2.0.0](../releases/v2.0.0.md).

Each alternative is `field=$operator.value` using the same operators as the table above. Separate alternatives with `||` (double pipe). The OR group is parenthesized and AND-combined with other query parameters.

```http
GET /db/public/articles?_or=title=$ilike.%search%||name=$ilike.%search%
GET /db/public/items?_or=status=$eq.active||status=$eq.pending&category=$eq.tech
```

The second example matches rows where `(status = 'active' OR status = 'pending') AND category = 'tech'`.

**Notes:**

- Empty or malformed `_or` values are ignored.
- Use `||` to separate alternatives — not a literal ` OR ` inside values.
- Comma-separated values within a single alternative (e.g. `$in`) are preserved.

---

### Notes
- Comma (`,`) is used to separate multiple values in `$in` and `$nin` operators.
- Pattern matching operators like `$like` and `$ilike` support SQL wildcards (`%`, `_`).
- LTree operators (`$ltreelanc`, `$ltreerdesc`, etc.) are useful for hierarchical data filtering.

---

### Returning rows from writes (`_returning`)

`PUT`, `PATCH`, and `DELETE` on `/{database}/{schema}/{table}` answer `{"rows_affected": n}` by default. Add `_returning` to get the affected rows instead, as SQL `RETURNING` does:

| query string         | Effect                                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------- |
| `_returning=*`       | Every column of each affected row                                                            |
| `_returning={field}` | Only that column. Repeat the parameter for more columns: `_returning=id&_returning=status`   |

```http
PATCH /db/public/orders?id=$eq.42&_returning=*
DELETE /db/public/orders?status=$eq.cancelled&_returning=id
```

The response is an array with one object per affected row:

```json
[{"id": 42, "status": "shipped", "total": 99.9}]
```

- Each field must be a column name, optionally dotted. Anything else is **400** `could not perform ReturningByRequest: Returning: invalid identifier`.
- `POST` already answers with the inserted row, so it does not take `_returning`.
- With `[access] restrict = true`, returned rows carry only the columns the user may read (the `read` permission's `fields`, see [Permissions](../get-started/permissions.md)). Since **v2.5.3** ([#1072](https://github.com/prest/prest/pull/1072)).
- MySQL has no SQL `RETURNING`. pREST builds the same response inside a transaction — see [MySQL](../databases/mysql.md).

---

### Response formats (`_renderer`)

Every response is JSON unless the request sets `_renderer`. It applies to every route that returns data: table reads, `_count`, `_groupby`, `_join`, custom queries and the catalog listings (`/databases`, `/schemas`, `/tables`).

| `_renderer`        | Response                                                                   | `Content-Type`     |
| ------------------ | -------------------------------------------------------------------------- | ------------------ |
| `json` (default)   | An array of row objects, as everywhere else on this page                   | `application/json` |
| `xml`              | Each row an `<object>` inside one `<objects>` element                      | `application/xml`  |
| `rows`             | Column names once in `metadata`, then one array of values per row          | `application/json` |
| `columns`          | Column names once in `metadata`, then one array of values per column       | `application/json` |

Any other `_renderer` value is ignored, and the response is JSON.

```http
GET /db/public/employees?_select=name,age&_renderer=xml
```

```xml
<objects><object><age>20</age><name>joao</name></object><object><age>19</age><name>maria</name></object></objects>
```

An error is XML too: `<objects><error>…</error></objects>`, with the error's status code.

#### Rows and columns

{% hint style="warning" %}
**Requires prest `main` (unreleased)** — [#1104](https://github.com/prest/prest/pull/1104).
{% endhint %}

`rows` and `columns` return the same data as arrays of arrays and send each column name once, instead of repeating it in every row object. Chart and data-grid libraries take this shape as input, and it drops the repeated keys from the payload.

```http
GET /db/public/employees?_select=name,age&_renderer=rows
GET /db/public/employees?_select=name,age&_renderer=columns
```

```json
{"metadata":["age","name"],"rows":[[20,"joao"],[19,"maria"]]}
{"metadata":["age","name"],"columns":[[20,19],["joao","maria"]]}
```

- `metadata` lists the columns in the order they have in the JSON response, and every row (or column) follows that order. With the default `PREST_JSON_AGG_TYPE=jsonb_agg`, PostgreSQL orders the keys itself (shorter names first), which is why `age` comes before `name` above. With `json_agg` they keep the order of `_select`. See [Configuring pREST](../get-started/configuring-prest.md).
- Values are copied as they are in the JSON response: large integers, decimals such as `1.50` and nested JSON come through unchanged.
- A column missing from some rows is `null` in those rows.
- A key that appears twice in a row object keeps a column for each value, as `json_agg` returns two aggregates of the same function (`sum:a` and `sum:b`). Name them with an [alias](#functions-support) to tell the columns apart.
- A single object, such as the row a `POST` returns or `_count_first=true`, becomes one row. An empty result is `{"metadata":[],"rows":[]}` (or `"columns":[]`), not `null`.
- An error keeps its usual `{"error": "…"}` body and status code.
