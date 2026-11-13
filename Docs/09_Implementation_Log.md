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

## 2026-11-10 - Reverse DNS validated for DC01

### Change

- Created the AD-integrated IPv4 reverse lookup zone `10.10.10.in-addr.arpa` with secure dynamic updates.
- Added the PTR record for `10.10.10.10` → `DC01.lab.internal`.

### Validation

- `nslookup 10.10.10.10` returned `DC01.lab.internal` from the local DNS server.

## 2026-11-10 - AD organisational-unit structure created

### Change

- Created the tiered OU structure for administrative accounts, user accounts, groups, workstations, servers and service accounts.
- Preserved the default `Users`, `Computers` and `Domain Controllers` containers/OUs; `DC01` remains in `Domain Controllers`.

### Next step

- Create a named Tier 0 administrative account, then begin SIEM deployment.

## 2026-11-10 - Named Tier 0 administrative account created

### Change

- Created `adm.tier0` in `Admin Accounts/Tier 0` and assigned it to the built-in `Domain Admins` group.

### Validation pending

- Authenticate as `LAB\adm.tier0` and verify its group membership and AD administrative access before retiring it to routine use.

## 2026-11-10 - Named Tier 0 administration validated

### Validation

- `adm.tier0` successfully authenticated as `LAB\adm.tier0`.
- Domain administrative access and assigned group membership were validated.

### Operational decision

- Use `adm.tier0` for routine domain administration. Retain the built-in `Administrator` account as break-glass only.

## 2026-11-10 - Initial AD test identities and groups created

### Change

- Created non-privileged users `usr.alice` and `usr.bob` in the test-user OU.
- Created global security role groups `GG-Helpdesk` and `GG-IT-Support`.

### Use

- These objects provide controlled identity and group-membership events for later SIEM ingestion and investigation exercises.

## 2026-11-10 - DC audit logging baseline validated

### Change

- Created and linked `GPO-DC-Audit-Logging` to the `Domain Controllers` OU.
- Enabled advanced AD authentication, account-management, directory-change and process-creation auditing, plus PowerShell logging and command-line capture.

### Validation

- `auditpol` confirmed Kerberos, credential-validation, account-management, directory-service-change and process-creation subcategories.
- PowerShell Event ID `4104` was observed after a controlled `Get-Process` command.

## 2026-11-10 - AD group-membership audit test completed

### Controlled test

- Temporarily added `usr.bob` to `GG-Helpdesk`, then removed the membership.

### Evidence

- Security Event ID `4728` recorded the addition to a security-enabled global group.
- Security Event ID `4729` recorded the removal.

### Result

- The DC audit baseline produces the expected identity-change evidence for SIEM ingestion.

## 2026-11-10 - DC01 recovery baseline backed up

### Backup

- Completed a manual Proxmox backup of VM 104 (`DC01`) to `Backup-320`.
- Used snapshot mode with ZSTD compression.

### Result

- A recoverable baseline exists after AD, DNS, audit-policy and identity configuration.
- A restore test and an off-host copy remain outstanding.

## 2026-11-11 - SIEM01 deployed; OPNsense DNS resolver issue identified

### Deployment

- Created `SIEM01` (VM 105) on `vmbr1` with static address `10.10.10.20` and Ubuntu Server installed.

### Issue and temporary action

- External DNS resolution through OPNsense Unbound returned `SERVFAIL`, blocking Ubuntu package updates.
- The approved temporary update-only workaround sets Cloudflare DNS at runtime with `resolvectl`; it is reverted after updates.

### Follow-up

- Repair OPNsense Unbound, verify the intended DC01 DNS forwarding path, then install Elastic Security.

## 2026-11-11 - OPNsense Unbound resolution restored

### Root cause and fix

- OPNsense relied on WAN DHCP DNS and Unbound returned `SERVFAIL` for external names.
- Configured Cloudflare upstream resolvers (`1.1.1.1`, `1.0.0.1`), disabled WAN DNS override, and restricted Unbound service access to the lab LAN.

### Validation

- `archive.ubuntu.com` resolved through OPNsense `10.10.10.1` and through DC01 `10.10.10.10`.
- The intended DNS chain is restored; proceed with Elastic installation after reverting any temporary SIEM01 DNS override.

## 2026-11-12 - Elasticsearch installed on SIEM01

### Deployment

- Installed and enabled Elasticsearch on `SIEM01` as the single-node `blue-team-lab` cluster.
- Configured persistent `vm.max_map_count=1048576` and a 2 GiB JVM heap cap for the constrained homelab host.

### Validation

- Configured the built-in `elastic` account password interactively.
- Validated the local HTTPS endpoint using `/etc/elasticsearch/certs/http_ca.crt`.

### Next step

- Install and enroll Kibana, then deploy Fleet Server and Elastic Agent.

## 2026-11-12 - Elasticsearch base configuration documented

### Implemented baseline

- Documented single-node security, TLS, service, heap and kernel-map-limit configuration for SIEM01.

### Before Kibana

- Validate SIEM01 NTP/timezone and restrict Elasticsearch HTTP binding to localhost when applicable.
- Keep Elasticsearch ports `9200` and `9300` inaccessible from outside SIEM01.

## 2026-11-13 - Kibana installed on SIEM01

### Deployment

- Installed Kibana on `SIEM01`, configured `server.name` as `siem01` and bound it to `10.10.10.20`.
- Enabled and started the Kibana systemd service.
- Generated a Kibana enrollment token locally; the secret token is not stored in the vault.

### Next step

- Create a source-restricted OPNsense port-forward for TCP 5601 to complete browser enrollment from the administrator workstation.

## 2026-11-13 - Kibana enrolled and restricted access configured

### Validation

- Completed the Kibana browser enrollment with the locally generated enrollment token.
- Configured OPNsense destination NAT from the administration workstation (`192.168.1.6`) to `SIEM01` TCP `5601`.
- The associated WAN pass rule is source-restricted; Elasticsearch ports `9200` and `9300` remain inaccessible externally.

### Next step

- Sign in to Kibana with the `elastic` account and deploy Fleet Server for Elastic Agent management.

## 2026-11-13 - Kibana encryption keys configured for Fleet

### Remediation

- Fleet initialization reported that the encrypted saved-object API key was missing.
- Generated persistent Kibana encryption keys and configured the Encrypted Saved Objects, Reporting and Security encryption settings in `/etc/kibana/kibana.yml`.
- Restarted Kibana. The generated keys and service-account token are secrets and were not recorded.

### Next step

- Confirm Fleet initializes successfully, then deploy a Fleet Server on SIEM01.

## 2026-11-13 - Fleet Server deployed on SIEM01

### Deployment

- Installed Elastic Agent `9.5.6` (Linux x86_64) on SIEM01 in Fleet Server mode.
- Enrolled it using the Fleet-generated `fleet-server-policy` with Elasticsearch local to SIEM01.
- Fleet Server listens internally at `https://10.10.10.20:8220`; no WAN/NAT exposure was created for this port.

### Next step

- Confirm the Fleet Server agent is Healthy in Kibana, then create and enroll the DC01 agent policy.

### Fleet Server validation

- Verified in Kibana Fleet > Agents that `siem01` is Healthy under Fleet Server Policy.
- Elastic Agent version: `9.5.6`; reported memory use: approximately `119 MB`.

## 2026-11-13 - DC01 Fleet policy created

- Created `Windows Domain Controllers - Baseline` (revision 1) for DC01.
- Configured FQDN host-name display, Default space and `default` namespace.
- The policy is not yet assigned to DC01; review its initial integration and add the Windows telemetry integration next.

## 2026-11-13 - DC01 Windows telemetry integration configured

- Added `Windows Security Telemetry - DC01` to the DC agent policy (policy revision 2).
- Enabled PowerShell, PowerShell Operational and Windows Defender Operational collection.
- Deferred Sysmon Operational collection until Sysmon is installed; disabled raw-event preservation to control storage use.

### Next step

- Enroll DC01 with the Windows Domain Controllers - Baseline policy.

## 2026-11-13 - Continuation checkpoint recorded

- Updated project status, TODO and roadmap to reflect the deployed Elastic management plane and pending DC01 agent enrollment.
- Created `Docs/12_Current_Lab_State.md` as the sanitized resumption checkpoint.
- No passwords, API keys, enrollment tokens, encryption keys or service tokens were recorded.
