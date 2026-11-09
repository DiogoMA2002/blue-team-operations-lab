---
title: Network Architecture
tags:
  - homelab
  - networking
  - architecture
status: active
---

# Network Architecture

## Current Architecture Diagram

![HomeLab Core Architecture](../001-Attachments/HomeLab-Core-Architecture.png)

## Scope

This note documents inherited HomeLab Core infrastructure and the target
isolated network for the active Blue Team Operations Lab.

## Physical Network

| Device | Role | Notes |
| --- | --- | --- |
| NOS Router | ISP router/default gateway | Current edge device |
| Unmanaged Switch | LAN switching | No VLAN support |
| PC1 / PC2 | Physical clients | Home-LAN clients |
| Homelab Server | Proxmox host | Runs core services and preserved lab VMs |

## Active Virtual Network

### `vmbr0` — Home LAN bridge

Physical bridge for Proxmox management, Pi-hole and `media-docker`. The OPNsense WAN is also connected here while VM 102 is retained for later reuse.

### Physical LAN — `192.168.0.0/23`

| Device / Service | Address | Assignment |
| --- | --- | --- |
| NOS Router | `192.168.1.1/23` | Existing gateway, DHCP and DNS |
| Proxmox (`pve01.lab.internal`) | `192.168.1.10/23` | Static |
| Pi-hole | `192.168.1.20/23` | Static |
| Media Docker VM | `192.168.1.30/23` | Planned reservation |
| OPNsense WAN | DHCP; `192.168.1.233/23` observed | Preserved future-lab VM |

Confirm the router DHCP-pool boundaries before creating further assignments or reservations.

## Preserved Future-Lab Network

### `vmbr1` — Reserved internal bridge

`vmbr1` has no physical ports, host IP address or gateway. It and OPNsense VM
102 form the starting point for the Blue Team Operations Lab.

### Target Blue Team Lab — `10.10.10.0/24`

| Role | Planned address | Purpose |
| --- | --- | --- |
| OPNsense LAN | `10.10.10.1` | Gateway, firewall and future log source |
| AD DS / DNS | `10.10.10.10` | Identity, DNS and directory telemetry |
| SIEM | `10.10.10.20` | Central log analysis and KQL workflow |
| Windows endpoint | DHCP or reserved address | Endpoint telemetry and controlled tests |

All target addresses require confirmation during Phase A1; this table is a
design baseline, not a record of deployed services.

## Core Network Objectives

- Keep Proxmox, Pi-hole and media services stable and private on the home LAN.
- Confirm stable addressing for `media-docker`.
- Improve backup and recovery before exposing services remotely.

## Future Project Boundary

The Blue Team Operations Lab will use the Proxmox host, `vmbr1` and OPNsense VM
102 for Active Directory, SIEM, telemetry, investigations and controlled tests.

## Related Notes

- [Proxmox](04_Proxmox.md)
- [Pi-hole](05_Pi-hole.md)
- [OPNsense](06_OPNsense.md)
- [Docker](08_Docker.md)
- [Active Directory](07_Active_Directory.md)
- [SIEM and Telemetry](13_SIEM_and_Telemetry.md)
- [Project Roadmap](12_Project_Roadmap.md)
- [TODO](../TODO.md)
