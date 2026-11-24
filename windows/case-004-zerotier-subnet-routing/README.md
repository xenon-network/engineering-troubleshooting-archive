# Case 004: ZeroTier Remote Access to a Home LAN — Windows Subnet Routing Troubleshooting

## Overview

This case documents an attempt to establish remote access to a home local network through ZeroTier, using a Windows PC as a gateway between a virtual network and the physical LAN.

The primary goal was to access ASIC management interfaces and the home router remotely from an Android smartphone connected through mobile data, without exposing management interfaces directly to the Internet.

The investigation covered ZeroTier network membership, managed routes, Windows IP forwarding, firewall configuration, Internet Connection Sharing (ICS), routing tables, interface configuration, and MTU troubleshooting.

**Status:** Unresolved  
**Investigation period:** August 2026

---

## 1. Objectives

- Establish a secure remote connection to the home LAN through ZeroTier.
- Access ASIC management interfaces from a smartphone outside the home network.
- Access the home router's management interface remotely.
- Use the existing Windows PC as a subnet gateway.
- Avoid purchasing additional networking equipment.
- Diagnose partial connectivity and connection timeouts.

---

## 2. Network Environment

### Gateway PC

- Operating system: Windows
- Ethernet adapter: Intel(R) Ethernet Connection (17) I219-V
- Local IP address: WINDOWS_LAN_IP
- ZeroTier virtual adapter: ZeroTier Virtual Port
- ZeroTier IP address: ZT_GATEWAY_IP
- Firewall management: SimpleWall
- Windows Firewall configuration changed during troubleshooting

### Home network

- LAN subnet: `192.168.1.0/24`
- Default gateway: ROUTER_LAN_IP
- Router: TP-Link Archer C6

### Remote client

- Android smartphone
- ZeroTier Android application
- Mobile data connection used for remote testing

### Target devices

- ASIC management interface: ASIC_IP
- Router management interface: ROUTER_LAN_IP

---

## 3. Intended Network Architecture

The Windows PC was intended to act as a gateway between the ZeroTier virtual network and the physical home LAN.

```text
Android Smartphone
Mobile Internet
      |
      v
ZeroTier Virtual Network
      |
      v
Windows Gateway PC
ZeroTier IP: ZT_GATEWAY_IP
LAN IP: WINDOWS_LAN_IP
      |
      v
Home LAN: 192.168.1.0/24
      |
      +---- TP-Link Archer C6
      |     ROUTER_LAN_IP
      |
      +---- ASIC
            ASIC_IP
```

The ASIC devices did not need to be registered as individual ZeroTier members. The intended architecture relied on subnet routing through the Windows gateway.

---

## 4. Initial ZeroTier Configuration

### Network membership

The smartphone and Windows PC were connected to the same ZeroTier virtual network.

Both devices were authorized in the ZeroTier network management interface.

### Managed Route

The following subnet route was configured:

| Parameter | Value |
|---|---|
| Destination | `192.168.1.0/24` |
| Via | ZT_GATEWAY_IP |
| Gateway | Windows PC |

The route was intended to direct traffic for the home LAN through the Windows PC's ZeroTier address.

### IP Forwarding

Windows IP forwarding was enabled using PowerShell:

```powershell
Set-NetIPInterface -Forwarding Enabled
```

Forwarding was also attempted on individual network interfaces.

---

## 5. Initial Connectivity Tests

Remote testing was performed using the smartphone with Wi-Fi disabled and mobile data enabled.

### ASIC access

Target:

```text
http://ASIC_IP
```

Observed result:

- The web interface began loading.
- Some dashboard elements and information appeared.
- Live status information was not fully updated.
- The interface displayed disconnected or stale data.
- The connection eventually timed out.

### Router access

Target:

```text
http://ASIC_IP
```

Observed result:

- The router management page did not load successfully.
- Connection attempts timed out.

The difference between partial ASIC page loading and the inability to load the router page complicated diagnosis.

---

## 6. Firewall Troubleshooting

### SimpleWall

SimpleWall was used to control Windows network traffic.

The ZeroTier application/service was identified in SimpleWall and permission settings were investigated.

Actions performed:

- Reviewed ZeroTier application rules.
- Investigated UDP port 9993.
- Checked whether ZeroTier traffic was allowed.
- Disabled SimpleWall completely during troubleshooting.
- Tested connectivity with SimpleWall disabled.

Disabling SimpleWall did not resolve the remote-access problem.

### Windows Firewall

Windows Firewall was also investigated and temporarily disabled during some tests.

The firewall configuration was changed multiple times to isolate whether traffic filtering was preventing forwarding.

No confirmed firewall rule configuration resolved the problem.

---

## 7. Internet Connection Sharing (ICS)

Internet Connection Sharing was tested as a possible way to enable forwarding between network interfaces.

### Configuration attempt

The Ethernet adapter's Sharing settings were modified to share its connection with the ZeroTier virtual adapter.

### Observed side effect

After enabling sharing, the Windows PC lost normal Internet connectivity.

The network configuration changed and Windows introduced an additional network configuration associated with ICS.

The sharing configuration was subsequently investigated and partially reversed.

This experiment demonstrated that enabling ICS without understanding the resulting interface and routing changes could disrupt the gateway PC's own network connectivity.

---

## 8. Routing Table Investigation

The Windows routing table was examined using:

```cmd
route print
```

The output showed the following relevant configuration:

| Destination | Netmask | Gateway | Interface |
|---|---|---|---|
| `0.0.0.0` | `0.0.0.0` | `ROUTER_LAN_IP` | `WINDOWS_LAN_IP` |
| `192.168.1.0` | `255.255.255.0` | On-link | `WINDOWS_LAN_IP` |
| `192.168.1.0` | `255.255.255.0` | `ZT_GATEWAY_IP` | `ICS_INTERFACE_IP` |
| `192.168.137.0` | `255.255.255.0` | On-link | `ICS_INTERFACE_IP` |

The output also contained an additional default route associated with the ICS configuration:

```text
0.0.0.0
0.0.0.0
UNEXPECTED_GATEWAY_IP
ICS_INTERFACE_IP
```

The interface list included:

- Intel Ethernet Connection (17) I219-V
- ZeroTier Virtual Port #2
- Software Loopback Interface

### Routing observations

The routing table contained multiple entries associated with the `192.168.1.0/24` destination network.

The presence of an additional network interface and routes associated with `ICS_INTERFACE_IP` complicated the routing configuration.

A manually added route was also attempted:

```cmd
route add 192.168.1.0 mask 255.255.255.0 ROUTER_LAN_IP
```

The command returned:

```text
OK!
```

However, successful route command execution did not establish successful remote connectivity.

---

## 9. MTU Troubleshooting

The possibility of packet fragmentation or tunnel overhead affecting the connection was investigated.

The Ethernet MTU was changed using:

```cmd
netsh interface ipv4 set subinterface "Ethernet" mtu=1400 store=persistent
```

A smaller MTU was subsequently tested:

```cmd
netsh interface ipv4 set subinterface "Ethernet" mtu=1100 store=persistent
```

Windows IP forwarding was also re-enabled during these tests.

### Results

- The ASIC dashboard continued to load partially.
- Connection timeouts remained.
- Lowering MTU did not produce a confirmed solution.
- The underlying cause of the partial loading was not established.

MTU-related packet loss remained a hypothesis rather than a verified root cause.

---

## 10. Additional Investigation

The following configuration areas were examined:

- ZeroTier member authorization.
- Managed Routes configuration.
- Difference between a ZeroTier network address and an individual member address.
- Correct gateway IP for the `Via` field.
- Windows interface forwarding.
- Ethernet interface configuration.
- Internet Connection Sharing.
- SimpleWall filtering.
- Windows Firewall.
- Routing table entries.
- Network interface metrics.
- MTU adjustments.
- Direct access to the router and ASIC.
- ZeroTier connectivity and member status.

The Windows PC's physical IP address was visible in the ZeroTier Central member information during investigation.

However, this alone did not prove that subnet forwarding was functioning correctly.

---

## 11. Failed and Unsuccessful Approaches

| Approach | Result |
|---|---|
| ZeroTier Managed Route to Windows PC | Configured, but end-to-end access remained unsuccessful |
| Windows IP forwarding | Enabled, but no confirmed successful forwarding |
| SimpleWall permissions | Adjusted and tested |
| Disable SimpleWall | Did not resolve the issue |
| Disable Windows Firewall | Did not resolve the issue |
| Internet Connection Sharing | Caused additional network configuration changes and loss of Internet connectivity |
| Manual route addition | Command succeeded, but remote access still failed |
| MTU 1400 | No confirmed improvement |
| MTU 1100 | No confirmed improvement |
| Router access test | Failed |
| ASIC access test | Partial page loading followed by timeout |
| Restart PC and smartphone | Did not produce a confirmed resolution |

---

## 12. Final Status

The project did not reach a confirmed working configuration.

The smartphone could establish a ZeroTier connection, and the ASIC web interface could partially load through the attempted routing configuration. However, the remote connection was unstable and incomplete.

The router management interface was not accessible through the tested configuration.

The investigation was stopped without a verified solution.

**Final status: Unresolved**

No additional hardware was purchased. The investigation was limited to the existing Windows PC, router, smartphone, software, and network configuration.

---

## 13. Lessons Learned

### 1. Network membership is not the same as subnet access

Connecting devices to the same virtual network does not automatically provide access to devices behind another member.

Subnet routing requires a correctly configured gateway, forwarding, routes, and return-path connectivity.

### 2. A managed route does not configure the gateway automatically

A route pointing to a Windows PC is only one part of the configuration.

The gateway must also correctly forward traffic between the virtual and physical interfaces.

### 3. Windows routing and ICS can interact in unexpected ways

Internet Connection Sharing can alter network interface configuration and routing.

Changes should be documented and verified using `ipconfig` and `route print` after every significant configuration change.

### 4. A partially loaded web interface is not proof of successful routing

Initial HTML loading does not guarantee that all application traffic, dynamic data, or persistent connections are working.

A proper test should verify full functionality rather than only whether a page appears.

### 5. Successful commands do not guarantee successful connectivity

Commands such as `route add` can return success while the end-to-end network path remains broken.

The actual result must be tested from the remote client.

### 6. Change one variable at a time

Firewall changes, ICS, routing, forwarding, and MTU adjustments were explored during the same troubleshooting period.

A more controlled testing process would make cause-and-effect relationships easier to establish.

---

## 14. AI-Assisted Troubleshooting Review

AI assistance was used to interpret configuration screenshots, explain ZeroTier subnet routing, suggest Windows commands, and propose troubleshooting steps.

Some recommendations were speculative, contradictory, or insufficiently verified.

Examples included:

- Treating partial page loading as proof that routing was fully functional.
- Repeatedly attributing timeouts to MTU or mobile-network limitations without confirmation.
- Changing Internet Connection Sharing configuration without adequately considering the potential effect on the Windows PC's existing network configuration.
- Suggesting additional routing and firewall changes before establishing a stable baseline.

The investigation highlighted the importance of verifying proposed commands, understanding the Windows routing table, and testing each change independently.

AI-generated explanations were treated as troubleshooting hypotheses, not definitive diagnoses.

---

## 15. Conclusion

This case documents an unsuccessful attempt to implement ZeroTier subnet routing through a Windows PC.

The investigation covered virtual network membership, managed routes, forwarding, firewall configuration, ICS, routing tables, and MTU.

Although partial access to the ASIC interface was observed, a stable remote connection to the home LAN was never confirmed.

The case remains an open troubleshooting record documenting the configuration, symptoms, experiments, and lessons learned.

**Root cause: Not conclusively identified.**
