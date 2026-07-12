---
description: >-
  pREST API reference — REST (Representational State Transfer) HTTP endpoints
  that map to SQL on the native PostgreSQL adapter (and certified engines).
---

# API Reference

**pREST** generates HTTP APIs from your database catalog. Today the **native** dialect is PostgreSQL (and Postgres-compatible engines documented under [Databases](../databases/README.md)). This reference covers endpoints, parameters, auth, and advanced query patterns.

## What is REST?

**REST (Representational State Transfer)** is an [architectural style](https://developer.mozilla.org/en-US/docs/Glossary/REST) for building APIs over HTTP. pREST exposes your SQL tables as REST resources at `/{database}/{schema}/{table}`, mapping standard verbs to CRUD on the same server as MCP.

Application backends, mobile and web clients, and any HTTP client that needs CRUD on SQL tables use that surface. This reference documents the endpoints below; see also [Parameters](parameters.md) and [Auth](auth.md).

_**prestd**_ implements HTTP verbs that map to SQL data operations on the connected engine.

### GET

> SQL `SELECT` (PostgreSQL dialect mapping)

| Endpoints                           | Description                                               |
| ----------------------------------- | --------------------------------------------------------- |
| `/_health`                          | Liveness probe — pings default database                 |
| `/_ready`                           | Readiness probe — pings default database and all registered aliases |
| `/_mcp`                             | MCP discovery — server metadata and available read-only tools ([guide](../get-started/mcp-over-http.md), [stdio adapter](../ai/install-prest-mcp.md)) |
| `/databases`                        | List all databases                                        |
| `/schemas`                          | List all schemas                                          |
| `/tables`                           | List all tables                                           |
| `/show/{DATABASE}/{SCHEMA}/{TABLE}` | Lists table structure - all fields contained in the table |
| `/{DATABASE}/{SCHEMA}`              | Lists table tables - find by schema                       |
| `/{DATABASE}/{SCHEMA}/{TABLE}`      | List all rows, find by database, schema and table         |
| `/{DATABASE}/{SCHEMA}/{VIEW}`       | List all rows, find by database, schema and view          |

### POST

| Endpoints | Description |
| --------- | ----------- |
| `/_mcp` | MCP JSON-RPC — `initialize`, `tools/list`, `tools/call` ([guide](../get-started/mcp-over-http.md), [stdio adapter](../ai/install-prest-mcp.md)) |

> SQL `INSERT` (PostgreSQL dialect mapping)

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

> SQL `UPDATE` (PostgreSQL dialect mapping)

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

> SQL `DELETE` (PostgreSQL dialect mapping)

Using query string to make filter (WHERE), example:

```
/{DATABASE}/{SCHEMA}/{TABLE}?{FIELD NAME}={VALUE}
```

> unconditional `delete` can delete unwanted record

### Related

- [Databases](../databases/README.md)
- [Acronyms](../prestd/acronyms.md) ([REST](../prestd/acronyms.md#rest), [MCP](../prestd/acronyms.md#mcp))
- [Parameters](parameters.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Auth](auth.md)
