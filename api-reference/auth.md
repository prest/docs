# Auth

_**prestd**_ has support in **JWT Token** generation based on two fields (example user and password), being possible to use an existing table from your database to login configuring some parameters in the configuration file (or environment variable), _by default this feature is_ **disabled**.

* Bearer - [RFC 6750](https://tools.ietf.org/html/rfc6750), bearer tokens to access OAuth 2.0-protected resources
* Basic - [RFC 7617](https://tools.ietf.org/html/rfc7617), base64-encoded credentials. More information below

> understand more about _http authentication_ [see this documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

---

## JWT verification (v2 defaults)

JWT middleware is **enabled by default** (`jwt.default = true`). All endpoints require a valid Bearer token unless listed in the JWT whitelist. See [Configuring pREST — JWT](../get-started/configuring-prest.md#jwt) for full configuration.

### Required verification material

When JWT is enabled and debug mode is off, the server **refuses to start** unless one of the following is configured:

| Setting | Environment variable | Purpose |
|---------|---------------------|---------|
| `jwt.key` | `PREST_JWT_KEY` | Shared secret for HS256 (and other HMAC algorithms) |
| `jwt.jwks` | `PREST_JWT_JWKS` | JSON Web Key Set for asymmetric verification |
| `jwt.wellknownurl` | `PREST_JWT_WELLKNOWNURL` | OpenID Connect well-known URL to fetch JWKS |

When `auth.enabled = true`, `jwt.key` is **required** for verifying tokens issued by the `/auth` endpoint.

Debug mode (`PREST_DEBUG=true` or `debug = true` in TOML) bypasses startup JWT validation and disables JWT middleware at runtime.

To disable JWT entirely, set `jwt.default = false`.

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
curl -i -X POST http://127.0.0.1:8000/auth -H "Content-Type: application/json" -d '{"username": "<username>", "password": "<password>"}'
```

### Basic

```sh
curl -i -X POST http://127.0.0.1:8000/auth --user "<username>:<password>"
```
