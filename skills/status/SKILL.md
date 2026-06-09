---
name: status
description: Check mcpwp plugin status on a connected WordPress site — version, Elementor, capabilities, and available updates.
user-invocable: true
---

# mcpwp Status

Check the current state of a mcpwp-connected WordPress site. $ARGUMENTS = site name or leave blank for the active connection.

---

## Connection Health

Call `wp_site_info()` to get a full status snapshot:

```
wp_site_info()
```

Look for:
- **Plugin version** — compare to latest at https://mumega.com/spai-updates/version.json
- **Elementor status** — active, version, layout mode (`container` vs `classic`)
- **WooCommerce** — active or not
- **PHP version** — must be 7.4+, ideally 8.1+
- **WordPress version** — must be 6.0+

---

## Check for Updates

Fetch the update manifest:
```
wp_check_update()
```

Or check manually:
```bash
curl -s https://mumega.com/spai-updates/version.json | jq '{version, changelog}'
```

If an update is available, trigger it:
```
wp_trigger_update()
```

---

## Capabilities Check

Call `wp_introspect()` for the full capability map — shows which tool categories are enabled for your current API key role.

Key things to verify:
- `elementor` tools available → Elementor is active and connected
- `woocommerce` tools available → WooCommerce detected
- `learnpress` tools available → LearnPress detected
- `admin` tools available → key has admin role

---

## Open Issues (mcpwp Plugin)

Current known issues and planned work on the plugin:
- GitHub: https://github.com/Mumega-com/mcp-for-wp/issues

Notable open issues:
- #193: Phase 2 revenue and marketing plan
- #41: WordPress.org assets (icon, banner, screenshots)
- #4: Agency tier plugin

---

## API Key Info

Check which keys exist and their roles:
```
wp_list_api_keys()
```

Create a restricted key for a specific use case:
```
wp_create_api_key(label="designer-key", role="designer")
```

Revoke a compromised key:
```
wp_revoke_api_key(key_id=KEY_ID)
```

---

## Common Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Tools missing | Key role too restricted | Create new key with wider role |
| Elementor tools unavailable | Elementor not active | Activate Elementor in WP Admin |
| Slow responses | Large site, no caching | Enable caching plugin |
| 429 Too Many Requests | Rate limit hit | Check wp_get_rate_limits() |
| Update stuck | Stale update option | Clear `mcpwp_update_info` option |
