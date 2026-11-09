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

The inherited Proxmox host, OPNsense VM 102 and internal `vmbr1` bridge
provide the base. The active lab will add:

- Active Directory Domain Services and DNS
- One or more domain-joined Windows endpoints
- A SIEM platform with KQL query capability
- Endpoint, Windows security and firewall telemetry
- Controlled, authorized test scenarios only

Personal home-LAN and media services remain outside the blue-team lab scope.

## Success Criteria

- Telemetry from identity, endpoints and firewall is searchable and time-aligned.
- Each detection has a test procedure and documented expected evidence.
- Investigations show scope, timeline, analyst decision and remediation result.
- Portfolio material is sanitized and reproducible.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [Proxmox](04_Proxmox.md)
- [OPNsense](06_OPNsense.md)
- [Active Directory](07_Active_Directory.md)
- [SIEM and Telemetry](13_SIEM_and_Telemetry.md)
- [Project Roadmap](12_Project_Roadmap.md)
