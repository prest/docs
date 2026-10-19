---
description: >-
  Extend pREST with Go plugins (.so) for custom endpoints and middleware — how
  to build them so prestd can load them, Docker usage, and v2.5.2 name rules.
---

# Plugins

Extend pREST with OS shared libraries (`.so`) for custom middleware and endpoints. Plugins load beside the built-in REST and MCP surfaces — they do not replace them.

[Here](https://github.com/prest/prest/discussions/466#discussion-30623) is a discussion of how we arrived at this architecture.

Write a shared library and configure pREST to load it. pREST uses the [Go plugin system](https://pkg.go.dev/plugin); on platforms where Go plugins are unavailable (notably Windows), endpoint plugins are not registered and middleware plugins are a no-op.

Set the library directory with `PREST_PLUGINPATH` (default `./lib`) or TOML:

```toml
pluginpath = "./lib"
```

`pluginpath` is a **top-level** key. Put it above the first `[table]` header in `prest.toml` — under a header such as `[http]` it is read as `http.pluginpath` and ignored.

### Extension-supported modules

* [Endpoint](endpoint-plugin.md) — `/_PLUGIN/{file}/{func}`
* [Middleware](middleware-plugin.md) — runs before CRUD table handlers

### Building plugins

Go's `plugin.Open` only accepts a `.so` built the same way as the `prestd` binary that loads it:

* the **same Go version**
* `CGO_ENABLED=1`, and the same `GOOS` / `GOARCH`
* the **same module versions** (build against the server's `go.mod` / `vendor`)
* the same `-trimpath` and `-mod=vendor` flags

Anything else fails at load time with `plugin was built with a different version of package ...`.

`etc/plugin/go-build.sh` in the server repo builds everything under `./lib/src` with those flags (`-buildmode=plugin -trimpath -ldflags "-s -w"`, plus `-mod=vendor` when `./vendor` exists — otherwise it warns):

| Source | Output | Kind |
|--------|--------|------|
| `./lib/src/<name>.go` | `./lib/<name>.so` | Endpoint |
| `./lib/src/<dir>/main.go` | `./lib/<dir>.so` | Endpoint |
| `./lib/src/middlewares/<name>.go` | `./lib/middlewares/<name>.so` | Middleware (v2.5.0) |

{% hint style="warning" %}
**The release archives cannot load plugins.** The `prestd` binaries attached to GitHub Releases are a portable build without cgo. Use the Docker image, or build `prestd` yourself with `CGO_ENABLED=1 go build -mod=vendor -trimpath` and build the plugins with the same toolchain.
{% endhint %}

### Plugins in Docker

The `prest/prest:<tag>` and `prest/prest:v2` images are based on the official Go image with `CGO_ENABLED=1` and `PREST_BUILD_PLUGINS=1`, and ship the server's `go.mod`, `go.sum`, and `vendor/` under `/app` (v2.5.0+, linux/amd64 and linux/arm64 from v2.5.1). At container start the entrypoint runs `go-build.sh`, compiling `/app/lib/src/**` into `/app/lib/`.

```yaml
services:
  prest:
    image: prest/prest:v2.5.2
    volumes:
      - ./lib/src:/app/lib/src
```

* Mount your sources at `/app/lib/src` (middleware under `/app/lib/src/middlewares`). Mounting over all of `/app/lib` also drops the bundled sample plugins.
* The `.so` files are rebuilt on every start. A build failure is logged and the server still starts.
* `-noplugins` tags have no Go toolchain and set `PREST_BUILD_PLUGINS=0`; they cannot build or load plugins.

### Security (v2.5.2)

From v2.5.2 ([GHSA-fmr6-9j48-xrqp](https://github.com/prest/prest/security/advisories/GHSA-fmr6-9j48-xrqp)), plugin file names must be letters, digits, `_`, or `-` (no dots), function names must be Go identifiers, and a library must resolve inside `pluginpath`. `/_PLUGIN` requires a token when `auth.enabled = true`. See [Endpoint Plugin — Name rules and auth](endpoint-plugin.md#name-rules-and-auth-v252).

## Related

- [Endpoint Plugin](endpoint-plugin.md)
- [Middleware Plugin](middleware-plugin.md)
- [Configuring pREST](../get-started/configuring-prest.md)
- [Deploying with Docker](../deployment/deploying-with-docker.md)
- [v2.5.0 release notes](../releases/v2.5.0.md) · [v2.5.2 release notes](../releases/v2.5.2.md)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [MCP](../prestd/acronyms.md#mcp)
