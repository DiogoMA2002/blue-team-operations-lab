# Blue Team Operations Lab

## Overview

Blue Team Operations Lab is a 24-week, evidence-driven cybersecurity project.
It uses the existing Proxmox homelab as infrastructure for an isolated
enterprise network with Active Directory, endpoint telemetry and a
KQL-capable SIEM for log analysis and investigations.

The goal is to demonstrate practical blue-team operations: collecting
telemetry, detecting suspicious activity, investigating alerts, documenting
findings and validating remediation.

## Current Status — 9 November 2026

Phase A1 is in progress.

- OPNsense VM 102 provides the isolated-lab gateway on `10.10.10.1`.
- `DC01.lab.internal` is deployed on `vmbr1` as the first domain controller
  for `lab.internal` (`LAB`).
- AD DS, DNS and Group Policy Management are installed.
- The DC uses static address `10.10.10.10/24`; `dcdiag /test:dns /v` passed.
- OPNsense `10.10.10.1` is the DNS forwarder, and Windows Time has been
  configured and resynchronized using `time.windows.com`.
- The SIEM selection, reverse-DNS zone, AD organisational units, endpoint and
  telemetry collection remain next steps.

## Lab Foundation

- Proxmox host: `pve01.lab.internal` (`192.168.1.10`)
- OPNsense VM 102: firewall and gateway for the isolated lab
- Internal bridge: `vmbr1`; lab network: `10.10.10.0/24`
- Domain controller: `DC01.lab.internal` (`10.10.10.10`)
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
- [Implementation Log](Docs/09_Implementation_Log.md)
- [Troubleshooting](Docs/10_Troubleshooting.md)



