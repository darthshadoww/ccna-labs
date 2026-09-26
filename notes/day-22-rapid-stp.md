# Day 22 — Rapid STP (802.1w / Rapid PVST+)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-22-rapid-stp](../labs/day-22-rapid-stp/)
**Builds on:** [Day 20](day-20-stp.md) · [Day 21](day-21-configuring-stp.md)

---

## Why RSTP

Classic 802.1D waits **forward delay × 2** (30 s) plus Max Age on some failures. RSTP (802.1w) keeps the same tree idea but **handshakes** on point-to-point links so a new root/designated port can go forwarding in about **one hello** (sub-second in good conditions).

Cisco’s implementation on these 2960s is **Rapid PVST+** — RSTP **per VLAN**.

```
spanning-tree mode rapid-pvst
show spanning-tree          ! protocol rstp
```

## Roles (mostly familiar)

| Role | Meaning |
|------|---------|
| Root | This switch’s path to the root |
| Designated | Forwards toward this segment |
| **Alternate** | Backup toward the **root** (old non-designated) |
| **Backup** | Backup toward a designated port on the *same* segment (rare; two links to one hub) |

Edge is a **port type**, not a fourth role: PortFast + host.

## States

RSTP drops Listening:

1. **Discarding** — no user frames (covers old Blocking + Listening)
2. **Learning**
3. **Forwarding**

IOS still prints `BLK` for discarding.

## Link type — the RSTP-specific part

| Type | How IOS decides | Rapid? |
|------|-----------------|--------|
| **Point-to-point (P2p)** | Full duplex | Yes — proposal / agreement |
| **Shared (Shr)** | Half duplex or a hub | No — 802.1D timers |
| **Edge** | `spanning-tree portfast` | Immediate forwarding |

Force it when autodetect is wrong (Packet Tracer hubs especially):

```
spanning-tree link-type point-to-point
spanning-tree link-type shared
```

If the **root port is shared**, RSTP cannot sync the rest of the switch quickly. Designated ports may stay discarding until you set types correctly or timers expire. That is the SW3 picture before `link-type`.

## Proposal / agreement (exam-level)

On a P2p designated port the switch sends a **proposal** BPDU. The neighbor, if it agrees this is its root port, **agrees** and puts other ports in sync (briefly discarding). Then the designated port forwards. Shared segments skip this and wait.

## Commands

```
spanning-tree mode rapid-pvst
show spanning-tree
show spanning-tree vlan 1
show spanning-tree interface f0/1 detail

interface f0/1
 spanning-tree link-type point-to-point
interface range f0/2 - 3
 spanning-tree link-type shared
interface f0/24
 spanning-tree portfast
 spanning-tree bpduguard enable
```

## Exam checklist

- [ ] Mode is Rapid PVST+ / `protocol rstp`
- [ ] Same root election as 802.1D (BID)
- [ ] Alternate = backup path to the root
- [ ] P2p = rapid; Shr = slow; Edge = PortFast
- [ ] A hub uplink must be **shared**
- [ ] Shared root port ⇒ no fast sync for the rest of that switch
