# Zomato

Official MCP server: `https://mcp-server.zomato.com/mcp` (OAuth). Source: https://github.com/Zomato/mcp-server-manifest, read on 2026-10-26.

Food delivery only. No groceries. Zomato allows it for personal use only, and "not allowing any third party apps to be built on top of Zomato MCP right now". Access needs a request through Zomato's form, linked from that README.

## Tools

Zomato publishes features, not tool names (unverified: names and schemas). Read the server's tool list and map it:

| job | what Zomato says it does |
|---|---|
| Restaurant discovery | Find restaurants near the person. |
| Menu browsing | Dishes with prices, descriptions and ratings. |
| Cart | Create a cart. |
| Order | Place an order with tracking. Payment is by QR code. |

## Planning takeout (`plan_takeout`)

1. Read the slot and the household's `diet` from the task and `get_household`.
2. Find two or three restaurants that deliver now and fit the diet. A veg home gets pure-veg places or veg dishes only.
3. For each, pick dishes that feed `household.people`, using dish names from the menu.
4. `complete_task` with `{ app: "zomato", suggestions: [{ restaurant, dishes, pricePaise }] }`. `pricePaise` is the menu total, or `null` if you didn't see prices.

Build no cart unless the person asks.

## Placing the order (present only)

Zomato documents no confirmation rule. Use the same one as groceries: show the restaurant, the dishes, the total and the delivery address. Wait for an explicit yes. Place once. Then `record_orders` with `app: "zomato"` and `kind: "food"`.

## Client support

Zomato's OAuth accepts only these redirects: Claude (claude.ai and Claude Desktop connectors), ChatGPT, VS Code and Postman. Claude Code, Codex and Cursor are not on the list, so sign-in from them may fail (unverified). Use Zomato from Claude or ChatGPT if it does.
