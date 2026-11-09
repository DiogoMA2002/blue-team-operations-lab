# Blue Team Operations Lab

## Overview

Blue Team Operations Lab is a 24-week, evidence-driven cybersecurity project.
It uses the existing Proxmox homelab as infrastructure for an isolated
enterprise network with Active Directory, endpoint telemetry and a
KQL-capable SIEM for log analysis and investigations.

The goal is to demonstrate practical blue-team operations: collecting
telemetry, detecting suspicious activity, investigating alerts, documenting
findings and validating remediation.

## Lab Foundation

- Proxmox host: `pve01.lab.internal` (`192.168.1.10`)
- OPNsense VM 102: preserved firewall baseline for the isolated lab
- Internal bridge: `vmbr1`, planned lab network `10.10.10.0/24`
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

See [Project Roadmap](Docs/12_Project_Roadmap.md) for measurable deliverables.

## Portfolio Evidence

Each completed case should include the scenario, telemetry source, detection
logic, KQL query, investigation timeline, evidence, remediation decision and
lessons learned. Secrets, production credentials, personal data and raw
sensitive logs must never be committed.
