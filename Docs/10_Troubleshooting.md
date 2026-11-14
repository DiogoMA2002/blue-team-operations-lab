---
title: Troubleshooting
tags: [blue-team, troubleshooting]
status: active
updated: 2026-11-14
---

# Troubleshooting

## Purpose and Status

Record incidents, evidence, causes, fixes and validation. The Fleet reboot issue
is resolved for the observed test cycle; controlled DC01 event ingestion is a
separate open validation task. This is not a record of completed detection cases.

## Fleet Server Check-in Blocked After SIEM01 Reboot

**Date:** 2026-11-14. **Status:** resolved for the observed reboot cycle.

### Symptoms and Environment

- SIEM01 runs Elasticsearch, Kibana and Fleet Server with Elastic Agent 9.5.6.
- After reboot, Kibana reported Fleet Server not Healthy; siem01 was Offline
  while the local agent showed Fleet STARTING and Agent HEALTHY Running.
- The Fleet Server process/API was Healthy and DC01 continued recent check-ins.
  A manual SIEM01 Elastic Agent service restart did not resolve the host check-in.
- Two Fleet Server integrations existed in Fleet Server Policy revision 2.
  One input was Healthy and the other Stopped; their roles swapped after restart.

### Evidence and Timeline

| Observation | Result |
| --- | --- |
| Services and resources | Elasticsearch/Kibana/Elastic Agent active; NRestarts=0 at initial check; SIEM01 5.3 GiB RAM, 1.4 GiB available and 794 MiB swap used; root filesystem 35% used |
| Fleet endpoints | TCP 8220 listening; loopback/internal-IP /api/status HEALTHY |
| Elasticsearch | Internal TCP 9200 responded HTTP 401 without credentials; authentication challenge, not authenticated access or ingestion proof |
| Local Fleet port | TCP 8221 bound to 127.0.0.1; /api/status HTTP 200/HEALTHY using a diagnostic -k probe |
| Error classification | Earlier invalid-API-key match was not established as current-boot evidence; boot-filtered results contained connection refused |
| Recent logs | At 21:18:04.227, dial to 127.0.0.1:8221 was refused near service startup; subsequent probes succeeded |
| Policy cleanup | Central policy changed; siem01 still reported revision 2/Outdated policy and retained both inputs locally |
| Controlled enrollment | Installed agent re-enrolled once in Fleet Server mode against the cleaned existing policy; by the 21:47 documentation checkpoint Fleet was Connected with one Healthy input |
| Reboot validation | Guest time 21:54:59, uptime 4 minutes: SIEM01 Fleet HEALTHY Connected / Agent HEALTHY Running |
| DC01 recovery | Brief EOF, then HEALTHY Connected / HEALTHY Running without reported manual intervention |
| Final Kibana screenshot | Both active agents Healthy on revision 3; siem01 activity 30 seconds ago and DC01 22 seconds ago |

Command output and screenshots were supplied in the conversation. No raw sensitive
logs or secrets are stored in these notes. Assistant checkpoint times are not
precise timestamps for the user's underlying actions.

### Cause and Mechanism

The policy had two Fleet Server inputs. In the installed Agent commit
468451e5f903c4791a8e785a183ab1250da744e8, restored state is applied before the
managed config manager starts its check-in gateway. The readiness test requires
every Fleet Server unit Healthy. A Stopped duplicate therefore blocks host
check-in even when another input serves agents successfully.

This explains the observed loop: cached duplicated policy -> readiness wait ->
no host check-in -> no updated policy to remove the duplicate.
The installed-source review and recovery evidence support this diagnosis;
cleanup and enrollment were not independently tested as separate remedies.
An invalid API key, memory exhaustion or a simple Elasticsearch startup race was
not established as the cause of this failure.

Sources: [startup sequence](https://github.com/elastic/elastic-agent/blob/468451e5f903c4791a8e785a183ab1250da744e8/internal/pkg/agent/application/managed_mode.go#L128-L152)
and [readiness test](https://github.com/elastic/elastic-agent/blob/468451e5f903c4791a8e785a183ab1250da744e8/internal/pkg/agent/application/managed_mode.go#L267-L299).

### Fix and Validation

1. Mapped the integration names to input IDs and retained fleet_server-1.
2. Removed the duplicate fleet_server-2 centrally; host check-in remained blocked
   on its cached configuration.
3. Performed one controlled enroll using the already installed Elastic Agent
   binary, existing fleet-server-policy and a locally entered service token.
   No uninstall/install loop, Elasticsearch/Kibana reinstall or DC01 re-enrollment.
4. Confirmed a single Healthy Fleet Server input and Fleet Connected.
5. Rebooted SIEM01 and confirmed both local agents recovered; final Kibana
   screenshot confirmed Healthy/current revision/recent check-ins.

Recovery enrollment used the internal Elasticsearch endpoint, the Kibana-provided
CA fingerprint and port 8220. Token entry was hidden using a shell variable;
the token itself is not recorded. The generated install command was adapted to
enroll, without --install-servers or --force. This is an incident-specific
recovery, not an instruction to re-enroll a healthy server.

### Inventory Identity and Remaining Work

| Record | ID | Disposition |
| --- | --- | --- |
| Active siem01 | 1a315292-b68b-4712-9a38-ef57b1e7f53d | Keep; Fleet Server Policy revision 3 |
| Historical siem01 | 6ef02f3c-8ea3-4671-91ee-98a862c6fe30 | Last seen Offline/revision 2; retirement not confirmed |
| Retained fleet_server-1 input | 8891773c-4946-4d77-9dbd-762e7517b1bc | Healthy |
| Removed fleet_server-2 input | 8cf28819-4b32-496c-a265-5ae1a5cbcd4f | Absent from recovered local status |

Before retiring historical inventory, check the full old Agent ID. Use unenrollment
only on that record; if unreachable, forced unenrollment invalidates its keys and
is irreversible. Do not uninstall the active host or remove its policy.
[Elastic unenrollment behavior](https://www.elastic.co/docs/reference/fleet/unenroll-elastic-agent)

A controlled DC01 event in Kibana is still pending. One successful reboot does
not prove recovery under every future interruption or power-loss scenario.

### Prevention

Keep one intended Fleet Server integration in this policy. Separate process/API
health, host check-in and event ingestion. Inspect policy inputs and timestamped
evidence before changing certificates, credentials or bootstrap state. Repeated
reinstallation and --force enrollment are not routine repair steps.

## DC01 Could Not Resolve Its NTP Peer

**Date:** 2026-11-09. **Status:** resolved in the recorded validation.

### Symptoms and Environment

Windows Server 2022 DC01, first DC/DNS server for lab.internal (10.10.10.10),
recorded Time-Service warning 134 around 14:00: time.windows.com,0x8 could not
resolve, error 0x80072AF9. Its earlier time source was Local CMOS Clock.

### Investigation, Fix and Validation

After promotion, NIC DNS was set to local AD DNS. AD discovery and
dcdiag /test:dns /v passed; the configured OPNsense forwarder (10.10.10.1)
resolved time.windows.com through DC01. Windows Time was reconfigured to the
manual peer, restarted and resynchronized. The recorded explanation is a
transient post-promotion DNS failure; exact initial triggering conditions were
not captured. Event 134 is historical, not a current NTP failure.

### Prevention

First DC NIC DNS should use its own AD DNS. Configure external resolution on the
DNS service forwarder, not as an alternate public NIC DNS server. Confirm DNS
before troubleshooting NTP. Do not infer current synchronization from an old test.

## SIEM01 External DNS SERVFAIL

**Date:** 2026-11-11. **Status:** resolved.

### Symptoms and Investigation

Ubuntu apt update stalled; curl could not resolve archive.ubuntu.com.
Direct query to OPNsense 10.10.10.1 returned SERVFAIL despite working IP/ICMP
connectivity to the gateway, DC01 and public IPv4.

### Fix and Validation

A temporary update-only resolvectl override used Cloudflare 1.1.1.1/1.0.0.1,
then reverted the runtime override. Permanent remediation set those upstreams,
disabled WAN DHCP DNS override, restricted Unbound listening/query access to the
lab LAN and used all outgoing interfaces. External names resolved through both
OPNsense and DC01.

The inherited record associated the failure with WAN DHCP DNS; the failed
upstream's exact behavior was not independently captured. Treat the resolver
path as the diagnosed fault domain, not proof of every upstream causal detail.

### Current Design and Prevention

On 2026-11-14, SIEM01 DNS became persistent via OPNsense, with lab.internal
Query Forwarding to DC01. External resolution no longer depends on DC01 uptime.
Keep temporary troubleshooting overrides separate from this permanent design.
See [OPNsense](05_OPNsense.md).

## DC01 Data Components Used the Wrong Elasticsearch Output

**Date:** 2026-11-14. **Status:** resolved at component-health level;
controlled ingestion still pending.

### Symptoms and Cause

Fleet check-in was Healthy, but DC01 data components were Degraded and attempted
https://127.0.0.1:9200. Loopback on DC01 is not Elasticsearch on SIEM01.
The default output was managed through xpack.fleet.outputs in kibana.yml.

### Fix and Validation

Elasticsearch was made reachable on the internal SIEM01 interface; the shared
output became https://10.10.10.20:9200 with TLS trust retained. Elasticsearch and
Kibana were restarted, followed by DC01's Elastic Agent service. DC01 applied
policy revision 3 and both agents became Healthy. A credential-free HTTP 401
confirmed reachability/authentication enforcement, not authenticated access or
end-to-end ingestion. No WAN NAT rule for Elasticsearch was added.

### Prevention

Remote agents need an internal Elasticsearch address reachable from their own
host. Local bootstrap and shared data-output addresses have different scopes.
Keep 9200/9300 off WAN NAT and validate an actual source event separately.

## Earlier Fleet Bootstrap Recovery and Rebuild

**Date:** 2026-11-14, before the final duplicate-input diagnosis.
**Status:** historical interim mitigations, superseded by the final recovery above.

The inherited implementation record documents an initial Fleet Server re-enrollment,
then a later clean SIEM01 Agent/Fleet Server rebuild retaining Elasticsearch,
Kibana, policies and DC01. It also records the Elasticsearch readiness script and
systemd drop-in, restart-on-failure behavior and manual DC01 service restart
during that earlier recovery. Healthy results were obtained, but automatic reboot
recovery had not yet passed. Their exact earlier root cause was not established.

These are distinct historical actions, not the final fix and not evidence that
all prior failures had the same duplicate-input cause. Keep the recorded startup
guard, but do not infer that a startup race was confirmed.
Paths and current behavior are summarized in [SIEM and Telemetry](07_SIEM_and_Telemetry.md).

## Reusable Read-only Diagnostic Checklist

On SIEM01, before intervention:

~~~bash
date -Is
uptime
systemctl is-active elasticsearch kibana elastic-agent
systemctl show elastic-agent -p ActiveState -p SubState -p Result -p ExecMainStatus -p NRestarts
free -h
df -h /
sudo ss -lnt '( sport = :9200 or sport = :8220 or sport = :8221 )'
sudo /opt/Elastic/Agent/elastic-agent status --output full
curl -k -sS --max-time 10 https://127.0.0.1:8221/api/status
~~~

-k is for this diagnostic probe only: it does not validate TLS trust or justify
weakening configured verification. Filter individual event timestamps rather than
log-file modification times. Review only sanitized errors; do not paste raw logs,
service configuration, diagnostics bundles, tokens or keys into public records.

## Case Template

- Symptoms and status, environment and time range.
- Investigation commands and sanitized evidence.
- Confirmed cause, or explicitly unconfirmed hypotheses.
- Fix, independent validation, limitations and remaining work.
- Prevention and related notes.

## Related Notes

- [Current Lab State](12_Current_Lab_State.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Implementation Log](09_Implementation_Log.md)
- [Lessons Learned](11_Lessons_learned.md)
- [Documentation Review](13_Documentation_Review.md)
