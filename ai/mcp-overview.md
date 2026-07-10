# MCP Overview

pREST exposes a **PostgreSQL MCP server** over HTTP so AI agents can discover schemas and read data through the same process that serves your REST API.

Requires **pREST v2.1.0+**.

---

## Two layers

| Layer | What it is | What it does |
|-------|------------|--------------|
| **HTTP MCP endpoint** | `/_mcp` on `prestd` | Implements MCP tools, catalog discovery, and read queries |
| **Stdio adapter** | `prest-mcp` binary | Forwards JSON-RPC between stdin/stdout and `POST /_mcp` |

The adapter is a **transport bridge only**. It does not introspect schemas, generate SQL, or implement tools — those live in pREST.

```text
MCP client / AI IDE
        ↓ stdio (NDJSON JSON-RPC)
prest-mcp
        ↓ HTTP POST
http://localhost:3000/_mcp
        ↓
pREST → PostgreSQL
```

| Concern | Native `/_mcp` | `prest-mcp` adapter |
|---------|----------------|---------------------|
| Transport | HTTP on the pREST server | Stdio process that proxies to HTTP |
| Tools / SQL | Implemented in pREST | None — forwards JSON-RPC as-is |
| When to use | HTTP-capable MCP clients | Clients that only launch stdio servers |
| Extra process | No | Yes — one lightweight binary |

---

## Security defaults for AI

- Prefer a **read-only** PostgreSQL role for agent workflows — see [Read-only PostgreSQL for AI](read-only-postgres.md).
- MCP tools in v2.1.0 are **read-only** (no insert/update/delete/DDL). REST may still allow writes on the same server.
- Do not expose unauthenticated `/_mcp` publicly.
- Prefer local or private-network pREST for development.
- When [auth](../api-reference/auth.md) is enabled, pass a JWT via `PREST_MCP_TOKEN` (adapter) or `Authorization` (HTTP).

---

## Environment variables (adapter)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PREST_MCP_URL` | yes | — | URL of the HTTP MCP endpoint, e.g. `http://localhost:3000/_mcp` |
| `PREST_MCP_TOKEN` | no | — | Bearer token when pREST auth is enabled |
| `PREST_MCP_TIMEOUT_MS` | no | `30000` | HTTP timeout in milliseconds |

---

## Next steps

- [Install pREST MCP Adapter](install-prest-mcp.md)
- [Use with Cursor](cursor.md) · [VS Code and Copilot](vscode-copilot.md) · [Claude Desktop](claude-desktop.md) · [Other AI tools](other-clients.md)
- Protocol and tools reference: [MCP over HTTP](../get-started/mcp-over-http.md)

## Related documentation

- [AI and MCP landing](README.md)
- [pREST for Cursor plugin](prest-cursor.md)
- [pREST for OpenClaw plugin](prest-openclaw.md)
- [v2.1.0 release notes](../releases/v2.1.0.md)
