---
description: >-
  SQLite REST and MCP support for pREST is on the roadmap (Phase 3) — turn a
  SQLite file into an API is planned, not shipped.
---

# SQLite (roadmap)

**Status: Roadmap (Phase 3).** pREST cannot open a SQLite file as a database backend in current releases.

*Last updated: July 11, 2026* · **Label:** Roadmap

---

## Planned positioning

> Turn a SQLite file into a REST and MCP API with one binary.

Target use cases: local development, desktop apps, internal tools, prototypes, edge deployments, AI agents over local data — without running a database server.

SQLite is not “small PostgreSQL”; a dedicated dialect must handle schemas, concurrency, types, and metadata differently.

---

## What works today

Use [PostgreSQL](postgresql.md) (including lightweight local Postgres/Docker) or another [available engine](README.md).

---

## Related

- [Database roadmap](roadmap.md)
- [Databases](README.md)
