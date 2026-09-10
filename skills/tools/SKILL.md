---
name: tools
description: Browse the mcpwp MCP tools by category (260+ with Pro, 180+ on Free) — pages, Elementor, WooCommerce, media, SEO, and more. Use to discover available tools before working with a WordPress site.
user-invocable: true
---

# mcpwp Tools Reference

Browse and learn about the mcpwp MCP tools (260+ with Pro, 180+ on Free). $ARGUMENTS = category name or tool name.

## Categories

| Category | Count | Description |
|----------|-------|-------------|
| `site` | ~37 | Site info, menus, options, custom CSS, design references, guides, workflows |
| `content` | ~28 | Pages, posts, drafts, bulk create/update, search, clone, meta |
| `elementor` | ~12 | Get/set Elementor data, edit sections, edit widgets, find-replace |
| `elementor-build` | ~8 | build_page from blueprints, landing pages, clone, globals |
| `elementor-templates` | ~15 | Templates, archetypes, reusable parts |
| `elementor-theme` | ~10 | Theme builder templates, conditions, custom code |
| `elementor-info` | ~5 | Widget schemas, widget help, status, CSS regeneration |
| `media` | ~7 | Upload file/URL/base64, delete, screenshot |
| `taxonomy` | ~5 | Categories, tags, custom taxonomy terms |
| `gutenberg` | ~4 | Blocks content, block types, patterns |
| `admin` | ~16 | API keys, rate limits, settings, updates, integrations, feedback |
| `seo` | ~10 | Meta tags, analysis, bulk SEO, scan, report, noindex, Google indexing |
| `woocommerce` | ~21 | Products, orders, categories, tags, customers, analytics, archetypes |
| `learnpress` | ~18 | Courses, lessons, quizzes, curriculum, categories, stats |
| `webhooks` | ~7 | Create/test/monitor webhook deliveries |

## Essential Tools (Start Here)

### First Connection
- `wp_introspect()` — Full site briefing: identity, tools, capabilities, recommended actions
- `wp_site_info()` — Site name, URL, theme, plugins, Elementor status
- `wp_get_site_context()` — Brand voice, design system, target audience

### Content
- `wp_list_pages(per_page, status)` — List all pages
- `wp_create_page(title, content, status)` — Create a page
- `wp_update_page(id, title, content)` — Update a page
- `wp_search(query)` — Search across all content

### Elementor Page Building
- `wp_build_page(title, sections[])` — Build entire page from blueprints
- `wp_get_elementor(id)` — Read full Elementor JSON for a page
- `wp_get_elementor_summary(id)` — Quick overview: section count, widget types, IDs
- `wp_set_elementor(id, elementor_data)` — Write full Elementor JSON
- `wp_edit_widget(page_id, widget_id, settings)` — Change one widget's settings
- `wp_edit_section(page_id, element_id, settings)` — Change section/container settings
- `wp_add_section(id, position, elements)` — Insert a new section
- `wp_get_widget_schema(widget_type)` — Get valid control keys for any widget

### Media
- `wp_upload_media_from_url(url, title, alt)` — Download and upload an image
- `wp_upload_media_b64(data, filename, title)` — Upload base64 image
- `wp_list_media(per_page, search)` — Browse media library

## Blueprint Types (for wp_build_page)

Each creates a fully styled Elementor section:

| Type | What it generates |
|------|------------------|
| `hero` | Full-width hero with heading, subheading, CTA button |
| `features` | 3-column icon-box grid with cards, shadows, buttons |
| `cta` | Call-to-action with dark background, centered text, button |
| `pricing` | Price table columns with features lists |
| `faq` | Accordion with questions and answers |
| `testimonials` | Quote cards with name, title, optional photo |
| `text` | Simple text section with heading and paragraph |
| `gallery` | Image gallery grid |
| `contact_form` | Contact form (requires form plugin) |
| `map` | Google Maps embed |
| `countdown` | Countdown timer |
| `stats` | Number counters with labels |
| `logo_grid` | Logo/partner grid |
| `video` | Video embed section |

## API Key Roles

Different roles limit which tool categories are available:

| Role | Categories | ~Tools | Use case |
|------|-----------|--------|----------|
| `admin` | All | 260+ | Full site management |
| `designer` | elementor*, gutenberg, media, site | ~82 | Page building |
| `author` | content, media, taxonomy | ~40 | Content writing |
| `editor` | content, elementor*, media, taxonomy, seo | ~99 | Content + design |
| `custom` | Pick your own | Variable | Specific use cases |

Create restricted keys: `wp_create_api_key(label, role, tool_categories[])`
