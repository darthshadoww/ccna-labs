# Day 36 — CDP & LLDP

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-36-cdp-lldp](../labs/day-36-cdp-lldp/)

---

## Two Layer-2 discovery protocols

| | CDP | LLDP |
|--|-----|------|
| Standard | Cisco proprietary | IEEE 802.1AB |
| Default on Cisco | **on** (global + every iface) | **off** |
| Timer / holdtime | 60 / 180 | 30 / 120 |
| Multicast MAC | `0100.0ccc.cccc` | `0180.c200.000e` |

Both stay on the local link (not forwarded). Can run together.

## Commands

```
show cdp neighbors [detail]
show cdp interface
show cdp entry HOST
cdp run                    ! global
no cdp enable              ! one interface (e.g. PC port)

lldp run                   ! global
interface g0/0
 lldp transmit
 lldp receive
show lldp neighbors [detail]
```

`no cdp run` kills CDP everywhere. `no cdp enable` is per port.

## Table columns

- **Device ID** — hostname
- **Local Intrfce** — *your* interface
- **Port ID** — *their* interface
- **Capability** — R router, S switch (CDP) / B bridge (LLDP)

`show cdp neighbors detail` adds IP address and IOS version — useful for labeling a `/30` when you only know one end.

## Exam checklist

- [ ] CDP on by default; LLDP must be enabled
- [ ] Place: map unlabeled cables with `show cdp neighbors`
- [ ] Disable CDP toward endpoints: `no cdp enable`
- [ ] LLDP needs **both** `transmit` and `receive` (or `lldp run`)
- [ ] Neighbor must also speak LLDP or it will not show
- [ ] Spelling: `neighbors`
