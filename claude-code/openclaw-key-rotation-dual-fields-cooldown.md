# openclaw API Key Rotation — Dual Fields, Cooldown Reset, and Safe Update Script

**Date:** 2026-03-01

## What happened

The Piper Telegram bot (openclaw on Pi) kept returning 401 errors even after updating the Anthropic API key. Keys were rotated 3× in one day. The key would work briefly, then break again ~1.5 hours later.

## What I learned

### 1. auth-profiles.json has TWO key fields — both must be updated

`~/.openclaw/agents/main/agent/auth-profiles.json` stores the Anthropic key in **two separate fields**:

```json
"anthropic:manual": {
  "type": "token",
  "provider": "anthropic",
  "token": "sk-ant-api03-...",
  "key":   "sk-ant-api03-..."
}
```

If you only update one (e.g. `key` but not `token`), openclaw may try both and fail on the stale one. Always update **both** to the same new key.

### 2. usageStats cooldown blocks valid keys after auth failures

When openclaw hits a 401, it writes failure stats back to `auth-profiles.json`:

```json
"usageStats": {
  "anthropic:manual": {
    "errorCount": 1,
    "lastFailureAt": 1772404359373,
    "cooldownUntil": 1772404419373
  }
}
```

The cooldown persists across service restarts. A valid new key can still be blocked until the cooldown expires (~60s). **Always clear usageStats when rotating a key.**

### 3. openclaw.json env block ≠ openclaw's own auth

Adding `ANTHROPIC_API_KEY` to the `env` block in `openclaw.json` passes it to agent subprocesses — but openclaw's own auth layer reads from `auth-profiles.json`, not the env block. The env block alone won't fix a 401.

### 4. Key rotation order matters

Once you create a new key on the Anthropic console, the old key is immediately invalid. Don't rotate on the console until you're ready to push the new key to the Pi right away. "I'll update the Pi in a bit" = bot broken in the meantime.

### 5. zsh read syntax differs from bash

The `read -rsp "prompt"` flag is bash-only. In zsh:

```zsh
read -rs 'K?Key: ' && print
```

## Fix: update script on the Pi

`~/update-openclaw-key.sh` handles everything — updates both key fields, clears the cooldown, restarts the service:

```bash
# On Mac terminal — enter key securely (not echoed, not in shell history)
umask 077 && read -rs 'K?Key: ' && print && print -rn -- "$K" > /tmp/.ak

# Pipe to Pi — updates auth-profiles.json, openclaw.json env block, restarts service
ssh bfosler@bryanfoslerpi5.local "bash ~/update-openclaw-key.sh" < /tmp/.ak && rm /tmp/.ak
```

Never paste API keys directly into Claude Code chat — messages are sent to Anthropic's servers and stored in local history files.

## Debugging tip

Check logs before rotating on the Anthropic console:

```bash
ssh bfosler@bryanfoslerpi5.local "npx openclaw logs --plain --limit 20"
```

Look for `HTTP 401 authentication_error: invalid x-api-key` to confirm it's actually a key issue vs. something else.
