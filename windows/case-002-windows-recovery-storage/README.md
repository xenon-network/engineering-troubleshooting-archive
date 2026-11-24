# Case 002: Windows Recovery Blocked by Storage Failures and Boot Media Problems

**Category:** Windows System Recovery / Hardware Troubleshooting / Storage / Boot Configuration
**Status:** Unresolved in the available historical record
**Platform:** Windows 10 → attempted Windows 11 recovery/installation
**Incident type:** Multi-layer troubleshooting

## 1. Problem Overview

The main desktop computer experienced a Windows system failure during troubleshooting of a corrupted Windows Defender Firewall service.

Attempts to repair the existing Windows installation and recover the operating system were unsuccessful. The system entered boot/recovery loops, and attempts to create installation media using an older HDD and USB–SATA adapter repeatedly failed with Input/Output errors.

The investigation eventually identified serious SMART errors on the existing system SSD. Additional difficulties involved the USB–SATA adapter, boot firmware, installation media, and the availability of usable storage devices.

The objective was to restore a functional Windows installation while preserving data on other drives.

## 2. Hardware Environment

### Main PC — XeSPC

- Motherboard: ASRock B660M-HDV
- CPU: Intel Core i5-12400F
- RAM: 16 GB DDR4
- PSU: Cougar XTC 650W

### Storage devices

| Device        | Model                  | Capacity | Intended role                                | Status during investigation          |
| ------------- | ---------------------- | -------: | -------------------------------------------- | ------------------------------------ |
| SATA3_0       | Kingston SUV400S37120G |   120 GB | System drive                                 | Critical SMART errors                |
| SATA3_1       | WDC WD10SPZX           |     1 TB | Data storage                                 | Data preservation priority           |
| M2_1          | Apacer AS2280P4        |   512 GB | Data storage / potential installation target | Intended to preserve existing data   |
| External SATA | WDC WD3200BPVT         |   320 GB | Temporary installation media                 | SMART passed, but ISO writing failed |

### Secondary computer

- Acer Aspire E1-531
- Ubuntu environment used for disk diagnostics and installation-media preparation
- USB–SATA adapter: AgeStar
- Adapter connection: USB 2.0
- Adapter power circuitry raised safety concerns during investigation.

## 3. Initial Symptoms

The original Windows installation experienced several problems:

- Windows Defender Firewall service (`mpssvc`) failed to start.
- The service was reported in abnormal states, including `START_PENDING`.
- Windows recovery and reset operations failed.
- Attempts to upgrade or reinstall Windows resulted in boot or OOBE loops.
- Recovery error `0xc000000e` appeared during boot attempts.
- The system could not reliably boot from external installation media connected through the AgeStar adapter.

The Firewall issue was the initial trigger for the recovery process, but the investigation expanded into operating-system recovery and storage diagnostics.

## 4. Recovery Attempts

The following operations were attempted during the investigation.

### Windows system repair

- `sfc /scannow`
- `DISM /Online /Cleanup-Image /RestoreHealth`
- `netsh advfirewall reset`
- `netsh int ip reset`
- `netsh winsock reset`

These attempts did not restore the Windows installation to a working state.

### Firewall service recovery

The investigation included:

- Manual recreation of the `mpssvc` service.
- Modification of service registry entries.
- Attempts to correct service dependencies.
- Changes to service security configuration.
- Registry cleanup through recovery environments.

The original Firewall problem remained unresolved.

### Windows recovery

Attempts included:

- Reset this PC using local recovery.
- Reset this PC using cloud recovery.
- Windows 11 Installation Assistant.
- Windows installation ISO.
- OOBE bypass attempts.
- Recovery environment troubleshooting.

Several attempts resulted in reboot loops or unsuccessful recovery.

## 5. Boot Media and Firmware Troubleshooting

The external 320 GB HDD was used as an attempted Windows installation medium.

The HDD was connected to the Acer laptop using the AgeStar USB–SATA adapter.

### Observed problems

- The desktop motherboard did not consistently present the USB-connected HDD as a bootable device.
- The drive was visible in some firmware/storage screens but absent from the expected boot menu.
- Changing CSM and boot settings did not produce a reliable boot result.
- Attempts to boot the prepared installation media resulted in errors or returned to the recovery screen.

The investigation considered possible compatibility problems between the USB–SATA bridge, the motherboard firmware, and the boot format.

These explanations were hypotheses and were not conclusively proven.

## 6. Storage Diagnostics

### Kingston SSD — SUV400S37120G

SMART information showed serious warning signs despite the overall SMART health result reporting `PASSED`.

Recorded values included:

| SMART attribute             |     Recorded value |
| --------------------------- | -----------------: |
| Reallocated Sector Count    |                 38 |
| Reported Uncorrectable      |               1173 |
| ATA Error Count             | Approximately 1172 |
| Power-On Hours              |             52,522 |
| Unexpected Power Loss Count |                466 |
| SSD Life Left               |                 55 |

The SMART error log also contained uncorrectable read errors.

**Assessment:** The drive was not considered reliable for a new operating-system installation or critical data storage.

The overall SMART `PASSED` status did not override the detailed error indicators.

### WDC WD3200BPVT — 320 GB HDD

Recorded diagnostics:

- SMART overall-health: `PASSED`
- Reallocated sectors: 0
- Current pending sectors: 0
- A read test initially achieved approximately 35 MB/s.

However, subsequent attempts to write the Windows ISO to this drive repeatedly failed with Input/Output errors.

This created an important distinction:

> A drive can pass basic SMART checks and read tests while a specific write operation still fails.

The exact cause of the write failures was not conclusively identified.

## 7. Installation Media Write Failures

The Windows ISO was approximately 5.5 GB.

Writing it to the external HDD was attempted using:

- Ubuntu Disks — Restore Disk Image
- `dd` from the Linux terminal

### Recorded failures

One `dd` attempt reported:

```text
1839202304 bytes copied
dd: Input/output error
```

Another attempt failed during image restoration in Ubuntu Disks.

The ISO was therefore not written completely.

A partial image cannot be considered valid installation media.

The failure could have involved the drive, adapter, connection, power delivery, or another hardware/transport issue. The available evidence did not establish one definitive cause.

## 8. USB–SATA Adapter Safety Investigation

The AgeStar adapter raised concerns about its power circuitry, including a Schottky diode identified during the investigation.

The adapter was being used under a cautious rule:

- USB-only power for smaller 2.5-inch drives.
- Avoid using the external power supply until the suspected circuitry issue was investigated.

The Power LED also behaved differently depending on the connected device.

The meaning of the LED colors was not independently verified through manufacturer documentation or electrical measurements. Therefore, the LED behavior should not be treated as a definitive voltage measurement.

The decision was made to replace the adapter rather than rely on a potentially unsafe or unstable device for further installation work.

## 9. Root Cause Assessment

### Confirmed findings

- The existing Kingston system SSD had serious SMART error indicators.
- The Windows installation was not successfully recovered through the attempted repair procedures.
- The external 320 GB HDD's ISO-writing attempts failed.
- The Windows installation media was incomplete after the failed writes.
- The boot process did not reliably recognize or start the prepared external installation media.

### Unconfirmed hypotheses

- USB–SATA adapter instability.
- Power-delivery problems.
- Adapter/controller compatibility.
- Firmware boot compatibility.
- Storage write failures not detected by basic SMART status.

No single cause was proven for every failure.

The incident involved several overlapping problems, and it would be inaccurate to attribute all failures to one component.

## 10. Final Decision and Recovery Plan

The investigation concluded that continuing to use the Kingston SSD as the Windows system drive was not a reliable option.

A new installation approach was selected:

1. Obtain a replacement 2.5-inch SATA SSD.
2. Obtain a dedicated USB flash drive for Windows installation media.
3. Create installation media using a more reliable method.
4. Replace the faulty Kingston SSD in the main PC.
5. Install Windows on the new system drive.
6. Keep the 1 TB HDD and 512 GB M.2 drive untouched during installation.
7. Verify the system boot process and storage health after installation.

At the end of the recorded investigation, new hardware had reportedly been ordered. The available history does not confirm that the replacement hardware arrived or that Windows installation was successfully completed.

## 11. Lessons Learned

### 1. SMART overall status is not sufficient

A drive reporting `PASSED` can still contain serious indicators of degradation.

Individual SMART attributes and error logs must be examined.

### 2. Read and write reliability are different

A successful read test does not prove that a drive can reliably perform a large sustained write operation.

### 3. Installation media must be verified

A partially written ISO is not a valid recovery solution.

Do not proceed to installation until the media creation process completes successfully.

### 4. Separate software and hardware troubleshooting

Repeated Windows repair commands were unlikely to resolve problems caused by a failing storage device or unreliable data connection.

### 5. Preserve data before recovery

When multiple drives are installed, identify them clearly and protect data drives from accidental formatting or partition deletion.

### 6. Do not trust assumptions about firmware

A drive being visible in one BIOS screen does not necessarily mean that it is available as a bootable device.

Boot compatibility must be verified independently.

### 7. Stop repeating failed procedures

After repeated failures with the same hardware configuration, change the diagnostic approach instead of repeating the same write or recovery operation.

## 12. AI-Assisted Troubleshooting Review

AI was used throughout the investigation to interpret errors, suggest recovery procedures, analyze SMART values, and propose alternative installation methods.

The investigation also demonstrated limitations of AI-assisted troubleshooting:

- Several suggested procedures were repeated despite previous failures.
- Some diagnoses were presented with excessive certainty before sufficient evidence was available.
- Hardware explanations were sometimes inferred without measurements.
- The available hardware inventory was not always considered early enough.
- Some proposed steps were impractical given the user's physical equipment and constraints.

A better troubleshooting process would have been:

1. Inventory all hardware and available installation media.
2. Collect SMART diagnostics from all relevant storage devices.
3. Identify the most reliable available target and installation medium.
4. Protect all data drives.
5. Choose one recovery path based on confirmed evidence.
6. Change the approach when a repeatable failure occurs.

## 13. Conclusion

This incident was not simply a Windows repair problem.

It became a combined investigation involving:

- Windows system recovery.
- Storage diagnostics.
- Boot configuration.
- Firmware compatibility.
- USB–SATA bridge behavior.
- Installation media reliability.
- Hardware safety.
- Data preservation.

The main engineering lesson is that troubleshooting should follow evidence across layers. When operating-system recovery repeatedly fails, hardware health and the recovery environment itself must be investigated before continuing with software repair procedures.

**Final status: Unresolved in the available history. Replacement storage and installation media were selected as the next recovery step, but the final installation result was not documented.**
