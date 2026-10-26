# Day 35 — Extended ACLs

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-35-extended-acls](../labs/day-35-extended-acls/)
**Builds on:** [Day 34 — Standard ACLs](day-34-standard-acls.md)

---

## Standard vs extended

| | Standard | Extended |
|--|----------|----------|
| Matches | source IP | protocol, src, dst, port |
| Numbers | 1–99, 1300–1999 | 100–199, 2000–2699 |
| Place | closest to **destination** | closest to **source** |

## Syntax

```
ip access-list extended 101
 deny ip   SRC WILD host DEST
 deny tcp  SRC WILD DEST WILD eq www
 deny udp  SRC WILD host DEST eq domain
 permit ip any any
interface g0/1
 ip access-group 101 in
```

One-line numbered form: `access-list 101 deny tcp … eq 80`

Port operators: `eq` `neq` `gt` `lt` `range`. Common names: `www` 80, `443`, `domain` 53, `ftp` 21.

## Easy misses

- No `permit ip any any` → implicit deny kills everything else
- `eq domain` = TCP 53; DNS queries are usually **UDP** 53
- `192.168.2.100 0.0.0.255` = `/24`, not host (use `host 192.168.2.100`)
- Numbered `no access-list 101 deny …` **deletes the whole list**, not one ACE (use named/sequence to edit)

## Exam checklist

- [ ] Extended = proto + src + dst + port
- [ ] Place near source
- [ ] `in` vs `out`
- [ ] `permit ip any any` after specific denies
- [ ] Port operators and `eq www` / `eq domain`
- [ ] TCP vs UDP for DNS
