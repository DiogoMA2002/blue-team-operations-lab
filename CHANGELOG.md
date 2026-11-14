---
title: Changelog
tags: [blue-team, changelog]
status: active
updated: 2026-11-14
---

# Changelog

## 2026-11-14

### Implemented and Validated

- Made SIEM01 DNS independent of DC01 uptime through OPNsense Unbound and
  conditional AD-domain forwarding.
- Enrolled DC01; corrected the shared Elasticsearch output to SIEM01's internal
  endpoint. Both active agents now use policy revision 3.
- Resolved the Fleet host check-in failure through duplicate-integration cleanup
  and one controlled enrollment; both agents recovered after a controlled SIEM01
  reboot. The old Offline inventory record's retirement is not yet confirmed.
- Controlled event ingestion remains unvalidated. Earlier bootstrap recoveries,
  rebuild and startup guard are preserved in the implementation history.

### Documentation

- Recorded final diagnosis, evidence, limitations and historical Agent IDs.
- Completed the user-approved consolidation: current baselines, chronological
  implementation log, one Fleet incident, concise lessons and updated TODO/roadmap.
- Corrected stale instructions, encoding/formatting and index navigation.
- Preserved historical facts and all files/images. No live lab change or Git commit
  was made by the documentation cleanup.

## 2026-11-13

- Installed/enrolled Kibana through source-restricted OPNsense access.
- Configured persistent Kibana encryption keys, deployed Fleet Server and prepared
  the DC01 Windows collection policy. Secrets were not retained in notes.

## 2026-11-12

- Installed and locally validated single-node Elasticsearch with a 2 GiB heap and
  persistent kernel-map limit.

## 2026-11-11

- Deployed SIEM01 and restored external DNS through OPNsense after SERVFAIL.

## 2026-11-10

- Created reverse DNS, tiered OUs, named administration and nonprivileged test
  identities/groups; validated local AD/PowerShell auditing.
- Completed DC01 baseline backup to Backup-320; restore validation remains pending.

## 2026-11-09

- Initialized the Blue Team Operations Lab scope, A1-A6 roadmap and evidence rules.
- Reordered technical documentation and replaced the target network diagram.
- Removed obsolete inherited personal/media-service documentation from the active
  project at that time; no such files were deleted in the 14 November cleanup.
- Deployed/promoted DC01 and validated DNS/time after the initial NTP warning.

## Related Notes

- [Implementation Log](Docs/09_Implementation_Log.md)
- [Troubleshooting](Docs/10_Troubleshooting.md)
- [Documentation Review](Docs/13_Documentation_Review.md)
