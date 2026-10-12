# Day 27 — OSPFv2 configuration (single area)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-27-ospf-part-2](../labs/day-27-ospf-part-2/)
**Builds on:** [Day 26 OSPF Part 1](day-26-ospf-part-1.md) · [Day 24 floating statics](day-24-floating-static.md)

---

## Enable OSPF on the interface

```
interface g0/0
 ip ospf 1 area 0
```

Process ID is local (does not have to match neighbors). Area **must** match. Equivalent old style:

```
router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
```

This lab asked for the interface form.

## Router ID

Highest loopback, else highest up interface, unless you set it:

```
router ospf 1
 router-id 1.1.1.1
```

Changing RID needs `clear ip ospf process`. If OSPF started **before** the loopback existed, you keep an interface IP as RID (this lab: R2 showed `10.0.24.1`).

## Passive interfaces

Advertise the prefix, **do not** send Hellos (no adjacency):

```
router ospf 1
 passive-interface loopback 0
 passive-interface g0/0          ! R4 PC LAN
```

Loopbacks and access LANs: passive. Router–router links: not passive. The ISP link on R1: **not in OSPF at all**.

## Cost / reference bandwidth

```
cost = reference-bandwidth / interface-bandwidth
```

Default reference is **100 Mbps** → FE and GE both cost 1. To make FE **100**:

```
auto-cost reference-bandwidth 10000    ! Mbps; GE → 10, FE → 100
```

Must match on **every** router or SPF disagrees. Manual override: `ip ospf cost 100` on the interface.

## Neighbor states (exam)

Down → Init → 2-Way → ExStart → Exchange → Loading → **Full**.

Broadcast Ethernet elects **DR/BDR** at 2-Way. Point-to-point skips DR.

## Packet types

| Type | Name | Role |
|------|------|------|
| 1 | Hello | discover / keep alive |
| 2 | DBD | describe LSDB |
| 3 | LSR | request LSAs |
| 4 | LSU | send LSAs |
| 5 | LSAck | acknowledge |

Hellos carry RID, area ID, timers (Hello 10 / Dead 40 on broadcast), neighbors, DR/BDR.

## ASBR default

```
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 1
 default-information originate
```

Without a default already in the table, originate injects nothing. Internal routers see **`O*E2 0.0.0.0/0 [110/1]`**.

- **E2** (default): external metric **does not** accumulate; stays 1.
- **E1** (`originate metric-type 1`): external + intra-area cost.
- Path selection among E2s still uses **cost to the ASBR**. Two ways to R1 from R4: via R2 cost 110 vs via R3 cost 200 → **one** default via R2.

## Exam checklist

- [ ] `ip ospf 1 area 0` vs `network` statement
- [ ] RID rules / `router-id`
- [ ] Passive = prefix yes, Hello no
- [ ] `auto-cost reference-bandwidth` so FE ≠ GE
- [ ] `default-information originate` needs a default on the ASBR
- [ ] `O*E2` vs E1; only the best path to the ASBR shows in the table
