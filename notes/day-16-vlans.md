# Day 16 — VLANs (Part 1)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-16-vlans](../labs/day-16-vlans/)

---

## What a VLAN is

A **VLAN (Virtual LAN)** is a Layer-2 broadcast domain created in software. Ports assigned to the same VLAN behave as if they were on their own switch. Ports in different VLANs do **not** exchange frames, even when they sit on the same physical switch.

Before VLANs, isolating departments meant buying a separate switch (and a router between them). VLANs give you that isolation on one switch.

| Same VLAN | Different VLANs |
|-----------|-----------------|
| Frames switched (MAC table) | Frames cannot cross at Layer 2 |
| Broadcasts stay here | Need a **router** (or L3 switch) |
| Ping TTL stays 128 on this lab’s PCs | Ping TTL drops to 127 after R1 |

## Access vs trunk (this lab is access only)

- **Access port** — belongs to **one** VLAN. End hosts (PCs) and, in this lab, each R1 uplink.
- **Trunk port** — carries **many** VLANs with an 802.1Q tag. That is the next lecture (router-on-a-stick). Not used here.

```
switchport mode access
switchport access vlan 10
```

Default VLAN is **1**. Unused ports left in VLAN 1 are a common security miss; this lab still shows VLAN 1 with leftover Fa9/1.

## VLAN IDs and names

- Normal-range VLANs: **1–1005** (1, 1002–1005 are reserved/default)
- Extended: 1006–4094 (need VTP transparent / config mode on real gear)
- The **ID** is what forwarding uses. The **name** is documentation.

```
vlan 10
 name Engineering
```

A typo in the name (`ENGINIGGEERING`) does not break connectivity.

## Inter-VLAN routing in this lab

R1 has **three physical interfaces**, one in each subnet:

```
G0/0  10.0.0.62/26   ← VLAN 10
G0/1  10.0.0.126/26  ← VLAN 20
G0/2  10.0.0.190/26  ← VLAN 30
```

The matching SW1 Gigabit port must be an **access port in that same VLAN**. If the uplink stays in VLAN 1, the PC’s gateway is unreachable.

This is **not** router-on-a-stick: no subinterfaces, no `encapsulation dot1q`.

## Broadcast domains

Ping the subnet broadcast and only that VLAN’s hosts light up:

| VLAN | Broadcast |
|------|-----------|
| 10 | `10.0.0.63` |
| 20 | `10.0.0.127` |
| 30 | `10.0.0.191` |

That is the point of Simulation Mode in the lab: prove VLANs split the broadcast domain.

## Addressing reminder (`/26`)

```
usable = 2^6 - 2 = 62
mask   = 255.255.255.192
block size = 64  →  .0 / .64 / .128 / .192
```

Lab rule: first usable addresses on PCs, **last usable** on the router (gateway).

## Commands

```
show vlan brief          ! switch — VLAN ID, name, access ports
show ip interface brief  ! router — gateway IPs up/up
show interfaces status   ! switch port VLAN / speed / duplex
show interfaces trunk    ! should be empty in this lab

vlan 10
 name Engineering
interface f3/1
 switchport mode access
 switchport access vlan 10
```

Packet Tracer notes from this lab:

- Some switch images **reject `do`**. Use `show` from privileged EXEC.
- `write` from `config-if` is invalid. `do write` on the router, or `end` then `write`.

## Exam checklist

- [ ] VLAN created **and** named before assigning ports
- [ ] PC access ports in the correct VLAN
- [ ] Router-facing switch ports in the **same** VLAN as that router interface
- [ ] PC mask and gateway match the VLAN subnet
- [ ] Same-VLAN ping works without a router
- [ ] Different-VLAN ping fails until the router is addressed and up
- [ ] Broadcast ping stays inside one VLAN
