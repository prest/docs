# Get pREST

There are several ways of downloading pREST, some are listed below. If it is your first time, we recommend following [Deploying with Docker](../deployment/deploying-with-docker.md). Compare channels in [Distribution](distribution.md).

* Use the Docker Hub image
* Use the GitHub image
* [Homebrew](start-with-homebrew.md) — `prestd` (homebrew-core) and `prest-mcp` ([prest/tap](https://github.com/prest/homebrew-tap))
* Download from [source](start-with-golang.md) and run
* Download the latest stable v1 binary from [releases/latest](https://github.com/prest/prest/releases/latest)
* [Distribution channels](distribution.md) — which method for server vs MCP adapter

## v2.1.0

**v2.1.0** is the latest stable v2 release. It adds native MCP over HTTP at `/_mcp`. See the [release notes](../releases/v2.1.0.md), [MCP over HTTP](../get-started/mcp-over-http.md), and [AI and MCP](../ai/README.md) for Cursor and Claude Desktop.

* **Binary:** [v2.1.0 assets](https://github.com/prest/prest/releases/tag/v2.1.0)
* **Docker:** `prest/prest:v2.1.0`
* **MCP adapter:** `brew install prest/tap/prest-mcp` or see [Install pREST MCP Adapter](../ai/install-prest-mcp.md)

## v2.0.0

**v2.0.0** includes multi-database support, config resilience, JWT auto-disable, OR clause filtering, structured JSON logging, and per-user permissions. See the [release notes](../releases/v2.0.0.md) and [upgrade guide](../get-started/upgrading-to-v2.md).

* **Binary:** [v2.0.0 assets](https://github.com/prest/prest/releases/tag/v2.0.0)
* **Docker:** `prest/prest:v2.0.0`
