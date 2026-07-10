# pREST for OpenClaw plugin

**pREST for OpenClaw** ([prest/prest-openclaw](https://github.com/prest/prest-openclaw)) is an OpenClaw **plugin** on [ClawHub](https://clawhub.ai/prest/plugins/prest-openclaw) (`@prest/prest-openclaw`). It provides skills, examples, and security guidance for building PostgreSQL-backed APIs with pREST. It is **not** the MCP binary and does **not** run pREST or PostgreSQL.

| Component | What it is |
|-----------|------------|
| **`prest-mcp`** | Stdio adapter that talks to `/_mcp` — see [Install pREST MCP Adapter](install-prest-mcp.md) |
| **prest-openclaw** | OpenClaw plugin (skills + examples) that guides config, SQL routes, security review, and MCP setup |

Use both together: install the adapter for live MCP tools, and the plugin for OpenClaw-native guidance.

---

## What the plugin provides

| Skill | Purpose |
|-------|---------|
| `generate-config` | Generate `prest.toml`, `.env.example`, optional Docker Compose |
| `generate-sql-route` | Parameterized SQL under `/_QUERIES/` |
| `review-security` | Audit configs, SQL, auth, and MCP exposure |
| `setup-mcp` | Wire `prest-mcp` and read-only Postgres for agents |
| `explain-config` | Plain-language `prest.toml` walkthrough |
| `prest-core-guidance` | Core pREST development rules |
| `prest-security-guidance` | Security defaults and warnings |
| `prest-mcp-guidance` | MCP and `prest-mcp` connection patterns |
| `agent-prompts` | Copy-paste prompts for common tasks |

Examples include Docker Compose setups, SQL template routes, multi-database configs, and a read-only MCP workflow (`examples/mcp-readonly/`).

Repository: [https://github.com/prest/prest-openclaw](https://github.com/prest/prest-openclaw)

---

## Install from ClawHub

```bash
openclaw plugins install clawhub:@prest/prest-openclaw
openclaw plugins inspect prest
```

Enable the plugin in your OpenClaw config if needed, then restart the gateway:

```bash
openclaw gateway restart
```

Listing: [clawhub.ai/prest/plugins/prest-openclaw](https://clawhub.ai/prest/plugins/prest-openclaw)

---

## MCP setup

The plugin does not replace `prest-mcp`. After installing the plugin:

1. Run pREST **v2.1.0+** with `/_mcp` available
2. Install the adapter: `brew install prest/tap/prest-mcp` — see [Install pREST MCP Adapter](install-prest-mcp.md)
3. Wire MCP in OpenClaw using `prest-mcp` and env vars

```bash
export PREST_MCP_URL='http://localhost:3000/_mcp'
# export PREST_MCP_TOKEN='...'   # when auth is required
prest-mcp
```

Example MCP config shape (from the plugin repo’s `examples/mcp-readonly/openclaw-mcp.example.json`):

```json
{
  "mcpServers": {
    "prest": {
      "command": "prest-mcp",
      "env": {
        "PREST_MCP_URL": "${PREST_MCP_URL}",
        "PREST_MCP_TOKEN": "${PREST_MCP_TOKEN}"
      }
    }
  }
}
```

| Variable | Required | Description |
|----------|----------|-------------|
| `PREST_MCP_URL` | Yes | Full MCP URL, e.g. `http://localhost:3000/_mcp` |
| `PREST_MCP_TOKEN` | No | Bearer token when pREST auth is enabled |
| `PREST_MCP_TIMEOUT_MS` | No | HTTP timeout override (default `30000`) |

MCP Registry name: `io.github.prest/prest`

---

## Plugin vs MCP checklist

| Task | Use |
|------|-----|
| Agent calls `prest.list_databases` / select tools | MCP via `prest-mcp` → `/_mcp` |
| Agent follows pREST skills (config, SQL routes, security) | prest-openclaw plugin |
| Least-privilege DB access | [Read-only PostgreSQL for AI](read-only-postgres.md) |

---

## Security

- Never expose `/_mcp`, Postgres, or pREST publicly without authentication
- Prefer a [read-only PostgreSQL role](read-only-postgres.md) for agent workflows
- Use environment variables for secrets — never commit real credentials
- Set `access.restrict = true` with explicit `[[access.tables]]` in production
- Do not use write-capable database roles in MCP configs

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Skills not appearing | Confirm plugin is enabled; restart gateway; run `openclaw plugins inspect prest` |
| MCP connection fails | `curl http://localhost:3000/_health`; verify `PREST_MCP_URL` ends with `/_mcp`; set `PREST_MCP_TOKEN` if auth is on |
| `prest-mcp` not found | `brew install prest/tap/prest-mcp` or use absolute path |

Publishing and validation issues: see the [plugin README](https://github.com/prest/prest-openclaw) and [OpenClaw ClawHub docs](https://docs.openclaw.ai/clawhub).

---

## Next steps

- [Install pREST MCP Adapter](install-prest-mcp.md)
- [MCP Overview](mcp-overview.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)

## Related documentation

- [AI and MCP landing](README.md)
- [pREST for Cursor plugin](prest-cursor.md) — sibling plugin for Cursor
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Distribution channels](../get-prest/distribution.md)
