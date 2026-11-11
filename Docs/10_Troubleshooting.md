---
title: Troubleshooting
tags: [blue-team, troubleshooting]
status: active
---

# Troubleshooting

## Case Template

### Issue: Title

#### Symptoms

- What failed and what was observed?

#### Environment

- Host, service, network and timestamp.

#### Investigation

- Commands, logs, queries and evidence reviewed.

#### Root Cause

- Confirmed cause, or explicitly state that it is not yet known.

#### Fix and Validation

- Change applied and evidence that it resolved the issue.

#### Prevention

- Reusable control, detection or operating lesson.

---

## Issue: DC01 could not resolve its configured NTP peer — Resolved

### Symptoms

- **Microsoft-Windows-Time-Service** recorded warning Event ID `134` at approximately `2026-11-09 14:00`.
- `NtpClient` could not use `time.windows.com,0x8` as a manual peer because DNS resolution returned `0x80072AF9` (*No such host is known*).
- The server previously reported `Local CMOS Clock` as its time source.

### Environment

- `DC01.lab.internal`, Windows Server 2022
- First domain controller and DNS server for `lab.internal`
- DC IPv4: `10.10.10.10`; OPNsense gateway and DNS forwarder: `10.10.10.1`

### Investigation

- AD domain discovery succeeded.
- `dcdiag /test:dns /v` passed after DNS client configuration was changed to use local DNS (`10.10.10.10`).
- DNS forwarding through OPNsense `10.10.10.1` was present and `Resolve-DnsName time.windows.com -Server 10.10.10.10` returned a valid CNAME and IPv4 address.

### Root Cause

- A transient post-promotion DNS-resolution failure prevented Windows Time from resolving its manually configured NTP peer. The DNS client and forwarding path were then validated.

### Fix and Validation

- Retained OPNsense `10.10.10.1` as the DNS forwarder on the DC DNS service.
- Reconfigured Windows Time to use `time.windows.com,0x8`, restarted the service and resynchronized it.
- `dcdiag /test:dns /v` passed. Event ID `134` is retained as historical evidence of the initial condition.

### Prevention

- On a first DC, configure the NIC to use only its own DNS address after promotion. Configure upstream resolution as a DNS forwarder, not as an alternate NIC DNS server.

---

## Issue: SIEM01 cannot resolve external Ubuntu repositories — Resolved

### Symptoms

- `apt update` stalled while connecting to Ubuntu repositories.
- `curl` reported it could not resolve `archive.ubuntu.com`.
- `nslookup archive.ubuntu.com 10.10.10.1` returned `SERVFAIL`.

### Environment

- `SIEM01` (`10.10.10.20/24`) on `vmbr1`
- DNS design: SIEM01 → DC01 (`10.10.10.10`) → OPNsense Unbound (`10.10.10.1`)

### Investigation

- ICMP connectivity to OPNsense, DC01 and public IPv4 was successful.
- The failure is DNS resolution through OPNsense, not IP addressing or basic routing.

### Temporary Workaround

```bash
sudo resolvectl dns ens18 1.1.1.1 1.0.0.1
sudo resolvectl flush-caches
sudo apt update
# After updates:
sudo resolvectl revert ens18
```

### Permanent Fix

- Repair OPNsense Unbound upstream resolution, test external lookup at `10.10.10.1`, then confirm resolution through `10.10.10.10`.

