# Tailscale + Termius for Remote Terminal Access

**Date:** 2026-02-27
**Time invested:** ~45 min

## What I learned

Set up remote terminal access to both my Raspberry Pi 5 and MacBook Air using Tailscale (mesh VPN) and Termius (SSH client for iPhone/iPad).

## How it works

- **Tailscale** creates a private mesh network between devices — no port forwarding, no public IP needed. Each device gets a stable Tailscale IP (e.g. `100.x.x.x`) that works from anywhere.
- **Termius** is an SSH client app (iOS/macOS) that makes it easy to save and connect to hosts. Works great with Tailscale IPs as hostnames.

## Setup steps

1. Install Tailscale on all devices (Pi, Mac, iPhone if desired) and log in with the same account
2. On Mac: System Settings → General → Sharing → enable **Remote Login**
3. In Termius: add a new host using the device's Tailscale IP, SSH key auth
4. Connect from anywhere on the Tailscale network

## Key details

- Pi Tailscale IP: `100.x.x.x` (find yours in the Tailscale app or dashboard)
- Mac is also enrolled on the same Tailscale network
- SSH key auth (no password prompts)

## Why it matters

Can now SSH into Mac or Pi from iPhone/iPad when away from home, without any router config or exposing ports to the internet.
