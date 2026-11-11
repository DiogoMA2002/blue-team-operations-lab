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
| Operating system | Ubuntu Server installed |
| DNS resolution | Resolved; intended internal DNS chain validated |
| Temporary workaround | Used for package update; revert it with `resolvectl revert ens18` if still active |

Do not install Elastic until the operating system has been updated. Restore the
intended internal DNS chain after the OPNsense resolver remediation.
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


