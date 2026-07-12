---
description: >-
  Acronyms used in pREST docs — REST, MCP, AI, SQL, HTTP, JWT, ACL, and related
  terms with short definitions and links to deeper guides.
---

# Acronyms

Short definitions for acronyms used across the pREST docs. Use bare terms in most pages and [link here from Related](#how-to-use-this-page). Expand inline only when a page is meant for SEO or discovery (homepage, FAQ, “What is …?” hubs).

**pREST** is the product name — not an expandable acronym on this page.

## How to use this page

- Prefer linking this glossary (and specific anchors) from a page’s **Related** section when those terms appear in prose.
- Deep guides for REST, MCP, and AI stay on their canonical pages; entries below stay short.

---

## ACL

**Access Control List.** In pREST, table- and user-level rules that restrict which resources clients can reach — see [Permissions](../get-started/permissions.md).

## AI

**Artificial intelligence.** In these docs, applications and agents that use models to reason and act on tools and data. pREST is not an LLM; it exposes SQL to AI clients via MCP and REST. Deep guide: [MCP over HTTP](../get-started/mcp-over-http.md). Client setup: [docs.prestd.com/ai](https://docs.prestd.com/ai).

## API

**Application Programming Interface.** A contract for calling a system over a network or library boundary. pREST serves HTTP APIs (REST and MCP) generated from your database schema.

## CRUD

**Create, Read, Update, Delete.** The four basic data operations. pREST maps them to HTTP verbs on `/{database}/{schema}/{table}` — see [API Reference](../api-reference/README.md).

## DDL

**Data Definition Language.** SQL statements that change schema (for example `CREATE TABLE`). pREST’s MCP surface in v2.1.0 does not run DDL; use your database tooling or allowed REST/custom-query paths as configured.

## HTTP

**Hypertext Transfer Protocol.** The application protocol pREST uses for REST and MCP over HTTP endpoints.

## IDE

**Integrated Development Environment.** Editors and tools (for example Cursor) that can connect as MCP or HTTP clients to pREST.

## JSON

**JavaScript Object Notation.** The usual request and response body format for pREST REST routes and MCP JSON-RPC payloads.

## JWT

**JSON Web Token.** A signed token format used when pREST auth is configured — see [Auth](../api-reference/auth.md).

## MCP

**Model Context Protocol.** An open standard for connecting AI apps and agents to tools and data. pREST exposes read-only MCP at `/_mcp`. Spec: [modelcontextprotocol.io](https://modelcontextprotocol.io/). Deep guide: [MCP over HTTP](../get-started/mcp-over-http.md).

## REST

**Representational State Transfer.** An architectural style for APIs over HTTP. pREST exposes SQL tables as REST resources at `/{database}/{schema}/{table}`. Glossary: [MDN REST](https://developer.mozilla.org/en-US/docs/Glossary/REST). Deep guide: [API Reference](../api-reference/README.md).

## SQL

**Structured Query Language.** The language of relational databases. pREST turns HTTP and MCP calls into parameterized SQL against the connected engine (PostgreSQL native today).

## Related

- [Homepage](../README.md)
- [Key features](prestd-key-features.md)
- [API Reference](../api-reference/README.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Databases](../databases/README.md)
