---
description: >-
  Download and run pREST — Docker, Homebrew, or Go — for instant REST and MCP
  APIs on SQL databases (PostgreSQL native today).
---

# Get pREST

Download and run pREST to get instant REST and MCP APIs for your SQL database.
PostgreSQL is the native adapter today — see [Databases](../databases/README.md).
If this is your first install, start with [Deploying with Docker](../deployment/deploying-with-docker.md). Compare channels in [Distribution](distribution.md).

* Use the Docker Hub image
* Use the GitHub image
* [Homebrew](start-with-homebrew.md) — `prestd` (homebrew-core) and `prest-mcp` ([prest/tap](https://github.com/prest/homebrew-tap))
* Download from [source](start-with-golang.md) and run
* GitHub [releases](https://github.com/prest/prest/releases)
* [Distribution channels](distribution.md) — which method for server vs MCP adapter

## Latest: v2.1.0

**[v2.1.0](../releases/v2.1.0.md)** adds MCP over HTTP plus everything in v2.0.0 (multi-database, config resilience, JWT auto-disable, OR filtering, permissions). See the [upgrade guide](../get-started/upgrading-to-v2.md), [MCP over HTTP](../get-started/mcp-over-http.md), and [AI and MCP](../ai/README.md) for Cursor and Claude Desktop.

* **Binary:** [v2.1.0 assets](https://github.com/prest/prest/releases/tag/v2.1.0)
* **Docker:** `prest/prest:v2.1.0`
* **Go:** `go install github.com/prest/prest/v2/cmd/prestd@v2.1.0`
* **MCP adapter:** `brew install prest/tap/prest-mcp` or see [Install pREST MCP Adapter](../ai/install-prest-mcp.md)

Database support labels: [Databases](../databases/README.md).
