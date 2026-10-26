# Aaj Kya Bana skill

An agent skill that runs an Indian home's kitchen loop with your own AI agent. It reads the week's plan from Aaj Kya Bana, fills your Swiggy Instamart or Zepto cart before the cook comes, writes the cook's card in their language, plans Zomato takeout when the cook is off, and records every order.

It works with any agent that speaks MCP: Claude, ChatGPT, Claude Code, Codex, Cursor.

## What your agent does

1. Asks Aaj Kya Bana for due tasks.
2. Fills the grocery cart with the vendor's own MCP server, signed in as you. Quantities scale to how many eat at home. Staples you always have are skipped. Packs are chosen to waste least.
3. Writes a short cook card: "सुनीता जी, आज: राजमा चावल, 4 लोग. राजमा रात से भीगा है।"
4. Records orders and updates the pantry.

It never places an order on a schedule. When you are in the chat and ask it to, it shows the items, the total and the address, and waits for your yes.

## Install

Adding the Aaj Kya Bana server as a connector is enough. The server sends this skill's rules to the agent itself, so Claude and ChatGPT behave the same without it. Installing the skill is optional, for Claude Code. See [SETUP.md](SETUP.md). In Claude Code:

```sh
/plugin marketplace add aaj-kya-bana/skill
/plugin install aaj-kya-bana@aaj-kya-bana
```

## Files

| path | what |
|---|---|
| `aaj-kya-bana/SKILL.md` | The loop, the rules, the cook card, the voice |
| `aaj-kya-bana/references/` | One file per vendor (Instamart, Zepto, Zomato), and scheduling |
| `commands/` | Claude Code slash commands that mirror the server's prompts |
| `.mcp.json` | The four MCP servers. Set `AKB_MCP_URL` to your server |
| `.claude-plugin/` | Claude Code plugin and marketplace manifests |

## Vendors

Swiggy, Zepto and Zomato run their own MCP servers. You sign in to each with your own account, and their terms apply. Zomato's is for personal use only. Aaj Kya Bana never calls a vendor, places an order or takes a payment.

## License

MIT. See [LICENSE](LICENSE).
