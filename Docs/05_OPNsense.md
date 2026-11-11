---
title: OPNsense
tags: [blue-team, firewall, networking]
status: active
---

# OPNsense

## Purpose

OPNsense VM 102 is the gateway and firewall-log source for the isolated Blue
Team Operations Lab.

## Confirmed Baseline

| Interface | Network | Status |
| --- | --- | --- |
| WAN / `vtnet0` | `vmbr0`, DHCP from home router | Active; Internet ICMP egress verified from SIEM01 |
| LAN / `vtnet1` | `vmbr1`, `10.10.10.1/24` | Active; gateway reachable from DC01 and SIEM01 |

The VM is installed on ZFS, has a preserved configuration backup and was
powered off after initial validation.

## DNS Resolver Incident — Resolved

On 2026-11-11, `SIEM01` successfully reached `10.10.10.1`, `DC01` and public
IPv4 (`8.8.8.8`), but an external DNS lookup through OPNsense returned
`SERVFAIL`:

```text
nslookup archive.ubuntu.com 10.10.10.1
** server can't find archive.ubuntu.com: SERVFAIL
```

The temporary SIEM01 update workaround uses Cloudflare resolvers (`1.1.1.1`,
`1.0.0.1`) at runtime with `resolvectl`; it must be reverted afterwards so the
server again uses `10.10.10.10`.

### Resolution (2026-11-11)

- Configured static upstream resolvers `1.1.1.1` and `1.0.0.1`; disabled WAN DHCP DNS override.
- Configured Unbound to listen only on LAN and use all outgoing interfaces; restricted queries to `10.10.10.0/24`.
- Verified `archive.ubuntu.com` resolves through both OPNsense (`10.10.10.1`) and DC01 (`10.10.10.10`).

### Required permanent remediation

- Repair and validate the Unbound upstream-resolution path on OPNsense.
- Confirm `archive.ubuntu.com` resolves through `10.10.10.1`, then through
  `DC01` (`10.10.10.10`).
- Retain the intended chain: `SIEM01 → DC01 DNS → OPNsense Unbound → Internet`.
## A1 Actions

- Power on VM 102 and verify persistent boot.
- Confirm WAN, LAN and DHCP operation.
- Confirm lab-to-home isolation while retaining required Internet egress.
- Configure firewall-log forwarding or collection for the selected SIEM.

## Related Notes

- [Network Architecture](03_Network_Architecture.md)
- [SIEM and Telemetry](07_SIEM_and_Telemetry.md)


