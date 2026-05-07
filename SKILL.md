---
name: topr
description: Expert guidance for using the topr CLI and MCP server to manage influencer marketing campaigns on the Topr platform. Use this skill whenever the user mentions topr, topr CLI, campaign analytics, creator posting status, brand budgets, missed posts, viral videos, or wants to query their Topr campaigns from the terminal or an AI agent. Trigger even if the user just asks "who posted today", "check my budget", "any viral videos", or "which creators missed posting" — these are topr questions.
---

# Topr CLI + MCP

Topr is an influencer marketing platform. This skill covers:
- **CLI** (`@speedy_devv/topr` on npm) — terminal commands for brand managers
- **MCP server** (`/api/mcp`) — 15 tools for AI agents (Claude Desktop, Claude Code, Codex)

Both use the same `topr_live_` PAT token for auth. One token, all brands.

---

## Installation

```bash
# Install globally
npm install -g @speedy_devv/topr

# Or run without installing
npx @speedy_devv/topr login
```

If you see "command not found" after install:
```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

---

## Authentication

```bash
topr login
```

Opens your default browser → topr.io/auth/cli → click **Authorize topr CLI** (one click if already logged in) → token saved to `~/.config/topr/credentials.json`.

The URL is always printed in the terminal in case the browser doesn't open — you can copy-paste it into any browser.

**CI/CD:** Set `TOPR_TOKEN=topr_live_...` as an environment variable. No browser needed.

```bash
topr logout   # revoke local credentials
topr whoami   # confirm who you're logged in as
```

---

## Command Reference

### Brands
```bash
topr brands list
# Shows all brands you belong to + your role (owner/admin)
# ID                                    NAME            ROLE
# 3f2a...                               Acme Co         owner
```

Always run this first if you don't know your brand IDs. The `--brand` flag on other commands defaults automatically if you only have one brand.

### Campaigns
```bash
topr campaigns list [--brand=<id>]
topr campaigns get <campaign-id>          # details + 7-day posting rate
topr campaigns budget <campaign-id>       # reserved / spent / remaining / burn rate / runway
```

### Creators
```bash
topr creators posted --campaign=<id> [--date=2026-05-07]   # who posted today (or on date)
topr creators missed  --campaign=<id> [--date=2026-05-07]   # who didn't post — sorted by days silent
topr creators stats   <username>          [--campaign=<id>]  # per-creator breakdown
```

`creators missed` is sorted most-at-risk first (highest days silent). Look for "← at risk" markers.

### Videos
```bash
topr videos top   --campaign=<id> [--limit=10]   # ranked by views
topr videos today --campaign=<id>                # posted today with live metrics
```

### Budget
```bash
topr budget summary [--brand=<id>]           # wallet balance + reserved + total funded
topr budget history [--brand=<id>] [--days=7]  # recent transactions
```

### Universal flags
- `--json` on any command outputs raw JSON — useful for piping or scripting
- `--brand=<id>` scopes to a specific brand when you manage multiple

---

## Daily Workflow

A typical morning check for a brand manager:

```bash
topr brands list                                      # get brand + campaign IDs
topr campaigns list                                   # see all active campaigns
topr creators missed --campaign=<id>                  # who hasn't posted?
topr videos top --campaign=<id> --limit=5             # what's performing?
topr campaigns budget <id>                            # how's the spend?
```

---

## MCP Server

The MCP server runs at `https://topr.io/api/mcp` and is compatible with Claude Desktop, Claude Code, and Codex.

**Configure in Claude Desktop** (`~/.claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "topr": {
      "url": "https://topr.io/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOPR_LIVE_TOKEN"
      }
    }
  }
}
```

Get your token: `cat ~/.config/topr/credentials.json`

**Discovery:** `GET https://topr.io/.well-known/oauth-protected-resource` — MCP clients can auto-discover the auth server and run the PKCE flow automatically.

### Available Tools (15)

| Tool | What it does |
|---|---|
| `get_brand_context` | Who you are + all your brands |
| `list_campaigns` | All campaigns with status |
| `get_campaign` | Single campaign + 7-day posting rate |
| `get_campaign_budget` | Reserved / spent / remaining / burn rate |
| `get_creators_posted` | Who posted on a date (default: today) |
| `get_creators_missed` | Who didn't post — at-risk creators first |
| `get_posting_rate` | % of creators posting over N days |
| `get_creator_stats` | Per-creator views, earnings, last post |
| `get_top_videos` | Ranked by view count |
| `get_videos_for_date` | All videos on a specific date |
| `get_viral_videos` | Videos above a view threshold |
| `get_daily_summary` | Posts + views + earnings for a day |
| `get_campaign_performance` | Aggregate metrics over a date range |
| `get_budget_summary` | Wallet balance across brands |
| `get_spend_today` | Transactions from today |

---

## Security Model

- Token format: `topr_live_<random32>` — only the SHA256 hash is stored server-side
- Token covers all brands — pass `--brand` or `brand_id` to scope to one
- Brand access is enforced per-request from `brand_members` table
- Fake/unowned brand IDs return empty results, not errors from other brands
- Revoke any token from topr.io → Settings → API Tokens

---

## Common Patterns

**"Who missed posting this week?"**
```bash
for date in 2026-05-01 2026-05-02 2026-05-03 2026-05-04 2026-05-05; do
  echo "=== $date ===" && topr creators missed --campaign=<id> --date=$date --json
done
```

**"What's our budget burn rate?"**
```bash
topr campaigns budget <id>
# Shows: burn/day + runway days remaining
```

**"Pipe to jq for custom queries"**
```bash
topr videos top --campaign=<id> --json | jq '[.[] | select(.view_count > 50000)]'
```

**MCP in an agent conversation:**
> "Check which creators haven't posted in the last 3 days on campaign X and summarize their last known performance"

The agent calls `get_creators_missed`, then `get_creator_stats` for each — no terminal needed.
