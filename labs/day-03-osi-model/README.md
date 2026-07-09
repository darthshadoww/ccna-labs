# Day 03 — The OSI Model (Simulation Mode)

**Topic:** OSI reference model, encapsulation, protocol analysis
**Simulator:** Cisco Packet Tracer — `Day 03 Lab - OSI Model.pkt`
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 3

---

## 🎯 Objective

Use Packet Tracer's **Simulation Mode** to watch real traffic move through the network and identify which **OSI layers** each protocol operates at. Then release and renew PC1's IP address to generate **Layer 7 (application)** traffic and analyze it packet-by-packet.

## 🗺️ Topology
<img width="2600" height="1682" alt="image" src="https://github.com/user-attachments/assets/d1c29fb5-d78f-4f1a-bf0f-5765d40d5e87" />


| Device | Interface / IP | Notes |
|--------|----------------|-------|
| SRV1 (Server-PT) | .100 in 192.168.1.0/24 | Server / DHCP-DNS candidate |
| SW1 (2960-24TT) | F0/1, G0/1, G0/2 | Access switch |
| PC1 (PC-PT) | 192.168.1.10 /24, GW 192.168.1.1 | Test host |
| SW2 (2960-24TT) | F0/1, G0/1 | Access switch for PC1 |
| R1 (2911) | G0/0 .1, G0/1 .1 | Router, 192.168.1.0/24 ↔ 10.0.0.0/24 |
| R2 (2911) | G0/0 .2 | Router in 10.0.0.0/24 |

**Topology screenshot:**

![Uploading image.png…]()

## 🧩 OSI model refresher

| # | Layer | PDU | Example protocols seen in this lab |
|---|-------|-----|------------------------------------|
| 7 | Application | Data | **DHCP**, DNS, HTTP |
| 6 | Presentation | Data | (encoding/encryption) |
| 5 | Session | Data | (session setup) |
| 4 | Transport | Segment | **UDP** (DHCP uses 67/68), TCP |
| 3 | Network | Packet | **IP**, **ICMP**, routing |
| 2 | Data Link | Frame | **Ethernet**, **ARP**, STP |
| 1 | Physical | Bits | copper/cabling, signaling |

## 🛠️ Tasks

### Task 1 — Analyze traffic in Simulation Mode

1. Switch from **Realtime** to **Simulation** (bottom-right).
2. Send a simple ping / let background traffic flow and step through with **Play / Capture-Forward**.
3. Click each packet (the envelope icon) and open the **OSI Model** tab in the PDU details window.
4. Record which layers each protocol touches.

**What layers of the OSI model are being used?**
- **ARP** → operates at Layer 2 (frame) but resolves a Layer 3 address, so you see L2 + L3 fields.
- **ICMP** (ping) → Layer 3, carried in an Ethernet frame (L2) over the physical link (L1).
- **STP / CDP** background frames → Layer 2.
- Once DHCP runs (Task 2) → **Layer 7** application data down through L4 (UDP) → L3 (IP) → L2 (Ethernet) → L1.

### Task 2 — Generate Layer 7 traffic (release/renew PC1)

On **PC1 → Desktop → Command Prompt**:

```
ipconfig /release
ipconfig /renew
```

`release` sends a **DHCP Release**; `renew` triggers **DHCP Discover → Offer → Request → Ack (DORA)** — all **Layer 7** application messages riding on **UDP (L4)**. Watch the DORA exchange step-by-step in Simulation Mode and inspect the OSI tab of each DHCP packet.

**CLI evidence:**
<img width="1446" height="1322" alt="image" src="https://github.com/user-attachments/assets/1ae1bc3b-5612-4092-8f00-3c3e6b1c699b" />

Reference output captured:

```
C:\>ipconfig
FastEthernet0 Connection:(default port)
   IPv4 Address..............: 192.168.1.10
   Subnet Mask...............: 255.255.255.0
   Default Gateway...........: 192.168.1.1

C:\>ipconfig /release
   IP Address................: 0.0.0.0
   Subnet Mask...............: 0.0.0.0
   Default Gateway...........: 0.0.0.0
   DNS Server................: 0.0.0.0
```

After `/release` the interface drops to `0.0.0.0`; after `/renew` it should pull an address back from the DHCP server (SRV1).

## ✅ Verification

| Check | Expected |
|-------|----------|
| `ipconfig` before release | 192.168.1.10 /24, GW 192.168.1.1 |
| `ipconfig /release` | address falls to 0.0.0.0 |
| `ipconfig /renew` | address re-leased from DHCP |
| Simulation Mode PDU → OSI tab | correct layers highlighted per protocol |
| DHCP exchange visible | Discover / Offer / Request / Ack |

## 💡 What I learned

_(2–3 sentences after finishing — e.g. how encapsulation adds a header at each layer going down and strips it going up, why ARP shows both L2 and L3 info, and how DHCP is an application-layer protocol even though it's about IP addressing.)_

## 📎 Files in this lab

- `README.md` — this writeup
- `day03-osi-model.pkt` — Packet Tracer save _(add your file)_
- Screenshots in this folder: `day03-topology.png`, `day03-ipconfig-release.png`
