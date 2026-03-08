# Multi-line Command Paste Issues in Warp, VS Code, Terminal

**Date:** 2026-03-08

When a long shell command is visually wrapped across multiple lines in chat, copying and pasting it often inserts literal newlines — causing bash to treat each line as a separate command. This breaks commands with flags split across lines (e.g., `-pass` on one line, `file:/etc/machine-id` on the next).

## Fix: Use Backslash Continuation

Write multi-line commands with `\` at the end of each continued line:

```bash
printf '%s' "$T" | openssl enc -aes-256-cbc -pbkdf2 \
  -pass "file:/etc/machine-id" \
  -out ~/.config/systemd/user/credstore/token.enc
```

`\` tells bash "this line continues" — safe to copy-paste as a block in any terminal.

## Or: Keep It on One Line

For commands you know will be copy-pasted, a single long line is the safest option — no wrapping, no ambiguity.

## Workaround When Neither Is Available

Run it as two steps: the flags that must stay together on one line, then the rest.
