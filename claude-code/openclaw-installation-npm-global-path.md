# openclaw / Claude Code — Installation & npm Global Path

**Date:** 2026-03-01

## What happened
Installing openclaw (the Claude Code CLI) via `npm install -g` succeeded, but running `openclaw` failed with "command not found" even though npm reported a successful global install.

## What I learned
npm's global bin directory is `~/.npm-global/bin`, **not** `/usr/local/bin`. The `~/.zshrc` already exports this path, but a new terminal tab or a fresh shell session may not have sourced the file yet.

```bash
# Check where npm puts global binaries
npm config get prefix
# → /Users/bryan/.npm-global

# If command not found after install, reload shell or run:
source ~/.zshrc

# Confirm the binary is there
ls ~/.npm-global/bin/openclaw
```

## Gotchas
- `npm install -g` prints success even if the binary lands in a directory that isn't on `$PATH` in the current shell.
- Don't use `sudo npm install -g` — it installs to `/usr/local` instead of `~/.npm-global` and causes permission chaos.
- After any global install, verify with `which openclaw` before assuming it worked.

## Fix
Add to `~/.zshrc` (already present in Bryan's setup, but good to know):
```bash
export NPM_CONFIG_PREFIX=~/.npm-global
export PATH=~/.npm-global/bin:$PATH
```
Then `source ~/.zshrc` or open a new terminal tab.
