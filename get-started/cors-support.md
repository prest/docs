---
description: >-
  Configure CORS in pREST with [cors]: allowed origins, headers, methods and
  credentials, defaults, env var lists, preflight behaviour and browser errors.
---

# CORS Support

Set `[cors]` so a web app on another origin can call pREST from the browser. CORS (Cross-Origin Resource Sharing) is how the browser asks a server whether a page from `https://app.example.com` may read responses from `https://api.example.com`. Before most cross-origin calls, the browser sends a **preflight** `OPTIONS` request naming the method and headers it wants to use, and only sends the real request if pREST answers with matching `Access-Control-Allow-*` headers.

pREST answers CORS for every route, including `/auth`, `/_QUERIES`, `/_mcp`, `/_health` and Studio. Requests sent from servers, scripts or `curl` are not affected by CORS at all.

{% hint style="warning" %}
**The defaults do not cover a typical browser app.** Out of the box pREST allows any origin but only the `Content-Type` request header, and does not allow `PATCH`. A browser that sends a JWT in `Authorization`, or that calls `PATCH`, is refused until you add them. See [A browser app that sends a JWT](#a-browser-app-that-sends-a-jwt).
{% endhint %}

---

## Settings

| TOML key | Environment variable | Default | Description |
| -------- | -------------------- | ------- | ----------- |
| `cors.alloworigin` | `PREST_CORS_ALLOWORIGIN` | `["*"]` | Origins allowed to read responses. `"*"` allows every origin. An entry can hold one `*` wildcard, such as `https://*.example.com` |
| `cors.allowheaders` | `PREST_CORS_ALLOWHEADERS` | `["Content-Type"]` | Request headers a preflight may ask for. `"*"` allows every header |
| `cors.allowmethods` | `PREST_CORS_ALLOWMETHODS` | `["GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS"]` | Methods a preflight may ask for. `PATCH` is **not** in the default |
| `cors.allowcredentials` | `PREST_CORS_ALLOWCREDENTIALS` | `true` | Send `Access-Control-Allow-Credentials: true`. See [Credentials](#credentials) |

There is no setting for `Access-Control-Max-Age` or `Access-Control-Expose-Headers`. Browsers therefore cache a preflight only briefly (5 seconds by default in the Fetch standard), and page scripts can read only the [CORS-safelisted response headers](https://developer.mozilla.org/en-US/docs/Glossary/CORS-safelisted_response_header) such as `Content-Type`.

### Environment variable lists

The three list variables are split on **whitespace**, not commas:

```shell
PREST_CORS_ALLOWORIGIN="https://app.example.com https://admin.example.com"
PREST_CORS_ALLOWHEADERS="Content-Type Authorization"
PREST_CORS_ALLOWMETHODS="GET HEAD POST PUT PATCH DELETE OPTIONS"
```

`PREST_CORS_ALLOWORIGIN=https://app.example.com,https://admin.example.com` is read as **one** origin containing a comma, which matches no browser, so every cross-origin call is refused. This differs from `PREST_HTTP_ALLOWED_HOSTS`, which is comma-separated. An environment variable replaces the whole list from `prest.toml`; it does not add to it.

[`PREST_CONFIG_JSON`](configuring-prest.md#configuration-as-json) (prest `main`, unreleased) also accepts a `cors` object with JSON arrays.

---

## Examples

### A browser app that sends a JWT

One single-page app on its own origin, calling every table route with a token in `Authorization`:

```toml
[cors]
alloworigin = ["https://app.example.com"]
allowheaders = ["Content-Type", "Authorization"]
allowmethods = ["GET", "HEAD", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
```

With this, a preflight for `PATCH` with `Authorization` and `Content-Type` is answered like this:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: authorization,content-type
Access-Control-Allow-Methods: PATCH
Access-Control-Allow-Origin: https://app.example.com
Vary: Origin, Access-Control-Request-Method, Access-Control-Request-Headers
```

Add every custom header your app sends (for example `X-Request-Id`) to `allowheaders`, or use `allowheaders = ["*"]`.

### Subdomains

```toml
[cors]
alloworigin = ["https://*.example.com", "http://localhost:5173"]
```

`https://*.example.com` matches `https://admin.example.com` and `https://a.b.example.com`, but not `https://example.com`. List the apex origin separately if you need it.

### Any origin

The default, `alloworigin = ["*"]`, answers every origin with `Access-Control-Allow-Origin: *`. This is fine for a public, read-only API called without cookies. Do not combine it with credentialed requests; see [Credentials](#credentials).

---

## How origins are matched

* An origin is the scheme, host and port the page was loaded from: `https://app.example.com`, `http://localhost:5173`. There is no path and no trailing slash. `https://app.example.com/` does not match.
* Matching ignores case.
* An entry can contain one `*`, which matches any text in that position: `https://*.example.com`, `http://localhost:*`.
* `"*"` anywhere in the list allows every origin, and the other entries are ignored.
* An empty list, `alloworigin = []`, also allows **every** origin. The `[cors]` settings cannot turn CORS off.
* When the origin is allowed by a list entry, pREST echoes it back in `Access-Control-Allow-Origin`. When the list is `"*"`, it answers `*`.

Header names in `allowheaders` are matched without regard to case. Methods in `allowmethods` must be written in uppercase.

---

## Credentials

`allowcredentials = true` (the default) adds `Access-Control-Allow-Credentials: true` to allowed responses. Browsers need it when the page sends cookies or HTTP authentication, that is, `fetch(url, { credentials: "include" })` or `xhr.withCredentials = true`.

Browsers refuse a credentialed response whose `Access-Control-Allow-Origin` is `*`. With the defaults (`alloworigin = ["*"]` and `allowcredentials = true`) pREST sends exactly that combination, so a credentialed call fails in the browser. List the real origins in `alloworigin` instead; pREST then echoes the caller's origin and the browser accepts it.

A token in the `Authorization` header that your code sets by hand is **not** a browser credential. It needs `Authorization` in `allowheaders`, but it works with `alloworigin = ["*"]` as long as the request does not use `credentials: "include"`.

Set `allowcredentials = false` if your app never sends cookies, so the header is never sent.

---

## What pREST answers

| Request | Origin and request allowed | Origin, method or header not allowed |
| ------- | -------------------------- | ------------------------------------ |
| Preflight (`OPTIONS` with `Access-Control-Request-Method`) | `204`, with `Access-Control-Allow-Origin`, `-Methods`, `-Headers` and, if enabled, `-Credentials` | `204` **without** any `Access-Control-Allow-*` header. The browser reports a CORS error and never sends the real request |
| Any other request | The normal response, plus `Access-Control-Allow-Origin` and, if enabled, `Access-Control-Allow-Credentials` | The normal response, **without** CORS headers. The request still runs; the browser only hides the response from the page |

Both kinds of response carry a `Vary` header naming `Origin`, so shared caches keep one copy per origin.

{% hint style="danger" %}
**CORS is not access control.** A request from an origin you did not list still reaches pREST, runs, and returns data. Only the browser stops the page from reading it, and non-browser clients ignore CORS entirely. Protect data with [JWT auth](../api-reference/auth.md) and [Permissions](permissions.md).
{% endhint %}

### Order with other checks

* [`http.allowed_hosts`](configuring-prest.md#allowed-hosts) (prest `main`, unreleased) runs first. A request for a host that is not listed gets `400` before CORS, so its preflight fails too.
* CORS runs before JWT. Preflights carry no token, and pREST answers them without asking for one, even when `jwt.default = true`.
* A `401` from the JWT check still includes the CORS headers, so the page can read the error body.

---

## Troubleshooting

Look at the browser console and at the network tab's `OPTIONS` request.

| Browser message | Cause | Fix |
| --------------- | ----- | --- |
| `Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present` | The preflight was refused: the origin, a header, or the method is not allowed | Compare the preflight's `Origin`, `Access-Control-Request-Method` and `Access-Control-Request-Headers` with `[cors]`. The usual culprits are `Authorization` missing from `allowheaders` and `PATCH` missing from `allowmethods` |
| `The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'` | Credentialed request with `alloworigin = ["*"]` | List the origins in `alloworigin`, or stop sending credentials. See [Credentials](#credentials) |
| CORS error although the origin is listed, set through an environment variable | Origins were separated with commas | Separate them with spaces. See [Environment variable lists](#environment-variable-lists) |
| Changes to `[cors]` in `prest.toml` have no effect, for example `Authorization` is still refused | The file was not loaded, so the defaults apply | Check the startup log for `config file unavailable`. Run `prestd` from the directory holding `prest.toml`, or set `PREST_CONF` to its path |
| CORS error and the preflight answered `400` `invalid Host header` | `http.allowed_hosts` does not list the host the browser called | Add that host. See [Allowed hosts](configuring-prest.md#allowed-hosts) |
| Mixed-content error, not a CORS error | An `https://` page called pREST over `http://` | Serve pREST over HTTPS. See [Serving over HTTPS](../deployment/serving-over-https.md) |

You can reproduce a preflight with `curl`:

```shell
curl -i -X OPTIONS http://localhost:3000/prest/public/my_table \
  -H "Origin: https://app.example.com" \
  -H "Access-Control-Request-Method: PATCH" \
  -H "Access-Control-Request-Headers: authorization,content-type"
```

An allowed preflight lists `Access-Control-Allow-Origin` in the response. A refused one has only the `Vary` header.

---

## Related

* [Configuring pREST](configuring-prest.md)
* [Auth](../api-reference/auth.md)
* [Permissions](permissions.md)
* [Serving over HTTPS](../deployment/serving-over-https.md)
* [Acronyms — HTTP](../prestd/acronyms.md#http) · [JWT](../prestd/acronyms.md#jwt)
