---
title: Implementation Log
tags: [blue-team, implementation-log]
status: active
---

# Implementation Log

## 2026-11-09 - Documentation structure reordered

### Change

- Renumbered the active technical notes into the ordered sequence from project
  overview through lessons learned.

### Result

- Blue Team architecture, OPNsense, Active Directory, SIEM and roadmap notes
  now appear in implementation order.

## 2026-11-09 - Blue Team Operations Lab initialized

### Change

- Re-scoped inherited homelab documentation to an AD, SIEM and investigation lab.
- Defined the A1–A6 roadmap and target isolated network.
- Removed personal-service documentation and obsolete diagrams from the active project.

### Result

- Phase A1 can begin with OPNsense activation, AD deployment and SIEM selection.

### Related Notes

- [Project Overview](01_Project_Overview.md)
- [Network Architecture](03_Network_Architecture.md)
- [Project Roadmap](08_Project_Roadmap.md)



## 2026-11-09 - Time synchronization warning identified on DC01

### Observation

- Windows Time Service Event ID `134` reported that the configured manual NTP peer could not be resolved.
- DNS diagnostic testing for the AD domain passed; NTP DNS forwarding and synchronization require remediation and validation.

### Related Note

- [Troubleshooting](10_Troubleshooting.md)

## 2026-11-09 - DC01 DNS and time validation completed

### Result

- `DC01` was promoted as the first domain controller for `lab.internal` (`LAB`).
- `dcdiag /test:dns /v` passed; the DC uses `10.10.10.10` as its local DNS client and forwards external lookups to OPNsense `10.10.10.1`.
- The NTP peer `time.windows.com,0x8` resolved through the DC DNS service and Windows Time was resynchronized.

### Next steps

- Create the reverse lookup zone and initial AD OUs, then deploy the selected SIEM before adding the Windows endpoint.
