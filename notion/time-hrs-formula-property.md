# Adding a Human-Readable Hours Formula to a Notion Number Property

**Date:** 2026-03-01

## What happened
The Notion database stores time as raw minutes (e.g. `90`) because the GitHub Actions workflow sums `Time:` comments and writes an integer. That's great for math, but hard to read at a glance in grouped views.

## What I learned
Notion formula properties can derive a display value from any other property — including number fields — without touching the underlying data. Adding a `Time (hrs)` formula alongside the raw `Time Spent (min)` field gives you both: accurate summing in the workflow and human-readable hours in the UI.

## The formula
```
round(prop("Time Spent (min)") / 60 * 10) / 10
```
Result: `90 min → 1.5 hrs`, `45 min → 0.8 hrs`, `60 min → 1 hrs`, `30 min → 0.5 hrs`

The `* 10 / 10` trick rounds to one decimal place (Notion doesn't have a `round(x, decimals)` overload).

## Adding it via the Notion API
```bash
NOTION_KEY=$(grep ^NOTION_API_KEY ~/.claude/.env | cut -d= -f2-)
DB_ID="your-database-id"

curl -s -X PATCH "https://api.notion.com/v1/databases/${DB_ID}" \
  -H "Authorization: Bearer ${NOTION_KEY}" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "Time (hrs)": {
        "formula": {
          "expression": "round(prop(\"Time Spent (min)\") / 60 * 10) / 10"
        }
      }
    }
  }'
```

## Using it in views
- Add `Time (hrs)` to any table view via **Properties**
- Set **Calculate → Sum** on the column to show total hours per group
- Works in Weekly Overview, Monthly Overview, and By Project grouped views

## Gotcha
Formula properties are read-only — you can't write to them from the API or workflow. The workflow should always write to the raw `Time Spent (min)` field; the formula updates automatically.
