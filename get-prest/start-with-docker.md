---
description: >-
  Run open-source pREST with Docker Compose — PostgreSQL plus prest image
  v2.1.0, auth migrate, JWT login, and optional MCP smoke check.
---

# Start with Docker

Use Docker Compose to bring up PostgreSQL and pREST for a local test environment.

{% hint style="info" %}
**v2 notes:** pin the image to `prest/prest:v2.1.0`, set `PREST_VERSION=2`, and configure JWT (`PREST_JWT_KEY` or equivalent) when JWT enforcement is enabled — or expect JWT to be auto-disabled with a warning (v2+). Use `PREST_DEBUG=true` for local development without JWT. See [Deploying with Docker](../deployment/deploying-with-docker.md) and [Upgrading to v2](../get-started/upgrading-to-v2.md).
{% endhint %}

```sh
# Download docker compose file
wget https://raw.githubusercontent.com/prest/prest/main/docker-compose-prod.yml -O docker-compose.yml

# Up (run) PostgreSQL and prestd
docker-compose up
# Run data migration to create user structure for access (JWT)
docker-compose exec prest prestd migrate up auth

# Create user and password for API access (via JWT)
## user: prest
## pass: prest
# v2 defaults auth.encrypt to bcrypt — hash with htpasswd (apache2-utils / httpd)
HASH=$(htpasswd -nbBC 10 x prest | cut -d: -f2)
docker-compose exec postgres psql -d prest -U prest -c "INSERT INTO prest_users (name, username, password) VALUES ('pREST Full Name', 'prest', '$HASH')"
# Check if the user was created successfully (by doing a select on the table)
docker-compose exec postgres psql -d prest -U prest -c "select * from prest_users"

# Generate JWT Token with user and password created
curl -i -X POST http://127.0.0.1:3000/auth -H "Content-Type: application/json" -d '{"username": "prest", "password": "prest"}'
# Access endpoint using JWT Token
curl -i -X GET http://127.0.0.1:3000/prest/public/prest_users -H "Accept: application/json" -H "Authorization: Bearer {TOKEN}"

# Optional: MCP discovery (v2.1.0+)
curl -s http://127.0.0.1:3000/_mcp | head
```

### Supported Operating System

* Linux
* macOS
* Windows
* BSD

## Related

- [Deploying with Docker](../deployment/deploying-with-docker.md)
- [Upgrading to v2](../get-started/upgrading-to-v2.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Acronyms](../prestd/acronyms.md)
