# Day 17 — VLANs (Part 2): trunks & router-on-a-stick

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-17-vlans](../labs/day-17-vlans/)
**Builds on:** [Day 16 — VLANs](day-16-vlans.md)

---

## Access vs trunk

| | Access | Trunk |
|---|---|---|
| VLANs | **One** | **Many** |
| Tag | None (host never sees 802.1Q) | 802.1Q tag (4 bytes) except native VLAN |
| Typical use | PC, printer, phone (data VLAN) | Switch–switch, switch–router (ROAS) |

```
switchport mode access
switchport access vlan 10

switchport mode trunk
switchport trunk native vlan 1001
switchport trunk allowed vlan 10,30
```

Always set `switchport mode trunk` (or `access`) instead of leaving DTP to negotiate. CCNA: know that **DTP** can form a trunk automatically; production practice is to hard-set the mode.

## 802.1Q tag

Inserted after the source MAC:

- TPID `0x8100`
- PCP / DEI / **VLAN ID** (12 bits → VLANs 1–4094)

The receiving switch reads the ID, then forwards inside that VLAN’s broadcast domain.

## Native VLAN

The native VLAN crosses the trunk **untagged**. Default is VLAN 1.

Rules that matter:

1. Both ends of a trunk must use the **same** native VLAN. A mismatch means untagged frames land in the wrong VLAN (and CDP/STP complaints on real gear).
2. Do **not** put user hosts in the native VLAN.
3. Pick an unused VLAN (this lab: **1001**) and create it on both switches.

```
vlan 1001
interface g0/1
 switchport mode trunk
 switchport trunk native vlan 1001
```

VLAN IDs are 1–4094. `10001` is invalid / a typo.

## Allowed VLAN list

By default a trunk allows **all** VLANs. Restrict it to what the far side actually needs:

```
switchport trunk allowed vlan 10,30
```

In this lab:

- SW1–SW2: **10, 30** (VLAN 20 exists only on SW2)
- SW2–R1: **10, 20, 30** (R1 must route all three)

`show interfaces trunk` columns:

- **Vlans allowed on trunk** — the configured list
- **Vlans allowed and active** — those that also exist (`vlan` created) and are not shut
- **Native vlan**
- **Encapsulation** — `802.1q`

## Router-on-a-stick (ROAS)

One physical router interface, one **subinterface per VLAN**.

```
interface g0/0
 no shutdown          ! physical port must be up; no IP here
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.62 255.255.255.192
```

Order matters: **`encapsulation` before `ip address`**. The VLAN ID in `dot1Q` must match the switch trunk.

Compared with Day 16 (one cable per VLAN):

| Day 16 | Day 17 ROAS |
|--------|-------------|
| 3 router ports | 1 router port |
| Access links to R1 | One 802.1Q trunk to R1 |
| `ip address` on G0/0, G0/1, G0/2 | `ip address` on G0/0.10 / .20 / .30 |

Same-switch, different-VLAN traffic still goes to the router and back (TTL 127). Same-VLAN traffic across SW1–SW2 stays Layer 2 and does **not** use R1.

`show interfaces trunk` is a **switch** command. On the router use `show ip interface brief` and confirm each subinterface is `up/up`.

## Commands

```
show vlan brief
show interfaces trunk
show interfaces g0/1 switchport
show ip interface brief

switchport mode trunk
switchport trunk native vlan 1001
switchport trunk allowed vlan 10,20,30

encapsulation dot1Q 10
```

## Exam checklist

- [ ] PC ports: `mode access` + correct `access vlan`
- [ ] VLANs created on **every** switch that needs them (including the native VLAN)
- [ ] Trunk mode set on both ends
- [ ] Native VLAN matches and is unused
- [ ] Allowed list is the minimum required
- [ ] ROAS: physical `no shutdown`, no IP on the physical port
- [ ] `encapsulation dot1Q` matches the VLAN before the gateway IP
- [ ] Inter-VLAN ping TTL decrements; same-VLAN across a trunk does not need the router
