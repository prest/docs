# Advanced Queries

_**prestd**_ allows you to do some advanced queries, but with some limitations in order to make the URL and parameters as clean as possible.

### Filter range

It is common to query a database in a range of time, e.g:

> I need all records that were created from `1997-11-03-03` to `1997-12-05`.

`created_at` field is a timestamp field, so you can use the `$gte` and `$lte` operators to filter by date range.

```sql
SELECT * FROM {SCHEMA}.{TABLE} WHERE created_at >= '1997-11-03-03' AND created_at <= '1997-12-05'
```

**How to do it in prestd?**

```http
GET /{DATABASE}/{SCHEMA}/{TABLE}?created_at='$gte.1997-11-03'&created_at='$lte.1997-12-0'
```

There are other types of operators, see them all here.

### JOIN

HTTP verb `GET`. Joins another table onto the queried one. Repeat `_join` to join several tables (v2.5.0, [#1033](https://github.com/prest/prest/pull/1033)); the clauses apply in the order given. One malformed clause fails the whole request with **400**.

```
/{DATABASE}/{SCHEMA}/{TABLE}?_join={TYPE}:{TABLE JOIN}:{TABLE.FIELD}:{OPERATOR}:{TABLE JOIN.FIELD}
```

Parameters:

1. Type (case-insensitive):
   * `inner`
   * `left`
   * `right`
   * `full` — not on [MySQL](../databases/mysql.md#known-limitations)
   * `cross`
2. **Table** used in the join — `table` or `schema.table`
3. **Table.field** - table name **dot** field
4. Operator:
   * `$eq`
   * `$ne`
   * `$lt`
   * `$gt`
   * `$lte`
   * `$gte`
5. **Table2.field** - table name **dot** field

Using query string to JOIN tables, example:

```
/{DATABASE}/{SCHEMA}/friends?_join=inner:users:friends.userid:$eq:users.id
```

Two joins, selecting columns from each table:

```
/{DATABASE}/{SCHEMA}/friends?_join=inner:users:friends.userid:$eq:users.id&_join=left:badges:users.id:$eq:badges.user_id&_select=friends.id,users.name,badges.badge
```

**Selecting joined columns.** In `_select`, a bare column name means the queried table. Write a joined table's column as `table.column`, or `table.*` for all of its columns (v2.5.0). Response keys are the bare column names, so give the same column from two tables different names with a [custom query](custom-queries.md) if you need both.

**Rejected joins** (400):

* Joining the queried table itself, or the same table twice (with or without a schema prefix): `table "<name>" is joined more than once: invalid join clause`.
* Any type other than the five above, for example `outer`: `invalid join clause`.
* Joining a [reserved table](../get-started/permissions.md#reserved-tables-v252) such as `prest_users` (v2.5.2): `table is reserved for pREST internal use`.

With `[access] restrict = true`, each joined table is checked against its own permissions: a table you cannot read adds no columns but still filters rows. See [Permissions — JOINs and restrict](../get-started/permissions.md#joins-and-restrict-v250).

For joins this syntax cannot express, use a [custom query](custom-queries.md).

### JSONb support

PostgreSQL offers type for storing jsonb data. To implement efficient query mechanisms for these data types.

```
?FIELD->>JSONFIELD:jsonb=VALUE
```

#### Example of how to do insertion via `cURL`

**Fields:**

* id: `INT`
* data: `JSONB`

````sh

```sh
curl -i -X POST 'http://127.0.0.1:3000/prest/public/t_mydata' \
  -H 'Content-Type: application/json' \
  -d '{"id": 80, "data": "{\"a\": 123}"}'
````

> [reference](https://github.com/prest/prest/issues/815)

### Full Text Search (with tsquery)

Full Text Searching (or just text search) provides the capability to identify natural-language documents that satisfy a query, and optionally to sort them by relevance to the query. The most common type of search is to find all documents containing given query terms and return them in order of their similarity to the query. Notions of query and similarity are very flexible and depend on the specific application. The simplest search considers query as a set of words and similarity as the frequency of query words in the document.

Is native feature of PostgreSQL since version 8.3, read more [here](https://www.postgresql.org/docs/9.5/textsearch-intro.html).

> A tsquery value stores lexemes that are to be searched for, and combines them honoring the Boolean operators & (AND), | (OR), and ! (NOT). Parentheses can be used to enforce grouping of the operators. `SELECT 'fat & rat'::tsquery;`

> **Security note (v2 rc5):** tsquery input validation was hardened ([#940](https://github.com/prest/prest/pull/940)). Always pass well-formed tsquery values — do not rely on pREST to sanitize malformed input.

```
?FIELD:tsquery=VALUE
```

**Set language**

You can specify the language you want to tokenize in, for example: **portuguese**

```
FIELD$LANGUAGE:tsquery=VALUE
```

**Language list:**

* simple
* arabic
* danish
* dutch
* english
* finnish
* french
* german
* hungarian
* indonesian
* irish
* italian
* lithuanian
* nepali
* norwegian
* portuguese
* romanian
* russian
* spanish
* swedish
* tamil
* turkish

To see all the languages available in your PostgreSQL run this query:

```sql
SELECT cfgname FROM pg_ts_config;
```

### Batch Insert

HTTP verb `POST`, you can insert many rows at once using batch endpoint `/batch/...`.

```
/batch/DATABASE/SCHEMA/TABLE

```

**JSON DATA:**

```
[
    {"FIELD1": "string value", "FIELD2": 1234567890},
    {"FIELD1": "other string value", "FIELD2":1234567891},
]
```

The default insert method is using multiple tuple values like `insert into table values ("value", 123), ("other", 456)`. Returns inserted rows.

You can change this behavior using the header `Prest-Batch-Method` with value `copy`. It's useful for large insertions, but the return is empty.
