# Day 37 — NTP

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-37-ntp](../labs/day-37-ntp/)

---

## Why NTP

Logs, certs, and OSPF timestamps are useless if clocks disagree. NTP (UDP **123**) syncs from a reference; **stratum** is hop count from that clock (1 = the server itself, 16 = unsync).

## Client vs master

```
ntp server 1.1.1.1          ! I take time from this IP
ntp master 8                ! I *advertise* time at stratum 8 (backup if WAN NTP dies)
ntp update-calendar         ! copy NTP time into hardware calendar
clock timezone NAME offset  ! software clock offset (config mode)
```

A client’s stratum = server’s stratum **+ 1**. This lab: Internet st 1 → R1 st 2 → R2/R3 st 3.

## Authentication

```
ntp authentication-key 1 md5 KEY
ntp authenticate
ntp trusted-key 1
ntp server 192.168.12.1 key 1
```

All four. The MD5 string is **not** typed on the `ntp server` line.

## Verify

```
show ntp status             ! synchronized? stratum? reference?
show ntp associations       ! * selected, reach 377 = healthy
```

`reach` is an 8-bit octal shift register of the last eight polls. `377` = 11111111.

## Exam checklist

- [ ] Stratum 16 = not synchronized
- [ ] UDP 123
- [ ] `ntp server` vs `ntp master`
- [ ] Auth: key + authenticate + trusted-key + `server … key`
- [ ] `ntp update-calendar`
- [ ] `show ntp status` / `show ntp associations`
