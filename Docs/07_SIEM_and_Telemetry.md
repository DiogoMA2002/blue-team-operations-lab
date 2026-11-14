---
title: SIEM and Telemetry
tags: [blue-team, siem, telemetry, kql]
status: in-progress
updated: 2026-11-14
---

# SIEM and Telemetry

## Purpose and Status

Document the current Elastic configuration and collection requirements.
The management plane and agent recovery are validated for one controlled reboot.
Controlled DC01 event ingestion remains unvalidated; Healthy is not proof of collection.

## Current Baseline

| Item | Recorded configuration |
| --- | --- |
| Platform | Elastic Security self-managed |
| Host | SIEM01, VM 105, Ubuntu Server, vmbr1 |
| Network | 10.10.10.20/24; gateway 10.10.10.1 |
| DNS | Persistent Netplan resolver 10.10.10.1; search domain lab.internal |
| Elasticsearch | Single-node blue-team-lab; node siem01; HTTPS/authentication enabled |
| Resources | Fixed 2 GiB heap (-Xms2g/-Xmx2g); persistent vm.max_map_count=1048576 |
| HTTP listener | Internal lab access on 10.10.10.20:9200; configured http.host: 0.0.0.0 |
| Kibana | Installed/enrolled; bound to 10.10.10.20:5601; persistent encryption keys |
| Fleet Server | Elastic Agent 9.5.6; fleet-server-policy; Fleet Server Policy revision 3 |
| Fleet integration | One Fleet Server integration, fleet_server-1 |
| Agent-facing Fleet URL | https://10.10.10.20:8220 |
| Local Fleet endpoint | https://127.0.0.1:8221, loopback only |
| Fleet data output | https://10.10.10.20:9200, managed through xpack.fleet.outputs in kibana.yml |

Do not revert Elasticsearch to loopback-only HTTP binding: remote DC01 needs
internal TCP 9200. Loopback can be used for SIEM01-local bootstrap connections,
but must not be the shared data output for remote agents.

Kibana access uses source-restricted OPNsense WAN TCP 5601 DNAT from the trusted
administration workstation to SIEM01. No WAN/NAT exposure is configured for
Elasticsearch 9200/9300 or Fleet 8220/8221. SIEM01 has no WAN-facing NIC.
Restricting listener access further with the host firewall remains a hardening task.

## DNS Resilience

SIEM01 uses OPNsense Unbound for external resolution. Query Forwarding sends
lab.internal to DC01 at 10.10.10.10:53. External package/registry resolution was
validated with DC01 off; AD names remain unavailable until DC01 is online.
DC01 itself continues to use its own AD DNS, forwarding external queries to OPNsense.

## DC01 Collection Policy

Policy: Windows Domain Controllers - Baseline, revision 3, default namespace.
Agent identity is displayed as DC01.lab.internal.

- System integration: Security, System and Application channels.
- Windows integration: Windows Security Telemetry - DC01 (recorded v3.13.1),
  collecting PowerShell, PowerShell Operational and Windows Defender Operational.
- Sysmon Operational is deferred until Sysmon is installed.
- Raw event.original preservation is disabled to conserve storage.

This is configured collection scope, not evidence that a controlled event has
arrived. AD auditing and local 4104/4728/4729 evidence are documented in
[Active Directory](06_Active_Directory.md).

## TLS, Secrets and Startup

Elasticsearch uses its generated HTTP CA and local authentication. Kibana has
persistent Encrypted Saved Objects, Reporting and Security encryption keys.
Passwords, service/enrollment tokens and encryption keys are not stored here.

Fleet retains the lab Quick Start/self-signed certificate arrangement. DC01 was
enrolled using the lab certificate exception; a lab-CA certificate covering the
internal IP and FQDN is still required for production-like verification. Earlier
inspection found SAN DNS:siem01 only; the certificate generated during recovery
has not been independently re-inspected. Do not infer its current SANs.

The recorded startup guard consists of /usr/local/sbin/wait-for-elasticsearch and
/etc/systemd/system/elastic-agent.service.d/10-wait-for-elasticsearch.conf. It
waits for the local Elasticsearch HTTPS authentication challenge and retries
service startup. It did not, by itself, resolve the later duplicate-input
readiness deadlock. Its exact script was not reviewed during this assessment.

## Recovery Validation - 2026-11-14

Duplicate integration cleanup followed by one controlled Fleet Server-mode
enrollment restored check-in. After a controlled reboot, SIEM01 was Healthy
Connected at 21:54:59 guest time with 4 minutes uptime. DC01 briefly reported EOF,
then recovered without reported manual intervention. The final Kibana screenshot
shows both active agents Healthy on revision 3 with recent activity.

See [Troubleshooting](10_Troubleshooting.md#fleet-server-check-in-blocked-after-siem01-reboot)
for the mechanism, evidence, exact inventory IDs and limitations.
The old Offline siem01 record is historical; its retirement is not yet confirmed.

## Remaining Validation and Capacity Work

1. Generate a harmless controlled DC01 event, verify it locally and find the
   corresponding event in Kibana using KQL within a recorded time window.
2. Confirm timestamps, source identity and usable evidence before marking ingestion validated.
3. Maintain a telemetry-source inventory and extend collection to the planned
   Windows client and OPNsense firewall logs.
4. Confirm time synchronization/timezone on SIEM01; retain Europe/Lisbon as the
   intended timezone, not an independently verified current setting.
5. Define retention only after data streams and ingestion volume are known.
   The earlier plan refers to a 100 GiB disk, but df showed a 48G root filesystem,
   16G used and 30G available during diagnosis. VM disk and filesystem/LV sizes
   are distinct and require confirmation. The previous 14-30 day retention goal
   is a planning target, not an applied or capacity-validated setting.

## Related Notes

- [Current Lab State](12_Current_Lab_State.md)
- [Network Architecture](03_Network_Architecture.md)
- [Active Directory](06_Active_Directory.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
- [Project Roadmap](08_Project_Roadmap.md)
