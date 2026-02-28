# Installing an iOS App on Your Own iPhone (Free, No App Store)

**Date:** 2026-02-27
**Time invested:** ~30 min

## What I learned

You can install a native iOS app you've built directly on your iPhone using Xcode and a free Apple ID — no $99/year developer account, no App Store submission required.

## The two tiers

| | Free (Personal Team) | Paid ($99/yr) |
|---|---|---|
| Install on your own device | ✅ | ✅ |
| App Store submission | ❌ | ✅ |
| TestFlight beta sharing | ❌ | ✅ |
| App expires after | 7 days | 1 year |

For personal projects you're building for yourself, the free tier is completely sufficient.

## What "code signing" is

iOS requires every app binary to be cryptographically signed before it can run. Xcode generates a **signing certificate** and stores it in your Mac's Keychain. When you see the prompt "codesign wants to access key Apple Development: Your Name in your keychain" — that's Xcode unlocking the Keychain to sign your app. Enter your Mac login password and allow it. Totally normal, only happens once per new certificate.

## Enabling Developer Mode (iOS 16+)

Apple added a Developer Mode lock in iOS 16. You have to explicitly enable it before a dev-signed app will run on your device.

1. Connect iPhone to Mac via USB
2. Hit Play in Xcode — it'll fail and tell you to enable Developer Mode
3. On iPhone: **Settings → Privacy & Security → Developer Mode → toggle ON**
4. iPhone restarts
5. After restart, confirm the prompt that appears

This is a kernel-level security change (not just a setting), so the restart is required.

## Full first-time setup flow

1. **Xcode → Settings → Accounts** — add your Apple ID
2. **Target → Signing & Capabilities** — set Team to "Your Name (Personal Team)"
3. **Mac Keychain prompt** — enter Mac login password to allow codesign
4. **iPhone → "Trust This Computer?"** — tap Trust, enter passcode
5. **Enable Developer Mode** on iPhone (see above)
6. Select your device in Xcode's run destination dropdown
7. Hit ▶ Play — app installs
8. **iPhone → Settings → General → VPN & Device Management → your Apple ID → Trust**

Step 8 is the "trust the developer" step — iOS blocks apps from unknown sources even after install. You manually approve your own certificate once, and every future app signed with the same cert skips this step.

## The 7-day expiry

The provisioning profile expires after 7 days. The app won't disappear from your phone, but iOS won't launch it. Fix: plug in to Xcode and hit Play again. Your app data survives — only the authorization resets.
