# Day 31 — IPv6 configuration (Part 1)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-31-ipv6-part-1](../labs/day-31-ipv6-part-1/)

---

## Dual-stack

One interface can have IPv4 **and** IPv6. This lab keeps `192.168.X.0/24` and adds `2001:db8:0:X::/64`.

## Address types you actually configure

| Type | Prefix | This lab |
|------|--------|----------|
| Global unicast | `2000::/3` (usually `2001:db8::/32` in docs) | `2001:db8:0:1::1/64` etc. |
| Link-local | `FE80::/10` | auto on the interface |
| Unique local | `FC00::/7` | not used here |
| Multicast | `FF00::/8` | `FF02::1`, `FF02::2` |
| Loopback | `::1/128` | on the router internally |

No broadcasts. Neighbor Discovery (NS/NA) replaces ARP.

## Compression

- Drop leading zeros in a hextet: `0db8` → `db8`
- One run of zeros → `::` (only once)
- `2001:0db8:0000:0001:0000:0000:0000:0001` → `2001:db8:0:1::1`

## Commands

```
ipv6 unicast-routing          ! required to forward / send RAs
interface g0/0
 ipv6 address 2001:db8:0:1::1/64
 ipv6 enable                  ! link-local only, if you skip a global

show ipv6 interface brief
show ipv6 interface g0/0
show ipv6 route
ping ipv6 2001:db8:0:3::2
```

`show ip interface brief` does **not** show IPv6.

## PC addressing

Same idea as IPv4: host `::2`, gateway = router `::1` on that `/64`. Default prefix length **64**.

## Exam checklist

- [ ] `ipv6 unicast-routing`
- [ ] Global unicast vs link-local vs multicast
- [ ] `::` compression rules
- [ ] `/64` on a LAN
- [ ] NDP, not ARP
- [ ] `show ipv6 …` not `show ip …` for IPv6
