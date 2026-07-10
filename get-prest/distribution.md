# Distribution channels

Ways to install **pREST** (`prestd`) and the **pREST MCP Adapter** (`prest-mcp`). Pick the channel that matches your platform and workflow.

Docs site: [https://docs.prestd.com/](https://docs.prestd.com/)

---

## Which method?

| Goal | Recommended |
|------|-------------|
| Try pREST quickly | [Docker](../deployment/deploying-with-docker.md) |
| macOS daily driver for `prestd` | [Homebrew core](start-with-homebrew.md) (`brew install prestd`) |
| Connect Cursor / Claude to MCP | [Homebrew tap](start-with-homebrew.md) (`brew install prest/tap/prest-mcp`) or [Go](../ai/install-prest-mcp.md#go) |
| Pin a release binary | [GitHub Releases](https://github.com/prest/prest/releases) |
| Build from source | [Start with Golang](start-with-golang.md) |
| MCP via OCI | `ghcr.io/prest/prest-mcp-adapter:0.1.3` |

MCP requires **pREST v2.1.0+**. Overview: [AI and MCP](../ai/README.md).

---

## pREST server (`prestd`)

| Channel | How |
|---------|-----|
| Docker | `prest/prest:v2.1.0` — [Deploying with Docker](../deployment/deploying-with-docker.md) |
| Homebrew | `brew install prestd` — [Start with Homebrew](start-with-homebrew.md) |
| Go | [Start with Golang](start-with-golang.md) |
| Binaries | [GitHub Releases](https://github.com/prest/prest/releases/tag/v2.1.0) |

---

## pREST MCP Adapter (`prest-mcp`)

Stdio ↔ HTTP bridge for `/_mcp`. Transport only — no schema/SQL/tools in the adapter.

| Channel | How |
|---------|-----|
| Homebrew | `brew install prest/tap/prest-mcp` — [Homebrew tutorial](../tutorials/install-prest-mcp-homebrew.md) |
| Go | `go install github.com/prest/prest-mcp-adapter/cmd/prest-mcp@latest` |
| Docker / OCI | `docker pull ghcr.io/prest/prest-mcp-adapter:0.1.3` |
| MCP Registry | Package name `io.github.prest/prest` |

Full guide: [Install pREST MCP Adapter](../ai/install-prest-mcp.md).

---

## Related documentation

- [Get pREST](README.md)
- [Start with Homebrew](start-with-homebrew.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [v2.1.0 release notes](../releases/v2.1.0.md)
