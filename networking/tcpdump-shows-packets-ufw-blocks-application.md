# tcpdump can show packets that your application never receives

If `tcpdump` (or a raw Python socket) shows traffic arriving but your application
gets zero bytes, the firewall is probably dropping it — and raw packet captures
bypass the firewall entirely.

## The trap

Debugging a GoPro USB webcam stream on a Raspberry Pi. `tcpdump` showed 640
UDP packets/second arriving on the right port. Every application-level test
(Python `socket.bind()`, `ffprobe`, `ffmpeg`) received nothing.

Spent a long time checking socket options, FFMPEG settings, and buffer sizes
before the real cause surfaced: UFW (iptables) was dropping the packets *after*
they appeared in the raw capture but *before* any application socket could see them.

## Why

Linux packet capture happens at a very early stage in the kernel network stack —
before iptables rules run. The capture path (AF_PACKET raw socket) and the
application path (AF_INET UDP socket) diverge right there:

```
NIC → kernel receive queue
  ├── AF_PACKET (tcpdump, Wireshark, raw sockets) ← sees everything, pre-firewall
  └── iptables/netfilter → route to app socket ← firewall can DROP here
```

So raw captures are pre-firewall. Application sockets are post-firewall. A packet
can appear in tcpdump and be silently dropped before any `recv()` call ever fires.

## The fix

One UFW rule:

```bash
sudo ufw allow in from 172.23.194.51 to any port 8554 proto udp
```

Stream opened instantly. All the FFMPEG debugging was unnecessary.

## Lesson

When raw sniffs show traffic but application code receives nothing, **check the
firewall before anything else.** Confirm application-layer receipt with a simple
test socket:

```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.bind(("", 8554))
s.settimeout(5)
data, addr = s.recvfrom(1500)
print(f"Got {len(data)} bytes from {addr}")
```

If this gets nothing while tcpdump sees packets, it's almost certainly iptables.

## Context

- Pi 5, Debian Trixie, UFW with default-deny-incoming
- GoPro Hero 12 in USB webcam mode — streams MPEG-TS UDP to port 8554
- GoPro USB interface creates eth1 at 172.23.194.51
