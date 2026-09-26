# Fix recipes

Every recipe below ends the same way: propose the change, put it through mcpwp's approval path,
and only then write it live. Two different mechanics exist depending on how the page is built —
check which one applies before you start:

- **Gutenberg pages/posts** — `wp_set_blocks` and `wp_patch_block_section` default to
  `approval_required: true`. Calling them with a non-admin key *creates a real pending approval
  request* automatically; you don't need a separate step to create one. Same for
  `wp_apply_internal_link`.
- **Elementor pages** — `wp_edit_widget`, `wp_edit_section`, `wp_set_elementor` write directly;
  there is no automatic approval-request object for Elementor edits today. Treat this as a real
  gap, not a shortcut: get explicit human sign-off on the exact before/after text before calling
  the write tool, and log the decision with `wp_remember`. Check `wp_elementor_status` /
  `wp_site_info` first to know which path you're on.
- **SEO metadata** (`wp_set_seo`, `wp_bulk_seo`, Pro) also writes directly — same rule: confirm
  with the human before calling it in this loop, don't rely on an approval object that doesn't
  exist for this surface.

After any write: `wp_validate_seo_readiness(id)` to confirm nothing regressed, and
`wp_remember(namespace: "seo", ...)` to log what changed and why.

---

## `almost_there`

The query is already close (position 3–20). Usually a single sentence closes the gap — don't
rewrite the page.

1. `wp_get_seo_trends(url: <page_url>, days: 90)` — export every query tied to this URL.
2. `wp_get_rendered_copy(id)` — read what the page currently says, including headings.
3. Compare: which queries in step 1 aren't covered by any existing heading or paragraph
   ("uncovered clusters")? That's the gap.
4. Draft **one sentence** that uses the exact flagged query phrase, placed where it fits
   naturally (near the most relevant existing heading). If it genuinely doesn't fit anywhere on
   this page, draft a short new section instead (2–3 H2s) covering the uncovered cluster — the
   "1-hour update": same page, more coverage, not a new page.
5. Apply:
   - Gutenberg: `wp_set_blocks(id, blocks: [...])` (creates a pending approval) or
     `wp_patch_block_section`.
   - Elementor: `wp_add_section` or `wp_edit_widget` after getting sign-off (see mechanics above).
6. Get the change approved (`wp_list_approvals` → `wp_approve_request` → `wp_apply_approval`)
   for Gutenberg, or explicit human OK for Elementor, then apply.
7. Saving the page bumps `post_modified` automatically — no separate "update date" step needed.
8. Request indexing: Pro sites can try `wp_submit_to_google_index` (URL_UPDATED), but read its
   tool description first — it only actually gets crawled for JobPosting/BroadcastEvent
   structured data, so a 200 response here is not proof of re-indexing for an ordinary page. For
   everything else, indexing happens on Google's own schedule; don't oversell this step to the
   owner.

---

## `no_clicks`

High impressions, near-zero CTR. First check whether this is a snippet problem or a **page-type
mismatch** (see the next recipe) — a title rewrite cannot fix the wrong page type.

1. `wp_get_rendered_copy(id)` and `wp_get_seo(id)` (Pro) or `wp_validate_seo_readiness(id)`
   (Free) — read the current title, meta description, URL, H1, and opening sentence.
2. Confirm page type matches intent (see `#page-type-mismatch` below) before touching copy.
3. Rewrite, working the exact flagged query into: the title, the meta description, the URL slug
   (only if the current slug is a poor match — see the redirect warning below), the H1, and the
   first sentence. Answer the query in the first two lines of body copy — don't bury the answer.
4. If the slug changes: **mcpwp has no redirect-creation tool.** `wp_update_page`'s own `slug`
   parameter description says changing it "breaks existing links unless a redirect plugin is
   active." Before changing a slug, confirm a redirect plugin is active
   (`wp_detect_plugins`) and tell the owner a 301/308 redirect from the old URL is required — set
   it up via that plugin or server config, outside mcpwp. Don't change a slug you can't redirect.
5. Apply via `wp_set_seo` (Pro, title/description/canonical) and `wp_update_page`/`wp_set_blocks`
   (H1/body) — get sign-off first per the mechanics note above, since neither of these auto-gates
   through approval.
6. `wp_validate_seo_readiness(id)` after writing to confirm title/meta/H1 are consistent.

---

## `page-type-mismatch` (the `wrong_intent` flag)

No title rewrite fixes this — the page is structurally the wrong shape for what the query wants.

1. Classify the query's intent from its wording: comparison (`vs`, `or`, `compare`), pricing
   (`cost`, `price`, `how much`), how-to/informational, transactional (`buy`, `best`, `near me`).
2. Check what the ranking page actually is (`wp_get_rendered_copy`, `wp_list_pages`/
   `wp_list_posts`) — is it a comparison page, a calculator/pricing table, a how-to guide, a
   product page?
3. If the gap is small (a comparison query landing on a single-product page that could add a
   comparison table; a pricing query landing on a page missing a price), extend the existing
   page — cheapest fix, same URL, no redirect risk.
4. If the gap is structural (a "how much does X cost" query landing on a blog post with no
   pricing anywhere near the business model), the fix is a **new page of the right type**, linked
   from the existing one (see LINK in `SKILL.md`) — don't force the wrong content into the wrong
   shape.
5. Whichever path: draft the addition/new page, get approval per the mechanics above, then write.

---

## `decaying`

Clicks down ≥30% week over week. Diagnose before fixing — a decay with no content problem
doesn't get fixed by adding a sentence.

1. `wp_get_signals` — check for site-wide health issues (this rules out "the whole site broke"
   before you blame one page).
2. Confirm the drop is isolated to this URL/query, not site-wide, by comparing several URLs in
   the same `wp_get_seo_trends` pull.
3. `wp_get_rendered_copy(id)` — has the page gone stale (outdated info, old year in the title,
   broken elements)? Freshness signals matter for both classic and AI-mode ranking.
4. If stale: refresh the outdated specifics (dates, prices, screenshots, examples) and treat the
   rest like `almost_there` — add coverage for whatever query cluster is decaying fastest.
5. If a competitor displaced the page (check the live SERP, not just mcpwp data): this may not be
   fixable by content alone — flag it for the owner rather than proposing a fix mcpwp can't
   verify will work.
6. Apply and log via the same approval mechanics as above.

---

## `untargeted`

The site gets traffic for a query nobody deliberately targeted. Decide: fold it into an existing
page, or flag it as a new-page candidate — don't do both, and don't skip the KILL check.

1. `wp_get_content_graph(post_types: "page,post", limit: 200)` — confirm no existing page already
   half-covers this topic (avoid creating a competing/duplicate page).
2. If a topically-adjacent page exists: treat this exactly like `almost_there` — add a section
   using the query's exact wording.
3. If no page covers it and volume/intent justify a new page (after the KILL check in `SKILL.md`
   step 3 — check who else ranks for it and whether that's the right crowd for this site): draft
   a new page brief (title, target query, intent, outline) and get the owner's go-ahead before
   creating it — a new page is a bigger commitment than a one-sentence edit.
4. Once approved, build with `wp_build_page` (Elementor) or `wp_create_post`/`wp_create_page`
   (Gutenberg) as a **draft**, run `wp_validate_seo_readiness` before publishing, then publish
   through the approval path (publishing requires an admin-scoped key as of mcpwp 3.13 — a
   write-scoped key can draft but not flip it live).
5. Once live, link it from the strongest related existing page (`wp_apply_internal_link`) so it
   isn't an orphan — see LINK in `SKILL.md`.

---

## `ai_mode`

Long, conversational queries (7+ words) — the phrasing people actually type into AI Mode/AI
Overviews and chat search. Build in their exact wording; don't paraphrase it into "SEO-speak."

1. Group the flagged long queries by theme (see `references/flags.md#6-ai_mode`) — a cluster of
   3+ similar long queries is a real signal; one outlier query usually isn't worth a dedicated fix.
2. Check whether an existing page already partially answers the cluster
   (`wp_get_content_graph`, `wp_get_rendered_copy` on likely candidates). If yes, treat as
   `almost_there`: add a section that answers the question **in the query's own words**, ideally
   as a direct Q&A-style heading (`## <the question, near-verbatim>`) followed by a direct
   two-sentence answer — this is the shape AI Overviews and chat answers tend to lift.
3. If no page answers it, this is a candidate for a new FAQ-style section or page — draft, get
   sign-off, publish through the approval path, same as `untargeted` step 3–4.
4. **Know the gap:** mcpwp has no IndexNow or Bing-submission tool, and `wp_submit_to_google_index`
   only affects Google (and only reliably for JobPosting/BroadcastEvent content). Since ChatGPT
   search runs on Bing's index, tell the owner that getting this content into Bing/IndexNow needs
   a tool or plugin outside mcpwp — don't imply mcpwp can push it there.
5. Re-check `wp_get_seo_trends` for this query cluster next cycle rather than assuming the fix
   worked — AI-mode visibility is harder to verify than classic SERP position, since there's no
   "position" metric for an AI answer.
