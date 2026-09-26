# The six flags

All six flags are computed from **imported Search Console data** — `wp_get_seo_trends`, which
reads rows previously stored by `wp_import_search_performance`. If that import is empty, stop
and ask the owner for a GSC export (see "Data honesty" in `SKILL.md`) instead of computing these
flags from DataForSEO or keyword-research numbers, which measure different things.

Pull the data once per cycle and derive all six flags from the same pull rather than calling the
tool six separate times with slightly different filters:

```
wp_get_seo_trends(days=90, limit=100)      # broad pull: top queries, top URLs, daily aggregates
```

Then narrow with `url`/`query`/`days` filters per-flag as needed below. Ignore every row that
doesn't clear a flag's threshold — the point of this step is a short, actionable list, not a
transcript of the query export.

---

## 1. `almost_there`

**Definition:** a non-brand query where the page's average position is roughly 3–20 (page 1
bottom through page 2). These are the closest to a click-through win — the content mostly works,
it just needs a nudge.

**How to compute:**
1. `wp_get_seo_trends(days=90, limit=100)` — read `top_queries` (or the per-URL breakdown).
2. Filter rows where `position` is between 3 and 20.
3. Exclude brand queries (the site name, product name, or obvious brand variants — check against
   `wp_get_site_context` / `wp_site_info` for the brand name before filtering).
4. For each surviving row, note the query text, the URL it's tied to, and the current position.

**Worth acting on when:** the query is a real buying/informational intent match for the page
(step 3, KILL, in `SKILL.md`) and impressions are non-trivial (don't chase a position-8 query
with 3 impressions/quarter). See `references/fix-recipes.md#almost_there` for the fix.

---

## 2. `no_clicks`

**Definition:** impressions > 500 and CTR < 0.5% for a query/URL pair. The page is *being shown*
for real search volume but nobody is clicking — usually a title/snippet problem, sometimes a
page-type mismatch (see flag 5).

**How to compute:**
1. From the same `wp_get_seo_trends` pull, filter rows where `impressions > 500`.
2. Compute `ctr = clicks / impressions` for each (or use the `ctr` field directly if present) and
   keep rows where `ctr < 0.005`.

**Worth acting on when:** the query's intent clearly matches what the page could serve — if the
crowd wants something the page fundamentally isn't (see flag 5, `wrong_intent`), fix the intent
mismatch before touching the title. See `references/fix-recipes.md#no_clicks`.

---

## 3. `decaying`

**Definition:** clicks down ≥30% week over week for a query or URL that used to perform.

**How to compute:**
1. Pull two adjacent windows: `wp_get_seo_trends(days=7)` for the current week and
   `wp_get_seo_trends(days=14)` for the trailing fortnight (or, if the daily aggregate array is
   granular enough, slice it into two 7-day buckets yourself from one `days=14` pull — cheaper
   than two calls).
2. For each URL/query with a meaningful click volume in the prior week, compute
   `(prior_week_clicks - current_week_clicks) / prior_week_clicks`. Flag anything ≥0.30.

**Worth acting on when:** the drop isn't explained by seasonality or a site-wide event (check
`wp_get_signals` for site health issues, and check whether the drop is site-wide vs. isolated to
one page — a site-wide drop is an infra/indexing problem, not a content problem). See
`references/fix-recipes.md#decaying`.

---

## 4. `untargeted`

**Definition:** the site ranks for (gets impressions/clicks on) a query that no page deliberately
targets — i.e. it's incidental traffic, not the result of anyone writing for that phrase.

**How to compute:**
1. From `wp_get_seo_trends`, take the query list per URL.
2. For each URL, pull its actual SEO targeting: `wp_get_seo(id)` (Pro — `focus_keyword`, `title`)
   or, on Free, `wp_validate_seo_readiness(id)` plus the page's own title/H1 via
   `wp_get_rendered_copy(id)`.
3. A query is `untargeted` if it doesn't overlap (by stem/phrase) with the page's focus keyword,
   title, H1, or first-paragraph copy, yet is driving real impressions/clicks.

**Worth acting on when:** the untargeted query has enough volume to deserve deliberate targeting
— either folding it into the existing page (if topically adjacent) or flagging it as a candidate
for a new page. Check `wp_get_content_graph` first to confirm no other page on the site already
half-covers it before proposing something new.

---

## 5. `wrong_intent`

**Definition:** the query's intent doesn't match the page type currently ranking for it — e.g. a
comparison query (`"x vs y"`) landing on a single-product page, or a `"how much does x cost"`
query landing on a page with no pricing information.

**How to compute:**
1. For flagged rows from `no_clicks` and `almost_there`, read the query text for intent
   signals: comparison (`vs`, `or`, `compare`), pricing (`cost`, `price`, `how much`), how-to,
   local (`near me`), transactional (`buy`, `best`).
2. Read what the ranking page actually is: `wp_get_rendered_copy(id)` (does it answer the
   question in the first two lines?) and `wp_get_page_by_slug` / `wp_list_pages` to confirm the
   page's structural type (product page, blog post, landing page, pricing page).
3. Flag a mismatch when the query's intent category doesn't match the page's actual job — a
   title/meta rewrite cannot fix this; the page needs a different shape of content or a different
   target page entirely.

**Worth acting on when:** the mismatch is cheap to close (adding a comparison table or a pricing
section to an existing page) vs. needing a whole new page — prefer the cheaper fix per
`references/fix-recipes.md#page-type-mismatch`.

---

## 6. `ai_mode`

**Definition:** queries of 7+ words — long, conversational phrasing typical of AI Mode / AI
Overviews and chat-style search (ChatGPT, Perplexity, Gemini) rather than classic 2-3 word
keyword search.

**How to compute:**
1. From the `wp_get_seo_trends` query list, count words per query string (split on whitespace).
2. Flag queries with 7 or more words.
3. Group them by theme — several long queries hitting the same page or the same unanswered
   sub-topic are the strongest signal (one long query could be noise; a cluster is a pattern).

**Worth acting on when:** the exact phrasing of a cluster reveals a question the site doesn't
answer anywhere, in those words. Build content in the searcher's exact wording — see
`references/fix-recipes.md#ai_mode`. Also see the "AI where the AI looks" note in `SKILL.md` —
mcpwp has no IndexNow/Bing-submission tool, which matters here since ChatGPT search runs on
Bing's index.
