# @pipeworx/fema-nfhl

FEMA's National Flood Hazard Layer — the authoritative digital flood map.
Effective FIRM flood hazard zones, base flood elevations, floodways, levees,
cross-sections, LOMRs and LOMAs, the FIRM panel index and political
jurisdictions, served as a public ArcGIS REST MapServer.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `nfhl_flood_zone_at_point(latitude, longitude, include_bfe?)` — the effective
  FIRM flood zone at a coordinate: zone code and subtype, Special Flood Hazard
  Area flag, static BFE, governing DFIRM id, plus nearby BFE lines.
- `nfhl_query(layer_id, where?, bbox?, out_fields?, limit?, offset?, order_by?, return_geometry?)` —
  any NFHL layer by WHERE clause and optional envelope.
- `nfhl_layers(filter?)` — every layer with a plain-English guide to the ones
  that matter.

## Auth

Keyless.

## Data sources

- <https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer> — the MapServer.
- Layer 28 `S_FLD_HAZ_AR` (flood hazard zones), 27 `S_FLD_HAZ_LN` (boundary
  lines), 16 `S_BFE`, 1 `S_LOMR`, 34 `S_LOMA`, 3 `S_FIRM_PAN`, 23 `S_LEVEE`,
  0 NFHL Availability.

**Different API from the `openfema` pack.** `openfema` serves the OpenFEMA
tabular datasets — disaster declarations, individual and public assistance,
NFIP claims and policies. This pack serves the geospatial flood *map*, which
OpenFEMA cannot answer. Pair them: NFHL for the zone, `openfema` for the claims
and declaration history of the county it sits in.

## Traps

- `STATIC_BFE` is `-9999` where no base flood elevation applies — a sentinel,
  not an elevation. This pack returns null for it.
- `SFHA_TF` (`'T'`/`'F'` as **strings**) is what answers "is this a Special
  Flood Hazard Area". `FLD_ZONE` alone does not: Zone X has two subtypes and
  one of them is the 0.2% annual chance (500-year) floodplain.
- **NFHL coverage is not national.** A community that has never been digitally
  mapped returns zero features, which means "not mapped", not "no flood risk".
  `nfhl_flood_zone_at_point` says so explicitly rather than returning an empty
  array that reads as a clean all-clear. Layer 0 (NFHL Availability) tells you
  which is which.
- Identifier fields are strings: `DFIRM_ID='48201C'`, not `48201C`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fema-nfhl": {
      "url": "https://gateway.pipeworx.io/fema-nfhl/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fema-nfhl/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/nfhl_flood_zone_at_point \
  -H 'Content-Type: application/json' \
  -d '{"latitude":29.7604,"longitude":-95.3698}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/nfhl_flood_zone_at_point`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "fema-nfhl": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fema-nfhl"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-fema-nfhl
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fema Nfhl data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
