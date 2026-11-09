# Day 39 — DHCP

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-39-dhcp](../labs/day-39-dhcp/)

---

## DORA

| Message | Who | Typical |
|---------|-----|---------|
| Discover | client → server | broadcast |
| Offer | server → client | broadcast or unicast |
| Request | client → server | broadcast |
| Ack | server → client | broadcast or unicast |

UDP **67** (server) / **68** (client). Release is unicast.

## Server

```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name jeremysitlab.com
 lease 0 12 0                 ! days hours minutes (optional)
```

Excluded addresses are **not** leased. First assignable in this lab is `.11`.

## Client (a router)

```
interface g0/0
 ip address dhcp
```

## Relay

Routers do not forward broadcasts. Put helper on the **client LAN** interface:

```
interface g0/1
 ip helper-address 203.0.113.1
```

Also forwards other UDP (TFTP, DNS, …) unless you tighten it.

## Verify

```
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show ip interface g0/1        ! Helper address
```

PC: `ipconfig /renew` then `ipconfig`.

## Exam checklist

- [ ] DORA + ports 67/68
- [ ] `excluded-address` before the pool
- [ ] `default-router` = the LAN gateway, not always the DHCP server
- [ ] `ip address dhcp` vs `ip dhcp pool`
- [ ] `ip helper-address` on the **client** interface
