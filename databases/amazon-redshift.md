---
description: >-
  Amazon Redshift with pREST — PostgreSQL-based warehouse connectivity with
  important dialect differences and a cautious support matrix.
---

# Amazon Redshift

**Amazon Redshift** is PostgreSQL-based but **not** full PostgreSQL. **pREST** can connect using the PostgreSQL driver path with vendor-required parameters. Label: **Compatible with caveats**.

*Last updated: July 11, 2026* · **Label:** Compatible with caveats

Also see: [Integrations — Amazon Redshift](../integrations/amazon-redshift.md).

---

## Supported features

| Capability | Status |
|------------|--------|
| Connection | partial — requires Redshift-compatible URL options |
| Schema discovery | partial — validate against your cluster |
| CRUD | partial — prefer read-heavy / controlled writes; warehouse semantics differ |
| Filtering / ordering / pagination | partial — SQL differences vs PostgreSQL |
| Transactions | partial — Redshift transaction model differs |
| Scripts (`/_QUERIES`) | preferred path for Redshift-safe SQL |
| MCP (`/_mcp`) | partial — use read-only roles; verify tools against catalog |
| ACL / permissions | supported at pREST layer; also enforce IAM/DB grants |

---

## Connection

Use a PostgreSQL-style URL with Redshift’s `OpenSourceSubProtocolOverride` (see AWS docs on Redshift vs PostgreSQL):

```sh
export PREST_VERSION=2
export DATABASE_URL='postgresql://user:pass@redshift-endpoint:5439/dev?OpenSourceSubProtocolOverride=true'
```

Use `PREST_PG_SSL_*` for TLS. See [Upgrading to v2](../get-started/upgrading-to-v2.md) for SSL env naming.

---

## Recommendations

- Treat Redshift primarily as **analytics / read** access through pREST.
- Prefer [custom queries](../api-reference/custom-queries.md) over assuming PostgreSQL CRUD parity.
- For AI agents, use read-only credentials and [MCP over HTTP](../get-started/mcp-over-http.md).
- Support labels and roadmap: [Databases](../databases/README.md).

Also see the shorter integration note: [Integrations — Amazon Redshift](../integrations/amazon-redshift.md).

---

## Known limitations

- Important SQL and type differences vs PostgreSQL — read [AWS: Redshift and PostgreSQL](https://docs.aws.amazon.com/redshift/latest/dg/c_redshift-and-postgres-sql.html).
- Not a substitute for a transactional OLTP PostgreSQL deployment.
- Do not assume every pREST filter operator maps cleanly; test before production.

---

## Related

- [Integrations — Amazon Redshift](../integrations/amazon-redshift.md)
- [Databases](README.md)
- [Roadmap](roadmap.md) (analytical read-only track)
