# Install pREST MCP Adapter with Homebrew

Install `prest-mcp` from the official pREST Homebrew tap so Cursor, Claude Desktop, and other stdio MCP clients can reach PostgreSQL through pREST’s `/_mcp` endpoint.

This page focuses on the **adapter**. For the pREST server formula (`prestd` on homebrew-core), see [Start with Homebrew](../get-prest/start-with-homebrew.md).

---

## Prerequisites

* macOS (or Linux with [Homebrew](https://brew.sh/))
* pREST **v2.1.0+** running with `/_mcp` available (local is fine)

Confirm MCP discovery:

```sh
curl -s http://localhost:3000/_mcp | head
```

---

## Install

One-shot:

```sh
brew install prest/tap/prest-mcp
```

Or tap, then install:

```sh
brew tap prest/tap
brew install prest-mcp
```

Tap repository: [prest/homebrew-tap](https://github.com/prest/homebrew-tap).

---

## Verify

```sh
which prest-mcp
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

The process waits on stdin for MCP JSON-RPC. Stop with Ctrl+C after a smoke test. Logs go to stderr only.

---

## Point a client at it

Minimal env for any stdio MCP client:

| Variable | Required | Notes |
|----------|----------|-------|
| `PREST_MCP_URL` | yes | e.g. `http://localhost:3000/_mcp` |
| `PREST_MCP_TOKEN` | no | JWT when auth is enabled |
| `PREST_MCP_TIMEOUT_MS` | no | Default `30000` |

Example Cursor `.cursor/mcp.json`:

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

If the GUI app cannot find Homebrew’s `PATH`, use `/opt/homebrew/bin/prest-mcp` (Apple Silicon) or `/usr/local/bin/prest-mcp` (Intel).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Formula not found | `brew update` then `brew install prest/tap/prest-mcp` |
| Stale binary | `brew upgrade prest-mcp` or reinstall |
| Client spawn failed | Absolute path; restart the IDE |
| Connection refused | Start pREST; check `PREST_MCP_URL` |

---

## Next steps

- Full install options (Go, Docker, registry): [Install pREST MCP Adapter](../ai/install-prest-mcp.md)
- [Use with Cursor](../ai/cursor.md) · [Claude Desktop](../ai/claude-desktop.md)
- End-to-end: [PostgreSQL to AI agent](postgres-to-ai-agent.md)
- [Start with Homebrew](../get-prest/start-with-homebrew.md) · [Distribution](../get-prest/distribution.md)
