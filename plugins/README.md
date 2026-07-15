# Plugins

Extend pREST with OS shared libraries (`.so`) for custom middleware and endpoints. Plugins load beside the built-in REST and MCP surfaces — they do not replace them.

[Here](https://github.com/prest/prest/discussions/466#discussion-30623) is a discussion of how we arrived at this architecture.

Write a shared library and configure pREST to load it at server start. pREST uses the [Go plugin system](https://pkg.go.dev/plugin); on platforms where Go plugins are unavailable (notably Windows), plugin endpoints will not load.

Set the library directory with `PREST_PLUGINPATH` (default `./lib`) or TOML:

```toml
pluginpath = "./lib"
```

### Extension-supported modules

* [Endpoint](endpoint-plugin.md)
* [Middleware](middleware-plugin.md)

### Process of building

The first plugin builder targets **Go** source under `./lib`. The runtime can still load `.so` libraries produced by other toolchains; the automatic Go constructor is the documented path today.

## Related

- [Configuring pREST](../get-started/configuring-prest.md)
- [API Reference](../api-reference/README.md)
- [MCP over HTTP](../get-started/mcp-over-http.md)
- [Acronyms](../prestd/acronyms.md) · [REST](../prestd/acronyms.md#rest) · [MCP](../prestd/acronyms.md#mcp)
