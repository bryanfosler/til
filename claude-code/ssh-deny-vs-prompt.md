# Claude Code: ssh in deny list = auto-reject, not prompt

If `Bash(ssh:*)` is in the `deny` list in `~/.claude/settings.json`, Claude Code **auto-rejects** SSH commands without showing an approval prompt — even if you're trying to approve them manually.

Deny rules take precedence over everything, including user interaction. The approval prompt only appears when a command is in neither `allow` nor `deny`.

**To get prompted for SSH approval:** remove `Bash(ssh:*)` from `deny`. Since it's not in `allow` either, it falls through to the default "ask each time" behavior.

```json
// deny list — remove this line to get prompted instead of auto-rejected:
"Bash(ssh:*)"
```

This is useful for Pi SSH sessions where you want visibility and approval without blocking entirely.
