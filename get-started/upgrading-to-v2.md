# Upgrading to v2

This guide covers migrating from pREST v1 to v2.

- **Latest stable v2:** [v2.5.2](https://github.com/prest/prest/releases/tag/v2.5.2)
- **Docker:** `prest/prest:v2.5.2`

See [Releases](../releases/README.md) for the full changelog and [v2.5.2 release notes](../releases/v2.5.2.md). If you are still on v2.2.0 or earlier, note that [v2.3.0](../releases/v2.3.0.md) included a critical `_select` SQL-injection fix — upgrade promptly.

{% hint style="danger" %}
**If you skipped v2.4.2, check your `jwt.key` length before deploying v2.5.2.** From v2.4.2 an HMAC key below the RFC 7518 minimum (32 bytes for HS256, 48 for HS384, 64 for HS512) is discarded at startup. pREST does not refuse to start — it unregisters `/auth` and serves requests unauthenticated. See [Auth — HMAC key requirements](../api-reference/auth.md#hmac-key-requirements-v242) and [v2.4.2 — Upgrading](../releases/v2.4.2.md#upgrading-from-v241).
{% endhint %}

{% hint style="danger" %}
**v2.5.1 and earlier are affected by three advisories fixed in v2.5.2** ([GHSA-2x9q-r7j9-g85f](https://github.com/prest/prest/security/advisories/GHSA-2x9q-r7j9-g85f), [GHSA-fhf5-m388-4w4p](https://github.com/prest/prest/security/advisories/GHSA-fhf5-m388-4w4p), [GHSA-fmr6-9j48-xrqp](https://github.com/prest/prest/security/advisories/GHSA-fmr6-9j48-xrqp)). From v2.5.2, `prest_users` and `prest_queries` are unreachable over REST, the catalog routes and `/_PLUGIN` require a token when `auth.enabled = true`, and `pg.single` is ignored. See [v2.5.2 — Upgrading](../releases/v2.5.2.md#upgrading-from-v251).
{% endhint %}

MCP over HTTP requires v2.1.0 or later. Studio requires v2.2.0 or later — [pREST Studio](prest-studio.md). For Cursor, Claude Desktop, and other stdio clients, install the [pREST MCP Adapter](prest-mcp-adapter.md) (`brew install prest/tap/prest-mcp`).

---

## 1. Choose your build

**v2.5.2 (recommended):**

- **Binary:** [v2.5.2 release assets](https://github.com/prest/prest/releases/tag/v2.5.2)
- **Docker:** `prest/prest:v2.5.2`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.5.2`

**v2.5.1** (affected by the advisories fixed in v2.5.2):

- **Binary:** [v2.5.1 release assets](https://github.com/prest/prest/releases/tag/v2.5.1)
- **Docker:** `prest/prest:v2.5.1`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.5.1`

**v2.5.0:**

- **Binary:** [v2.5.0 release assets](https://github.com/prest/prest/releases/tag/v2.5.0)
- **Docker:** `prest/prest:v2.5.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.5.0`

**v2.4.2:**

- **Binary:** [v2.4.2 release assets](https://github.com/prest/prest/releases/tag/v2.4.2)
- **Docker:** `prest/prest:v2.4.2`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.4.2`

**v2.4.0:**

- **Binary:** [v2.4.0 release assets](https://github.com/prest/prest/releases/tag/v2.4.0)
- **Docker:** `prest/prest:v2.4.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.4.0`

**v2.3.0:**

- **Binary:** [v2.3.0 release assets](https://github.com/prest/prest/releases/tag/v2.3.0)
- **Docker:** `prest/prest:v2.3.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.3.0`

**v2.2.0:**

- **Binary:** [v2.2.0 release assets](https://github.com/prest/prest/releases/tag/v2.2.0)
- **Docker:** `prest/prest:v2.2.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.2.0`

**v2.1.0:**

- **Binary:** [v2.1.0 release assets](https://github.com/prest/prest/releases/tag/v2.1.0)
- **Docker:** `prest/prest:v2.1.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.1.0`

**v2.0.0:**

- **Binary:** [v2.0.0 release assets](https://github.com/prest/prest/releases/tag/v2.0.0)
- **Docker:** `prest/prest:v2.0.0`
- **Go install:** `go install github.com/prest/prest/v2/cmd/prestd@v2.0.0`

**Tip of `main` branch** (for development only):

```sh
go install github.com/prest/prest/v2/cmd/prestd@main
```

> **Note:** Windows ARM binaries are not published ([#970](https://github.com/prest/prest/pull/970)).

---

## 2. Set `PREST_VERSION=2`

Set the environment variable `PREST_VERSION=2` (recommended for all new v2 deployments). This enables v2 naming conventions for configuration, especially PostgreSQL SSL settings.

```sh
export PREST_VERSION=2
```

---

## 3. Migrate SSL environment variables

v2 removed the deprecated `PREST_SSL_*` variables (rc2, [#919](https://github.com/prest/prest/pull/919)). Rename them to the `PREST_PG_SSL_*` equivalents:

| v1 (removed) | v2 |
|--------------|-----|
| `PREST_SSL_MODE` | `PREST_PG_SSL_MODE` |
| `PREST_SSL_CERT` | `PREST_PG_SSL_CERT` |
| `PREST_SSL_KEY` | `PREST_PG_SSL_KEY` |
| `PREST_SSL_ROOTCERT` | `PREST_PG_SSL_ROOTCERT` |

In TOML, use `[pg.ssl]` instead of the top-level `[ssl]` block:

```toml
[pg.ssl]
mode = "require"
sslcert = "./PATH"
sslkey = "./PATH"
sslrootcert = "./PATH"
```

**Default change:** when no config file is found, v2 defaults `pg.ssl.mode` to `disable` (v1 used `require`). Set `PREST_PG_SSL_MODE=require` explicitly if you relied on the v1 default.

---

## 4. Configure JWT verification

JWT behavior depends on which build you run:

| Build | Missing JWT key when `jwt.default = true` |
|-------|-------------------------------------------|
| **v2.0.0+** (current) | JWT middleware **auto-disabled** with error log ([#974](https://github.com/prest/prest/pull/974)) |
| **v2.0.0-rc6 tag** (historical) | Server **refuses to start** ([#960](https://github.com/prest/prest/pull/960)) |

In **v2+**, `jwt.default` defaults to **`false`** — set `jwt.default = true` explicitly to enable JWT enforcement.

**Options for production:**

1. **Provide verification material** (recommended):

```sh
export PREST_JWT_KEY="your-secret-key"
```

   Or configure JWKS / well-known URL via `PREST_JWT_JWKS` / `PREST_JWT_WELLKNOWNURL`.

2. **Disable JWT middleware:**

```toml
[jwt]
default = false
```

3. **Use debug mode** (development only):

```sh
export PREST_DEBUG=true
```

Always check startup logs to confirm JWT and auth are in the expected state.

The JWT whitelist default is regex `^\/auth$` (v1 used `[/auth]`). Review your whitelist if you customized it.

See [Configuring pREST — JWT](configuring-prest.md#jwt) and [Auth](../api-reference/auth.md).

---

## 5. Multi-database

v2.0.0 includes full multi-database support. Configure a database registry for multi-cluster routing. See the [Multi-database guide](multi-database.md).

Define `[[databases]]` entries or `DATABASE_ALIAS_N` / `DATABASE_URL_N` environment pairs. To serve a few extra databases on one host without a registry, list them in `pg.allowed_databases` (v2.5.2). `pg.single` was removed in v2.5.2 and is ignored with a warning — see [Multi-database — Migrating from `pg.single`](multi-database.md#migrating-from-pgsingle).

Use `GET /_ready` for Kubernetes readiness probes when running multiple databases.

---

## 6. Review identifier validation

rc4 hardened identifier validation ([GHSA-p46v-f2x8-qp98](https://github.com/prest/prest/security/advisories/GHSA-p46v-f2x8-qp98)). Invalid field or table names in query parameters are now rejected. Test your existing API calls.

---

## 7. Review auth encryption default

v2 defaults `auth.encrypt` to `bcrypt` (v1 used `MD5`). If you use the built-in `/auth` endpoint with existing password hashes, ensure compatibility or update the `encrypt` setting.

---

## 8. Test new v2 features

If applicable, verify:

- **OR filtering:** `_or=field=$eq.a||field=$eq.b` (see [Parameters](../api-reference/parameters.md))
- **Per-user permissions:** `[[access.users]]` (see [Permissions](permissions.md#user-level-permissions))
- **Multi-database:** alias routing (see [Multi-database](multi-database.md))
- **MCP over HTTP:** `GET` / `POST /_mcp` (v2.1.0+; see [MCP over HTTP](mcp-over-http.md))
- **pREST Studio:** open `/_studio/` (v2.2.0+; see [pREST Studio](prest-studio.md))
- **Database-backed queries:** optional `queries.storage = "database"` (v2.2.0+; see [Custom Queries](../api-reference/custom-queries.md))
- **Logging:** `PREST_LOG_LEVEL=debug` for structured JSON logs

---

## 9. Docker-specific checklist

```sh
docker run -d -p 3000:3000 \
    -e PREST_VERSION=2 \
    -e PREST_PG_URL=postgres://username:password@hostname:port/dbname \
    -e PREST_JWT_KEY=a-secret-key-of-at-least-32-bytes \
    prest/prest:v2.5.2
```

For local development without JWT, add `-e PREST_DEBUG=true`.

In **v2+**, the server starts even without `PREST_JWT_KEY`, but JWT will be auto-disabled — check logs.

See [Deploying with Docker](../deployment/deploying-with-docker.md).
