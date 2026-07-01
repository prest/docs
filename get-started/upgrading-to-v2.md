# Upgrading to v2

This guide covers migrating from pREST v1 to **v2.0.0-rc6**. v2 is a **release candidate** — test thoroughly before production use. See [Releases](../releases/README.md) for the full rc1–rc6 changelog.

---

## 1. Download v2.0.0-rc6

Choose one:

- **Binary:** [v2.0.0-rc6 release assets](https://github.com/prest/prest/releases/tag/v2.0.0-rc6)
- **Docker:** `prest/prest:v2.0.0-rc6`

> **Note:** Windows ARM binaries are not published in rc6 ([#970](https://github.com/prest/prest/pull/970)).

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

v2 enforces JWT configuration at startup (rc6, [#960](https://github.com/prest/prest/pull/960)):

- When `jwt.default = true` (default) and `debug = false`, the server **refuses to start** unless `jwt.key`, `jwt.jwks`, or `jwt.wellknownurl` is set.
- When `auth.enabled = true`, `jwt.key` is **required**.

**Options:**

1. **Provide verification material** (recommended for production):

```sh
export PREST_JWT_KEY="your-secret-key"
```

   Or configure JWKS / well-known URL in TOML or via `PREST_JWT_JWKS` / `PREST_JWT_WELLKNOWNURL`.

2. **Disable JWT middleware** (development only):

```toml
[jwt]
default = false
```

3. **Use debug mode** (development only — bypasses startup JWT validation):

```sh
export PREST_DEBUG=true
```

The JWT whitelist default changed to regex `^\/auth$` (v1 used `[/auth]`). Review your whitelist if you customized it.

See [Configuring pREST — JWT](configuring-prest.md#jwt) and [Auth](../api-reference/auth.md).

---

## 5. Review identifier validation

rc4 hardened identifier validation ([#GHSA-p46v-f2x8-qp98](https://github.com/prest/prest/security/advisories/GHSA-p46v-f2x8-qp98)). Invalid field or table names in query parameters are now rejected. Test your existing API calls — especially dynamic filters and `_select` field lists — against v2.

---

## 6. Review auth encryption default

v2 defaults `auth.encrypt` to `bcrypt` (v1 used `MD5`). If you use the built-in `/auth` endpoint with existing password hashes, ensure your stored passwords use a compatible algorithm or update the `encrypt` setting explicitly.

---

## 7. Test new v2 features

If applicable, verify:

- **OR filtering:** `_or=field=$eq.a||field=$eq.b` (see [Parameters](../api-reference/parameters.md))
- **Per-user permissions:** `[[access.users]]` blocks (see [Permissions](permissions.md#user-level-permissions))
- **Logging:** `PREST_LOG_LEVEL=debug` for structured JSON logs

---

## 8. Docker-specific checklist

```sh
docker run -d -p 3000:3000 \
    -e PREST_VERSION=2 \
    -e PREST_PG_URL=postgres://username:password@hostname:port/dbname \
    -e PREST_JWT_KEY=your-secret-key \
    prest/prest:v2.0.0-rc6
```

For local development without JWT, add `-e PREST_DEBUG=true`.

See [Deploying with Docker](../deployment/deploying-with-docker.md).
