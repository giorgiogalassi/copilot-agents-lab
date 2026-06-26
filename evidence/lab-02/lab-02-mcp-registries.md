# Lab 02 — MCP Registries and Plugin Management

## Server added

Context7 (`@upstash/context7-mcp`), a documentation lookup server exposing two tools: `resolve-library-id` and `query-docs`.

---

## Registry-sourced vs. manually configured

The `before` config uses an HTTP remote endpoint (`type: http`, `url: https://mcp.context7.com/mcp`): Copilot connects to the hosted service directly, with no local process. The server version is opaque -- you have no control over what is deployed at that URL.

The `registries` config uses a local process (`command: npx`, `args: ["-y", "@upstash/context7-mcp@3.2.2", ...]`): npx downloads and runs the package locally. The version is explicit in the args array (`@3.2.2`).

| | HTTP remote | npx local |
|---|---|---|
| Version control | None -- whatever is deployed | Explicit via `@x.y.z` in args |
| Reproducibility | Low -- server can update silently | High -- pinned version is stable |
| Suitable for CI | No | Yes, with pinned version |

---

## Version pinning and tool scoping

An HTTP server at a fixed URL behaves like `latest`: the tool surface can change at any time without notice. An npx config without a version pin (`@upstash/context7-mcp` without `@x.y.z`) has the same problem.

Pinning a version (`@3.2.2`) freezes the tool surface. A pipeline run six months from now uses the same tools with the same signatures.

**On tool scoping (`tools: ["*"]` vs. explicit list):**

Using `"*"` means the agent can access all tools the server exposes. If the server version changes, new tools become available and existing ones may be renamed or removed. With `"*"` the agent will not fail on a missing tool name, but task behavior can still change: the agent may pick a different tool than before, or a previously used tool may no longer exist. This reduces explicit failures but does not eliminate behavioral drift.

Using an explicit tool list (e.g. `"tools": ["resolve-library-id", "query-docs"]`) pins the permitted surface. If those tools are removed in a new version, the failure is immediate and visible -- which is preferable in CI over silent behavioral drift.

In interactive exploration, `"*"` is fine. In CI, pin both the server version and the tool list.

---

## Key takeaway

Version pinning in MCP follows the same principle as Actions and package management: `actions/checkout@v4` not `@latest`, `npm ci` not `npm install`, `@upstash/context7-mcp@3.2.2` not `@upstash/context7-mcp`. Reproducibility requires explicit versions at every layer.
