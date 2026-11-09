---
title: Docker
tags:
  - homelab
  - docker
  - linux
  - containers
status: in-progress
---

# Docker

## Purpose

Docker will host self-hosted infrastructure, monitoring, dashboarding, and utility services.

## Docker Host

| Item | Value |
|---|---|
| OS | Ubuntu Server |
| Hostname | `media-docker` |
| IP address | DHCP during install; planned reservation `192.168.1.30/23` |
| Network | `vmbr0` / LAN normal, outside the OPNsense lab network |
| Storage path | `/data` mounted from second VM disk on `hdd-1tb-media` |
| Docker compose path | `/opt/docker` |
| Backup path | TODO |
| Docker status | Installed |
| Base directories | Created |
| Media disk backup | Disabled for now; may be enabled later after backup strategy is defined |
| `/data` status | Mounted and ready |
| `/data` size | 900 GiB virtual disk; about 840 GiB available after formatting/filesystem overhead |

The primary Docker host is for personal/media services and should stay outside
the OPNsense lab network. OPNsense remains dedicated to the isolated
pentest/enterprise lab.

## Planned Containers

| Service | Purpose | Status |
|---|---|---|
| Homepage | Dashboard | Planned |
| Portainer | Docker management | Running |
| Grafana | Monitoring dashboards | Planned |
| Prometheus | Metrics collection | Planned |
| Uptime Kuma | Uptime monitoring | Planned |
| Vaultwarden | Password manager | Planned |
| Nextcloud | File sync/collaboration | Planned |
| Gitea | Git hosting | Planned |
| Nginx Proxy Manager | Reverse proxy | Planned |
| Jellyfin | Media server | Running and configured |
| Sonarr | TV automation | Running |
| Radarr | Movie automation | Running |
| Lidarr | Music automation | Running; public music indexer availability is limited and needs later troubleshooting |
| Prowlarr | Indexer management | Running; apps connected; some public indexers need review |
| Navidrome | Music streaming | Running |
| qBittorrent | Torrent client through Gluetun VPN test using VPNBook OpenVPN UDP 53 | Running |
| Tailscale | Private external access | Planned |
| Bazarr | Subtitles for Sonarr/Radarr media | Planned next |

See the service notes in this document for the`r`ncurrent media stack design and VM installation decisions.

## Jellyfin Transcoding

Current status:

```text
Software/direct-play Jellyfin setup: working
Hardware transcoding: not configured yet
Target acceleration option: Intel Quick Sync from the i7-8700 iGPU, if needed
```

Hardware transcoding is not required for direct play. It should be configured
later only if clients require transcoding or if CPU usage becomes too high.

## Future Improvements

- Reverse proxy
- SSL certificates
- Automated backups
- GitOps
- Infrastructure as Code

## Related Notes

- [Proxmox](04_Proxmox.md)
- [Network Architecture](03_Network_Architecture.md)
- [Implementation Log](09_Implementation_Log.md)
- [Project Roadmap](12_Project_Roadmap.md)
