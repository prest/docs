# Use with Claude Desktop

Connect [Claude Desktop](https://claude.ai/download) to PostgreSQL through pREST’s read-only MCP tools. Claude can discover schemas and query data via your running pREST instance.

pREST speaks MCP over HTTP at `/_mcp`. Claude Desktop launches **stdio** MCP servers, so you need the official [`prest-mcp`](install-prest-mcp.md) adapter as a bridge.

Requires pREST **v2.1.0+**.

---

## Prerequisites

* pREST running locally or on a reachable host (see [Get pREST](../get-prest/README.md))
* `prest-mcp` installed ([Homebrew](../tutorials/install-prest-mcp-homebrew.md), [Go](install-prest-mcp.md#go), or Docker)
* Claude Desktop with MCP support

Verify the HTTP endpoint before configuring Claude:

```sh
curl -s http://localhost:3000/_mcp | head
```

You should see JSON with `"name": "prest"` and a `tools` array. If auth is enabled, include your usual `Authorization` header.

Prefer a [read-only PostgreSQL role](read-only-postgres.md) for AI access.

---

## Install the adapter

```sh
brew install prest/tap/prest-mcp
```

Other options: [Install pREST MCP Adapter](install-prest-mcp.md). Confirm the binary is on your `PATH` (Claude Desktop often needs an absolute path — see below):

```sh
which prest-mcp
```

---

## Configure Claude Desktop

Edit Claude’s MCP config file:

| Platform | Path |
|----------|------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

### Basic config

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

### With authentication

When [auth](../api-reference/auth.md) is enabled on pREST:

```json
{
  "mcpServers": {
    "prest": {
      "command": "prest-mcp",
      "env": {
        "PREST_MCP_URL": "https://api.example.com/_mcp",
        "PREST_MCP_TOKEN": "your-jwt-here"
      }
    }
  }
}
```

Do not commit this file to a shared repo if it contains tokens.

### Absolute path

GUI apps on macOS often have a minimal `PATH`. Prefer an absolute command:

```json
{
  "mcpServers": {
    "prest": {
      "command": "/opt/homebrew/bin/prest-mcp",
      "env": {
        "PREST_MCP_URL": "http://localhost:3000/_mcp"
      }
    }
  }
}
```

On Intel Mac Homebrew installs, use `/usr/local/bin/prest-mcp`.

---

## Verify in Claude Desktop

1. Fully quit and reopen Claude Desktop (MCP config is loaded at startup).
2. Open a new chat and look for the MCP / tools indicator for the `prest` server.
3. Ask Claude to use pREST, for example:
   - “Using the prest MCP server, list available databases.”
   - “Describe table `public.users`.”
   - “Fetch up to 5 rows from `public.orders`.”

Claude should invoke tools such as `prest.list_databases`, `prest.describe_table`, or schema-aware `prest.select.*` tools. See [MCP over HTTP](../get-started/mcp-over-http.md) for arguments, auth, and the 100-row select limit.

---

## Optional: Docker-backed adapter

```json
{
  "mcpServers": {
    "prest": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e",
        "PREST_MCP_URL=http://host.docker.internal:3000/_mcp",
        "ghcr.io/prest/prest-mcp-adapter:0.1.3"
      ]
    }
  }
}
```

Ensure Docker Desktop is running before starting Claude.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `prest` never appears | Quit Claude completely and reopen; check JSON syntax in the config file |
| Server error / spawn failed | Use an absolute path to `prest-mcp`; GUI apps may not see Homebrew’s `PATH` |
| Auth failures | Set `PREST_MCP_TOKEN` |
| Connection refused | Start pREST; confirm `PREST_MCP_URL` |
| Empty tools | Check [permissions](../get-started/permissions.md) and `curl` discovery on `/_mcp` |

Logs from the adapter go to stderr; Claude Desktop surfaces MCP server errors in its developer / MCP UI depending on version.

More detail: [Install pREST MCP Adapter](install-prest-mcp.md) and [MCP over HTTP](../get-started/mcp-over-http.md#troubleshooting).

---

## Next steps

- [Read-only PostgreSQL for AI](read-only-postgres.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
- [Use with Cursor](cursor.md) · [Use with VS Code and Copilot](vscode-copilot.md)
- [Other AI tools](other-clients.md)

## Related documentation

- [MCP Overview](mcp-overview.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Permissions](../get-started/permissions.md)
- [Auth](../api-reference/auth.md)
