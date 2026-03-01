# Adding Notion Sync to the TIL Repo

**Date:** 2026-03-01

## What happened
TIL entries weren't appearing in Notion even though other repos (midi-control, pi-setup) sync correctly. The TIL repo was missing the sync workflow entirely.

## What I learned

### Why it wasn't working
The `notion-sync.yml` GitHub Actions workflow is **per-repo** — it doesn't automatically apply to every repo just because you have it elsewhere. The `bryanfosler/til` repo had no `.github/workflows/` folder at all.

### How the sync works (in repos that have it)
1. A GitHub Actions workflow fires on issue events (opened, edited, closed, commented).
2. It reads `Time: Xm` comments and sums them → writes to Notion's "Time Spent (min)" column.
3. It tags the row with a hard-coded **Project** field (e.g., "Pi Setup") so you can filter in Notion by project.

### Steps to add sync to TIL repo

1. Copy `.github/workflows/notion-sync.yml` from a working repo (e.g., `bryanfosler/pi-setup`).
2. Change the `Project` tag in that file from `Pi Setup` → `TIL`.
3. Add the same GitHub Actions secrets to `bryanfosler/til`:
   - `NOTION_API_KEY`
   - `NOTION_DATABASE_ID`
   - (any others the workflow uses — check the working repo's workflow for all `secrets.*` references)
4. Create a test issue, add a `Time: 5m` comment, and verify the Notion row appears.

### Labels for monthly rollups
Use a `2026-03` label on TIL issues to make monthly filtering trivial in both GitHub and Notion.

## Gotcha
The Notion database must have a "Project" column that accepts the exact string value you hard-code in the workflow. If it's a Select field, the value must already exist as an option or Notion will reject it silently.
