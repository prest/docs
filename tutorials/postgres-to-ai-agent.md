# PostgreSQL to AI agent

Connect an AI agent (Cursor, VS Code / Copilot, Claude Desktop, or any stdio MCP client) to PostgreSQL using pREST: a REST API from PostgreSQL plus a read-only **PostgreSQL MCP server** on the same process.

This tutorial is local-first. You will:

1. Run pREST against Postgres
2. Prefer a read-only database role
3. Install the `prest-mcp` adapter
4. Wire an AI client to MCP tools

Requires **pREST v2.1.0+**.

---

## Architecture

```text
AI client (Cursor / VS Code / Claude / …)
        ↓ stdio
   prest-mcp
        ↓ HTTP POST /_mcp
   prestd
        ↓
   PostgreSQL (read-only role recommended)
```

The adapter only bridges transports. Schema discovery and queries are implemented in pREST — see [MCP Overview](../ai/mcp-overview.md).

---

## 1. Run PostgreSQL and pREST

Start Postgres locally (Docker, Homebrew, or your existing instance). Then install and run pREST — pick a channel from [Distribution](../get-prest/distribution.md) or [Get pREST](../get-prest/README.md).

Example with Homebrew for the server:

```sh
brew install prestd
```

Configure connection settings ([Configuring pREST](../get-started/configuring-prest.md)) and start `prestd`. Confirm the REST API and MCP discovery:

```sh
curl -s http://localhost:3000/_mcp | head
```

You should see `"name": "prest"` and a `tools` array.

---

## 2. Use a read-only role

Create a least-privilege role with `CONNECT`, `USAGE`, and `SELECT` only — full SQL: [Read-only PostgreSQL for AI](../ai/read-only-postgres.md). Point pREST’s Postgres credentials at that role for AI-facing instances.

---

## 3. Install the MCP adapter

On macOS:

```sh
brew install prest/tap/prest-mcp
```

Or:

```sh
go install github.com/prest/prest-mcp-adapter/cmd/prest-mcp@latest
```

Details: [Install pREST MCP Adapter](../ai/install-prest-mcp.md) · [Homebrew tutorial](install-prest-mcp-homebrew.md).

Quick check:

```sh
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

(Leave it running only if you are testing manually; MCP clients spawn it themselves.)

---

## 4. Connect your AI client

### Cursor

Create `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "prest": {
      "command": "prest-mcp",
      "env": {
        "PREST_MCP_URL": "http://localhost:3000/_mcp"
      }
    }
  }
}
```

Full guide: [Use with Cursor](../ai/cursor.md). Optional rules/skills: [pREST for Cursor plugin](../ai/prest-cursor.md).

### VS Code / GitHub Copilot

Create `.vscode/mcp.json` with a `servers` entry (`"type": "stdio"`, command `prest-mcp`, env `PREST_MCP_URL`). Use Copilot **Agent** mode to verify. Guide: [Use with VS Code and Copilot](../ai/vscode-copilot.md).

### Claude Desktop

Edit `claude_desktop_config.json` with the same `command` / `env` pattern (prefer an absolute path on macOS). Guide: [Use with Claude Desktop](../ai/claude-desktop.md).

### OpenClaw

```bash
openclaw plugins install clawhub:@prest/prest-openclaw
```

Configure MCP with `prest-mcp` and `PREST_MCP_URL` (see plugin `examples/mcp-readonly/openclaw-mcp.example.json`). Plugin guide: [pREST for OpenClaw](../ai/prest-openclaw.md).

### Other tools

Cline, Continue, Windsurf, and generic stdio clients: [Other AI tools](../ai/other-clients.md).

Registry package name: `io.github.prest/prest`.

---

## 5. Try it

Ask the agent:

* “List databases available through pREST.”
* “Describe `public.users`.”
* “Select 5 rows from `public.users`.”

Expected tools include `prest.list_databases`, `prest.describe_table`, and `prest.select.*`. Limits and auth: [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| No `/_mcp` response | Upgrade to pREST v2.1.0+; confirm host/port |
| Adapter exits on start | Set `PREST_MCP_URL` |
| Client cannot spawn binary | Absolute path to `prest-mcp`; `brew install prest/tap/prest-mcp` |
| Empty tools | Grants / [permissions](../get-started/permissions.md); restart pREST after schema changes |
| Auth errors | Set `PREST_MCP_TOKEN` when JWT auth is enabled |

---

## Next steps

- [AI and MCP landing](../ai/README.md)
- [MCP Overview](../ai/mcp-overview.md)
- [Install adapter with Homebrew](install-prest-mcp-homebrew.md)
- [Distribution channels](../get-prest/distribution.md)
