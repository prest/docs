# AI and MCP with pREST

Connect AI agents and IDEs to PostgreSQL through pREST’s read-only [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) endpoint. Use the same server that already exposes a REST API from PostgreSQL — no separate query service required.

pREST v2.1.0+ ships HTTP MCP at `/_mcp`. Clients that only speak **stdio** (Cursor, VS Code / Copilot, Claude Desktop, and many others) use the official **pREST MCP Adapter** (`prest-mcp`) as a transport bridge.

---

## What you get

| Piece | Role |
|-------|------|
| **pREST `/_mcp`** | Read-only MCP tools: list databases/schemas/tables, describe columns, select rows (max 100) |
| **`prest-mcp` adapter** | Stdio ↔ HTTP bridge only — does **not** introspect schemas, generate SQL, or implement tools |
| **pREST for Cursor** | Optional [Cursor plugin](https://github.com/prest/prest-cursor) (rules and skills) — separate from the MCP binary |
| **pREST for OpenClaw** | Optional [OpenClaw plugin](https://clawhub.ai/prest/plugins/prest-openclaw) (skills and examples) — separate from the MCP binary |

```text
Cursor / VS Code / Claude / OpenClaw / Cline / Continue
        ↓ stdio
   prest-mcp
        ↓ HTTP POST
   prestd /_mcp
        ↓
   PostgreSQL
```

---

## Start here

1. [MCP Overview](mcp-overview.md) — architecture, adapter vs server, security
2. [Install pREST MCP Adapter](install-prest-mcp.md) — Homebrew, Go, Docker, MCP Registry
3. Pick a client:
   - [Use with Cursor](cursor.md)
   - [Use with VS Code and Copilot](vscode-copilot.md)
   - [Use with Claude Desktop](claude-desktop.md)
   - [Other AI tools](other-clients.md) (Cline, Continue, Windsurf, OpenClaw, generic stdio)
4. [Read-only PostgreSQL for AI](read-only-postgres.md) — least-privilege role for agent access
5. Optional agent plugins: [pREST for Cursor](prest-cursor.md) · [pREST for OpenClaw](prest-openclaw.md)

End-to-end walkthrough: [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md).

---

## Protocol reference

Deep tools, auth, ACL, and JSON-RPC details stay in [MCP over HTTP](../get-started/mcp-over-http.md). This AI section focuses on install and client setup.

---

## Next steps

- Install the adapter: [Install pREST MCP Adapter](install-prest-mcp.md) or [Homebrew tutorial](../tutorials/install-prest-mcp-homebrew.md)
- Run pREST locally: [Get pREST](../get-prest/README.md) · [Distribution channels](../get-prest/distribution.md)
- Confirm discovery: `curl -s http://localhost:3000/_mcp`
