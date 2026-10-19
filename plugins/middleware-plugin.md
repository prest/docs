---
description: >-
  Add your own pREST middleware with Go plugins — configure
  pluginmiddlewarelist, naming, where it runs, and how to verify it loaded.
---

# Middleware Plugin

With prestd's middleware plugin system you can add middleware that runs before the CRUD table handlers. From **v2.5.0** ([#1039](https://github.com/prest/prest/pull/1039)), plugin builds use the server's Go module (including vendor in the Docker image), and a missing plugin falls back to no-op middleware.

Before v2.5.0 the documented `func() negroni.Handler` factory never loaded ([#948](https://github.com/prest/prest/issues/948)). It is now the preferred shape; the direct `func(http.ResponseWriter, *http.Request, http.HandlerFunc)` shape is still accepted. A factory that returns `nil` is rejected.

### Configuration

```toml
pluginpath = "./lib"   # top-level key; middleware loads from ./lib/middlewares

[[pluginmiddlewarelist]]
file = "hello"
func = "Hello"
```

Configure the list in TOML. An invalid value logs a warning and leaves the list empty.

{% hint style="warning" %}
**Only the first entry that loads is used.** pREST tries `[[pluginmiddlewarelist]]` entries in order. A failed entry is logged (`unable to load middleware plugin function`) and skipped; the first one that loads becomes the middleware and later entries are ignored. To run several middlewares, compose them inside one plugin.
{% endhint %}

### Where it runs

* Only on CRUD table routes — `/{database}/{schema}/{table}` and `/batch/...` — after auth, access control, exposure, and cache.
* Not on `/_QUERIES`, `/_PLUGIN`, `/_mcp`, the catalog routes, `/auth`, or `/_health`.
* Loaded once when the server starts. If no entry loads, a no-op is used and the server still starts.

### Naming patterns

The prestd configuration file receives two parameters:

* **File name:** `file` in the config. The library is loaded from `{pluginpath}/middlewares/{file}.so` (default `./lib/middlewares/`).
* **Function name:** `func` in the config. It is the prefix of the exported symbol, `{func}MiddlewareLoad`.

From v2.5.2 the same name rules as [endpoint plugins](endpoint-plugin.md#name-rules-and-auth-v252) apply: `file` is letters, digits, `_`, or `-` with no dots, `func` is a Go identifier, and the library must resolve inside `pluginpath/middlewares`. An entry that breaks them is logged and skipped.

#### Filename

The file name will be used in the middleware to identify which library will be loaded **when the server (prestd) loads**.

> After the **server loads, the library is not loaded again**, it is just executed, i.e. if the file (`.so`) is changed after the server loads, the changes are not applied.

#### Function name

When talking about a compiled _library_ we have no way of identifying its functions. Given this characteristic we have defined some name and behavior patterns to develop _libraries_ for _**prestd**_.

**function name:** `{Function Name}MiddlewareLoad`

* `{Function Name}`: The name of the function that will be called
* `MiddlewareLoad`: The suffix of the function name - the function returns `negroni.Handler` (an interface; use `negroni.HandlerFunc` as adapter)

> `fmt.Sprintf("%sMiddlewareLoad", funcName)`

### Example

* **Source code name:** `./lib/src/middlewares/hello.go`
* **Library file name:** `./lib/middlewares/hello.so`
* **Function name:** `HelloMiddlewareLoad`

```go
// nolint
// all plugins must have their package name as `main`
// each plugin is isolated at compile time
package main

import (
	"net/http"

	"github.com/urfave/negroni/v3"
)

// BUILD:
// go build -mod=vendor -trimpath -ldflags "-s -w" \
//   -o ./lib/middlewares/hello.so \
//   -buildmode=plugin ./lib/src/middlewares/hello.go
func HelloMiddlewareLoad() negroni.Handler {
	return negroni.HandlerFunc(func(rw http.ResponseWriter, rq *http.Request, next http.HandlerFunc) {
		rw.Header().Add("X-Hello-Middleware", "Hello Middleware")
		next(rw, rq)
	})
}
```

The build must match the `prestd` binary — see [Plugins — Building plugins](README.md#building-plugins).

### Verify

Call any table route and check the header:

```sh
curl -i http://127.0.0.1:3000/prest/public/{table}
# X-Hello-Middleware: Hello Middleware
```

If the header is missing, look for `unable to load middleware plugin function` in the startup log.

## Related

- [Plugins](README.md)
- [Endpoint Plugin](endpoint-plugin.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [v2.5.0 release notes](../releases/v2.5.0.md)
- [Acronyms](../prestd/acronyms.md) · [HTTP](../prestd/acronyms.md#http)
