# Study notes

Bulk notes taken ~every 10 lectures (authored in Obsidian, exported here as markdown).

## Index

| Notes | Lectures / topics covered |
|-------|---------------------------|
| [Day 15 — VLSM](day-15-vlsm.md) | FLSM vs VLSM, host-bit formula, largest-first design, `/30` P2P, classless static routes |
| [Day 16 — VLANs](day-16-vlans.md) | Access VLANs, broadcast domains, inter-VLAN routing on three physical links |
| [Day 17 — VLANs Part 2](day-17-vlans.md) | 802.1Q trunks, native VLAN 1001, allowed lists, router-on-a-stick |
| [Day 18 — Multilayer switching](day-18-multilayer-switching.md) | SVIs, routed ports, `ip routing`, replacing ROAS |
| [Day 20 — STP](day-20-stp.md) | Classic IEEE STP: BID, cost, root / designated / alternate |
| [Day 21 — Configuring STP](day-21-configuring-stp.md) | root primary/secondary, port cost vs priority, PortFast, BPDU Guard |
| [Day 22 — Rapid STP](day-22-rapid-stp.md) | RSTP / Rapid PVST+, Alternate, P2p vs shared vs edge |
| [Day 23 — EtherChannel](day-23-etherchannel.md) | LACP, PAgP, static L3 Po, load-balance src-mac → src-dst-ip |
| [Day 24 — Floating statics](day-24-floating-static.md) | AD 111 backup, OSPF 110 primary, failover through ISP |
| [Day 26 — OSPF Part 1](day-26-ospf-part-1.md) | Single-area OSPFv2, RID, passive, default-information originate |
| [Day 27 — OSPF Part 2](day-27-ospf-part-2.md) | Interface enable, ref-bw so FE=100, E2 default via best ASBR path |
| [Day 28 — OSPF Part 3](day-28-ospf-part-3.md) | Broadcast vs P2P, Hello/Dead match, Type 1/2/5 LSAs |
| [Day 29 — HSRP](day-29-hsrp.md) | HSRPv2, VIP, Active/Standby, priority, preempt |
| [Day 31 — IPv6 Part 1](day-31-ipv6-part-1.md) | Dual-stack, global unicast, link-local, ipv6 unicast-routing |
| [Day 32 — IPv6 Part 2](day-32-ipv6-part-2.md) | EUI-64, ipv6 enable (LL only), statics via FE80 next hop |
| [Day 34 — Standard ACLs](day-34-standard-acls.md) | Source-only ACLs, named vs numbered, closest-to-destination |

## Suggested cheatsheets to add over time

- Subnetting quick-reference (CIDR ↔ mask ↔ hosts) — started in [Day 15 — VLSM](day-15-vlsm.md)
- OSI vs TCP/IP model with PDUs per layer
- Common IOS commands (`show`, `config`, troubleshooting)
- Port numbers & protocols
- VLAN / trunking / STP command reference — [Day 16](day-16-vlans.md) + [Day 17](day-17-vlans.md) + [Day 20](day-20-stp.md) + [Day 21](day-21-configuring-stp.md) + [Day 22](day-22-rapid-stp.md)
- EtherChannel — [Day 23](day-23-etherchannel.md)
- Static & dynamic routing (OSPF) command reference — [Day 24](day-24-floating-static.md) + [Day 26](day-26-ospf-part-1.md) + [Day 27](day-27-ospf-part-2.md) + [Day 28](day-28-ospf-part-3.md)
- FHRP / HSRP — [Day 29](day-29-hsrp.md)
- IPv6 — [Day 31](day-31-ipv6-part-1.md) + [Day 32](day-32-ipv6-part-2.md)
- ACL and NAT syntax patterns — started in [Day 34](day-34-standard-acls.md)
