# Configs

Saved running-configurations and reusable CLI snippets per device.

**Naming convention:** `labXX-<device>.txt` or `dayXX-<device>.txt` — e.g. `lab01-ASA1.txt`, `day15-R1.txt`.

| File | Lab |
|------|-----|
| [day15-R1.txt](day15-R1.txt) | Day 15 VLSM — R1 interfaces + static routes |
| [day15-R2.txt](day15-R2.txt) | Day 15 VLSM — R2 interfaces + static routes |
| [day16-R1.txt](day16-R1.txt) | Day 16 VLANs — R1 one interface per VLAN |
| [day16-SW1.txt](day16-SW1.txt) | Day 16 VLANs — SW1 access VLANs + router uplinks |

Grab a config from a device with:

```
enable
show running-config
```

...then paste it into a `.txt` file here. Handy for quick reference and for reviewers who don't want to open Packet Tracer.
