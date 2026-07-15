# Auth

_**prestd**_ has support in **JWT Token** generation based on two fields (example user and password), being possible to use an existing table from your database to login configuring some parameters in the configuration file (or environment variable), _by default this feature is_ **disabled**.

* Bearer - [RFC 6750](https://tools.ietf.org/html/rfc6750), bearer tokens to access OAuth 2.0-protected resources
* Basic - [RFC 7617](https://tools.ietf.org/html/rfc7617), base64-encoded credentials. More information below

> understand more about _http authentication_ [see this documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

---

## JWT verification (v2 defaults)

JWT middleware is **disabled by default** in v2+ (`jwt.default = false`). Set `jwt.default = true` to require a valid Bearer token on all endpoints except those in the JWT whitelist. See [Configuring pREST — JWT](../get-started/configuring-prest.md#jwt) for full configuration.

MCP (`/_mcp`, v2.1.0+) inherits the same JWT and auth stack as REST routes. When auth is enabled, send credentials on every `GET` and `POST` to `/_mcp`. Stdio clients that use the [pREST MCP Adapter](../get-started/prest-mcp-adapter.md) can pass a bearer token via `PREST_MCP_TOKEN`.

### Required verification material

When `jwt.default = true` and debug mode is off, you should configure one of the following:

| Setting | Environment variable | Purpose |
|---------|---------------------|---------|
| `jwt.key` | `PREST_JWT_KEY` | Shared secret for HS256 (and other HMAC algorithms) |
| `jwt.jwks` | `PREST_JWT_JWKS` | JSON Web Key Set for asymmetric verification |
| `jwt.wellknownurl` | `PREST_JWT_WELLKNOWNURL` | OpenID Connect well-known URL to fetch JWKS |

**In v2+ ([#974](https://github.com/prest/prest/pull/974)):** if JWT is enabled but no verification material is configured, JWT middleware is **auto-disabled** with an error log — the server continues to start. When `auth.enabled = true` without `jwt.key`, auth is also auto-disabled.

> **v2.0.0-rc6 tagged binary:** the rc6 release **refuses to start** in the same situations. See [v2.0.0-rc6](../releases/v2.0.0-rc6.md#jwt-fail-closed-startup-960).

Debug mode (`PREST_DEBUG=true` or `debug = true` in TOML) bypasses JWT enforcement at runtime.

To disable JWT entirely, leave `jwt.default = false` (the default in v2+).

### Whitelist

Endpoints matching the whitelist regex do not require a JWT. The v2 default whitelist is `^\/auth$` (only the `/auth` endpoint). Configure additional patterns in TOML or via `PREST_JWT_WHITELIST`:

```toml
[jwt]
whitelist = ["\\/auth", "\\/ping", "\\/ping\\/.*"]
```

### Upgrading from v1

v1 used `[/auth]` as the default whitelist and did not enforce JWT key configuration at startup. See [Upgrading to v2](../get-started/upgrading-to-v2.md) for migration steps.

---

## Token generation (`/auth` endpoint)

When `auth.enabled = true`, pREST exposes a `/auth` endpoint that validates credentials against a database table and returns a signed JWT.

### Bearer

```sh
curl -i -X POST http://127.0.0.1:3000/auth -H "Content-Type: application/json" -d '{"username": "<username>", "password": "<password>"}'
```

### Basic

```sh
curl -i -X POST http://127.0.0.1:3000/auth --user "<username>:<password>"
```

## Related

- [Configuring pREST — JWT](../get-started/configuring-prest.md#jwt)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Install pREST MCP Adapter](../ai/install-prest-mcp.md)
- [Permissions](../get-started/permissions.md)
- [Acronyms](../prestd/acronyms.md) · [JWT](../prestd/acronyms.md#jwt) · [MCP](../prestd/acronyms.md#mcp)
