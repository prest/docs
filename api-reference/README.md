---
description: Endpoints of the API
---

# API Reference

_**prestd**_ automatically generates an API based on the structure of the connected database (Postgres), in this _API reference_ session you will understand all the endpoints, parameters, advanced methods, auth and much more.

_**prestd**_ implements all http verbs, transcribing to SQL ANSI (American National Standards Institute) statements.

### GET

> Postgres `SELECT` instruction

| Endpoints                           | Description                                               |
| ----------------------------------- | --------------------------------------------------------- |
| `/_health`                          | Health check endpoint                                     |
| `/databases`                        | List all databases                                        |
| `/schemas`                          | List all schemas                                          |
| `/tables`                           | List all tables                                           |
| `/show/{DATABASE}/{SCHEMA}/{TABLE}` | Lists table structure - all fields contained in the table |
| `/{DATABASE}/{SCHEMA}`              | Lists table tables - find by schema                       |
| `/{DATABASE}/{SCHEMA}/{TABLE}`      | List all rows, find by database, schema and table         |
| `/{DATABASE}/{SCHEMA}/{VIEW}`       | List all rows, find by database, schema and view          |

### POST

> Postgres `INSERT` instruction

```
/{DATABASE}/{SCHEMA}/{TABLE}
```

**JSON DATA:**

```
{
    "FIELD1": "string value",
    "FIELD2": 1234567890
}
```

### PATCH and PUT

> Postgres `UPDATE` instruction

Using query string to make filter (WHERE), example:

```
/{DATABASE}/{SCHEMA}/{TABLE}?{FIELD NAME}={VALUE}
```

JSON DATA:

```
{
    "FIELD1": "string value",
    "FIELD2": 1234567890,
    "ARRAYFIELD": ["value 1","value 2"]
}
```

> unconditional `update` can update unwanted record

### DELETE

> Postgres `DELETE` instruction

Using query string to make filter (WHERE), example:

```
/{DATABASE}/{SCHEMA}/{TABLE}?{FIELD NAME}={VALUE}
```

> unconditional `delete` can delete unwanted record
