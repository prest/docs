---
name: write-docs
description: >-
  Write and edit pREST documentation in this GitBook-synced repo. Use when
  creating or changing markdown pages, SUMMARY.md, frontmatter, hints/tabs,
  phrasing/voice, or when the user asks how docs should be structured for
  GitBook / docs.prestd.com.
---

# Write pREST docs (GitBook)

Read [voice.md](voice.md) for phrasing and [gitbook.md](gitbook.md) for GitBook syntax. For LLM/answer-engine optimization, also follow [../llm-seo/SKILL.md](../llm-seo/SKILL.md).

**Published as:** GitBook space synced from this repo → [docs.prestd.com](https://docs.prestd.com) (also serves `.md` and `llms.txt`).

---

## When to apply

- Any new or edited `*.md` page in this repo
- Changes to [`SUMMARY.md`](../../../SUMMARY.md)
- Frontmatter, callouts, tabs, steppers, images
- “How should we phrase this?” / docs style questions

---

## Product voice

**Positioning (use once when defining the product):**

> Instant REST and MCP APIs for SQL databases. PostgreSQL-first, now multi-database.

**Category (SEO/LLM, when needed):** open-source REST and MCP APIs for SQL databases

**“open-source”:** use it where it adds meaning (product identity, category, community/license). Do **not** strip it. Avoid only the **gateway** appositive and `(PostgreSQL-first)` on every lead.

**Page leads:** start with the reader’s job, then one short benefit. Do **not** use the gateway appositive pattern:

```text
Bad:  Install **pREST**, an open-source REST and MCP gateway for SQL databases (PostgreSQL-first).
Good: Download and run pREST — open-source instant REST and MCP APIs for your SQL database.
```

Prefer linking [Databases](../../../databases/README.md) over stuffing “(PostgreSQL-first)” into every sentence. Full variants: [voice.md](voice.md).

---

## Page structure

1. YAML frontmatter with answer-first `description` (~150–160 chars) on hubs and new pages  
2. Single H1 matching the topic (align with SUMMARY title when practical)  
3. First 2–3 sentences answer the page’s main question  
4. One job per section; use tables/lists for facts  
5. End with **Related** (3+ internal links when useful). If the page uses glossary acronyms in prose (REST, MCP, AI, SQL, JWT, …), include [`prestd/acronyms.md`](../../../prestd/acronyms.md) (and anchors when practical) — see [voice.md](voice.md#acronyms-reference-first). Expand acronyms inline only when SEO/discovery warrants it.  
6. No competitor trademarks; no unshipped install claims ([voice.md](voice.md#do-not-claim))

---

## GitBook rules (summary)

- Read [`SUMMARY.md`](../../../SUMMARY.md) before structural changes; keep it in sync with new/moved files  
- Reference each markdown file **once** in SUMMARY (unique URL)  
- Indent nested pages with **spaces**, not tabs: `* [Title](path.md)`  
- Frontmatter must be the first thing in the file  
- Internal links: relative paths only (`../folder/page.md`)  
- Use `{% hint %}`, `{% tabs %}`, `{% stepper %}` when they clarify — don’t decorate every paragraph  
- Images/files: `.gitbook/assets/` when adding media  
- Titles + descriptions should stay citation-ready for published `llms.txt` / `.md` URLs  

Details and examples: [gitbook.md](gitbook.md).

---

## Honesty (databases)

Use support labels from [databases/README.md](../../../databases/README.md). Roadmap stubs must not claim install steps. See [../llm-seo/SKILL.md](../llm-seo/SKILL.md) for database page minimums and SEO rules.

---

## Verification before merge

- [ ] Lead starts with the reader’s job or the page question (no gateway appositive)  
- [ ] Frontmatter `description` unique and answer-first  
- [ ] SUMMARY updated if the URL set changed  
- [ ] Relative links resolve  
- [ ] Support / roadmap claims honest  
- [ ] GitBook blocks used only where they help  
- [ ] Glossary acronyms in prose → Related links to `prestd/acronyms.md` (expand inline only for SEO/discovery hubs)

---

## Related skills

- [llm-seo](../llm-seo/SKILL.md) — AEO, question bank, support labels, database SEO rules  
- [voice.md](voice.md) · [gitbook.md](gitbook.md)
