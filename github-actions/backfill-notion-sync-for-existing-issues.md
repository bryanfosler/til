# Backfilling Notion Sync for Pre-Existing Repo Content

**Date:** 2026-03-01

## What happened
After adding the Notion sync workflow to the TIL repo, existing entries (committed before sync existed) didn't appear in Notion — because the workflow only fires on issue events, not commits.

## What I learned
Notion sync is issue-driven, not commit-driven. To get pre-existing work into Notion you have to create a GitHub issue for it, add a time comment, and close it. The workflow fires on each event and builds the Notion row.

## The backfill pattern

```bash
# 1. Create the issue
gh issue create --repo bryanfosler/REPO \
  --title "TIL: <entry title>" \
  --body "Backfill issue for TIL entry added YYYY-MM-DD.

Entry: <link to file on GitHub>

<one-line description of what was learned>"

# 2. Log time (if you remember roughly how long it took)
gh issue comment ISSUE_NUMBER --repo bryanfosler/REPO --body "Time: 45m"

# 3. Close it — triggers a final sync that sets Status = Done
gh issue close ISSUE_NUMBER --repo bryanfosler/REPO \
  --comment "Backfilled. Work done YYYY-MM-DD."
```

## Correcting a time entry after the fact
If you logged the wrong time, just add another `Time:` comment — the workflow sums **all** `Time:` comments on the issue each time it runs:

```bash
# Already logged Time: 45m but it was actually 1h — add the difference
gh issue comment ISSUE_NUMBER --repo bryanfosler/REPO --body "Time: 15m"
# Notion will now show 60 min total
```

## Gotcha
You can add comments to closed issues via `gh` — the workflow still fires and updates Notion. So you can correct time entries even after closing.
