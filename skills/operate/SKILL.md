---
name: operate
description: Operate a client's WordPress site brand-first and governed — the core loop every marketing/website agent runs on connect. Onboard, read the brand book, then build/edit on-brand with approval-first safety. Use the moment you connect to any mcpwp site to do real work.
user-invocable: true
---

# Operate a WordPress Site (brand-first, governed)

This is the **operating loop** — the agent-half of a deployable site. Reference skills tell you
*what tools exist* (`/mcpwp:tools`, `/mcpwp:elementor`, `/mcpwp:design`). This skill tells you *how to
behave* when you take over a live client site: read the brand before you touch anything, conform to it,
and never mutate production without a gate. Proven on real client sites (digid.ca, crophelp.ai).

> The brand book is the constraint that turns AI from a slop-generator into an on-brand operator.
> Read it first, every time. Everything you produce conforms to it or it does not ship.

## The loop (run in order, every session)

### 1. Onboard — know the site before you act
Call **`wp_onboard`** first, always. One call returns: site identity, content inventory, active
integrations, the full tool surface by category, the embedded site context, and recommended first
actions. Never start work from assumptions — start from the briefing.

If anything is unclear after onboard, call `wp_site_info`, `wp_detect_plugins`, `wp_get_site_state`.

### 2. Read the brand book — the central context
Call **`wp_get_site_context`**. This is the site's master style guide / system context — the
**brand crystal**. It defines: brand identity (name, tagline, mission, voice), color palette (CSS vars +
hex, dark/light themes), typography scale, spacing rhythm, navigation, page structure, and design rules.

- **Everything you generate conforms to this.** Colors, type sizes, spacing, section patterns, voice.
  Off-brand output does not ship — that is the whole product.
- The response includes `effective_context` and an `inheritance` block (`scope`, `archetype_class`,
  `style`, `matched`). When a site inherits from a base context (a snapshot/blueprint), `effective_context`
  is what actually applies — honor it.
- **New client with no context yet?** Establish it first with **`wp_set_site_context`** (markdown:
  identity → palette → type → spacing → nav → page structure → design rules), get it approved by the
  human, *then* build. A site with no brand book is the first thing to fix, not skip.

### 3. Read state + pick the governed path
- **`wp_get_site_state`** — content, graph, SEO, approvals, events, capabilities, recommended actions.
- **`wp_get_agent_playbook`** (no arg) lists deterministic contracts; call with a name for the full
  contract (required reads, tool order, validation gates, approval gates, rollback path, stop conditions).
  Current playbooks: `build_gutenberg_page`, `update_gutenberg_section`, `seo_audit_triage`,
  `internal_link_improvement`, `rollback_change`. **Follow the matching playbook** rather than improvising.

### 4. Do the work — brand-first
- Build/edit pages, posts, menus, media, SEO — each conforming to the brand book from step 2.
- Elementor work: read `/mcpwp:elementor` first; always `wp_get_elementor_summary` before editing;
  prefer surgical tools (`wp_edit_section`, `wp_edit_widget`, `wp_patch_elementor`) over full-page rewrites.
- Use the brand palette/type/spacing values literally — do not invent new colors or sizes.
- Generate on-brand copy in the brand voice; use `wp_keyword_research` to ground content in real search demand.

### 5. Govern every mutation
- **Approval-first.** Production-facing changes go through the approval lifecycle
  (`wp_list_approvals` → `wp_approve_request` → `wp_apply_approval`), never a silent direct write,
  unless a playbook is explicitly marked read-only or the human pre-authorized the specific change.
- **Validate before publish:** `wp_validate_seo_readiness`, `wp_validate_blocks` / Elementor warnings.
- **Rollback on regret:** every applied change stores a rollback payload — `wp_rollback_approval`.
- **Coherence check:** `wp_get_content_coherence_report` / `wp_get_signals` to confirm the brand crystal
  still holds after a batch of work.

## Hard rules (never)
- Never push raw HTML/JS as a shortcut into Gutenberg or as page content. Use native blocks / Elementor data.
- Never invent facts, citations, products, prices, or SEO claims. Ground everything in real site data.
- Never introduce a color, font, or spacing value that is not in the brand book.
- Never mutate production without an approval gate (unless read-only playbook or explicit human OK).
- Customer-facing sends (email, social, publish-live) go through the human gate — propose, don't broadcast.

## Stop conditions
Stop and ask the human when: the brand book is missing or contradicts the request; a change is
destructive with no rollback; an approval is required but absent; the task needs a fact you cannot verify
on the site; or a playbook's stop condition triggers.

## Why this is the snapshot's agent-half
A deployable site snapshot = **this operating skill (the agent's behavior) + the site's brand book +
blueprint (the WordPress structure)**. Author a client's brand book into `wp_set_site_context`, capture
their structure as a blueprint, and this skill is the squad's reusable playbook for running it. One skill,
every client site. That is how one proven build (digid.ca) becomes a repeatable product.

## Reference skills (load as needed)
`/mcpwp:tools` (discover tools) · `/mcpwp:elementor` (page building) · `/mcpwp:design` (design
principles) · `/mcpwp:status` (site/plugin health) · `/mcpwp:connect` (wiring a client).
