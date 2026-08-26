# Meridian Subscription Dashboard

Second portfolio project — an interactive Tableau Public dashboard, kept as a
separate project from [`subscription-analytics-sql`](../subscription-analytics-sql)
by design, though it shares that project's synthetic "Meridian Analytics"
SaaS dataset.

Where the SQL repo is the query-proficiency showcase, this one is the
visualization/interactivity showcase — going one step past the GA4
dashboard's single cross-filter action into **Dynamic Zone Visibility**
(a view-toggle that swaps whole chart panels in place) and a **Parameter
Action** (clicking a channel drives a drilldown chart elsewhere on the
dashboard).

**Status:** data prepped, full build plan written. Tableau build itself is a
native, hands-on step (see why in the build plan) — not something to
hand-author blind the way GA4's initial worksheets were.

## What's here

- `data/` — 7 pre-aggregated mart CSVs, ready for Tableau Public's Text File
  import (Tableau Public has no live DB connection regardless of source, so
  this is the same CSV-export pattern GA4 used, not a step down from it).
- `docs/dashboard_build_plan.md` — the full build plan: worksheet-by-worksheet
  shelf structure, the two parameters needed (`View Selector`,
  `Selected Channel`), the Dynamic Zone Visibility setup, and the parameter
  action wiring, in the same level of detail as the GA4 project log.

## Next step

Work through `docs/dashboard_build_plan.md` in Tableau Public (or Desktop),
natively — the plan is written to be followed step by step, the same way
the GA4 project's log tracked phase-by-phase progress. Once published,
paste the live Tableau Public link at the top of this README.
