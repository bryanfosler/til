# OpenClaw: CLI Commands Need OPENCLAW_CONFIG_PATH to Match Running Service

**Date:** 2026-03-08

When `start-secure.sh` decrypts secrets into `/dev/shm/openclaw-runtime.json` and the service runs with `OPENCLAW_CONFIG_PATH=/dev/shm/openclaw-runtime.json`, CLI commands (`npx openclaw logs`, `npx openclaw skills list`, etc.) still default to reading `~/.openclaw/openclaw.json` — which has empty/stale secrets.

This causes "gateway token mismatch" errors because the CLI sends the wrong token to authenticate with the gateway.

## Fix

Add to `~/.bashrc` on the Pi:

```bash
export OPENCLAW_CONFIG_PATH=/dev/shm/openclaw-runtime.json
```

## Also: Both Token Fields Must Match

`start-secure.sh` must inject the gateway token into **both** config fields:

```bash
jq \
  --arg gw "$(decrypt openclaw-gateway-token)" \
  '.gateway.auth.token = $gw | .gateway.remote.token = $gw' \
  "$TEMPLATE" > "$RUNTIME"
```

- `gateway.auth.token` — what the **server** accepts
- `gateway.remote.token` — what the **client** sends when connecting

If only one is set, every CLI command fails with `token_mismatch`.
