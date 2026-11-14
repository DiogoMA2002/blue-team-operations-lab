---
title: Blue Team Operations Lab Overview
tags:
  - blue-team
  - soc
  - active-directory
  - siem
status: active
---

# Blue Team Operations Lab Overview

## Objective

Build and operate a small enterprise-style lab that produces defensible
blue-team evidence: endpoint and identity telemetry, SIEM analysis with KQL,
detection validation, investigation cases and remediation follow-up.

## Architecture Scope

The Proxmox host, OPNsense VM 102 and internal vmbr1 bridge provide the
foundation. Active Directory/DNS, Elastic SIEM and Fleet management are deployed.
Both enrolled agents recovered after a controlled SIEM01 reboot. The next
validation is a controlled DC01 event arriving in Kibana.

Planned outcomes remain:

- Domain-joined Windows endpoints and validated endpoint telemetry.
- Searchable identity, Windows security and firewall collection.
- Controlled, authorized detection tests and complete investigations.

Personal home-LAN and media services remain outside the blue-team lab scope.

## Success Criteria

- Telemetry from identity, endpoints and firewall is searchable and time-aligned.
- Each detection has a test procedure and documented expected evidence.
- Investigations show scope, timeline, analyst decision and remediation result.
- Portfolio material is sanitized and reproducible.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [Proxmox](04_Proxmox.md)
- [OPNsense](05_OPNsense.md)
- [Active Directory](06_Active_Directory.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Project Roadmap](08_Project_Roadmap.md)
