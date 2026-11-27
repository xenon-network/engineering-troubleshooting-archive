# Case 004 — ZeroTier Remote Access to a Home LAN: Windows Subnet Routing Troubleshooting

## Overview

This case documents an unsuccessful attempt to provide remote access to a home local network through ZeroTier, using a Windows PC as a gateway between a ZeroTier virtual network and the physical LAN.

The goal was to remotely access LAN device management interfaces and the home router from an Android smartphone over mobile data, without exposing those management interfaces directly to the public Internet.

The investigation included ZeroTier network membership and managed routes, Windows IP forwarding, firewall filtering, Internet Connection Sharing (ICS), routing-table inspection, interface configuration, and MTU changes.

- **Status:** Unresolved
- **Investigation period:** August 2026
- **Root cause:** Not conclusively identified

## 1. Objectives

- Establish remote connectivity to the home LAN through ZeroTier.
- Access LAN device management interfaces from a smartphone outside the home network.
- Access the home router's management interface remotely.
- Use the existing Windows PC as a subnet gateway.
- Avoid purchasing additional networking equipment.
- Investigate partial connectivity and connection timeouts.
- Avoid exposing management interfaces directly to the Internet.

## 2. Network Environment

### Windows gateway PC

- Operating system: Windows
- Ethernet adapter: Intel Ethernet Connection (17) I219-V
- Physical LAN address: `WINDOWS_LAN_IP`
- ZeroTier virtual adapter: ZeroTier Virtual Port
- ZeroTier address: `ZT_GATEWAY_IP`
- Firewall management: SimpleWall
- Windows Firewall configuration was changed during troubleshooting

### Home network

- LAN subnet: `192.168.x.0/24` (anonymized)
- Default gateway: `ROUTER_LAN_IP`
- Router: TP-Link Archer C6

### Remote client

- Android smartphone
- ZeroTier Android application
- Mobile data used during remote-access tests; Wi-Fi was disabled

### Target devices

- One or more LAN devices with web-based management interfaces: `TARGET_DEVICE_IP`
- Home router management interface: `ROUTER_LAN_IP`

Specific device types are generalized because they are not essential to the routing investigation.

## 3. Intended Architecture

The Windows PC was intended to forward traffic between its ZeroTier virtual interface and the physical home LAN.

```text
Android smartphone
        |
   Mobile data
        |
 ZeroTier virtual network
        |
        v
 Windows gateway PC
  - ZeroTier: ZT_GATEWAY_IP
  - LAN: WINDOWS_LAN_IP
        |
        v
 Home LAN: 192.168.x.0/24
        |
        +---- Home router (ROUTER_LAN_IP)
        |
        +---- Target LAN device (TARGET_DEVICE_IP)
```

The target LAN devices did not need to be enrolled as individual ZeroTier members. The intended design relied on subnet routing through the Windows gateway.

## 4. Initial ZeroTier Configuration

### Network membership

- The smartphone and Windows PC were connected to the same ZeroTier virtual network.
- Both members were authorized in the ZeroTier network management interface.

### Managed route

A managed route was configured for the home LAN, with the Windows PC's ZeroTier address specified as the route gateway.

| Parameter | Configuration |
|---|---|
| Destination | `192.168.x.0/24` (anonymized) |
| Via | `ZT_GATEWAY_IP` |
| Intended gateway | Windows PC |

The route was intended to direct traffic for the home LAN through the Windows PC's ZeroTier address.

### Windows IP forwarding

IP forwarding was enabled using PowerShell:

```powershell
Set-NetIPInterface -Forwarding Enabled
```

Forwarding was also attempted on individual interfaces. The available history does not establish a final, verified forwarding configuration.

## 5. Initial Connectivity Tests

Tests were performed from the Android smartphone with Wi-Fi disabled and mobile data enabled.

### Target LAN device

The device's web management interface was tested using its anonymized address:

```text
http://TARGET_DEVICE_IP
```

**Observed:**

- The web interface began loading.
- Some dashboard elements and information appeared.
- Live status information was not fully updated.
- The interface showed stale or disconnected data.
- The connection eventually timed out.

### Router management interface

Remote access to the home router's management interface was also attempted.

**Observed:**

- The management page did not load successfully.
- Connection attempts timed out.

The original history contains an inconsistent target address in this router-test section. The precise URL used for the router attempt cannot be confirmed from the record; the outcome is recorded as a failed router-management access test.

The contrast between partial loading of a target device interface and failure to load the router interface complicated diagnosis.

## 6. Firewall Troubleshooting

### SimpleWall

SimpleWall was used to control Windows network traffic. ZeroTier-related application and permission settings were investigated.

Actions recorded:

- Reviewed ZeroTier application rules.
- Investigated UDP port `9993`.
- Checked whether ZeroTier traffic was allowed.
- Disabled SimpleWall during troubleshooting.
- Retested connectivity with SimpleWall disabled.

Disabling SimpleWall did not resolve the remote-access problem.

### Windows Firewall

Windows Firewall was also investigated and temporarily disabled during some tests. Firewall configuration was changed to help determine whether filtering was preventing forwarding.

No firewall-rule configuration was confirmed to resolve the problem. Disabling firewalls was a diagnostic experiment, not a recommended permanent configuration.

## 7. Internet Connection Sharing (ICS)

Internet Connection Sharing was tested as a possible method of enabling connectivity between network interfaces.

### Configuration attempt

The Ethernet adapter's sharing settings were modified to share its connection with the ZeroTier virtual adapter.

### Observed side effects

- The Windows PC lost normal Internet connectivity after sharing was enabled.
- Windows changed network configuration and introduced an additional configuration associated with ICS.
- The sharing configuration was subsequently investigated and partially reversed.

The experiment showed that enabling ICS without first understanding its interface and routing changes could disrupt the gateway PC's existing connectivity. No successful remote-access result was confirmed from this test.

## 8. Routing Table Investigation

The Windows routing table was examined with:

```cmd
route print
```

Relevant entries were recorded in the source history. Address values have been anonymized:

| Destination | Netmask | Gateway | Interface |
|---|---|---|---|
| `0.0.0.0` | `0.0.0.0` | `ROUTER_LAN_IP` | `WINDOWS_LAN_IP` |
| `192.168.x.0` | `255.255.255.0` | On-link | `WINDOWS_LAN_IP` |
| `192.168.x.0` | `255.255.255.0` | `ZT_GATEWAY_IP` | `ICS_INTERFACE_IP` |
| `192.168.137.0` | `255.255.255.0` | On-link | `ICS_INTERFACE_IP` |

The output also contained an additional default route associated with the ICS configuration:

```text
Destination: 0.0.0.0
Netmask:    0.0.0.0
Gateway:    UNEXPECTED_GATEWAY_IP
Interface:  ICS_INTERFACE_IP
```

The interface list included the Intel Ethernet adapter, a ZeroTier virtual adapter, and a software loopback interface.

### Routing observations

- Multiple entries were present for the home-LAN destination network.
- Routes and an additional interface associated with ICS complicated interpretation of the routing configuration.
- The exact final state of the routing table was not established.

A manual route was also attempted:

```cmd
route add 192.168.x.0 mask 255.255.255.0 ROUTER_LAN_IP
```

The command returned `OK!`, but end-to-end remote access was still unsuccessful. A successful route-command response did not verify that traffic was reaching its destination or that return traffic was routed correctly.

## 9. MTU Troubleshooting

Packet fragmentation or tunnel overhead was considered as a possible cause of the incomplete connection.

The Ethernet MTU was changed to `1400`:

```cmd
netsh interface ipv4 set subinterface "Ethernet" mtu=1400 store=persistent
```

A lower value of `1100` was subsequently tested:

```cmd
netsh interface ipv4 set subinterface "Ethernet" mtu=1100 store=persistent
```

Windows IP forwarding was also re-enabled during these tests.

### Results

- The target device's dashboard continued to load only partially.
- Connection timeouts remained.
- Neither MTU setting produced a confirmed improvement.
- The cause of the partial interface loading was not established.

MTU-related packet loss remained a hypothesis, not a verified root cause.

## 10. Additional Investigation

The following areas were reviewed or investigated:

- ZeroTier member authorization and member status.
- Managed Routes configuration.
- Difference between a ZeroTier network route and an individual member address.
- Gateway address specified in the managed route.
- Windows interface forwarding.
- Ethernet interface configuration.
- Internet Connection Sharing.
- SimpleWall filtering.
- Windows Firewall.
- Routing-table entries.
- Network interface metrics.
- MTU adjustments.
- Direct remote access attempts to the router and target LAN device.

The Windows PC's physical LAN address was visible in the ZeroTier Central member information during the investigation. This observation alone did not demonstrate that subnet forwarding was working.

## 11. Unsuccessful Approaches and Results

| Approach | Result |
|---|---|
| ZeroTier managed route through Windows PC | Configured, but end-to-end access remained unsuccessful |
| Windows IP forwarding | Enabled/attempted, but successful forwarding was not verified |
| SimpleWall permissions | Adjusted and tested |
| Disable SimpleWall | Did not resolve the issue |
| Temporarily disable Windows Firewall | Did not resolve the issue |
| Internet Connection Sharing | Changed network configuration and disrupted the PC's Internet connectivity |
| Manual route addition | Command succeeded, but remote access remained unsuccessful |
| MTU `1400` | No confirmed improvement |
| MTU `1100` | No confirmed improvement |
| Router management test | Failed; connection timed out |
| Target device management test | Partial page loading followed by timeout |
| Restart PC and smartphone | No confirmed resolution |

## 12. Final Status

The investigation did not reach a confirmed working configuration.

The smartphone could connect to ZeroTier. The target LAN device's web interface loaded partially during testing, but the connection was unstable and incomplete. Access to the router's management interface was not confirmed.

The investigation was stopped without a verified solution. No additional hardware was purchased; testing was limited to the existing Windows PC, router, smartphone, software, and network configuration.

**Final status: Unresolved.**

## 13. Lessons Learned

### 1. ZeroTier membership is not the same as subnet access

Connecting devices to the same virtual network does not automatically provide access to devices behind another member. Subnet routing depends on the gateway, forwarding, route configuration, and return-path connectivity being correct.

### 2. A managed route does not configure the gateway automatically

A route pointing to a Windows PC is only one component of the configuration. The gateway must also forward traffic correctly between its virtual and physical interfaces.

### 3. Windows routing and ICS can interact in unexpected ways

Internet Connection Sharing can alter interface configuration and routing. Changes should be documented and verified using `ipconfig` and `route print` after significant changes.

### 4. Partial web-page loading is not proof of complete connectivity

The initial page may load while dynamic content or other connections fail. Testing should verify the intended functionality, not only whether a page appears.

### 5. A successful command does not prove end-to-end connectivity

A command such as `route add` can report success while the network path remains broken. Testing from the remote client is necessary.

### 6. Change one variable at a time

Firewall settings, ICS, routing, forwarding, and MTU were investigated during the same troubleshooting period. A controlled sequence with a baseline and one change per test would make cause-and-effect easier to determine.

## 14. AI-Assisted Troubleshooting Review

AI assistance was used to interpret configuration screenshots, explain ZeroTier subnet routing, suggest Windows commands, and propose troubleshooting steps.

Some recommendations were speculative, contradictory, or insufficiently verified. Issues noted in the history included:

- Treating partial page loading as proof that routing was fully functional.
- Repeatedly attributing timeouts to MTU or mobile-network limitations without confirmation.
- Suggesting ICS changes without adequately considering their effect on the Windows PC's existing network configuration.
- Proposing additional routing and firewall changes before establishing a stable baseline.

The investigation reinforced the need to verify commands, understand the routing table, and test each change independently. AI-generated explanations were treated as hypotheses rather than definitive diagnoses.

## 15. Conclusion

This case documents an unsuccessful attempt to implement ZeroTier subnet routing through a Windows PC to enable remote access to a home LAN.

The investigation covered virtual-network membership, managed routes, forwarding, firewall configuration, ICS, routing tables, and MTU adjustments. Partial access to a target device's web interface was observed, but stable remote access to the LAN was never confirmed.

The case remains an open troubleshooting record of the configuration, symptoms, experiments, and lessons learned.

**Root cause:** Not conclusively identified.
