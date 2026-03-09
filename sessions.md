# TIL — Sessions

---

## Session 6 — Docker UFW Bypass Fix + Obsidian Vault Planning

**Date:** 03.08.2026
**Time spent:** ~35m

### What We Built
- Updated `raspberry-pi/docker-bypasses-ufw.md` TIL with real implementation details (catch-all rule ordering, iptables-persistent, per-port targeting)
- Created TIL backlog issue #23: Learn Obsidian Tasks + Dataview plugins

### What Shipped
- Docker UFW bypass fixed on Pi (DOCKER-USER chain, port 3000 Tailscale-only, persisted via iptables-persistent)
- 7am Telegram reminder scheduled via Pi cron for Obsidian vault migration

### Bugs Fixed
- None (continuation session from previous)

### Decisions Made
- Open-WebUI (port 3000): Tailscale-only access (not LAN) — zero friction since Mac always has Tailscale running
- Obsidian: one vault (expand existing ObsidianVault/) rather than separate work/personal/AI vaults — cross-linking and unified search outweigh any organizational benefit of splitting
- iCloud sync for iPhone Obsidian access (Syncthing doesn't support iOS)

---

## Session 12 — Claude Code allow list optimization

**Date:** 03.01.2026
**Time spent:** ~10m

### What We Built
- Analyzed 5 sessions of Bash command history to measure auto-approval vs manual approval rates
- Found 55% of commands needed manual approval; `ssh` (19x), `cd` (15x), `find` (7x) were the top three

### What Shipped
- `~/.claude/settings.json` updated: added `find`, `cd`, `tail`, `grep` to allow list
- Approval rate should drop from ~55% to ~22% (ssh and rm remain intentionally manual)

### Bugs Fixed
- None

### Decisions Made
- `ssh` stays manual — Pi access should always require a conscious approval
- `rm` stays manual — too destructive to auto-approve
- `cd` approved despite being a compound-command prefix (e.g. `cd /path && python3 ...`) — dangerous parts like `rm -rf` are already in deny list

---

## Session 11 — Piper bot 401 debugging + openclaw key rotation fix

**Date:** 03.01.2026
**Time spent:** ~50m

### What We Built
- Diagnosed recurring Anthropic API key 401 errors on Piper (openclaw on Pi)
- Root cause: `auth-profiles.json` has TWO key fields (`token` + `key`) — previous rotations only updated one
- Secondary cause: `usageStats` cooldown in auth-profiles.json persists across restarts and blocks valid keys
- Created `~/update-openclaw-key.sh` on the Pi — updates both fields, clears cooldown, restarts service

### What Shipped
- `~/update-openclaw-key.sh` on Pi — one-script key rotation
- TIL entry: `claude-code/openclaw-key-rotation-dual-fields-cooldown.md`
- `openclaw.json` env block updated with `ANTHROPIC_API_KEY` (belt + suspenders)

### Bugs Fixed
- Piper bot returning 401 on all messages due to stale/revoked keys in auth-profiles.json

### Decisions Made
- Secure key rotation workflow: `read -rs 'K?Key: '` (zsh) → `/tmp/.ak` → pipe to Pi via SSH (scp is in deny list)
- Never paste API keys in Claude Code chat — messages sent to Anthropic servers + stored in history files
- `read -rsp` is bash-only; zsh equivalent is `read -rs 'K?Key: ' && print`

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

---

## Session 4 — Discord Guild Message Fix (Post-Migration)

**Date:** 03.08.2026
**Time spent:** ~45m

### What We Built
- Diagnostic tooling: raw Discord WebSocket gateway test script to isolate whether events arrive at all

### What Shipped
- `channels.discord.groupPolicy` changed from `allowlist` → `open` in `openclaw-template.json`
- Piper bot added as Member to `#general` with View Channel, Send Messages, Read Message History, Attach Files, Embed Links, Add Reactions

### Bugs Fixed
- Discord guild messages silently dropped after secrets migration — root cause was bot not having View Channel permission in #general

### Decisions Made
- When Discord bot messages fail silently, skip straight to raw WebSocket gateway test to determine if Discord is even delivering events
- If no MESSAGE_CREATE events in 30s raw test window, it's a Discord permissions/channel issue — not openclaw config
- `groupPolicy: "open"` is safe for PiperPi5 since the server is private (Bryan only); dmPolicy still restricts DMs

---

## Session 5 — Fix claude_usage.py Week Reset Day

**Date:** 03.08.2026
**Time spent:** ~10m

### What We Built
- Nothing new

### What Shipped
- `claude_usage.py`: week start now uses Thursday (weekday 3) instead of Monday
- Bar label updated to `week (Thu↺)` for clarity
- Day counter now 1=Thu through 7=Wed

### Bugs Fixed
- Week bar showed "day 7/7, 3% left" on Sundays — was using Mon–Sun calendar week, not Claude's Thu–Wed reset window

### Decisions Made
- Claude Code Pro weekly limit resets Thursday per claude.ai
- `/usage` slash command is UI-only, no programmatic output — script reading local JSONL is the right approach
