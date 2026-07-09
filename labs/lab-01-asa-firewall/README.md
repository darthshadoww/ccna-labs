# Lab 01 — ASA 5505 Firewall: Basic Settings & Firewall Configuration

**Topic:** Network Security / Cisco ASA firewall
**Simulator:** Cisco Packet Tracer
**Reference:** CCNA Security — ASA 5505 comprehensive lab

---

## 🎯 Objective

Build a two-site network (New York ↔ Tokyo) connected across "the Internet," place a Cisco **ASA 5505** firewall in front of the Tokyo servers, and configure it so that:

- Inside hosts can reach the outside (Internet / other branch).
- Outside hosts (including the **ATTACKER** laptop) **cannot** initiate connections into the inside.
- Basic device hardening and connectivity are verified.

## 🗺️ Topology
<img width="1033" height="424" alt="image" src="https://github.com/user-attachments/assets/b204385e-bb76-48aa-80ad-8e1a79009614" />

```
 New York Branch                      The Internet                 Tokyo Branch
 ┌───────────────┐                                              ┌───────────────┐
 PC1 ─┐          │                                              │          ┌─ SR1
      ├─ SW1 ─ FW1(ASA/2911) ─ R1 ── [ Internet ] ── ASA1 ── R2 ── SW2 ─┤
 PC2 ─┘                                    │                              └─ SR2
                                           │
                                       ATTACKER (Laptop)
```

Devices:

| Device | Role |
|--------|------|
| PC1, PC2 | Inside hosts (New York) |
| SW1 | Access switch (New York) |
| FW1 / R1 | New York edge |
| ASA1 (5505) | Firewall protecting Tokyo |
| R2 | Tokyo edge router |
| SW2 | Access switch (Tokyo) |
| SR1, SR2 | Servers (Tokyo) |
| ATTACKER | Untrusted host on the outside/Internet |

> Add the topology screenshot as `lab01-topology.png` in this lab folder and it will render below.
>
> `![Topology](lab01-topology.png)`

## 🔐 Key ASA concepts practiced

- **Security levels** — `inside` = 100 (trusted), `outside` = 0 (untrusted). Traffic flows high→low by default; low→high is denied unless explicitly allowed.
- **Interfaces & nameif** — naming and assigning security levels to VLAN interfaces on the 5505.
- **Default routing** to the upstream router.
- **NAT/PAT** — translating inside addresses to the outside interface.
- **Inspection / stateful firewall** — return traffic for inside-initiated sessions is allowed automatically; the ATTACKER's unsolicited traffic is dropped.

## 🛠️ Configuration steps (outline)

> Fill in the exact addresses you used; the commands below are the ASA workflow.

```bash
! --- Basic settings ---
enable
configure terminal
hostname ASA1
domain-name lab.local
enable password <your-password>

! --- Inside interface (VLAN 1) ---
interface vlan 1
 nameif inside
 security-level 100
 ip address <inside-ip> <mask>
 no shutdown

! --- Outside interface (VLAN 2) ---
interface vlan 2
 nameif outside
 security-level 0
 ip address <outside-ip> <mask>
 no shutdown

! --- Assign physical ports to VLANs ---
interface ethernet0/0
 switchport access vlan 2      ! outside
interface ethernet0/1
 switchport access vlan 1      ! inside

! --- Default route to upstream (R2) ---
route outside 0.0.0.0 0.0.0.0 <next-hop>

! --- PAT: inside hosts hide behind the outside interface ---
object network INSIDE-NET
 subnet <inside-net> <mask>
 nat (inside,outside) dynamic interface

! --- Save ---
write memory
```

## ✅ Verification

| Test | Expected result |
|------|-----------------|
| `ping` from PC1/PC2 → SR1/SR2 (or vice-versa per lab goal) | Success (allowed direction) |
| `ping` from **ATTACKER** → inside host | **Fails** — blocked by ASA |
| `show interface ip brief` | Interfaces up/up with correct IPs |
| `show nameif` | inside=100, outside=0 |
| `show run nat` / `show xlate` | Translations building for inside traffic |
| `show route` | Default route present |

Capture screenshots of a successful ping and a blocked ATTACKER ping into this lab folder as evidence.

## 💡 What I learned

I learned how to work with CCNA packet Tracker and gained overall knowledge 

## 📎 Files in this lab

- `README.md` — this writeup
- `lab01.pkt` — Packet Tracer save _(add your file here)_
- Evidence screenshots live in this lab folder (e.g. `lab01-topology.png`, `lab01-ping-success.png`)
- Device configs (optional) in [`../../configs/`](../../configs/)
