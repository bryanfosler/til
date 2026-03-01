# openclaw — Telegram Pairing: Devices vs. Pairing Flow

**Date:** 2026-03-01

## What happened
Trying to connect Telegram to openclaw got confusing because the UI has two different concepts that sound alike: **Devices** and **Pairing**.

## What I learned

### The distinction

| Term | What it means |
|------|--------------|
| **Devices** | The list of already-authorized clients connected to your Telegram account. |
| **Pairing** | The one-time flow to add a *new* client (like openclaw) to that list. |

You pair once → the new device shows up in the Devices list → pairing is done.

### The correct pairing flow

1. In openclaw, initiate the Telegram connection (it will prompt for a phone number or give you a QR code).
2. In Telegram on your phone: **Settings → Devices → Link Desktop Device** (scan QR or enter the code openclaw shows).
3. Once linked, openclaw appears in your Devices list. Done — you do not need to "pair" again unless you log out.

### Debugging "it's not connecting"

- **Check Telegram → Settings → Devices** first. If openclaw already appears there, it's authorized — the issue is elsewhere (polling, bot token, etc.).
- If openclaw does **not** appear, re-run the pairing flow from scratch.
- Telegram bot tokens (`123456:ABC-DEF…`) are separate from account-level pairing — bots don't show up in Devices at all.

## Gotcha
Logging out of Telegram on your phone can revoke all linked devices including openclaw. You'd need to re-pair.
