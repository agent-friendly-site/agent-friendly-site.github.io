# Rendering adapter — how this platform presents shops to different agents

Research: 2026-11-10, Claude + Grok UI conventions. Full report: research_notes/agent-ui-format-conventions-claude-grok-20261110-0531/report.md (in the builder's workspace; the demo repo carries this summary).

## Principle

The shop ships data only, never HTML/CSS. The platform owns presentation.
One canonical IR per shop (`shops/:shop/api/presentation.json`), then one
adapter per client. Markdown text is the contract; rich UI is progressive
enhancement. Every tool returns Markdown text + structuredContent (the IR).

## The canonical IR

See `presentation-schema.json` (root) and each shop's `api/presentation.json`:
intent, title, narrative, products[{id, name, price{amount,currency},
stock{status,quantity}, product_url, attributes, snippet}], options[{id,
question, required, choices[{id,label,detail,disabled}]}], actions (max 3,
semantic tool/args, never raw HTML), totals (authoritative, printed verbatim),
policies, guide, markdown_fallback, provenance.
Caps: products <= 5, one option group per turn, choices <= 6, actions <= 3.

## Adapters

| Client | Rendering |
|---|---|
| Muse | Clickable tap-to-reply options, list / shopping cards, map, agenda, HTML widget. Agent picks from the IR. |
| Claude (web/desktop) | MCP Apps: `io.modelcontextprotocol/ui`, `text/html;profile=mcp-app`, sandboxed iframe, platform-owned template fed by structuredContent. Allow/Always-allow prompt. Text fallback mandatory. |
| Claude Code | Text only + elicitation for one blocking choice/confirm (form mode, enums); URL mode for payment/sign-in. |
| ChatGPT | MCP Apps (same open standard as Claude). |
| Grok (grok.com / X) | Markdown/text only. No documented rich-UI convention for third parties. |
| Generic / unknown | Markdown/text only. Default when capabilities are unknown. |

## Client detection

From MCP `initialize` capabilities, never User-Agent or model name:
`extensions['io.modelcontextprotocol/ui']` + mimeTypes containing
`text/html;profile=mcp-app` => MCP Apps capable; `capabilities.elicitation`
form/url => elicitation capable; otherwise Markdown fallback.

## Fallback rules (text-only clients)

1. Text is the contract, UI is progressive enhancement. Every tool returns Markdown text + structuredContent (this IR).
2. One question per turn in text. Never present two option groups at once.
3. Numbered options, not buttons: '1. 手冲 — 喜欢花果香、清爽' and 'Reply with the number'.
4. Links, not embedded actions: product and checkout are plain Markdown links.
5. Totals verbatim, once: print the authoritative total; do not recompute or reformat it.
6. Confirm before money moves. Payment happens on the shop checkout page, never card details in chat.
7. Degrade the catalogue, not the truth: text shows at most 5 products; never truncate a price, stock status, or total.

## What shops must NOT do

- Do not ship HTML/CSS/JS from a shop. MCP Apps HTML is the platform's sandboxed template, fed by structuredContent — never merchant markup.
- Do not assume buttons, cards, or quick-replies exist. They exist only inside an MCP App iframe (Claude/ChatGPT) or as Muse options; everywhere else they are numbered text options.
- Do not request a Claude Artifact or a Grok Studio canvas. No API exists; the model decides. Do not use the leaked <xaiArtifact> tag.
- Do not use elicitation for sensitive data, payment credentials, or a catalogue/cart (flat schema only). Payment goes via URL-mode elicitation (Claude) or a plain checkout link (Grok).
- Do not emit Adaptive Cards, Slack Block Kit, or Telegram keyboards to Claude or Grok — neither consumes them.
- Do not put a fact only in the widget, only in an image, or only in a link destination. Price, stock, total, and policy are in the IR text too.

## Build order

1. IR + Markdown renderer + MCP tools returning content + structuredContent (serves Grok, Claude Code, every text client).
2. Elicitation for the single blocking choice / confirm on Claude Code.
3. One generic MCP App widget for Claude web/desktop (and ChatGPT), gated on client capability, text path kept as fallback.
4. Re-check Grok quarterly: if xAI documents MCP Apps rendering or any UI convention, the Grok adapter gains a widget path with no shop changes.

## Unverified (do not design on these)

- claude.ai web elicitation support; claude.ai surfacing of MCP prompts/resources.
- Grok Studio current status; whether Grok renders MCP Apps; whether Grok passes structuredContent/images through faithfully.
