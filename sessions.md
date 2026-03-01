# TIL — Sessions

---

## Session 7 — Add notion/ TIL entry for Time (hrs) formula

**Date:** 03.01.2026
**Time spent:** ~10m

### What We Built
- New `notion/` topic folder with first entry

### What Shipped
- `notion/time-hrs-formula-property.md` — covers formula syntax, API call, view usage, and formula-is-read-only gotcha

### Bugs Fixed
- None

### Decisions Made
- None

---

## Session 6 — Notion views + Time (hrs) formula

**Date:** 03.01.2026
**Time spent:** ~10m

### What We Built
- Logged TIL issue #7 for Notion grouped table view setup
- Added `Time (hrs)` formula property to Notion database via API

### What Shipped
- Formula: `round(prop("Time Spent (min)") / 60 * 10) / 10` — displays hours as decimals in all views

### Bugs Fixed
- None

### Decisions Made
- Keep raw `Time Spent (min)` for workflow summing; formula column is display-only

---

## Session 5 — Claude Code permissions settings

**Date:** 03.01.2026
**Time spent:** ~20m

### What We Built
- TIL entry for Claude Code `settings.json` permissions (allow/deny rules)

### What Shipped
- `claude-code/settings-permissions-allow-deny.md`
- `~/.claude/settings.json` updated with allow + deny rules

### Bugs Fixed
- None

### Decisions Made
- Deny rules prioritized around secrets and SSH given Pi setup with key auth
- `ssh:*` deny accepted knowing it means manual approval for Pi commands

---

## Session 1 — Initial TIL entries + Notion sync setup

**Date:** 03.01.2026
**Time spent:** ~45m

### What We Built
- New `claude-code/` topic folder with 5 TIL entries from the morning's openclaw/Notion session
- `.github/workflows/notion-sync.yml` wired to the shared Notion database (Project: TIL)

### What Shipped
- 5 TIL entries: openclaw installation, Anthropic API key 401, Telegram pairing flow, Notion sync for TIL repo, automated Notion sync setup
- Notion sync live and verified end-to-end (created → time logged → closed → confirmed in Action logs)

### Bugs Fixed
- Merge conflict on README.md during push (remote had a new iOS entry; resolved by keeping both)

### Decisions Made
- Store `NOTION_API_KEY` in `~/.claude/.env` for automated `gh secret set` on future repos — no manual key handling needed
- New repo checklist added to global CLAUDE.md so Notion sync question is always asked

---

## Session 2 — Backfill pre-existing TIL entries into Notion

**Date:** 03.01.2026
**Time spent:** ~10m

### What We Built
- Backfill issues for the two TIL entries that existed before Notion sync was set up

### What Shipped
- Issue #3: iOS device install (45m logged) — closed, synced to Notion
- Issue #4: Tailscale + Termius (60m logged) — closed, synced to Notion

### Bugs Fixed
- None

### Decisions Made
- Backfill pattern: create issue with "Backfill issue for TIL entry added YYYY-MM-DD" body, add Time: comment(s), close

---

## Session 4 — Add Week/Month fields for Notion reporting

**Date:** 03.01.2026
**Time spent:** ~30m

### What We Built
- Week and Month select properties added to Notion database schema via API
- All 5 notion-sync.yml workflows updated to populate Week and Month on every sync
- Backfill script to patch all 44 existing Notion records

### What Shipped
- 44 existing records backfilled with correct Week/Month values
- All 5 repos updated and pushed
- Notion view setup instructions for Weekly Overview, Monthly Overview, By Project, Board

### Bugs Fixed
- pi-setup had a pre-existing merge conflict in petcam/petcam.py — staged only the workflow file and left the conflict untouched
- `node -e` can't handle `!` in inline scripts due to shell history expansion — workaround: write script to /tmp file

### Decisions Made
- Used ISO week format (2026-W09) so groups sort correctly by default in Notion
- Used select (not formula) for Week/Month — easier to group and aggregate in Notion views

---

## Session 3 — Add github-actions TIL entries

**Date:** 03.01.2026
**Time spent:** ~15m

### What We Built
- New `github-actions/` topic with 2 entries

### What Shipped
- Backfilling Notion sync for pre-existing repo content
- Debugging GitHub Actions runs with the gh CLI (run list, run view --log, skipped vs success vs failure)

### Bugs Fixed
- None

### Decisions Made
- Gap analysis should happen proactively before wrap-up, not only when Bryan asks
