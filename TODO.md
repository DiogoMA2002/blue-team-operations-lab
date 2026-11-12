---
title: Blue Team Operations Lab TODO
tags:
  - blue-team
  - soc
  - todo
status: active
---

# TODO

## A1 â€” Foundation and First Investigations

- [x] Select Elastic Security self-managed as the target SIEM and KQL workflow
- [x] Power on and validate OPNsense VM 102 and `vmbr1`
- [x] Deploy `DC01` (`lab.internal`) as the Active Directory domain controller with DNS
- [ ] Deploy and domain-join a Windows client
- [/] Deploy `SIEM01` (`10.10.10.20`); Ubuntu and Elasticsearch installed, Kibana/Fleet pending
- [x] Repair OPNsense Unbound external DNS resolution and restore the internal DNS chain
- [ ] Complete three documented investigations

## A2â€“A6 â€” Roadmap Preparation

- [ ] Build the KQL query library and identity investigation cases
- [ ] Create detection test cases and a triage playbook
- [ ] Integrate firewall and PCAP evidence
- [ ] Define automation output and CTI workflow
- [ ] Prepare the final investigation and portfolio presentation

## Infrastructure Reliability

- [x] Create a manual `DC01` baseline backup on `Backup-320` (snapshot mode, ZSTD)

- [ ] Configure Proxmox TOTP or WebAuthn; recovery keys already exist
- [ ] Restore one backup into a new unused ID
- [ ] Define an off-host copy for important backups

## Evidence Standards

- [ ] Create a reusable investigation-case template
- [ ] Create a telemetry-source inventory
- [ ] Create a detection and KQL-query register







