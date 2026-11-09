---
title: Lessons Learned
tags:
  - homelab
  - lessons-learned
status: active
---

# Lessons Learned

## 2026-11-06 - Separate connectivity tests from DHCP validation

- An active virtual NIC does not prove the guest has a usable IPv4 address.
  Check the address and route before troubleshooting the browser or DNS.
- A temporary manual address outside the intended DHCP pool restored access to
  OPNsense. This was a workaround, not evidence that DHCP was fixed.
- Confirm virtual bridge and MAC mappings before changing firewall rules.
- Keep the lab administration client on `vmbr1` only; do not add a home-LAN NIC
  simply to work around a lab connectivity problem.
- LAN and WAN must use non-overlapping subnets; the default OPNsense LAN
  `192.168.1.1/24` had to change to `10.10.10.1/24` in this topology.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Troubleshooting](10_Troubleshooting.md)

## 2026-06-27

Using a single NIC means OPNsense cannot protect the physical network without changing the physical topology.

Therefore:

- Pi-hole was placed directly on vmbr0.
- OPNsense protects only the isolated lab.
- This simplifies migration to Phase 2.
