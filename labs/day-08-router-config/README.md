# Day 08 — Router Configuration: Connecting Multiple Networks

**Topic:** Router interfaces, IPv4 addressing, inter-network connectivity
**Simulator:** Cisco Packet Tracer
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 8
**Video walkthrough:** https://www.youtube.com/watch?v=FiAatRd84XI

---

## 🎯 Objective

Configure router **R1** so it connects three separate LANs (behind SW1, SW2, SW3), each on a different subnet. Give each router interface an IP address + description, bring it up, verify, and save. Then set static IPv4 addresses on the PCs and confirm they can ping across networks — proving the router is routing between them.

## 🗺️ Topology

<img width="1440" height="932" alt="image" src="https://github.com/user-attachments/assets/d2be94ab-6b1f-4c7d-8700-7344522676c5" />


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

<img width="784" height="724" alt="image" src="https://github.com/user-attachments/assets/0f0b85ec-48db-4e51-adbc-3e810ea1c3cb" />


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
<img width="764" height="724" alt="image" src="https://github.com/user-attachments/assets/12170bd0-14a1-47ed-b2f9-cbc06882120a" />


### 5.1 — Review the running-config
```
do show running-config
```
Confirms each interface's `ip address`, `description`, and that none are shut down.

> _Placeholder — `day08-answer5-1.png` (`show running-config` interface sections)_
<img width="783" height="724" alt="image" src="https://github.com/user-attachments/assets/5523627f-062f-4cd4-a5d1-ab333acbeb78" />


### 5.2 — Save to startup-config
```
do copy running-config startup-config
! Destination filename [startup-config]?  -> press Enter
! Building configuration... [OK]
```

<img width="784" height="724" alt="image" src="https://github.com/user-attachments/assets/65784d12-47ea-46a7-aed8-b05439804df8" />


## 💻 PC configuration (Static IPv4)

On each PC → **Config → FastEthernet0 → IP Configuration → Static**, set the IPv4 address, subnet mask, and default gateway (the router interface on that PC's network).

> _Placeholder — `day08-pc1-config.png` (PC1: 15.0.0.1 /8)_
<img width="717" height="724" alt="image" src="https://github.com/user-attachments/assets/801d1b51-1ad0-4730-93b2-2d050e6cbabd" />

>
> _Placeholder — `day08-pc2-config.png` (PC2: 182.98.0.1 /16)_
<img width="788" height="724" alt="image" src="https://github.com/user-attachments/assets/ca971b42-3546-48bb-90b3-cf8e5f99e988" />

>
> _Placeholder — `day08-pc3-config.png` (PC3: 201.191.20.1 /24)_
<img width="702" height="724" alt="image" src="https://github.com/user-attachments/assets/e99c4b12-0709-4845-97b3-2cc4f2eb3d31" />


## ✅ Verification — ping across networks

From **PC1**, ping a PC on a *different* subnet:
```
ping 201.191.20.1      ! PC3's network
ping 182.98.0.1        ! PC2's network
```
Expect the **first packet to time out** (ARP to the gateway resolving) then replies — e.g. `Sent = 4, Received = 3, Lost = 1 (25% loss)`. Success across subnets proves R1 is routing between the LANs.

>  — `day08-pc1-ping.png` (PC1 pinging across networks)_
>
<img width="701" height="724" alt="image" src="https://github.com/user-attachments/assets/b09d5d0f-1c14-4722-abd8-5afd41f6e063" />


## 💡 What I learned

I learned how to connect different networks together through a router — each interface sits in its own subnet and acts as the gateway that routes traffic between them. I learned to distinguish the parts of an IP address: the **network address** and **broadcast address** are reserved, while the addresses in between are the unique/usable host addresses. On the IOS CLI I practiced selecting and entering interfaces (e.g. `interface g0/2`), viewing interface status with `show ip interface brief`, assigning IP addresses to the router, adding a `description` to document each link, and saving the config with `copy running-config startup-config`. I also learned how to statically configure the IPv4 address, subnet mask, and default gateway on the PCs so they can reach other networks.

## 📎 Files in this lab

- `README.md` — this writeup
- `day08-router-config.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day08-topology.png`, `day08-answer1-2.png`, `day08-answer3.png`, `day08-answer4.png`, `day08-answer5-1.png`, `day08-answer5-2.png`, `day08-pc1-config.png`, `day08-pc2-config.png`, `day08-pc3-config.png`, `day08-pc1-ping.png`
