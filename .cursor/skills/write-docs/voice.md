# Voice reference — pREST docs

## Positioning

**Line (canonical):** Instant REST and MCP APIs for SQL databases. PostgreSQL-first, now multi-database.

**Category (when needed):** open-source REST and MCP APIs for SQL databases

Do not force the word **gateway** into every lead. Prefer the positioning line or a job-focused sentence.

## Page-lead pattern

1. Reader’s job (Install / Configure / Connect X / Which databases…)  
2. One benefit or outcome sentence  
3. Link to Databases or roadmap when scope matters  

### Bad vs good

| Bad | Good |
|-----|------|
| Install **pREST**, an open-source REST and MCP gateway for SQL databases (PostgreSQL-first). | Download and run pREST to get instant REST and MCP APIs for your SQL database. |
| Configure **pREST**, an open-source REST and MCP gateway for SQL databases, for production. | Configure pREST for production — REST, auth, multi-database, and MCP on your SQL backend. |
| **pREST** is an open-source REST and MCP gateway for SQL databases. This page is… | This page lists which SQL databases pREST supports today and what is still on the roadmap. |

### Variants by page type

**Homepage definition (once):**

```markdown
**pREST** gives you instant REST and MCP APIs for SQL databases.
Point it at a database and get CRUD, custom SQL routes, auth, ACL, and
(from v2.1.0) read-only MCP — without hand-writing a backend.
PostgreSQL is the first native adapter.
```

**Install (Get pREST):**

```markdown
Download and run pREST to get instant REST and MCP APIs for your SQL database.
PostgreSQL is the native adapter today — see [Databases](../databases/README.md).
If this is your first install, start with [Deploying with Docker](...).
```

**Configure (Get Started):**

```markdown
Configure pREST for production: server settings, auth, multi-database routing, and MCP.
PostgreSQL is native today; see [Databases](../databases/README.md) for support labels and the [roadmap](../databases/roadmap.md).
```

**Database landing:**

```markdown
Use pREST to expose {Engine} as REST and MCP APIs.
**Label:** Native | Certified | …
```

**Roadmap:**

```markdown
This page describes **planned** SQL adapters. Nothing here is installable until a release ships it.
PostgreSQL is native today — see [Databases](README.md).
```

## Do-not-claim

| Avoid | Use |
|-------|-----|
| "Best" / "#1" / "only" | Specific outcomes: instant REST + MCP, PostgreSQL-first |
| MySQL/SQLite/SQL Server as shipped | **Roadmap** + link `databases/roadmap.md` |
| Competitor trademarks | Categories: hand-written APIs, BaaS, Postgres-native API servers |
| `(PostgreSQL-first)` on every line | One clear native-adapter sentence + Databases link |
| Thin SEO clones | Original matrix + limitations per engine |

## Frontmatter descriptions

Answer-first, unique per page, ~150–160 characters. Prefer outcomes over “gateway” labels:

```yaml
description: >-
  Download and run pREST — Docker, Homebrew, or Go — for instant REST and
  MCP APIs on SQL databases (PostgreSQL native today).
```
