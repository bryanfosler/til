# Anthropic API Key — 401 "invalid-x-api-key" Error

**Date:** 2026-03-01

## What happened
After setting an Anthropic API key in openclaw, API calls returned `401 Unauthorized: invalid-x-api-key`. The key looked correct but wasn't being accepted.

## What I learned

### Common causes (in order of likelihood)

1. **Wrong key format** — Anthropic API keys start with `sk-ant-api03-…`. Keys that start with anything else are not valid Anthropic API keys.

2. **Trailing whitespace or newline in the key** — Copy-pasting from the Anthropic Console often picks up an invisible trailing space. Always trim before saving.

3. **Key stored in wrong location** — openclaw reads `ANTHROPIC_API_KEY` from the environment or from its own config. If you exported it in one shell tab and ran openclaw from a different tab, the variable isn't present.

4. **Key was revoked / never activated** — Newly created keys can take a few seconds to activate. Deleted or rotated keys fail immediately.

### How to verify

```bash
# Check the key is set and looks right (first/last few chars)
echo $ANTHROPIC_API_KEY | cut -c1-20

# Quick API sanity check (curl)
curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-haiku-4-5-20251001","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}' \
  | jq .type
# → "message" means key is valid; "error" + 401 means key is bad
```

## Fix
Get a fresh key from https://console.anthropic.com/settings/keys → copy carefully → set in both your shell and openclaw's config. Re-source `.zshrc` and restart openclaw.
