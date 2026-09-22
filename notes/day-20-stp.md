# Day 20 — Spanning Tree (classic IEEE)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-20-analyzing-stp](../labs/day-20-analyzing-stp/)

---

## Why STP exists

Ethernet frames have no TTL. Two switches with two cables between them will flood forever (**broadcast storm**), corrupt MAC tables, and duplicate frames. STP blocks **just enough** ports so the remaining graph is a tree — still connected, no L2 loop.

This lab is **802.1D** (`protocol ieee`), not RSTP/MST.

## Bridge ID

```
Bridge ID = priority + MAC
Displayed priority = configured priority + VLAN (sys-id-ext)
```

Default configured priority **32768**. On VLAN 1 that shows as **32769**.

Lower BID wins. Priority is compared first; MAC is the tiebreak.

```
spanning-tree vlan 1 priority 24576    ! must be a multiple of 4096
show spanning-tree
```

## Root port (non-root switches only)

Lowest **root path cost**. Ties:

1. Lowest **upstream** Bridge ID
2. Lowest **upstream** port ID (priority.number)

IEEE port costs (this lab):

| Speed | Cost |
|-------|-----:|
| 10 Mb | 100 |
| 100 Mb (Fa) | **19** |
| 1 Gb (Gi) | **4** |
| 10 Gb | 2 |

A path of two Gig hops = **8**, which beats one FastEthernet hop (**19**).

## Designated vs non-designated

Each **segment** (cable) gets **one** designated port — the end with the better path to the root. The other end is **non-designated / alternate** and goes **BLK**.

On the root, **every** port is designated.

IOS names:

| Role | Sts | Meaning |
|------|-----|---------|
| Root | FWD | This switch’s path to the root |
| Desg | FWD | Designated for this segment |
| Altn | BLK | Alternate (backup) to the root — non-designated |

## Port states (802.1D)

1. Blocking — no user frames
2. Listening — 15 s (forward delay)
3. Learning — 15 s, builds MAC table
4. Forwarding

Max Age **20 s**, Hello **2 s**. A topology change can take ~30–50 s to reconverge. (RSTP, later, is much faster.)

## Reading `show spanning-tree`

```
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    24577
             Address     00E0.F9E6.44A5
             Cost        19
             Port        4 (FastEthernet0/4)
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
```

- If Root ID Address **equals** Bridge ID Address → **this switch is the root**.
- **Cost** under Root ID is *this switch’s* cost to the root (0 on the root).
- **Port** under Root ID is the root port.

## Decision order (exam)

1. Elect root (lowest BID)
2. Each non-root switch: one root port (lowest cost to root)
3. Each link: one designated port (closest to root)
4. Everything else: block

## Commands

```
show spanning-tree
show spanning-tree vlan 1
show spanning-tree root
show spanning-tree bridge
show spanning-tree interface f0/1 detail

spanning-tree vlan 1 priority 24576
spanning-tree vlan 1 root primary     ! sets a low priority for you
```

Packet Tracer: hide link lights so you are forced to read the CLI (`Options → Preferences → Show Link Lights`).

## Exam checklist

- [ ] Root = lowest priority, then lowest MAC
- [ ] Shown priority includes VLAN sys-id-ext
- [ ] Root port = lowest cost, then neighbor BID, then neighbor port ID
- [ ] Gigabit cost 4 vs FastEthernet 19 — add hops
- [ ] Alternate / BLK is the backup, not a broken port
- [ ] Root switch has no root port — all designated
