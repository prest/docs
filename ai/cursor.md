# Use with Cursor

Connect [Cursor](https://cursor.com/) to PostgreSQL through pREST’s read-only MCP tools. The agent can discover schemas and query data via your running pREST instance.

pREST speaks MCP over HTTP at `/_mcp`. Cursor launches **stdio** MCP servers, so you need the official [`prest-mcp`](install-prest-mcp.md) adapter as a bridge.

Requires pREST **v2.1.0+**.

For Cursor rules and skills (separate from MCP), see [pREST for Cursor plugin](prest-cursor.md).

---

## Prerequisites

* pREST running locally or on a reachable host (see [Get pREST](../get-prest/README.md))
* `prest-mcp` installed ([Homebrew](../tutorials/install-prest-mcp-homebrew.md), [Go](install-prest-mcp.md#go), or Docker)
* Cursor with MCP support enabled

Verify the HTTP endpoint before configuring Cursor:

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

Other options: [Install pREST MCP Adapter](install-prest-mcp.md). Confirm the binary is on your `PATH`:

```sh
which prest-mcp
```

---

## Configure Cursor

### Project config (recommended)

Create or edit `.cursor/mcp.json` in your project root:

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

Prefer injecting the token from your environment or a secret store rather than committing JWTs to git. If you must keep secrets out of the repo, use Cursor’s user-level MCP settings instead of the project file.

### Absolute path

If Cursor cannot find `prest-mcp` on `PATH`:

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

On Intel Mac Homebrew installs, the path is often `/usr/local/bin/prest-mcp`.

---

## Verify in Cursor

1. Restart Cursor or reload MCP servers from Settings → MCP.
2. Confirm the `prest` server shows as connected.
3. Ask the agent something that needs catalog access, for example:
   - “List databases available through pREST.”
   - “Describe the `public.users` table.”
   - “Select the first 5 rows from `public.orders` ordered by `id`.”

The agent should call tools such as `prest.list_databases`, `prest.describe_table`, or `prest.select.{database}.{schema}.{table}`. Tool behavior and limits are documented in [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Optional: Docker-backed adapter

If you prefer not to install a local binary:

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

`host.docker.internal` reaches pREST on the host from Docker Desktop (macOS/Windows). On Linux you may need `--add-host=host.docker.internal:host-gateway` or the host’s LAN IP.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Server fails to start | Ensure `prest-mcp` is installed and the `command` path is correct |
| Tools never appear | Confirm `curl` to `/_mcp` works; check ACL / empty catalog |
| Auth errors | Set `PREST_MCP_TOKEN` to a valid JWT |
| Connection refused | Start pREST; match host/port in `PREST_MCP_URL` |
| Stale tools after schema change | Restart pREST (schema-aware tools are generated at startup) |

More detail: [Install pREST MCP Adapter](install-prest-mcp.md) and [MCP over HTTP](../get-started/mcp-over-http.md#troubleshooting).

---

## Next steps

- [pREST for Cursor plugin](prest-cursor.md) — rules and skills for pREST workflows
- [Read-only PostgreSQL for AI](read-only-postgres.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
- [Use with VS Code and Copilot](vscode-copilot.md) · [Use with Claude Desktop](claude-desktop.md)

## Related documentation

- [MCP Overview](mcp-overview.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Permissions](../get-started/permissions.md)
- [Auth](../api-reference/auth.md)
