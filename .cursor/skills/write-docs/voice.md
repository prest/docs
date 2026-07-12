# Voice reference — pREST docs

## Positioning

**Line (canonical):** Instant REST and MCP APIs for SQL databases. PostgreSQL-first, now multi-database.

**Category (when needed):** open-source REST and MCP APIs for SQL databases

**“open-source” is encouraged** on definitional, category, license, and community sentences. Use it where it adds meaning — do not strip it. Avoid only the awkward **gateway** label and `(PostgreSQL-first)` parentheticals on every lead.

## Acronyms (reference-first)

**Canonical glossary:** [`prestd/acronyms.md`](../../../prestd/acronyms.md) — short definitions with anchors (`#rest`, `#mcp`, `#ai`, …).

**Default:** Use the bare acronym in body copy. Do **not** expand every acronym on every page.

**Always when glossary terms appear in prose:** Add end-of-page **Related** links to the glossary — at least `[Acronyms](…/prestd/acronyms.md)`, and when practical anchors for the main terms that page uses (e.g. `#mcp`, `#rest`).

**Expand inline only for SEO / discovery** — homepage lead, FAQ (“What is …?”), canonical “What is …?” sections, and other answer-first hubs where a reader or answer engine would treat *this* page as a definition. Decision test: *Would “what is \<acronym\> + pREST” reasonably land here?* If yes → expand once (use approved one-liners below for REST / MCP / AI). If no → bare term + Related to the glossary.

**Do not expand for SEO by default on** narrow how-tos, connection snippets, release changelogs, or pure procedure pages — Reference is enough.

**Exceptions (no Related required):** acronyms only inside code fences, URLs, env vars, paths, or JSON keys.

Do **not** expand REST inside the product name **pREST**.

### Approved expansion copy (when SEO warrants it)

#### REST

> **REST (Representational State Transfer)** is an architectural style for building APIs over HTTP. pREST exposes your SQL tables as REST resources at `/{database}/{schema}/{table}`, mapping standard verbs to CRUD on the same server as MCP.

Canonical “What is REST?”: [API Reference](../../../api-reference/README.md). Glossary: [Acronyms § REST](../../../prestd/acronyms.md#rest).

#### MCP

> **MCP (Model Context Protocol)** is an open standard for connecting AI apps and agents to tools and data. pREST exposes a read-only MCP endpoint at `/_mcp` so clients can discover schemas and query tables through the same server as the REST API.

Canonical “What is MCP?”: [MCP over HTTP](../../../get-started/mcp-over-http.md). Glossary: [Acronyms § MCP](../../../prestd/acronyms.md#mcp).

#### AI

> **AI (artificial intelligence)** here means applications and agents that use models to reason and act on tools and data. pREST connects them to your SQL catalog through read-only MCP at `/_mcp` and the same auth/ACL as the REST API.

Do **not** claim pREST is an LLM or “AI product” beyond API access for AI clients. Canonical “What is AI (in these docs)?”: [MCP over HTTP](../../../get-started/mcp-over-http.md). Glossary: [Acronyms § AI](../../../prestd/acronyms.md#ai). Client setup: https://docs.prestd.com/ai.

## Page-lead pattern

1. Reader’s job (Install / Configure / Connect X / Which databases…)  
2. One benefit or outcome sentence (may include “open-source”)  
3. Link to Databases or roadmap when scope matters  

### Bad vs good

| Bad | Good |
|-----|------|
| Install **pREST**, an open-source REST and MCP gateway for SQL databases (PostgreSQL-first). | Download and run pREST — open-source instant REST and MCP APIs for your SQL database. |
| Configure **pREST**, an open-source REST and MCP gateway for SQL databases, for production. | Configure open-source pREST for production — REST, auth, multi-database, and MCP on your SQL backend. |
| **pREST** is an open-source REST and MCP gateway for SQL databases. This page is… | **pREST** is open-source software for instant REST and MCP APIs on SQL databases. This page lists support labels… |

### Variants by page type

**Homepage definition (once):**

```markdown
**pREST** is open-source software that gives you instant
[REST (Representational State Transfer)](api-reference/README.md) and
[Model Context Protocol (MCP)](get-started/mcp-over-http.md) APIs for SQL databases.
Point it at a database and get CRUD, custom SQL routes, auth, ACL, and
(from v2.1.0) a read-only MCP endpoint — without hand-writing a backend.
PostgreSQL is the first native adapter.
```

**Install (Get pREST):**

```markdown
Download and run pREST — open-source instant REST (Representational State Transfer)
and MCP APIs for your SQL database.
PostgreSQL is the native adapter today — see [Databases](../databases/README.md).
If this is your first install, start with [Deploying with Docker](...).
```

**Configure (Get Started):**

```markdown
Configure pREST for production: REST (Representational State Transfer) APIs, auth,
multi-database routing, and MCP (Model Context Protocol).
PostgreSQL is native today; see [Databases](../databases/README.md) for support labels
and the [roadmap](../databases/roadmap.md).
```

(Install/configure pages can omit “open-source” if the sentence is pure procedure; prefer it on product definitions.)

**Database landing:**

```markdown
Use pREST to expose {Engine} as REST and MCP APIs.
**Label:** Native | Certified | …
```

**Roadmap:**

```markdown
This page describes **planned** SQL adapters for open-source pREST. Nothing here is installable until a release ships it.
PostgreSQL is native today — see [Databases](README.md).
```

## Do-not-claim

| Avoid | Use |
|-------|-----|
| "Best" / "#1" / "only" | Specific outcomes: open-source instant REST + MCP, PostgreSQL-first |
| MySQL/SQLite/SQL Server as shipped | **Roadmap** + link `databases/roadmap.md` |
| Competitor trademarks | Categories: hand-written APIs, BaaS, Postgres-native API servers |
| `(PostgreSQL-first)` on every line | One clear native-adapter sentence + Databases link |
| Banning “open-source” | Use it on identity/category sentences |
| Thin SEO clones | Original matrix + limitations per engine |

## Frontmatter descriptions

Answer-first, unique per page, ~150–160 characters. Prefer outcomes over “gateway” labels; “open-source” is fine:

```yaml
description: >-
  Download and run open-source pREST — Docker, Homebrew, or Go — for instant
  REST and MCP APIs on SQL databases (PostgreSQL native today).
```
