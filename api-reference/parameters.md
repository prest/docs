# Parameters

_**prestd**_ uses query string to apply filtering, sorting, paginating, and etc to api queries.

### Filters

HTTP method `GET`

| query string                             | Description                                                                                                                                                                                |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `_page={set page number}`                | the api return is paged, this parameter sets which page you want                                                                                                                           |
| `_page_size={number to return by pages}` | delimits the number of records per page, default `10`. Every time you specify a page size, you must include the page you are accessing.                                                    |
| `?_select={field name 1},{field name 2}` | Limit fields list on result. Comma-separated field names are whitespace-trimmed (`id, name` is equivalent to `id,name`). |
| `?_count={field name}`                   | Count per field - `*` representation all fields                                                                                                                                            |
| `?_count_first=true`                     | Query string `_count` returns a list, passing this parameter will return the first record as a non-list object, **by default** this parameter is set to `false` (_return list non-object_) |
| `?_renderer=xml`                         | Set API render syntax, supported: `json` (by default), `xml`                                                                                                                               |
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

**`GROUP BY` with function:**

```
/{DATABASE}/{SCHEMA}/{TABLE}?_groupby=fieldname->>having:GROUPFUNC:FIELDNAME:CONDITION:VALUE-CONDITION
/{DATABASE}/{SCHEMA}/{TABLE}?_select=fieldname00,sum:fieldname01&_groupby=fieldname01->>having:sum:fieldname01:$gt:500
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
