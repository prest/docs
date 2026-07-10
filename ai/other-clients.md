# Other AI tools

Any MCP client that can launch a **stdio** server can connect Cursor-style to PostgreSQL through pREST. Point the client at `prest-mcp` and set `PREST_MCP_URL` to your `/_mcp` endpoint.

Requires pREST **v2.1.0+** and the [pREST MCP Adapter](install-prest-mcp.md).

---

## Generic stdio pattern

```sh
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

Or in client config JSON:

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

Optional: `PREST_MCP_TOKEN`, `PREST_MCP_TIMEOUT_MS` (default `30000`). See [Install pREST MCP Adapter](install-prest-mcp.md).

Prefer a [read-only PostgreSQL role](read-only-postgres.md) for agent access.

---

## VS Code / GitHub Copilot

VS Code uses a different MCP JSON shape (`servers` with `"type": "stdio"`, typically in `.vscode/mcp.json`). Full walkthrough: [Use with VS Code and Copilot](vscode-copilot.md).

---

## OpenClaw

Install the official plugin from [ClawHub](https://clawhub.ai/prest/plugins/prest-openclaw):

```bash
openclaw plugins install clawhub:@prest/prest-openclaw
```

Wire MCP with `prest-mcp` and `PREST_MCP_URL` as in the generic pattern above. Full guide: [pREST for OpenClaw plugin](prest-openclaw.md).

---

## Cline

In Cline’s MCP settings, add a stdio server with:

* **Command:** `prest-mcp` (or absolute path)
* **Env:** `PREST_MCP_URL=http://localhost:3000/_mcp`

Reload MCP servers after saving. Ask Cline to list databases or describe a table to confirm tools appear.

---

## Continue

Add an MCP server entry in Continue’s config (shape varies by Continue version). Typical fields:

* Command / args launching `prest-mcp`
* Environment with `PREST_MCP_URL`

If Continue cannot resolve Homebrew binaries, use `/opt/homebrew/bin/prest-mcp` or `/usr/local/bin/prest-mcp`.

---

## Windsurf

Configure Windsurf’s MCP / Cascade server list the same way as other stdio clients: command `prest-mcp`, env `PREST_MCP_URL`. Restart the IDE after changes.

---

## Docker as the command

When you do not want a host binary:

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

---

## HTTP-native clients

If your client can call MCP over HTTP directly, you may not need the adapter. Point it at `http://localhost:3000/_mcp` (or your deployed URL) and send the same auth headers as REST. Protocol details: [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Process exits immediately | Set `PREST_MCP_URL` |
| Spawn / PATH errors | Use an absolute path to `prest-mcp` |
| Empty tools | Confirm `curl` discovery; check [permissions](../get-started/permissions.md) |
| Auth errors | Set `PREST_MCP_TOKEN` |

---

## Next steps

- [Use with Cursor](cursor.md) · [VS Code and Copilot](vscode-copilot.md) · [Claude Desktop](claude-desktop.md) · [OpenClaw](prest-openclaw.md)
- [MCP Overview](mcp-overview.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
