---
title: Blue Team Operations Lab TODO
tags:
  - blue-team
  - soc
  - todo
status: active
---

# TODO

## A1 — Foundation and First Investigations

- [ ] Confirm the target SIEM and its KQL workflow
- [ ] Power on and validate OPNsense VM 102 and `vmbr1`
- [x] Deploy `DC01` (`lab.internal`) as the Active Directory domain controller with DNS
- [ ] Deploy and domain-join a Windows client
- [ ] Configure endpoint and Windows event telemetry ingestion
- [ ] Validate time synchronization, host identity and log arrival
- [ ] Complete three documented investigations

## A2–A6 — Roadmap Preparation

- [ ] Build the KQL query library and identity investigation cases
- [ ] Create detection test cases and a triage playbook
- [ ] Integrate firewall and PCAP evidence
- [ ] Define automation output and CTI workflow
- [ ] Prepare the final investigation and portfolio presentation

## Infrastructure Reliability

- [ ] Configure Proxmox TOTP or WebAuthn; recovery keys already exist
- [ ] Restore one backup into a new unused ID
- [ ] Define an off-host copy for important backups

## Evidence Standards

- [ ] Create a reusable investigation-case template
- [ ] Create a telemetry-source inventory
- [ ] Create a detection and KQL-query register


