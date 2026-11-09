---
title: Implementation Log
tags:
  - homelab
  - implementation-log
status: active
---

# Implementation Log

Use this note to record every major change, install, configuration decision, and test result.

## Log Template

```markdown
## YYYY-MM-DD - Change Title

### Change

- What was changed?

### Reason

- Why was it changed?

### Result

- What happened after the change?

### Issues

- Any errors or unexpected behavior?

### Related Notes

- [TODO](../TODO.md)
```

## Entries

## 2026-11-09 - Blue Team Operations Lab scope established

### Change

- Defined the 24-week A1–A6 roadmap for an isolated AD, SIEM and blue-team
  investigation lab.
- Reworked the vault workflow instructions around evidence, KQL, detection
  validation and sanitized portfolio cases.

### Result

- The inherited Proxmox and OPNsense configuration is now documented as the
  starting infrastructure for Phase A1 rather than a deferred future project.

### Related Notes

- [Project Overview](01_Project_Overview.md)
- [Project Roadmap](12_Project_Roadmap.md)
- [SIEM and Telemetry](13_SIEM_and_Telemetry.md)

## 2026-11-09 - Documentation scope cleanup

### Change

- Removed two documentation chapters transferred to another project.
- Repaired references after the remaining documentation chapters were
  renumbered.

### Result

- The HomeLab Core documentation no longer links to the transferred material.

### Related Notes

- [Index](../Index.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-09 - Backup creation confirmed and documentation reconciled

### Change

- Confirmed backup archives on `Backup-320` for OPNsense VM 102 and
  Pi-hole CT 100.
- Reconciled the active HomeLab Core documentation with the deferred Enterprise
  Security Lab scope and restored the accidentally deleted workflow note.

### Result

- Backup creation is working; restore testing and off-host backup remain
  outstanding.

### Related Notes

- [Proxmox](04_Proxmox.md)
- [TODO](../TODO.md)
- [Claudian Instructions](../Claudian_Instructions.md)

## 2026-11-09 - Proxmox access recovery and backup validation started

### Change

- Generated Proxmox two-factor-authentication recovery keys.
- Completed the extended SMART test for the 320 GB backup disk; the user
  reported that the result is healthy.
- Created an enabled backup job for `pve01`, scheduled weekly on Sunday
  at 01:00 with `keep-last=3`, and started a manual test.

### Result

- The backup job has not yet been validated; wait for the task result and
  inspect the resulting archive before marking it complete.
- The currently selected storage is `local`, which documentation defines
  as the Proxmox system SSD rather than the dedicated 320 GB backup disk.
- The first manual execution nearly filled `local`; the job must be
  corrected before a further backup test.
- The task log showed VM 101 included both its 100 GiB system disk and the
  900 GiB media disk on `scsi1`, creating a 1,000 GiB backup stream.

### Follow-up Result

- Provisioned the 320 GB Maxtor as storage `Backup-320`.
- Confirmed backup archives for OPNsense VM 102 and Pi-hole CT 100 are present
  on that storage. A restore test remains pending.

### Related Notes

- [Proxmox](04_Proxmox.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-09 - OPNsense baseline preserved for future project

### Change

- Moved the downloaded OPNsense configuration backup out of the Ubuntu live
  session and exported a current final configuration backup.
- Detached the OPNsense installation ISO, verified boot order and confirmed
  persistent boot and GUI access.
- Powered off VM 102 after completing the preservation steps.

### Reason

- Preserve the existing OPNsense private-LAN baseline without continuing its
  implementation in the HomeLab Core project.

### Result

- VM 102 and its configuration are ready to be reused by the future Enterprise
  Security Lab.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-09 - HomeLab Core scope separated from Enterprise Security Lab

### Change

- Re-scoped this vault as HomeLab Core Infrastructure.
- Simplified active documentation and tasks to Proxmox, home-LAN services,
  Pi-hole, Docker/media operations, storage and resilience.
- Marked the OPNsense private-LAN baseline, Active Directory and security-monitoring notes as
  deferred reference material for a separate Enterprise Security Lab.

### Reason

- Keep personal/home services separate from a future enterprise security
  training environment, with a clearer and more achievable scope for each
  portfolio project.

### Result

- Existing private-lab configuration is preserved without being expanded.
- The active roadmap now prioritizes OPNsense backup preservation, Proxmox
  protection, disk health and backups.

### Related Notes

- [Project Overview](01_Project_Overview.md)
- [Network Architecture](03_Network_Architecture.md)
- [OPNsense](06_OPNsense.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - Home-LAN IPv4 block active; ICMP test observed

### Change

- Added inbound quick IPv4 block on LAN from LAN network to `192.168.0.0/23`,
  any protocol/port, with logging; placed it before the general IPv4 permit.
- Advised leaving automatic rules unchanged and disabling the general IPv6
  permit for this IPv4-only phase. IPv6 rule disablement is not yet confirmed.

### Reason

- Stop lab clients from reaching the home network while retaining Internet
  access and administration through OPNsense LAN `10.10.10.1`.

### Result

- Rule-order screenshot confirmed the intended ordering.
- Live View showed an inbound LAN ICMP block labelled `Block LAN to HOME LAN`.
  Source/destination appeared as `10.10.10...` / `192.168...`; exact IPs were truncated.
- This proves an active matching rule, not full isolation or a successful TCP test.

### Follow-up Validation

- User later confirmed TCP access to Proxmox (`192.168.1.10:8006`) is blocked,
  while OPNsense LAN administration and public Internet access continue to work.
  No TCP log screenshot was supplied.
- The general LAN IPv6 pass rule remains enabled by user choice. In the current
  IPv4-only lab it has not been shown to carry traffic, but it must be disabled
  or replaced with equivalent IPv6 isolation before IPv6 is enabled.
- Management restrictions and effective-rule review remain pending. Do not clear
  all states unnecessarily.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Network Architecture](03_Network_Architecture.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - LAN DHCP and client Internet access validated

### Change

- Reviewed Dnsmasq settings: enabled on LAN, DNS port `53053`, DHCP local
  domain and automatic DHCP firewall rules enabled.
- Guided changing the initial pool `.41-.245` to `10.10.10.100-10.10.10.199`,
  setting `lab.internal`, disabling Router advertisements for the IPv4-only
  phase, applying settings and reconnecting Ubuntu with automatic IPv4/DNS.

### Reason

- Restore automatic client addressing and avoid overlap with planned static
  devices and the temporary administration IP `10.10.10.200`.

### Result

- User confirmed completion, receipt of an IP and Internet access from Ubuntu.
- DHCP incident marked resolved and matching TODO/roadmap subtasks completed.
- Exact lease/route and final saved settings have not been independently seen.
  The original DHCP failure cause was not isolated.

### Next Steps

- Record lease details, test DNS explicitly, review NAT/offloading and configure
  and test lab-to-home isolation and management restrictions.
- Full deployment remains in progress. Preserve a persistent backup; ISO
  detachment and boot-order review remain unconfirmed.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Network Architecture](03_Network_Architecture.md)
- [Troubleshooting](10_Troubleshooting.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - OPNsense installed to ZFS and booted from disk

### Change

- Ran `opnsense-installer` from the root shell and selected ZFS stripe on the
  sole QEMU disk `da0`, matching VM 102's 40 GiB SCSI disk on `local-lvm`.
- Installer progress confirmed cloning; the user then supplied the completed
  boot/login screen showing root filesystem `zroot/ROOT/default`.

### Reason

- Replace the temporary live session with a persistent disk installation.

### Result

- Disk installation and ZFS root boot confirmed. The console still shows
  `OPNsense.lab.internal`, LAN `10.10.10.1/24`, WAN DHCP `192.168.1.233/23`.
- Updated installation subtasks in TODO and roadmap; full deployment remains
  in progress because DHCP and firewall validation are outstanding.

### Issues and Next Steps

- No installer failure reported. A single-disk ZFS stripe provides no redundancy.
- Detach the installation ISO and review boot order, preserve the backup outside
  the Ubuntu live session, and verify web settings, DHCP, DNS and isolation.
- No successful DHCP lease or lab Internet test has yet been supplied.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Proxmox](04_Proxmox.md)
- [Troubleshooting](10_Troubleshooting.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - Live configuration backup downloaded

### Change and Result

- User confirmed downloading the OPNsense configuration backup before disk
  installation. The file has not been inspected and its storage location,
  encryption and restore validity are not confirmed.

### Next Steps

- Keep Ubuntu running if the backup is stored in its live session; preserve a
  copy on persistent storage. Run the OPNsense installer and verify the target
  virtual disk before any formatting. Installation is still pending.

### Related Notes

- [OPNsense](06_OPNsense.md)

## 2026-11-06 - Web wizard completed; live media mode confirmed

### Change

- Completed the initial OPNsense web wizard. General settings shown were
  `OPNsense.lab.internal`, `Europe/Lisbon`, English and WAN DNS override enabled.

### Result

- Final screenshot explicitly confirms live media mode and warns that reboot
  resets configuration. Wizard completion is not a persistent disk installation.
- Earlier installation ambiguity is resolved for the running session: it is live.

### Issues and Next Steps

- Do not reboot before preserving the configuration outside live-session storage.
- Export configuration, run the disk installer, verify disk boot and review or
  restore settings. Backup and installation completion are not yet confirmed.
- DHCP and isolation validation remain pending.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Troubleshooting](10_Troubleshooting.md)
- [TODO](../TODO.md)

## 2026-11-06 - OPNsense VM setup and initial web access

### Change

- Created VM 102 (`OpnSense`): 2 cores, 4 GiB fixed RAM, 40 GiB SCSI disk on
  `local-lvm`, i440fx / SeaBIOS and VirtIO SCSI single.
- Attached VirtIO WAN `net0` / `vtnet0` to `vmbr0` and LAN `net1` / `vtnet1`
  to `vmbr1`. The running console reports OPNsense 26.7 amd64.
- WAN obtained DHCP lease `192.168.1.233/23`; changed LAN from overlapping
  default `192.168.1.1/24` to `10.10.10.1/24`.
- Created VM 103 (`UbuntuDesktop`): 2 cores, 6 GiB RAM, 40 GiB SCSI disk on
  `local-lvm`, Ubuntu Desktop 26.04.1 media and one VirtIO NIC on `vmbr1`.
- Used a temporary manual IPv4 address `10.10.10.200/24` on Ubuntu, with
  gateway and DNS `10.10.10.1`, after automatic IPv4 assignment failed.

### Reason

- Establish the lab firewall and a browser-equipped internal administration
  client while leaving personal/media services on the home LAN.
- Restore access to the firewall UI without opening WAN administration or
  disabling firewalls, so the DHCP issue can be investigated.

### Result

- Hardware and interface addressing were confirmed from user screenshots.
- User confirmed access worked after the manual-IP workaround and the
  `https://10.10.10.1` browser test. Login/wizard completion is not confirmed.
- OPNsense deployment is in progress, not complete; updated partial tasks in
  TODO and roadmap and recorded the issue in Troubleshooting.

### Issues and Next Steps

- LAN DHCP did not supply an IPv4 lease to Ubuntu; root cause remains unknown.
- Installation-to-disk status needs explicit verification: the earlier answer
  was brief and both ISO images remain attached in the hardware screenshots.
  Do not infer persistent installation merely from the console menu.
- Verify web setup, virtual NIC offloading, DHCP, DNS, outbound NAT, Internet
  access, lab-to-home isolation and management restrictions before completion.
- Export a configuration backup after validation; return the administration
  client to DHCP once fixed. No successful ping or Internet test was supplied.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Proxmox](04_Proxmox.md)
- [Network Architecture](03_Network_Architecture.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Lessons Learned](11_Lessons_learned.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - Internal lab bridge created

### Change

- Created Linux bridge `vmbr1` and applied the network configuration on `pve01`,
  as confirmed by the user after the guided setup.
- Used no physical bridge ports, no host IP addresses or gateways, autostart
  enabled, VLAN awareness disabled and MTU 1500.

### Reason

- Prepare the internal virtual network for the OPNsense LAN without changing
  the existing `vmbr0` management and home LAN configuration.

### Result

- Bridge creation marked complete in TODO, roadmap and the Proxmox note.
- Next: create and install the OPNsense VM, then configure and test firewall
  isolation. Those steps are not yet complete.

### Issues

- No issues reported. Completion is based on user confirmation, not a live
  inspection or post-change connectivity test by the assistant.

### Related Notes

- [Proxmox](04_Proxmox.md)
- [Network Architecture](03_Network_Architecture.md)
- [OPNsense](06_OPNsense.md)
- [TODO](../TODO.md)
- [Project Roadmap](12_Project_Roadmap.md)

## 2026-11-06 - Initial OPNsense WAN DHCP decision

### Change

- Selected IPv4 DHCP client mode for the OPNsense WAN on `vmbr0`.
- Deferred the proposed static WAN address `192.168.1.11/23`.
- Kept the planned LAN gateway at `10.10.10.1/24` on internal bridge `vmbr1`.

### Reason

- Simplify initial deployment without assigning a static WAN address before
  confirming the NOS router DHCP pool and exclusions.

### Result

- Planning documentation aligned; no OPNsense installation or configuration
  has been confirmed yet. Record the WAN lease after installation.

### Issues

- The WAN address may change until a reservation or static address is configured.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Network Architecture](03_Network_Architecture.md)

## 2026-09-09 - Bazarr planned for subtitles

### Change

- Added Bazarr as the next planned media stack service.

### Reason

- Provide subtitle management for movies and TV shows handled by Radarr and
  Sonarr.

### Result

- Bazarr is planned as the next implementation step.

### Issues

- Not installed yet.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Lidarr troubleshooting deferred

### Change

- Decided to defer Lidarr music indexer troubleshooting to a later phase.
- Recorded that public music indexers appear limited/insufficient based on
  testing and user research.

### Reason

- qBittorrent connectivity from Lidarr was confirmed, but searches still found
  few or no useful music results.
- Public general-purpose indexers are not reliable enough for music automation.

### Result

- Lidarr remains installed and available, but music automation is not considered
  fully validated yet.

### Issues

- Better music-specific sources/indexers are needed before Lidarr can be
  properly validated.

### Related Notes

- [Docker](08_Docker.md)
- [Troubleshooting](10_Troubleshooting.md)
- [TODO](../TODO.md)

## 2026-09-09 - Media import path workflow validated

### Change

- Validated that Radarr imports completed movie downloads from
  `/data/torrents/complete/movies` into `/data/media/movies`.
- Confirmed Jellyfin can see imported movies after library scan.
- Manually imported Loki episodes in Sonarr after they appeared under Wanted
  rather than Activity.

### Reason

- Confirm the end-to-end workflow from qBittorrent to the media libraries and
  Jellyfin.

### Result

- Movie import and Jellyfin visibility are working.
- Sonarr manual import can resolve season/episode items that remain under
  Wanted.

### Issues

- Some Sonarr items may require manual import if Activity does not show a queue
  item after download completion.

### Related Notes

- [Docker](08_Docker.md)
- [Troubleshooting](10_Troubleshooting.md)

## 2026-09-09 - VPNBook identified as qBittorrent peer issue

### Change

- Tested qBittorrent without the VPNBook VPN path.

### Reason

- qBittorrent could not find peers while routed through VPNBook.

### Result

- qBittorrent started finding peers and completed some downloads without
  VPNBook.
- VPNBook is confirmed as unsuitable for this setup beyond basic testing.

### Issues

- Completed downloads are not yet appearing in Jellyfin, likely because the
  files are still in the torrent download path and have not been imported into
  the Jellyfin media library paths.

### Related Notes

- [Docker](08_Docker.md)
- [Troubleshooting](10_Troubleshooting.md)

## 2026-09-09 - Prowlarr apps and indexers configured

### Change

- Connected Prowlarr to Sonarr, Radarr and Lidarr.
- Added and tested public indexers in Prowlarr.

### Reason

- Centralize indexer management and synchronize working indexers to the media
  automation apps.

### Result

- Prowlarr app integration is working.
- Some indexers were added successfully.

### Issues

- Some public indexers, including 1337x, failed to connect with DNS/SSL-related
  errors and need later review.

### Related Notes

- [Docker](08_Docker.md)
- [Troubleshooting](10_Troubleshooting.md)
- [TODO](../TODO.md)

## 2026-09-09 - qBittorrent deployed behind Gluetun VPN

### Change

- Deployed Gluetun and qBittorrent on `media-docker` using Docker Compose.
- Configured qBittorrent to use Gluetun networking.
- Tested VPNBook OpenVPN using UDP port 53.

### Reason

- Ensure torrent traffic does not leave directly through the normal NOS WAN.
- Validate the intended downloader architecture before integrating with
  Sonarr, Radarr and Lidarr.

### Result

- Gluetun and qBittorrent are functional.
- VPNBook OpenVPN UDP 53 is working for testing.

### Issues

- VPNBook is considered temporary/test-only and should be replaced or reviewed
  before treating this as a final setup.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Navidrome installed

### Change

- Deployed Navidrome on `media-docker` using Docker Compose.
- Mounted `/data/media/music` into Navidrome as a read-only music library.

### Reason

- Add dedicated music streaming to the media stack.
- Keep Jellyfin available for general media while Navidrome handles music
  playback more cleanly.

### Result

- Navidrome is running on the LAN.
- The initial admin setup is ready or completed through the web interface.

### Issues

- No issue recorded.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Lidarr installed

### Change

- Deployed Lidarr on `media-docker` using Docker Compose.
- Mounted `/data` into the container for shared media and download paths.

### Reason

- Add music automation to the media stack.
- Prepare the music library for future integration with Prowlarr and the
  downloader.

### Result

- Lidarr is running on the LAN.
- The root folder can use `/data/media/music`.

### Issues

- Downloader integration is still pending.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Sonarr and Radarr installed

### Change

- Deployed Sonarr on `media-docker` using Docker Compose.
- Deployed Radarr on `media-docker` using Docker Compose.
- Mounted `/data` into both containers for shared media and download paths.

### Reason

- Add TV and movie automation services to the media stack.
- Prepare both services for future integration with Prowlarr and the torrent
  downloader.

### Result

- Sonarr and Radarr are running on the LAN.
- Root folders can use `/data/media/tv` and `/data/media/movies`.

### Issues

- Downloader integration is still pending.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Prowlarr installed

### Change

- Deployed Prowlarr on `media-docker` using Docker Compose.
- Confirmed the Docker user mapping as `PUID=1000` and `PGID=1000`.

### Reason

- Prepare centralized indexer management for Sonarr, Radarr and Lidarr.

### Result

- Prowlarr is running and ready for indexer/app integration.

### Issues

- No issue recorded.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Jellyfin installed and configured

### Change

- Deployed Jellyfin on `media-docker` using Docker Compose.
- Configured initial Jellyfin libraries for movies, music and shows.
- Mounted the media path from `/data/media` into the Jellyfin container.

### Reason

- Establish the main media server before adding automation services such as
  Prowlarr, Sonarr, Radarr and Lidarr.

### Result

- Jellyfin is running and accessible on the LAN.
- Initial libraries are visible in the Jellyfin interface.

### Issues

- No issue recorded.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Media data disk mounted in media-docker

### Change

- Added a 900 GiB second virtual disk from `hdd-1tb-media` to the
  `media-docker` VM.
- Formatted the disk as `ext4`.
- Mounted the disk inside Ubuntu at `/data`.
- Created the planned media and torrent directory structure under `/data`.

### Reason

- Use the dedicated 1 TB HDD-backed Proxmox storage for large media and torrent
  data.
- Keep the VM system disk focused on Ubuntu, Docker and container configuration.

### Result

- `/data` is mounted and ready for Jellyfin and the media stack.
- `df -h` reports about 840G available from the 900 GiB virtual disk, which is
  expected after filesystem formatting, reserved blocks and unit differences.

### Issues

- No issue recorded. The available capacity difference is expected.

### Related Notes

- [Docker](08_Docker.md)
- [Proxmox](04_Proxmox.md)

## 2026-09-09 - 1 TB media storage prepared for media-docker

### Change

- Configured the 1 TB WDC WD10EZEX HDD as Proxmox Directory storage named
  `hdd-1tb-media`.
- Prepared this storage to provide a second virtual disk to the `media-docker`
  VM for media data.
- Decided to keep backup disabled on the media disk for now.

### Reason

- Keep Ubuntu, Docker and configuration files on the SSD-backed system disk.
- Store large media and torrent data on the dedicated 1 TB HDD.
- Avoid oversized VM backups until a proper media backup strategy is defined.

### Result

- The 1 TB disk is ready to be attached to `media-docker` and mounted inside
  Ubuntu as `/data`.

### Issues

- Media disk backup is intentionally disabled for now and should be revisited
  later.

### Related Notes

- [Hardware Inventory](02_Hardware_Inventory.md)
- [Proxmox](04_Proxmox.md)
- [Docker](08_Docker.md)

## 2026-09-09 - Portainer installed on media-docker

### Change

- Installed Portainer CE on the `media-docker` VM using Docker Compose.
- Skipped Edge Compute during the initial Portainer setup.

### Reason

- Provide a web UI to manage local Docker containers, stacks, volumes and logs.
- Keep the setup focused on the local Docker environment rather than remote edge
  deployments.

### Result

- Portainer is running and ready to manage the local Docker environment.

### Issues

- None recorded.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - media-docker base setup completed

### Change

- Completed the initial Ubuntu Server setup for `media-docker`.
- Updated and upgraded the system.
- Installed base tools and `qemu-guest-agent`.
- Installed Docker and confirmed Docker Compose availability.
- Created the planned base directories under `/opt/docker` and `/data`.
- Deferred Tailscale installation to a later phase.

### Reason

- Prepare the VM as the primary Docker host for personal/media services.
- Keep the first implementation stage focused on the local LAN deployment before
  adding external access.

### Result

- `media-docker` is ready for Portainer installation and initial container
  stack deployment.

### Issues

- DHCP reservation for the planned `192.168.1.30` address still needs to be
  confirmed or configured on the NOS router.
- Portainer is not installed yet.

### Related Notes

- [Docker](08_Docker.md)
- [TODO](../TODO.md)

## 2026-09-09 - Media Docker VM planning and setup started

### Change

- Decided to run personal/media services on a dedicated VM named `media-docker`.
- Planned the VM on `vmbr0`, outside the OPNsense lab network.
- Chose Ubuntu Server as the guest OS.
- Planned Docker, Portainer, Tailscale and the media stack as containers on the VM.
- Deferred Tailscale installation to a later phase.
- Kept OPNsense dedicated to the isolated lab/pentest network.

### Reason

- Keep daily-use services easier to access from the home LAN and through
  Tailscale.
- Avoid unnecessary routing complexity through OPNsense for Jellyfin, Sonarr,
  Radarr, Lidarr, Navidrome and related services.
- Preserve OPNsense as a realistic firewall/router for pentest and enterprise
  lab simulation.

### Result

- VM creation settings and Ubuntu Server setup choices are documented in the
  Media Stack + Tailscale Plan.
- The vault now distinguishes between the LAN-based `media-docker` VM and any
  optional future Docker host inside the lab network.
- Current implementation focus is Ubuntu Server, Docker and Portainer first.

### Issues

- Ubuntu Server installation is in progress and not yet marked complete.
- DHCP reservation for `media-docker` still needs to be created on the NOS router.

### Related Notes

- [Docker](08_Docker.md)
- [Network Architecture](03_Network_Architecture.md)
- [TODO](../TODO.md)

## 2026-06-30 - Pi-hole LXC container initialized

### Change

- Created unprivileged LXC container `100` for Pi-hole.
- Assigned 2 CPU cores, 1024 MiB RAM and an 8 GiB root disk on `local-lvm`.
- Enabled the LXC nesting feature.

### Reason

- Prepare a lightweight Pi-hole deployment connected to the existing LAN before
  introducing OPNsense and the isolated lab network.

### Result

- The corrected bridge allowed container `100` to start.
- Pi-hole was installed with Cloudflare DNSSEC upstream resolution, query
  logging, the default blocklist and full query visibility.
- A client successfully used `192.168.1.20` for DNS resolution and filtering.

### Issues

- Startup still failed after increasing the allocation to 2 CPU cores and
  1024 MiB RAM and enabling nesting.
- Proxmox reported `Failed to create network device` and
  `Failed to create the network`.
- Debug output identified that `vmbr0` exists as an interface but is not
  recognized as a Linux or Open vSwitch bridge.
- The Proxmox installer had created
  `/usr/local/lib/systemd/network/50-pmx-vmbr0.link`, pinning the physical NIC
  to the reserved bridge-style name `vmbr0`.
- Removed the active pinning rule, regenerated the initramfs and rebooted.
- The physical NIC is now `enp3s0`; `vmbr0` is a Linux bridge using `enp3s0`
  as its physical port.

### Related Notes

- [Proxmox](04_Proxmox.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Pi-hole](05_Pi-hole.md)
- [TODO](../TODO.md)

## 2026-06-30 - Proxmox VE installed and initialized

### Change

- Configured the server BIOS for virtualization, UEFI boot, AHCI, XMP and future
  PCI passthrough.
- Installed Proxmox VE on the entire 500 GB WD_BLACK SN710 SSD using `ext4` and
  the default `LVM-thin` layout.
- Configured `pve01.lab.internal` with management address `192.168.1.10/23`,
  gateway `192.168.1.1` and DNS `192.168.1.1`.
- Confirmed `vmbr0` as the physical LAN bridge.
- Deferred creation of the isolated `vmbr1` bridge until OPNsense deployment.
- Formatted the 320 GB and 1 TB HDDs.
- Created a personal administrator account; two-factor authentication remains
  deferred.
- Assigned the 320 GB HDD to planned weekly backups and the 1 TB HDD to media,
  bulk data and optional low-performance VM data.

### Reason

- Establish the initial virtualization host while keeping the lab network and
  authentication setup simple during the first deployment phase.

### Result

- Proxmox VE is installed and its web interface is available at
  `https://192.168.1.10:8006`.
- The initial storage roles and weekly backup policy have been defined.

### Issues

- The NOS router DHCP pool boundaries still need to be confirmed.
- The 320 GB backup disk requires an extended SMART test before it is trusted.
- Important data will eventually require an additional external or off-host
  backup copy.

### Related Notes

- [Hardware Inventory](02_Hardware_Inventory.md)
- [Network Architecture](03_Network_Architecture.md)
- [Proxmox](04_Proxmox.md)

## 2026-06-26 - Vault structure created

### Change

- Created initial documentation structure for the Enterprise Cybersecurity Homelab vault.

### Related Notes

- [Project Overview](01_Project_Overview.md)
- [Hardware Inventory](02_Hardware_Inventory.md)
- [Network Architecture](03_Network_Architecture.md)
- [TODO](../TODO.md)
- [CHANGELOG](../CHANGELOG.md)
