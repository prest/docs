---
description: >-
  YugabyteDB REST and MCP with pREST — connect via YSQL PostgreSQL wire
  protocol, with co-location and support matrix guidance.
---

# YugabyteDB

**pREST** connects to **YugabyteDB** through the **YSQL** PostgreSQL-compatible API. Label: **Certified (PostgreSQL wire)**. Prefer co-locating `prestd` with the database nodes you query.

*Last updated: July 11, 2026* · **Label:** Certified (PostgreSQL wire)

Deep walkthrough (multi-node examples): [Integrations — YugabyteDB](../integrations/yugabytedb.md).

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | supported (YSQL port, default **5433**) |
| Schema discovery | supported |
| CRUD | supported for YSQL tables/views |
| Filtering / ordering / pagination | supported |
| Transactions | supported with YugabyteDB transaction model |
| Scripts (`/_QUERIES`) | supported for YSQL-accepted SQL |
| MCP (`/_mcp`) | supported (v2.1.0+) |
| ACL / permissions | supported |

---

## Connection

```sh
export PREST_VERSION=2
export PREST_PG_URL='postgres://yugabyte:yugabyte@yb-tserver-n1:5433/yugabyte'
```

Single-node Docker pattern (see integration guide for multi-node):

```sh
docker network create yb-net
docker run -d --hostname yb-tserver-n1 -p 7000:7000 \
  --network yb-net yugabytedb/yugabyte:latest \
  yugabyted start --daemon false --listen yb-tserver-n1

docker run -d -p 3001:3000 --network yb-net \
  -e PREST_VERSION=2 \
  -e PREST_PG_URL=postgres://yugabyte:yugabyte@yb-tserver-n1:5433/yugabyte \
  -e PREST_DEBUG=true \
  prest/prest:v2.1.0
```

---

## Node locality

- One `prestd` behind a load balancer over YugabyteDB nodes, or  
- One `prestd` per node (lower latency in geo-distributed setups)

Details: [Integrations — YugabyteDB](../integrations/yugabytedb.md).

---

## CRUD

**REST (Representational State Transfer)** maps HTTP verbs to CRUD on YSQL tables at `/{database}/{schema}/{table}` — [API Reference](../api-reference/README.md).

---

## MCP

**MCP (Model Context Protocol)** is an open standard for connecting AI apps and agents to tools and data. Use `/_mcp` on the same server — [MCP over HTTP](../get-started/mcp-over-http.md).

---

## Known limitations

- YSQL is PostgreSQL-compatible, not a drop-in for every PostgreSQL extension.
- Default SQL port is **5433**, not 5432.
- Cluster-aware Go drivers are optional advanced setups documented upstream; stock pREST uses the standard PostgreSQL driver path.
- Validate distributed-SQL behavior (transactions, uniqueness) against YugabyteDB docs for your version.

---

## Related

- [Integrations — YugabyteDB](../integrations/yugabytedb.md)
- [Databases](README.md)
- [CockroachDB](cockroachdb.md)
- [PostgreSQL](postgresql.md)
