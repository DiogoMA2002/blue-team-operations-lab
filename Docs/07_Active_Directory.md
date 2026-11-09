---
title: Active Directory
tags:
  - homelab
  - windows-server
  - active-directory
status: planned
---

# Active Directory

> [!note] Active project component
> Active Directory is a core component of the Blue Team Operations Lab. Its
> deployment and telemetry validation start in Phase A1.

## Purpose

The Windows Server environment provides enterprise identity, DNS, policy management, and future certificate/file services.

## Planned Roles

- Active Directory Domain Services
- DNS
- Group Policy
- Certificate Services/future PKI
- File Server/future shares

## Domain Design

| Item | Value |
|---|---|
| Domain name | TODO |
| NetBIOS name | TODO |
| Domain controller | TODO |
| DNS server | TODO |
| Time source | TODO |

## Windows Server

Roles:

- Active Directory
- DNS
- Group Policy
- Certificate Services (future)
- File Server (future)

## Windows Client

Purpose: enterprise workstation joined to Active Directory.

Used to test:

- GPOs
- Authentication
- Security policies
- Hardening

## GPO Ideas

- Password policy
- Account lockout policy
- Windows Defender configuration
- Local administrator restrictions
- Audit policy
- PowerShell logging
- Sysmon deployment

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [Implementation Log](09_Implementation_Log.md)
- [Troubleshooting](10_Troubleshooting.md)
