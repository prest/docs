# Read-only PostgreSQL for AI

When connecting AI agents to PostgreSQL through pREST MCP, give the agent a **least-privilege** database role: connect, use schemas, and `SELECT` — nothing that can change data.

MCP tools in pREST v2.1.0 are already read-only, but REST on the same server may still allow writes. A read-only Postgres role is the durable safety net for local and shared environments.

---

## What to grant

| Privilege | Why |
|-----------|-----|
| `CONNECT` on the database | Allow the role to open a session |
| `USAGE` on schemas the agent may browse | Allow resolving objects in those schemas |
| `SELECT` on tables (or views) you want exposed | Allow reads only |
| Optional `ALTER DEFAULT PRIVILEGES` | Auto-grant `SELECT` on future tables in a schema |

Do **not** grant `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, or DDL to the AI role.

---

## Example: create a read-only role

Run as a privileged database admin (superuser or owner). Replace names to match your database, schema, and tables.

```sql
-- Create a login role for AI / MCP exploration (set your own password).
DO $$
BEGIN
  IF NOT EXISTS (SELECT FROM pg_roles WHERE rolname = 'prest_readonly') THEN
    CREATE ROLE prest_readonly LOGIN PASSWORD 'change-me';
  END IF;
END
$$;

-- Allow connection to your database (replace `mydb`).
GRANT CONNECT ON DATABASE mydb TO prest_readonly;

-- Allow schema usage (replace `public` if needed).
GRANT USAGE ON SCHEMA public TO prest_readonly;

-- Grant SELECT on specific tables:
GRANT SELECT ON TABLE public.users TO prest_readonly;
-- GRANT SELECT ON TABLE public.orders TO prest_readonly;

-- Or grant SELECT on all existing tables in the schema:
-- GRANT SELECT ON ALL TABLES IN SCHEMA public TO prest_readonly;

-- Optional: future tables created by the current role get SELECT for prest_readonly
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT ON TABLES TO prest_readonly;
```

Point pREST at this role via your usual Postgres connection settings ([Configuring pREST](../get-started/configuring-prest.md)). Restart `prestd` after changing credentials.

---

## Verify

As the read-only role (or via pREST MCP):

```sql
SELECT current_user;
-- Should be prest_readonly (or your role name)

SELECT * FROM public.users LIMIT 5;
-- Should succeed if SELECT was granted

-- INSERT / UPDATE / DELETE should fail with permission denied
```

Then confirm MCP discovery:

```sh
curl -s http://localhost:3000/_mcp | head
```

Ask your AI client to list tables and select a few rows. Tools and ACL behavior: [MCP over HTTP](../get-started/mcp-over-http.md) · [Permissions](../get-started/permissions.md).

---

## Tips

* Use a strong password or prefer peer/cert auth in production; never commit demo passwords.
* Scope `SELECT` to the tables agents actually need — avoid `ALL TABLES` on sensitive schemas.
* Combine with pREST [permissions](../get-started/permissions.md) so tool discovery matches what Postgres allows.
* For Cursor-specific examples, see [prest-cursor](https://github.com/prest/prest-cursor) (`examples/mcp-readonly`).

---

## Next steps

- [Install pREST MCP Adapter](install-prest-mcp.md)
- [Use with Cursor](cursor.md) · [Claude Desktop](claude-desktop.md)
- [PostgreSQL to AI agent](../tutorials/postgres-to-ai-agent.md)

## Related documentation

- [MCP Overview](mcp-overview.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Permissions](../get-started/permissions.md)
- [Auth](../api-reference/auth.md)
