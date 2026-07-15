# Week 4 — Insights Dashboard + Report

## Files
- **`Week4_IPL_Dashboard.xlsx`** — the dashboard. Open the `Dashboard` sheet for KPIs and 7 charts, all driven by live formulas (COUNTIF/COUNTIFS/AVERAGEIFS/SUMPRODUCT) against the `Matches` sheet — nothing is hardcoded, so the whole thing recalculates if the underlying data changes.
- **`Week4_Insights_Report.docx`** — 2-page report: 3 findings, each with a supporting chart and a specific action.
- **`build_dashboard.py`** / `prep_dashboard_data.py` — regenerate the Excel file from source data.
- **`build_report.js`** — regenerate the report (`node build_report.js`, requires `npm install docx`).
- **`matches_for_dashboard.csv`** — the exact dataset the dashboard was built from.

## Workbook structure
| Sheet | Contents |
|---|---|
| Dashboard | KPI cards + 7 charts |
| Matches | 1,212-row fact table (Excel Table, filterable) |
| Team_Stats | Wins / matches played / win rate per team (formulas) |
| Season_Stats | Matches, toss-decision %, avg margin per season (formulas) |
| Venue_Stats | Top 10 venues by matches hosted (formulas, cleaned venue names) |
| Player_Stats | Top 10 Player-of-the-Match award winners (formulas) |
| HeadToHead | Top 10 most frequent team matchups (formulas) |

## Two data-quality fixes applied here (carried through from Weeks 1–3)
1. **Phantom winners on "no result" matches** (found in Week 3): a match with no result can't have a winner — nulled out before loading.
2. **Inconsistent venue names** (found while building this dashboard): the same physical ground was split across multiple text variants (e.g. "Wankhede Stadium" vs. "Wankhede Stadium, Mumbai"). Normalizing this **changes the answer** to "which venue has hosted the most matches" — from Eden Gardens (77, wrong) to Wankhede Stadium (130, correct). See the report for the full explanation and the one intentional exception (Feroz Shah Kotla / Arun Jaitley Stadium, a real 2019 rename, kept separate since merging it needs outside knowledge, not just string matching).

## The 3 recommendations (see report for full detail)
1. Stop treating "won the toss" as a meaningful edge in win-probability models — it's a 51.6% coin flip, even though 80%+ of the league now fields first by habit.
2. Run all venue-level analysis on the cleaned `venue_clean` field, and standardize venue names as a permanent pipeline step, not a one-off fix.
3. Use win rate, not cumulative win count, as the headline team-performance metric — it flags Delhi Capitals, Punjab Kings, and Sunrisers Hyderabad (270+ matches, sub-50% win rate) for real strategic review, which raw win totals hide.
