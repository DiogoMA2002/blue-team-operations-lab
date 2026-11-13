---
title: Current Lab State
tags:
  - blue-team
  - checkpoint
  - homelab
status: active
updated: 2026-11-13
---

# Current Lab State

## Continuation Checkpoint — 13 November 2026

This note records the verified state at the pause point so work can resume
without re-discovering configuration.

## Infrastructure

| Component | Verified state |
| --- | --- |
| Proxmox | `pve01.lab.internal`; 16 GiB physical RAM; current VMs require memory monitoring |
| OPNsense (VM 102) | LAN `10.10.10.1/24` on `vmbr1`; WAN DHCP address currently `192.168.1.233`; Unbound external resolution restored |
| DC01 (VM 104) | Windows Server domain controller for `lab.internal`; `10.10.10.10`; AD DS/DNS/GPO/auditing configured; baseline backup completed |
| SIEM01 (VM 105) | Ubuntu Server; `10.10.10.20`; Elasticsearch, Kibana, Elastic Agent/Fleet Server installed |

## Elastic Stack

- Elasticsearch: single-node `blue-team-lab`, local HTTPS validated, 2 GiB
  heap, persistent `vm.max_map_count=1048576`.
- Kibana: enrolled and running; access from the administration PC is restricted
  through OPNsense WAN TCP `5601` DNAT to `10.10.10.20:5601`.
- Fleet: initialized with persistent Kibana encryption keys; keys and tokens
  are not documented.
- Fleet Server: healthy on `siem01`, Elastic Agent `9.5.6`, internal endpoint
  `https://10.10.10.20:8220`. No WAN/NAT exposure for TCP `8220`.
- The current Fleet Server certificate is Quick Start/self-signed. Internal
  lab agents will require `--insecure` until replaced with a certificate from a
  lab CA.

## DC01 Telemetry Policy

The existing policy is `Windows Domain Controllers - Baseline`, revision 2:

- System integration (Security, System and Application logs).
- Windows integration `Windows Security Telemetry - DC01`:
  PowerShell, PowerShell Operational and Windows Defender Operational enabled.
- Sysmon Operational deliberately deferred until Sysmon is installed.
- Raw `event.original` preservation remains disabled to conserve SIEM disk.
- No agent is enrolled on DC01 yet.

## DNS and Connectivity Notes

- Normal intended chain: `SIEM01 -> DC01 (10.10.10.10) -> OPNsense (10.10.10.1) -> Cloudflare`.
- OPNsense has Cloudflare upstream resolvers `1.1.1.1` and `1.0.0.1`, WAN DNS
  override disabled, and Unbound restricted to the lab LAN.
- While DC01 is offline, SIEM01 needs its resolver set to `10.10.10.1` to
  reach Elastic Package Registry and Ubuntu repositories. This was used to
  restore Fleet setup; make it persistent only if operating SIEM01
  independently of DC01 is the intentional design.

## Immediate Next Step

In Kibana, use **Add agent** from `Windows Domain Controllers - Baseline`,
select Windows, and enroll the Elastic Agent on DC01. Confirm the resulting
agent is Healthy, then generate a controlled PowerShell event and verify it in
Kibana.

Do not share enrollment or service tokens. Do not expose Elasticsearch `9200`
or `9300`, Fleet Server `8220`, or the OPNsense management interface through
WAN.

## Related Notes

- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Active Directory](06_Active_Directory.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
