# Install pREST MCP Adapter

Install `prest-mcp`, the official stdio adapter for pREST’s HTTP MCP endpoint. Use it when your MCP client expects a **stdio** process instead of calling `/_mcp` over HTTP directly.

Repository: [prest/prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter) · Homebrew: [prest/homebrew-tap](https://github.com/prest/homebrew-tap)

Requires pREST **v2.1.0+** with `/_mcp` available. The adapter is a transport bridge only — tools and SQL live in pREST ([MCP Overview](mcp-overview.md)).

---

## Homebrew (recommended on macOS)

```sh
brew install prest/tap/prest-mcp
```

Or tap first:

```sh
brew tap prest/tap
brew install prest-mcp
```

Step-by-step: [Install adapter with Homebrew](../tutorials/install-prest-mcp-homebrew.md). For `prestd` itself, see [Start with Homebrew](../get-prest/start-with-homebrew.md).

---

## Go

```sh
go install github.com/prest/prest-mcp-adapter/cmd/prest-mcp@latest
```

Ensure `$(go env GOPATH)/bin` is on your `PATH`.

---

## Docker / OCI

```sh
docker pull ghcr.io/prest/prest-mcp-adapter:0.1.3
```

Run against pREST on the host (Docker Desktop):

```sh
docker run --rm -i \
  -e PREST_MCP_URL=http://host.docker.internal:3000/_mcp \
  ghcr.io/prest/prest-mcp-adapter:0.1.3
```

---

## MCP Registry

Published name: **`io.github.prest/prest`**

Use this identifier when installing from the [MCP Registry](https://github.com/modelcontextprotocol/registry) or clients that resolve registry packages.

---

## Environment variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PREST_MCP_URL` | yes | — | URL of the pREST HTTP MCP endpoint, e.g. `http://localhost:3000/_mcp` |
| `PREST_MCP_TOKEN` | no | — | Bearer token when pREST auth is enabled |
| `PREST_MCP_TIMEOUT_MS` | no | `30000` | HTTP timeout in milliseconds |

---

## Local usage

```sh
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

With a JWT (when [auth](../api-reference/auth.md) is enabled):

```sh
PREST_MCP_URL=https://api.example.com/_mcp \
PREST_MCP_TOKEN=secret \
prest-mcp
```

Protocol messages are newline-delimited JSON-RPC on stdin/stdout. Logs go only to stderr.

---

## MCP client configuration

Minimal stdio server entry (Cursor, Claude Desktop, and similar clients):

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

VS Code / Copilot uses `"servers"` and `"type": "stdio"` instead — see [Use with VS Code and Copilot](vscode-copilot.md).

Add `PREST_MCP_TOKEN` when auth is enabled. If `prest-mcp` is not on `PATH`, set `"command"` to the absolute binary path (Homebrew: typically `/opt/homebrew/bin/prest-mcp` on Apple Silicon, `/usr/local/bin/prest-mcp` on Intel Mac).

Client walkthroughs:

- [Use with Cursor](cursor.md)
- [Use with VS Code and Copilot](vscode-copilot.md)
- [Use with Claude Desktop](claude-desktop.md)
- [Other AI tools](other-clients.md)

---

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Adapter exits immediately mentioning `PREST_MCP_URL` | Environment variable not set |
| Client shows server disconnected / failed to start | `prest-mcp` not on `PATH`, or wrong absolute path in config |
| `401` / `403` from tools | Auth enabled but `PREST_MCP_TOKEN` missing or invalid |
| Connection refused / timeout | pREST not running, wrong host/port, or Docker networking (`host.docker.internal`) |
| Empty tool list | ACL hides all tables — see [Permissions](../get-started/permissions.md) and [MCP over HTTP](../get-started/mcp-over-http.md) |

---

## Next steps

- [MCP Overview](mcp-overview.md)
- [Read-only PostgreSQL for AI](read-only-postgres.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)
- Protocol reference: [MCP over HTTP](../get-started/mcp-over-http.md)

## Related documentation

- [Start with Homebrew](../get-prest/start-with-homebrew.md)
- [Distribution channels](../get-prest/distribution.md)
- [v2.1.0 release notes](../releases/v2.1.0.md)
- [prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter) · [homebrew-tap](https://github.com/prest/homebrew-tap)
