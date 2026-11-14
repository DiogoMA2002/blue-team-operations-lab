---
title: Blue Team Operations Lab TODO
tags: [blue-team, soc, todo]
status: active
updated: 2026-11-14
---

# TODO

## Immediate Operational Work

- [ ] Generate a controlled DC01 event, verify local evidence and find the matching event in Kibana using KQL
- [ ] Retire only the historical siem01 Agent ID after identity verification; removal is not yet confirmed
- [ ] Confirm SIEM01 time synchronization/timezone and disk versus filesystem/LV capacity before sizing retention

Fleet recovery is validated for the observed reboot cycle: both active agents
Healthy, policy revision 3, with recent Kibana check-ins. The old Offline record
does not represent a second running VM. Details: [Troubleshooting](Docs/10_Troubleshooting.md).

## A1 - Foundation and First Investigations

- [x] Select Elastic Security self-managed as the SIEM and KQL workflow
- [x] Validate OPNsense VM 102 and vmbr1 as the lab gateway/network foundation
- [x] Deploy DC01 AD DS/DNS, initial identities, GPO auditing and local audit-event tests
- [x] Deploy SIEM01 Elasticsearch, Kibana and Fleet Server; enroll DC01
- [x] Repair Unbound resolution and make SIEM01 external DNS independent of DC01 uptime
- [x] Resolve duplicated Fleet Server inputs and validate both agents after a controlled SIEM01 reboot
- [ ] Validate identity/Windows telemetry end-to-end; no ingestion claim from agent health alone
- [ ] Deploy and domain-join a Windows client after DNS and telemetry validation
- [ ] Complete three investigations with the required evidence and remediation validation

## Evidence Standards

- [ ] Create a reusable investigation-case template
- [ ] Create a telemetry-source inventory
- [ ] Create a detection and KQL-query register

## Infrastructure Reliability and Hardening

- [x] Create DC01 baseline backup on Backup-320, snapshot mode/ZSTD
- [ ] Test restore into a new unused VM ID
- [ ] Define an off-host copy for important backups
- [ ] Configure Proxmox TOTP or WebAuthn; recovery keys already exist
- [ ] Validate lab-to-home isolation and necessary egress rules
- [ ] Replace Fleet Quick Start certificate handling with lab-CA verification
- [ ] Review SIEM01 host-firewall access to internal Elasticsearch
- [ ] Configure retention after confirming data streams, ingestion volume and capacity
- [ ] Deploy reviewed Sysmon configuration in a later controlled change

## A2-A6 Preparation

- [ ] Build the KQL library and identity/phishing cases
- [ ] Create tested detections, a triage playbook, escalation and handover
- [ ] Integrate OPNsense firewall and PCAP evidence
- [ ] Define automation output and CTI workflow
- [ ] Prepare the final investigation and sanitized portfolio presentation

## Documentation Review

- [x] Record final Fleet explanation and review stale/repetitive documentation
- [x] Obtain user approval and consolidate the reviewed text without deleting files or media

## Related Notes

- [Project Roadmap](Docs/08_Project_Roadmap.md)
- [Current Lab State](Docs/12_Current_Lab_State.md)
- [Documentation Review](Docs/13_Documentation_Review.md)
