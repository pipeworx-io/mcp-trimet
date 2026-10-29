# @pipeworx/trimet

Live arrivals, stop lookup, vehicle positions and service alerts for Portland,
Oregon — TriMet bus, MAX light rail, WES commuter rail and Portland Streetcar.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

**Requires an API key** (TriMet calls it an *appID*) — free and self-service at
<https://developer.trimet.org/appid/registration/>.

- Platform key: `PLATFORM_TRIMET_KEY`
- Caller-supplied: `_apiKey` argument on any tool

## Tools

| Tool | What it answers |
|---|---|
| `trimet_arrivals` | Live predicted arrivals at one or more stops |
| `trimet_stops_near` | Stops near a lat/lon, with the routes serving each |
| `trimet_vehicles` | Live fleet position, with schedule deviation |
| `trimet_alerts` | Current detours and disruptions |

## Traps

**Two APIs on one credential, and they disagree about case.** The modern JSON
web service is `/ws/v2/...` (lowercase `v`). The GTFS-Realtime feeds are
`/ws/V1/...` (uppercase `V`) and answer protobuf, not JSON. Getting the case
wrong returns an error page, not a redirect.

**TriMet reports failures inside a 200 body.** An invalid stop id or a bad
parameter comes back as HTTP 200 with `resultSet.errorMessage` set and an empty
arrival list. `readJson()` checks for it and throws, so the caller sees the real
reason instead of "no buses are coming".

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "trimet": {
      "url": "https://gateway.pipeworx.io/trimet/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/trimet/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/trimet_arrivals \
  -H 'Content-Type: application/json' \
  -d '{"stop_ids":"7646"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/trimet_arrivals`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-trimet` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/trimet/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "trimet": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-trimet"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Trimet data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
