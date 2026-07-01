# Amazon Redshift

Analyze all of your data with the fastest and most widely used cloud data warehouse.

> Amazon Redshift is based on PostgreSQL. Amazon Redshift and PostgreSQL have a number of very important differences that you must be aware of as you design and develop your data warehouse applications. [read more](https://docs.aws.amazon.com/redshift/latest/dg/c\_redshift-and-postgres-sql.html)

Amazon Redshift is compatible with postgresql just use the ODBC connection in the environment variable `DATABASE_URL` with the parameter `OpenSourceSubProtocolOverride` to make pREST connect to the database.

```sh
export PREST_VERSION=2
DATABASE_URL=postgresql://localhost:5432/postgres?OpenSourceSubProtocolOverride=true
```

> **v2 notes:** set `PREST_VERSION=2` for v2 deployments. The deprecated `PREST_SSL_*` variables were removed in v2 — use `PREST_PG_SSL_*` instead. When `PREST_DEBUG` is not set, you must provide JWT configuration (`PREST_JWT_KEY`, `PREST_JWT_JWKS`, or `PREST_JWT_WELLKNOWNURL`) or the server will refuse to start. See [Upgrading to v2](../get-started/upgrading-to-v2.md).
