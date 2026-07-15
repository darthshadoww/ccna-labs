# Day 09 — Switch & Router Interface Configuration

**Topic:** Interface speed/duplex, descriptions, disabling unused ports, L2 vs L3 addressing
**Simulator:** Cisco Packet Tracer — `Day 09 Lab - Interface Configuration.pkt`
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 9 (Switch Interfaces)
**Lecture:** https://www.youtube.com/watch?v=cCqluocfQe0
**Lab walkthrough:** https://www.youtube.com/watch?v=rzDb5DoBKRk

---

## 🎯 Objective

Take one router (R1) and two switches (SW1, SW2) in a single `172.16.0.0/16` LAN and configure them properly: set hostnames, address the router + PCs, **manually set speed/duplex on the device-to-device links**, describe every interface, **shut down the unused switch ports**, and save.

## 🗺️ Topology
<img width="1108" height="724" alt="image" src="https://github.com/user-attachments/assets/730ba128-0043-4c4a-b04b-b2ae2f75cc2d" />


Network: **172.16.0.0/16** (mask 255.255.0.0, gateway = R1 = 172.16.255.254)

| Device | Interface | IP / setting | Link to |
|--------|-----------|--------------|---------|
| R1 | G0/0 | 172.16.255.254 /16 | SW1 G0/1 |
| SW1 | G0/1 | (L2, no IP) | R1 G0/0 |
| SW1 | G0/2 | (L2, no IP) | SW2 G0/1 |
| SW1 | F0/1, F0/2 | (L2) | PC1, PC2 |
| SW2 | G0/1 | (L2, no IP) | SW1 G0/2 |
| SW2 | F0/1, F0/2 | (L2) | PC3, PC4 |
| PC1 | — | 172.16.0.1 /16, GW .255.254 | SW1 F0/1 |
| PC2 | — | 172.16.0.2 /16, GW .255.254 | SW1 F0/2 |
| PC3 | — | 172.16.0.3 /16, GW .255.254 | SW2 F0/1 |
| PC4 | — | 172.16.0.4 /16, GW .255.254 | SW2 F0/2 |

> 🔑 **Switches are Layer-2** here, so their access/uplink ports don't get IP addresses — only **R1** and the **PCs** are addressed. (A switch would only get an IP on a management SVI, e.g. `interface vlan 1`.)

## 🧠 Concepts from the lecture

- **`show interfaces status`** — the switch-focused view: port, status (connected/notconnect/disabled), VLAN, **duplex**, **speed**, and type. `show ip interface brief` is the router-focused view (IP + up/down).
- **Speed & duplex** — can be autonegotiated (default) or set manually. Best practice is to hard-set both ends on device-to-device links so there's no mismatch.
- **Duplex** — *full duplex* = send + receive simultaneously (switched links); *half duplex* = one direction at a time (legacy hubs, uses **CSMA/CD** to handle collisions within a **collision domain**).
- **`interface range`** — configure many ports at once, e.g. `interface range f0/3 - 24`.

## 🛠️ Configuration

### 1 — Hostnames (all three devices)
```
enable
configure terminal
hostname SW1        ! (hostname SW2 / hostname R1 on the others)
```
<img width="777" height="724" alt="image" src="https://github.com/user-attachments/assets/b3e3f274-740e-4e23-87fd-cce4a0796c00" />


### 2 — IP addressing (R1 + PCs)
R1's routed interface:
```
interface g0/0
 ip address 172.16.255.254 255.255.0.0
 no shutdown
```
PCs: **Config → FastEthernet0 → IP Configuration → Static** → IP `172.16.0.x`, mask `255.255.0.0`, gateway `172.16.255.254`.

> _Placeholder — `day09-answer2.png` (R1 G0/0 IP config + equivalent IOS commands)_
<img width="751" height="724" alt="image" src="https://github.com/user-attachments/assets/882a6cda-bd46-4856-b417-70810918c6df" />


### 3 — Manual speed & duplex on device-to-device links only
Only the **router↔switch** and **switch↔switch** (Gigabit) links — *not* the ports to PCs:
```
interface g0/0            ! on R1  (and g0/1, g0/2 on SW1; g0/1 on SW2)
 speed 1000
 duplex full
```
> _Placeholder — `day09-answer3.png` (R1: speed 1000 / duplex full on G0/0)_
<img width="712" height="724" alt="image" src="https://github.com/user-attachments/assets/f66b47a9-d1f1-4803-967c-9e667a3a8c59" />


### 4 — Descriptions on each interface
```
interface g0/1
 description ## to R1 ##
interface g0/2
 description ## to SW2 ##
```
Verify with `do show interfaces status` (or `show int status`).

<img width="1030" height="724" alt="image" src="https://github.com/user-attachments/assets/82b8b7cf-50e0-48c4-9f04-89ac480190c5" />


### 5 — Disable unused interfaces
Bring up the ports in use, then shut and label the rest so nobody can plug into a live port:
```
interface range f0/1 - 2
 no shutdown
exit
interface range f0/3 - 24
 description ## not in use ##
 shutdown
```
Each disabled port reports `changed state to administratively down`.

> _Placeholder — `day09-answer5.png` (SW1 disabling the unused port range)_
<img width="1072" height="724" alt="image" src="https://github.com/user-attachments/assets/b0f781b6-1a57-4fd5-a4b1-9f69a329f3a3" />


### 6 — Save the configuration
```
end
copy running-config startup-config      ! press Enter → Building configuration... [OK]
```
> ℹ️ `show int status` on a **router** returns "Invalid input" — that's a switch command; use `show ip interface brief` on R1.

> _Placeholder — `day09-answer6.png` (R1 `show ip int brief` + `copy run start` [OK])_
<img width="1059" height="724" alt="image" src="https://github.com/user-attachments/assets/64c8eebd-d900-41d3-8b38-38f3020e5e27" />


## ✅ Verification

| Check | Expected |
|-------|----------|
| `show interfaces status` (switches) | uplinks `connected`, a-full/a-100 or forced 1000/full; unused ports `disabled` |
| `show ip interface brief` (R1) | G0/0 `up/up` with 172.16.255.254 |
| PC → PC ping across switches | success (all in 172.16.0.0/16) |
| `show running-config` | descriptions + speed/duplex present; startup matches after save |

## 💡 What I learned

The new skills this lab added: **shutting down unused switch ports** (and labelling them `## not in use ##`) as a basic security hardening step, and manually setting **speed** and **duplex** on the links between networking devices (router↔switch and switch↔switch) instead of relying on autonegotiation. I also reinforced connecting routers and switches together, configuring IP addresses manually on the PCs and the router (while remembering that Layer-2 switch ports don't take IPs), adding descriptions to document each link, reading port state with `show interfaces status`, using `interface range` to configure many ports at once, and saving with `copy running-config startup-config`.

## 📎 Files in this lab

- `README.md` — this writeup
- `day09-interface-config.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day09-topology.png`, `day09-answer1.png` … `day09-answer6.png`
