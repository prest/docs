# Configuring pREST

The _**prestd**_ configuration is via an _environment variable_ or _toml_ file. Starting from version [`v1.2.0`](https://github.com/prest/prest/releases/tag/v1.2.0) it will be possible to use `prestd` without any _environment variable_ or the _toml_ file, but the configurations used will be the described in the default column bellow.

### Environment variables

| var                                  | default          | description                                                                                                                              |
| ------------------------------------ | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `PREST_VERSION`                      | `1`              | version used for environment variables. Set to `2` for v2 deployments (recommended). v2 uses `PREST_PG_SSL_*` for PostgreSQL SSL. |
| `PREST_CONF`                         | `./prest.conf`   |                                                                                                                                          |
| `PREST_MIGRATIONS`                   | `./migrations`   |                                                                                                                                          |
| `PREST_QUERIES_LOCATION`             | `./queries`      |                                                                                                                                          |
| `PREST_HTTP_HOST`                    | `0.0.0.0`        |                                                                                                                                          |
| `PREST_HTTP_PORT` **or** `PORT`      | `3000`           | `PORT` is for cloud factor, _when declared this variable overwrittes_ `PREST_HTTP_PORT`                                                  |
| `PREST_PG_HOST`                      | `127.0.0.1`      | host used to connect                                                                                                                     |
| `PREST_PG_USER`                      | `postgres`       | user used to connect                                                                                                                     |
| `PREST_PG_PASS`                      | `postgres`       | password used to connect                                                                                                                 |
| `PREST_PG_DATABASE`                  | `prest`          | database name used to connect                                                                                                            |
| `PREST_PG_PORT`                      | `5432`           |                                                                                                                                          |
| `PREST_PG_URL` **or** `DATABASE_URL` |                  | cloud factor, _when declaring this variable all the previous connection fields are overwritten_                                          |
| `PREST_PG_SSL_MODE`                  | `disable`        | Postgres connection SSL mode (`disable`, `require`, `verify-ca`, `verify-full`). Sample default matches `[pg.ssl]` when no config is set. |
| `PREST_PG_SSL_CERT`                  |                  | Postgres connection SSL certificate                                                                                                      |
| `PREST_PG_SSL_KEY`                   |                  | Postgres connection SSL key                                                                                                              |
| `PREST_PG_SSL_ROOTCERT`              |                  | Postgres connection SSL root certificate                                                                                                 |
| `PREST_PG_SINGLE`                    | `true`           | When `false`, allows routing to multiple databases or aliases. See [Multi-database](multi-database.md).                                  |
| `DATABASE_ALIAS_N`                   |                  | Multi-database registry alias (1-based index). Also `PREST_DATABASE_ALIAS_N`. See [Multi-database](multi-database.md).                    |
| `DATABASE_URL_N`                     |                  | Connection URL for `DATABASE_ALIAS_N`. Also `PREST_DATABASE_URL_N`. Env wins over TOML.                                                  |
| `PREST_CACHE_ENABLED`                | false            | embedded cache system                                                                                                                    |
| `PREST_CACHE_TIME`                   | 10               | TTL in minute (time to live)                                                                                                             |
| `PREST_CACHE_STORAGEPATH`            | ./               | path where the cache file will be created                                                                                                |
| `PREST_CACHE_SUFIXFILE`              | .cache.prestd.db | suffix of the name of the file that is created                                                                                           |
| `PREST_JWT_DEFAULT`                  | `false`          | v2+: enable default JWT middleware on all routes (except whitelist)                                                                        |
| `PREST_STUDIO_ENABLED`               | `true`           | v2.2.0+: serve embedded pREST Studio at `/_studio/` (set `false` to disable; returns 404)                                                  |
| `PREST_JWT_KEY`                      |                  |                                                                                                                                          |
| `PREST_JWT_ALGO`                     | HS256            | (Deprecated) Not used                                                                                                                    |
| `PREST_JWT_WELLKNOWNURL`             |                  | URL of .wellknown config of IDP used to fetch the JWKS used to verify token signature. Ignored if PREST_JWT_JWKS is set                  | 
| `PREST_JWT_JWKS`                     |                  | JWKS used to verify token signature. If set, PREST_JWT_WELLKNOWNURL is ignored                                                           | 
| `PREST_JWT_WHITELIST`                | `^\/auth$`       | Regex patterns for endpoints that skip JWT verification (v2)                                                                               |
| `PREST_AUTH_ENABLED`                 | `false`          |                                                                                                                                          |
| `PREST_AUTH_ENCRYPT`                 | `bcrypt`         | v2 default; v1 used `MD5`                                                                                                                |
| `PREST_AUTH_TYPE`                    | `body`           |                                                                                                                                          |
| `PREST_AUTH_SCHEMA`                  | `public`         |                                                                                                                                          |
| `PREST_AUTH_TABLE`                   | `prest_users`    |                                                                                                                                          |
| `PREST_AUTH_USERNAME`                | `username`       |                                                                                                                                          |
| `PREST_AUTH_PASSWORD`                | `password`       |                                                                                                                                          |
| `PREST_SSL_MODE`                     | `require`        | **v1 only (removed in v2)** — use `PREST_PG_SSL_MODE` instead                                                                              |
| `PREST_SSL_CERT`                     |                  | **v1 only (removed in v2)** — use `PREST_PG_SSL_CERT` instead                                                                              |
| `PREST_SSL_KEY`                      |                  | **v1 only (removed in v2)** — use `PREST_PG_SSL_KEY` instead                                                                               |
| `PREST_SSL_ROOTCERT`                 |                  | **v1 only (removed in v2)** — use `PREST_PG_SSL_ROOTCERT` instead                                                                          |
| `PREST_LOG_LEVEL`                    |                  | v2 only: log verbosity (`debug`, `info`, `warn`, `error`)                                                                                |
| `PREST_PLUGINPATH`                   | `./lib`          | path to plugin storage `.so`                                                                                                             |
| `PREST_EXPOSE_ENABLED`               | `false`          | when `true`, disables all listing endpoints (`/databases`, `/schemas`, `/tables`). See [Expose Data](#expose-data)                         |
| `PREST_EXPOSE_TABLES`                | `true`           | when `false`, disables table listing. See [Expose Data](#expose-data)                                                                      |
| `PREST_EXPOSE_SCHEMAS`               | `true`           | when `false`, disables schema listing. See [Expose Data](#expose-data)                                                                      |
| `PREST_EXPOSE_DATABASES`             | `true`           | when `false`, disables database listing. See [Expose Data](#expose-data)                                                                   |
| `PREST_JSON_AGG_TYPE`                | `jsonb_agg`      | changes how pREST encodes data from the database, can be set also to `json_agg`                                                          |

### TOML

Optionally the prestd can be configured by TOML file.

You can follow this sample and create your own `prest.toml` file and put this on the same folder that you run `prestd` command.

```toml
migrations = "./migrations"

# debug = true
# enabling debug mode will disable JWT authorization

[http]
port = 3000

[jwt]
default = false
key = "secret"
algo = "HS256"

[auth]
enabled = true
type = "body"
encrypt = "bcrypt"
table = "prest_users"
username = "username"
password = "password"

[pg]
host = "127.0.0.1"
user = "postgres"
pass = "mypass"
port = 5432
database = "prest"
single = true
## or used cloud factor
# URL = "postgresql://user:pass@localhost/mydatabase/?sslmode=disable"

[pg.ssl]
mode = "disable"
cert = "./PATH"
key = "./PATH"
rootcert = "./PATH"

[expose]
enabled = false
databases = true
schemas = true
tables = true
```

### Authorization

#### JWT

JWT middleware is controlled by `jwt.default`. In **v2+**, the default is **`false`**; you must set `jwt.default = true` (or `PREST_JWT_DEFAULT=true`) to enable JWT enforcement. Enabling debug mode disables JWT at runtime.

`/_mcp` (v2.1.0+) uses the same auth stack as other HTTP routes — when JWT or auth is enabled, MCP clients must send the same credentials. See [MCP over HTTP](mcp-over-http.md) and [Auth](../api-reference/auth.md).

```toml
[jwt]
default = true
key = "your-secret"
```

**JWT configuration (v2+):** when `jwt.default = true` and no verification material is provided (`jwt.key`, `jwt.jwks`, or `jwt.wellknownurl`), pREST **auto-disables** the JWT middleware and logs an error — the server continues to start ([#974](https://github.com/prest/prest/pull/974), shipped in v2.0.0). When `auth.enabled = true` without `jwt.key`, the auth endpoint is also auto-disabled. This prevents authentication bypass via an empty HMAC key ([GHSA-fj7v-859r-2fm4](https://github.com/prest/prest/security/advisories/GHSA-fj7v-859r-2fm4)).

> **v2.0.0-rc6 tagged binary:** the [v2.0.0-rc6](../releases/v2.0.0-rc6.md) release **refuses to start** in the same situations. See [Changes since rc6](../releases/main-since-rc6.md) for the difference.

Debug mode bypasses JWT enforcement at runtime.

See [Upgrading to v2](upgrading-to-v2.md) for migration guidance from v1.

The algorithm used is the one contained in the header of the token. The token is then validated with the provided key or JWKS.

The supported signing algorithms are described in the [documentation of the JWK package](https://pkg.go.dev/github.com/lestrrat-go/jwx/jwk#section-readme).

They include the following algorithms:

* The [HMAC signing method](https://en.wikipedia.org/wiki/HMAC): `HS256`, `HS384`, `HS512`
* The [RSA signing method](https://en.wikipedia.org/wiki/RSA_(cryptosystem)): `RS256`, `RS384`, `RS512`
* The [ECDSA signing method](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm): `ES256`, `ES384`, `ES512`

Instead of the key, you could provide the URL of a .well-known OpenID configuration or the JWKS directly through `PREST_JWT_WELLKNOWNURL` or `PREST_JWT_JWKS` as environment variables or by using the TOML configuration file:

```toml
[jwt]
wellknownurl = https://accounts.google.com/.well-known/openid-configuration
jwks = {"keys": [{"kty": "RSA", "alg": "RS256", "kid": "93b495162af0c87cc7a51686294097040daf3b43", "n": "3NXwASNf_7-9hOWDKyZ39qgz-yl_npuIsBgxnhNoE7WyQQl-muajPsQRdFqM-HWsAAbS_WtLrmf2aRSmjXBm8wXHIeJjcrZiWeUnSyfZLDr13jxXhN0rDvdiZEsAlaKuh-iCgwC_pXd0TtWpaYlv5FFguuSitKTOiDR6z3eSZUd0XNxr8POCDQ7VlG_4HyzhsO7nOwgivO-PzekDEbcoLI93U8uzKZXYHSRxYWhoSp47PbM9D5WbuwXqbmXRp9TjiJUy6GqEOJ4K2FNvqe-g6C3BnpPVuHZNaVf8QGP806rWrWPdJ0irGBhg-EasC-sdFSrH3kxMxBFfVsuj69U-7Q", "use": "sig", "e": "AQAB" }]}
```

_**prestd**_ will parse the JWKS to find the corresponding key from the token and fail if it does not find it.

### White list

By default the `/auth` endpoint does not require JWT. The **whitelist** option configures which endpoints skip JWT verification. In v2, entries are **regular expressions**.

```toml
[jwt]
default = true
whitelist = ["^\\/auth$", "^\\/ping$", "^\\/ping\\/.*"]
```

### Auth

pREST has support in jwt token generation based on two fields (example user and password), being possible to use an existing table from your database to login configuring some parameters in the configuration file (or environment variable), _by default this feature is_ **disabled**.

```toml
[auth]
enabled = true
type = "body"
encrypt = "bcrypt"
table = "prest_users"
username = "username"
password = "password"
```

| Name       | Description                                                                                                                                                            |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`  | **Boolean** field that activates or deactivates token generation endpoint support                                                                                      |
| `type`     | Type that will receive the login, support for **body and http basic authentication**                                                                                   |
| `encrypt`  | Type of encryption used in password field. v2 default is `bcrypt`; also supports `MD5` and `SHA1`                                                                     |
| `table`    | Table name we will consult _(query)_                                                                                                                                   |
| `username` | User **field** that will be consulted - if your software uses email just abstract name username (at prestd code level it was necessary to define an internal standard) |
| `password` | Password **field** that will be consulted                                                                                                                              |

> to validate all endpoints with generated jwt token must be activated jwt option

### Expose Data

The expose data settings control access to listing endpoints:

* `/databases`
* `/schemas`
* `/tables`

By default, all listing endpoints are **enabled** (`expose.enabled = false` means listings are allowed).

An example disabling all listings:

```toml
[expose]
enabled = true
```

To disable just the database listing:

```toml
[expose]
databases = false
```

| Name        | Description                                                                        |
| ----------- | ---------------------------------------------------------------------------------- |
| `enabled`   | Set to `true` to **disable** all listing endpoints.                                |
| `databases` | Set to `false` to **disable** _databases_ listing only.                          |
| `schemas`   | Set to `false` to **disable** _schemas_ listing only.                              |
| `tables`    | Set to `false` to **disable** _tables_ listing only.                               |

#### Default values for Exposure Settings

| Name      | Default Value |
| --------- | ------------- |
| enabled   | `false`       |
| databases | `true`        |
| schemas   | `true`        |
| tables    | `true`        |

### SSL

PostgreSQL connection SSL is configured via `[pg.ssl]` in TOML or `PREST_PG_SSL_*` environment variables (v2). The legacy `[ssl]` block and `PREST_SSL_*` variables were removed in v2. See [Upgrading to v2](upgrading-to-v2.md).

There are 4 options to set on ssl mode:

| Name          | Description                      | Comment                                                                                                                       |
| ------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `require`     | Always SSL                       | skips SSL verification step; v1 default                                                                                       |
| `disable`     | SSL off                          | v2 default when no config file is found                                                                                       |
| `verify-ca`   | Always SSL                       | verifies that the certificate presented is signed by a trusted CA                                                             |
| `verify-full` | Always SSL                       | verifies that the certificate presented is signed by a trusted CA and the server host name matches the one in the certificate |

### Debug Mode

Set environment variable `PREST_DEBUG` or `debug=true` in `prest.toml`.

```toml
debug = true
```

Debug mode disables JWT middleware at runtime and bypasses startup JWT validation.

### Logging

v2 uses Go's `slog` package for structured logging. JSON logs are emitted to stdout on **every** startup.

Database credentials are redacted in error logs ([#972](https://github.com/prest/prest/pull/972)).

Set the `PREST_LOG_LEVEL` environment variable to control verbosity:

| Level   | Description |
| ------- | ----------- |
| `debug` | SQL queries and detailed diagnostics |
| `info`  | General operational messages |
| `warn`  | Warnings such as public mode, config fallbacks, or auto-disabled features |
| `error` | Errors only |

### Custom queries storage

Filesystem scripts under `PREST_QUERIES_LOCATION` / `[queries] location` are the default. In **v2.2.0+**, database-backed storage and the `/_QUERIES/registry` admin API are also available — see [Custom Queries](../api-reference/custom-queries.md) and [v2.2.0](../releases/v2.2.0.md). The commented sample in [`samples/prest.sample.toml`](https://github.com/prest/prest/blob/v2.2.0/samples/prest.sample.toml) lists every `[queries]` key.

### pREST Studio

Embedded UI at `/_studio/` (v2.2.0+, enabled by default):

```toml
[studio]
enabled = true
```

Or `PREST_STUDIO_ENABLED=false` to disable. See [pREST Studio](prest-studio.md).

### Configuration resilience

Since **v2.0.0** ([#974](https://github.com/prest/prest/pull/974)), pREST does not abort startup because of configuration problems. Instead it logs warnings and applies safe fallbacks:

| Problem | Fallback |
| ------- | -------- |
| Missing, unreadable, or malformed TOML | Viper defaults and environment overrides |
| Invalid structured keys (`access.tables`, `cache.endpoints`, `databases`) | Zero/empty values + warning |
| Queries path unavailable | Retry `~/queries`; if that fails, disable custom queries |
| Cache storage path unavailable | Retry `./`; if that fails, disable cache |
| Invalid database registry entry | Entry skipped with warning |
| Unsafe JWT/auth config | JWT or auth auto-disabled with error log |

### Multi-database

pREST supports routing to multiple databases or clusters via a database registry. Set `pg.single = false` and configure `[[databases]]` entries or `DATABASE_ALIAS_N` / `DATABASE_URL_N` environment pairs.

See the [Multi-database guide](multi-database.md) for URL routing, TOML examples, Kubernetes setup, and connection pooling. Full sample: [examples/multi-database-config.toml](examples/multi-database-config.toml).

Since **v2.3.0** ([#999](https://github.com/prest/prest/pull/999)), each alias can auto-select a **Postgres or Timescale** adapter (MySQL/SQLite still roadmap).

### CORS support

**Cross-Origin Resource Sharing**

See [CORS Support](cors-support.md).

### Health check endpoints

| Endpoint | Purpose | Behavior |
|----------|---------|----------|
| `GET /_health` | Liveness | Pings the default database. Returns 503 when unhealthy. |
| `GET /_ready` | Readiness | Pings the default database and every registered alias ([#973](https://github.com/prest/prest/pull/973)). Use for Kubernetes readiness probes. |

For multi-database deployments, prefer `/_ready` over `/_health` as the readiness probe. See [Deploying with Docker](../deployment/deploying-with-docker.md).

## Related

- [Multi-database](multi-database.md)
- [examples/multi-database-config.toml](examples/multi-database-config.toml)
- [MCP over HTTP](mcp-over-http.md)
- [pREST Studio](prest-studio.md)
- [Custom Queries](../api-reference/custom-queries.md)
- [Auth](../api-reference/auth.md)
- [v2.2.0 release notes](../releases/v2.2.0.md)
- [Acronyms](../prestd/acronyms.md) · [JWT](../prestd/acronyms.md#jwt) · [MCP](../prestd/acronyms.md#mcp)
