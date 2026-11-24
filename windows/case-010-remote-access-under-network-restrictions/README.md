# Case 010: Remote Access Solutions Under Network Restrictions

## Overview

This case documents an investigation into remote access to a Windows PC from an Android smartphone over an external network, where network restrictions, connection failures, and software installation limitations prevented the establishment of a reliable remote connection.

The investigation focused on RustDesk and AnyDesk, their configuration, firewall requirements, connectivity limitations, and possible alternatives.

The objective was to find a reliable way to remotely access the home Windows PC and subsequently manage devices on the home LAN without exposing management interfaces directly to the public internet.

**Investigation Date:** August 2026

**Status:** Unresolved

## 1. Problem Description

The objective was to establish remote access to a Windows PC from an Android smartphone when the smartphone was connected through a mobile network rather than the home Wi-Fi network.

The connection was unsuccessful despite installing and configuring remote access software.

The investigation was complicated by network restrictions affecting software installation and connectivity.

### Requirements

- Access the Windows PC remotely from an Android smartphone.
- Establish connectivity without being connected to the same local Wi-Fi network.
- Avoid unnecessary exposure of services directly to the internet.
- Use a solution that remains functional under network restrictions.
- Ideally, access local network devices through the remotely connected PC.

---

## 2. Environment

### Windows PC

- Operating system: Windows
- Remote access software: RustDesk
- Firewall: Windows Firewall / third-party firewall environment
- Network: Home LAN

### Android Smartphone

- Device: Redmi 9
- Remote access software: RustDesk and AnyDesk
- Network: Mobile internet during remote access testing

Exact software versions and network configuration should be recorded in the Investigation Data section.

---

## 3. RustDesk Investigation

RustDesk was selected as one of the potential remote access solutions.

The investigation included reviewing its configuration, connection settings, security-related information, and firewall requirements.

### 3.1 Configuration Review

RustDesk configuration was examined through its graphical interface.

The investigation included:

- Reviewing connection settings.
- Checking whether custom or self-hosted server configuration was being used.
- Investigating transport-related settings.
- Reviewing the device fingerprint displayed by the application.
- Comparing the available configuration options with troubleshooting instructions.

Some suggested settings did not correspond to the options available in the installed application version.

This highlighted the importance of verifying troubleshooting instructions against the actual software version and interface.

### 3.2 Firewall Configuration

Firewall rules were created to allow additional RustDesk communication ports.

The investigated configuration included the following TCP ports:

| Port  | Protocol |
| ----- | -------- |
| 21115 | TCP      |
| 21116 | TCP      |
| 21117 | TCP      |
| 21118 | TCP      |
| 21119 | TCP      |

An additional rule was created for:

| Port  | Protocol |
| ----- | -------- |
| 21116 | UDP      |

The rules were configured to allow connections through Windows Firewall.

Despite these changes, the remote connection remained unsuccessful.

**Result:** Allowing the investigated ports did not resolve the connectivity problem.

The investigation did not conclusively determine whether the failure was caused by upstream network restrictions, server connectivity, NAT traversal, transport behavior, application configuration, or another factor.

---

## 4. RustDesk Connection Failure

After reviewing configuration settings and creating firewall rules, connection attempts were repeated.

### Observations

- RustDesk was installed on the Windows PC.
- RustDesk was available on the Android smartphone.
- Firewall rules for additional communication ports were configured.
- The connection still failed.
- No successful remote session was confirmed.

### Troubleshooting limitations

The investigation encountered several limitations:

1. Some proposed configuration options were not available in the installed version.
2. The exact network-level cause of the failure was not identified.
3. Firewall changes did not produce a confirmed successful connection.
4. No packet capture or complete server-side connection logs were available to conclusively identify the failure point.

The unsuccessful connection should not be attributed to a specific network restriction without additional evidence.

---

## 5. AnyDesk Investigation

AnyDesk was considered as an alternative remote access solution.

The application was already available on the Android smartphone.

However, installing AnyDesk on the Windows PC was unsuccessful because of network restrictions.

The installation problem persisted even when a VPN connection was being used.

### Observed limitations

- AnyDesk was available on the smartphone.
- Installation on Windows was blocked or unsuccessful.
- VPN usage did not resolve the Windows installation issue.
- A complete AnyDesk connection test could not be performed.

Therefore, AnyDesk could not be verified as a working alternative in this environment.

The inability to install the application was a separate obstacle from the RustDesk connection failure.

---

## 6. Network Restrictions

The investigation demonstrated that remote access troubleshooting can involve several separate layers:

### Layer 1 — Software availability

The application must be successfully installed and executable on both endpoints.

In this investigation, AnyDesk installation on Windows was unsuccessful.

### Layer 2 — Local configuration

The remote access software must have appropriate configuration, including connection settings, identity information, and authentication.

### Layer 3 — Host firewall

The Windows firewall must permit the traffic required by the application.

Firewall rules were modified during the RustDesk investigation, but the connection remained unsuccessful.

### Layer 4 — Network connectivity

The endpoint must be able to communicate with the remote access infrastructure or the other endpoint.

Possible limitations include:

- NAT behavior.
- Carrier-grade NAT (CGNAT).
- UDP restrictions.
- TCP restrictions.
- DNS restrictions.
- Filtering of application traffic.
- VPN interference.
- Network policies imposed by an ISP or mobile provider.

These factors were considered as possible explanations but were not individually verified as the root cause.

### Layer 5 — Remote access infrastructure

Depending on the remote access solution, connectivity may require direct connections, relay infrastructure, or both.

The investigation did not establish which connection mechanism was failing.

---

## 7. Security Considerations

Remote access introduces security risks, particularly when a device is reachable from outside the home network.

### Direct port forwarding

Exposing remote management services directly to the internet can increase the risk of:

- Unauthorized login attempts.
- Password brute-force attacks.
- Exploitation of vulnerable services.
- Discovery of exposed management interfaces.

Direct exposure of sensitive management interfaces was not considered a preferred solution.

### VPN-based access

A VPN-based architecture was considered a safer long-term approach for accessing devices on the home LAN.

A VPN can provide an authenticated encrypted connection into the home network, allowing access to internal resources without individually exposing their management interfaces.

However, establishing a VPN server and configuring remote connectivity was outside the scope of the completed investigation.

### Authentication and device identity

The investigation also included reviewing RustDesk's fingerprint information.

Device identity verification is an important security mechanism when establishing remote connections.

Sharing a fingerprint does not substitute for protecting authentication credentials, passwords, or other access tokens.

---

## 8. Troubleshooting Summary

| Attempt                               | Result                              | Status       |
| ------------------------------------- | ----------------------------------- | ------------ |
| Install and configure RustDesk        | Application available               | Completed    |
| Review RustDesk configuration         | Configuration investigated          | Completed    |
| Review connection-related settings    | Some suggested settings unavailable | Completed    |
| Configure RustDesk TCP firewall rules | Rules created                       | Completed    |
| Configure RustDesk UDP firewall rule  | Rule created                        | Completed    |
| Retry RustDesk connection             | Connection still failed             | Unsuccessful |
| Install AnyDesk on Android            | Application already available       | Confirmed    |
| Install AnyDesk on Windows            | Restricted / unsuccessful           | Unsuccessful |
| Retry Windows installation using VPN  | Still unsuccessful                  | Unsuccessful |
| Establish remote access session       | No successful session confirmed     | Unresolved   |

---

## 9. Root Cause

The root cause of the remote access failure was not conclusively established.

The investigation identified multiple potential obstacles:

- Network restrictions.
- Software installation limitations.
- Firewall configuration.
- Remote access application configuration.
- Possible NAT traversal or transport limitations.

However, there was insufficient evidence to identify one specific cause.

The failure of RustDesk after firewall modifications does not prove that the firewall was not involved.

Likewise, the failure to install AnyDesk does not establish that AnyDesk's remote connection protocol itself was blocked.

---

## 10. Lessons Learned

### 10.1 Separate installation problems from connectivity problems

An application that cannot be installed cannot be properly evaluated as a connectivity solution.

Installation, configuration, and connection establishment should be investigated as separate stages.

### 10.2 Firewall rules alone may not solve remote access problems

Allowing local firewall ports does not guarantee connectivity through upstream network infrastructure.

The complete connection path must be investigated.

### 10.3 Verify instructions against the installed application version

Remote access applications frequently change their interface and configuration options.

Troubleshooting instructions should be verified against the actual installed version.

### 10.4 Avoid assuming the root cause

A failed connection can result from several independent network layers.

A reliable diagnosis requires evidence such as:

- Connection logs.
- Network reachability tests.
- DNS resolution results.
- Firewall logs.
- Packet captures.
- Verification of relay or server connectivity.

### 10.5 Prefer secure remote access architecture

Remote access to home network devices should ideally use an authenticated VPN or another securely configured remote access solution instead of exposing device management ports directly to the internet.

---

## 11. Final Outcome

The investigation did not establish a working remote access connection from the Android smartphone to the Windows PC over the external network.

RustDesk remained unsuccessful despite configuration review and firewall changes.

AnyDesk could not be properly tested because installation on Windows was unsuccessful under the encountered network restrictions.

The investigation identified several potential areas for further testing but did not confirm the underlying cause.

**Final status: Unresolved — no successful remote access session confirmed.**

---

## 12. Future Investigation

Potential next steps:

- Record the exact RustDesk connection error and logs.
- Verify DNS resolution for remote access infrastructure.
- Test basic connectivity independently of the remote access application.
- Investigate whether TCP and UDP traffic is restricted.
- Capture packets during a connection attempt.
- Test connectivity using a different external network.
- Investigate a VPN-based architecture for remote access to the home LAN.
- Evaluate other remote access solutions under the same restrictions.
- Separate remote desktop access from remote LAN routing requirements.
- Document each test and its result before changing multiple configuration variables.

---

## Disclaimer

This case documents a real troubleshooting investigation, including unsuccessful attempts.

No successful remote access configuration or confirmed root cause is claimed.

Network addresses, device identifiers, account details, and other sensitive information should be anonymized before publishing diagnostic logs or screenshots.
