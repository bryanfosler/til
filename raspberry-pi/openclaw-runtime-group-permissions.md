# OpenClaw: Runtime Config Must Be Group-Readable for Service Users

**Date:** 2026-04-10

`start-secure.sh` decrypts secrets into `/dev/shm/openclaw-runtime.json` and sets `chmod 600` — owner-only read. This is intentional for the main service, but breaks any secondary process that runs as a different user in the same group.

We hit this when `piper_cron.py` (runs as the `piper` user via sudo crontab) tried to read `/dev/shm/openclaw-runtime.json` to get `NOTION_API_KEY`. The `piper` user is in the `bfosler` group, but `600` means group has zero permission — the read failed silently and logged "NOTION_API_KEY not found" on every cron tick.

## Fix

Change `start-secure.sh` to set `640` instead of `600`:

```bash
# Before
chmod 600 "$RUNTIME"

# After — group-readable so piper (in bfosler group) can read it
chmod 640 "$RUNTIME"
```

Apply immediately to the live file:

```bash
chmod 640 /dev/shm/openclaw-runtime.json
```

## Why 640 Is Still Safe

- Owner (`bfosler`) has full read/write
- Group (`bfosler`) has read-only — no write access to secrets
- World has zero access
- Only users explicitly added to the `bfosler` group can read it

## The Trap

`chmod 600` looks secure and it is — for a single-user service. The moment you add a secondary agent user (piper, deployer, etc.) that needs to read secrets, group permission becomes load-bearing. `ls -la` will show `rw-------` and the issue is invisible until you check who's running the failing process.
