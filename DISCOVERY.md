# Discovery test — multi-merchant, one subdirectory per shop

Base: `https://agent-friendly-platform.refined-x.workers.dev/`

## 1. Platform index

```bash
curl -s https://agent-friendly-platform.refined-x.workers.dev/llms.txt
curl -s https://agent-friendly-platform.refined-x.workers.dev/shops.json
curl -s https://agent-friendly-platform.refined-x.workers.dev/openapi.json | head -c 600
```

Expected: `llms.txt` lists all three shops with links to each
shop's `llms.txt`. `shops.json` lists the same shops machine-readably.

## 2. One shop, from its URL alone

Pick any shop, e.g. bean-bloom. An agent given only
`https://agent-friendly-platform.refined-x.workers.dev/shops/bean-bloom/` should do, in order:

```bash
SHOP=https://agent-friendly-platform.refined-x.workers.dev/shops/bean-bloom
curl -s $SHOP/llms.txt
curl -s $SHOP/openapi.json | head -c 600
curl -s $SHOP/agent.json
curl -s $SHOP/api/products.json
curl -s $SHOP/policies.json
curl -s $SHOP/mcp.json
```

Expected: `llms.txt` is the shop entry point and points to
`openapi.json`, `agent.json`, `api/catalog.json`, `policies.json`,
and the MCP endpoint. `agent.json` and `.well-known/agent.json`
agree. `products.json` has price and stock per product.

Repeat with `paper-lantern` and `terra-pots` — same relative
paths, different content. That is the point: the discovery
algorithm is shop-independent; only the base URL changes.

```bash
for s in bean-bloom paper-lantern terra-pots; do
  echo "== $s =="
  curl -s https://agent-friendly-platform.refined-x.workers.dev/shops/$s/llms.txt | head -5
  curl -s https://agent-friendly-platform.refined-x.workers.dev/shops/$s/api/products.json | python3 -c "import json,sys; print([(p['id'], p['price']) for p in json.load(sys.stdin)['products']])"
done
```

## 3. What is NOT live here

- `POST /shops/:shop/mcp` — documented in `mcp.json`, not served.
  GitHub Pages is static-only. A live MCP server needs a real
  backend; the URL, tools, and protocol version are already fixed
  so the backend can be dropped in without changing agent files.
- Orders / payments — `create_order` is a documented mock tool.
  No order is created, no money moves.

## 4. Failure cases this structure is designed to catch

- Fetching only the host root (`https://agent-friendly-platform.refined-x.workers.dev/llms.txt`) must NOT be
  mistaken for one shop. It is a platform index.
- Fetching `https://agent-friendly-platform.refined-x.workers.dev/shops/bean-bloom/llms.txt` must NOT return
  another shop's catalog. Each shop's files are namespaced by
  its subdirectory.
- `/.well-known/agent.json` at the host root describes the
  platform, not a shop. A shop's card lives at
  `/shops/:shop/.well-known/agent.json`.
