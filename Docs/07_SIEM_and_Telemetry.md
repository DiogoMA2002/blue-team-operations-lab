---
title: SIEM and Telemetry
tags:
  - blue-team
  - siem
  - telemetry
  - kql
status: planned
---

# SIEM and Telemetry

## Purpose

Provide centralized, time-aligned telemetry for blue-team detection and
investigation. The selected SIEM must support KQL for the planned query
workflow.

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
