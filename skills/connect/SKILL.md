---
name: connect
description: MCP connection reference for Claude Code, Claude Desktop, Cursor, and Windsurf. Use /mcpwp:setup for first-time guided setup.
user-invocable: true
---

# MCP Connection Reference

Quick reference for connecting different MCP clients to a mcpwp-enabled WordPress site. $ARGUMENTS = client name or site URL + API key.

> First time? Run `/mcpwp:setup` instead — it walks you through everything interactively.

## Config Template

Replace `YOUR-SITE.com` and `mcpwp_YOUR_KEY` in every config below.

## Claude Code

**Global** (`~/.claude/settings.json`):
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

**Per-project** (`.mcp.json` in project root):
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

## Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):
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
Restart Claude Desktop after saving.

## Cursor

Settings > MCP Servers > Add:
- **Name:** mcpwp
- **URL:** `https://YOUR-SITE.com/wp-json/mcpwp/v1/mcp`
- **Header:** `X-API-Key: mcpwp_YOUR_KEY`

## Windsurf

Settings > MCP > Add Server:
- **URL:** `https://YOUR-SITE.com/wp-json/mcpwp/v1/mcp`
- **Headers:** `{"X-API-Key": "mcpwp_YOUR_KEY"}`

## Multiple Sites

Use a unique name per site in `mcpServers`:
```json
{
  "mcpServers": {
    "mcpwp-staging": {
      "url": "https://staging.example.com/wp-json/mcpwp/v1/mcp",
      "headers": {"X-API-Key": "mcpwp_STAGING_KEY"}
    },
    "mcpwp-prod": {
      "url": "https://example.com/wp-json/mcpwp/v1/mcp",
      "headers": {"X-API-Key": "mcpwp_PROD_KEY"}
    }
  }
}
```

## Verify Connection

After saving config, call `wp_introspect()` — returns site info, available tools, and capabilities.

## Troubleshooting

- **401 Unauthorized:** API key wrong or missing. Must start with `mcpwp_`.
- **404 Not Found:** Plugin not activated. Check `https://YOUR-SITE.com/wp-json/mcpwp/v1/site-info`
- **500 Error:** Check WP debug log. Common cause: PHP < 7.4.
- **CORS error:** Add your domain in WP Admin > mcpwp > Settings > Allowed Origins.
