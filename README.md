# agent-friendly-site — multi-merchant agent-first demo

Demo only. No real shops, no real orders, no real payments.

Live site: https://agent-friendly-site.github.io/

## Idea

One host, one shop per subdirectory. Each shop is agent-only:
no human shopping UI. An agent given only a shop URL should be
able to discover, in order, what the shop sells, how to interact
with it, and what its policies are.

- Platform: `https://agent-friendly-site.github.io/`
- Shop: `https://agent-friendly-site.github.io/shops/:shop/` — e.g. `https://agent-friendly-site.github.io/shops/bean-bloom/`

## Shops (invented)

- [Bean Bloom Coffee (晨光咖啡)](https://agent-friendly-site.github.io/shops/bean-bloom/) — Small-batch roasted coffee beans, shipped from Yunnan.
- [Paper Lantern Stationery (纸灯文具)](https://agent-friendly-site.github.io/shops/paper-lantern/) — Notebooks and pens for people who still write by hand.
- [Terra Pots Ceramics (陶土手作)](https://agent-friendly-site.github.io/shops/terra-pots/) — Hand-thrown stoneware mugs and plates, made in Jingdezhen.

## URL structure

```
https://agent-friendly-site.github.io/
  llms.txt                  # platform index, links to every shop
  llms-full.txt             # platform corpus (short)
  openapi.json              # platform-level API
  robots.txt
  sitemap.xml
  shops.json                # machine-readable shop list
  RENDER_ADAPTER.md         # how the platform presents shops to Muse / Claude / Grok
  presentation-schema.json  # JSON Schema for the canonical presentation IR
  .well-known/agent.json    # platform agent card
  index.html                # minimal platform index (humans may read it; shops are agent-only)

  shops/:shop/
    index.html              # minimal shop landing; links to agent files
    llms.txt                # shop entry point — start here
    llms-full.txt           # full shop corpus: products + policies
    openapi.json            # shop API contract
    agent.json              # shop agent card
    .well-known/agent.json  # same card, well-known location
    mcp.json                # MCP endpoint + tools (documented; see below)
    api/catalog.json        # catalog
    api/products.json       # products with price + stock
    policies.json           # shipping, returns, privacy
    guide.md                # shopping-assistant guide: how to sell, not just list
    api/recommendations.json  # structured recommendations, bundles, free-shipping tips
    api/presentation.json  # canonical presentation IR: products, options, actions, totals, Markdown fallback
```

## Discovery — what an agent should do with a shop URL

1. `GET {shop}/` — get the shop landing / manifest links.
2. `GET {shop}/llms.txt` — entry point, index, links to everything else.
3. `GET {shop}/openapi.json` — the interaction contract.
4. `GET {shop}/agent.json` — agent card, capabilities, MCP endpoint.
5. `GET {shop}/api/products.json` — live-ish catalog for this demo.
6. In production: `POST {shop}/mcp` — `initialize`, then `tools/list`,
   then `tools/call`. Tools: `list_products`, `get_product`,
   `create_order`, `get_order`.

See `DISCOVERY.md` for the exact curl / test steps.

## Honest limitation: no live MCP on GitHub Pages

GitHub Pages is static-only. It cannot run a `POST /mcp` MCP server,
checkout, or payments. This repo therefore demonstrates the *static*
half of agent discovery — `llms.txt`, `openapi.json`, `agent.json`,
catalog, policies — and *documents* the MCP endpoint in each shop's
`mcp.json` instead of serving it live.

In production the same URL structure works with a real server behind
`/:shop/mcp` (a Worker / Cloudflare Worker / small API server). The
agent files do not change; only `mcp.json`'s `status` changes from
`documented-demo-only` to live.

## Test it

See `DISCOVERY.md`.
