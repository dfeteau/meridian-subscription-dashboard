# Meridian Subscription Dashboard — Build Plan

Companion Tableau Public dashboard to the [`subscription-analytics-sql`](../../subscription-analytics-sql) repo. Same "Meridian Analytics" fictional SaaS company and dataset — this project is the visualization/interactivity showcase; the SQL repo is the query-proficiency showcase. Kept as two separate repos on purpose (per your original scoping call), but sharing one dataset so the pipeline between them is visible.

**Why this dashboard, and what it's for:** the GA4 dashboard proved out KPI strip → chart grid → cross-filter action. This one is deliberately scoped to go one level up in interactivity — Dynamic Zone Visibility (view-toggling) and Parameter Actions (click-driven drilldown) — the technique you flagged in the GA4 project log as worth learning next, rather than repeating the same pattern on new data.

**Audience framing (same as GA4):** a hiring manager gives this ~30 seconds. The view-toggle needs to open on the single most important view (Revenue), with Retention and Segmentation one click away — not buried.

---

## Data sources (in `data/`, all Tableau Public–compatible Text File / CSV imports)

| File | Grain | Powers |
|---|---|---|
| `mart_mrr_trend.csv` | month | Revenue view: MRR trend line |
| `mart_mrr_waterfall.csv` | month × bucket (New/Expansion/Contraction/Flat) | Revenue view: MRR growth waterfall bar |
| `mart_channel_performance.csv` | channel | Revenue view: channel bar chart (the click source for the parameter action) |
| `mart_retention_by_channel.csv` | channel × month_offset (0,1,3,6,9,12) | Retention view: retention curve, filtered by the clicked channel |
| `mart_plan_segment_mix.csv` | company_size_band × plan_name | Segmentation view: plan mix by size |
| `mart_industry_churn.csv` | industry | Segmentation view: industry churn ranking |
| `mart_feature_adoption.csv` | feature_name | Segmentation view: feature adoption bar |

Import each as its own Text File data source (same approach as GA4 — Tableau Public has no live DB connections regardless of source, so CSV is the right call here too, not a compromise).

---

## Step 1 — Parameter: `View Selector`

Create a **String** parameter named `View Selector` with a list of allowed values: `Revenue`, `Retention`, `Segmentation`. Add it to a floating "tab strip" — three text/button worksheets or a single worksheet with a parameter action wired to buttons (see Step 4) — this becomes the top-level navigation.

## Step 2 — Build all three view's worksheets first, natively, ungated

Before touching zone visibility, build every worksheet exactly like the GA4 project did — native UI, one chart at a time, verify each opens correctly:

**Revenue view** (3 worksheets)
- KPI strip: current-month MRR, MoM growth %, active paying subscriptions (big-number text marks, same pattern as GA4)
- MRR trend: `BILLING_MONTH` on Columns (continuous), `SUM(MRR)` on Rows, line mark
- MRR waterfall: `BILLING_MONTH` on Columns, `SUM(MRR_DELTA)` on Rows, color by `BUCKET`, bar mark (stacked or side-by-side — try both natively, side-by-side usually reads clearer for a 4-category waterfall)
- Channel performance bar: `CHANNEL` on Rows, `SUM(CUSTOMERS)` or `AVG(CHURN_RATE_PCT)` on Columns (pick churn rate — it's the more interesting sort) — **this is the worksheet the parameter action will read clicks from**

**Retention view** (1-2 worksheets)
- Retention curve: `MONTH_OFFSET` on Columns (discrete, sorted numerically), `AVG(RETAINED_PCT)` on Rows, line mark. Filtered to a single channel via the parameter action (Step 5) — with no filter yet at this stage, it'll show all channels overlaid by color, which is fine as a fallback/all-channels view before the action is wired up.

**Segmentation view** (3 worksheets)
- Plan mix: `COMPANY_SIZE_BAND` on Columns, `SUM(CUSTOMERS)` on Rows, color by `PLAN_NAME`, stacked bar (100% stacked reads best here — Format > Pane > or a quick calculated field for % of total)
- Industry churn: `INDUSTRY` on Rows (sorted desc by churn), `CHURN_RATE_PCT` on Columns, bar mark
- Feature adoption: `FEATURE_NAME` on Rows, `ADOPTION_RATE_PCT` on Columns, bar mark

Verify every one of these opens and looks right natively before moving to Step 3 — this mirrors the GA4 lesson: build and confirm each native piece before layering complexity on top.

## Step 3 — Assemble the dashboard container structure

One dashboard, sized to a fixed frame (e.g. 1200×800, same discipline as GA4). Layout:

```
┌─────────────────────────────────────────────────────────┐
│  [Meridian Analytics — Subscription Performance]         │  ← title
│  ( Revenue )  ( Retention )  ( Segmentation )             │  ← view-toggle buttons
├─────────────────────────────────────────────────────────┤
│                                                            │
│         [ Container A: Revenue panels ]                   │
│         [ Container B: Retention panels ]  (same space)   │
│         [ Container C: Segmentation panels ] (same space) │
│                                                            │
└─────────────────────────────────────────────────────────┘
```

Put the Revenue worksheets in one floating (or tiled, your call) container, the Retention worksheets in a second container occupying the *same screen region*, and the Segmentation worksheets in a third. All three containers overlap in the same space — that's what makes Dynamic Zone Visibility feel like tab-switching rather than scrolling.

## Step 4 — Wire the view-toggle buttons

Simplest native approach: three small worksheets (or a single worksheet with `View Selector`'s domain as marks) acting as clickable buttons, each running a **Parameter Action** (Dashboard menu → Actions → Add Action → Change Parameter) that sets `View Selector` to the clicked button's value on **Select**. Use Format → Borders/fill on the button worksheets so the currently-active one is visually distinct — that's a nice-to-have, not required for function.

## Step 5 — Dynamic Zone Visibility on the three containers

For each of the three containers (Revenue / Retention / Segmentation panels):
1. Select the container in the dashboard.
2. In the Layout pane, check **Control visibility using value** (this is the Dynamic Zone Visibility toggle — Tableau 2022.3+).
3. Set it to a calculated field, e.g. `[Show Revenue] = ([View Selector] = "Revenue")`, one boolean calc per container.

This is the piece worth treating exactly like GA4's dual-axis/map-conversion/new-data-source steps: **do this natively, don't hand-author it in the `.twb` XML.** Dynamic Zone Visibility's XML representation isn't one you have a working example of anywhere in this file yet (unlike GA4, where every successful hand-edit copied a pattern the file already contained) — so if a future session ever tries to script this from general knowledge instead of the Tableau UI, expect the same class of failure as the GA4 `bold`/`<title>` incident. Native UI, every time, for this step specifically.

## Step 6 — Parameter Action: channel click → retention drilldown

1. On the Revenue view's channel bar chart, this needs a second parameter, e.g. **Selected Channel** (String, default value = your best/most-interesting channel, e.g. "Paid Search" so the Retention view isn't blank before first click).
2. Dashboard menu → Actions → Add Action → **Change Parameter**. Source sheet: the channel bar chart. Target parameter: `Selected Channel`. Field: `Channel`. Run action on: **Select**.
3. On the Retention curve worksheet, add a filter: `Channel` = `[Selected Channel]` (right-click the parameter → Show Filter isn't needed here; instead add `[Selected Channel]` as a filter condition via a calculated field `[Channel] = [Selected Channel]` dragged to Filters, set to True — or a simpler direct filter if Tableau's relative-filter-by-parameter UI supports it directly in your version).
4. Test: click a channel bar. Retention view (even if not currently the active toggle) should update; switch the toggle to Retention to confirm the curve changed to match the clicked channel.

## Step 7 — Story page (optional, matches GA4's pattern)

If you want the same narrative wrap-up GA4 had: a 3-4 point Story with one point per view (Revenue, Retention, Segmentation) plus a closing "Methodology & Interactivity" point explaining the Dynamic Zone Visibility / Parameter Action pattern explicitly — this is worth naming out loud in interviews the same way the GA4 cross-filter was.

## Step 8 — Publish, grab the link, update this doc

Same close-out as GA4: publish to Tableau Public, paste the live link at the top of this file, and note anything that needed a workaround along the way (there almost always is one — GA4 hit the `derivation='User'`/`usr:` XML issue and the zone-visibility content-model failure; this build will likely surface its own version-specific quirk with Dynamic Zone Visibility or parameter-driven filters, worth capturing here the same way).

---

## Open items / decisions left to you

- Container layout is tiled vs. floating — GA4 went tiled for the main grid; floating containers are usually required for true overlapping Dynamic Zone Visibility panels, so plan on floating for the three swappable regions specifically even if the KPI strip/title stays tiled above them.
- Color palette: reuse GA4's implicit default-blue convention, or pick a distinct one for this project so the two portfolio pieces don't look identical at a glance.
- Whether `Selected Channel`'s default value should be the best or worst channel — worst (Paid Search, highest churn) is the more interesting story to land on first.
