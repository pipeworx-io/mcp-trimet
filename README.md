# @pipeworx/trimet

Live arrivals, stop lookup, vehicle positions and service alerts for Portland,
Oregon — TriMet bus, MAX light rail, WES commuter rail and Portland Streetcar.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/trimet_arrivals \
  -H 'Content-Type: application/json' \
  -d '{"stop_ids":"7646"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/trimet_arrivals`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-trimet
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
