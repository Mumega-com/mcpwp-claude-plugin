# mcpwp — Claude Code Plugin

Connect Claude Code to any WordPress site. 260+ MCP tools for managing pages, Elementor, WooCommerce, media, SEO, and more through natural language.

## Install

```bash
claude plugin marketplace add https://github.com/Mumega-com/mcpwp-claude-plugin.git
claude plugin install mcpwp@mcpwp
```

Then restart Claude Code or run `/reload-plugins`.

**Alternative** — clone manually:
```bash
git clone https://github.com/Mumega-com/mcpwp-claude-plugin.git ~/.claude/plugins/mcpwp
```

## What's Inside

### Skills (slash commands)
| Command | What it does |
|---------|-------------|
| `/mcpwp:setup` | **Start here** — guided first-time setup |
| `/mcpwp:connect` | MCP config reference for all clients |
| `/mcpwp:tools` | Browse the site's tools by category |
| `/mcpwp:elementor` | Elementor building guide — layouts, widgets, flex grids |
| `/mcpwp:design` | Modern web design principles for page building |
| `/mcpwp:status` | Check plugin version, Elementor, and available updates |

### Agents
| Agent | What it does |
|-------|-------------|
| `wp-builder` | Builds and edits Elementor pages with design best practices |

## Quick Start

1. Install the plugin (see above)
2. Run `/mcpwp:setup` — it walks you through everything:
   - Installing mcpwp on your WordPress site
   - Generating an API key
   - Configuring your MCP client
   - Verifying the connection
3. Start building: "Build a landing page with a hero section, 3 feature cards, and a CTA"

## What is mcpwp?

mcpwp is a free WordPress plugin that turns your site into an MCP server. Any AI assistant that supports the Model Context Protocol (Claude, Gemini, GPT, Cursor, Windsurf) can manage the entire site through natural language.

- **260+ MCP tools** (180+ on the Free plan; Pro adds the rest)
- **Elementor 4 support** with validation, auto-fix, and blueprint system
- **Role-scoped API keys** (admin, designer, author, editor, custom)
- **Free plan** plus **Pro** for SEO market data, WooCommerce, staging, Figma and more

## Links

- **Plugin website:** https://mcpwp.net
- **WordPress plugin:** https://updates.mcpwp.net/mcpwp-latest.zip
- **Documentation:** https://docs.mcpwp.net
- **Plugin on WordPress.org:** (pending approval, slug: mcpwp)

## License

MIT
