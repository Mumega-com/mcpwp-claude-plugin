---
name: elementor
description: Build and edit Elementor pages via mcpwp — layout modes, widget keys, flex grids, blueprints, validation, CSS regeneration. Use before working with Elementor data.
user-invocable: true
---

# Elementor Building Guide

How to build and edit Elementor pages via mcpwp. $ARGUMENTS = action or page ID.

## Layout Modes

Check `wp_site_info()` → `elementor.layout_mode`:
- **container** (Elementor 3.x+ flexbox): `container > container(isInner:true) > widget`
- **classic** (legacy): `section > column(_column_size sums to 100) > widget`

Most modern sites use container mode.

## Building a Full Page

```
wp_build_page(
  title: "Services",
  sections: [
    {type: "hero", heading: "Our Services", subheading: "What we do best", button_text: "Get Started", background: "#0a0a0a"},
    {type: "features", columns: 3, heading: "Why Choose Us", items: [
      {icon: "fas fa-rocket", title: "Fast", desc: "Lightning speed", button_text: "Learn More", url: "/fast"},
      {icon: "fas fa-shield-alt", title: "Secure", desc: "Bank-grade security"},
      {icon: "fas fa-heart", title: "Reliable", desc: "99.9% uptime"}
    ]},
    {type: "cta", heading: "Ready?", button_text: "Start Now", button_url: "/contact", background: "#1e40af"}
  ]
)
```

## Surgical Edits

### Edit one widget
```
# 1. Get the page structure
wp_get_elementor_summary(id=42)

# 2. Find the widget ID from the summary
# 3. Edit it
wp_edit_widget(page_id=42, widget_id="abc12345", settings={title_text: "New Title", title_color: "#FF0000"})
```

### Edit a section/container
```
wp_edit_section(page_id=42, element_id="xyz789", settings={
  background_color: "#1e293b",
  padding: {top: "80", bottom: "80", unit: "px"}
})
```

### Add a section to an existing page
```
wp_add_section(id=42, position="end", blueprint="cta", params={
  heading: "Get Started Today",
  button_text: "Contact Us",
  button_url: "/contact"
})
```

## Widget Control Keys

**Always check before setting properties:**
```
wp_get_widget_schema(widget_type="button")
```

### Common Widgets Quick Reference

**heading:** `title`, `header_size` (h1-h6), `align`, `title_color`

**button:** `text`, `link` ({url, is_external}), `size` (xs/sm/md/lg/xl), `align`, `background_color`, `button_text_color`, `border_radius`

**icon-box:** `selected_icon` ({value, library}), `title_text`, `description_text`, `position` (top/left/right), `align` (left/center/right), `primary_color`, `title_color`

**image:** `image` ({url, id}), `image_size`, `align`, `link`

**text-editor:** `editor` (HTML string), `align`

## Container Flex Properties

For card grids in container mode:

**Parent container:**
- `flex_direction: "row"`
- `flex_wrap: "wrap"`
- `flex_gap: {size: 20, unit: "px"}`
- `content_width: "boxed"`

**Child card containers:**
- `_element_width: "initial"`
- `width: {size: 30, unit: "%"}` (3-col) / `{size: 47, unit: "%"}` (2-col)
- `background_color: "#FFFFFF"`
- `border_radius: {top_left: "12", top_right: "12", bottom_right: "12", bottom_left: "12", unit: "px"}`
- `box_shadow_box_shadow_type: "yes"`
- `box_shadow_box_shadow: {horizontal: 0, vertical: 4, blur: 20, spread: 0, color: "rgba(0,0,0,0.08)"}`
- `padding: {top: "30", bottom: "30", left: "30", right: "30", unit: "px"}`

## Validation

mcpwp auto-validates and fixes:
- Missing element IDs → auto-generated
- Missing `isInner` on nested containers → auto-set
- Wrong widget keys → renamed if known, warned if unknown
- Widget type typos → fuzzy-matched with "did you mean?" suggestions

Use `dry_run: true` on `wp_set_elementor` to validate without saving.

## CSS Regeneration

After bulk changes, regenerate CSS:
```
wp_regenerate_elementor_css(force: true)        # All pages
wp_regenerate_elementor_css(page_id: 42)        # Single page
```
