# Claude Code — Auditing your allow list with session history

You added a bunch of commands to your `settings.json` allow list, but you're
still clicking "approve" a lot. How do you actually know what's slipping
through?

Claude Code stores every session as a JSONL file in
`~/.claude/projects/<encoded-path>/`. Each Bash tool call appears as a
`tool_use` block in the `assistant` message entries. You can parse these to
see exactly which commands hit the approval prompt.

## Analysis script

```python
import json, re
from pathlib import Path
from collections import Counter

# Load your current allow list
settings = json.loads(Path('/Users/bryan/.claude/settings.json').read_text())
allowed_cmds = set()
for p in settings['permissions']['allow']:
    m = re.match(r'Bash\((\w+)', p)
    if m:
        allowed_cmds.add(m.group(1))

# Collect Bash calls from last N sessions
files = sorted(
    Path('/Users/bryan/.claude/projects/-Users-bryan/').glob('*.jsonl'),
    key=lambda f: f.stat().st_mtime, reverse=True
)[:5]

auto, manual = [], []
for f in files:
    for line in f.read_text().splitlines():
        entry = json.loads(line)
        for block in entry.get('message', {}).get('content', []) or []:
            if isinstance(block, dict) and block.get('type') == 'tool_use' and block.get('name') == 'Bash':
                cmd = block.get('input', {}).get('command', '').strip()
                first = cmd.split()[0].split('/')[-1] if cmd else ''
                (auto if first in allowed_cmds else manual).append(cmd)

print(f"Auto: {len(auto)}  Manual: {len(manual)}")
print("\nManual approval breakdown:")
for cmd, n in Counter(c.split()[0].split('/')[-1] for c in manual).most_common():
    print(f"  {n:3}x  {cmd}")
```

## What to do with the results

- **ssh** — leave manual if you have key-auth remote machines (intentional friction)
- **rm** — leave manual (destructive)
- **find, cd, tail, grep** — safe to add; they dominated the "needs approval" list
- **cd + compound commands** — `cd /path && python3 script.py` starts with `cd`,
  so adding `Bash(cd *)` covers these. The dangerous suffix commands (rm -rf, etc.)
  are already in your deny list.

## Pattern format gotcha

The allow list matches on the full command string prefix. Both formats work:

```json
"Bash(find *)",    // matches "find /path -name ..."
"Bash(find:*)"     // alternate format Claude Code also accepts
```

Adding both is redundant but harmless. Pick one.
