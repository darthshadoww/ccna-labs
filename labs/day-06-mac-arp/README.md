# Day 06 — MAC Address Tables & ARP

**Topic:** Switch MAC learning, ARP, broadcast flooding
**Simulator:** Cisco Packet Tracer
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 6
**Video walkthrough:** https://www.youtube.com/watch?v=5q1pqdmdPjo

---

## 🎯 Objective

Starting from **empty MAC address tables** on both switches and **empty ARP tables** on all PCs, observe what actually crosses the wire when PC1 pings PC3, watch the switches learn MAC addresses, read those tables with `show` commands, and then clear them.

## 🗺️ Topology

<img width="2532" height="616" alt="image" src="https://github.com/user-attachments/assets/34696006-5c9b-4838-92df-16b4ac53e936" />


```
 PC1(.1) ─┐                                   ┌─ PC3(.3)
          ├─ SW1 ═══(G0/1 ── G0/1)═══ SW2 ────┤
 PC2(.2) ─┘  F0/1/F0/2            F0/1/F0/2    └─ PC4(.4)
                        192.168.1.0/24
```

| Device | IP | Connects to |
|--------|----|-----|
| PC1 | 192.168.1.1 | SW1 F0/1 |
| PC2 | 192.168.1.2 | SW1 F0/2 |
| PC3 | 192.168.1.3 | SW2 F0/1 |
| PC4 | 192.168.1.4 | SW2 F0/2 |
| SW1 ↔ SW2 | — | G0/1 ↔ G0/1 (trunk/uplink) |

## 🛠️ Tasks

### 1️⃣ If PC1 pings PC3, what messages are sent — and which devices receive them?

Both tables start empty, so PC1 knows PC3's **IP** but not its **MAC**. Before it can build the ICMP frame it must resolve the MAC via **ARP**:

1. **ARP Request** — PC1 broadcasts "Who has 192.168.1.3? Tell 192.168.1.1" with destination MAC `FFFF.FFFF.FFFF`. Because it's a **broadcast**, SW1 floods it out every other port and SW2 does the same, so **PC2, PC3, and PC4 all receive it**. Both switches learn **PC1's** MAC (source of the frame) as it passes.
2. **ARP Reply** — only **PC3** (the owner of .3) answers, with a **unicast** reply back to PC1's MAC. Path: **PC3 → SW2 → SW1 → PC1**. The switches now learn **PC3's** MAC too.
3. **ICMP Echo Request / Reply** — now that PC1 has PC3's MAC, the actual ping is unicast both ways: **PC1 → SW1 → SW2 → PC3** and back.

> **Summary:** ARP Request (broadcast, reaches everyone) → ARP Reply (unicast, PC3→PC1) → then ICMP echo request/reply. After ARP resolves, PC1 caches PC3's MAC and pings succeed.

### 2️⃣ Send the ping and verify in Simulation Mode

On **PC1 → Desktop → Command Prompt**:
```
ping 192.168.1.3
```
(Note: `ping PC3` fails — "could not find host" — because there's no DNS; ping the IP instead.)

<img width="740" height="724" alt="image" src="https://github.com/user-attachments/assets/053ab4ed-7a23-4d17-80d7-fbd2be28021a" />


Observed:
```
C:\>ping 192.168.1.3
Reply from 192.168.1.3: bytes=32 time<1ms TTL=128   (x4)
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### 3️⃣ Generate traffic so both switches learn every PC's MAC

Ping between the PCs so each switch sees a frame sourced from every host. Watch the **first** packet of a fresh ping: it's often lost (or slower) because ARP has to resolve first — e.g. `Sent = 2, Received = 1, Lost = 1 (50% loss)`. Then confirm the learned entry on the PC:
```
arp -a
```


Observed:
```
C:\>arp -a
  Internet Address    Physical Address    Type
  192.168.1.3         0004.9a6e.d870      dynamic
```

### 4️⃣ Use `show` commands to identify each PC's MAC

On **SW1 / SW2** (privileged EXEC, or `do show` from config mode):
```
show mac address-table
```
<img width="713" height="724" alt="image" src="https://github.com/user-attachments/assets/66cdcb01-e791-4c33-a82d-2d2789ec837d" />


Observed on SW1:
```
Vlan  Mac Address       Type      Ports
1     0004.9a6e.d870    DYNAMIC   Gig0/1     <- PC3, reached via the uplink to SW2
1     00d0.d3ad.9cab    DYNAMIC   Fa0/1      <- PC1, directly attached
```
The **port** column tells you where the switch will forward frames for that MAC — locally attached PCs show their access port (Fa0/x), remote PCs show the uplink (Gig0/1).

### 5️⃣ Clear the dynamic MAC addresses on each switch
```
clear mac address-table dynamic
show mac address-table
```
> ⚠️ `clear mac address-table dynamic` is a **privileged EXEC** command. From config mode it errors ("Invalid input"); prefix it with `do`: `do clear mac address-table dynamic`.
<img width="732" height="724" alt="image" src="https://github.com/user-attachments/assets/415d7afa-0ba0-4c6e-880c-80f107e2a056" />


After clearing, the table is empty until new traffic re-populates it (switches also age out entries automatically after 300 s of inactivity by default).

## 🔑 Key takeaways

| Concept | Detail |
|---------|--------|
| MAC learning | A switch learns the **source** MAC of every frame and maps it to the ingress port. |
| Unknown-unicast / broadcast | Flooded out all ports except the one it arrived on. ARP Requests are broadcasts. |
| ARP | Resolves a known **IP** to an unknown **MAC** before Layer-2 delivery. |
| First-packet loss | The very first ping can drop while ARP resolves — normal. |
| `clear mac address-table dynamic` | Wipes learned entries (privileged EXEC / `do`). Entries also age out at 300 s. |

## 💡 What I learned

I learned how to send ARP request, how to send ping over internet. I also learned how to see MAC table in Switch and how Switch dynamically learns the MAC Addresses of the devices by ARP Requests. I also saw how ping success at first fails every time. It was because of lacking ARP Request

## 📎 Files in this lab

- `README.md` — this writeup
- `day06-mac-arp.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day06-topology.png`, `day06-pc1-ping.png`, `day06-pc1-arp.png`, `day06-sw1-mac-table.png`, `day06-sw1-clear-mac.png`
