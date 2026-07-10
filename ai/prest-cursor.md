# pREST for Cursor plugin

**pREST for Cursor** ([prest/prest-cursor](https://github.com/prest/prest-cursor)) is a Cursor **plugin**: rules, skills, and examples that help the agent work with pREST. It is **not** the MCP binary.

| Component | What it is |
|-----------|------------|
| **`prest-mcp`** | Stdio adapter that talks to `/_mcp` — see [Install pREST MCP Adapter](install-prest-mcp.md) |
| **prest-cursor** | Cursor marketplace plugin (rules/skills) that guides setup and workflows |

Use both together: install the adapter for live MCP tools, and the plugin for Cursor-native guidance.

---

## What the plugin provides

* Rules for configuring and using pREST MCP safely
* Skills for common setup steps (including MCP)
* Examples such as a [read-only Postgres role](read-only-postgres.md) for AI access

Repository: [https://github.com/prest/prest-cursor](https://github.com/prest/prest-cursor)

---

## Install

Follow the install instructions in the [prest-cursor README](https://github.com/prest/prest-cursor). Typical options:

* Install from the Cursor plugin / marketplace listing when available
* Or clone / link the plugin locally for development

After install, still configure MCP separately — the plugin does not replace `prest-mcp`:

1. Run pREST **v2.1.0+** with `/_mcp`
2. Install [`prest-mcp`](install-prest-mcp.md) (`brew install prest/tap/prest-mcp`)
3. Add `.cursor/mcp.json` as in [Use with Cursor](cursor.md)

---

## MCP vs plugin checklist

| Task | Use |
|------|-----|
| Agent calls `prest.list_databases` / select tools | MCP via `prest-mcp` → `/_mcp` |
| Agent follows pREST-specific rules and skills | prest-cursor plugin |
| Least-privilege DB access | [Read-only PostgreSQL for AI](read-only-postgres.md) |

---

## Next steps

- [Use with Cursor](cursor.md)
- [MCP Overview](mcp-overview.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)

## Related documentation

- [AI and MCP landing](README.md)
- [pREST for OpenClaw plugin](prest-openclaw.md) — sibling plugin for OpenClaw
- [Install pREST MCP Adapter](install-prest-mcp.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
