---
name: seo-loop
description: Run a daily or weekly SEO loop on a connected WordPress site — read Search Console/DataForSEO signals, judge which queries are worth acting on, fix pages before writing new ones, link internally, and measure conversions (not just clicks). Use this whenever the user asks for an SEO check-in, wants to improve rankings/CTR, mentions Search Console data, decaying traffic, content refreshes, internal linking, or "AI search"/AI Mode visibility on a site connected via mcpwp. Encodes a disciplined read→judge→fix→write→measure cycle instead of dumping keyword lists — reach for this before running ad hoc SEO tool calls.
user-invocable: true
---

# The SEO Loop

A disciplined cycle for growing organic traffic on a site connected via mcpwp, built from a
public case study that drove a 47x impression gain in two weeks by fixing pages instead of
writing new ones. The judgment matters more than the tools: most "SEO work" is scanning long
keyword exports and doing nothing with them. This loop flags six specific signals, fixes the
highest-leverage one before writing anything new, and never ships a change without going
through mcpwp's approval lifecycle.

**Cadence:** run the READ step daily if search-performance data is imported regularly; run the
full loop (through MEASURE) weekly. Don't run FIX/WRITE steps more often than data can move —
rankings don't shift hour to hour.

## The loop, in order

1. **READ** — pull the six flags, nothing else. See `references/flags.md` for each flag's
   definition and exact tool calls.
2. **JUDGE** — decide which flagged items are worth acting on this cycle. Never hand the user a
   raw keyword list; every flag you surface should come with a proposed action or a reason to
   ignore it. Most flagged rows are not worth touching — judgment is the deliverable, not the scan.
3. **KILL before FIX** — for any query you're about to target, check who currently ranks for it
   (`wp_get_rendered_html` or a manual look at the SERP intent). If the ranking pages serve a
   different audience or intent than this site, drop the query. Search volume orders the queue;
   it never decides whether a keyword is worth pursuing — volume tools (Google Suggest vs.
   DataForSEO) disagree by up to 4x, so treat any single number as a rough sort key, not ground
   truth.
4. **FIX before WRITE** — the highest-leverage move is almost always improving a page that
   already exists, not publishing a new one. See `references/fix-recipes.md` for the exact steps
   per flag (`almost_there`, `no_clicks`, page-type mismatch, `decaying`, `untargeted`, `ai_mode`).
5. **LINK** — once a page is fixed, let a strong page carry a stuck one: internal-link from a
   page that already ranks well to one that's stuck, when they're topically related. Use
   `wp_get_content_graph` / `wp_suggest_internal_links` / `wp_apply_internal_link` /
   `wp_validate_internal_links` — never hand-guess link targets, and never create an orphan or a
   spammy link farm. Rotate which pages link out monthly rather than piling links onto one hub.
6. **MEASURE conversions, not clicks** — a page with rising clicks and flat signups is a leak,
   not a win. Where the site has WooCommerce orders or form entries, join them to the landing
   page. If the site can't attribute conversions to a page, say so explicitly and ask the site
   owner for their analytics setup rather than reporting clicks as the success metric.
7. **HYGIENE** — any redirect created as part of a slug change is `301`/`308` (permanent), never
   `302`/`307`. Publish fixes in small batches (a handful of pages, not the whole site) and watch
   indexing before scaling up — mcpwp has no bulk-publish-and-pray mode for a reason.
8. **GOVERNANCE** — every content change goes through mcpwp's approval lifecycle. Log the cycle.
   See "Governance" below — this step is mandatory, not optional, on every run of this loop.
9. **DATA HONESTY** — before reading any Search Console number, confirm data has actually been
   imported. See "Data honesty" below.

## Governance (mandatory every cycle)

mcpwp is approval-gated by design, and this loop must never bypass that:

- **Propose, don't force.** Scoped API keys (`write`, `read`) cannot publish — publishing a
  status change requires an `admin`-scoped key as of 3.13. Draft changes and create approval
  requests; do not attempt to force a publish around that gate. Most write tools in this loop
  default to `approval_required: true` for exactly this reason (e.g. `wp_apply_internal_link`) —
  leave that default alone unless the human explicitly pre-authorized direct writes.
- **Approval lifecycle:** `wp_list_approvals` → human reviews → `wp_approve_request` →
  `wp_apply_approval`. If a proposal is rejected, don't re-propose the same fix next cycle —
  record why with `wp_remember` (see below) so the next run knows it was already tried and declined.
- **Remember owner decisions.** Use `wp_remember(namespace: "seo", key: ..., value: ...)` for:
  decisions the owner made ("keep /pricing targeting 'cost' not 'price'"), rejection reasons,
  and keywords killed in step 3 so you don't re-propose them. Use `wp_recall(namespace: "seo")`
  at the start of every cycle before proposing anything, so the loop doesn't repeat itself.
- **Rate limit:** mcpwp enforces roughly 10 writes per 10 seconds per API key. Batch and pace
  write calls (`wp_bulk_seo`, `wp_batch`, `wp_apply_internal_link` in sequence) rather than
  firing them as fast as possible — a burst will get throttled, not sped up.
- **Weekly log.** Fill out `references/weekly-report.md` every cycle and keep it (or hand it to
  the owner) — flags fired, actions proposed, approvals pending, and what actually converted.
  This is what turns "I ran some SEO tools" into a trackable program.

## Data honesty (read this before trusting any number)

- **Search Console data in mcpwp is import-based, not live.** `wp_get_seo_trends` only returns
  what's already been imported via `wp_import_search_performance`. If nothing has been imported
  — check with `wp_get_seo_trends` returning empty, or ask — **stop and tell the owner**: ask for
  a GSC export (or ask them to wire up an import) rather than guessing at rankings or inventing
  numbers. Never present DataForSEO or keyword-research estimates as if they were Search Console
  truth.
- **DataForSEO "position" is not GSC "average position."** `wp_seo_visibility` /
  `wp_refresh_seo_visibility` / `wp_seo_opportunities` return DataForSEO's own rank-tracking
  position (Pro, spends credit on refresh). `wp_get_seo_trends` returns Google's own average
  position from imported Search Console rows. These are two different measurement systems that
  can and do disagree — never average them together or treat one as confirmation of the other.
  Say which source a number came from whenever you report a position.
- **`wp_keyword_research`'s `enrich=true` DataForSEO metrics** (volume/CPC/competition/
  difficulty) are estimates from yet another provider. Cite the source when comparing volume
  numbers across tools — a 4x disagreement between two volume estimates is normal, not a bug.

## Reference files

- `references/flags.md` — the six flags this loop reads for: definition, exact mcpwp tool calls
  to compute each one, and when a flagged row is worth acting on.
- `references/fix-recipes.md` — step-by-step fix recipes (with tool calls and the approval step)
  for `almost_there`, `no_clicks`, page-type mismatch, `decaying`, `untargeted`, and `ai_mode`.
- `references/weekly-report.md` — the log template to fill in at the end of every cycle.

## What mcpwp doesn't have (don't invent it)

- **No IndexNow / Bing submission tool.** mcpwp can *import* Bing Webmaster data as read-only
  evidence (`wp_import_search_performance` with `provider: bing_webmaster`), but there is no tool
  to push URLs to Bing or IndexNow. Since ChatGPT's search product is built on Bing's index, this
  is a real gap for the AI step of this loop — say so rather than fabricating a submission call.
  `wp_submit_to_google_index` exists (Pro) but only affects Google, and per its own tool
  description only actually gets crawled for `JobPosting` or `BroadcastEvent` structured data —
  a 200 response for an ordinary page is not evidence it will be indexed.
- **No direct conversion/analytics pull.** mcpwp has WooCommerce order tools and form-entry tools,
  but nothing that automatically joins "this landing page" to "these conversions" outside of what
  you construct yourself from `wc_list_orders` / `wp_get_form_entries` plus the page's own
  traffic data. If the site has no WooCommerce and no forms plugin, say the site can't attribute
  conversions rather than treating clicks as the success metric.
- **No redirect-creation tool.** `wp_update_page`'s own `slug` parameter warns that changing it
  "breaks existing links unless a redirect plugin is active" — mcpwp cannot create the 301/308
  itself. Before changing any slug, confirm a redirect plugin is active (`wp_detect_plugins`) and
  have the owner set up the redirect through that plugin or server config.
