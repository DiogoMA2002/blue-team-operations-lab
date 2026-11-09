---
title: Proxmox
tags:
  - homelab
  - proxmox
  - virtualization
status: active
---

# Proxmox

## Purpose

Proxmox VE is the hypervisor for the HomeLab Core and hosts its home-LAN
services, plus preserved VMs for the future Enterprise Security Lab.

## Phase 1 Goals

- [x] Install Proxmox VE
- [x] Configure management access
- [x] Create virtual networking
- [x] Create storage layout
- [x] Preserve OPNsense baseline for future reuse
- [x] Deploy [Pi-hole](05_Pi-hole.md) and configure DNS filtering
- [x] Deploy [Docker](08_Docker.md) on `media-docker`
- [x] Document active core-guest resource allocation

## Virtual Machines

| VM | Purpose | Status | Related Note |
|---|---|---|---|
| OpnSense (VM 102) | Preserved firewall baseline | Installed on ZFS; backed up and powered off pending Enterprise Security Lab | [OPNsense](06_OPNsense.md) |
| UbuntuDesktop (VM 103) | Preserved lab administration client | Not active in the HomeLab Core scope | [OPNsense](06_OPNsense.md) |
| Pi-hole | DNS filtering | Running and validated | [Pi-hole](05_Pi-hole.md) |
| media-docker | Personal/media container services on LAN | Ubuntu Server and Docker installed; Portainer and initial media services deployed | [Docker](08_Docker.md) |

## Networking

- `vmbr0`: LAN bridge for Proxmox management and internet access
- `vmbr1`: internal lab bridge created and configuration applied on 2026-11-06,
  as confirmed by the user; no physical ports, host IP or gateway; autostart
  enabled, VLAN awareness disabled, MTU 1500. It is preserved for the future
  Enterprise Security Lab and is not active in the HomeLab Core scope.

## OPNsense and Administration Client Resources

Confirmed from hardware screenshots on 2026-11-06:

| VM | CPU | Memory | Disk | NICs |
|---|---|---|---|---|
| 102 - OpnSense | 1 socket / 2 cores, x86-64-v2-AES | 4 GiB, ballooning disabled | 40 GiB SCSI on `local-lvm` | VirtIO: `net0` on `vmbr0`, `net1` on `vmbr1` |
| 103 - UbuntuDesktop | 1 socket / 2 cores, x86-64-v2-AES | 6 GiB | 40 GiB SCSI on `local-lvm` | VirtIO: `net0` on `vmbr1`, NIC firewall enabled |

Both use i440fx, SeaBIOS and VirtIO SCSI single. OPNsense disk installation,
ZFS root boot, ISO detachment and boot order are confirmed.
Ubuntu permanent installation is not confirmed. These two VMs alone
allocate 10 GiB; account for other guests and host memory before starting more VMs.
Detailed networking and validation status: [OPNsense](06_OPNsense.md).

## Pi-hole Container

The initial unprivileged LXC container uses:

| Setting | Value |
|---|---|
| CT ID | `100` |
| CPU | 2 cores |
| Memory | 1024 MiB |
| Nesting | Enabled |
| Root disk | 8 GiB on `local-lvm` |

The container initially failed while creating its network device because the
physical NIC had been pinned to the name `vmbr0`, preventing `vmbr0` from being
a real Linux bridge. The pinning rule was removed, the physical NIC returned to
`enp3s0`, and `vmbr0` was recreated as a Linux bridge with `enp3s0` as its
port. The container subsequently started, Pi-hole was installed, and DNS
resolution and filtering were validated from a client.

## Installation Notes

### Installer Network Configuration

| Setting | Value |
|---|---|
| Hostname (FQDN) | `pve01.lab.internal` |
| Management IP | `192.168.1.10/23` |
| Gateway | `192.168.1.1` |
| DNS server | `192.168.1.1` |

The management address must be reserved or excluded from the NOS router DHCP
pool to prevent an address conflict.

### Installer Storage Configuration

| Setting | Value |
|---|---|
| Target disk | 500 GB WD_BLACK SN710 SSD |
| Filesystem | `ext4` |
| Volume management | Default `LVM-thin` layout |
| Disk allocation | Entire SSD |
| ZFS | Not used |

The installer creates the EFI, system and swap partitions automatically.
`local` is used for the Proxmox system, ISO images and templates, while
`local-lvm` stores VM and container disks.

The two HDDs were formatted after installation. Their planned roles are:

| Disk | Planned use |
|---|---|
| 320 GB Maxtor 6V320F0 | Proxmox storage `Backup-320`; selected core-guest backups |
| 1 TB WDC WD10EZEX-0 | Configured as Directory storage `hdd-1tb-media` for media data |

The backup disk passed its extended SMART test on 2026-11-09. A Proxmox disk
view on the same date showed `/dev/sda` (Maxtor 6V320F0, 320.07 GB) with
a GPT table but no active usage, filesystem or mounted directory storage. It
was provisioned as Proxmox storage `Backup-320`. It now contains backup
archives for OPNsense VM 102 and Pi-hole CT 100.
The initial backup policy is one snapshot-mode backup per week using
ZSTD compression, retaining the latest two or three copies as capacity permits.
This local disk is the first backup layer and should not become the only copy of
important data.

The extended SMART test for the 320 GB backup disk was completed successfully
on 2026-11-09. After the initial `local` capacity issue, storage
`Backup-320` was created and the successful backup output shows archives
for OPNsense VM 102 and Pi-hole CT 100. A restore into new, unused IDs remains
the required final validation.

## media-docker Storage

The 1 TB HDD is configured in Proxmox as Directory storage:

| Setting | Value |
|---|---|
| Disk | `/dev/sdb` |
| Filesystem | `ext4` |
| Storage ID | `hdd-1tb-media` |
| Purpose | Media data for `media-docker` |

A second SCSI disk of 900 GiB has been added to the `media-docker` VM from
`hdd-1tb-media` and is mounted at `/data` inside Ubuntu, with about 840 GiB
available after formatting/filesystem overhead, as recorded during setup.

Current backup decision for this media disk:

- Backup was documented as disabled, but the first manual Proxmox backup on
  2026-11-09 showed that it was included as `scsi1`, a 900 GiB disk.
- Exclude this disk from VM backups before retrying. It requires a separate
  media-data backup strategy and should not be assumed protected by the
  Proxmox VM-backup job.

### Initial Administration Decisions

- A personal administrator account was created for regular management.
- `root@pam` remains available for recovery.
- Proxmox two-factor-authentication recovery keys were generated on 2026-11-09.
  A TOTP or WebAuthn second factor still needs to be configured; recovery keys
  are for account recovery and are not the normal second factor.
- `vmbr1` was initially deferred; it was created on 2026-11-06 in preparation
  for the OPNsense deployment.
- The Proxmox web interface is available at `https://192.168.1.10:8006`.

## Related Notes

- [Hardware Inventory](02_Hardware_Inventory.md)
- [Network Architecture](03_Network_Architecture.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Project Roadmap](12_Project_Roadmap.md)
- [Pi-hole](05_Pi-hole.md)
