# pREST MCP Adapter

Official stdio adapter for pREST’s HTTP MCP endpoint. Use it when your MCP client (Cursor, Claude Desktop, and many others) expects a **stdio** process instead of calling `/_mcp` over HTTP directly.

The adapter is a tiny transport bridge. It does **not** introspect schemas, generate SQL, or implement MCP tools — those live in pREST ([MCP over HTTP](mcp-over-http.md)).

Repository: [prest/prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter) · Homebrew: [prest/homebrew-tap](https://github.com/prest/homebrew-tap)

Requires pREST **v2.1.0+** with `/_mcp` available.

---

## How it fits

```text
MCP client / AI IDE
        ↓ stdio (NDJSON JSON-RPC)
prest-mcp
        ↓ HTTP POST
http://localhost:3000/_mcp
        ↓
pREST
```

| Concern | Native `/_mcp` | `prest-mcp` adapter |
|---------|----------------|---------------------|
| Transport | HTTP on the pREST server | Stdio process that proxies to HTTP |
| Tools / SQL | Implemented in pREST | None — forwards JSON-RPC as-is |
| When to use | HTTP-capable MCP clients | Clients that only launch stdio servers |
| Extra process | No | Yes — one lightweight binary |

---

## Install

### Homebrew (recommended on macOS)

```sh
brew install prest/tap/prest-mcp
```

Or tap first:

```sh
brew tap prest/tap
brew install prest-mcp
```

See [Start with Homebrew](../get-prest/start-with-homebrew.md) for `prestd` (homebrew-core) and the custom tap.

### Go

```sh
go install github.com/prest/prest-mcp-adapter/cmd/prest-mcp@latest
```

### Docker / OCI

```sh
docker pull ghcr.io/prest/prest-mcp-adapter:0.1.3
```

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

### Docker

```sh
docker run --rm -i \
  -e PREST_MCP_URL=http://host.docker.internal:3000/_mcp \
  ghcr.io/prest/prest-mcp-adapter:0.1.3
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

Add `PREST_MCP_TOKEN` when [auth](../api-reference/auth.md) is enabled. If `prest-mcp` is not on `PATH`, set `"command"` to the absolute binary path (Homebrew: typically `/opt/homebrew/bin/prest-mcp` on Apple Silicon, `/usr/local/bin/prest-mcp` on Intel Mac).

End-to-end walkthroughs:

- [Connect Cursor to pREST MCP](mcp-with-cursor.md)
- [Connect Claude Desktop to pREST MCP](mcp-with-claude-desktop.md)

### Generic stdio client

Any client that can launch a stdio MCP server:

```sh
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

---

## Security

- Prefer a **read-only** PostgreSQL user for AI/MCP workflows unless write access is explicitly required (MCP itself is read-only in v2.1.0; REST may still allow writes).
- Do not expose unauthenticated `/_mcp` endpoints publicly.
- Prefer local or private-network pREST instances.
- When auth is enabled, set `PREST_MCP_TOKEN` to a valid JWT.

---

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Adapter exits immediately mentioning `PREST_MCP_URL` | Environment variable not set |
| Client shows server disconnected / failed to start | `prest-mcp` not on `PATH`, or wrong absolute path in config |
| `401` / `403` from tools | Auth enabled but `PREST_MCP_TOKEN` missing or invalid |
| Connection refused / timeout | pREST not running, wrong host/port, or Docker networking (`host.docker.internal`) |
| Empty tool list | ACL hides all tables — see [Permissions](permissions.md) and [MCP over HTTP](mcp-over-http.md) |

---

## Related documentation

- [MCP over HTTP](mcp-over-http.md) — server endpoint, tools, auth, limits
- [Connect Cursor to pREST MCP](mcp-with-cursor.md)
- [Connect Claude Desktop to pREST MCP](mcp-with-claude-desktop.md)
- [Start with Homebrew](../get-prest/start-with-homebrew.md)
- [v2.1.0 release notes](../releases/v2.1.0.md)
- [prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter) · [homebrew-tap](https://github.com/prest/homebrew-tap)
