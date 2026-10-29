# @pipeworx/west-virginia-rules

West Virginia administrative rules (the Code of State Rules): the agency regulations that implement the state's statutes, such as the Board of Medicine's physician-assistant supervision requirements or the nursing boards' practice rules. Complements `wv_code_section`, which serves the statutes only.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_rules(agency?, title_series?, rule_name?, authority?, full_text?, full_text_mode?, status?, limit?)` — find rules by agency, rule family, title words, authorizing WV Code citation, or words inside the rule documents. Returns counts by status and agency first, then rule versions (active first).
- `get_rule(title_series | document)` — one rule family (e.g. `11-01B`): current version, every earlier version, and the filing history with document ids.
- `read_rule_text(doc_id, offset?, max_chars?, find?)` — the text of a filed rule, paged, or the passages around a phrase.

## Auth

Keyless.

## Data sources

- West Virginia Secretary of State, Code of State Rules — the state's official register of agency rules, their versions and filings.

Rules, agencies, statutory citations and filing dates only; no personal data is read or returned. Practitioner-level licensing data (individual licensees) is deliberately out of scope.

Things worth knowing:

- Every title-series lists all versions ever filed. One is the current rule; the rest are `Historical`. Sort and `status` filter accordingly.
- Filings described "Modified File" are redlines: struck and inserted text both appear (old and new section numbers side by side). "Agency Approved" and "Valid" filings are cleaner reads.
- Full text with several words behaves as a strict match in the register, so a long phrase can return nothing where one or two words return many; the no-match response says so.
- The register's own status dropdown does not narrow results, so the status filter is applied by the pack.
- A rule family not yet filed in the register, or an agency spelled differently, returns an explicit no-match naming the filters tried.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "west-virginia-rules": {
      "url": "https://gateway.pipeworx.io/west-virginia-rules/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/west-virginia-rules/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/search_rules \
  -H 'Content-Type: application/json' \
  -d '{"full_text":"physician assistant","status":"Active Only","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_rules`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-west-virginia-rules` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/west-virginia-rules/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "west-virginia-rules": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-west-virginia-rules"]
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
ask_pipeworx({ question: "your question about West Virginia Rules data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
