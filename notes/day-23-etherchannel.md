# Day 23 — EtherChannel

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-23-etherchannel](../labs/day-23-etherchannel/)
**Builds on:** [Day 17 trunks](day-17-vlans.md) · [Day 18 MLS](day-18-multilayer-switching.md) · [Day 20–22 STP](day-20-stp.md)

---

## Why bundle?

Two links between the same switches without EtherChannel: STP blocks one. With EtherChannel, STP sees **one** Port-channel, both members forward, and a hash picks the member per flow.

## Protocols

| | LACP | PAgP | Static |
|---|------|------|--------|
| Standard | IEEE 802.3ad | Cisco | none |
| Modes | `active` / `passive` | `desirable` / `auto` | `on` |
| Forms a channel | active+active, active+passive | desirable+desirable, desirable+auto | on+on only |
| Will **not** form | active+on, passive+passive | desirable+on, auto+auto | on + LACP/PAgP |

Passive+passive and auto+auto never start negotiation.

## Layer 2 vs Layer 3

```
! L2 (access–distribution) — usually a trunk
interface range g0/1 - 2
 channel-group 1 mode active
 switchport mode trunk
interface port-channel 1
 switchport mode trunk

! L3 (distribution–distribution)
interface range g1/0/1 - 2
 no switchport
 channel-group 3 mode on
interface port-channel 3
 no switchport
 ip address 10.0.0.1 255.255.255.252
```

Put the IP on the **Po**, not the physical members. Need `ip routing` on the MLS.

## Must match on members

Speed, duplex, native VLAN, allowed VLANs, access vs trunk, and protocol/mode. A mismatch leaves the port **standalone (I)** or **suspended**.

## `show etherchannel summary`

```
Po1(SU)   LACP   Gi0/1(P) Gi0/2(P)
Po3(RU)   -      Gi1/0/1(P) Gi1/0/2(P)
```

- **S/R** — Layer 2 / Layer 3
- **U/D** — up / down
- **P/I** — bundled / standalone

Also: `show etherchannel 1 port-channel`, `show interfaces port-channel 1`.

## Load-balancing (the hash)

Global command:

```
show etherchannel load-balance
port-channel load-balance src-dst-ip
```

| Method | Hash input |
|--------|------------|
| `src-mac` | **default** on these switches |
| `dst-mac` | destination MAC |
| `src-dst-mac` | MAC XOR |
| `src-ip` / `dst-ip` | one IP |
| `src-dst-ip` | src XOR dst IP (IPv4/IPv6) |

One flow (same hash) **always** uses the same member. EtherChannel is per-flow, not per-packet. Default src-mac is enough when many hosts sit behind the channel; one server talking to many clients wants dest or src-dst so the far addresses differ.

`src-dest-ip` is not valid — it is `src-dst-ip`.

## Exam checklist

- [ ] Why STP + EtherChannel (one logical link)
- [ ] LACP vs PAgP vs `on` — which pairs form
- [ ] L2 trunk Po vs L3 `no switchport` Po
- [ ] Flags: SU, RU, P, I
- [ ] Default load-balance is **src-mac**
- [ ] `port-channel load-balance` is global
