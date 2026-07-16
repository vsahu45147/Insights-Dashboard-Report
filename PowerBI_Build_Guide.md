# Power BI Dashboard — Build Guide
### IPL Match Analytics (2008–2026)

This gives you everything to build the Week 4 dashboard natively in Power BI Desktop
(free) in about 20–30 minutes. The data is already fully cleaned — both data-quality
fixes from Weeks 1–3 are baked in (no-result "phantom winners" removed, venue names
standardized).

## Files
- **`IPL_Matches_PowerBI.csv`** — 1,212 rows, the main fact table
- **`Teams_PowerBI.csv`** — team lookup table
- **`Players_PowerBI.csv`** — player lookup table

---

## Step 1 — Import the data
1. Open **Power BI Desktop** → **Get Data → Text/CSV**
2. Load all three CSVs: `IPL_Matches_PowerBI.csv`, `Teams_PowerBI.csv`, `Players_PowerBI.csv`
3. In Power Query Editor, confirm data types:
   - `Match Date` → Date
   - `Win By Runs`, `Win By Wickets`, `POTM Player ID` → Whole Number (these have blanks — that's correct, see the README on why)
   - Everything else → Text
4. Click **Close & Apply**

## Step 2 — Build relationships
Power BI auto-detects some relationships, but set these manually in the **Model** view
if needed (all as **many-to-one**, single direction):
- `IPL_Matches_PowerBI[Player of the Match]` → not needed as a relationship (name is
  already a text column pulled in during cleaning) — skip this one, it's just a column.
- Optional: if you want a proper star schema instead of the flat file, relate
  `Teams_PowerBI[team_name]` to each of `Team 1`, `Team 2`, `Toss Winner`, `Match Winner`
  individually (Power BI only allows one *active* relationship between two tables, so
  make 3 of the 4 **inactive** and use `USERELATIONSHIP()` in DAX if you go this route).
  **Simpler option** (recommended): skip relationships entirely — the matches table
  already has readable team/player names in every column, so you can build every visual
  directly off the one flat table.

## Step 3 — Add the DAX measures
Go to the `IPL_Matches_PowerBI` table → **New Measure**, and add each of these:

```dax
Total Matches = COUNTROWS(IPL_Matches_PowerBI)
```

```dax
Total Teams = DISTINCTCOUNT(IPL_Matches_PowerBI[Team 1])
```

```dax
Total Seasons = DISTINCTCOUNT(IPL_Matches_PowerBI[Season])
```

```dax
Toss & Match Winner Same =
CALCULATE(
    COUNTROWS(IPL_Matches_PowerBI),
    IPL_Matches_PowerBI[Toss Winner] = IPL_Matches_PowerBI[Match Winner],
    IPL_Matches_PowerBI[Result] = "win"
)
```

```dax
Completed Matches = CALCULATE(COUNTROWS(IPL_Matches_PowerBI), IPL_Matches_PowerBI[Result] = "win")
```

```dax
Toss Win Rate % = DIVIDE([Toss & Match Winner Same], [Completed Matches], 0)
```
Format this measure as a **Percentage** (1 decimal place) via the Measure Tools ribbon.

```dax
Avg Win Margin (Runs) = AVERAGE(IPL_Matches_PowerBI[Win By Runs])
```

```dax
% Chose to Field =
DIVIDE(
    CALCULATE(COUNTROWS(IPL_Matches_PowerBI), IPL_Matches_PowerBI[Toss Decision] = "field"),
    COUNTROWS(IPL_Matches_PowerBI),
    0
)
```
Format as Percentage.

**Note on "wins by team":** for a simple bar chart of total wins, you don't need a
measure at all — just drag `Match Winner` to the axis and use the default `Count of
Match Winner` as the value. You only need DAX measures for **win rate** (wins ÷ matches
played), because "matches played" requires counting a team whether it appears in
`Team 1` or `Team 2`. That's what Step 4 below sets up cleanly.

## Step 4 — (Recommended) Build a "Team Appearances" table for clean win-rate charts
DAX measures for "matches played by a team" get awkward on a flat table where each row
has two team columns. The clean fix — do this once in **Power Query**, not DAX:

1. In Power Query, duplicate the `IPL_Matches_PowerBI` query, name the copy `Team Appearances`
2. Select `Team 1` and `Team 2` columns → **Unpivot Columns** (right-click → Unpivot Columns)
3. Rename the resulting `Attribute` column to `Team Role` and `Value` column to `Team`
4. Now every match produces 2 rows (one per team) — this makes "matches played per team"
   a plain `COUNTROWS` grouped by `Team`, and "wins per team" is
   `CALCULATE(COUNTROWS(...), [Team] = [Match Winner])`
5. Close & Apply

With this table, add:
```dax
Team Matches Played = COUNTROWS('Team Appearances')
```
```dax
Team Wins = CALCULATE(COUNTROWS('Team Appearances'), 'Team Appearances'[Team] = 'Team Appearances'[Match Winner])
```
```dax
Team Win Rate % = DIVIDE([Team Wins], [Team Matches Played], 0)
```

## Step 5 — Build the report page
Add a new page, name it **"IPL Dashboard"**, and add these visuals:

| Visual | Type | Fields |
|---|---|---|
| KPI cards | Card (x4) | `[Total Matches]`, `[Total Teams]`, `[Total Seasons]`, `[Toss Win Rate %]` |
| Matches per season | Clustered column chart | Axis: `Season` (sort by `Season ID`) · Values: `[Total Matches]` |
| Wins by team | Bar chart | Axis: `Match Winner` · Values: Count of rows |
| Toss outcome | Pie/Donut chart | Legend: a calculated column `"Toss winner won"` vs `"Toss winner lost"` (or just use `[Toss Win Rate %]` as a Gauge visual instead — simpler) |
| Toss decision trend | Line chart | Axis: `Season` · Values: `[% Chose to Field]` |
| Top venues | Bar chart | Axis: `Venue` · Values: Count of rows · **Filter: Top N = 10** |
| Top players | Bar chart | Axis: `Player of the Match` · Values: Count of rows · **Filter: Top N = 10** |
| Team win rate | Bar chart (from `Team Appearances` table) | Axis: `Team` · Values: `[Team Win Rate %]` · sort descending |

Add **slicers** for `Season` and `Team 1`/`Team` at the top of the page so the whole
dashboard is interactive — this is the one thing Excel can't do as smoothly, and it's
the main reason to use Power BI over the Week 4 Excel version.

## Step 6 — Save
**File → Save As → `IPL_Dashboard.pbix`**. That file is now your native Power BI
dashboard, ready to submit or publish to the Power BI Service if you have an account.

---

## Data-quality notes carried into this file
- **No-result matches**: 9 rain-abandoned matches have a blank `Match Winner` — this is
  intentional (see Week 3 finding), don't fill it in.
- **Venue names**: already standardized in the `Venue` column (41 real grounds instead
  of 59 raw text variants). Use `Venue`, not `Venue (Original)`, for any venue-level
  visual.
- **`Win By Runs` / `Win By Wickets`**: blank values are structurally correct (a team
  can only win by one or the other) — don't fill with 0, and use `AVERAGE()` (which
  ignores blanks automatically), not a manual sum/count.
