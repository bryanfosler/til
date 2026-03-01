# Homebridge + Google Nest → Apple HomeKit Bridge on Pi (No $5 Fee)

You can bridge pre-Matter Google Nest devices (thermostat, doorbell/camera) to Apple HomeKit using Homebridge on a Raspberry Pi — without paying Google's $5 Device Access Console fee.

## The Setup

**Stack:** Homebridge in Docker + `homebridge-nest-accfactory` plugin + Google cookie auth

```bash
sudo docker run -d \
  --name homebridge \
  --restart=unless-stopped \
  --net=host \
  -e PGID=1000 -e PUID=1000 \
  -e TZ=America/Los_Angeles \
  -v /opt/homebridge:/homebridge \
  ghcr.io/homebridge/homebridge:latest
```

Install the plugin to the correct path (not global npm):
```bash
sudo docker exec homebridge npm install --prefix /var/lib/homebridge homebridge-nest-accfactory
```

## Auth Without the $5 Fee

Google charges $5 to access the official Smart Device Management (SDM) API. The `homebridge-nest-accfactory` plugin supports an unofficial cookie-based auth method instead.

**Must use Safari** — Chrome/Firefox don't generate valid tokens due to ITP differences.

1. Safari → Private window → DevTools → Network tab → Preserve Log → filter `issueToken`
2. Navigate to `https://home.nest.com` → Sign in with Google
3. Click the `iframerpc` request → Headers tab:
   - **Summary → URL** = `issuetoken`
   - **Request → Cookie** = `cookie` (must include `SIDCC=`, copy the entire line)

**config.json format** (key names are exact — wrong names cause silent failures):

```json
{
    "platform": "NestAccfactory",
    "google": {
        "issuetoken": "https://accounts.google.com/o/oauth2/iframerpc?...",
        "cookie": "SIDCC=xxx; SID=xxx; ...",
        "fieldTest": false
    },
    "name": "NestAccfactory"
}
```

## Gotchas

- Plugin must be installed with `--prefix /var/lib/homebridge` — Homebridge uses `-P /var/lib/homebridge/node_modules`, global npm is ignored
- Wrong config keys (`googleAuth`, `cookies`, `issueToken`) fail silently with "No connections specified"
- Do NOT log out of `home.nest.com` — invalidates the tokens
- 1st and 2nd gen Nest Learning Thermostats were dropped by Google in October 2025 — neither SDM nor cookie auth will work for those models

## What Works in HomeKit

- Thermostat: current temp, setpoint, mode (heat/cool/auto/off), Siri control
- Doorbell: live view (via cloud), motion events, doorbell press triggers
- HKSV recording: add `"hksv": true` to `options` block in config
