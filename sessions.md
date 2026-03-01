# TIL — Sessions

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
