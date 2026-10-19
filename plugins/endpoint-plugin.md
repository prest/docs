---
description: >-
  Add custom pREST endpoints with Go plugins at /_PLUGIN/{file}/{func} —
  naming, handler signatures, build flags, name rules, and auth (v2.5.2).
---

# Endpoint Plugin

With prestd's HTTP plugin system you can add your own endpoints under `/_PLUGIN/{file}/{func}`. Any HTTP method is routed there. The route has no table ACL; with `auth.enabled = true` it requires a bearer token (v2.5.2).

### Naming patterns

The plugin endpoint (`/_PLUGIN/{file}/{func}`) receives two parameters:

* **File name:** The name of the file without the extension (`{file}`)
* **Function name:** The name of the function (`{func}`)

### Name rules and auth (v2.5.2)

Up to v2.5.1, `{file}` could traverse out of `pluginpath` and load any `.so` on disk, and the route had no auth of its own ([GHSA-fmr6-9j48-xrqp](https://github.com/prest/prest/security/advisories/GHSA-fmr6-9j48-xrqp)). From v2.5.2:

* `{file}` must be letters, digits, `_`, or `-`, 1–63 characters, with **no dots**.
* `{func}` must be a Go identifier.
* `{pluginpath}/{file}.so` must stay inside `pluginpath`, after symlinks are resolved. A symlink that points outside is rejected.
* With `auth.enabled = true`, the route requires a token. **With auth off it is open to any caller**, so only deploy libraries you trust anyone to call.

#### File name

The library is `{pluginpath}/{file}.so` (default `./lib/{file}.so`). It is loaded when it **receives the first request**.

> After the **first access the library will not be loaded again**, it will only be executed, i.e., if the file (`.so`) is changed after the first execution it will have no effect because it has already been loaded.

Calls into one library are serialized, because `HTTPVars` and `URLQuery` are package-level variables. When the [cache](../get-started/cache.md) is enabled, the response is stored under the request URL.

#### Function name

When talking about a compiled _library_ we have no way of identifying its functions. Given this characteristic we have defined some name and behavior patterns to develop _libraries_ for _**prestd**_.

**function name:** `{HTTP Method}{Function Name}Handler`

* `{HTTP Method}`: The HTTP method that the function will be called for (in upper case letters)
* `{Function Name}`: The name of the function that will be called
* `Handler`: The suffix of the function name - it is always `Handler`

> `fmt.Sprintf("%s%sHandler", r.Method, funcName)`

#### Handler signature

| Signature | Response status |
|-----------|-----------------|
| `func() string` | `200` |
| `func() (string, int)` | The returned `int` |

The returned string is written as the response body. The plugin **must** also export `HTTPVars map[string]string` and `URLQuery map[string][]string`; pREST fills them before each call. If either is missing or has another type, the request fails with `404`.

### Example

* **Source code name:** `./lib/src/hello.go`
* **Library file name:** `./lib/hello.so`
* **Function names:** `GETHelloHandler`, `GETHelloWithStatusHandler`
* **Endpoints:** `/_PLUGIN/hello/Hello`, `/_PLUGIN/hello/HelloWithStatus`
* **Verb HTTP:** `GET`

```go
// all plugins must have their package name as `main`
// each plugin is isolated at compile time
package main

import (
 "encoding/json"
 "net/http"
)

var (
 // HTTPVars route variables for the current request
 HTTPVars map[string]string
 // URLQuery parses RawQuery and returns the corresponding values
 URLQuery map[string][]string
)

// Response return structure of the get method
type Response struct {
 HTTPVars map[string]string   `json:"http_vars"`
 URLQuery map[string][]string `json:"url_query"`
 MSG      string              `json:"msg"`
}

// GETHelloHandler plugin
// function is invoked via [go language plugin](https://pkg.go.dev/plugin),
// it is not possible to pass parameters, that's why there are global
// variables to receive data from http protocol
//
// BUILD (same flags as prestd; see Plugins — Building plugins):
// go build -mod=vendor -trimpath -ldflags "-s -w" -buildmode=plugin -o lib/hello.so lib/src/hello.go
func GETHelloHandler() (ret string) {
 resp := Response{
  HTTPVars: HTTPVars,
  URLQuery: URLQuery,
  MSG:      "Hello plugin caller!",
 }
 respJSON, err := json.Marshal(resp)
 if err != nil {
  return
 }
 ret = string(respJSON)
 return
}

// GETHelloWithStatusHandler plugin
// same function as GETHelloHandler, but this time we can return status code.
func GETHelloWithStatusHandler() (ret string, code int) {
	resp := Response{
		HTTPVars: HTTPVars,
		URLQuery: URLQuery,
		MSG:      "Hello plugin caller!",
	}
	respJSON, err := json.Marshal(resp)
	if err != nil {
		return
	}
	ret = string(respJSON)
	code = http.StatusAccepted
	return
}
```

The build must match the `prestd` binary — see [Plugins — Building plugins](README.md#building-plugins). In the Docker image, put the file in `/app/lib/src` and it is built at start.

**Request:**

```http
GET /_PLUGIN/hello/Hello?abc=123 HTTP/1.1
```

**Response:**

```json
{
  "http_vars": {
    "file": "hello",
    "func": "Hello"
  },
  "url_query": {
    "abc": [
      "123"
    ]
  },
  "msg": "Hello plugin caller!"
}
```

**Request with a status code:**

```http
GET /_PLUGIN/hello/HelloWithStatus HTTP/1.1
```

**Response:** `HTTP/1.1 202 Accepted`, with the same body shape (`"func": "HelloWithStatus"`, empty `url_query`).

### Errors

| Response | Cause |
|----------|-------|
| `400` `{"error": "invalid plugin name"}` | `{file}` or `{func}` breaks the [name rules](#name-rules-and-auth-v252), or the library resolves outside `pluginpath` (v2.5.2). The rejected name is not echoed. |
| `401` `{"error":"authorization token is empty"}` | `auth.enabled = true` and no token was sent (v2.5.2). |
| `404` with the error text | The `.so` file, the `{METHOD}{func}Handler` symbol, or `HTTPVars` / `URLQuery` is missing, or the handler has another signature. |

## Related

- [Plugins](README.md)
- [Middleware Plugin](middleware-plugin.md)
- [Auth — Routes that require auth](../api-reference/auth.md#routes-that-require-auth-v252)
- [v2.5.2 release notes](../releases/v2.5.2.md)
- [Acronyms](../prestd/acronyms.md) · [HTTP](../prestd/acronyms.md#http) · [JSON](../prestd/acronyms.md#json)
