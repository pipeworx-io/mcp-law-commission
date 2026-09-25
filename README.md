# @pipeworx/law-commission

Consultation papers, reports, summaries and law-reform projects published by
the **Law Commission of England & Wales** (lawcom.gov.uk) — the independent
statutory body that reviews the law and recommends reform to Parliament.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `search_law_commission_publications(query?, limit?)` — search publication
  titles and body text; returns title, link, date and excerpt.
- `get_law_commission_publication(id?, link?)` — fetch one publication's full
  text, the law-reform project it belongs to, and any attached PDF reports.
- `list_law_commission_projects(status?, limit?)` — list law-reform
  workstreams, optionally filtered to `current`, `completed`, `upcoming` or
  `other`.

## Auth

Keyless. No signup, no key.

## Data sources

- <https://lawcom.gov.uk/wp-json/wp/v2/publication> — publications (275 live,
  probed 2026-09-18), custom WordPress post type.
- <https://lawcom.gov.uk/wp-json/wp/v2/project> — law-reform projects (118 live).
- <https://lawcom.gov.uk/wp-json/wp/v2/media?parent=<id>> — attached PDFs.
- <https://lawcom.gov.uk/wp-json/wp/v2/project_status> — the four status terms
  (`current`, `completed`, `upcoming-projects-current`, `other`) used to
  resolve the `status` filter on `list_law_commission_projects`.

## Welsh-language duplicates

Every publication and project the Commission issues in Welsh is a **separate
WordPress post** — same report, same date, its own numeric id — carrying a
`language` taxonomy term (`Cymraeg`, the only term the `language` taxonomy
has; English posts carry no term at all, since English is the unmarked
default). A plain title search for "leasehold" returns both "Commercial
leasehold: overcoming barriers to transactions" (English) and "Lesddaliad
masnachol: goresgyn rhwystrau rhag trafodion" (the same report, in Welsh) as
two hits.

`search_law_commission_publications` and `list_law_commission_projects` both
filter out any row carrying a `language` term before applying `limit`, so a
caller gets one English row per report rather than two. The filter fetches
`limit * 3` rows upstream (capped at 100) to leave headroom for the ones it
drops — in practice Welsh duplicates run well under half of any result set.
The Welsh original is still directly reachable via `get_law_commission_publication`
by id or link; the filter only applies to search/listing, and the returned
`language` field on that tool tells you which you got (`en` / `cy`).

`X-WP-Total` from the upstream API counts **both** languages, so it is surfaced
as `total_matching_all_languages` rather than `total` — using it as the count
of English rows would overstate coverage by roughly the Welsh-duplicate share.

## Caveats worth passing to a user

- **This is not case law or legislation.** The Commission's own reports
  recommend reform; they are not binding law. `legislation-uk` covers
  legislation.gov.uk, `uk-caselaw` covers judgments — this pack is the third
  leg (Law Commission reports) named in Bruce's "Parliamentary materials,
  explanatory notes and Law Commission reports" ask (`docs/uk-primary-law-scope.md`).
- **Full report text is PDF, not HTML.** The `publication` post body typically
  carries only a link to its project — no summary text and no `excerpt` field
  content — so `search_law_commission_publications`' `excerpt` is usually
  empty and `get_law_commission_publication`'s `text` is one line. The actual
  report is the PDF in `attachments`. This pack does not extract PDF text.
- **Search is title/body text, not the PDFs.** Given the above, that means
  search is effectively title-only for most publications; a phrase that only
  appears inside the attached PDF will not match.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "law-commission": {
      "url": "https://gateway.pipeworx.io/law-commission/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/law-commission/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/search_law_commission_publications \
  -H 'Content-Type: application/json' \
  -d '{"query":"leasehold","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_law_commission_publications`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "law-commission": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-law-commission"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-law-commission
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Law Commission data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
