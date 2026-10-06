# Day 24 — Floating static routes

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-24-floating-static](../labs/day-24-floating-static/)
**Builds on:** [Day 15 statics](day-15-vlsm.md) · OSPF shows up as the primary IGP here

---

## Administrative distance (the whole trick)

IOS compares **AD** before metric. Lower AD wins, even if the metric is huge.

| Source | Default AD |
|--------|------------|
| Connected | 0 |
| Static | 1 |
| eBGP | 20 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |
| iBGP | 200 |

A **floating static** is a static whose AD you set **higher** than the IGP you trust:

```
ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
```

`111` > OSPF `110` → backup only. Leave the AD off and the static (AD 1) **overrides** OSPF.

## How you know it is floating

| Command | Link up | Link down |
|---------|---------|-----------|
| `show run \| include ip route` | static is listed | still listed |
| `show ip route` | **not** in the table | `S … [111/0]` |

`show ip route 10.0.2.0` is useful: it prints candidate vs installed.

## Recursion / next hop

The next hop must be **reachable** (usually a connected `/30`). Here `203.0.113.1` is ISPR1 on R1’s G0/0/0. If that ISP link is also down, the floating static cannot install either.

Exit interface form (`ip route 10.0.2.0 255.255.255.0 g0/0/0 111`) is legal on point-to-point Ethernet in some IOS; next-hop IP is the usual CCNA style.

## vs default route

`S* 0.0.0.0/0` is a different prefix. It sends **unknown** destinations to the Internet. It does **not** replace a more-specific `10.0.2.0/24` when you want PC1 ↔ SRV1 to stay “internal” until the internal link fails. That is why this lab adds a floating **LAN** static, not a second default.

## Failover sequence (this lab)

1. OSPF FULL on R1–R2 G0/2/0 → `O 10.0.2.0/24 [110/2] via 10.0.0.2`
2. Configure AD 111 statics → they stay in NVRAM/running-config only
3. `shutdown` G0/2/0 → neighbor DOWN, `O` withdrawn
4. `S 10.0.2.0/24 [111/0] via 203.0.113.1` installs
5. Return traffic needs the **mirror** static on R2 (`10.0.1.0/24` via `203.0.113.5`)

## Exam checklist

- [ ] AD is why “floating” works
- [ ] OSPF 110 vs static 1 vs static 111
- [ ] Backup does not appear in `show ip route` until the better route is gone
- [ ] Both directions need a floating static for a ping to return
- [ ] More-specific prefix beats a default, even after failover
