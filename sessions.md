# TIL — Sessions

---

## Session 10 — Token usage tracking in GitHub Issues → Notion

**Date:** 03.01.2026
**Time spent:** ~45m

### What We Built
- `~/utils/python/session_tokens.py` — reads current session JSONL, sums all token types, prints `Tokens: XXXXXXX  Cost: XX.XX`
- Two new Notion DB properties: `Tokens` (number) and `API Equiv ($)` (number)
- Updated all 5 `notion-sync.yml` workflows to parse and write token/cost data
- Updated wrap-up skill to run `session_tokens.py` and post combined time+tokens comment
- Updated `~/Documents/Claude/CLAUDE.md` with new session time-logging format

### What Shipped
- `bryanfosler/utils` — `python/session_tokens.py` (new file)
- `bryanfosler/til`, `midi-control`, `pi-setup`, `run-route-generator`, `reddit-research-tool` — `notion-sync.yml` updated in all 5
- Notion DB patched with two new number properties
- End-to-end verified: posted test comment, confirmed `Created Notion page for issue #13 (5 min, 1000 tokens, $0.01)` in workflow logs

### Bugs Fixed
- None

### Decisions Made
- Token values are raw integers in comments (no M suffix) — keeps regex simple
- Null (not 0) written to Notion when no token data present — avoids cluttering old issues with zeroes
- One combined comment per session (`Time: Xm  Tokens: X  Cost: X`) triggers one workflow run

---

## Session 9 — Mac cleanup: zsh fix + API key rotation

**Date:** 03.01.2026
**Time spent:** ~30m

### What We Built
- Diagnosed and fixed `compdef: command not found` error on terminal open
- Found `~/.openclaw/` left behind on Mac with live API keys
- Rotated Anthropic + Notion API keys; deleted `~/.openclaw/`
- Verified openclaw/Piper still working on Pi after key rotation

### What Shipped
- `~/.zshrc` cleaned up (compinit added, then openclaw source line removed)
- 2 new TIL entries: zsh compdef fix, abandoned CLI key exposure
- New `zsh/` and `security/` topic folders in TIL repo

### Bugs Fixed
- `compdef: command not found` on every terminal open (missing compinit before completion source)

### Decisions Made
- `rm -rf` stays in deny list — user runs destructive commands manually

---

## Session 8 — Claude Code voice hooks + queue-based TTS

**Date:** 03.01.2026
**Time spent:** ~50m

### What We Built
- `Stop` hook that reads the last Claude response aloud via macOS `say`, stripping code blocks
- Queue + daemon system (`tts_enqueue.py` + `tts_daemon.py`) so multiple hooks don't talk over each other
- `CLAUDE_MUTE=1` env var guard across all hooks for session-based silence
- `claude-talk` / `claude-quiet` aliases in `~/.zshrc`
- Switched from PingVoice API to native macOS `say` — no API key, no browser required

### What Shipped
- `~/.claude/hooks/stop.py` — reads transcript, strips code, speaks response
- `~/.claude/hooks/utils/tts_enqueue.py` + `tts_daemon.py` — serialized audio queue
- All four hooks (stop, session_start, notification, subagent_stop) updated
- `~/.claude/settings.json` wired with Stop hook
- GitHub issue: bryanfosler/utils#3 (closed), future ideas: #2

### Bugs Fixed
- Multiple `say` processes spawning concurrently (fixed by queue daemon)

### Decisions Made
- Skipping code blocks in spoken output is the right call — hearing code read aloud is useless
- Bryan unsure about always-on voice; next session will revisit and decide direction
- Future ideas logged: pause/resume, speed control, ding + manual trigger, popup HUD

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
