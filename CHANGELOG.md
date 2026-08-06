# Balfour Beatty Audits Dashboard — Change Log

**Deployed:** [ahanseter.github.io/balfour_beatty_audits_dashboard_report](https://ahanseter.github.io/balfour_beatty_audits_dashboard_report/)

---

## Data Updates

### April 2026 Data (applied to repo copy)
- Added audit events, question observations, and tasks through April 2026
- Extended month key maps, `kpiTotals`, `rByMonth`, `pg`/`ng` bar chart data

### May 2026 Data (applied to OneDrive copy)
- Added audit events, question observations, and tasks through May 2026
- Added two new location groups: **RAIL** and **Shared** (first appear in May)
- Extended all data structures: `kpiTotals`, `rByMonth`, `_negQ`, `_posQ`, `pg`/`ng`
- Updated status bar and footer to read "Jan–May 2026"

**Final event counts:** Jan=28, Feb=53, Mar=51, Apr=36, May=39 (207 total)  
**Final task counts:** Feb=4, Mar=7, Apr=5, May=8 (24 total)

### June 2026 Data (applied to OneDrive copy, sourced from `v3_BalfourBeattyAuditEventData_20260702.csv`, `v4_BalfourBeattyAuditEventQuestionData_20260702.csv`, `v3_BalfourBeattyAuditTaskData_20260702.csv`)
- Added 32 audit events for June, all in **SE HIGHWAYS** (20) and **SW HIGHWAYS** (12) — no WATER/RAIL/Shared activity this month
- Added 1 new task (To Do, Austin group, created 2026-06-29)
- Extended all data structures with a `jun` key: `kpiTotals`, `_rbmByGroup`, `_negQ`, `_posQ`, `pg`/`ng`, and both month-key maps (`moMap` and the inline map in the group-bar `k` calculation)
- Recomputed all `ytd` totals as Jan–Jun sums (e.g. `kpiTotals.all.ytd` pos 6952→7591, neg 107→114)
- Updated status bar and footer to read "Jan–June 2026"

**Final event counts:** Jan=28, Feb=53, Mar=51, Apr=36, May=39, Jun=32 (239 total)  
**Final task counts:** Feb=4, Mar=7, Apr=5, May=8, Jun=1 (25 total)

---

## Bug Fixes

### Top 10 Negative Observations undercounting June (and any off-YTD-leader month)
- **Root cause:** `RAW._negQ[group]` only tracked the top 10 negative observations *by Jan–Jun year-to-date total*. When a month's worst items differed from the year's overall worst items, those items simply weren't in the tracked list, so filtering to that month silently dropped them — e.g. June's "All Locations" negative table showed only 2 rows (3 of 7 actual negative responses, 43% coverage) because "Fall Protection/Guardrails", "Scaffolds & Ladders", and "Cranes & Rigging" acceptable-checks spiked in June but ranked low for the full Jan–Jun period.
- **Fix:** Replaced the fixed top-10 list with the *full set of distinct negative observations* per group (49 "all" / 34 SE HIGHWAYS / 24 SW HIGHWAYS / 2 WATER / 2 RAIL / 0 Shared — recomputed directly from the source CSVs), regenerated month-by-month. The existing `sortDesc(negTop).slice(0,10)` render logic already picks the top 10 for *whichever period is selected*, so once the full set is available it naturally produces the correct top 10 (and full coverage) for every month, not just YTD. Also caught in the same pass: WATER's negative list had been left as an empty array (`[]`) since the May data load — same class of bug as the RAIL fix below, just not previously caught — and several `_posQ` entries for WATER/RAIL/Shared were stale. All were corrected from source data.
- `_posQ` (positive observations) intentionally stays capped at top-10-by-YTD — positive responses are spread near-evenly across 85–188 checklist categories per group, so a "top 10" list was always going to show partial coverage there by design (communicated via the "X (%) of total positive responses" line), unlike negatives, which concentrate in a small number of recurring problem areas.

### April bar charts showing March data
- **Root cause:** Two separate `const k=...` declarations exist inside `compute()` — one for KPI totals/top-10, one for the Positive/Negative Responses by Location Group bar charts. The second declaration used a ternary chain that fell through to `'mar'` for any month beyond February.
- **Fix:** Updated the second `k` declaration to use an inline map matching the first:
  ```js
  const k = isAll ? 'ytd' : ({'2026-1':'jan','2026-2':'feb','2026-3':'mar','2026-4':'apr','2026-5':'may'}[months[0]] || 'jan');
  ```

### RAIL top-10 observations not showing
- **Root cause:** `RAW._negQ['RAIL']` and `RAW._posQ['RAIL']` were populated as empty arrays `[]`. The fallback `|| RAW._negQ['all']` never triggered because `[]` is truthy in JavaScript.
- **Fix (interim):** Changed fallback logic to check `.length`.
- **Final fix:** Removed fallback entirely (each group shows only its own data), then populated RAIL's actual observations from the May CSV:
  - 2 negative observations (Segregation/Container Labeling, Electrical)
  - 10 positive observations (Equipment & Tools, Caught-In-Between categories)

---

## Feature Changes

### Total Responses by Month — month filter
- Chart previously showed all months regardless of the selected month filter.
- **Fix:** Demo mode now only includes the selected month(s) in `rByMonth`; live mode pre-filters `RAW.questions` to the selected months before tallying.

### Total Responses by Month — location group filter
- Chart previously showed totals across all groups regardless of the location group filter.
- **Fix:** Added per-group monthly totals (computed from CSV via PowerShell) to a `_rbmByGroup` lookup in demo mode. Live mode now also filters `RAW.questions` by `Location Data Group Name` before tallying.

### Total Responses by Month — zero months on YTD view
- When viewing All Months (YTD), groups with no activity in a given month showed no bar rather than a 0 bar, making the chart inconsistently sized across groups.
- **Fix:** Demo mode now includes all 5 months for every group, with explicit `0` values for months with no data. Live mode pads missing months with `0` when `isAll`.
- Specific month filters are unaffected — only the relevant month's bar is shown.

### Sticky filter bar
- The month/group filter bar now floats beneath the sticky topbar as the user scrolls.
- **Change:** Added `position: sticky; top: 64px; z-index: 199; box-shadow: var(--shadow-md)` to `.filterbar`.

---

## Deployment (2026-08-05 / 2026-08-06)

### Published to GitHub Pages
- Repo: `github.com/ahanseter/balfour_beatty_audits_dashboard_report`, deployed as `index.html` via GitHub Pages.

### Branding
- Replaced the externally-linked Balfour Beatty logo (subject to link rot) with a locally hosted copy (`assets/balfour-logo.png`).
- Added a favicon matching Balfour Beatty's own site branding.

### Access control
- Added a client-side access-code gate shown before the dashboard loads (code is SHA-256 hashed client-side, not stored in plaintext). Unlock persists per-browser via `localStorage`.
- This is a lightweight deterrent appropriate for a low-sensitivity shared report link — not a substitute for real authentication.

### Removed
- Removed the "Upload New Data" CSV-upload feature (button, modal, and the PapaParse dependency) now that data updates flow through the pipeline below.

### Data pipeline
- The dashboard now loads its dataset from a same-origin `data.json` file at startup, falling back to the original built-in demo dataset if `data.json` is missing or malformed — the page never breaks even mid-refresh.
- July 2026 data added (43 new audit events). Jan–Jul YTD totals: 282 events, 8,985 question responses, 28 tasks. The current in-progress month is always excluded from the dataset by design, so partial-month data never appears.
- Data refreshes are sourced from the same SQL exports used historically (`v3/v4_BalfourBeatty*.sql`); a scripted refresh path pulling live from the J. J. Keller API is in progress but not yet active pending an API Management subscription step on Balfour Beatty's side.

### Mobile responsiveness
- Added responsive breakpoints (900px, 560px): KPI and Task Stat cards wrap from 4 columns down to 2, then 1, as viewport width shrinks, instead of a fixed 4-column layout that would overflow on phones.
- KPI and Task Stat numbers use fluid (`clamp()`) font sizing so large counts shrink to fit their card rather than overflowing on small screens.
- The "Customize Dashboard" section-reorder drag handle now uses Pointer Events instead of the HTML5 drag-and-drop API, so reordering works with touch (mobile/tablet) as well as mouse — native HTML5 drag never fires on touch devices.
