# Day 34 — Standard ACLs

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-34-standard-acls](../labs/day-34-standard-acls/)
**Builds on:** [Day 26–28 OSPF](day-26-ospf-part-1.md)

---

## What a standard ACL matches

**Source IPv4 only.** No protocol, no dest port, no dest IP. That is why you put them **as close to the destination as you can** — otherwise you block the source from *everything* downstream.

| | Numbered | Named |
|--|----------|-------|
| ID | `1–99`, `1300–1999` | `ip access-list standard NAME` |
| Edit | rewrite the whole list (`no access-list N`) | `no sequence`, insert by sequence |
| Apply | `ip access-group N in\|out` | `ip access-group NAME in\|out` |

Last line is always **implicit deny any**.

## Wildcards (the easy mistakes)

| Wildcard | Means | Example |
|----------|-------|---------|
| `0.0.0.0` | this host | `permit 172.16.1.1 0.0.0.0` (PC1 only) |
| `0.0.0.255` | last octet any = `/24` | `permit 172.16.1.1 0.0.0.255` = `172.16.1.0/24` |
| `0.0.255.255` | `/16` | `172.16.0.0 0.0.255.255` |

`permit 172.16.1.1` with no wildcard = host (same as `0.0.0.0`).

## This lab's placement

| Policy | Where | Why |
|--------|-------|-----|
| Only PC1/PC3 → `192.168.1.0/24` | R2 G0/0 **out** | closest to SRV1 |
| `172.16.2.0/24` ↛ `192.168.2.0/24` | R2 G0/1 **out** | closest to SRV2 |
| The two `172.16` LANs ↛ each other | R1 LAN interfaces | closest to those dests |

## Commands

```
access-list 1 deny 172.16.2.0 0.0.0.255
access-list 1 permit any
interface g0/0
 ip access-group 1 out

ip access-list standard TO_SRV1
 permit 172.16.1.1 0.0.0.0
 permit 172.16.2.1 0.0.0.0
interface g0/0
 ip access-group TO_SRV1 out

show ip access-lists
show ip interface g0/0
```

`show ip route` does not show ACLs. Unreachable from the **router's** interface IP = that router dropped the packet.

## Exam checklist

- [ ] Standard = source only
- [ ] Place near destination
- [ ] Named vs numbered syntax
- [ ] `in` vs `out` on the correct interface
- [ ] Host wildcard `0.0.0.0` vs subnet `0.0.0.255`
- [ ] Implicit deny any
- [ ] `permit any` if you used an explicit `deny` and still want the rest through
