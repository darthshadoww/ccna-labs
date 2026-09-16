# Day 15 — VLSM (Variable Length Subnet Mask)

**Course:** Jeremy's IT Lab — CCNA 200-301
**Pairs with:** [labs/day-15-vlsm](../labs/day-15-vlsm/)

---

## FLSM vs VLSM

**FLSM (Fixed Length Subnet Mask)** — every subnet of a major network uses the **same** mask. Easy to design, expensive on addresses. A point-to-point link that only needs two hosts still burns the same block as a 60-host LAN.

**VLSM (Variable Length Subnet Mask)** — subnets of the **same** major network can use **different** masks. You size each subnet to its actual host count. This is how real networks use IPv4, and it is what IOS means when `show ip route` says a prefix is **variably subnetted**.

VLSM requires **classless** routing. The mask is part of the route; you cannot assume “`192.168.5.0` is always `/24`.”

## The two formulas

```
usable hosts = 2^n - 2
prefix length = 32 - n
```

`n` is the number of **host bits**. Subtract 2 because the all-zeros host part is the **network address** and the all-ones host part is the **broadcast address**.

| Prefix | Mask | Host bits | Usable hosts | Typical use |
|--------|------|----------:|-------------:|-------------|
| `/24` | `255.255.255.0` | 8 | 254 | small LAN (too big if you can carve it) |
| `/25` | `255.255.255.128` | 7 | 126 | ~64–126 hosts |
| `/26` | `255.255.255.192` | 6 | 62 | ~32–62 hosts |
| `/27` | `255.255.255.224` | 5 | 30 | ~16–30 hosts |
| `/28` | `255.255.255.240` | 4 | 14 | ~8–14 hosts |
| `/29` | `255.255.255.248` | 3 | 6 | tiny LAN |
| `/30` | `255.255.255.252` | 2 | 2 | Ethernet / serial point-to-point |
| `/32` | `255.255.255.255` | 0 | 1 | host route (IOS local `L` entries) |

Pick the **smallest** prefix (longest mask) that still has enough usable addresses. Example: 64 hosts → `/26` is only 62, so you must use `/25`.

## Design process (largest first)

1. List every LAN and the point-to-point links with their host counts.
2. Sort **largest → smallest**.
3. Starting at the major network address, assign the first block, then the next free address is the start of the next block.
4. Write down for each block: network, first usable, last usable, broadcast, mask.
5. Address hosts: this lab used **first usable = PC**, **last usable = router gateway**.

Worked example for this lab — major network `192.168.5.0/24`:

```
LAN2  64 hosts  → /25  192.168.5.0   – .127     PC .1    R1 .126
LAN1  45 hosts  → /26  192.168.5.128 – .191     PC .129  R1 .190
LAN3  14 hosts  → /28  192.168.5.192 – .207     PC .193  R2 .206
LAN4   9 hosts  → /28  192.168.5.208 – .223     PC .209  R2 .222
P2P    2 hosts  → /30  192.168.5.224 – .227     R1 .225  R2 .226
leftover               192.168.5.228 – .255
```

If this design had been FLSM `/25` everywhere, five subnets would need `5 × 128 = 640` addresses — more than one `/24`. VLSM is the only way to fit the topology in `192.168.5.0/24`.

## Binary checkpoints (so the blocks actually line up)

A network address must have **all host bits = 0**. That is why:

- `/25` jumps by 128: `.0`, `.128`
- `/26` jumps by 64: `.0`, `.64`, `.128`, `.192`
- `/28` jumps by 16: `.0`, `.16`, … `.192`, `.208`, `.224`
- `/30` jumps by 4: `.224`, `.228`, `.232`, …

`192.168.5.128/26` is legal (`128` is on a 64-boundary). `192.168.5.129/26` is **not** a network address — it is the first host.

IOS error if you get this wrong on a static route:

```
%Inconsistent address and mask
```

Fix: enter the **network** number, not a host IP.

## Routing with VLSM

Connected routes (`C`) appear automatically from `ip address` + `no shutdown`. Remote VLSM subnets still need static (or dynamic) routes, and the **mask in `ip route` must match the subnet you designed**:

```
ip route 192.168.5.192 255.255.255.240 192.168.5.226
ip route 192.168.5.0   255.255.255.128 192.168.5.225
```

A single `ip route 192.168.5.0 255.255.255.0 ...` would be a classful/summary `/24`. That can work as a **summary**, but only if every address in that `/24` really lives behind that next-hop. On R1 it would be wrong: R1 already owns part of `192.168.5.0/24` locally.

`show ip route` clues:

- `C` — connected (this router has an interface in that subnet)
- `L` — local `/32` of the router’s own interface IP
- `S` — static
- `variably subnetted, X subnets, Y masks` — VLSM is in effect

Next-hop must be a **directly connected neighbor**. Pointing a static route at your own interface IP (`192.168.5.225` on R1) does not reach the other router.

## Point-to-point links

Two routers on a link need two host addresses → `/30` (`2^2 - 2 = 2`). `/31` (RFC 3021) also exists in real IOS for P2P, but Packet Tracer labs and CCNA examples usually stick to `/30`.

## Exam / lab checklist

- [ ] Size each subnet from host count (`2^n - 2`), largest first
- [ ] Network, broadcast, first/last usable written down **before** configuring
- [ ] Mask on the PC matches the mask on the router interface
- [ ] Gateway = the router IP on **that** LAN, not a different subnet
- [ ] `ip route` uses the subnet’s network address + that subnet’s mask
- [ ] Next-hop = neighbor on the transit link
- [ ] `show ip route` shows `variably subnetted` and the expected `C` / `S` entries
- [ ] Repeat the first ping if one packet times out (ARP)

## Commands

```
show ip interface brief
show ip route
show running-config
ip route <network> <mask> <next-hop>
no ip route <network> <mask> <next-hop>
```
