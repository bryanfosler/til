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
