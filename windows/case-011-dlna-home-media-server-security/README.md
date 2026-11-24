# Case 011: DLNA Home Media Server Security and Network Exposure

## Investigation Date: April 2026


## Overview

This case documents the security investigation of a Home Media Server (HMS) running on Windows and used to stream video content to a DEXP television over the local network.

The investigation focused on DLNA/UPnP media discovery and streaming, HTTP management access, router port forwarding, local network exposure, and the security implications of allowing media devices to communicate across the home network.

The main objective was to retain local media streaming functionality while removing unnecessary external network exposure.

## Environment

### Media Server

- Operating System: Windows
- Application: Home Media Server (Домашний медиа-сервер)
- HTTP management interface: TCP port 8080
- Media sharing protocols: DLNA / UPnP / HTTP

### Client Device

- Device: DEXP television
- Connection: Local home network
- Media access: DLNA media discovery and streaming

### Network Infrastructure

- Router: TP-Link Archer C6
- LAN subnet: `192.168.1.0/24`
- Windows PC: `WINDOWS_PC_IP`
- TV: `TV_IP`
- Router management interface: HTTPS
- Security software: Kaspersky

## Objectives

- Understand how DLNA media streaming works within the local network.
- Determine whether port forwarding is required for local media streaming.
- Investigate the security implications of exposing the HMS HTTP interface.
- Review UPnP and router security settings.
- Remove unnecessary internet-facing access to the media server.
- Understand the difference between local network access and remote internet access.
- Assess the remaining risks from devices inside the home LAN.

## Initial Configuration

The Home Media Server was running on a Windows PC and sharing media with a DEXP television.

The application used DLNA/UPnP for device discovery and media streaming, with HTTP available for management and related functionality.

A Virtual Server / Port Forwarding rule had been configured on the router:

| Parameter | Configuration |
|---|---|
| External port | 8080 |
| Internal destination | Windows PC |
| Internal port | 8080 |
| Protocol | ALL (TCP and UDP) |
| Purpose | HMS access |

The rule potentially allowed incoming internet traffic to reach the Windows PC on the forwarded port.

However, remote access to the media server was not required.

## Investigation

### 1. Understanding DLNA and UPnP

DLNA is used to share and stream media between compatible devices on a local network.

In this environment:

- The Windows PC hosts the media server.
- The DEXP TV discovers the media server.
- Media is streamed over the local network.
- The router provides local network connectivity between devices.

Local media discovery and streaming do not inherently require a port-forwarding rule from the internet.

The distinction between local network communication and externally forwarded traffic was a central part of the investigation.

### 2. Home Media Server Ports and Functionality

The HMS application configuration indicated support for:

- UPnP
- DLNA
- HTTP

The application did not need a built-in port-forwarding feature.

Port forwarding is configured on the router. It redirects incoming traffic from an external network to a device inside the LAN.

The HTTP interface was associated with port 8080, while DLNA/UPnP functionality was used for local media sharing and discovery.

### 3. Reviewing the Port Forwarding Rule

The existing router rule forwarded port 8080 to the Windows PC.

Although the rule was technically configured to direct traffic to the HMS host, it was unnecessary for the intended use case.

The media server was only required to serve the television inside the home network.

Keeping the forwarding rule would unnecessarily expose the PC's listening service to incoming connections from outside the LAN.

### Action Taken

The Virtual Server / Port Forwarding rule for HMS was deleted.

The objective was to remove unnecessary internet-facing access without disabling local DLNA media streaming.

### 4. Testing the Difference Between Local and External Access

The investigation considered the difference between:

- Local access from the television to the Windows PC.
- Access from outside the home network through the router.
- Access to the router's management interface.
- Access to the HMS HTTP interface.

The absence of an internet-facing port-forwarding rule does not mean that the media server becomes inaccessible inside the LAN.

Local devices can still communicate directly over the home network, subject to host firewall rules and network configuration.

## Router Security Review

Several router settings were reviewed as part of the HMS security investigation.

### UPnP

UPnP was disabled on the router.

This was an important security measure because UPnP-enabled applications and devices may request automatic port mappings on the router.

Disabling router UPnP prevents automatic port mapping requests from being accepted through that feature.

It does not disable DLNA media discovery and streaming within the local network by itself.

### CWMP

CWMP was disabled.

This management protocol was reviewed as part of the router's external-management and network security configuration.

### AP Isolation

AP Isolation was enabled on both 2.4 GHz and 5 GHz Wi-Fi configurations.

This setting was reviewed in the context of restricting communication between wireless clients.

However, AP Isolation does not necessarily isolate all devices on the LAN from one another, including devices connected through wired Ethernet or other network paths.

### DoS Protection

DoS Protection was configured to Medium for ICMP, UDP, and TCP-SYN protection.

This setting was considered a router-level mitigation rather than direct protection for the HMS application.

## Security Assessment

### External Exposure

The port-forwarding rule for TCP/UDP port 8080 was unnecessary for the required local streaming functionality.

Removing it reduced the externally reachable attack surface.

Without an active forwarding rule, unsolicited internet traffic would not be intentionally redirected through that rule to the Windows PC.

This does not guarantee that the PC is unreachable through every possible route or mechanism; it removes the specific forwarding configuration that was investigated.

### Local Network Exposure

The HMS remained available to devices within the local network, which was necessary for the television to access media.

This introduced a separate security consideration: devices on the same LAN may be able to reach services hosted on the Windows PC.

Potential risks include:

- Unauthorized media access.
- Access to the HTTP management interface.
- Service enumeration.
- Attempts to exploit vulnerabilities in the media server.
- Access from compromised devices already connected to the LAN.
- Abuse of insecure or unauthenticated media-sharing services.

### DLNA / UPnP Security Limitations

DLNA and related local media-sharing implementations are generally designed for convenience rather than strong security boundaries.

Depending on the implementation, local media discovery and control may lack strong authentication or encryption.

A compromised device or unauthorized LAN client could potentially discover and interact with media services.

Disabling router UPnP reduces the risk of applications automatically creating external port mappings, but it does not prevent local devices from communicating with the media server.

## Kaspersky and Router Management HTTPS

A separate issue was encountered when accessing the TP-Link router's HTTPS management interface.

The router used a certificate that Kaspersky did not recognize as trusted, resulting in a browser security warning.

The Kaspersky configuration reviewed included:

- DNS over HTTPS (DoH): enabled.
- Secure connection scanning: configured to check connections when requested by protection components.
- Certificate store: Windows certificate store.
- SSL 2.0: blocked.

The proposed resolution was to add the local router management address to Kaspersky's trusted-address configuration rather than disabling protection globally.

This was a router-management HTTPS issue and was not directly caused by DLNA streaming.

## Router Management Access

The router's management interface was accessed through HTTPS using its LAN address.

The standard HTTPS endpoint was used without explicitly specifying port 8443.

The investigation distinguished the router's own management interface from the HMS HTTP service on the Windows PC.

These are separate services with different purposes and security implications.

## Final Configuration and Actions

| Setting | Final Action / Status |
|---|---|
| HMS media streaming | Retained for local network use |
| DLNA / UPnP media functionality | Retained for local media sharing |
| HMS Port Forwarding | Deleted |
| External access to HMS | Not required |
| Router UPnP | Disabled |
| CWMP | Disabled |
| AP Isolation | Enabled on both Wi-Fi bands |
| DoS Protection | Medium |
| Router management | Accessed through HTTPS |
| Kaspersky router certificate warning | Trusted-address exception investigated |

## Outcome

The investigation established that internet port forwarding was unnecessary for the intended local media-streaming scenario.

The unnecessary forwarding rule for port 8080 was deleted, while the local media-server functionality was intended to remain available to the television.

The investigation also identified the distinction between reducing internet exposure and securing services against other devices already connected to the home network.

The case resulted in a more restrictive network configuration without requiring remote access to the HMS.

## Lessons Learned

- Local DLNA media streaming does not require internet port forwarding.
- Port forwarding is a router function, not a feature that an application must implement.
- A service accessible inside the LAN may still represent a security risk even when it is not exposed to the internet.
- Disabling router UPnP helps prevent automatic external port mappings but does not disable all local UPnP/DLNA functionality.
- AP Isolation should not be treated as complete LAN segmentation.
- Router security controls and application-level security controls address different threats.
- Unnecessary Virtual Server rules should be removed.
- Local services should only expose the interfaces and ports required for their intended functionality.
- Remote access should be implemented deliberately rather than enabled by default.

## Future Work

- Review Windows Firewall rules associated with the HMS application.
- Confirm which interfaces and ports the HMS service listens on.
- Verify that local streaming continues to work after removing the port-forwarding rule.
- Determine whether HMS provides authentication or access-control options.
- Investigate whether media folders can be restricted to a dedicated directory.
- Consider network segmentation for untrusted IoT and guest devices.
- Review router isolation features and their limitations.
- Periodically audit router Virtual Server rules.
- Avoid exposing the HMS HTTP interface to the internet unless a secure remote-access requirement exists.

## Conclusion

This investigation focused on securing a Windows-based home media server without breaking local media streaming.

The primary security improvement was removing unnecessary internet port forwarding while retaining local DLNA/UPnP functionality.

The remaining risk was associated with access from devices already present on the local network, emphasizing that removing internet exposure is an important security improvement but does not replace local network access control.
