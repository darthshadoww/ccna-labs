# Day 26 — OSPF Part 1 (single area)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-26-ospf-part-1](../labs/day-26-ospf-part-1/)
**Builds on:** [Day 24](day-24-floating-static.md) (OSPF as an IGP you already *saw*; now you configure it)

---

## Why OSPF

Link-state: routers flood **LSAs**, each builds the same LSDB, SPF computes a tree. AD **110**. Metric is **cost** (reference bandwidth / interface bandwidth). This lecture is **one area (0)**.

## Core commands

```
router ospf 1
 router-id 1.1.1.1
 network 10.0.12.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 passive-interface loopback0
 default-information originate
```

Interface style (same result):

```
interface g0/0
 ip ospf 1 area 0
```

RID: manual `router-id` > highest loopback > highest physical. Set it; reload or `clear ip ospf process` if you change it later.

## Wildcards (exam muscle memory)

| Mask | Wildcard | Example |
|------|----------|---------|
| `/32` | `0.0.0.0` | loopback |
| `/30` | `0.0.0.3` | P2P |
| `/24` | `0.0.0.255` | LAN |

`network` matches **interfaces**, not “the remote subnet.” If the interface IP falls in the statement, that link is in OSPF.

## Passive vs no OSPF at all

| | In OSPF LSDB? | Hellos? |
|---|---|---|
| Normal | yes | yes — neighbors |
| `passive-interface` | **yes** (subnet advertised) | **no** |
| Interface omitted | no | no |

Use passive on **loopbacks** and **host LANs**. Do **not** put the ISP link in OSPF at all if you do not want that prefix flooded (this lab).

## Neighbor states (need to recognize)

Down → Init → 2-Way → ExStart → Exchange → Loading → **FULL**

Must match: area, timers (hello/dead), network type, stub flags, MTU. Point-to-point `/30` is the easy case (no DR/BDR fight that matters).

## External default: `O*E2`

```
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 1
 default-information originate
```

- **ASBR** = router that injects an external prefix
- **E2** (default) = metric stays the same everywhere (`[110/1]` in this lab)
- **E1** (`originate metric-type 1`) = metric **plus** internal cost
- `*` = candidate default (`Gateway of last resort`)

R4 can install **two** E2 defaults if both paths to the ASBR tie.

`default-information originate always` advertises even if R1 has no default of its own.

## Cost / ECMP

Equal cost → both next hops (see R2 to `3.3.3.3`, R4 to `1.1.1.1` and to `0.0.0.0`). Change with `ip ospf cost` or `auto-cost reference-bandwidth`.

## Exam checklist

- [ ] `network` wildcard vs `ip ospf 1 area 0`
- [ ] RID rules
- [ ] Passive = advertise, no hello
- [ ] Do not OSPF the ISP link; originate a default instead
- [ ] `O*E2` = external type 2 default
- [ ] `/30` cannot use the network address as a host IP
