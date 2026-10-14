# Day 29 — HSRP (FHRP)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-29-hsrp](../labs/day-29-hsrp/)
**Builds on:** [Day 24](day-24-floating-static.md) (backup paths) · default gateway on a LAN

---

## Why FHRP

Two routers on a LAN, hosts with **one** default gateway. If that IP lives on a single router, that router dying black-holes the LAN. A **First Hop Redundancy Protocol** shares a **virtual IP** (and virtual MAC). Hosts keep the same gateway; the routers elect who owns it.

## The three CCNA FHRPs

| | HSRP | VRRP | GLBP |
|---|------|------|------|
| Roles | Active / Standby | Master / Backup | AVG / AVF |
| Virtual MAC | v1 `0000.0C07.ACxx` · v2 `0000.0C9F.Fxxx` | `0000.5E00.01xx` | `0007.B400.xxyy` |
| Multicast | v1 224.0.0.2 · v2 **224.0.0.102** | 224.0.0.18 | 224.0.0.102 |
| Cisco-only | yes | no (IETF) | yes |
| Load share | no (one Active) | no | yes (AVFs) |

This lab is **HSRPv2**.

## Commands

```
interface g0/0
 standby version 2
 standby 1 ip 10.0.1.254
 standby 1 priority 255      ! default 100; 0–255
 standby 1 preempt
 standby 1 timers 3 10       ! hello / hold (defaults)
show standby
show standby brief
```

VIP must be in the same subnet, unused by a physical interface.

## Election

1. Highest **priority**
2. Tie → highest physical IP

**Preempt off (default):** a new better router does **not** steal Active. **Preempt on:** it does, after hold time.

States: Init → Learn → Listen → Speak → **Standby** / **Active**.

## Virtual MAC (v2)

Group 1 → `0000.0C9F.F001`. Hosts cache this MAC for `10.0.1.254`. On failover the new Active sources from the **same** vMAC (gratuitous ARP), so hosts do not need a new ARP entry.

## What `tracert` shows

Hop 1 is often the Active router’s **real** IP (`.252` or `.253`), even though the PC gateway is `.254`. That is a quick “who is Active?” check.

## Exam checklist

- [ ] VIP vs physical IPs
- [ ] Active/Standby vs VRRP Master/Backup vs GLBP AVG/AVF
- [ ] v2 MAC `0000.0C9F.Fxxx` vs v1 `0000.0C07.ACxx`
- [ ] Default priority 100; preempt required to take over
- [ ] `standby 1 preempt` (group number required)
- [ ] Hello 3 / hold 10
