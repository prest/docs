# Start with Homebrew

### Prerequisites

* macOS (or Linux with [Homebrew](https://brew.sh/))
* [Homebrew](https://brew.sh/)

***

## Install prestd (homebrew-core)

Install the pREST server from the official Homebrew core formula:

```sh
brew install prestd
```

Formula: [formulae.brew.sh/formula/prestd](https://formulae.brew.sh/formula/prestd) [![Homebrew](https://img.shields.io/badge/dynamic/json.svg?url=https://formulae.brew.sh/api/formula/prestd.json\&query=$.versions.stable\&label=homebrew)](https://formulae.brew.sh/formula/prestd)

For MCP features you need **prestd v2.1.0+**. Check what Homebrew installed:

```sh
brew info prestd
```

If your formula is older than v2.1.0, upgrade with `brew upgrade prestd` or install from [GitHub Releases](https://github.com/prest/prest/releases) / [Docker](../deployment/deploying-with-docker.md) until the core formula catches up.

***

## Install prest-mcp (pREST Homebrew tap)

Clients such as Cursor and Claude Desktop speak MCP over **stdio**. pREST exposes MCP over **HTTP** at `/_mcp`. The official adapter bridges the two.

Formulae that are not in homebrew-core ship from the [prest/homebrew-tap](https://github.com/prest/homebrew-tap) repository:

| Formula | Package | Description |
|---------|---------|-------------|
| `prest-mcp` | [prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter) | Stdio MCP adapter for pREST’s `/_mcp` endpoint |

### Install

```sh
brew install prest/tap/prest-mcp
```

Or tap once, then install:

```sh
brew tap prest/tap
brew install prest-mcp
```

### Quick check

```sh
PREST_MCP_URL=http://localhost:3000/_mcp prest-mcp
```

Point MCP clients at this binary — see [pREST MCP Adapter](../get-started/prest-mcp-adapter.md), [Cursor](../get-started/mcp-with-cursor.md), and [Claude Desktop](../get-started/mcp-with-claude-desktop.md).

***

## Next steps

1. Configure and run **prestd** against your Postgres database — [Configuring pREST](../get-started/configuring-prest.md).
2. Confirm MCP discovery: `curl -s http://localhost:3000/_mcp`.
3. Connect your IDE with the adapter — [Cursor](../get-started/mcp-with-cursor.md) or [Claude Desktop](../get-started/mcp-with-claude-desktop.md).

## Related documentation

- [pREST MCP Adapter](../get-started/prest-mcp-adapter.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Get pREST](README.md)
- [homebrew-tap](https://github.com/prest/homebrew-tap) · [prest-mcp-adapter](https://github.com/prest/prest-mcp-adapter)
