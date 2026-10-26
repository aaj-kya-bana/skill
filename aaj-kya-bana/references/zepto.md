# Zepto

Official consumer MCP server: `https://mcp.zepto.co.in/mcp` (OAuth, Indian mobile number and OTP). Source: https://github.com/zeptonow/mcp, read on 2026-10-26.

Every order is a real Zepto order. There is no sandbox or demo mode. Never test checkout.

## Tools

Zepto publishes capabilities, not tool names or parameters (unverified: names and schemas). Read the tool list the server returns and map it to these jobs:

| job | what Zepto says it does |
|---|---|
| Search products | Natural-language search of the live catalogue, with availability and prices. |
| Cart | Add items, change quantities, remove items. The cart syncs with the person's Zepto account. |
| Place order | Places a real order. Payment by cash on delivery, UPI link, card, Zepto Cash, or UPI Reserve Pay. |
| Order history | Past orders with status, items and prices. |

Zepto documents no address tool, no minimum order and no rate limits.

## Filling the cart

1. Read the current cart first. If the cart tool sets quantities rather than replacing the cart, change only your items. Don't remove the person's own items.
2. Search per ingredient. Pick the pack that covers the scaled quantity with least waste. Take the price from the search result or the cart, never a guess.
3. Read the cart back and use its total for `totalPaise`.

## Placing the order (present only)

Zepto's README describes the agent presenting "the full cart for your review" before an order. Treat its confirmation the same as Swiggy's:

1. Show the items, the total, the delivery address and the payment method.
2. Ask one question and wait for an explicit yes.
3. Place once. On an error, read order history before trying again.
4. `record_orders` with `app: "zepto"`.

If the person pays by UPI link, the order isn't placed until they pay. Record it only once order history shows it.

## Client support

Zepto lists setup for Claude (custom connector), Claude Desktop and Cursor (`npx mcp-remote https://mcp.zepto.co.in/mcp`), and Claude Code (`claude mcp add --transport http --scope user zepto https://mcp.zepto.co.in/mcp`). ChatGPT support is "in beta, pending approval from OpenAI". Codex is not mentioned (unverified).
