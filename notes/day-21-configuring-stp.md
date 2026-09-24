# Day 21 — Configuring STP

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-21-configuring-stp](../labs/day-21-configuring-stp/)
**Builds on:** [Day 20 — analyzing STP](day-20-stp.md)

---

## PVST+ (one tree per VLAN)

Cisco’s default on these 2960s is **PVST+** (`protocol ieee` in Packet Tracer). VLAN 1 and VLAN 2 elect **separate** roots and can block different ports.

Displayed priority = configured priority + VLAN ID (`sys-id-ext`).

| Command | Typical priority |
|---------|-----------------:|
| `spanning-tree vlan 1 root primary` | 24576 → shows **24577** |
| `spanning-tree vlan 1 root secondary` | 28672 → shows **28673** |
| default | 32768 → shows **32769** |

`root primary` is “make me root unless someone uses an even lower priority.” `root secondary` is the backup root if the primary dies.

## Port cost vs port priority

Root-port decision order (same as Day 20):

1. Lowest **root-path cost**
2. Lowest **upstream Bridge ID**
3. Lowest **upstream port ID** (priority.number)

```
interface f0/2
 spanning-tree vlan 1 cost 1000          ! changes step 1
interface f0/1
 spanning-tree vlan 1 port-priority 240  ! changes step 3 only
```

Default port priority is **128**. Valid values are multiples of 16 (0–240). Higher number = *worse* (less preferred).

This lab’s demo:

- SW4 F0/2 cost **1000** → direct link to the VLAN 1 root loses to a two-hop path of cost **38** → **root port moves**.
- SW1 F0/1 port-priority **240** → SW3 still uses F0/1; the other path costs 38 vs 19 → **no move**.

## PortFast

Access ports to PCs should not wait 30 s (listen + learn). PortFast jumps to **forwarding**.

```
interface f0/3
 switchport mode access
 spanning-tree portfast
```

Never on a port that faces another switch. A loop plus PortFast = broadcast storm before STP can block.

Global option (real IOS): `spanning-tree portfast default` (still only access ports).

## BPDU Guard

If an edge port hears a **BPDU**, something that is not a PC got plugged in.

```
spanning-tree bpduguard enable
```

Result: port **err-disabled** (`administratively down`).

```
%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU ...
%PM-4-ERR_DISABLE: bpduguard error detected
```

Recover:

1. Unplug the rogue switch.
2. `shutdown` then `no shutdown` on the interface.

Optional on real gear: `errdisable recovery cause bpduguard` + `errdisable recovery interval 30`.

PortFast and BPDU Guard are a pair: PortFast for speed, Guard so a mistaken trunk cannot loop.

## Commands

```
spanning-tree vlan 1 root primary
spanning-tree vlan 2 root secondary
spanning-tree vlan 1 priority 24576     ! manual equivalent

interface f0/2
 spanning-tree vlan 1 cost 1000
 spanning-tree vlan 1 port-priority 240
 spanning-tree portfast
 spanning-tree bpduguard enable

show spanning-tree
show spanning-tree vlan 1
show spanning-tree interface f0/3 detail
show interfaces status                  ! err-disabled
```

## Exam checklist

- [ ] Primary / secondary root **per VLAN** (PVST+)
- [ ] Cost changes root-path cost; priority is a tiebreak
- [ ] PortFast only on access / host ports
- [ ] BPDU Guard err-disables on any BPDU
- [ ] Clear err-disable with shut / no shut after fixing the cabling
