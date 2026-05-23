# 🖧 Ping Cheatsheet — Penetration Tester

---

## Basics

|Command|Description|
|---|---|
|`ping <host>`|Default ping|
|`ping -c 4 <host>`|Send exactly 4 packets (Linux)|
|`ping -n 4 <host>`|Send exactly 4 packets (Windows)|
|`ping -i 0.2 <host>`|Reduce interval to 0.2s|
|`ping -s 1472 <host>`|Custom packet size (bytes)|
|`ping -t <host>`|Ping continuously without stopping (Windows)|

---

## Host Discovery (Recon)

|Command|Description|
|---|---|
|`ping -c 1 <host>`|Check if host is alive|
|`ping -c 1 -W 1 <host>`|1s timeout — faster when scanning|
|`for i in {1..254}; do ping -c 1 -W 1 192.168.1.$i &>/dev/null && echo "192.168.1.$i UP"; done`|Ping sweep entire subnet|
|`ping -b 192.168.1.255`|Broadcast ping (host discovery)|

---

## Firewall / Filter Detection

|Command|Description|
|---|---|
|`ping -c 1 <host>`|High TTL (~128) → Windows; Low TTL (~64) → Linux|
|`ping -c 1 -t 1 <host>`|TTL=1, watch ICMP Time Exceeded — manual traceroute|
|`ping -M do -s 1472 <host>`|Check MTU / Path MTU Discovery|
|`ping -f <host>`|**Flood ping** — stress test (requires root) ⚠️|

---

## TTL Fingerprinting (OS Detection)

|TTL Received|Likely OS|
|---|---|
|**64**|Linux / macOS / Android|
|**128**|Windows|
|**255**|Cisco / Network devices|
|**No reply**|ICMP blocked by firewall or host is down|

---

## Ping with IPv6

```bash
ping6 <ipv6_address>
ping -6 <ipv6_address>
ping6 -c 4 fe80::1%eth0
```

---

## Bypass & Evasion

|Technique|Command|
|---|---|
|Change packet size to avoid IDS|`ping -s 64 <host>`|
|Manually set TTL|`ping -t 200 <host>` (Windows)|
|Slow interval to avoid detection|`ping -i 5 <host>`|
|Fragmented ping|`ping -f -s 1500 <host>`|

---

## Linux vs Windows Comparison

|Feature|Linux|Windows|
|---|---|---|
|Limit packet count|`-c <n>`|`-n <n>`|
|Continuous ping|default|`-t`|
|Interval|`-i <seconds>`|not natively supported|
|Packet size|`-s <bytes>`|`-l <bytes>`|
|Set TTL|`-t <ttl>`|`-i <ttl>`|
|Flood ping|`-f` (root)|not supported|

---

## Alternative / Advanced Tools

|Tool|Description|
|---|---|
|`fping -a -g 192.168.1.0/24`|Fast ping sweep of entire subnet|
|`hping3 -S <host> -p 80`|Ping over TCP (bypass ICMP filter)|
|`nmap -sn 192.168.1.0/24`|Host discovery without port scan|
|`arping <host>`|ARP ping (layer 2, LAN only)|
|`masscan --ping 192.168.1.0/24`|Ultra-fast ping sweep|

---

## Real-World Examples

```bash
# 1. Quick host check
ping -c 1 -W 1 10.10.10.1

# 2. Subnet /24 ping sweep
fping -a -g 10.10.10.0/24 2>/dev/null

# 3. Bypass ICMP block using hping3 TCP
hping3 -S 10.10.10.1 -p 443 -c 3

# 4. OS fingerprint via TTL
ping -c 1 10.10.10.1 | grep ttl

# 5. Check MTU
ping -M do -s 1472 10.10.10.1
```

---
