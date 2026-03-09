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

Docker respects a special `DOCKER-USER` chain that runs before its own rules. Add rules there — **but watch out for the catch-all ACCEPT rule** Docker inserts (see gotcha below).

```bash
CONTAINER_IP=172.17.0.2  # get from: docker inspect <name> --format '{{.NetworkSettings.IPAddress}}'

# Insert at position 3 (before the Docker catch-all ACCEPT at rule 3):
sudo iptables -I DOCKER-USER 3 -p tcp -d $CONTAINER_IP --dport 3000 -j DROP
sudo iptables -I DOCKER-USER 3 -s 127.0.0.1 -p tcp -d $CONTAINER_IP --dport 3000 -j ACCEPT
sudo iptables -I DOCKER-USER 3 -s 100.64.0.0/10 -p tcp -d $CONTAINER_IP --dport 3000 -j ACCEPT

# Persist across reboots
sudo apt install iptables-persistent -y && sudo netfilter-persistent save
# Rules saved to /etc/iptables/rules.v4
```

Check your current rule positions first with `sudo iptables -L DOCKER-USER -n --line-numbers` and insert before the `ACCEPT all from 0.0.0.0/0` catch-all.

## Gotchas

- **Catch-all ACCEPT rule**: Docker adds an `ACCEPT all` rule to DOCKER-USER. If your DROP is inserted *after* it, the DROP is unreachable — all traffic is allowed. Always check rule order and insert at the right position.
- **Container IP can change** if Docker restarts with other containers present. For a single-container setup the IP is usually stable. Verify with `docker inspect` if rules stop working.
- **Per-port targeting**: Use `--dport` to restrict rules to a specific port; otherwise rules affect all traffic to the container.

## Affects

Any container started with `-p PORT:PORT` (no IP specified). Homebridge using `--network host` is NOT affected — host-network containers use the normal INPUT chain where UFW rules apply.
