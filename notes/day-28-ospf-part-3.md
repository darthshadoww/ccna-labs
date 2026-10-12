# Day 28 — OSPF network types, timers, LSAs

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-28-ospf-part-3](../labs/day-28-ospf-part-3/)
**Builds on:** [Day 26](day-26-ospf-part-1.md) · [Day 27](day-27-ospf-part-2.md)

---

## Network types (must match on the link)

| Type | Typical media | DR/BDR? | Hello / Dead |
|------|---------------|---------|--------------|
| **Broadcast** | Ethernet | Yes | 10 / 40 |
| **Point-to-point** | Serial, or Ethernet you force | No | 10 / 40 |
| **Non-broadcast (NBMA)** | Frame Relay | Yes | 30 / 120 |
| **Point-to-multipoint** | NBMA alternative | No | 30 / 120 |

```
ip ospf network point-to-point
ip ospf network broadcast
no ip ospf network point-to-point    ! back to default for the media
```

Mismatch (P2P vs broadcast) = **no adjacency**, even if IPs and area match. This lab: R3–R4 Ethernet had been set to P2P; `10.0.2.0/24` never left R3 until it was broadcast again.

On broadcast, DROther routers go **FULL only with DR and BDR** (2-Way with each other). Serial P2P: two routers, FULL, no Type-2 LSA.

## Timers (must match)

```
ip ospf hello-interval 10
ip ospf dead-interval 40
```

Dead defaults to **4 × Hello**. R5 at 5/20 vs neighbors at 10/40 will never form. `show ip ospf interface g0/0` prints both.

## Serial

```
show controllers s0/0/0     ! DCE or DTE
interface s0/0/0
 clock rate 128000          ! DCE only
 ip ospf 1 area 0
```

Default encapsulation HDLC; OSPF type **point-to-point**.

## LSA types in this lab’s LSDB

| Type | Name | Who originates | This topology |
|------|------|----------------|---------------|
| 1 | Router | Every OSPF router | Five router LSAs |
| 2 | Network | **DR** of a broadcast segment | `/29` (R5 DR) and R3–R4 after it became broadcast |
| 3 | Summary | ABR | none (single area 0) |
| 4 | ASBR-summary | ABR | ASBR `203.0.113.1` (R5) |
| 5 | External | ASBR | `0.0.0.0` from `default-information originate` |

`show ip ospf database`

## Default for Internet

R5:

```
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 1
 default-information originate
```

Internal routers install `O*E2 0.0.0.0/0`. Without this, PC1/PC2 ping LAN-to-LAN but not `8.8.8.8`.

## Exam checklist

- [ ] Broadcast vs P2P: DR or not, Type-2 or not
- [ ] Hello/Dead mismatch = no neighbor
- [ ] `clock rate` on DCE; serial is P2P
- [ ] Type 1 / 2 / 5 from a single-area LSDB
- [ ] Type 2 only from the DR
- [ ] Originate a default from the ASBR
