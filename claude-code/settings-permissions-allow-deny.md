# Claude Code settings.json — permissions allow/deny

Claude Code's `~/.claude/settings.json` supports a `permissions` block that
pre-approves or permanently blocks specific tool calls, saving you from
approving the same safe commands repeatedly.

## Structure

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)"
    ],
    "deny": [
      "Bash(rm -rf*)"
    ]
  }
}
```

- **allow** — runs automatically without prompting
- **deny** — blocked even if you try to manually approve

## Pattern syntax

- Exact match: `"Bash(npm run lint)"` — only that exact command
- Wildcard suffix: `"Bash(npm run test:*)"` — matches any subcommand
- Tool-scoped: `"Read(~/.zshrc)"` — specific file read auto-approved

## Recommended deny rules

High-value blocks that prevent accidental or unreviewed destructive actions:

```json
"deny": [
  "Bash(curl:*)",
  "Bash(wget:*)",
  "Bash(rm -rf*)",
  "Bash(git push --force*)",
  "Bash(cat ~/.env*)",
  "Bash(printenv*)",
  "Bash(env:*)",
  "Bash(ssh:*)",
  "Bash(scp:*)",
  "Bash(nc:*)"
]
```

## Notes

- `ssh:*` deny means SSH commands require manual approval each time — worth it
  if you have remote machines with key auth set up
- `allow` entries in a project-level `.claude/settings.json` stack with global ones
- Changes take effect immediately — no restart needed
