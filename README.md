---
title: Blue Team Operations Lab
tags: [blue-team, soc, portfolio]
status: active
updated: 2026-11-14
---

# Blue Team Operations Lab

## Overview

Blue Team Operations Lab is a 24-week, evidence-driven cybersecurity project.
It uses the existing Proxmox homelab as infrastructure for an isolated
enterprise network with Active Directory, endpoint telemetry and a
KQL-capable SIEM for log analysis and investigations.

The goal is to demonstrate practical blue-team operations: collecting
telemetry, detecting suspicious activity, investigating alerts, documenting
findings and validating remediation.

## Current Status - 14 November 2026

Phase A1 is in progress. The Elastic management plane is operational, DC01 is
enrolled and both agents recovered after one controlled SIEM01 reboot.

- OPNsense VM 102 provides the lab gateway at 10.10.10.1.
- DC01.lab.internal is the deployed AD DS/DNS server, with audited local activity
  and a baseline backup.
- SIEM01 runs Elasticsearch, Kibana and Fleet Server. Both active agents are
  Healthy in Kibana on their respective policy revision 3, with recent check-ins.
- Kibana TCP 5601 administration is source-restricted through OPNsense; no
  Elasticsearch or Fleet Server WAN port-forward is configured.
- Windows collection is configured for DC01. A controlled event arriving in Kibana
  is still unvalidated; no Windows client has been deployed.
- The old Offline siem01 record is historical inventory; retirement is not confirmed.

Current configuration: [Current Lab State](Docs/12_Current_Lab_State.md).
Final Fleet diagnosis and recovery: [Troubleshooting](Docs/10_Troubleshooting.md#fleet-server-check-in-blocked-after-siem01-reboot).

## Lab Foundation

- Proxmox host: `pve01.lab.internal` (`192.168.1.10`)
- OPNsense VM 102: firewall and gateway for the isolated lab
- Internal bridge: `vmbr1`; lab network: `10.10.10.0/24`
- Domain controller: `DC01.lab.internal` (`10.10.10.10`)
- SIEM: `SIEM01` (`10.10.10.20`)
- Existing backup storage: `Backup-320`

Personal/media services remain on the home LAN and are out of scope for this
project.

## Roadmap

| Phase | Weeks | Outcome |
| --- | --- | --- |
| A1 — Foundation and first investigations | 1–4 | Validated collection and three complete cases |
| A2 — Identity and queries | 5–8 | Two additional cases, KQL queries and phishing investigation |
| A3 — Detection quality and operations | 9–12 | Five tested detections, playbook, escalation and handover |
| A4 — Network context and remediation | 13–16 | Firewall logs, PCAP analysis and post-remediation reassessment |
| A5 — Automation and CTI | 17–20 | Standardized investigation output, integrated tool and CTI note |
| A6 — Final case and presentation | 21–24 | Consolidated investigation, demonstration and publishable version |

See [Project Roadmap](Docs/08_Project_Roadmap.md) for measurable deliverables.

## Portfolio Evidence

Each completed case should include the scenario, telemetry source, detection
logic, KQL query, investigation timeline, evidence, remediation decision and
lessons learned. Secrets, production credentials, personal data and raw
sensitive logs must never be committed.

## Key Documentation

- [Network Architecture](Docs/03_Network_Architecture.md)
- [Active Directory](Docs/06_Active_Directory.md)
- [SIEM and Telemetry](Docs/07_SIEM_and_Telemetry.md)
- [Current Lab State](Docs/12_Current_Lab_State.md)
- [Implementation Log](Docs/09_Implementation_Log.md)
- [Troubleshooting](Docs/10_Troubleshooting.md)
