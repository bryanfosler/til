# Abandoned CLI installs can leave API keys on disk

When you uninstall a CLI tool via `npm uninstall -g`, the binary is removed — but the tool's config directory (usually `~/.toolname/`) is often left behind. If that config contains API keys, those keys are sitting on disk indefinitely.

## What happened

`openclaw` was installed on the Mac, ran through its onboarding wizard (which stored an Anthropic API key and a Notion API key in config files), and was then uninstalled. The `~/.openclaw/` directory remained with live keys in:

- `~/.openclaw/agents/main/agent/auth-profiles.json` — Anthropic API key
- `~/.openclaw/openclaw.json` — Notion API key

The only symptom was a `compdef` error in the terminal — easily missed or ignored.

## Lesson

After uninstalling a CLI tool that required authentication:

1. Check for a leftover config directory (`~/.toolname/`)
2. Look for key files: `auth*.json`, `config.json`, `*.json` with `key`, `token`, or `secret` fields
3. Rotate any exposed keys before deleting the directory
4. Delete the directory

## Detection

```bash
# Find config dirs for tools you no longer have installed
ls ~/.*  # look for .toolname folders
# Then check for credentials
grep -r "sk-ant\|api_key\|token\|secret" ~/.toolname/ 2>/dev/null
```
