---
description: >-
  MySQL and MariaDB REST/MCP support for pREST is on the roadmap (Phase 2) —
  not available in current releases.
---

# MySQL family (roadmap)

**Status: Roadmap (Phase 2).** pREST does **not** ship a MySQL or MariaDB adapter yet. Current releases speak the **PostgreSQL** wire protocol only.

*Last updated: July 11, 2026* · **Label:** Roadmap

---

## Planned coverage

One MySQL-family adapter is intended to unlock:

- MySQL  
- MariaDB (separate dialect profile over time)  
- Amazon Aurora MySQL  
- TiDB (after compatibility testing)  
- Some Vitess / PlanetScale-style deployments after testing  

Intended SEO/product pages later: MySQL REST API, MySQL MCP, MariaDB REST API, Aurora MySQL REST API, TiDB REST API — each with original matrices, not name-only clones.

---

## What works today

Use [PostgreSQL](postgresql.md) or a [certified Postgres-compatible engine](README.md). There is **no** supported `mysql://` connection path in current `prestd` builds.

---

## Related

- [Database roadmap](roadmap.md)
- [Databases](README.md)
- [Homepage](../README.md)
