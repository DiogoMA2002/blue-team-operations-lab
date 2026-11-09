---
title: Changelog
tags:
  - homelab
  - changelog
status: active
---

# Changelog

All notable documentation and infrastructure changes should be recorded here.

## 2026-11-09

### Changed

- Removed the two documentation chapters transferred to another project and
  cleaned their references from this vault.

- Re-scoped the vault as **HomeLab Core Infrastructure**, focused on Proxmox,
  home-LAN services, Pi-hole, Docker, media storage and resilience.
- Moved private-network expansion, Active Directory, security monitoring, VLANs,
  IDS/IPS and security exercises into the future **Enterprise Security Lab**.
- Simplified the active network documentation and retained OPNsense VM 102 as
  a preserved handoff baseline.

### Completed

- Preserved the OPNsense configuration backups outside the Ubuntu live session,
  detached the installation ISO, verified boot order and persistent boot, then
  exported the final configuration backup. VM 102 is powered off pending the
  Enterprise Security Lab.
- Generated Proxmox two-factor-authentication recovery keys and completed a
  successful extended SMART test for the 320 GB backup disk.

### Added

- Added an enabled weekly Proxmox backup job on `pve01`. Its initial
  test targeted `local`; selected backups were subsequently stored on
  `Backup-320`.
- Added a current HomeLab Core architecture diagram to the network
  documentation.

### Fixed

- Documented that the first manual backup execution nearly filled `local`;
  backup validation remains pending while the target, VM selection and retention
  are reviewed.
- Corrected the media-disk backup assumption: the task log showed VM 101's
  900 GiB `scsi1` disk was included despite earlier documentation saying
  it was excluded.
- Corrected the backup-disk record after inspection: the 320 GB Maxtor has GPT
  but no active filesystem or mounted Proxmox storage, so it cannot yet receive
  VZDump backups.

### Completed

- Provisioned the 320 GB Maxtor as Proxmox storage `Backup-320` and
  created backup archives for OPNsense VM 102 and Pi-hole CT 100.
- Restored the accidentally deleted Claudian workflow instructions and cleaned
  stale project-scope, OPNsense and Proxmox references.

### Related Notes

- [Project Overview](Docs/01_Project_Overview.md)
- [Network Architecture](Docs/03_Network_Architecture.md)
- [OPNsense](Docs/06_OPNsense.md)
- [TODO](TODO.md)
- [Project Roadmap](Docs/12_Project_Roadmap.md)

## 2026-11-06

### Added

- Recorded user confirmation of the live OPNsense configuration backup download;
  persistent backup storage and disk installation remain unconfirmed.
- Recorded initial OPNsense wizard completion and the explicit live-media
  warning; persistent installation and pre-install configuration preservation
  are now the next tasks. Updated deployment status and troubleshooting notes.
- Recorded the observed configuration of OPNsense VM 102 and UbuntuDesktop
  administration VM 103, WAN lease `192.168.1.233/23`, LAN `10.10.10.1/24`,
  and user-confirmed web access with temporary client address `10.10.10.200/24`.
- Added the unresolved LAN DHCP issue, manual-address workaround, lessons
  learned and remaining deployment checks to the relevant notes and task lists.

### Completed

- Configured the IPv4 LAN-to-home block above the general allow rule and
  observed matching ICMP traffic blocked in Live View. User also confirmed TCP
  blocking to Proxmox and continued firewall/public Internet access. IPv6 policy
  remains pending because the broad LAN IPv6 pass rule is still enabled.
- Recorded user-confirmed DHCP assignment and Internet access on UbuntuDesktop
  after Dnsmasq setup and returning the client to automatic IPv4/DNS. Updated
  network/VM status, resolved the DHCP incident and completed matching subtasks;
  exact lease details, explicit DNS/NAT checks and security validation remain pending.
- Installed OPNsense VM 102 to its 40 GiB virtual disk using ZFS stripe;
  post-install console confirmed `zroot/ROOT/default` and retained hostname
  and interface addresses. Updated tasks and resolved the live-mode issue;
  ISO detachment and security validation remain outstanding; DHCP was subsequently validated.
- Recorded user-confirmed creation and application of internal Linux bridge
  `vmbr1`; updated the network and Proxmox notes, TODO, roadmap and implementation
  log. OPNsense deployment is now in progress; firewall isolation tests remain pending.

### Changed

- Selected DHCP from the NOS router for the initial OPNsense WAN setup instead
  of the proposed static address; aligned the network, OPNsense and media plan
  notes and recorded the decision in the implementation log. WAN DHCP was
  subsequently observed working; full deployment validation remains pending.

### Fixed

- Corrected the Proxmox installation checkbox in TODO to match the completed
  implementation log entry and roadmap.
- Updated the Proxmox note to reflect the already completed Docker deployment,
  `media-docker` base setup and initial services, and mounted media disk.
- These are documentation corrections based on existing implementation records;
  those corrections alone did not represent new infrastructure changes.

### Related Notes

- [TODO](TODO.md)
- [Proxmox](Docs/04_Proxmox.md)
- [Implementation Log](Docs/09_Implementation_Log.md)
- [Project Roadmap](Docs/12_Project_Roadmap.md)
- [OPNsense](Docs/06_OPNsense.md)
- [Network Architecture](Docs/03_Network_Architecture.md)
- [Troubleshooting](Docs/10_Troubleshooting.md)
- [Lessons Learned](Docs/11_Lessons_learned.md)

## 2026-09-09

### Added

- Added the Media Stack + Tailscale Plan note.
- Added `media-docker` VM planning, Ubuntu Server setup decisions, and planned media services.
- Added media stack tasks for Portainer, Tailscale, and containers.
- Recorded completion of the initial `media-docker` Ubuntu Server and Docker setup.
- Recorded Portainer installation on `media-docker`.
- Recorded the 1 TB HDD configuration as Proxmox Directory storage `hdd-1tb-media`.
- Recorded mounting of the 900 GiB media disk at `/data` inside `media-docker`.
- Recorded Jellyfin deployment and initial library configuration.
- Recorded Prowlarr deployment.
- Recorded Sonarr and Radarr deployment.
- Recorded Lidarr deployment.
- Recorded Navidrome deployment.
- Recorded qBittorrent deployment behind Gluetun using VPNBook OpenVPN UDP 53 for testing.
- Recorded Prowlarr app integration with Sonarr, Radarr and Lidarr.
- Added troubleshooting note for failing public Prowlarr indexers such as 1337x.
- Recorded that VPNBook was the likely cause of qBittorrent not finding peers.
- Added troubleshooting note for completed qBittorrent downloads not appearing in Jellyfin.
- Recorded validation of Radarr imports, Jellyfin visibility and Sonarr manual import for Loki.
- Recorded that Lidarr music indexer troubleshooting is deferred because public music indexers appear limited.
- Added Bazarr as the next planned service for subtitle management.

### Changed

- Standardized firewall references as OPNsense and renamed the firewall note to `06_OPNsense.md`.
- Updated the planned network architecture so personal/media services run on the normal LAN through `vmbr0`.
- Clarified that OPNsense is dedicated to the isolated lab/pentest network.
- Updated Docker documentation with the planned `media-docker` host details.
- Deferred Tailscale installation for the media VM to a later phase.
- Marked Docker deployment and the `media-docker` Ubuntu Server setup as completed.
- Marked Portainer installation as completed.
- Documented that the `media-docker` media disk backup is disabled for now and may be enabled later.
- Documented that `df -h` shows about 840G available on `/data`, expected after formatting/filesystem overhead.
- Marked Jellyfin as running and configured.
- Marked Prowlarr as running.
- Marked Sonarr and Radarr as running.
- Marked Lidarr as running.
- Marked Navidrome as running.
- Marked qBittorrent behind VPN as running.
- Marked Prowlarr app integration as completed.

### Related Notes

- [Index](Index.md)
- [Network Architecture](Docs/03_Network_Architecture.md)
- [OPNsense](Docs/06_OPNsense.md)
- [Docker](Docs/08_Docker.md)
- [TODO](TODO.md)
- [Project Roadmap](Docs/12_Project_Roadmap.md)

## 2026-06-30

### Added

- Documented the completed BIOS configuration and Proxmox VE installation.
- Added the confirmed NOS LAN addressing and initial lab addressing plan.
- Added the Proxmox storage roles, administration decisions and weekly backup
  policy.
- Added an implementation log entry covering the initial Proxmox deployment.
- Documented the initial Pi-hole LXC container and its resource allocation.
- Added troubleshooting notes for the Pi-hole container startup failure.
- Added dedicated Pi-hole deployment and validation documentation.

### Changed

- Marked Proxmox VE as installed and the initial management and storage tasks as
  complete.
- Recorded `pve01.lab.internal` at `192.168.1.10/23`, using `192.168.1.1` as
  gateway and DNS.
- Recorded the formatted 320 GB HDD for weekly backups and the formatted 1 TB
  HDD for media, bulk data and optional VM data.
- Deferred `vmbr1` creation and two-factor authentication.
- Recorded that Pi-hole container `100` still fails to create its LXC network
  device after enabling nesting and assigning 2 CPU cores and 1024 MiB RAM.
- Identified from LXC debug output that `vmbr0` is not recognized as a Linux or
  Open vSwitch bridge.
- Removed the installer-created rule that pinned the physical NIC to `vmbr0`,
  restored its predictable name `enp3s0`, and created a functional Linux bridge
  at `vmbr0`.
- Marked Pi-hole deployment and DNS filtering as completed after a successful
  client test.

### Related Notes

- [Hardware Inventory](Docs/02_Hardware_Inventory.md)
- [Network Architecture](Docs/03_Network_Architecture.md)
- [Proxmox](Docs/04_Proxmox.md)
- [Pi-hole](Docs/05_Pi-hole.md)
- [Implementation Log](Docs/09_Implementation_Log.md)

## 2026-06-29

### Added

- Added reusable Claudian instructions for future chats, including vault workflow rules, changelog updates, index maintenance, task completion handling, link hygiene, and documentation quality checks.

### Changed

- Updated the vault index to include lessons learned, project roadmap, and Claudian instructions.
- Translated the Claudian instructions file to English because most conversations are expected to be in English.

### Related Notes

- [Index](Index.md)
- [Claudian Instructions](Claudian_Instructions.md)

## 2026-06-26

### Added

- Created initial Obsidian documentation structure.
- Added project overview, hardware inventory, network architecture, Proxmox, OPNsense, Active Directory, Docker, implementation log, troubleshooting, TODO, and changelog notes.

### Related Notes

- [Project Overview](Docs/01_Project_Overview.md)
- [Implementation Log](Docs/09_Implementation_Log.md)
- [TODO](TODO.md)
