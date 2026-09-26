# Weekly SEO loop report — template

Fill this in at the end of every cycle (weekly, or daily if running the READ step daily). Keep
past reports — either in `wp_remember(namespace: "seo", key: "weekly_report_<date>", value: ...)`
or handed directly to the owner — so the next cycle can see what already happened instead of
re-proposing the same fix.

```markdown
## SEO loop report — <site> — week of <YYYY-MM-DD>

### Data source check
- Search performance import status: <last import date / provider(s) / "nothing imported — asked owner for GSC export">
- DataForSEO configured: <yes/no> — visibility snapshot last refreshed: <date or "never">

### Flags fired this cycle
| Flag | Query / URL | Metric | Action taken |
|------|-------------|--------|---------------|
| almost_there | ... | position X | proposed / fixed / skipped (why) |
| no_clicks | ... | impressions X, CTR Y% | ... |
| decaying | ... | clicks -X% WoW | ... |
| untargeted | ... | ... | ... |
| wrong_intent | ... | ... | ... |
| ai_mode | ... | word count X | ... |

(Only list rows that were judged worth acting on or explicitly worth noting as skipped — this
report is not a keyword export.)

### Killed keywords this cycle
- <query>: killed because <who ranks for it / wrong audience / wrong intent>

### Actions proposed / approvals
| Change | Page | Approval status | Approval ID |
|--------|------|------------------|-------------|
| ... | ... | pending / approved / applied / rejected | ... |

### Internal links added/rotated
- <source page> → <target page>: <reason>

### Conversion check
- Pages reviewed for conversion (not just clicks): <list>
- Attribution available: <yes, via WooCommerce/forms — numbers below> / <no — asked owner for analytics access>
- Leaks found (clicks up, conversions flat/down): <list, or "none found">

### Hygiene
- Redirects needed this cycle: <list slug changes + redirect-plugin status, or "none">
- Batch size published this cycle: <N pages/sections — kept small intentionally>

### Owner decisions to remember
- <decision>: stored via wp_remember(namespace="seo", key="...", value="...")

### Carried into next cycle
- <anything pending approval, waiting on data, or waiting on the owner>
```

## Why this shape

- **Data source check first** — every number below it is worthless if nothing's been imported;
  this line forces that check before anyone reads further.
- **Flags fired, not flags scanned** — the table only holds rows judged worth acting on. If a
  cycle produces zero rows worth acting on, write "no actionable flags this cycle" — that is a
  valid, honest result, not a failure to find something.
- **Killed keywords are logged, not just dropped** — otherwise the KILL step gets redone every
  cycle on the same bad keyword.
- **Conversion check is its own section** — it's the part most SEO reporting skips, and it's the
  one the case study insists on: clicks that don't convert are a leak, not a win.
