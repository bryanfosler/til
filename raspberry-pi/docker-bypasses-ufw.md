# Docker Bypasses UFW Firewall Rules

**Date:** 2026-03-07

Docker publishes ports by adding iptables DNAT rules in its own `DOCKER` chain, which runs in the `PREROUTING` hook — before UFW's `INPUT` chain. This means UFW rules for Docker-published ports are silently ignored.

## Example

UFW rule: `3000 ALLOW IN 100.64.0.0/10` (Tailscale only)
Reality: port 3000 is accessible from the entire internet.

You can confirm with:
```bash
sudo iptables -t nat -L DOCKER -n
# Shows: DNAT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:3000 to:172.17.0.2:8080
# The 0.0.0.0/0 source means all IPs — UFW rule doesn't apply
```

## Fix: DOCKER-USER chain

Docker respects a special `DOCKER-USER` chain that runs before its own rules. Add rules there:

```bash
CONTAINER_IP=172.17.0.2  # get from: docker inspect <name> --format '{{.NetworkSettings.IPAddress}}'

# Allow Tailscale
sudo iptables -I DOCKER-USER -s 100.64.0.0/10 -d $CONTAINER_IP -j ACCEPT
# Allow localhost
sudo iptables -I DOCKER-USER -s 127.0.0.1 -d $CONTAINER_IP -j ACCEPT
# Block everything else
sudo iptables -I DOCKER-USER -d $CONTAINER_IP -j DROP

# Persist across reboots
sudo apt install iptables-persistent -y && sudo netfilter-persistent save
```

## Affects

Any container started with `-p PORT:PORT` (no IP specified). Homebridge using `--network host` is NOT affected — host-network containers use the normal INPUT chain where UFW rules apply.
