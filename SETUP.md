# Setup

You connect four MCP servers and, where the client supports it, install the skill.

| server | URL | sign-in |
|---|---|---|
| Aaj Kya Bana | your server's `/mcp`, `http://localhost:8787/mcp` in development | OAuth (email code), or a personal token from the app |
| Swiggy Instamart | `https://mcp.swiggy.com/im` | OAuth, phone and OTP |
| Zepto | `https://mcp.zepto.co.in/mcp` | OAuth, phone and OTP |
| Zomato | `https://mcp-server.zomato.com/mcp` | OAuth. Request personal access first through the form in [Zomato's README](https://github.com/Zomato/mcp-server-manifest) |

Connect only the grocery app you use. Zomato is optional, for days the cook is off.

Cloud clients (Claude, ChatGPT) connect from their own servers, so `localhost` won't work there. Use a public Aaj Kya Bana URL.

## Claude (claude.ai, Desktop, mobile)

1. Customize > Connectors > + Add > Add custom connector. Add each server above by name and URL, then Connect and sign in. Free plans get one custom connector.
2. Skill (unverified: the menu path): upload the `aaj-kya-bana` folder as a zip under Customize > Skills. Without it, the Aaj Kya Bana prompts `plan_my_week`, `fill_tomorrows_cart`, `cook_is_off_today` and `sync_my_orders` still run the loop.
3. To run it on a schedule, create a routine: [scheduling.md](aaj-kya-bana/references/scheduling.md#claude-code-routine-cloud-recommended).

## ChatGPT

Unverified: OpenAI's help page returned 403. These steps are from Zepto's and Swiggy's docs. Swiggy says developer mode is on Business, Enterprise and Edu plans, rolling out more broadly. Zepto says its ChatGPT app is in beta.

1. Settings > Apps > Advanced settings > enable Developer mode.
2. Create an app for each server with its URL. Sign in when the OAuth window opens.
3. In a chat, mention the app (`@zepto`) or pick it from the tools menu, then ask "Fill tomorrow's cart with Aaj Kya Bana".

ChatGPT doesn't load `SKILL.md`. The Aaj Kya Bana server's prompts carry the loop.

## Claude Code

```sh
/plugin marketplace add OWNER/REPO        # this repo on GitHub, or ./skill from a clone
/plugin install aaj-kya-bana@aaj-kya-bana
```

That installs the skill, the four servers, and `/aaj-kya-bana:plan-week`, `fill-cart`, `cook-off` and `sync-orders`. Point at a deployed server with `export AKB_MCP_URL=https://your-host/mcp` before starting Claude Code. Then run `/mcp` and sign in to each server once.

Tested on Claude Code 2.1.306: the marketplace adds, the plugin installs, and `claude mcp list` shows all four servers with the `AKB_MCP_URL` default and override.

## Codex

`~/.codex/config.toml`:

```toml
[mcp_servers.aaj-kya-bana]
url = "http://localhost:8787/mcp"

[mcp_servers.swiggy-instamart]
url = "https://mcp.swiggy.com/im"

[mcp_servers.zepto]
url = "https://mcp.zepto.co.in/mcp"
```

Sign in to each: `codex mcp login aaj-kya-bana` and so on. To use a personal token instead of OAuth, add `bearer_token_env_var = "AKB_TOKEN"` under `aaj-kya-bana` and export `AKB_TOKEN`.

Skill:

```sh
mkdir -p ~/.agents/skills
ln -s "$PWD/skill/aaj-kya-bana" ~/.agents/skills/aaj-kya-bana
```

Unverified: Swiggy and Zepto don't list Codex as a client. If sign-in fails, use Claude Code or Claude.

## Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "aaj-kya-bana": { "url": "http://localhost:8787/mcp" },
    "swiggy-instamart": { "url": "https://mcp.swiggy.com/im" },
    "zepto": { "url": "https://mcp.zepto.co.in/mcp" }
  }
}
```

For a personal token, add `"headers": { "Authorization": "Bearer ${env:AKB_TOKEN}" }` to `aaj-kya-bana`. Cursor reads skills from `~/.agents/skills/`, so the Codex symlink above covers Cursor too.

## Scheduling

Routines, ChatGPT tasks, laptop cron, and the Aaj Kya Bana trigger are in [scheduling.md](aaj-kya-bana/references/scheduling.md).
