# Scheduling the loop

Aaj Kya Bana makes a `fill_cart` task due `household.leadMinutes` (default 120) before each cook visit, and a `sync_orders` task at 22:00. Something has to wake an agent to run them. Pick one way below. Hourly is enough, since `next_tasks` returns only what is due.

Every scheduled run is absent. No one is there to confirm, so it never places an order. Use this prompt everywhere:

```
Use the aaj-kya-bana skill. This is an unattended run and the person is not present. Work through every due Aaj Kya Bana task, then stop. Do not place any order.
```

## Claude Code routine (cloud, recommended)

Runs on Anthropic's cloud with your claude.ai connectors. Needs a Pro, Max, Team or Enterprise plan. Source: https://code.claude.com/docs/en/routines.

1. Add the connectors at https://claude.ai/customize/connectors: Aaj Kya Bana, Swiggy Instamart, Zepto (see SETUP.md). Routines include all connected connectors by default. Servers added locally with `claude mcp add` don't appear there.
2. Create the routine at https://claude.ai/code/routines (New task), or run `/schedule` in Claude Code. Paste the prompt above.
3. Add a schedule trigger, for example every hour from 6:00 to 22:00. Custom cron runs at most once an hour and is UTC unless prefixed, as in `CRON_TZ=Asia/Kolkata 0 6-22 * * *`.
4. Optional. To let Aaj Kya Bana wake the routine when a task is due, add an **API trigger** in the routine's edit dialog on the web. Copy the Fire URL and the token, which is shown once. Register both with Aaj Kya Bana (below).

The routine receives the fire text inside a `<routine-fire-payload>` block. The prompt above doesn't need it, since `next_tasks` finds the work.

Limits: 100 scheduled runs and 100 API fires an hour per account. Runs draw on your subscription.

## Register the Aaj Kya Bana trigger

Aaj Kya Bana holds one trigger per household and fires it once per due task.

```sh
curl -X PUT "$AKB_URL/v1/trigger" \
  -H "Authorization: Bearer $AKB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"kind":"claude_routine","url":"<Fire URL>","token":"<routine token>"}'
```

`AKB_URL` is the server root, such as `http://localhost:8787`. `AKB_TOKEN` is your app session token (unverified: whether a personal token from `POST /v1/tokens` is accepted on `/v1/trigger`). For anything else, such as n8n or a bridge to ChatGPT, use `"kind":"webhook"` with your URL. Aaj Kya Bana then POSTs `{ taskId, kind, dueAt }` with `Authorization: Bearer <token>`.

## ChatGPT scheduled task

Unverified: OpenAI's help pages for scheduled tasks and developer mode returned 403 when checked, and it is not confirmed that a scheduled task can call developer-mode connectors.

1. Add the connectors in developer mode (see SETUP.md).
2. In a chat with those connectors on, ask: "Every day at 6:00, 13:00 and 21:30: <the prompt above>".
3. Check the task under your profile's Tasks page after the first run.

## Laptop cron with Claude Code

Install the plugin and sign in to each server once in an interactive session (`/mcp`). Headless runs reuse those sign-ins.

`crontab -e`:

```cron
0 6-22 * * * cd "$HOME" && /usr/local/bin/claude -p "Use the aaj-kya-bana skill. This is an unattended run and the person is not present. Work through every due Aaj Kya Bana task, then stop. Do not place any order." --permission-mode dontAsk --allowedTools "mcp__plugin_aaj-kya-bana_aaj-kya-bana__*,mcp__plugin_aaj-kya-bana_swiggy-instamart__get_addresses,mcp__plugin_aaj-kya-bana_swiggy-instamart__search_products,mcp__plugin_aaj-kya-bana_swiggy-instamart__your_go_to_items,mcp__plugin_aaj-kya-bana_swiggy-instamart__get_cart,mcp__plugin_aaj-kya-bana_swiggy-instamart__update_cart,mcp__plugin_aaj-kya-bana_swiggy-instamart__get_orders" --disallowedTools "mcp__plugin_aaj-kya-bana_swiggy-instamart__checkout" >> "$HOME/.aaj-kya-bana.log" 2>&1
```

`dontAsk` denies every tool not in `--allowedTools`, so checkout can't run even if the model tries. The deny rule is a second lock. For Zepto, run `/mcp` once to see its tool names and add only the search, cart and order-history ones (unverified: Zepto doesn't publish them). Use the full path from `which claude`.

## Laptop cron with Codex

Add the servers to `~/.codex/config.toml` (SETUP.md) and limit each grocery server to safe tools:

```toml
[mcp_servers.swiggy-instamart]
url = "https://mcp.swiggy.com/im"
enabled_tools = ["get_addresses", "search_products", "your_go_to_items", "get_cart", "update_cart", "get_orders"]
```

```cron
0 6-22 * * * cd "$HOME" && /usr/local/bin/codex exec "Use the aaj-kya-bana skill. This is an unattended run and the person is not present. Work through every due Aaj Kya Bana task, then stop. Do not place any order." >> "$HOME/.aaj-kya-bana.log" 2>&1
```

Unverified: whether `codex exec` runs MCP tools without an approval prompt. If runs stall, set `default_tools_approval_mode = "approve"` on the Aaj Kya Bana and grocery servers, which is safe only with `enabled_tools` set as above.

A laptop that is asleep misses its runs. A Claude routine doesn't.
