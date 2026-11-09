---
title: Active Directory
tags: [blue-team, windows-server, active-directory, identity]
status: active
---

# Active Directory

## A1 Domain-Controller Baseline

| Item | Approved value |
| --- | --- |
| VM name / hostname | `DC01` |
| Domain | `lab.internal` |
| NetBIOS name | `LAB` |
| Server edition | Windows Server 2022 Evaluation, Desktop Experience |
| vCPU / RAM / disk | 2 vCPU / 4 GiB fixed RAM / 60 GiB |
| Proxmox bridge | `vmbr1` |
| Address | `10.10.10.10/24` |
| Gateway | `10.10.10.1` (OPNsense) |
| Preferred DNS | `10.10.10.10` after the DNS role is installed |
| Time source | Domain hierarchy; configure the PDC emulator to use reliable external NTP |

This is an isolated training domain. Do not use real users, passwords, public
services or router port forwards.

## Deployment Status

> [!success] `DC01` deployed and validated
> On 2026-11-09, `DC01` was promoted as the first domain controller for
> `lab.internal` (`LAB`) with AD DS, DNS and Group Policy Management installed.

| Validation item | Recorded result |
| --- | --- |
| Domain discovery | `Get-ADDomain` returned `lab.internal`; PDC emulator is `DC01.lab.internal` |
| Static IPv4 | `10.10.10.10/24`, gateway `10.10.10.1` |
| DNS client configuration | Local DNS only: `10.10.10.10` plus IPv6 loopback `::1` |
| DNS health | `dcdiag /test:dns /v` passed for `lab.internal` |
| DNS forwarding | OPNsense `10.10.10.1` (`OPNsense.lab.internal`) configured as forwarder |
| External lookup | `time.windows.com` resolved through DNS server `10.10.10.10` |
| Time service | Manual peer `time.windows.com,0x8` configured and Windows Time resynchronized |

Outstanding AD foundation work: reverse lookup zone, initial OUs, a named
administrative account, and SIEM telemetry ingestion.
## Roles in A1

- Active Directory Domain Services (AD DS)
- DNS Server
- Group Policy Management

Certificate Services and file services are deliberately deferred until a later
phase.

## Deployment Sequence

1. Start OPNsense VM 102 and confirm its LAN address is `10.10.10.1`.
2. Create the `DC01` VM on `vmbr1`, install Windows Server and apply updates.
3. Set the approved static IPv4 configuration and rename the server to `DC01`.
4. Install AD DS and DNS, then promote it as the first forest/domain controller
   for `lab.internal`.
5. Reboot and validate DNS, AD DS, time and the `LAB` domain.
6. Create only the initial administrative structure; join the Windows endpoint
   only after DNS and telemetry validation succeed.

## Initial Directory Structure

Create these OUs after promotion:

- `OU=Servers`
- `OU=Workstations`
- `OU=Users`
- `OU=Groups`
- `OU=Service Accounts`

Use a separate named administrative account for routine administration after
the first validation. Keep the built-in `Administrator` account as a sealed
break-glass account; do not use it for daily work.

## Required Validation Evidence

- Screenshot or command output showing `DC01.lab.internal` and its IPv4 address.
- `dcdiag` summary with no failures.
- DNS forward and reverse lookups for `DC01`.
- Evidence that a lab client uses `10.10.10.10` for DNS.
- Windows event collection into the selected SIEM before domain-joining clients.

## Security Baseline (after ingestion works)

- Enable Advanced Audit Policy, including logon, account logon, account
  management, directory-service changes and process creation.
- Enable PowerShell Script Block Logging and Module Logging.
- Deploy Sysmon using a reviewed, version-controlled configuration.
- Create GPOs gradually and record every change and its telemetry effect.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)


