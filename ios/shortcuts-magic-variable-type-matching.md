# iOS Shortcuts: Magic Variables Resolve by Type, Not by Proximity

When you use a "magic variable" (a blue auto-resolving token) in an iPhone Shortcut action, Shortcuts finds the variable by scanning backwards through previous actions and matching on **output type** — not by picking the nearest action.

## Why This Matters

If you have two actions that could satisfy the type:
- Action A outputs TEXT (e.g., a `Get Variable` storing a logID string)
- Action B outputs FILE (e.g., a `Get File` from iCloud Drive)

And a later action expects a **FILE** type as input, Shortcuts will grab the FILE output — even if Action A (TEXT) appears between them.

## Real Example

Building a shortcut to deduplicate Fitbit weight entries:

```
Get file from Shortcuts → "fitbit-last-log-id.txt"  [FILE type, empty if not exist]
Get Variable: LogID                                  [TEXT type]
Save File → [logID] → Shortcuts/fitbit-last-log-id.txt
```

The `Save File` action expects a **file** to save. Its magic variable token resolved to the output of `Get file from Shortcuts` (FILE type, empty) — completely skipping `Get Variable: LogID` (TEXT type). The shortcut ran without error but kept saving an empty file instead of the logID string.

Result: deduplication never worked, and the shortcut logged duplicate weight entries to Apple Health on every run.

## The Fix

Add a `Text` action immediately before `Save File`:

```
Text: [LogID]
Save File → [Text] → Shortcuts/fitbit-last-log-id.txt
```

The `Text` action outputs **TEXT type**. `Save File` accepts TEXT (it coerces it to file content), and since it's immediately before the action, the magic variable resolves correctly.

## General Rule

Any time a Shortcuts action uses the "wrong" variable — especially when there are multiple outputs of different types in your chain — suspect type-matching. The fix is usually to add an explicit `Text [variable]` step to force the correct type into the pipeline right before the consuming action.
