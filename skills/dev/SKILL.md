---
name: dev
description: MCPWP plugin development guide — add tools, bump versions, run tests, use the mcpwp_register_tools hook API. Use when building or modifying the MCPWP WordPress plugin.
user-invocable: true
---

# MCPWP Plugin Dev Guide

For agents and contributors working on the MCPWP plugin source at `mcpwp/`. $ARGUMENTS = task or question.

## Repo Layout

```
wp-ai-operator/
  mcpwp/            ← Plugin source (volume-mounted to local WP)
    mcpwp.php       ← Bootstrap: version constants, requires, hook registration
    includes/
      mcp/
        class-spai-mcp-free-tools.php    ← Free tier tools (~125 tools)
        class-spai-mcp-pro-tools.php     ← Pro tier tools (~130 tools)
        class-spai-mcp-tool-registry.php ← Base registry (define_tool, get_tools, get_tool_map)
        class-spai-custom-tool-registry.php ← Third-party hook API (mcpwp_register_tools)
      api/
        class-spai-rest-mcp.php          ← MCP dispatch: tools/list, tools/call
        class-spai-rest-*.php            ← REST controllers per surface
      core/
        class-spai-analytics.php         ← PostHog server-side events
        class-spai-*.php                 ← Core services (pages, posts, media, SEO…)
    tests/                  ← PHPUnit tests
    docs/                   ← openapi-chatgpt.yaml, blog post, PH comment
  scripts/
    build-wporg.sh          ← Build WP.org free zip
    build-freemius.sh       ← Build Freemius paid zip
    publish_update_release.sh ← Publish to mumega.com static + R2
```

## Adding a New Tool

### 1. Pick the right file

| Tool type | File |
|-----------|------|
| Free-tier tool | `includes/mcp/class-spai-mcp-free-tools.php` |
| Pro-tier tool | `includes/mcp/class-spai-mcp-pro-tools.php` |
| Third-party plugin | `add_filter('mcpwp_register_tools', ...)` in your plugin |

### 2. Define the tool

```php
$tools[] = $this->define_tool(
    'wp_my_tool',
    'Verb-first description. Say what it returns. Mention key use cases and synonyms for BM25 search.',
    array(
        'param_name' => array(
            'type'        => 'string',    // string | number | boolean | array | object
            'description' => 'What this param controls.',
            'required'    => true,
        ),
        'optional_param' => array(
            'type'        => 'integer',
            'description' => 'Items per page (default 20).',
        ),
    ),
    // optional 4th arg: destructive hint
    // optional 5th arg: open_world hint (calls external service)
);
```

### 3. Add the REST route handler

Add a case in `includes/api/class-spai-rest-mcp.php` → `handle_tools_call()`, or create a new REST controller in `includes/api/class-spai-rest-*.php` and register it.

### 4. Fire the analytics hook at every exit point

```php
do_action( 'mcpwp_tool_called', $tool, $category, $duration_ms, $error_code );
// $error_code: '' (success) | 'tool_not_found' | 'execution_error' | 'scope_denied'
```

### 5. Bump the version (3 files — all three required)

```bash
# mcpwp/mcpwp.php
# header:  Version: X.Y.Z
# constant: define( 'MCPWP_VERSION', 'X.Y.Z' );

# mcpwp/readme.txt
# Stable tag: X.Y.Z
# == Changelog ==
# = X.Y.Z =
# * New: description

# version.json (root)
# "version": "X.Y.Z"
# prepend changelog HTML fragment to "changelog" key
```

## Local Test Stack

```bash
# WordPress at http://localhost:8080
# Docker Compose: ~/projects/themusicalunicorn/wp-test/docker-compose.yml
# Plugin is volume-mounted — file edits are live instantly, no deploy needed

# WP-CLI
docker exec wp-test-wordpress-1 wp <command> --allow-root

# Generate API key
docker exec wp-test-wordpress-1 bash -c 'php -r "
require_once \"/var/www/html/wp-load.php\";
\$key = \"mcpwp_\" . bin2hex(random_bytes(24));
update_option(\"mcpwp_api_key\", wp_hash_password(\$key));
echo \$key;
"'

# Quick smoke test
KEY="mcpwp_..."
curl -s http://localhost:8080/wp-json/mcpwp/v1/site-info -H "X-API-Key: $KEY" | jq .capabilities
```

## CI

GitHub Actions on every push to `Mumega-com/mcpwp`:

| Check | What it runs |
|-------|-------------|
| PHP Syntax Lint | `find mcpwp -name '*.php' \| xargs -n1 php -l` |
| PHP 7.4–8.2 Validation | Syntax + static checks per PHP version |
| lint-and-test | PHPUnit (`tests/`) + PHP Syntax Lint |
| Proxy Worker Tests | Vitest on `spai-proxy-worker/` |

**PHP lint locally (if PHP available):**
```bash
find mcpwp -name '*.php' -not -path '*/vendor/*' | xargs -n1 php -l
```

**PHPUnit:**
```bash
cd mcpwp && ./vendor/bin/phpunit tests/
```

## Third-Party Tool Registration (`mcpwp_register_tools`)

Third-party plugins register tools without extending any class:

```php
add_filter( 'mcpwp_register_tools', function( $tools ) {
    $tools[] = [
        'name'        => 'digid_list_listings',   // prefix_action format
        'description' => 'List active real estate listings from the Digid property database.',
        'rest_path'   => '/digid/v1/listings',    // full WP REST route
        'method'      => 'GET',
        'category'    => 'listings',
        'input_props' => [
            'per_page' => [ 'type' => 'integer', 'description' => 'Items per page.' ],
            'status'   => [ 'type' => 'string',  'description' => 'active | sold | pending' ],
        ],
    ];
    return $tools;
} );
```

Required fields: `name`, `description`, `rest_path`
Optional: `method` (GET), `category` ('custom'), `input_props`, `destructive`, `open_world`, `param_remap`

The `rest_path` must be the full WP REST route (e.g. `/digid/v1/endpoint`). MCPWP dispatches directly to it — no `/mcpwp/v1` prefix is added for custom tools.

## Tool Description Quality (BM25)

Bad: `'Get kit CSS.'`
Good: `'Read the Elementor Kit global CSS. Returns custom CSS rules applied site-wide via the Elementor Kit settings.'`

Rules:
- Start with a strong verb (Read, Create, Update, Delete, List, Search, Generate)
- State what it returns
- Include synonyms for the main concepts (e.g. "heading widget" not just "widget")
- Mention key use cases ("Use before wp_set_kit_css to see existing rules")
- Minimum ~60 characters

## Deployment

```bash
# Release to Freemius (paid)
FREEMIUS_BEARER_TOKEN=... bash scripts/release_freemius.sh --version X.Y.Z --skip-bump

# Publish static update manifest + zip to mumega.com
bash scripts/publish_update_release.sh --build

# Push to GitHub
git add -A && git commit -m "feat: description (closes #issue)" && git push origin main
```
