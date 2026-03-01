# zsh: `compdef` requires `compinit` to run first

If you see this error every time you open a terminal:

```
/path/to/some-tool/completions/tool.zsh:NNN: command not found: compdef
```

It means a shell completion script is being sourced **before** zsh's completion system is initialized.

## Why it happens

`compdef` is a zsh function that only exists after `compinit` is called. If a tool adds a `source /path/to/completions.zsh` line to your `.zshrc` and that line runs before `compinit`, zsh doesn't know what `compdef` means yet.

## Fix

Add `autoload -Uz compinit && compinit` **before** the offending `source` line in `~/.zshrc`:

```zsh
autoload -Uz compinit && compinit
source "/path/to/completions/tool.zsh"
```

## What actually happened

This came up because `openclaw` was briefly installed on the Mac (later uninstalled), but its completion script `source` line remained in `.zshrc`. The binary was gone but the line lingered — and the completion script used `compdef` without zsh being initialized first.
