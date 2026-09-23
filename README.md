# mcp-scraperapi

ScraperAPI MCP — wraps ScraperAPI (scraperapi.com), a proxy-based web

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1669+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `scraperapi_scrape` | Scrape any web page through ScraperAPI's rotating proxies and return its content. Returns HTML by default, or clean markdown with output_format:"markdown" (ideal for feeding an LLM). Use render:true for JavaScript-heavy pages (SPAs), premium:true for hard-to-scrape sites (residential proxies), and country_code to geotarget. Example: scraperapi_scrape({ url: "https://example.com", output_format: "markdown", render: true, _apiKey: "your-key" }) |
| `scraperapi_amazon_product` | Get structured Amazon product data by ASIN — title, price, rating, reviews count, features, images, availability. Example: scraperapi_amazon_product({ asin: "B0BFC7WQ6R", country: "us", tld: "com", _apiKey: "your-key" }) |
| `scraperapi_amazon_search` | Search Amazon and get structured results — product titles, ASINs, prices, ratings, thumbnails. Example: scraperapi_amazon_search({ query: "wireless earbuds", country: "us", tld: "com", _apiKey: "your-key" }) |
| `scraperapi_google_search` | Run a Google search and get structured SERP results — organic results, titles, links, snippets, and related data. Example: scraperapi_google_search({ query: "best running shoes 2026", country_code: "us", tld: "com", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "scraperapi": {
      "url": "https://gateway.pipeworx.io/scraperapi/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/scraperapi/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1669+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/scraperapi_scrape`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "scraperapi": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-scraperapi"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-scraperapi
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Scraperapi data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
