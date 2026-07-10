# Upgrading to v2

This guide covers migrating from pREST v1 to v2.

- **Latest stable v2:** [v2.1.0](https://github.com/prest/prest/releases/tag/v2.1.0)
- **Docker:** `prest/prest:v2.1.0`

See [Releases](../releases/README.md) for the full changelog and [v2.1.0 release notes](../releases/v2.1.0.md).

MCP over HTTP requires v2.1.0 or later. See the [MCP over HTTP guide](mcp-over-http.md). For Cursor, Claude Desktop, and other stdio clients, install the [pREST MCP Adapter](prest-mcp-adapter.md) (`brew install prest/tap/prest-mcp`).

---

## 1. Choose your build

**v2.0.0 (recommended):**

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
| **v2.0.0** (current) | JWT middleware **auto-disabled** with error log ([#974](https://github.com/prest/prest/pull/974)) |
| **v2.0.0-rc6 tag** (historical) | Server **refuses to start** ([#960](https://github.com/prest/prest/pull/960)) |

In **v2.0.0**, `jwt.default` defaults to **`false`** — set `jwt.default = true` explicitly to enable JWT enforcement.

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

Set `pg.single = false` and define `[[databases]]` entries or `DATABASE_ALIAS_N` / `DATABASE_URL_N` environment pairs.

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
- **Logging:** `PREST_LOG_LEVEL=debug` for structured JSON logs

---

## 9. Docker-specific checklist

```sh
docker run -d -p 3000:3000 \
    -e PREST_VERSION=2 \
    -e PREST_PG_URL=postgres://username:password@hostname:port/dbname \
    -e PREST_JWT_KEY=your-secret-key \
    prest/prest:v2.0.0
```

For local development without JWT, add `-e PREST_DEBUG=true`.

In **v2.0.0**, the server starts even without `PREST_JWT_KEY`, but JWT will be auto-disabled — check logs.

See [Deploying with Docker](../deployment/deploying-with-docker.md).
