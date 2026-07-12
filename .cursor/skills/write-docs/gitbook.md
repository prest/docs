# GitBook reference — pREST docs

This repo is Git-synced to GitBook. Prefer patterns from [GitBook’s editor skill](https://gitbook.com/docs/skill.md).

## Repo layout

```text
/
  SUMMARY.md          # Sidebar / table of contents (required for nav control)
  README.md           # Space homepage
  .gitbook/
    assets/           # Images and downloadable files
    includes/         # Reusable snippets (if used)
    vars.yaml         # Space-level variables (if used)
  section/
    page.md
```

Published docs also expose markdown via `https://docs.prestd.com/path.md` and an index at `/llms.txt`. Keep H1s and `description` frontmatter clear for those surfaces.

## SUMMARY.md

```markdown
# Table of contents

* [pREST](README.md)
* [Databases](databases/README.md)
  * [PostgreSQL](databases/postgresql.md)
```

Rules:

- `#` title once at the top
- `*` list items with markdown links
- Nest children with **spaces** (2 or 4), never tabs
- **Do not** list the same `.md` file twice (each page = one URL)
- Optional sidebar-only title: `* [Page title](path.md "Short sidebar")`
- After adding/renaming/deleting pages, update SUMMARY in the same change

## Frontmatter

Place at the very top of the file:

```markdown
---
description: Answer-first summary for SEO and page previews (~150–160 chars).
icon: database
hidden: false
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
---
```

Common fields:

| Field | Use |
|-------|-----|
| `description` | SEO / preview / llms.txt-facing blurb — required on hubs |
| `icon` | Font Awesome-style icon name in GitBook UI |
| `hidden` | Hide from sidebar while keeping the page |
| `layout` | Toggle title, TOC, outline, pagination, width (`default` \| `wide`) |

## Links

- Internal: `[Databases](../databases/README.md)`  
- External: full `https://…` URLs  
- Avoid absolute filesystem paths and duplicated deep links that break on rename  

## Blocks (use sparingly)

### Hint

```markdown
{% hint style="info" %}
Prefer read-only DB roles for MCP traffic.
{% endhint %}
```

Styles: `info`, `success`, `warning`, `danger`.

### Tabs

```markdown
{% tabs %}
{% tab title="Docker" %}
...
{% endtab %}

{% tab title="Homebrew" %}
...
{% endtab %}
{% endtabs %}
```

### Stepper

```markdown
{% stepper %}
{% step %}
Install prestd.
{% endstep %}

{% step %}
Set `PREST_PG_URL`.
{% endstep %}
{% endstepper %}
```

## Do not

- Reference one markdown file twice in SUMMARY  
- Put frontmatter below the H1  
- Embed OpenAPI specs as raw markdown (upload via GitBook API/CLI/UI)  
- Commit huge binaries outside `.gitbook/assets/` without need  
- Rely on GitBook UI-only structure without updating SUMMARY in git  

## Workflow

1. Read SUMMARY for where the page belongs  
2. Write/edit markdown + frontmatter  
3. Update SUMMARY if structure changed  
4. Prefer relative links and short Related sections  
5. After publish, spot-check docs.prestd.com (and `.md` URL if relevant)
