# Day 23 — EtherChannel

**Topic:** L2 LACP / PAgP trunks, L3 static EtherChannel, load-balancing
**Simulator:** Cisco Packet Tracer — `Day 23 Lab - EtherChannel.pkt`
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 23 (EtherChannel)

---

## 🎯 Objective

> Bundle parallel links so STP sees **one** logical port. Access–distribution is Layer 2 (LACP or PAgP) and a trunk. Distribution–distribution is Layer 3 (`mode on`, `no switchport`). Then add static routes so the PCs reach the server, and change the hash from the default **src-mac** to **src-dst-ip**.

## 🗺️ Topology

Two 2960 access switches, two 3650 distribution switches. Hosts and SVIs are preconfigured.

| Device | Role | Subnet / IP |
|--------|------|-------------|
| ASW1 | L2 access | 172.16.1.0/24 (PC1 `.1`, PC2 `.2`) |
| DSW1 | L3 distribution | VLAN 1 SVI `172.16.1.254` · L3 bundle `.1` |
| ASW2 | L2 access | 172.16.2.0/24 (SRV1 `.1`) |
| DSW2 | L3 distribution | VLAN 1 SVI `172.16.2.254` · L3 bundle `.2` |
| DSW1 ↔ DSW2 | L3 EtherChannel | `10.0.0.0/30` |

Links to bundle:

- ASW1 G0/1–2 ↔ DSW1 G1/0/3–4
- ASW2 G0/1–2 ↔ DSW2 G1/0/3–4
- DSW1 G1/0/1–2 ↔ DSW2 G1/0/1–2

![Day 23 topology — EtherChannel tasks](day23-topology.png)

## 🧠 Lesson: one channel, three ways to negotiate

| Protocol | Command (`channel-group N mode …`) | Standard | Use here |
|----------|-------------------------------------|----------|----------|
| **LACP** | `active` / `passive` | IEEE 802.3ad | ASW1 ↔ DSW1 |
| **PAgP** | `desirable` / `auto` | Cisco | ASW2 ↔ DSW2 |
| **Static** | `on` / `on` | None | DSW1 ↔ DSW2 (L3) |

`on` + anything that speaks LACP/PAgP will **not** form a channel. Both sides must match speed, duplex, and (for L2) switchport mode / native VLAN.

`show etherchannel summary` flags that matter:

| Flag | Meaning |
|------|---------|
| **S** | Layer 2 |
| **R** | Layer 3 |
| **U** | in use |
| **D** | down |
| **P** | member in the bundle |
| **I** | standalone (not bundled yet) |

Layer 2 Port-channel: `switchport mode trunk`. Layer 3: `no switchport` then `ip address` on the **Port-channel** (not the members). `switchport` on an L3 Po is the wrong direction.

## 🛠️ Configuration

### 1 — L2 LACP trunk (ASW1 ↔ DSW1) → Po1

```
interface range g0/1 - 2
 channel-group 1 mode active
 switchport mode trunk
interface port-channel 1
 switchport mode trunk
```

(DSW1: `interface range g1/0/3 - 4`, same group / mode.)

![ASW1 — Po1 (SU) LACP](day23-asw1-ethersum.png)

### 2 — L2 PAgP trunk (ASW2 ↔ DSW2) → Po2

```
interface range g0/1 - 2
 channel-group 2 mode desirable
 switchport mode trunk
```

Members start **(I)** standalone until the neighbor agrees; then they go **(P)** and the group is **(SU)**.

![ASW2 — Po2 (SU) PAgP](day23-asw2-ethersum.png)

### 3 — L3 static EtherChannel (DSW1 ↔ DSW2)

```
interface range g1/0/1 - 2
 no switchport
 channel-group 3 mode on
interface port-channel 3
 no switchport
 ip address 10.0.0.1 255.255.255.252
```

(DSW2: `10.0.0.2`.) Protocol column is `-` (static). DSW1 first showed **Po3 (RU)** — Layer 3, in use.

![DSW1 — Po1 LACP + Po3 L3 static](day23-dsw1-etherchannel.png)

![DSW2 — Po2 PAgP + static members](day23-dsw2-etherchannel.png)

Later `show ip route` lists the `/30` on **Port-channel2** (same two links, renumbered in that save). The idea is unchanged: IP lives on the Po, members are `mode on`.

### 4 — Routes so PC1/PC2 reach SRV1

SVIs are already `.254`. Enable routing on both 3650s and point at the other side of the `/30`.

```
! DSW1
ip routing
ip route 172.16.2.0 255.255.255.0 10.0.0.2

! DSW2
ip routing
ip route 172.16.1.0 255.255.255.0 10.0.0.1
```

`ip route 172.16.2.1 255.255.255.0 …` is illegal (host + /24 mask). Use the **subnet**.

![DSW1 — connected /30 + static to 172.16.2.0/24](day23-q4-dsw1.png)

![DSW2 — connected 172.16.2.0/24 + static to 172.16.1.0/24](day23-q4-dsw2.png)

PC1 → `172.16.2.1`: first ping can drop on ARP; second run is 4/4.

![PC1 ping SRV1](day23-q4-ping.png)

### 5 — Default load-balance?

`show etherchannel load-balance` → **`src-mac`** on ASW1, ASW2, DSW1, and DSW2.

![ASW1 default src-mac](day23-asw1-lb-default.png)

![ASW2 default src-mac](day23-asw2-lb-default.png)

![DSW1 default src-mac](day23-dsw1-lb-default.png)

![DSW2 default src-mac](day23-dsw2-lb-default.png)

Source-MAC hashing pins every frame from one PC onto the **same** member. Two PCs behind ASW1 can still spread (different MACs); all of PC1’s traffic will not.

### 6 — Hash on source and destination IP

```
port-channel load-balance src-dst-ip
```

That is **global**, not per-interface. `src-dest-ip` is not a keyword.

![ASW1 — src-dst-ip](day23-asw1-lb-srcdst.png)

![ASW2 — src-dst-ip](day23-asw2-lb-srcdst.png)

![DSW1 — src-dst-ip](day23-dsw1-lb-srcdst.png)

![DSW2 — src-dst-ip](day23-dsw2-lb-srcdst.png)

IPv4 then uses src XOR dst IP. Non-IP still falls back to MAC XOR.

## ✅ Verification

| Check | Expected |
|-------|----------|
| ASW1 / DSW1 `show etherchannel summary` | Po1 **(SU)** LACP, both members **(P)** |
| ASW2 / DSW2 | Po2 **(SU)** PAgP, both members **(P)** |
| DSW1–DSW2 | Static Po **(RU)**, protocol `-` |
| `show ip route` | `/30` connected on the L3 Po; static to the far `/24` |
| PC1 ping 172.16.2.1 | success after ARP |
| `show etherchannel load-balance` | `src-dst-ip` after Q6 |

## 💡 What I learned

EtherChannel is STP-friendly bandwidth: two Gig links become one logical port. LACP and PAgP negotiate; `on` does not talk to either. L2 channels are trunks; L3 channels are routed ports with the IP on the Port-channel. Default hash is **src-mac** (fine for many hosts, bad if one talker should use both links). `src-dst-ip` spreads better once you have different IP pairs. Members that sit in **(I)** usually mean the neighbor protocol or trunk settings do not match yet.

## 📎 Files in this lab

- `README.md` — this writeup
- `day23-etherchannel.pkt` — Packet Tracer save
- `day23-topology.png` — tasks + addressing (main photo)
- `day23-asw1-ethersum.png` / `day23-asw2-ethersum.png` / `day23-dsw1-etherchannel.png` / `day23-dsw2-etherchannel.png`
- `day23-q4-dsw1.png` / `day23-q4-dsw2.png` / `day23-q4-ping.png`
- `day23-*-lb-default.png` / `day23-*-lb-srcdst.png` — Q5–Q6 on all four switches
