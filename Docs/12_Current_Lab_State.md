---
title: Current Lab State
tags: [blue-team, checkpoint, homelab]
status: active
updated: 2026-11-14
---

# Current Lab State

## Purpose and Scope

Single current continuation checkpoint for the isolated lab. Historical changes
belong in the [Implementation Log](09_Implementation_Log.md); incident detail
belongs in [Troubleshooting](10_Troubleshooting.md).

## Infrastructure

| Component | Recorded current state |
| --- | --- |
| Proxmox | pve01.lab.internal; 192.168.1.10/23; 16 GiB physical RAM; monitor capacity |
| OPNsense VM 102 | Lab gateway 10.10.10.1/24 on vmbr1; WAN DHCP, 192.168.1.233 last recorded |
| DC01 VM 104 | lab.internal domain controller/DNS; 10.10.10.10; AD/GPO/auditing configured |
| SIEM01 VM 105 | Ubuntu Server; 10.10.10.20; Elasticsearch, Kibana and Fleet Server operational |
| Windows client | Not deployed |
| Backup | DC01 baseline on Backup-320 completed; restore test and off-host copy pending |

## Network and Service Baseline

- vmbr1 has no physical ports, Proxmox host address or gateway.
- SIEM01 persistent DNS: OPNsense 10.10.10.1, lab.internal search domain.
  Unbound forwards lab.internal queries to DC01; external resolution works
  independently of DC01. DC01 uses its own AD DNS with OPNsense as forwarder.
- Fleet URL: https://10.10.10.20:8220. TCP 8221 is local Fleet communication.
- Shared Elasticsearch data output: https://10.10.10.20:9200, not remote loopback.
- Kibana TCP 5601 access is restricted through OPNsense to the administration
  workstation. No WAN/NAT exposure for 9200/9300 or 8220/8221.
- Elastic Agent version 9.5.6; Elasticsearch single node, 2 GiB heap and persistent
  vm.max_map_count=1048576. Fleet/Kibana persistent keys are configured; no secrets here.
- Fleet still uses lab self-signed certificate handling. CA-issued verification
  and tighter host-firewall restrictions remain hardening tasks.
- A systemd Elasticsearch readiness guard is recorded on SIEM01; see
  [SIEM and Telemetry](07_SIEM_and_Telemetry.md) for paths and limitations.

## Latest Validated Outcome - 2026-11-14

- Fleet Server Policy revision 3 contains only fleet_server-1.
- DC01 uses Windows Domain Controllers - Baseline revision 3: System plus Windows
  PowerShell/Defender channels; Sysmon deferred and raw event.original disabled.
- After duplicate-input cleanup and one controlled enrollment, a SIEM01 reboot
  returned local Fleet HEALTHY Connected / Agent HEALTHY Running at guest time
  21:54:59, with 4 minutes uptime.
- DC01's transient EOF cleared on a later status check without a reported manual
  service restart. The final Kibana screenshot confirms both active agents
  Healthy, revision 3, with activity 30 seconds ago (siem01) and 22 seconds ago
  (DC01). These are point-in-time observations, not live monitoring.
- This validates recovery for one observed reboot cycle. A controlled DC01 event
  arriving in Kibana has NOT yet been demonstrated.

## Historical Inventory Still Present

The last screenshot still contained the old siem01 record Offline on revision 2.
Retirement instructions were provided, but completion has not been confirmed.
Match Agent IDs in [Troubleshooting](10_Troubleshooting.md#fleet-server-check-in-blocked-after-siem01-reboot),
not hostname alone. Do not uninstall the active agent or remove its policy.

## Next Operational Step

Generate a harmless controlled PowerShell event on DC01; record its time and
local event evidence, then find the matching event in Kibana using KQL.
Do not claim ingestion, a detection or a completed investigation from agent health alone.

Other open work: retire only the historical inventory record after identity
verification, validate time synchronization, confirm disk/filesystem capacity
before sizing retention, and complete the remaining A1 deliverables in [TODO](../TODO.md).

## Related Notes

- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Active Directory](06_Active_Directory.md)
- [Network Architecture](03_Network_Architecture.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Documentation Review](13_Documentation_Review.md)
