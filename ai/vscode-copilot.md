# Use with VS Code and Copilot

Connect [GitHub Copilot](https://code.visualstudio.com/docs/copilot/overview) in [Visual Studio Code](https://code.visualstudio.com/) to PostgreSQL through pREST’s read-only MCP tools. In **Agent** mode, Copilot can discover schemas and query data via your running pREST instance.

pREST speaks MCP over HTTP at `/_mcp`. VS Code launches **stdio** MCP servers, so you need the official [`prest-mcp`](install-prest-mcp.md) adapter as a bridge.

Requires pREST **v2.1.0+**.

Official VS Code MCP docs: [Add and manage MCP servers](https://code.visualstudio.com/docs/agent-customization/mcp-servers).

The [pREST for Cursor plugin](prest-cursor.md) does **not** apply to VS Code. You still get full MCP tools through `prest-mcp`; only Cursor-specific rules and skills are unavailable here.

---

## Prerequisites

* pREST running locally or on a reachable host (see [Get pREST](../get-prest/README.md))
* `prest-mcp` installed ([Homebrew](../tutorials/install-prest-mcp-homebrew.md), [Go](install-prest-mcp.md#go), or Docker)
* VS Code with [GitHub Copilot Chat](https://code.visualstudio.com/docs/copilot/copilot-chat) and **Agent** mode available

Verify the HTTP endpoint before configuring VS Code:

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

## Configure VS Code

VS Code MCP config uses a top-level **`servers`** key (not `mcpServers`). Each server entry includes `"type": "stdio"`.

### Workspace config (recommended)

Create or edit `.vscode/mcp.json` in your project root:

```json
{
  "servers": {
    "prest": {
      "type": "stdio",
      "command": "prest-mcp",
      "env": {
        "PREST_MCP_URL": "http://localhost:3000/_mcp"
      }
    }
  }
}
```

### User config

For a global setup that applies across workspaces, run **MCP: Open User Configuration** from the Command Palette and add the same `servers.prest` block.

### With authentication

When [auth](../api-reference/auth.md) is enabled on pREST:

```json
{
  "servers": {
    "prest": {
      "type": "stdio",
      "command": "prest-mcp",
      "env": {
        "PREST_MCP_URL": "https://api.example.com/_mcp",
        "PREST_MCP_TOKEN": "your-jwt-here"
      }
    }
  }
}
```

Prefer injecting the token from your environment or a secret store rather than committing JWTs to git. Use user-level MCP configuration when you need secrets out of the repo.

### Absolute path

If VS Code cannot find `prest-mcp` on `PATH`:

```json
{
  "servers": {
    "prest": {
      "type": "stdio",
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

## Optional: MCP gallery / registry

VS Code can install servers from the MCP gallery or registry. The published package name is **`io.github.prest/prest`**.

After installing from the gallery, set `PREST_MCP_URL` (and `PREST_MCP_TOKEN` if needed) in the server’s environment so the adapter can reach your pREST instance. Details: [Install pREST MCP Adapter](install-prest-mcp.md#mcp-registry).

---

## Verify in Copilot Agent mode

1. Start or restart the `prest` MCP server from the VS Code MCP UI (or reload the window).
2. Open Copilot Chat and switch to **Agent** mode.
3. Confirm the `prest` tools are available to the agent.
4. Ask something that needs catalog access, for example:
   - “List databases available through pREST.”
   - “Describe the `public.users` table.”
   - “Select the first 5 rows from `public.orders` ordered by `id`.”

The agent should call tools such as `prest.list_databases`, `prest.describe_table`, or `prest.select.{database}.{schema}.{table}`. Tool behavior and limits are documented in [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Optional: Docker-backed adapter

If you prefer not to install a local binary:

```json
{
  "servers": {
    "prest": {
      "type": "stdio",
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
| Config ignored / wrong shape | Use `"servers"` (not `mcpServers`) and include `"type": "stdio"` |
| Tools never appear | Confirm `curl` to `/_mcp` works; check ACL / empty catalog; use **Agent** mode |
| Auth errors | Set `PREST_MCP_TOKEN` to a valid JWT |
| Connection refused | Start pREST; match host/port in `PREST_MCP_URL` |
| Stale tools after schema change | Restart pREST (schema-aware tools are generated at startup) |

More detail: [Install pREST MCP Adapter](install-prest-mcp.md) and [MCP over HTTP](../get-started/mcp-over-http.md#troubleshooting). VS Code reference: [MCP servers in VS Code](https://code.visualstudio.com/docs/agent-customization/mcp-servers).

---

## Next steps

- [Read-only PostgreSQL for AI](read-only-postgres.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
- [Use with Cursor](cursor.md) · [Use with Claude Desktop](claude-desktop.md)
- [Other AI tools](other-clients.md)

## Related documentation

- [MCP Overview](mcp-overview.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Permissions](../get-started/permissions.md)
- [Auth](../api-reference/auth.md)
- [VS Code MCP servers](https://code.visualstudio.com/docs/agent-customization/mcp-servers)
