# Case 006: Home Router Security Hardening and Exposure Assessment

**Investigation Date:** June 2026

## Overview

This case documents a security review and configuration hardening investigation of a TP-Link Archer C6 home router.

The investigation focused on reducing unnecessary exposure of the router's management interface, reviewing wireless security settings, assessing UPnP and port forwarding, examining remote management features, and understanding the risks associated with devices accessible through the local network.

The review also covered a Home Media Server configuration used for streaming video to a television and remote management of ASIC mining devices.

The router was partially hardened. Several security settings were changed successfully, while some configuration questions and potential risks remained unresolved.

---

## Objectives

- Review the router's administrative access security.
- Disable unnecessary remote management services.
- Review wireless security and encryption settings.
- Investigate UPnP and port forwarding.
- Reduce unnecessary internet-facing services.
- Examine risks associated with local network devices.
- Review firmware and system management settings.
- Understand the security trade-offs between convenience and network exposure.

---

## Environment

| Component | Description |
|---|---|
| Router | TP-Link Archer C6 |
| Hardware version | V2.0 |
| Firmware | 1.3.3 Build 20230808 rel.60959 |
| Management interface | Web GUI |
| Administrative access | HTTPS |
| Local network | Private IPv4 LAN |
| Wireless security | WPA2-PSK |
| Connected devices | PCs, television, ASIC miners, smartphones |
| Media server | Home Media Server (HMS) |

All IP addresses and infrastructure identifiers have been anonymized.

---

## 1. Router Administrative Access

The router's administrative interface was reviewed to reduce exposure to unauthorized access.

### Configuration reviewed

| Setting | Status |
|---|---|
| Administrator password | Strong password configured |
| HTTPS management | Enabled |
| HTTP management | Disabled |
| Remote Management | Disabled |
| Management access from WAN | Disabled |
| HTTPS management port | Non-default configured port |

The router was accessed through its local management interface using HTTPS.

### Security considerations

- Strong administrative credentials reduce the risk of password guessing.
- HTTPS protects the management session against plaintext transmission.
- Disabling remote management reduces the router's internet-facing administrative attack surface.
- Management access should remain limited to trusted devices and networks.

### Result

The router's administrative access was configured to use HTTPS, and remote management was disabled.

---

## 2. Wireless Security Review

Both wireless bands were reviewed for security and performance-related settings.

### Security settings

| Setting | Status |
|---|---|
| WPA2-PSK | Configured |
| WPS | Disabled |
| Group Key Update — 2.4 GHz | Reviewed |
| Group Key Update — 5 GHz | Corrected |
| Guest Network | Disabled |
| WDS | Disabled |
| AP Isolation | Enabled during review |

### Group Key Update Period

The 5 GHz configuration initially showed a Group Key Update Period of `0`.

This setting was identified as requiring correction because a value of zero may disable periodic group-key rotation, depending on the router's implementation.

The value was changed to:

```text
3600 seconds
```

This corresponds to a one-hour update interval.

### Result

The user confirmed that the 5 GHz Group Key Update Period was corrected.

---

## 3. WPS and Wireless Attack Surface

WPS was reviewed and found disabled.

Disabling WPS removes an additional wireless enrollment mechanism that may introduce security weaknesses depending on the implementation.

The wireless network continued to use WPA2-PSK authentication.

The investigation did not include a wireless penetration test, password-strength audit, or packet-capture-based assessment of the wireless authentication process.

---

## 4. WAN Ping and DoS Protection

The router's DoS Protection settings were reviewed.

### Configuration

| Setting | Status |
|---|---|
| DoS Protection | Medium |
| WAN Ping Ignore | Enabled |
| LAN Ping Block | Not enabled |

WAN Ping Ignore was enabled to prevent the router from responding to ping requests originating from the WAN interface, where supported by the router.

This does not prevent all forms of network discovery or guarantee that the router is invisible to internet scanning.

### Important distinction

WAN Ping Ignore affects WAN-side behavior. Devices on the local network may still be able to ping the router.

---

## 5. UPnP Investigation

UPnP was identified as an important security and operational consideration.

The Home Media Server application used UPnP functionality for network-related services.

### Risk assessment

UPnP allows applications and devices on the local network to request automatic changes to router port mappings.

Potential risks include:

- Unintended exposure of internal services to the internet.
- Malware on an internal device requesting port mappings.
- Difficulty maintaining a clear inventory of externally accessible services.
- Increased attack surface without explicit administrator approval for each mapping.

### Initial configuration

UPnP was enabled because it was being used by the Home Media Server environment.

The investigation considered disabling UPnP and replacing automatic port mapping with manually configured port forwarding.

However, the final state of UPnP was not conclusively confirmed in the available records.

### Lesson

UPnP should be enabled only when its functionality is required and the resulting port mappings are understood.

---

## 6. Port Forwarding and Home Media Server

The Home Media Server was running on a Windows PC and used for local media streaming to a television.

The router's Virtual Servers / Port Forwarding configuration was reviewed.

An existing or proposed rule involved port 8080.

### Example of the reviewed rule

| Field | Value |
|---|---|
| Service | Home Media Server |
| External port | 8080 |
| Internal port | 8080 |
| Internal host | Windows PC |
| Protocol | TCP/UDP or ALL, as configured |

The rule was considered unnecessary because remote access to the media server from the internet was not required.

### Decision

The port forwarding rule was identified for removal.

The security reasoning was:

- No external access requirement existed.
- A port-forwarding rule could expose a service on the internal Windows PC.
- Internet scanning could identify an externally reachable service.
- The rule increased exposure without providing a required function.

### Result

The investigation concluded that unnecessary port forwarding should be removed.

The exact final state of the rule was not independently verified in the available documentation.

---

## 7. Remote ASIC Management

Several ASIC mining devices were accessible through their web interfaces on the local network.

The investigation identified a security concern: some miner interfaces were accessible through HTTP without authentication.

### Risks

- HTTP does not encrypt communication.
- Unauthenticated management interfaces may permit unauthorized configuration changes.
- A compromised local device could potentially interact with exposed management interfaces.
- Exposing miner management ports through port forwarding would increase risk substantially.

### Remote access approach

Direct internet exposure of miner web interfaces was rejected as an unsafe approach.

A VPN-based remote access architecture was considered preferable.

The intended security model was:

```text
Remote Phone
     |
     | Encrypted VPN
     |
Home Network
     |
     +---- Router
     |
     +---- ASIC Management Interface
```

The VPN approach was not fully implemented as part of this case.

### Result

Remote management through direct port forwarding was not adopted.

The miners remained part of the local network security review.

---

## 8. CWMP / TR-069 Management

CWMP/TR-069 configuration was reviewed as part of the router's management-plane security assessment.

The investigation identified a concern regarding the ACS credentials.

### Finding

The ACS credentials had not been changed or disabled at the time of the review.

The available records did not confirm whether the service was required by the ISP or whether disabling it would affect provider support or configuration management.

### Recommended investigation

- Determine whether the ISP requires CWMP.
- Confirm the ACS endpoint and management requirements.
- Change default credentials if the feature is retained and the router allows it.
- Disable the feature only after confirming it is not required.

### Status

Unresolved.

This finding was retained rather than incorrectly marked as fixed.

---

## 9. Firmware Review

The router's firmware update interface was reviewed.

The router reported that its installed firmware did not require an update.

The installed version was recorded as:

```text
1.3.3 Build 20230808 rel.60959
```

The investigation did not identify a confirmed firmware update available through the router's built-in update function.

### Security consideration

Firmware status should be verified periodically using the manufacturer's official support information for the exact hardware revision.

An automatic update message alone does not establish that the device is free from known vulnerabilities.

---

## 10. Other Configuration Considerations

Several additional settings were reviewed.

| Setting | Decision / Status |
|---|---|
| SPI Firewall | Reviewed as a potential security feature |
| DoS Protection | Medium |
| ALG settings | Unnecessary ALGs were left disabled |
| DDNS | Not required for the current use case |
| VPN server | Not configured as part of this case |
| Port forwarding | Reviewed for unnecessary rules |
| NTP / time settings | Reviewed |
| Guest network | Disabled |
| WDS | Disabled |

The review emphasized avoiding features that were not required for the current network functionality.

---

## 11. Troubleshooting and Challenges

### Challenge 1 — Balancing security with media streaming

**Problem:** Disabling UPnP could affect the existing Home Media Server configuration.

**Investigation:** The media server's networking features and router port mapping were reviewed.

**Decision:** The preferred direction was to avoid unnecessary automatic port mapping and use explicitly configured rules only when required.

**Outcome:** A complete replacement of UPnP functionality was not confirmed.

---

### Challenge 2 — Port forwarding without a remote access requirement

**Problem:** A port forwarding rule existed or was being considered for the media server despite no requirement for internet access.

**Investigation:** The rule was reviewed against the intended use of the application.

**Decision:** The rule was deemed unnecessary and identified for removal.

**Lesson:** A working configuration is not necessarily a secure or necessary configuration.

---

### Challenge 3 — Router management security

**Problem:** The router's management interface and remote management options could increase exposure.

**Investigation:** Administrative access, HTTPS, HTTP, and Remote Management settings were reviewed.

**Action:** Remote Management was disabled and HTTPS was retained for local administration.

**Outcome:** The administrative attack surface was reduced.

---

### Challenge 4 — Group key update interval

**Problem:** The 5 GHz wireless configuration showed a Group Key Update Period of zero.

**Investigation:** The setting was reviewed in the router's wireless security configuration.

**Action:** The period was changed to 3600 seconds.

**Outcome:** The user confirmed the change.

---

### Challenge 5 — Unresolved CWMP credentials

**Problem:** CWMP configuration included credentials that had not been changed.

**Investigation:** The potential role of ISP remote management was considered.

**Outcome:** The issue remained unresolved because the necessity of the service had not been established.

**Lesson:** Security hardening should not disable provider-managed services blindly; first identify their function and dependencies.

---

## 12. Final Security Status

| Security Control | Status |
|---|---|
| Strong router administrator password | Configured |
| HTTPS administrative access | Enabled |
| HTTP administrative access | Disabled |
| Remote Management | Disabled |
| WPS | Disabled |
| 5 GHz Group Key Update Period | Corrected to 3600 seconds |
| WAN Ping Ignore | Enabled |
| DoS Protection | Medium |
| UPnP | Reviewed; final status unconfirmed |
| Port forwarding for HMS | Identified as unnecessary; final removal unconfirmed |
| CWMP credentials | Unresolved |
| Firmware | Router reported no update required |
| Direct remote ASIC port exposure | Not adopted |
| VPN remote management | Planned, not implemented |

---

## 13. Lessons Learned

1. Secure management interfaces before considering additional remote access functionality.
2. HTTPS protects administrative sessions against plaintext transmission but does not replace strong authentication.
3. Disabling remote management reduces exposure when remote administration is unnecessary.
4. UPnP introduces convenience but can create automatic port mappings that are difficult to audit.
5. Port forwarding should be tied to an explicit requirement.
6. Internal services should not be exposed to the internet merely because they function locally.
7. Unauthenticated HTTP management interfaces on IoT or mining devices represent a local network risk.
8. VPN-based remote access is preferable to exposing individual device management interfaces directly.
9. Router firmware and ISP management features should be reviewed carefully.
10. Security hardening must account for dependencies such as media streaming and ISP remote management.
11. Every configuration change should be verified, not simply assumed successful.
12. Unresolved findings should remain documented until evidence confirms remediation.

---

## Conclusion

The investigation improved the security posture of the TP-Link Archer C6 by reviewing administrative access, wireless security, WAN exposure, and unnecessary network services.

Several configuration changes were confirmed, including disabling remote management and WPS, retaining HTTPS administration, correcting the 5 GHz Group Key Update Period, and enabling WAN Ping Ignore.

Other findings remained unresolved, particularly UPnP configuration, the final state of the Home Media Server port forwarding rule, and CWMP credentials.

The case demonstrated the importance of reviewing both router configuration and the security implications of devices and applications connected to the local network.

---

## Future Work

- Verify the final UPnP state.
- Confirm removal of unnecessary port forwarding.
- Investigate CWMP requirements with the ISP.
- Review the router's firmware against official vendor information.
- Implement secure remote access using a VPN.
- Improve local network segmentation for IoT and mining devices.
- Review miner web interface authentication and HTTP exposure.
- Document the router's active port mappings.
- Periodically repeat the configuration review.

---

*Documentation note: This case reflects the recorded investigation and confirmed changes. Unverified configuration states and unresolved findings are explicitly identified rather than presented as completed remediation.*
