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

The scan reported:
Windows Resource Protection found corrupt files
and successfully repaired them.

However, repairing system files did not resolve
the Firewall service startup problem.
DISM
The following repair command was attempted:
DISM /Online /Cleanup-Image /RestoreHealth


The issue remained after the repair attempt.

6. Service Investigation
The Firewall service was found to be missing from
the service management interface during part of
the investigation.

Service registration was examined using:
sc query mpssvc
sc qc mpssvc


The service was manually recreated during
troubleshooting.

After recreation, the service appeared in the
service database but failed to start normally.

Observed state included:
STATE: 2 START_PENDING


The service remained stuck in a pending startup state.

7. Dependency Investigation
The following services were examined:
sc query BFE
sc query RpcSs
sc query netprofm
sc query Dhcp


The investigation included checking whether
required services were running and examining
the Firewall service configuration.

The available evidence did not establish
a definitive dependency-related root cause.

8. Firewall Configuration Attempts
Firewall configuration was attempted using:
netsh advfirewall reset


and:
netsh advfirewall set allprofiles state on


These operations failed because Windows could
not communicate successfully with the Firewall service.

The graphical Firewall interface was also examined.

9. Recovery Attempts
The investigation included several recovery approaches:
- Service configuration changes
- Service recreation
- Dependency configuration
- Service account configuration
- Windows system integrity repair
- DISM repair
- System Restore
- Restart and shutdown procedures
- Windows Recovery Environment investigation
Some recovery attempts were unsuccessful.

The exact result of every recovery procedure
was not consistently preserved.

10. Unsafe Troubleshooting Incident
During troubleshooting, a command intended to
terminate a process was executed:
taskkill /f /im svchost.exe


This command terminated a critical Windows
service-host process and resulted in a
Blue Screen of Death (BSOD) and system reboot.

This was an unsafe troubleshooting action.
Lesson
Processes such as svchost.exe host multiple
Windows services and must not be terminated
indiscriminately.

Before executing process termination commands:
- Identify the exact target process
- Determine which services it hosts
- Verify the scope of the command
- Understand possible system consequences
- Avoid broad process termination commands
This incident highlighted the importance of
independently validating AI-generated commands
before executing them.

11. Results
Confirmed observations
- Windows system file corruption was detected.
- SFC reported successful repair of corrupted files.
- The Firewall service was missing during part
  of the investigation.
- The service was manually recreated.
- The service subsequently remained in a
  pending startup state.
- Firewall management commands continued
  to fail.
- A troubleshooting command resulted in
  a BSOD and reboot.
Unconfirmed conclusions
The investigation did not establish whether
the underlying cause was:
- Service registry corruption
- System component corruption
- Driver-related failure
- Service dependency failure
- Permissions or service account configuration
- A combination of system-level issues
No single root cause was confirmed.

12. Final Status
Unresolved.

A Windows repair installation was considered
as a potential recovery approach.

The available investigation record does not
confirm that the repair installation was
completed or that it resolved the issue.

13. Lessons Learned
1. A service appearing in the service database
   does not necessarily mean it can start successfully.
2. System integrity repairs may not resolve
   every service configuration or runtime problem.
3. Service dependencies should be investigated
   using actual system evidence rather than assumptions.
4. Recreating system services manually can
   introduce additional configuration problems.
5. Broad process termination commands can
   cause serious system instability.
6. AI-generated troubleshooting instructions
   must be evaluated before execution.
7. Troubleshooting should have clear stopping
   conditions when the risk of further changes
   exceeds the potential benefit.
8. An unresolved investigation can still provide
   useful technical knowledge when evidence,
   experiments, and limitations are documented.

14. Further Investigation
Potential next steps, if the investigation
were resumed:
- Collect relevant Service Control Manager events
- Inspect service configuration and dependencies
- Review system integrity and component repair logs
- Compare service configuration against a
  healthy system with the same Windows version
- Investigate driver and filtering-platform state
- Consider a Windows repair installation
- Verify SMB functionality after system recovery
These steps were not all completed and should
not be interpreted as confirmed findings.

15. AI-Assisted Troubleshooting
AI was used throughout the investigation to:
- Generate troubleshooting hypotheses
- Suggest commands
- Interpret error messages
- Propose service recovery procedures
- Explore possible causes
Some suggestions were ineffective, overly
confident, or unsafe.

This case demonstrates both the usefulness
and limitations of AI-assisted technical
troubleshooting.

AI recommendations should be verified,
especially when they involve system services,
registry modifications, drivers, permissions,
or process termination.

Disclaimer
This document is a historical troubleshooting
case study.

It is not a universal Windows repair guide.

Some procedures attempted during the original
investigation were unsuccessful or unsafe.

Do not reproduce commands involving service
recreation, registry modification, or process
termination without understanding their
specific consequences.
