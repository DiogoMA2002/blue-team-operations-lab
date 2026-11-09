---
title: OPNsense
tags: [blue-team, firewall, networking]
status: planned
---

# OPNsense

## Purpose

OPNsense VM 102 is the gateway and firewall-log source for the isolated Blue
Team Operations Lab.

## Confirmed Baseline

| Interface | Network | Status |
| --- | --- | --- |
| WAN / `vtnet0` | `vmbr0`, DHCP from home router | Previously validated |
| LAN / `vtnet1` | `vmbr1`, `10.10.10.1/24` | Preserved configuration |

The VM is installed on ZFS, has a preserved configuration backup and was
powered off after initial validation.

## A1 Actions

- Power on VM 102 and verify persistent boot.
- Confirm WAN, LAN and DHCP operation.
- Confirm lab-to-home isolation while retaining required Internet egress.
- Configure firewall-log forwarding or collection for the selected SIEM.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
