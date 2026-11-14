---
title: OPNsense
tags: [blue-team, firewall, networking]
status: active
updated: 2026-11-14
---

# OPNsense

## Purpose and Status

VM 102 is the active gateway for the isolated lab and a planned firewall-log
source. It was inherited as a preserved VM, initially powered off, then activated
for Phase A1. The inherited record documents a ZFS installation and configuration backup.

## Confirmed Baseline

| Interface | Network | Recorded state |
| --- | --- | --- |
| WAN / vtnet0 | vmbr0, DHCP from home router | Active uplink; 192.168.1.233/23 last observed, not a static assignment |
| LAN / vtnet1 | vmbr1, 10.10.10.1/24 | Gateway reachable from DC01 and SIEM01 |

IP/ICMP egress was verified from SIEM01. Firewall rule isolation from the home
LAN and persistent gateway boot should be reviewed as distinct controls, not
inferred solely from Internet connectivity.

## Current DNS Routing - 2026-11-14

SIEM01 persistently uses Unbound at 10.10.10.1 with lab.internal search domain.
Unbound Query Forwarding sends lab.internal to DC01 DNS at 10.10.10.10:53.
Other lookups use the configured external resolution path.

~~~text
SIEM01 -> OPNsense Unbound (10.10.10.1)
          |- lab.internal -> DC01 DNS (10.10.10.10)
          |- external names -> configured upstream resolvers
~~~

- Upstream resolvers: 1.1.1.1 and 1.0.0.1; WAN DHCP DNS override disabled.
- Unbound listens only on lab LAN, queries restricted to 10.10.10.0/24;
  all outgoing interfaces were configured.
- DC01 uses its own AD DNS, with external forwarding to OPNsense.
- External resolution was validated with DC01 off; AD names then timed out by
  design. The previous SIEM01 -> DC01 -> OPNsense chain is historical, not current.

## Restricted Administration

Kibana TCP 5601 is DNATed to 10.10.10.20:5601 only for the trusted administration
workstation (192.168.1.6). Do not create WAN NAT for Elasticsearch 9200/9300,
Fleet 8220/8221 or the OPNsense management interface. No home-router Internet
port-forward is part of the lab design.

## Historical DNS Incident - Resolved 2026-11-11

Unbound returned SERVFAIL for archive.ubuntu.com despite working IP connectivity,
blocking Ubuntu updates. A temporary runtime Cloudflare DNS override was used for
updates and reverted. Setting upstream resolvers, disabling WAN DNS override and
restricting Unbound restored external lookup through OPNsense and DC01.
The deliberate SIEM DNS resilience change followed on 14 November.
Evidence and cause limitations: [Troubleshooting](10_Troubleshooting.md#siem01-external-dns-servfail).

## Remaining Work

- Validate lab-to-home isolation and necessary egress firewall rules.
- Confirm DHCP requirements/operation before client deployment; do not infer it
  was validated from the static DC01/SIEM01 setup.
- Integrate firewall logs into the SIEM and validate source timestamps/queries.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)
- [Implementation Log](09_Implementation_Log.md)
- [TODO](../TODO.md)
