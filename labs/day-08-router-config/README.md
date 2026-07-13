# Day 08 — Router Configuration: Connecting Multiple Networks

**Topic:** Router interfaces, IPv4 addressing, inter-network connectivity
**Simulator:** Cisco Packet Tracer
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 8
**Video walkthrough:** https://www.youtube.com/watch?v=FiAatRd84XI

---

## 🎯 Objective

Configure router **R1** so it connects three separate LANs (behind SW1, SW2, SW3), each on a different subnet. Give each router interface an IP address + description, bring it up, verify, and save. Then set static IPv4 addresses on the PCs and confirm they can ping across networks — proving the router is routing between them.

## 🗺️ Topology

> _Placeholder — drop the lab's own topology photo as `day08-topology.png`_
>
> ![Day 08 topology](day08-topology.png)

| Router interface | IP address | Mask | Description | Network |
|------------------|-----------|------|-------------|---------|
| G0/0 | 15.255.255.254 | 255.0.0.0 (/8) | `## to SW1 ##` | 15.0.0.0/8 |
| G0/1 | 182.98.255.254 | 255.255.0.0 (/16) | `## to SW2 ##` | 182.98.0.0/16 |
| G0/2 | 201.192.20.1 | 255.255.255.0 (/24) | `## to SW3 ##` | 201.192.20.0/24 |

| PC | IPv4 | Mask | Default gateway (router interface) |
|----|------|------|-----------------------------------|
| PC1 | 15.0.0.1 | 255.0.0.0 | 15.255.255.254 |
| PC2 | 182.98.0.1 | 255.255.0.0 | 182.98.255.254 |
| PC3 | 201.191.20.1 | 255.255.255.0 | 201.192.20.1 |

> _(Use the exact addresses from your own `.pkt` — the values above are read off the screenshots.)_

## 🧠 Addressing concept — host vs network address

Every subnet reserves two addresses that can't be assigned to a device:
- **Network address** (all host bits 0) — e.g. `15.0.0.0` — names the network itself.
- **Broadcast address** (all host bits 1) — e.g. `15.255.255.255` — reaches every host at once.

Everything in between is a **usable/unique host address**. The router's gateway (`15.255.255.254`) is just one of those host addresses, conventionally the highest usable one.

## 🛠️ Router configuration (R1)

### 1 & 2 — Hostname + inspect starting state
```
enable
configure terminal
hostname R1
do show ip interface brief      ! interfaces start "unassigned / administratively down"
```
> ℹ️ The correct verification command is **`show ip interface brief`** (not `show ip address-config`, which IOS rejects as invalid).

> _Placeholder — `day08-answer1-2.png` (hostname set + initial `show ip interface brief`)_
>
> ![Answer 1 & 2](day08-answer1-2.png)

### 3 — Configure an interface (IP + description + enable)
```
interface g0/2
 description ## SW3 ##
 ip address 201.192.20.1 255.255.255.0
 no shutdown
exit
```
`no shutdown` brings the interface up (`%LINK-3-UPDOWN ... changed state to up`). Repeat for G0/0 and G0/1 with their addresses.

> _Placeholder — `day08-answer3.png` (configuring G0/2: ip address + description + no shut)_
>
> ![Answer 3](day08-answer3.png)

### 4 — Verify all interfaces are up
```
do show ip interface brief
```
All three (G0/0, G0/1, G0/2) should read **`up / up`** with their assigned IPs.

> _Placeholder — `day08-answer4.png` (all interfaces up with IPs)_
>
> ![Answer 4](day08-answer4.png)

### 5.1 — Review the running-config
```
do show running-config
```
Confirms each interface's `ip address`, `description`, and that none are shut down.

> _Placeholder — `day08-answer5-1.png` (`show running-config` interface sections)_
>
> ![Answer 5.1](day08-answer5-1.png)

### 5.2 — Save to startup-config
```
do copy running-config startup-config
! Destination filename [startup-config]?  -> press Enter
! Building configuration... [OK]
```

> _Placeholder — `day08-answer5-2.png` (`copy run start` → [OK])_
>
> ![Answer 5.2](day08-answer5-2.png)

## 💻 PC configuration (Static IPv4)

On each PC → **Config → FastEthernet0 → IP Configuration → Static**, set the IPv4 address, subnet mask, and default gateway (the router interface on that PC's network).

> _Placeholder — `day08-pc1-config.png` (PC1: 15.0.0.1 /8)_
> ![PC1 config](day08-pc1-config.png)
>
> _Placeholder — `day08-pc2-config.png` (PC2: 182.98.0.1 /16)_
> ![PC2 config](day08-pc2-config.png)
>
> _Placeholder — `day08-pc3-config.png` (PC3: 201.191.20.1 /24)_
> ![PC3 config](day08-pc3-config.png)

## ✅ Verification — ping across networks

From **PC1**, ping a PC on a *different* subnet:
```
ping 201.191.20.1      ! PC3's network
ping 182.98.0.1        ! PC2's network
```
Expect the **first packet to time out** (ARP to the gateway resolving) then replies — e.g. `Sent = 4, Received = 3, Lost = 1 (25% loss)`. Success across subnets proves R1 is routing between the LANs.

> _Placeholder — `day08-pc1-ping.png` (PC1 pinging across networks)_
>
> ![PC1 ping](day08-pc1-ping.png)

## 💡 What I learned

I learned how to connect different networks together through a router — each interface sits in its own subnet and acts as the gateway that routes traffic between them. I learned to distinguish the parts of an IP address: the **network address** and **broadcast address** are reserved, while the addresses in between are the unique/usable host addresses. On the IOS CLI I practiced selecting and entering interfaces (e.g. `interface g0/2`), viewing interface status with `show ip interface brief`, assigning IP addresses to the router, adding a `description` to document each link, and saving the config with `copy running-config startup-config`. I also learned how to statically configure the IPv4 address, subnet mask, and default gateway on the PCs so they can reach other networks.

## 📎 Files in this lab

- `README.md` — this writeup
- `day08-router-config.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day08-topology.png`, `day08-answer1-2.png`, `day08-answer3.png`, `day08-answer4.png`, `day08-answer5-1.png`, `day08-answer5-2.png`, `day08-pc1-config.png`, `day08-pc2-config.png`, `day08-pc3-config.png`, `day08-pc1-ping.png`
