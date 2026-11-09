---
title: Lessons Learned
tags: [blue-team, lessons-learned]
status: active
---

# Lessons Learned

## Inherited Network Constraint

- A single physical NIC and unmanaged switch prevent physical VLAN
  segmentation. The Blue Team lab therefore begins on the isolated virtual
  `vmbr1` bridge behind OPNsense.
- Lab and home-LAN subnets must not overlap; the lab baseline is
  `10.10.10.0/24`.

## Blue Team Principle

- A detection is not complete until its telemetry, test procedure, query,
  analyst evidence and remediation validation are documented.
