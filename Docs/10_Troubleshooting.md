---
title: Troubleshooting
tags:
  - homelab
  - troubleshooting
status: active
---

# Troubleshooting

Use this note to document recurring problems, fixes, commands, and lessons learned.

## Troubleshooting Template

```markdown
## Issue: Short Description

### Symptoms

- What was observed?

### Environment

- Host/VM:
- Service:
- Date:

### Investigation

- Commands run
- Logs checked
- Hypotheses tested

### Root Cause

- What caused the issue?

### Fix

- What solved it?

### Prevention

- How to avoid it next time?
```

## Known Issues

| Issue | Area | Status | Related Note |
|---|---|---|---|
| Manual Proxmox backup nearly filled `local` storage | Proxmox backup | Corrected: media disk excluded and selected backups stored on `Backup-320`; restore test pending | [Proxmox](04_Proxmox.md) |
| OPNsense wizard completed while still running from live media | OPNsense installation | Resolved by ZFS installation, ISO detachment and backup preservation | [OPNsense](06_OPNsense.md) |
| UbuntuDesktop receives no IPv4 DHCP lease on the lab LAN | OPNsense / Ubuntu / DHCP | Resolved: user confirmed DHCP and Internet; original cause not isolated | [OPNsense](06_OPNsense.md) |
| Pi-hole LXC failed to create its network device | Proxmox/LXC | Resolved | [Proxmox](04_Proxmox.md) |
| Single NIC limits physical OPNsense deployment | Networking | Known limitation | [OPNsense](06_OPNsense.md) |
| Unmanaged switch prevents VLANs | Networking | Known limitation | [Network Architecture](03_Network_Architecture.md) |
| Some Prowlarr public indexers fail to connect, including 1337x | Docker/Prowlarr | Under review | [Docker](08_Docker.md) |
| Sonarr/Radarr report qBittorrent download path `/downloads` missing inside containers | Docker/media stack | Under review | [Docker](08_Docker.md) |
| qBittorrent behind VPNBook finds no peers | Docker/qBittorrent/VPN | Root cause identified | [Docker](08_Docker.md) |
| Completed qBittorrent downloads do not appear in Jellyfin | Docker/media stack | Under review | [Docker](08_Docker.md) |
| Sonarr only downloads the first two episodes of a selected series | Docker/Sonarr | Under review | [Docker](08_Docker.md) |
| Lidarr finds few or no music results through public indexers | Docker/Lidarr/Prowlarr | Deferred | [Docker](08_Docker.md) |

## Issue: OPNsense wizard completed in live media mode

- Date: 2026-11-06; VM 102.
- The wizard completion page explicitly warns that OPNsense is in live media
  mode and that rebooting resets its configuration.
- A working console and web UI do not prove a persistent installation.
- Correction: user downloaded the backup and ran `opnsense-installer`, selecting
  ZFS stripe on `da0`. Subsequent boot shows root `zroot/ROOT/default`.
- Hostname and LAN/WAN addresses survived the installation and boot. No
  configuration loss was reported; remaining web settings still need review.
- The downloaded backup was preserved, the ISO detached and disk boot confirmed
  on 2026-11-09.

## Issue: UbuntuDesktop receives no IPv4 DHCP lease on the lab LAN

### Environment

- Date: 2026-11-06; Proxmox node `pve01`.
- OPNsense 26.7, VM 102; UbuntuDesktop, VM 103; internal bridge `vmbr1`.

### Symptoms

- Ubuntu wired connection remained at Connecting; Firefox could not connect
  to `10.10.10.1`.
- `ip a` showed `ens18` UP / LOWER_UP with only link-local IPv6 and no IPv4.
- Entering an HTTPS URL into the shell produced `No such file or directory`;
  this was a command-entry mistake, not an OPNsense error. URLs belong in the browser.

### Investigation

- Console screenshot confirmed LAN `vtnet1` at `10.10.10.1/24` and WAN `vtnet0`
  at DHCP address `192.168.1.233/23`.
- Hardware screenshots confirmed WAN `net0` on `vmbr0`, LAN `net1` on `vmbr1`,
  and Ubuntu's sole NIC on `vmbr1`; MAC addresses matched the guest interfaces.
- The Ubuntu NIC firewall flag was enabled; OPNsense NIC firewall flags were
  not enabled. No evidence identified the Proxmox firewall as the cause.

### Root Cause

- Original cause not isolated. The client initially lacked an IPv4 lease, but
  no packet capture or service log established the precise failure mechanism.

### Workaround and Result

- Set Ubuntu IPv4 manually to `10.10.10.200/24`, gateway and DNS `10.10.10.1`,
  then reconnect the wired profile. The address is outside the intended
  `10.10.10.100-10.10.10.199` DHCP pool.
- User confirmed access worked after this change and the browser test at
  `https://10.10.10.1`. No separate ping result was provided.
- This restores administration access only; DHCP, DNS and Internet access are
  not thereby proven functional. No firewall disabling was required.

### Resolution - 2026-11-06

- After wizard completion and disk installation, reviewed Dnsmasq: enabled on
  LAN, DNS port `53053`, DHCP local domain and automatic DHCP firewall rules enabled.
- The initial range shown was `.41-.245`. Guided adjustment to `.100-.199`
  avoids planned static addresses and the temporary `.200` workaround address.
- Instructed disabling Router advertisements for this IPv4-only phase, applying
  settings and reconnecting Ubuntu with automatic IPv4 and DNS.
- User confirmed completion, receipt of an IP and Internet access. Marked the
  incident functionally resolved; neither an exact lease nor final settings
  screenshots were provided. Do not attribute the original failure to the pool
  size or Router advertisements without further evidence.
- No firewall disabling was required. Explicit DNS tests, NAT rule review and
  firewall isolation remain separate validation tasks.

### Prevention

- Keep dynamic pools clear of static addresses and test leases with a client
  actually set to DHCP. A working manual IP is only a temporary workaround.
- Live-session client network changes may not survive a reboot.

### Related Notes

- [OPNsense](06_OPNsense.md)
- [Network Architecture](03_Network_Architecture.md)
- [Implementation Log](09_Implementation_Log.md)

## Issue: Pi-hole LXC failed to start

### Symptoms

- Container `100` failed during startup.
- Proxmox reported `Failed to spawn container "100"` and warned that the
  detected systemd version might require nesting.

### Environment

- Host: `pve01`
- Service: Proxmox LXC / planned Pi-hole container
- Date: 2026-06-30

### Root Cause

- The LXC debug output reported:
  `bridge 'vmbr0' is neither a linux nor an OVS bridge!`
- The Proxmox installer pinning rule
  `/usr/local/lib/systemd/network/50-pmx-vmbr0.link` assigned the physical NIC
  the name `vmbr0`.
- Consequently, `vmbr0` was a physical Ethernet interface rather than a Linux
  bridge, and the configuration attempted to use it as its own bridge port.

### Fix

- Enabled nesting.
- Increased the allocation to 2 CPU cores and 1024 MiB RAM.
- Backed up and removed the `50-pmx-vmbr0.link` pinning rule.
- Updated `/etc/network/interfaces` to use physical interface `enp3s0` as the
  port of bridge `vmbr0`.
- Regenerated the initramfs and rebooted the host.

### Result

- `enp3s0` and `vmbr0` now appear as separate interfaces.
- `vmbr0` is confirmed as a Linux bridge.
- Container `100` started and Pi-hole DNS filtering was successfully tested.

## Issue: Some Prowlarr public indexers fail to connect

### Symptoms

- Some Prowlarr indexers connect successfully.
- Other public indexers, including 1337x, fail during testing.
- Prowlarr reports DNS/SSL/connectivity-related errors.

### Environment

- Host/VM: `media-docker`
- Service: Prowlarr
- Date: 2026-09-09

### Investigation

- Confirmed that the Prowlarr app integrations with Sonarr, Radarr and Lidarr
  can be configured using full `http://IP:port` URLs.
- Confirmed that not all indexers fail, which suggests the Prowlarr service is
  generally functional.

### Root Cause

- Not confirmed yet.
- Likely causes include indexer-specific SSL/DNS issues, blocked public indexer
  domains, unstable public mirrors, or provider-side availability.

### Fix

- Use the working indexers for now.
- Review failing indexers later by testing DNS, SSL and alternative base URLs.

### Prevention

- Prefer reliable indexers.
- Avoid depending on a single public indexer.
- Keep Prowlarr and container images updated.

## Issue: Sonarr/Radarr cannot see qBittorrent `/downloads` path

### Symptoms

- Sonarr and Radarr health checks report that qBittorrent places downloads in
  `/downloads`, but that directory does not exist inside the Sonarr/Radarr
  containers.
- Downloads may not import correctly after completion.
- A test torrent/magnet can remain at `Downloading metadata`.

### Environment

- Host/VM: `media-docker`
- Services: qBittorrent, Gluetun, Sonarr, Radarr
- Date: 2026-09-09

### Investigation

- qBittorrent was configured with `/data/torrents` mounted as `/downloads`.
- Sonarr/Radarr were configured with `/data` mounted as `/data`.
- This means qBittorrent and the automation containers see the same host data
  under different container paths.

### Root Cause

- Path mismatch between qBittorrent and Sonarr/Radarr container volume mappings.
- The `Downloading metadata` state may additionally indicate no reachable peers,
  a weak public test VPN, indexer/magnet issues, or DHT/connection limitations.

### Fix

- Prefer consistent paths across containers:
  - mount `/data:/data` into qBittorrent;
  - set qBittorrent download paths to `/data/torrents/complete` and
    `/data/torrents/incomplete`;
  - keep Sonarr/Radarr/Lidarr using `/data` paths.
- If keeping qBittorrent's `/downloads` path, configure Remote Path Mapping in
  Sonarr/Radarr/Lidarr from `/downloads/complete` to `/data/torrents/complete`.

### Prevention

- Use one consistent path scheme across the media stack.
- Prefer `/data/...` inside every media/download container.

## Issue: qBittorrent behind VPNBook finds no peers

### Symptoms

- qBittorrent remains stuck at `Downloading metadata`.
- qBittorrent does not find peers.

### Environment

- Host/VM: `media-docker`
- Services: qBittorrent, Gluetun
- VPN: VPNBook OpenVPN UDP 53 test configuration
- Date: 2026-09-09

### Investigation

- Gluetun/OpenVPN can establish a VPN tunnel.
- Public IP through Gluetun was reported as a VPNBook endpoint.
- DNS resolution inside Gluetun works for tracker lookups.
- VPNBook UDP 53 established a tunnel, but qBittorrent did not find peers.
- VPNBook TCP 443 was tested afterwards, but OpenVPN failed during TLS
  negotiation with `TLS key negotiation failed` / `TLS handshake failed`.

### Root Cause

- VPNBook test VPN was the likely cause. After bypassing VPNBook, qBittorrent
  was able to find peers and complete downloads.

### Fix

- Test qBittorrent with a legal Linux ISO torrent to separate VPN/P2P problems
  from indexer/magnet problems.
- Try a different VPNBook profile, preferably TCP 443, if UDP 53 cannot find
  peers.
- If TCP 443 fails TLS negotiation, revert to the working UDP 53 profile or try
  another VPNBook server/profile.
- Consider replacing VPNBook with a more reliable VPN provider for the final
  setup.

### Prevention

- Treat VPNBook as test-only.
- Use a VPN provider that supports stable torrent traffic and, ideally, port
  forwarding or reliable P2P routing.

## Issue: Completed qBittorrent downloads do not appear in Jellyfin

### Symptoms

- qBittorrent successfully completes some downloads.
- Completed downloads do not appear in Jellyfin libraries.

### Environment

- Host/VM: `media-docker`
- Services: qBittorrent, Sonarr, Radarr, Lidarr, Jellyfin
- Date: 2026-09-09

### Investigation

- Jellyfin libraries are mounted from `/data/media`.
- qBittorrent downloads are expected under `/data/torrents`.
- Completed downloads do not appear in Jellyfin automatically unless they are
  downloaded directly into Jellyfin library folders or imported by
  Sonarr/Radarr/Lidarr.
- Confirmed case: a movie torrent completed in
  `/data/torrents/complete/movies/`, but it was not immediately visible in
  Jellyfin.

### Root Cause

- Not confirmed yet.
- Likely causes include completed files remaining in `/data/torrents/complete`,
  Sonarr/Radarr/Lidarr not importing them, wrong qBittorrent category, path
  mismatch, permissions, or Jellyfin library scan not yet refreshed.

### Fix

- Check Sonarr/Radarr/Lidarr Activity/Queue and import errors.
- Ensure root folders are:
  - Sonarr: `/data/media/tv`
  - Radarr: `/data/media/movies`
  - Lidarr: `/data/media/music`
- Ensure qBittorrent completed path is `/data/torrents/complete`.
- Ensure categories match the automation apps:
  - `tv`
  - `movies`
  - `music`
- Configure Remote Path Mapping only if qBittorrent reports paths differently
  from what the automation containers can see.
- Refresh/scan Jellyfin libraries after files are imported into `/data/media`.
- Current decision: standardize qBittorrent to use `/data` directly instead of
  relying on Remote Path Mapping.

### Prevention

- Keep one consistent `/data` path scheme across all media containers.
- Let Sonarr/Radarr/Lidarr import and organize files instead of expecting
  Jellyfin to scan torrent download folders directly.

## Issue: Sonarr only downloads the first two episodes of a selected series

### Symptoms

- A series is added to Sonarr.
- Sonarr only sends the first two episodes to qBittorrent.

### Environment

- Host/VM: `media-docker`
- Services: Sonarr, Prowlarr, qBittorrent
- Date: 2026-09-09

### Investigation

- Not completed yet.
- The Sonarr Activity queue did not show pending items, but the episodes were
  visible under Wanted.
- Manual import from Wanted resolved the immediate issue.

### Root Cause

- Not confirmed yet.
- Likely causes include series monitoring settings, only two episodes being
  monitored, search mode used during add, unavailable releases from current
  indexers, quality profile restrictions, or existing queue/history state.

### Fix

- Check that the whole series or season is monitored.
- Use manual/interactive search on missing episodes to confirm whether indexers
  return valid releases.
- Review quality profile, language profile and release rejection reasons.
- If Activity is empty but episodes remain Wanted, use Wanted/Manual Import to
  import completed season packs or files from the torrent download path.

### Prevention

- When adding a series, choose the correct monitor option for the intended
  behavior.
- Confirm missing episodes are monitored before triggering automatic search.

## Issue: Lidarr public music indexers are limited

### Symptoms

- Lidarr search finds few or no usable music results.
- qBittorrent download client test works from Lidarr.
- Some queued Lidarr items showed `Pending - Download client is unavailable`,
  but connectivity to qBittorrent was later confirmed.

### Environment

- Host/VM: `media-docker`
- Services: Lidarr, Prowlarr, qBittorrent
- Date: 2026-09-09

### Investigation

- Lidarr can reach qBittorrent.
- Public indexers appear to be weak or insufficient for music.
- User research also indicated that good public music indexers are limited.

### Root Cause

- Deferred.
- Current assumption: lack of reliable public music indexers rather than a core
  Lidarr/qBittorrent connectivity issue.

### Fix

- Defer Lidarr troubleshooting to a later phase.
- Revisit with better music-specific indexers, private indexers, Usenet, or
  alternative legal music library sources.

### Prevention

- Do not depend on public general-purpose torrent indexers for music automation.
- Validate music indexer support directly in Prowlarr before expecting Lidarr
  automation to work.

## Useful Commands

### Proxmox

```bash
# TODO: add Proxmox commands
```

### Linux

```bash
# TODO: add Linux commands
```

### Windows

```powershell
# TODO: add Windows commands
```
