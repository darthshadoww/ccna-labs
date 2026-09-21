# Day 18 — Multilayer switching

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-18-multilayer-switching](../labs/day-18-multilayer-switching/)
**Builds on:** [Day 17 — trunks & ROAS](day-17-vlans.md)

---

## Why an L3 switch?

Router-on-a-stick (Day 17) works, but every inter-VLAN packet must leave the switch, be routed, and come back on the same trunk. A **multilayer switch** (MLS) routes **inside the chassis** using:

- **SVIs** — virtual gateways (`interface vlan 10`)
- **Routed ports** — physical ports with IPs (`no switchport`)
- **`ip routing`** — actually turn on IPv4 forwarding (off by default on many L3 switches)

Campus design: L3 at the distribution layer, L2 access switches behind it. This lab is that in miniature (SW2 = distribution, SW1 = access, R1 = WAN/Internet edge).

## SVI (Switch Virtual Interface)

```
interface vlan 10
 ip address 10.0.0.62 255.255.255.192
 no shutdown
```

The SVI is up only if:

1. The VLAN exists (`vlan 10`)
2. At least one **access or trunk** port in that VLAN is up (or the SVI is forced)
3. The SVI itself is `no shutdown`

Hosts still ARP for the gateway MAC — it is now SW2’s SVI MAC, not R1’s.

## Routed port vs SVI vs trunk

| | SVI | Routed port | Trunk |
|---|---|---|---|
| Command | `interface vlan 10` | `no switchport` then `ip address` | `switchport mode trunk` |
| Layer | L3 bound to a VLAN | L3 on one physical port | L2, many VLANs |
| This lab | Gateways for 10/20/30 | SW2 G1/0/2 ↔ R1 | SW1 ↔ SW2 G1/0/1 |

You **cannot** trunk and IP the same port. Replacing ROAS means removing `switchport mode trunk` on the R1 link and using `no switchport` instead.

## `ip routing`

Without it, SVIs are only for management (like VLAN 1 on an L2 switch). With it, SW2 builds a routing table and forwards between Vlan10, Vlan20, Vlan30, and the `/30`.

```
ip routing
show ip route
```

Expect `C` routes for each SVI and the routed port.

## Default route on the MLS

VLAN subnets are connected. Everything else (Internet `1.1.1.0/24`) is not, so SW2 needs:

```
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

R1 already has a default toward the Internet (`S* 0.0.0.0/0` via G0/0/0) plus return routes for `10.0.0.0` in this lab file.

## Reading TTL

| Ping | TTL | Meaning |
|------|-----|---------|
| Same VLAN | 128 | L2 only |
| Inter-VLAN via SW2 | **127** | One router hop (the MLS) |
| PC → `1.1.1.1` | **253** | Reply started at 255; R1 and SW2 each decrement |

If an inter-VLAN ping still shows a path through **R1**, ROAS was not fully removed.

## Commands

```
ip routing
interface vlan 10
 ip address 10.0.0.62 255.255.255.192
 no shutdown

interface g1/0/2
 no switchport
 ip address 10.0.0.193 255.255.255.252

ip route 0.0.0.0 0.0.0.0 10.0.0.194

show ip route
show ip interface brief
show vlan brief
show interfaces trunk    ! should list SW1 uplink, not the R1 link
```

## Exam checklist

- [ ] `ip routing` enabled on the MLS
- [ ] SVI per VLAN, last usable IP, `no shutdown`
- [ ] VLAN exists and has an active port or the SVI will stay down
- [ ] Uplink to the WAN router: `no switchport` + IP, **not** a trunk
- [ ] Default route on the MLS pointing at the WAN router
- [ ] Inter-VLAN ping does **not** need the WAN router
- [ ] Internet ping uses MLS default → edge router → cloud
