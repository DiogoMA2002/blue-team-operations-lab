---
title: Pi-hole
tags:
  - homelab
  - dns
  - pihole
status: active
---

# Pi-hole

## Purpose

Pi-hole provides DNS filtering for the homelab and an initial platform for
observing DNS requests before OPNsense and the isolated lab network are deployed.

## Deployment

| Setting | Value |
|---|---|
| Platform | Unprivileged Proxmox LXC |
| CT ID | `100` |
| Address | `192.168.1.20/23` |
| Gateway | `192.168.1.1` |
| Bridge | `vmbr0` |
| CPU | 2 cores |
| Memory | 1024 MiB |
| Root disk | 8 GiB on `local-lvm` |
| Nesting | Enabled |

## DNS Configuration

- Upstream provider: Cloudflare with DNSSEC
- Query logging: enabled
- Blocklist: Pi-hole default list
- Privacy mode: show everything
- Administration interface: `http://192.168.1.20/admin`

## Validation

- The administration interface is reachable.
- A client was configured to use `192.168.1.20` as its DNS server.
- DNS resolution and filtering were tested successfully.

The DNS change was initially limited to one client. Network-wide DNS
distribution should only be enabled after confirming the NOS router DHCP scope
and planning the desired fallback behavior.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [Proxmox](04_Proxmox.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Project Roadmap](12_Project_Roadmap.md)
- [TODO](../TODO.md)
