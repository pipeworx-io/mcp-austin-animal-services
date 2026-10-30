# @pipeworx/austin-animal-services

Live shelter intake and outcome records from the Austin Animal Center (Austin,
TX) — recent intakes, animal lookup, and outcome summaries (adoptions,
transfers, euthanasia, etc).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1863+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `austin_recent_intakes(days?, animal_type?, intake_type?, limit?)` — recent
  intakes (default last 30 days), with a total count for the window plus a
  page of rows. Answers "how many dogs did Austin Animal Center take in last
  month".
- `austin_search_animal(animal_id?, name?, breed?, limit?)` — look up an
  animal by id, name, or breed across both intake and outcome records.
- `austin_outcomes_summary(since?, group_by?)` — counts of outcomes grouped
  by `outcome_type`, `animal_type`, or `month`.

## Auth

Keyless. Pass your own Socrata app token via `_apiKey` for higher rate
limits.

## Data sources

- <https://data.austintexas.gov/resource/pyqf-r2dc.json> — Austin Animal
  Center Intakes (current, hourly refresh, live since May 2025).
- <https://data.austintexas.gov/resource/gsvs-ypi7.json> — Austin Animal
  Center Outcomes (current, hourly refresh, live since May 2025).

The city migrated to a new shelter system (ShelterBuddy) in May 2025. The
older "Austin Animal Center Intakes/Outcomes (10/01/2013 to 05/05/2025)"
datasets (`wter-evkm` / `9t4d-g238`) are now frozen historical snapshots with
no rows after 2025-05-19 — this pack deliberately uses the current,
hourly-refreshed successors instead so "recent" and "last month" queries
answer correctly. Socrata paging is `$limit`/`$offset`; every call in this
pack sets `$limit` explicitly. Field names differ between the current and
historical datasets (e.g. intake timestamp is `source_date` on the current
dataset, `datetime` on the historical one) — don't assume they're
interchangeable.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "austin-animal-services": {
      "url": "https://gateway.pipeworx.io/austin-animal-services/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/austin-animal-services/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1863+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/austin_recent_intakes \
  -H 'Content-Type: application/json' \
  -d '{"days":30,"animal_type":"Dog"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/austin_recent_intakes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-austin-animal-services` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/austin-animal-services/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "austin-animal-services": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-austin-animal-services"]
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
ask_pipeworx({ question: "your question about Austin Animal Services data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
