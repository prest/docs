---
description: >-
  Serve pREST over HTTPS with [https]: certificate and key settings, self-signed
  local certs, Let's Encrypt, Docker, reverse proxies, and TLS error fixes.
---

# Serving pREST over HTTPS

Set `[https] mode = true` and point `cert` and `key` at PEM files, and `prestd` serves every route over HTTPS on its normal port. You can also leave pREST on plain HTTP and let a reverse proxy terminate TLS (Transport Layer Security) in front of it. Both setups are described below.

{% hint style="info" %}
`[https]` secures the connection **from clients to pREST**. `[pg.ssl]` secures the connection **from pREST to PostgreSQL**. They are independent, and setting one never affects the other. See [HTTPS versus database SSL](#https-versus-database-ssl).
{% endhint %}

---

## Settings

| TOML key | Environment variable | Default | Description |
| -------- | -------------------- | ------- | ----------- |
| `https.mode` | `PREST_HTTPS_MODE` | `false` | `true` serves HTTPS instead of HTTP |
| `https.cert` | `PREST_HTTPS_CERT` | `/etc/certs/cert.crt` | PEM certificate. When a CA issued it, include the intermediate certificates after the server certificate (the "full chain") |
| `https.key` | `PREST_HTTPS_KEY` | `/etc/certs/cert.key` | PEM private key for that certificate, unencrypted (no passphrase) |

```toml
[http]
host = "0.0.0.0"
port = 443

[https]
mode = true
cert = "/etc/prest/certs/fullchain.pem"
key = "/etc/prest/certs/privkey.pem"
```

How the listener behaves:

* HTTPS uses the address from `http.host` and `http.port` (or `PORT`). pREST opens **one** listener, so turning HTTPS on turns plain HTTP off. There is no redirect from HTTP to HTTPS.
* The certificate and key are read **once, at startup**. After you renew or replace them, restart `prestd`.
* If a file is missing, unreadable, or invalid, `prestd` logs `HTTP server failed` and exits with status 1. See [Troubleshooting](#troubleshooting).
* TLS versions and ciphers follow Go's server defaults and are not configurable: TLS 1.2 and newer, with HTTP/2 negotiated for clients that support it.
* pREST does not request or verify client certificates (mutual TLS).

---

## Local HTTPS with a self-signed certificate

This works on macOS and Linux with OpenSSL 1.1.1 or newer.

{% stepper %}
{% step %}
### Create a certificate for `localhost`

```shell
mkdir -p certs
openssl req -x509 -newkey rsa:2048 -nodes -days 365 \
  -keyout certs/key.pem -out certs/cert.pem \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

Browsers and `curl` check the `subjectAltName`, not the `CN`, so keep the `-addext` line. If your `openssl` rejects `-addext` (the LibreSSL bundled with older macOS releases does), use Homebrew's `openssl@3`.
{% endstep %}

{% step %}
### Point pREST at it

```toml
[https]
mode = true
cert = "./certs/cert.pem"
key = "./certs/key.pem"
```

Or, without editing `prest.toml`:

```shell
PREST_HTTPS_MODE=true \
PREST_HTTPS_CERT=./certs/cert.pem \
PREST_HTTPS_KEY=./certs/key.pem \
prestd
```
{% endstep %}

{% step %}
### Call it

```shell
curl --cacert certs/cert.pem https://localhost:3000/databases
```

`--cacert` tells `curl` to trust your certificate. Browsers will warn until you trust it in the OS keychain (on macOS: Keychain Access → System → import `cert.pem` → **Always Trust**). Locally trusted CA tools such as [mkcert](https://github.com/FiloSottile/mkcert) do this for you and print a cert/key pair you can use the same way.
{% endstep %}
{% endstepper %}

---

## Let's Encrypt

Certbot writes the files pREST needs under `/etc/letsencrypt/live/<domain>/`:

```toml
[https]
mode = true
cert = "/etc/letsencrypt/live/api.example.com/fullchain.pem"
key = "/etc/letsencrypt/live/api.example.com/privkey.pem"
```

* Use `fullchain.pem`, not `cert.pem`. `cert.pem` lacks the intermediate certificate, so many clients reject the connection.
* There is no root certificate setting for `[https]`. `rootcert` exists only under `[pg.ssl]`, where it verifies the **database** server.
* By default `privkey.pem` is readable only by root. Run `prestd` as a user that can read it, or have a deploy hook copy the pair somewhere that user can read.
* pREST does not reload certificates, so restart it after each renewal, for example with `certbot renew --deploy-hook "systemctl restart prestd"`.

---

## Docker

Mount the certificate directory and set the `PREST_HTTPS_*` variables. The container listens on `3000` by default; publish it on `443`:

```shell
docker run -d -p 443:3000 \
    -v /etc/prest/certs:/etc/prest/certs:ro \
    -e PREST_VERSION=2 \
    -e PREST_PG_URL=postgres://username:password@hostname:port/dbname \
    -e PREST_HTTPS_MODE=true \
    -e PREST_HTTPS_CERT=/etc/prest/certs/fullchain.pem \
    -e PREST_HTTPS_KEY=/etc/prest/certs/privkey.pem \
    prest/prest:v2.5.2
```

* `-noplugins` images run as the unprivileged `prest` user, so the mounted key must be readable by that user. Otherwise startup fails with `permission denied`.
* `/etc/letsencrypt/live/<domain>/` holds symlinks into `/etc/letsencrypt/archive/`. Mount all of `/etc/letsencrypt` (read-only) rather than the `live/<domain>` directory alone, or the links will not resolve inside the container.
* Restart the container after a renewal: `docker restart <container>`.

Outside Docker, binding a port below 1024 such as `443` on Linux needs root or the `CAP_NET_BIND_SERVICE` capability (`sudo setcap 'cap_net_bind_service=+ep' "$(command -v prestd)"`).

---

## Behind a reverse proxy

When a load balancer, ingress, or proxy (nginx, Caddy, Traefik, a cloud load balancer) already handles certificates and renewal, let it terminate TLS and keep pREST on plain HTTP. Leave `https.mode = false` and bind pREST to an address only the proxy can reach:

```toml
[http]
host = "127.0.0.1"
port = 3000
```

A minimal nginx server block:

```nginx
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The proxy handles certificate renewal and reloads, so pREST never needs a restart for them.

---

## Calling pREST from an HTTPS page

Browsers block `http://` requests made by a page loaded over `https://` (mixed content). If a web app, wiki, or dashboard served over HTTPS calls pREST from the browser, pREST must be reachable over HTTPS, either directly or through a proxy. The page's origin must also be allowed in [CORS](../get-started/cors-support.md):

```toml
[cors]
alloworigin = ["https://wiki.example.com"]
```

---

## HTTPS versus database SSL

| | `[https]` | `[pg.ssl]` |
| - | --------- | ---------- |
| Secures | Clients → pREST | pREST → PostgreSQL |
| Environment variables | `PREST_HTTPS_MODE`, `PREST_HTTPS_CERT`, `PREST_HTTPS_KEY` | `PREST_PG_SSL_MODE`, `PREST_PG_SSL_CERT`, `PREST_PG_SSL_KEY`, `PREST_PG_SSL_ROOTCERT` |
| Certificate is | pREST's server certificate | An optional client certificate pREST presents to PostgreSQL, plus the root CA used to verify PostgreSQL |
| Details | This page | [Configuring pREST — SSL](../get-started/configuring-prest.md#ssl) |

In v1 the PostgreSQL settings lived in a top-level `[ssl]` block (`PREST_SSL_*`), which was easy to mistake for HTTPS. v2 renamed it to `[pg.ssl]`. See [Upgrading to v2](../get-started/upgrading-to-v2.md).

---

## Troubleshooting

| Symptom | Cause | Fix |
| ------- | ----- | --- |
| Firefox shows `SSL_ERROR_RX_RECORD_TOO_LONG`, Chrome `ERR_SSL_PROTOCOL_ERROR`; `curl` shows `wrong version number` | The client speaks HTTPS but pREST is serving plain HTTP | Set `https.mode = true`. If it is set in `prest.toml`, check the startup log for `config file unavailable`, which means the file was not loaded: run `prestd` from the directory that holds `prest.toml` or set `PREST_CONF` to its path |
| Response body `Client sent an HTTP request to an HTTPS server.` | The client used `http://` against an HTTPS listener | Use `https://` |
| `HTTP server failed` … `open /etc/certs/cert.crt: no such file or directory` | `https.mode` is on but `cert` / `key` were not set, so the defaults are used | Set `https.cert` and `https.key` (or `PREST_HTTPS_CERT` / `PREST_HTTPS_KEY`) |
| `HTTP server failed` … `permission denied` | The user running `prestd` cannot read the key | Fix the file's owner or permissions, or copy it somewhere readable |
| `HTTP server failed` … `PEM inputs may have been switched` | `cert` and `key` are swapped | Put the certificate in `cert` and the private key in `key` |
| `curl: (60) SSL certificate problem: self-signed certificate` | The client does not trust the certificate | Pass `--cacert`, trust the certificate in the OS, or use a CA-issued certificate |
| `initializing app` … `SSL is not enabled on the server` | A **database** TLS error. `[pg.ssl] mode` (or `PREST_PG_SSL_MODE`) asks for TLS but PostgreSQL does not offer it | Enable TLS on PostgreSQL, or set `mode = "disable"` under `[pg.ssl]` (`PREST_PG_SSL_MODE=disable`). This is unrelated to `[https]` |

---

## Related

* [Configuring pREST](../get-started/configuring-prest.md)
* [Deploying with Docker](deploying-with-docker.md)
* [CORS Support](../get-started/cors-support.md)
* [Upgrading to v2](../get-started/upgrading-to-v2.md)
* [Acronyms — TLS](../prestd/acronyms.md#tls) · [HTTP](../prestd/acronyms.md#http)
