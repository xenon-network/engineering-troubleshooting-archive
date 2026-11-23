# Case 001: Windows Defender Firewall Service Failure

## Status

**Unresolved — system-level recovery investigation**

## Summary

This case documents a prolonged investigation into a
Windows Defender Firewall service failure.

The Windows Defender Firewall service (`mpssvc`) was
missing from the Windows Services management console
and could not be started or used to manage firewall
configuration.

The investigation involved Windows service management,
system integrity checks, dependency analysis, registry
configuration, recovery attempts, and troubleshooting
of a Windows SMB file-sharing problem.

The root cause was not conclusively identified.

---

## 1. Problem Description

The Windows Defender Firewall service was unavailable
or malfunctioning.

The issue affected firewall management and contributed
to difficulties accessing a Windows SMB file share
from an Android device over the local network.

Observed symptoms included:

- Windows Defender Firewall missing from `services.msc`
- Service registration problems
- Firewall service startup failures
- Firewall configuration commands failing
- SMB connectivity problems
- Service remaining in a pending startup state

---

## 2. Environment

- Operating System: Windows 10
- Windows image version recorded during investigation:
  10.0.19045.6456
- System language: Russian
- SMB client: Android
- Network environment: Home LAN

Additional hardware and system information was not fully
preserved in the troubleshooting record.

---

## 3. Symptoms and Error Messages

The investigation encountered several errors:

| Error | Context |
|------|---------|
| 1058 | Service startup/configuration failure |
| 1060 | Service not found |
| 3523 | Service startup failure |
| 0x800706d9 | Firewall-related operation failure |
| START_PENDING | Service remained in pending startup state |

The exact meaning and context of each error should be
evaluated independently. The error codes alone did not
establish the underlying root cause.

---

## 4. Initial Investigation

The following areas were investigated:

- Windows service registration
- Windows Defender Firewall service configuration
- Base Filtering Engine (`BFE`)
- Remote Procedure Call (`RpcSs`)
- Service dependencies
- Service account configuration
- Windows system file integrity
- Firewall configuration interface
- SMB connectivity

---

## 5. System Integrity Checks

### System File Checker

The following command was executed:

```cmd
sfc /scannow
