---
title: Implementation Log
tags: [blue-team, implementation-log]
status: active
updated: 2026-11-14
---

# Implementation Log

## Purpose

Chronological implementation history. Entries record changes and observed results,
not a live checklist. Current state is in [Current Lab State](12_Current_Lab_State.md);
pending deliverables are in [TODO](../TODO.md). Secrets and raw sensitive logs are excluded.

## 2026-11-09 - Project Scope and Documentation

- Re-scoped inherited homelab notes to the Blue Team Operations Lab: isolated
  AD/SIEM/endpoint telemetry and a 24-week A1-A6 evidence-driven portfolio.
- Reordered technical notes into implementation sequence, replaced the target
  diagram and removed obsolete personal/media-service documentation from the
  active project. Those historical changes predate this cleanup.

## 2026-11-09 - DC01 Promotion, DNS and Time

- Promoted Windows Server 2022 DC01 as first lab.internal (LAB) domain controller,
  with AD DS, DNS and Group Policy Management.
- Set DC NIC DNS to 10.10.10.10 plus IPv6 loopback; external DNS forwarding through
  OPNsense 10.10.10.1. Domain discovery and dcdiag /test:dns /v passed.
- Time-Service warning 134 around 14:00 recorded failure to resolve
  time.windows.com,0x8 (0x80072AF9); the earlier source was Local CMOS Clock.
  The peer subsequently resolved, Windows Time was reconfigured/restarted and
  resynchronized. See [Troubleshooting](10_Troubleshooting.md#dc01-could-not-resolve-its-ntp-peer).

## 2026-11-10 - AD Structure, Identities and Audit Tests

- Created AD-integrated reverse zone 10.10.10.in-addr.arpa with secure dynamic
  updates and PTR 10.10.10.10 -> DC01.lab.internal; lookup validated.
- Created tiered OUs for administrative/user/service accounts, groups,
  workstations and servers. Preserved default containers and DC01 in Domain Controllers.
- Created adm.tier0 in Admin Accounts/Tier 0, assigned Domain Admins and validated
  authentication/administrative access. Built-in Administrator retained as break-glass.
- Created nonprivileged usr.alice and usr.bob, plus global security groups
  GG-Helpdesk and GG-IT-Support for controlled future tests.
- Linked GPO-DC-Audit-Logging to Domain Controllers: authentication, account/group,
  directory changes, process creation with command lines, PowerShell Script Block
  and Module Logging. auditpol validated the configured subcategories.
- Local event 4104 followed a harmless Get-Process command. Adding/removing
  usr.bob to/from GG-Helpdesk generated local Security events 4728/4729.
  These are local audit tests, not SIEM ingestion validation.
- Backed up VM 104 DC01 to Backup-320 using snapshot mode and ZSTD. Restore testing
  and off-host copying were not validated.

## 2026-11-11 - SIEM01 Deployment and DNS Recovery

- Deployed VM 105 SIEM01 on vmbr1 with Ubuntu Server, static 10.10.10.20/24,
  gateway 10.10.10.1.
- Ubuntu package access failed on DNS SERVFAIL through Unbound despite working IP
  connectivity. Used a temporary runtime Cloudflare DNS override for updates,
  then reverted it.
- Set OPNsense upstreams 1.1.1.1/1.0.0.1, disabled WAN DHCP DNS override and
  restricted Unbound to lab-LAN queries, with all outgoing interfaces.
- archive.ubuntu.com resolved through OPNsense and DC01. At this stage SIEM01 used
  the DC01 -> OPNsense DNS chain. It was deliberately replaced on 14 November.

## 2026-11-12 - Elasticsearch Baseline

- Installed/enabled Elasticsearch: single-node blue-team-lab, node siem01.
- Fixed heap at 2 GiB; persisted vm.max_map_count=1048576.
- Set the built-in account password interactively and validated local HTTPS
  with the generated HTTP CA. No password recorded.
- Initial local-only access planning preceded remote agent enrollment; it is
  historical, not the current shared data-output design.

## 2026-11-13 - Kibana, Fleet and DC Policy

- Installed/enabled Kibana, server.name siem01, bound to 10.10.10.20:5601.
  Completed browser enrollment through OPNsense DNAT restricted to the trusted
  administration workstation (192.168.1.6). No Elasticsearch WAN forwarding.
- Fleet initialization required persistent Kibana Encrypted Saved Objects,
  Reporting and Security encryption keys; generated/configured them and restarted
  Kibana. Key material and enrollment/service tokens were not documented.
- Installed Elastic Agent 9.5.6 in Fleet Server mode on SIEM01 using
  fleet-server-policy; internal URL https://10.10.10.20:8220, no WAN NAT.
  siem01 Healthy was verified in Kibana; Quick Start certificate handling was retained.
- Created Windows Domain Controllers - Baseline revision 1 with FQDN identification,
  then revision 2 with System and Windows Security Telemetry - DC01 integration.
  Enabled PowerShell/Operational and Defender channels, deferred Sysmon, disabled
  raw event.original preservation.
- Recorded the initial continuation checkpoint; DC01 enrollment was then pending.

## 2026-11-14 - DNS Resilience and DC01 Enrollment

- Initial SIEM01 Fleet bootstrap re-enrollment restored a Healthy record while a
  prior Offline record remained. This was an interim recovery, not reboot validation.
- Made SIEM01 Netplan DNS 10.10.10.1 persistent, with lab.internal search domain.
  Replaced the prior DNSmasq forwarding approach with OPNsense Unbound Query
  Forwarding of lab.internal to 10.10.10.10:53.
- resolvectl showed OPNsense as sole/current resolver; external names resolved
  with DC01 off, while AD names timed out as expected.
- Installed Agent 9.5.6 on DC01 under its baseline policy. Prior partial attempts
  had left stale inventory; the active FQDN record was Healthy.
- Corrected the managed Fleet data output from remote loopback to
  https://10.10.10.20:9200 and made Elasticsearch reachable internally
  (http.host: 0.0.0.0), without WAN NAT. Restarted Elasticsearch, Kibana and the
  DC01 Agent service; revision 3 applied and components became Healthy.
- HTTP 401 without credentials confirmed the authentication challenge, not
  authenticated access or controlled event ingestion.

## 2026-11-14 - Interim Fleet Rebuild and Startup Guard

- The inherited record documents a clean SIEM01 Agent/Fleet Server uninstall and
  reinstall after unreliable bootstrap recovery; Elasticsearch, Kibana, policies
  and DC01 were retained. A fresh locally generated service token was used.
- Added /usr/local/sbin/wait-for-elasticsearch and systemd drop-in
  /etc/systemd/system/elastic-agent.service.d/10-wait-for-elasticsearch.conf.
  Recorded behavior: wait for local ES HTTPS readiness, order after Elasticsearch
  and network-online.target, retry on failure.
- Fleet and DC01 became Healthy after intervention, including an earlier manual
  DC01 service restart. The exact earlier cause was unconfirmed; calling this a
  proven startup race would overstate the evidence. The subsequent reboot still failed.

## 2026-11-14 - Final Fleet Diagnosis and Validated Recovery

- Post-reboot SIEM01 host check-in remained STARTING/Offline while Fleet Server
  APIs and DC01 check-ins worked. A SIEM01 Agent service restart did not fix it.
- Confirmed duplicate Fleet Server integrations: one input Healthy, one Stopped.
  Installed-source review found the all-units-Healthy gate before host check-in.
- Removed the duplicate centrally; cached revision 2 remained Outdated on the
  blocked host. One controlled Fleet Server-mode enroll using the installed
  binary restored Connected status with one Healthy input and policy revision 3.
  This final recovery did not require uninstall/reinstall or DC01 re-enrollment.
- Enrollment created a replacement Agent ID. Exact old/new/input IDs and source
  references are retained once in
  [Troubleshooting](10_Troubleshooting.md#fleet-server-check-in-blocked-after-siem01-reboot).
- After a controlled SIEM01 reboot, guest time 21:54:59 with 4 minutes uptime,
  local Fleet/Agent were Healthy. DC01's transient EOF cleared without a reported
  intervening manual service restart.
- Final screenshot confirmed both active agents Healthy on revision 3 with recent
  check-ins. The old siem01 record remained Offline on revision 2; retirement was
  explained but completion is unconfirmed. Controlled DC01 ingestion is still pending.
- This supports the duplicate-input readiness diagnosis and validates recovery for
  one reboot cycle, not all future interruptions or independent effects of each fix.

## 2026-11-14 - Approved Documentation Consolidation

- Reviewed core notes and registered findings before changing existing text.
- User explicitly approved section-level consolidation/correction, preserving
  historical facts and all existing files/media.
- Replaced stacked checkpoints with one current baseline, consolidated Fleet
  troubleshooting and same-day log narration, corrected stale pending actions,
  DNS/output guidance, encoding, navigation and storage wording.
- Kept DNS/NTP/output incidents, AD audit evidence, backup history, earlier
  recovery attempts and technical limitations; no credentials/raw sensitive logs read.
- No file/image deleted or moved, no live lab change and no commit performed.
  Review scope, validation and exclusions:
  [Documentation Review](13_Documentation_Review.md).

## Related Notes

- [Current Lab State](12_Current_Lab_State.md)
- [Troubleshooting](10_Troubleshooting.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [CHANGELOG](../CHANGELOG.md)
