---
title: Lab Hardware Inventory
tags: [blue-team, hardware]
status: active
---

# Lab Hardware Inventory

| Component | Detail | Blue Team role |
| --- | --- | --- |
| Proxmox host | Intel Core i7-8700, 16 GB RAM | Runs firewall, AD, SIEM and endpoints |
| System storage | 500 GB WD_BLACK SN710 SSD | Proxmox and guest disks |
| Backup storage | 320 GB Maxtor, `Backup-320` | Selected VM/CT backups |
| Network | One Gigabit NIC, unmanaged switch | Lab uplink; physical VLANs unavailable |

## Constraints

- The isolated enterprise network uses the virtual `vmbr1` bridge.
- Managed switching, additional NICs and physical segmentation are future
  improvements, not A1 dependencies.

## Related Notes

- [Proxmox](04_Proxmox.md)
- [Network Architecture](03_Network_Architecture.md)
