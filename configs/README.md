# Configs

Saved running-configurations and reusable CLI snippets per device.

**Naming convention:** `labXX-<device>.txt` or `dayXX-<device>.txt` — e.g. `lab01-ASA1.txt`, `day15-R1.txt`.

| File | Lab |
|------|-----|
| [day15-R1.txt](day15-R1.txt) | Day 15 VLSM — R1 interfaces + static routes |
| [day15-R2.txt](day15-R2.txt) | Day 15 VLSM — R2 interfaces + static routes |
| [day16-R1.txt](day16-R1.txt) | Day 16 VLANs — R1 one interface per VLAN |
| [day16-SW1.txt](day16-SW1.txt) | Day 16 VLANs — SW1 access VLANs + router uplinks |
| [day17-R1.txt](day17-R1.txt) | Day 17 VLANs — R1 router-on-a-stick |
| [day17-SW1.txt](day17-SW1.txt) | Day 17 VLANs — SW1 access + trunk to SW2 |
| [day17-SW2.txt](day17-SW2.txt) | Day 17 VLANs — SW2 trunks to SW1 and R1 |
| [day18-R1.txt](day18-R1.txt) | Day 18 MLS — R1 /30 to SW2 + Internet |
| [day18-SW2.txt](day18-SW2.txt) | Day 18 MLS — SW2 SVIs, routed port, default route |
| [day20-stp.txt](day20-stp.txt) | Day 20 STP — priorities + show commands |
| [day21-stp.txt](day21-stp.txt) | Day 21 STP — roots, cost, PortFast, BPDU Guard |

Grab a config from a device with:

```
enable
show running-config
```

...then paste it into a `.txt` file here. Handy for quick reference and for reviewers who don't want to open Packet Tracer.
