---
title: SIEM and Telemetry
tags:
  - blue-team
  - siem
  - telemetry
  - kql
status: in-progress
---

# SIEM and Telemetry

## Purpose

Provide centralized, time-aligned telemetry for blue-team detection and
investigation. The selected SIEM must support KQL for the planned query
workflow.

## A1 Deployment Status

| Item | Status |
| --- | --- |
| Platform | Elastic Security self-managed — selected |
| VM | `SIEM01` / VM 105 on `vmbr1` |
| Network | Static `10.10.10.20/24`; gateway `10.10.10.1`; intended DNS `10.10.10.10` |
| Operating system | Ubuntu Server installed and updated |
| Elasticsearch | Installed, enabled and validated locally over HTTPS |
| Elasticsearch resources | JVM heap capped at 2 GiB; `vm.max_map_count=1048576` configured persistently |
| DNS resolution | Resolved; intended internal DNS chain validated |
| Temporary workaround | Used for package update; revert it with `resolvectl revert ens18` if still active |

Do not install Elastic until the operating system has been updated. Restore the
intended internal DNS chain after the OPNsense resolver remediation.
## Elasticsearch Baseline

Elasticsearch is installed on `SIEM01` as a single-node `blue-team-lab`
cluster. It runs as a systemd service and is locally validated over HTTPS using
the generated HTTP CA certificate. The built-in `elastic` account password was
set interactively and is not recorded in this vault.

Kibana, Fleet Server, Elastic Agent enrollment and source telemetry remain
pending.
## Elasticsearch Base Configuration

### Implemented

- Single-node cluster: `blue-team-lab`; node: `siem01`.
- Elastic security auto-configuration, generated HTTP CA certificate and local
  authentication enabled.
- Elasticsearch systemd service enabled and HTTPS endpoint validated locally.
- Persistent `vm.max_map_count=1048576`.
- JVM heap fixed at `2 GiB` (`-Xms2g`, `-Xmx2g`) to preserve host memory.

### Required before Kibana

- Confirm NTP synchronization and `Europe/Lisbon` timezone on SIEM01.
- Confirm whether `http.host` is `0.0.0.0`; if so, bind it to `127.0.0.1` so
  Elasticsearch remains local to SIEM01. Kibana and Fleet will use local access.
- Do not expose Elasticsearch TCP ports `9200` or `9300` through OPNsense.

### Deferred

- Configure data-stream lifecycle/retention only after source integrations create
  data streams; target retention is 14–30 days for the 100 GiB lab disk.
## Required Sources

| Source | Minimum evidence |
| --- | --- |
| Active Directory / domain controller | Authentication, account and directory changes |
| Windows endpoint | Security events, PowerShell logging and process telemetry |
| OPNsense | Firewall allow/block events and network context |
| SIEM platform | Searchable ingestion, timestamps and KQL queries |

## Validation Criteria

- Each source has a documented hostname, log type and ingestion status.
- Events arrive with usable timestamps and can be correlated across systems.
- A known test event is found by KQL before a case is marked complete.

## Related Notes

- [Project Overview](01_Project_Overview.md)
- [Network Architecture](03_Network_Architecture.md)
- [Active Directory](06_Active_Directory.md)
- [Project Roadmap](08_Project_Roadmap.md)




