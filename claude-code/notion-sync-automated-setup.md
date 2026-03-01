# Automating Notion Sync for New GitHub Repos

**Date:** 2026-03-01

## What I built
A repeatable, zero-manual-steps workflow to wire any new GitHub repo into the shared Notion database — no copy-pasting keys in chat, no visiting GitHub settings pages.

## How it works

### 1. Store the Notion API key locally once
```bash
read -rs NOTION_KEY && echo "NOTION_API_KEY=${NOTION_KEY}" >> ~/.claude/.env
```
`read -rs` reads silently (no echo). The key lands in `~/.claude/.env` and never appears in chat or terminal history.

### 2. Add the workflow file to the new repo
Copy `.github/workflows/notion-sync.yml` from any existing repo (e.g. `bryanfosler/pi-setup`). Change only one line — the `Project` select name — to match the new repo's Notion tag:
```javascript
'Project': {
  select: { name: 'Your Project Name Here' }
},
```

### 3. Set the GitHub Actions secret automatically
```bash
gh secret set NOTION_API_KEY \
  --repo bryanfosler/YOUR_REPO \
  --body "$(grep ^NOTION_API_KEY ~/.claude/.env | cut -d= -f2-)"
```
This reads the key from the local file and sets it as a repo secret in one shot — no browser, no copy-paste.

### 4. Verify end-to-end
```bash
# Create a test issue
gh issue create --repo bryanfosler/YOUR_REPO \
  --title "Test: Notion sync" \
  --body "Verifying sync."

# Wait ~10s, check the run succeeded
gh run list --repo bryanfosler/YOUR_REPO --limit 3

# Add a time comment and confirm update
gh issue comment 1 --repo bryanfosler/YOUR_REPO --body "Time: 5m"
gh run view <RUN_ID> --repo bryanfosler/YOUR_REPO --log | grep "Notion page"

# Close the test issue
gh issue close 1 --repo bryanfosler/YOUR_REPO --comment "Sync confirmed."
```

## How the sync workflow works
- Triggers on: issue opened/edited/closed/labeled + comments containing `Time:`
- Searches Notion for an existing page matching the issue's GitHub URL
- Creates the page if new, patches it if existing
- Sums all `Time: Xm` / `Time: 1h30m` comments → writes total to "Time Spent (min)"
- Hard-coded database ID: `30a95da6-c9dc-805a-8dcc-cf1dc2694ea2`

## Gotchas
- `cut -d= -f2-` (not `f2`) — Notion keys contain `=` padding; `f2` truncates the key
- `read -rs` requires being run directly in your shell — won't work inside a script or subshell
- The Notion `Project` field is a Select — the value must either already exist as an option or Notion will auto-create it (it does, silently)
- GitHub secrets set via `gh secret set --body` take effect immediately on the next workflow run
