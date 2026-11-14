---
title: Lessons Learned
tags: [blue-team, lessons-learned]
status: active
updated: 2026-11-14
---

# Lessons Learned

## Purpose

Retain reusable operational lessons, not a second incident timeline.

## Network and DNS

- One physical NIC and an unmanaged switch constrain physical segmentation.
  The lab uses the isolated virtual vmbr1 behind OPNsense; home-LAN and lab
  subnets must not overlap.
- An operational service's package/registry resolution should not depend
  unintentionally on a powered-off DC. SIEM01 uses OPNsense for external names,
  with conditional forwarding of lab.internal to AD DNS.
- Loopback is host-local: a shared Elasticsearch output of 127.0.0.1 points each
  remote agent back at itself, not at SIEM01.

## Fleet Recovery - 2026-11-14

- Distinguish Fleet Server process/API health, the host agent's Fleet check-in
  and actual event ingestion. One can work while another fails.
- Inspect every unit and policy integration. In the installed build, the
  all-units-Healthy readiness gate blocked check-in with a Stopped duplicate,
  preventing retrieval of the policy that removed it.
- After central duplicate cleanup, one controlled enrollment recovered the cached
  policy deadlock. This is an incident-specific recovery, not a reason to
  reinstall or repeatedly enroll after every reboot.
- Enrollment changed the host's Agent ID. Validate the replacement and match
  exact IDs before retiring old inventory; hostname alone is insufficient.
- Test recovery after a controlled reboot, not just immediately after setup.
  SIEM01 recovered at 4 minutes uptime in this test. DC01's brief EOF cleared
  without reported intervention; recheck current status before reacting.
- Log-file modification time is not event time. Capture current evidence before
  intervention and filter timestamps before treating a historical error as active.

## Evidence and Documentation

- A Healthy agent or unauthenticated HTTP 401 is not proof of event ingestion,
  detection coverage or a completed investigation. Validate a known event and query.
- Keep configuration notes current, the implementation log chronological,
  incident evidence in Troubleshooting and pending work in TODO. Link rather
  than copying the same evolving checkpoint into every note.
- Keep observed, planned and validated claims separate. Preserve historical
  facts, but do not leave old next actions presented as live instructions.

## Related Notes

- [Troubleshooting](10_Troubleshooting.md)
- [Current Lab State](12_Current_Lab_State.md)
- [Implementation Log](09_Implementation_Log.md)
- [Documentation Review](13_Documentation_Review.md)
