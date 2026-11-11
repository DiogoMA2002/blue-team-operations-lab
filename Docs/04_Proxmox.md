---
title: Proxmox
tags: [blue-team, proxmox, virtualization]
status: active
---

# Proxmox

## Purpose

`pve01.lab.internal` is the Blue Team Operations Lab hypervisor. It hosts
the isolated firewall, identity, SIEM and endpoint workloads.

## Confirmed Foundation

| Item | Value |
| --- | --- |
| Management address | `192.168.1.10/23` |
| Uplink bridge | `vmbr0` |
| Isolated lab bridge | `vmbr1`, no physical port or host address |
| Backup storage | `Backup-320` |
| OPNsense | VM 102; active gateway for `vmbr1` |
| DC01 baseline backup | Completed on 2026-11-10 to `Backup-320` |

## A1 Guest Plan

| Role | Planned network | Status |
| --- | --- | --- |
| OPNsense VM 102 | WAN `vmbr0`; LAN `vmbr1` | Active; LAN gateway `10.10.10.1` |
| AD DS / DNS (`DC01`) | `10.10.10.10` | Deployed; baseline backup completed |
| SIEM | `10.10.10.20` | Planned |
| Windows endpoint | `10.10.10.0/24` | Planned |

## Administration

- Recovery keys exist; configure TOTP or WebAuthn before production-like use.
- Restore testing and an off-host backup copy remain outstanding.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [OPNsense](05_OPNsense.md)
- [Project Roadmap](08_Project_Roadmap.md)


