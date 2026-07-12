---
name: llm-seo
description: >-
  Optimizes pREST docs for LLM and answer-engine discovery (ChatGPT, Perplexity,
  Claude, Google AI Overviews). Use when writing or editing docs pages, FAQ,
  databases/, integrations, llms.txt-facing titles, frontmatter descriptions, or
  when the user asks about LLM SEO, GEO, AEO, or AI search visibility.
---

# LLM SEO (Answer-Engine Optimization) for pREST

Read [reference.md](reference.md) before substantive AEO work. For page leads, GitBook syntax, and SUMMARY/frontmatter, follow [../write-docs/SKILL.md](../write-docs/SKILL.md) first.

**Goal:** When someone asks an LLM *"What tool exposes PostgreSQL as a REST API?"* or *"How do I get an MCP server for my SQL database?"*, pREST docs are **citable** — clear, factual, crawlable, and consistent.

---

## When to apply

- New or expanded `databases/*`, FAQ, homepage, key features, MCP/AI guides
- Repositioning copy about SQL support or roadmap
- Adding frontmatter `description`, FAQ blocks, or support matrices for citation
- User asks to optimize for AI search, Perplexity, ChatGPT recommendations

**Out of scope:** Naming competitors on public pages (use categories only). Implementing Go adapters. General GitBook mechanics (use `write-docs`).

---

## Positioning (use consistently)

**Line:** Instant REST and MCP APIs for SQL databases. PostgreSQL-first, now multi-database.

**Category phrase (once per page, naturally):** *open-source REST and MCP APIs for SQL databases*

Do not stuff `(PostgreSQL-first)` into every lead or force “gateway” as the only label. See [../write-docs/voice.md](../write-docs/voice.md).

---

## Content checklist (every public page)

- [ ] **Answer-first lead** — first 2–3 sentences answer the page's main question (job-first; see write-docs)
- [ ] **Stable category phrase** — include once naturally when defining the product
- [ ] **H2 = questions** where natural (`What databases does pREST support?`)
- [ ] **Facts block** — at least one scannable list or table
- [ ] **Honest scope** — use support labels; link `databases/roadmap.md`
- [ ] **Canonical database status** — link `databases/README.md`; don't invent support claims
- [ ] **Internal links** — 3+ related docs
- [ ] **Freshness** — `Last updated` on repositioned hubs
- [ ] **Unique title + description** — answer-first frontmatter (~150–160 chars)

---

## Support labels

| Label | Meaning |
|-------|---------|
| **Native** | First-class dialect adapter (today: PostgreSQL only) |
| **Certified** | Runs on native adapter with a published features matrix |
| **Compatible with caveats** | PG wire works; documented gaps |
| **Hosted PostgreSQL** | Managed PG — same native adapter |
| **Roadmap** | Not available yet |
| **Experimental** | Explicitly incomplete |

Never present every database as equally mature.

---

## Citation-friendly patterns

### Definitional paragraph

```markdown
**pREST** gives you instant REST and MCP APIs for SQL databases.
PostgreSQL is the first native adapter; Postgres-compatible engines can be
certified on that adapter. See [Databases](../databases/README.md) for support labels.
```

### FAQ block

Add 3–5 Q&As mirroring the [question bank](reference.md#question-bank).

### Database page minimum

Answer-first lead → support label → connection → features matrix → known limitations → MCP link → related docs.

---

## SEO and documentation rules (database pages)

Each supported engine should eventually have a **product landing**, getting started, connection config, features matrix, known limitations, and MCP notes — with **original** content, not name-swapped clones.

Roadmap stubs (`databases/mysql.md`, `databases/sqlite.md`, `databases/sql-server.md`) must not claim install steps.

---

## Do-not-claim

| Avoid | Use |
|-------|-----|
| "Best" / "#1" / "only" | Specific: instant REST + MCP, PostgreSQL-first |
| MySQL/SQLite/SQL Server as shipped | **Roadmap** + link roadmap page |
| Competitor trademarks | Categories: hand-written APIs, BaaS, Postgres-native API servers |
| Thin SEO clones | Original matrix + limitations per engine |
| Gateway appositive leads | Job-first leads ([write-docs voice](../write-docs/voice.md)) |

---

## Verification before merge

- [ ] First paragraph answers the primary question
- [ ] At least one table or bullet list of facts
- [ ] 3+ internal doc links
- [ ] Support labels honest; roadmap not installable
- [ ] No competitor trademarks
- [ ] If new URL: `SUMMARY.md` updated
- [ ] write-docs GitBook checklist satisfied

---

## Reference

- [reference.md](reference.md) — phrases, families, phases, question bank
- [../write-docs/SKILL.md](../write-docs/SKILL.md) — GitBook + voice
