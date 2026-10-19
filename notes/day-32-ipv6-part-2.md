# Day 32 — IPv6 configuration (Part 2)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-32-ipv6-part-2](../labs/day-32-ipv6-part-2/)
**Builds on:** [Day 31](day-31-ipv6-part-1.md) (dual-stack, global unicast, `ipv6 unicast-routing`)

---

## EUI-64

48-bit MAC → 64-bit interface ID:

1. Split MAC: `0030.F236.4502` → `0030.F2` + `36.4502`
2. Insert `FFFE` → `0030.F2FF.FE36.4502`
3. Flip U/L bit (bit 7 of first byte): `00` → `02`
4. IID = `230:F2FF:FE36:4502`

```
interface g0/1
 ipv6 address 2001:db8::/64 eui-64
```

You do **not** paste the MAC as the address. Prefix + `eui-64`. IOS also builds `FE80::` + the same IID.

This lab:

| Iface | MAC | Global |
|-------|-----|--------|
| R1 G0/1 | `0030.F236.4502` | `2001:DB8::230:F2FF:FE36:4502` |
| R2 G0/1 | `0001.63B0.B802` | `2001:DB8:0:1:201:63FF:FEB0:B802` |

PC gateway = that global, not `::1`.

## `ipv6 enable`

No global on the interface. Still gets a link-local. Use on the R1–R2 link so you can next-hop without wasting a `/64`.

```
interface g0/0
 ipv6 enable
```

R2 G0/0: `FE80::201:63FF:FEB0:B801` only. G0/1 on the same box is `…B802` plus the global.

## IPv6 static routes (preview of Day 33)

```
ipv6 route PREFIX/LEN NEXT-HOP [exit-interface]
```

If NEXT-HOP is **link-local**, the exit interface is **required**.

```
ipv6 route 2001:db8:0:1::/64 FE80::201:63FF:FEB0:B802 GigabitEthernet0/0
```

`show ipv6 route` codes: **C** connected, **L** local (/128), **S** static.

Forwarding still needs `ipv6 unicast-routing`.

## Exam checklist

- [ ] EUI-64 steps: insert `FFFE`, flip U/L
- [ ] `ipv6 address PREFIX/64 eui-64`
- [ ] `ipv6 enable` = link-local only
- [ ] PC gateway is the router's EUI-64 global
- [ ] Static to FE80 next hop must include the outgoing interface
- [ ] `show ipv6 interface brief` / `show ipv6 route` (not `show ip …`)
