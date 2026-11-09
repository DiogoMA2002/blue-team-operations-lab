---
title: OPNsense
tags:
  - homelab
  - OPNsense
  - firewall
  - networking
status: deferred
---

# OPNsense

> [!note] Project boundary
> This VM and its documented private-LAN baseline are preserved for the separate **Enterprise Security Lab** project. Backup preservation, ISO removal and boot-order verification were completed on 2026-11-09. VM 102 is retained powered off until that project begins.

## Purpose

OPNsense provides firewalling, NAT, DHCP, DNS forwarding, and future VPN functionality for the virtual lab network.

## Initial Scope

Because the current server has one Gigabit Ethernet interface and the switch is unmanaged, OPNsense will initially protect only the virtual lab network.

## Planned Services

- Firewall
- NAT
- DHCP for lab network
- DNS forwarding
- VPN in a future phase
- IDS/IPS in a future phase

## Interfaces

| Interface | Purpose | Notes |
|---|---|---|
| WAN | Upstream access through Proxmox/LAN | `net0` / `vtnet0` on `vmbr0`; DHCP lease `192.168.1.233/23` observed on 2026-11-06 |
| LAN | Internal lab network | `net1` / `vtnet1` on `vmbr1`; configured static gateway `10.10.10.1/24` |

### Initial WAN Addressing Decision

On 2026-11-06, DHCP was selected for the initial WAN setup instead of the
previously proposed static address `192.168.1.11/23`. The console subsequently
confirmed lease `192.168.1.233/23`; this address may change. A reservation or
static address can be considered later. LAN DHCP is intended to serve
`10.10.10.100-10.10.10.199`; the user has now confirmed automatic IP assignment
and Internet access from Ubuntu after the DHCP configuration steps below.

## Deployment Progress - 2026-11-06

### Confirmed VM Configuration

| Setting | Observed value |
|---|---|
| Node / VM ID / name | `pve01` / `102` / `OpnSense` |
| Running version | OPNsense 26.7 amd64 |
| CPU | 1 socket, 2 cores; `x86-64-v2-AES` |
| Memory | 4096 MiB; ballooning disabled |
| Machine / BIOS | i440fx / SeaBIOS |
| Disk | SCSI, 40 GiB on `local-lvm`, IO thread enabled |
| SCSI controller | VirtIO SCSI single |
| WAN NIC | VirtIO, MAC `BC:24:11:47:60:70`, `vmbr0` |
| LAN NIC | VirtIO, MAC `BC:24:11:51:33:C4`, `vmbr1` |
| Proxmox NIC firewall flag | Not enabled on either OPNsense NIC |
| Installation media | ISO detached after persistent boot validation on 2026-11-09 |

The console initially showed LAN `192.168.1.1/24`, overlapping the home WAN
subnet. It was changed to `10.10.10.1/24`, confirmed by a later console screenshot.
No LAGGs, VLANs or optional interfaces were assigned during console setup.

### Administration Client and Initial Access

- VM `103` (`UbuntuDesktop`) was created on `pve01`: 2 cores / 1 socket,
  6 GiB RAM, i440fx / SeaBIOS, VirtIO SCSI single, 40 GiB disk on `local-lvm`.
- Ubuntu Desktop 26.04.1 amd64 media is mounted. It was intended as a live
  administration client; a permanent Ubuntu installation is not confirmed.
- Its only NIC is VirtIO on `vmbr1`, MAC `BC:24:11:AB:94:1A`; the Proxmox NIC
  firewall flag remains enabled. Guest interface: `ens18`.
- Ubuntu initially had an active link but no IPv4 lease, only link-local IPv6.
- Temporary manual configuration: `10.10.10.200/24`, gateway and DNS
  `10.10.10.1`. This address is outside the intended DHCP pool.
- The user confirmed access worked after the manual address change, following
  the test of `https://10.10.10.1`. Subsequent screenshots confirm authenticated
  wizard access and completion, but not Internet access. No firewall disabling
  was needed for this workaround.

### Wizard Completed in Live Media Mode

The initial wizard completion screenshot reported **live media mode** and warned
that rebooting would reset configuration. Disk installation was therefore
performed afterwards, as recorded below; wizard completion alone was not installation.

- General settings screenshot: hostname `OPNsense`, domain `lab.internal`,
  timezone `Europe/Lisbon`, English, blank DNS list and WAN DNS override enabled.
- Guidance for the remaining wizard steps: keep Unbound enabled, defer DNSSEC;
  WAN DHCP with private-network blocking unchecked and bogon blocking enabled;
  LAN `10.10.10.1/24` with DHCP configuration enabled; disable Multiwan/IPsec
  optimization and retain automatic DHCP/DNS registration. Final saved values
  for these later choices still require review after installation.
- The next action was to export the live configuration before running the installer.
- The user subsequently confirmed downloading the configuration backup.
  A copy on persistent storage outside the Ubuntu live session is not yet
  confirmed. Disk installation subsequently completed as described below.

### Disk Installation and First Boot Confirmed

- Started `opnsense-installer` from the root console shell.
- Selected ZFS, single-disk `stripe` (no redundancy), and the only listed
  QEMU disk `da0`, matching the VM's 40 GiB SCSI disk.
- Installer progress showed cloning the current system to disk.
- The subsequent boot screenshot and user confirmation show root filesystem
  `zroot/ROOT/default` and a login prompt: OPNsense is now booted on ZFS,
  rather than the previous live-media root.
- The displayed hostname remains `OPNsense.lab.internal`, LAN remains
  `10.10.10.1/24`, and WAN DHCP is still `192.168.1.233/23`.
- ISO detachment and boot-order review were completed on 2026-11-09. Subsequent
  web screenshots confirm post-install UI access; further validation is deferred.

### DHCP and Internet Access Confirmed

- Dnsmasq screenshots showed Enable checked, interface LAN, DNS listen port
  `53053`, DHCP local domain and DHCP register firewall rules enabled.
- The initial pool shown was `10.10.10.41-10.10.10.245`, overlapping planned
  static addresses and the temporary administration address `.200`.
- Guided changes: use pool `10.10.10.100-10.10.10.199`, domain `lab.internal`,
  disable Router advertisements for this IPv4-only phase, apply settings, and
  return Ubuntu IPv4 and DNS to automatic mode before reconnecting.
- User subsequently confirmed completion, an assigned IP and Internet access.
  The exact lease address, route and final saved-setting screenshots were not
  supplied; do not retain `.200` as the client's current address.
- The original DHCP symptom is functionally resolved. Its precise original
  cause was not isolated; no firewall disabling was required.
- Dedicated DNS queries and NAT rule inspection remain to be documented;
  lab-to-home IPv4 blocking was subsequently configured and partially tested below.

### Handoff Actions

- Preserved the downloaded configuration backup outside the Ubuntu live session.
- Detached the OPNsense ISO, verified disk boot order and confirmed persistent
  boot and GUI access.
- Exported a current validated configuration backup for future restoration.
- Powered off VM 102 after preservation; no further private-LAN configuration
  is active in the HomeLab Core project.
- Review the wizard settings after installation, including private-WAN settings.
- Record the actual DHCP lease and validate DNS explicitly; review outbound NAT
  rules. Basic Internet access is already confirmed by the user.
- Verify hardware offloading settings for the virtualized firewall.
- TCP blocking to Proxmox port 8006, continued OPNsense LAN administration and
  public Internet access were subsequently confirmed by the user. Disable the
  broad LAN IPv6 permit or implement an explicit IPv6 policy before enabling
  IPv6 services; IPv4 isolation is validated, IPv6 isolation is not.
- Export a configuration backup after the initial configuration is validated.

See [Troubleshooting](10_Troubleshooting.md) for the DHCP incident and resolution.

## Rules

User-defined LAN rules observed on 2026-11-06, in order:

| Rule | Source | Destination | Port/Protocol | Action | Reason |
|---|---|---|---|---|---|
| Block LAN to HOME LAN | LAN network | `192.168.0.0/23` | IPv4 / any | Block, quick, inbound, logged | Protect the home LAN from lab clients |
| General IPv4 permit | LAN network | any | IPv4 / any | Pass, quick, inbound | Initial outbound connectivity; follows the home-LAN block |
| General IPv6 permit | LAN network | any | IPv6 / any | Pass; user confirmed it remains enabled | Broad permission; deferred IPv6 policy risk |

Automatic rules were left unchanged in the guidance. The displayed rule list
confirmed the IPv4 block above the general allow rule. Live View subsequently
showed an inbound LAN ICMP packet with source `10.10.10...` and destination
`192.168...` blocked by `Block LAN to HOME LAN`, proving that this rule is active
and matches the tested traffic. Full addresses were truncated in the screenshots.

The logged ICMP block is direct evidence of the IPv4 rule matching. The user
subsequently confirmed TCP access to Proxmox port 8006 is blocked while
`https://10.10.10.1` and public Internet access remain functional; exact TCP
log evidence was not supplied. Other WAN multicast UDP blocks shown belong to
default rules. Green WAN entries labelled `let out anything from firewall host
itself` do not represent client traffic.

The client previously showed only link-local IPv6 and Router Advertisements were
disabled for this IPv4-only phase. The enabled broad IPv6 pass rule therefore has
not been demonstrated as an active bypass, but it is unsafe to leave in place if
IPv6 is later enabled without equivalent IPv6 isolation rules.

## Future Improvements

- Become main firewall in Phase 2
- Protect whole home network
- VLAN segmentation
- VPN access
- IDS/IPS integration

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [Proxmox](04_Proxmox.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Implementation Log](09_Implementation_Log.md)
- [TODO](../TODO.md)
