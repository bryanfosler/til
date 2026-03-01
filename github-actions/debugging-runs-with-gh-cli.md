# Debugging GitHub Actions Runs with the gh CLI

**Date:** 2026-03-01

## What I learned
You don't need to open the GitHub website to debug Actions runs. The `gh` CLI gives you run status and full logs right in the terminal.

## Key commands

### See recent runs for a workflow
```bash
gh run list --repo bryanfosler/REPO --workflow=notion-sync.yml --limit 5
```
Output columns: status, conclusion, title, workflow, branch, event, run ID, duration, timestamp.

### Read the full logs for a specific run
```bash
gh run view RUN_ID --repo bryanfosler/REPO --log
```
Prints every line of every step. Pipe through `grep` to find what you care about:
```bash
gh run view RUN_ID --repo bryanfosler/REPO --log | grep -E "Created Notion|Updated Notion|error"
```

### Watch a run in progress
```bash
gh run watch RUN_ID --repo bryanfosler/REPO
```

## Understanding `skipped` vs `success` vs `failure`

| Conclusion | Meaning |
|------------|---------|
| `success` | Ran and completed without error |
| `skipped` | The workflow's `if:` condition evaluated to false — job was intentionally not run |
| `failure` | Ran but exited with an error |

**`skipped` is not a problem.** For the Notion sync workflow, issue_comment events are skipped when the comment body doesn't contain `Time:` — that's the `if:` condition working correctly, not a bug.

## Practical workflow for verifying a new sync setup

```bash
# 1. Create a test issue
gh issue create --repo bryanfosler/REPO --title "Test: Notion sync" --body "Verify sync."

# 2. Wait ~10s, then check the run
gh run list --repo bryanfosler/REPO --workflow=notion-sync.yml --limit 1

# 3. Confirm the log says "Created Notion page"
gh run view $(gh run list --repo bryanfosler/REPO --limit 1 --json databaseId -q '.[0].databaseId') \
  --repo bryanfosler/REPO --log | grep "Notion page"

# 4. Add a time comment and confirm "Updated Notion page"
gh issue comment 1 --repo bryanfosler/REPO --body "Time: 5m"
sleep 12
gh run list --repo bryanfosler/REPO --workflow=notion-sync.yml --limit 1
```

## Gotcha
`gh run list` without `--workflow=` shows runs from ALL workflows in the repo. Filter by workflow name to avoid noise.
