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
