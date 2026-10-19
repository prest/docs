# Deploying with Docker

This guide assumes you already have a PostgreSQL (or [Postgres-compatible](../databases/README.md)) database running and helps you set up _**prestd**_ with Docker.

In case you’d like to run _**prestd**_ with a fresh PostgreSQL database, follow this guide to deploy _**prestd**_ along with a Postgres instance using _Docker Compose_.

### Prerequisites

* [Docker](https://docs.docker.com/get-docker/) (version 20.10.7 or later)
* [Docker Compose](https://docs.docker.com/compose/install/) (version 1.29.2 or later)

Create an installation folder called `prestd` where you would like your `prestd` installation and data storage.

**`cd`** (open/join) into the installation folder.

***

### Quick Start

We will use docker to run pREST and connect to an existing database. To simplify the example we leave the authentication module off and enable debug mode (which disables JWT enforcement).

```shell
docker run -d -p 3000:3000 \
    -e PREST_VERSION=2 \
    -e PREST_PG_URL=postgres://username:password@hostname:port/dbname \
    -e PREST_DEBUG=true \
    prest/prest:v2.5.2
```

> **v2 JWT requirement**: when `PREST_DEBUG` is not set and `jwt.default` is enabled, configure `PREST_JWT_KEY` (or `PREST_JWT_JWKS` / `PREST_JWT_WELLKNOWNURL`) explicitly. In **v2+**, missing verification material auto-disables JWT with a warning — the server still starts. The **v2.0.0-rc6** tagged binary refuses to start in that case. See [Configuring pREST](../get-started/configuring-prest.md#jwt).

> **Docker images**: v2 images are built via GoReleaser. Pin to a specific tag like `v2.5.2` rather than using `latest` in production. Images are linux/amd64 and linux/arm64 from v2.5.1. MCP over HTTP (`/_mcp`) is available on v2.1.0+; Studio (`/_studio/`) on v2.2.0+; MySQL (`engine = "mysql"`) on v2.5.0+.

Edit the `PREST_PG_URL` env var value, so that you can connect to your Postgres instance.

Examples of `PREST_PG_URL`:

* postgres://admin:password@localhost:5432/my-db
* postgres://admin:@localhost:5432/my-db _(if there is no password)_

> If your password contains special characters (e.g. #, %, $, @, etc.), you need to URL encode them in the `PREST_PG_URL` env var (e.g. %40 for @). You can check the logs to see if the database credentials are proper and if pREST is able to connect to the database. pREST needs access permissions to your Postgres database as described in permissions page.

#### Network config

If your Postgres instance is running on `localhost`, the following changes will be needed to the `docker run` command to allow the Docker container to access the host’s network.

Add the `--net=host` flag to access the host’s Postgres service.

This is what your command should look like:

```shell
docker run -d --net=host -p 3000:3000 \
    -e PREST_PG_URL=...
```

> if you are using another operating system we recommend reading the [docker network documentation](https://docs.docker.com/network/host/), on **macOS** and **Windows** it is different.

### With Docker Compose

> Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the [list of features](https://docs.docker.com/compose/#features).

Compose file in the prest repo: [docker-compose-prod.yml](https://github.com/prest/prest/blob/main/docker-compose-prod.yml)

**Download docker compose file**

```sh
wget https://raw.githubusercontent.com/prest/prest/main/docker-compose-prod.yml -O docker-compose.yml
```

**Up (run) PostgreSQL and prestd**

```sh
docker-compose up
```

**Run data migration to create user structure for access (JWT)**

```sh
docker-compose exec prest prestd migrate up auth
```

**Create user and password for API access (via JWT)**

* **user:** prest
* **pass:** prest

v2 defaults `auth.encrypt` to `bcrypt`. Hash the password with `htpasswd` (apache2-utils / httpd) on the host, then insert:

```sh
HASH=$(htpasswd -nbBC 10 x prest | cut -d: -f2)
docker-compose exec postgres psql -d prest -U prest -c "INSERT INTO prest_users (name, username, password) VALUES ('pREST Full Name', 'prest', '$HASH')"
```

**Check if the user was created successfully (by doing a select on the table)**

```sh
docker-compose exec postgres psql -d prest -U prest -c "select * from prest_users"
```

### First call on API

With auth enabled, get a token first. From v2.5.2 the catalog routes (`/databases`, `/schemas`, `/tables`) also require it — see [Auth](../api-reference/auth.md#routes-that-require-auth-v252).

```sh
# Generate JWT Token with user and password created
curl -i -X POST http://127.0.0.1:3000/auth -H "Content-Type: application/json" -d '{"username": "prest", "password": "prest"}'
# List the databases this instance serves
curl -i -X GET http://127.0.0.1:3000/databases -H "Authorization: Bearer {TOKEN}"
# Access endpoint using JWT Token
# (prest_users itself is reserved and returns 401 from v2.5.2 — query one of your tables)
curl -i -X GET "http://127.0.0.1:3000/prest/public/{TABLE}?_page_size=5" -H "Accept: application/json" -H "Authorization: Bearer {TOKEN}"
# Optional: MCP discovery (v2.1.0+)
curl -s http://127.0.0.1:3000/_mcp -H "Authorization: Bearer {TOKEN}" | head
```

{% hint style="warning" %}
**Use a least-privilege database role in production.** The sample compose files connect as a PostgreSQL superuser for convenience. Give pREST a role without superuser, `pg_read_server_files`, or `pg_execute_server_program`, and grant it only the schemas it serves. A non-superuser role cannot run `COPY ... FROM PROGRAM`, which bounds what any SQL-execution bug can reach ([GHSA-2x9q-r7j9-g85f](https://github.com/prest/prest/security/advisories/GHSA-2x9q-r7j9-g85f)).
{% endhint %}

### Kubernetes readiness probe

For multi-database or production deployments, use `GET /_ready` as the readiness probe. It pings the default database and every registered alias ([#973](https://github.com/prest/prest/pull/973)), so it returns **503** while any alias is down — including one registered lazily because it was unreachable at startup (v2.5.2).

```yaml
readinessProbe:
  httpGet:
    path: /_ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 10
livenessProbe:
  httpGet:
    path: /_health
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 30
```

See the [Kubernetes deployment manifest](https://github.com/prest/prest/blob/main/install-manifests/kubernetes/deployment.yaml) in the prest repo for a multi-secret example. From v2.5.2 it uses `prest/prest:v2` and shows `PREST_DB_URL` / `PREST_PG_ALLOWED_DATABASES`; pin a specific tag such as `v2.5.2` in production.

## Related

- [Start with Docker](../get-prest/start-with-docker.md)
- [Upgrading to v2](../get-started/upgrading-to-v2.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Acronyms](../prestd/acronyms.md)
