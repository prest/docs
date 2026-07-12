---
description: >-
  pREST MCP over HTTP — Model Context Protocol (MCP) read-only tools at /_mcp
  for AI agents to discover schemas and query SQL tables on the same server as REST.
---

# MCP over HTTP

pREST v2.1.0+ exposes a read-only [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) endpoint at `/_mcp` on the same server that already serves CRUD, catalog, and custom script routes. MCP clients can discover schema and read table data through JSON-RPC tools — without a separate process or transport.

Today MCP runs on the **Native PostgreSQL adapter** and [Postgres-compatible engines](../databases/README.md) you connect with that adapter. Other SQL families are on the [roadmap](../databases/roadmap.md).

Introduced in [v2.1.0](../releases/v2.1.0.md) ([#977](https://github.com/prest/prest/pull/977)).

---

## What is MCP?

**MCP (Model Context Protocol)** is an [open standard](https://modelcontextprotocol.io/) for connecting AI applications and agents to external tools and data sources. Instead of each product inventing a private plugin format, MCP defines how clients discover tools and call them (for example over JSON-RPC).

AI agents, IDEs, and other MCP-capable clients use that protocol to list available tools and run them with structured arguments. pREST implements a **read-only** MCP surface at `/_mcp` so those clients can:

- Discover databases, schemas, and tables  
- Describe columns  
- Select rows (with limits)  

through the **same** `prestd` process, auth, ACL, and database routing as the REST API.

MCP in pREST is **not** a separate database, a second port, or a write/DDL interface. In v2.1.0 it does not insert, update, delete, or run arbitrary admin SQL — use REST or curated `/_QUERIES` scripts for writes when your deployment allows them.

---

## How it fits in pREST

The MCP endpoint reuses the existing pREST request pipeline:

```text
MCP client → GET or POST /_mcp → auth / ACL → catalog & query execution → SQL database (PG family today)
```

That means MCP inherits the same deployment model, authentication, access control, and database routing as the REST API. There is no second port to configure: MCP runs in-process on the same `prestd` HTTP server.

Many AI clients (Cursor, Claude Desktop, and others) expect a **stdio** MCP process rather than HTTP. For those, use the official [pREST MCP Adapter](../ai/install-prest-mcp.md) (`prest-mcp`) — a tiny stdio ↔ HTTP bridge. The adapter does not implement tools; pREST still owns schema discovery and queries. See [AI and MCP](../ai/README.md) for install and client tutorials.

| Concern | Behavior |
|---------|----------|
| Server process | Same `prestd` instance as REST |
| Authentication | HTTP auth stack — `/_mcp` is protected when auth is enabled |
| Permissions | `access.tables` and per-user rules apply to tool discovery and execution |
| Multi-database | Database aliases from the registry are used in tool names and arguments |
| Writes | **Not supported** in v2.1.0 — read-only by design |

---

## Endpoint shape

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/_mcp` | Discovery payload with server metadata and available tools |
| `POST` | `/_mcp` | JSON-RPC 2.0 requests for MCP operations |

Supported JSON-RPC methods:

| Method | Description |
|--------|-------------|
| `initialize` | Returns server info, capabilities, and usage instructions |
| `tools/list` | Returns the full tool catalog with `inputSchema` per tool |
| `tools/call` | Executes a named tool with optional arguments |

Unsupported methods return `400 Bad Request` with `unsupported method`.

---

## Discovery (`GET /_mcp`)

A `GET` request returns a JSON document describing the server and all tools visible to the current caller (after auth and ACL filtering):

```json
{
  "name": "prest",
  "protocol": "0.1",
  "endpoint": "/_mcp",
  "description": "Read-only MCP endpoint backed by pREST catalog and query execution.",
  "tools": [
    {
      "name": "prest.list_databases",
      "description": "List accessible databases.",
      "inputSchema": { "type": "object", "properties": {} }
    }
  ],
  "capabilities": {
    "tools": { "listChanged": false }
  }
}
```

Use discovery to inspect available tools and their typed `inputSchema` before calling them.

---

## Tools reference

### `prest.list_databases`

List database aliases accessible to the current caller.

**Arguments:** none.

**Returns:** array of database objects. In registry multi-database mode, each entry includes:

| Field | Description |
|-------|-------------|
| `name` | Registered alias (used in URLs and tool names) |
| `datname` | Same as `name` |
| `physical_name` | Physical Postgres database name on the target cluster |

In legacy single-host mode, returns databases from the catalog query.

---

### `prest.list_schemas`

List readable schemas for a database alias.

**Arguments:**

| Field | Required | Description |
|-------|----------|-------------|
| `database` | No | Database alias; defaults to `pg.database` when omitted |

**Returns:** array of schema objects (filtered by `access.tables` permissions).

---

### `prest.list_tables`

List readable tables for a database alias and optional schema.

**Arguments:**

| Field | Required | Description |
|-------|----------|-------------|
| `database` | No | Database alias; defaults to `pg.database` when omitted |
| `schema` | No | Filter to a single schema; omit to list across accessible schemas |

**Returns:** array of table objects with `name`, `schema`, and related metadata.

---

### `prest.describe_table`

Return column metadata for a table.

**Arguments:**

| Field | Required | Description |
|-------|----------|-------------|
| `database` | No | Database alias; defaults to `pg.database` when omitted |
| `schema` | Yes | Schema name |
| `table` | Yes | Table name |

**Returns:**

```json
{
  "database": "prest-test",
  "schema": "public",
  "table": "test",
  "columns": [
    {
      "name": "id",
      "data_type": "integer",
      "nullable": false,
      "position": 1
    }
  ],
  "count": 1
}
```

Column objects may also include `max_length`, `generated`, `updatable`, and `default_value` when available from the catalog.

---

### `prest.select_table`

Read rows from a table using a generic tool name.

**Arguments:**

| Field | Required | Description |
|-------|----------|-------------|
| `database` | No | Database alias; defaults to `pg.database` when omitted |
| `schema` | Yes | Schema name |
| `table` | Yes | Table name |
| `columns` | No | Project specific columns; omit to select all permitted columns |
| `filters` | No | Column-aware equality filters (see below) |
| `order_by` | No | Sort fields — use `field` for ascending, `-field` for descending |
| `limit` | No | Row limit (1–100); defaults to **100** when omitted or out of range |
| `offset` | No | Row offset; defaults to **0** |

**Returns:**

```json
{
  "database": "prest-test",
  "schema": "public",
  "table": "Reply",
  "columns": ["id", "name"],
  "rows": [{ "id": 1, "name": "prest tester" }],
  "count": 1
}
```

---

### `prest.select.{database}.{schema}.{table}`

Schema-aware tools generated from the catalog at startup. Each readable table gets its own tool with a typed `inputSchema` derived from column metadata.

**Tool name pattern:**

```text
prest.select.{database}.{schema}.{table}
```

Example: `prest.select.prest-test.public.Reply`

**Arguments:** same as `prest.select_table`, except `database`, `schema`, and `table` are encoded in the tool name and cannot be overridden.

The generated `inputSchema` exposes:

- allowed `columns` (enum of permitted column names)
- allowed `order_by` values (`field` and `-field` for each column)
- column-aware `filters` with types matching each column's data type
- `limit` (integer, 1–100) and `offset` (integer, minimum 0)

In multi-database mode, tools are generated for every registered alias. Tables the caller cannot read are omitted from the tool list.

---

## Examples

### Initialize

```http
POST /_mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize"
}
```

Response includes `serverInfo`, `capabilities`, and `instructions`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "serverInfo": { "name": "prest", "version": "0.1" },
    "capabilities": { "tools": { "listChanged": false } },
    "instructions": "Read-only tools are exposed through pREST auth and ACL."
  }
}
```

### List tools

```http
POST /_mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list"
}
```

### Describe a table

```http
POST /_mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "prest.describe_table",
    "arguments": {
      "database": "prest-test",
      "schema": "public",
      "table": "test"
    }
  }
}
```

### Schema-aware select

```http
POST /_mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "prest.select.prest-test.public.Reply",
    "arguments": {
      "columns": ["id", "name"],
      "filters": { "name": "prest tester" },
      "order_by": ["id"],
      "limit": 5,
      "offset": 0
    }
  }
}
```

### List databases

```http
POST /_mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "tools/call",
  "params": {
    "name": "prest.list_databases"
  }
}
```

---

## Authentication and permissions

When [auth](../api-reference/auth.md) is enabled, `/_mcp` requires the same credentials as other protected routes. Send the appropriate `Authorization` header (or basic auth, depending on your configuration) with every `GET` and `POST` request.

[Permissions](permissions.md) apply to MCP the same way they apply to REST:

- Tool discovery only lists tables and columns the caller can read.
- `tools/call` on a table without read permission returns an error.
- Per-user `[[access.users]]` rules are respected.

If a tool list appears empty or a select returns a permission error, check your `access.tables` configuration.

---

## Multi-database

MCP tools use **database aliases** — the same identifiers used in REST URLs (`/{database}/{schema}/{table}`). When the `database` argument is omitted, pREST uses the default database (`pg.database`).

| Mode | `database` in tools | See also |
|------|---------------------|----------|
| Legacy multi-DB | Postgres database name | [Multi-database](multi-database.md) |
| Registry multi-cluster | Registered alias | [Multi-database](multi-database.md) |

`prest.list_databases` returns both the alias (`name`) and `physical_name` in registry mode, so MCP clients can distinguish routing aliases from cluster database names.

When `pg.single = true` and a registry is active, only the default database alias is accepted.

---

## Safety and limits

| Rule | Detail |
|------|--------|
| Read-only | No insert, update, delete, or DDL tools in v2.1.0 |
| Row cap | Maximum **100** rows per select (`limit` defaults to 100) |
| Identifier validation | Database, schema, table, column, and filter names are validated |
| Unsupported tools | Return `400 Bad Request` with `unsupported tool` |
| Custom queries | `/_QUERIES` scripts are not exposed through MCP |
| Separate process | MCP runs in-process on `prestd`; optional [stdio adapter](../ai/install-prest-mcp.md) for clients that cannot call HTTP |

Calling a write tool (for example `prest.drop_table`) returns:

```json
{
  "error": {
    "message": "unsupported tool: prest.drop_table"
  }
}
```

---

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| `401` / `403` on `/_mcp` | Auth enabled but credentials missing or invalid — see [Auth](../api-reference/auth.md) |
| `400 unsupported tool` | Tool name not in the catalog or write operation attempted |
| `400 unsupported method` | JSON-RPC method other than `initialize`, `tools/list`, or `tools/call` |
| `400 unsupported column` | `columns`, `filters`, or `order_by` references a column not permitted or not on the table |
| Empty tool list | ACL hides all tables, or catalog is unreachable for registered aliases |
| Permission error on select | Caller lacks read access to the table or specific columns — see [Permissions](permissions.md) |
| `invalid identifier in path` | Schema or table name failed identifier validation |

For integration test examples, see [`integration/controllers/mcp_test.go`](https://github.com/prest/prest/blob/v2.1.0/integration/controllers/mcp_test.go) in the prest repository.

---

## Client tutorials

- [AI and MCP](../ai/README.md) — landing for agents and IDEs
- [Install pREST MCP Adapter](../ai/install-prest-mcp.md) — Homebrew, Go, Docker, MCP Registry
- [Use with Cursor](../ai/cursor.md) · [Claude Desktop](../ai/claude-desktop.md) · [Other AI tools](../ai/other-clients.md)
- [Read-only PostgreSQL for AI](../ai/read-only-postgres.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
- [Start with Homebrew](../get-prest/start-with-homebrew.md) — `prestd` and `prest/tap/prest-mcp`

## Related documentation

- [MCP Overview](../ai/mcp-overview.md)
- [v2.1.0 release notes](../releases/v2.1.0.md)
- [Multi-database](multi-database.md)
- [Permissions](permissions.md)
- [Auth](../api-reference/auth.md)
- [API Reference](../api-reference/README.md)
