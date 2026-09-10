---
name: setup
description: First-time setup — install mcpwp on WordPress, generate an API key, and configure your MCP client. Start here.
user-invocable: true
---

# mcpwp Setup

Welcome. This skill walks you through connecting Claude Code to your WordPress site for the first time. $ARGUMENTS = site URL (optional, skips step 1).

Run through the steps below in order. Each one takes about 30 seconds.

---

## Step 1 — Install the WordPress Plugin

If mcpwp is not yet installed on your site, install it now.

**Option A — WP-CLI:**
```bash
wp plugin install https://updates.mcpwp.net/mcpwp-latest.zip --activate
```

**Option B — WP Admin:**
1. Download https://updates.mcpwp.net/mcpwp-latest.zip
2. Go to WP Admin > Plugins > Add New > Upload Plugin
3. Upload the zip and click Activate

**Option C — WP-CLI (Docker):**
```bash
docker exec <container> wp plugin install https://updates.mcpwp.net/mcpwp-latest.zip --activate --allow-root
```

Confirm it's running: visit `https://YOUR-SITE.com/wp-json/mcpwp/v1/site-info` — you should get a JSON response.

---

## Step 2 — Generate an API Key

In WP Admin, go to **mcpwp > Settings** and click **Generate API Key**.

Copy the key — it starts with `mcpwp_` and looks like:
```
mcpwp_a1b2c3d4e5f6...
```

Choose a role for the key based on what you need:

| Role | Tools | Best for |
|------|-------|----------|
| `admin` | All (260+ with Pro) | Full site management |
| `designer` | ~82 | Page building only |
| `editor` | ~99 | Content + design |
| `author` | ~40 | Content writing only |

For most use cases, start with `admin` — you can create restricted keys later with `wp_create_api_key()`.

---

## Step 3 — Configure Your MCP Client

Take your site URL and API key from Step 2, then add the MCP server config.

**Claude Code** — add to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "mcpwp": {
      "url": "https://YOUR-SITE.com/wp-json/mcpwp/v1/mcp",
      "headers": {"X-API-Key": "mcpwp_YOUR_KEY"}
    }
  }
}
```

**Per-project instead** — create `.mcp.json` in your project root with the same JSON.

**Claude Desktop** — same JSON, saved to:
- Mac: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

**Cursor / Windsurf** — use `/mcpwp:connect` for client-specific instructions.

After saving, restart Claude Code (or run `/reload-plugins` if already running).

---

## Step 4 — Test the Connection

Call:
```
wp_introspect()
```

A successful response includes:
- Site name, URL, and WordPress version
- Elementor status and layout mode
- List of available tool categories
- Recommended next actions

If you get an error, check the troubleshooting section in `/mcpwp:connect`.

---

## Step 5 — You're Connected

Here's what you can do now:

| Task | How |
|------|-----|
| Build a new page | "Build a landing page with a hero, 3 feature cards, and a CTA" |
| Edit an existing page | `wp_get_elementor_summary(id=PAGE_ID)` then describe what to change |
| Browse all tools | `/mcpwp:tools` |
| Elementor reference | `/mcpwp:elementor` |
| Design guide | `/mcpwp:design` |
| Check plugin status | `/mcpwp:status` |
| Build a full page (agent) | `use wp-builder` |

---

## Quick Config Generator

If you have your site URL and API key ready, here is the complete config block to copy:

```json
{
  "mcpServers": {
    "mcpwp": {
      "url": "https://REPLACE_WITH_YOUR_SITE/wp-json/mcpwp/v1/mcp",
      "headers": {
        "X-API-Key": "REPLACE_WITH_YOUR_KEY"
      }
    }
  }
}
```

Replace both values and paste into your client config file.
