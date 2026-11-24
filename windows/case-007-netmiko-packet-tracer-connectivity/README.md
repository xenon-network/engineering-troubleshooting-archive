# Case 007: Netmiko Network Automation — Connection Failures and Simulator Limitations

**Investigation Date:** Jule 2026

## Overview

This case documents an introductory network automation investigation using Python and Netmiko.

The objective was to establish a Python-based connection to a network device and automate CLI operations. The investigation involved a real TP-Link home router and a Cisco router simulated in Cisco Packet Tracer.

Several obstacles were identified:

- The real home router did not accept the attempted Telnet connection.
- The simulated Cisco router was isolated from the host PC's network.
- Python could not reach the Packet Tracer router directly.
- Installing and verifying Python libraries introduced additional environment-related questions.

The investigation helped distinguish Python automation problems from network reachability, device management capabilities, and simulator limitations.

The automation workflow was not successfully completed. The investigation instead identified the limitations of the original test environment and led to the decision to explore GNS3.

---

## Investigation Data:

### Environment

| Parameter | Value |
|---|---|
| Host OS | Windows |
| Network simulator | Cisco Packet Tracer 8.x |
| Automation language | Python |
| Automation library | Netmiko |
| Real router | TP-Link Archer C6 |
| Real router management | Web GUI (HTTPS) |
| Simulated device | Cisco IOS-style router |
| Physical network | Ethernet connection |
| Virtualization | VirtualBox network adapter also present |
| Python package manager | pip |
| Package mirror | Yandex PyPI mirror |

### Target Devices

**Real home router:**

- LAN address: `192.168.1.1`
- Management interface: Web GUI
- Telnet connection: refused
- CLI automation: not established

**Cisco Packet Tracer router:**

- Simulated address: `192.168.10.1`
- CLI access: available inside Packet Tracer
- Host PC connectivity: unsuccessful
- Host ping test: Request timed out

### Investigation Outcome

- Netmiko installation was investigated.
- Python connection attempts were made.
- Connection refusal was encountered when targeting the real router.
- The simulated router was configured for remote CLI access.
- Host-to-simulator connectivity was investigated.
- Packet Tracer isolation was identified as the fundamental limitation for the intended workflow.
- GNS3 was selected as the next environment to investigate.

---

## 1. Objective

The initial objective was to use Python to connect to a network device and execute CLI commands automatically.

The planned workflow:

```text
Python Script
     |
     v
Netmiko Library
     |
     v
SSH / Telnet Session
     |
     v
Network Device CLI
     |
     v
Command Execution
     |
     v
Output Returned to Python
```

The intended tasks included:

- Connecting to a router.
- Authenticating.
- Executing CLI commands.
- Retrieving device information.
- Exploring configuration automation.
- Understanding how NetDevOps engineers automate repetitive operations.

---

## 2. Initial Target: Real TP-Link Router

The first target was the existing home router.

The router's management interface was accessible through a web browser, but the attempted Python connection was unsuccessful.

The connection attempt produced:

```text
[WinError 10061] Connection refused
```

The investigation identified that the router's management interface was GUI-oriented and did not expose the Telnet CLI endpoint that the attempted Netmiko connection expected.

The router was not an appropriate target for the initial CLI automation exercise.

### Finding

A device being accessible through a browser does not automatically mean it supports SSH or Telnet automation.

Management interfaces may include:

- Web GUI over HTTPS
- SSH
- Telnet
- SNMP
- REST API
- NETCONF
- Vendor-specific management protocols

The availability of one management method does not imply the availability of others.

---

## 3. Netmiko Installation and Python Environment

Netmiko was installed or investigated through the Python package management environment.

A package-related issue involved the Yandex PyPI mirror.

The terminal indicated that the installed pip version was:

```text
22.0.4
```

A newer version was available, but the configured mirror did not provide it at that time.

The investigation established that an outdated pip version was not automatically a blocker for installing or using Netmiko.

### Checking the Netmiko version

An attempt to check Netmiko using a standalone command was incorrect:

```text
netmiko --version
```

Netmiko is a Python library rather than a standalone command-line application.

The correct way to check the installed library version is:

```bash
python -c "import netmiko; print(netmiko.__version__)"
```

This command imports the library and prints its version.

The exact installed Netmiko version was not confirmed in the available history.

---

## 4. Initial Connection Attempt

The Python script attempted to connect to the real router at:

```text
192.168.1.1
```

The connection was refused.

The attempted management method was incompatible with the router's available management interface.

### Troubleshooting reasoning

The investigation distinguished between:

**Connection refused**

A TCP connection attempt was rejected. This can occur when a service is not listening on the requested port or the target explicitly rejects the connection.

**Connection timeout**

A connection attempt receives no response within the configured timeout. This can indicate routing, filtering, connectivity, or device availability issues.

The two errors should not be treated as equivalent.

---

## 5. Switching to Cisco Packet Tracer

The next approach was to use the simulated Cisco router instead of the real home router.

The simulated router was configured through the Packet Tracer CLI.

The lab included:

- Router CLI access
- Interface configuration
- IP addressing
- Remote access configuration
- VTY lines
- Username and password configuration
- Telnet-related management access
- Router interface verification

Example configuration concepts:

```cisco
enable
configure terminal

hostname Home-Gateway

interface GigabitEthernet0/0
 no shutdown
exit

line vty 0 4
 login
exit

end
```

The actual configuration used during the experiment included additional authentication settings and should be reconstructed if exact commands are required.

---

## 6. VTY Configuration

The investigation covered Virtual Teletype (VTY) lines.

The configuration:

```cisco
line vty 0 4
```

defines five VTY lines, numbered 0 through 4.

This was used to understand how remote CLI sessions are handled.

The investigation also examined:

- Remote access authentication
- Username and password configuration
- Telnet versus SSH
- VTY session limits
- Management-plane security

### Security observation

Telnet does not encrypt the management session.

SSH is generally preferred because it provides encrypted remote management.

The lab introduced remote management configuration concepts but did not establish a verified secure SSH automation workflow.

---

## 7. Testing Host-to-Simulator Connectivity

After configuring the simulated router, the host PC attempted to reach the simulated IP address.

The test:

```text
ping 192.168.10.1
```

Result:

```text
Request timed out.
```

The host PC could not communicate directly with the simulated router.

### Why this mattered

Packet Tracer operates as a simulation environment. Devices inside its topology do not automatically become directly reachable through the physical host's network interfaces.

The simulated router may have a valid IP address and working interfaces inside the simulation while remaining inaccessible from the host operating system.

This creates a separation between:

- Packet Tracer's simulated network
- The Windows host network
- Physical Ethernet interfaces
- Virtual network adapters

The failed ping established that basic host-to-simulated-router reachability was not available in the tested configuration.

---

## 8. Understanding the Network Isolation

The investigation included examining the Windows host's network configuration and ARP table.

A VirtualBox host-only network was also observed.

The presence of virtual adapters does not automatically create connectivity to a Packet Tracer topology.

A virtual interface may belong to:

- VirtualBox
- VMware
- Hyper-V
- WSL
- VPN software

These interfaces represent separate virtual networks unless an appropriate bridge, routing mechanism, or integration feature connects them.

### Important distinction

The existence of a simulated router IP does not guarantee that the real host can reach it.

For Python automation to work, the host must have actual IP connectivity to the target device's management address.

Netmiko cannot overcome the absence of network connectivity.

---

## 9. Packet Tracer Limitations

Packet Tracer was useful for learning:

- Cisco CLI navigation
- Basic router configuration
- VLANs
- Trunking
- Router-on-a-Stick
- Interface states
- Routing fundamentals
- Network simulation

However, the investigation demonstrated limitations when attempting to use Packet Tracer as a target for automation from the host operating system.

The simulator did not provide the direct host connectivity required for the intended Netmiko workflow in the tested setup.

### Result

Packet Tracer remained useful for networking fundamentals, but the attempted host-based automation workflow was not supported by the environment as configured.

---

## 10. Decision: GNS3

After the host-to-simulator connectivity test failed, GNS3 was selected as the next environment to investigate.

The reason was the possibility of creating virtual network interfaces and connectivity that would allow the host PC to communicate with emulated network devices.

The planned GNS3 workflow:

```text
Windows Host
     |
     | Python + Netmiko
     |
     v
Host Network Interface
     |
     v
GNS3 Virtual Network
     |
     v
Emulated Router
     |
     v
SSH Management Interface
```

The investigation considered GNS3 and EVE-NG as alternatives to Packet Tracer for network automation labs.

GNS3 installation and a successfully working Netmiko session were not confirmed in this case.

---

## 11. Additional Networking Concepts Investigated

The automation investigation also involved related network management concepts.

### Management plane

The management plane is the part of a device used to configure, monitor, and administer it.

Examples:

- SSH
- Telnet
- HTTPS management GUI
- SNMP
- API access

### VTY

Virtual Teletype lines provide remote CLI sessions on Cisco-style devices.

The investigation covered the VTY range:

```cisco
line vty 0 4
```

This represents five lines.

### TR-069 / CWMP

The home router's management settings included TR-069-related concepts.

The investigation examined:

- ACS (Auto Configuration Server)
- Inform messages
- Authentication
- Remote device management
- Security implications of management credentials

This was a supporting theoretical investigation rather than part of the successful Netmiko workflow.

---

## 12. Troubleshooting Summary

| Problem | Investigation | Result |
|---|---|---|
| Connection refused to home router | Management protocol and target service examined | Router did not accept the attempted Telnet connection |
| Netmiko version check | Standalone CLI command investigated | Corrected to Python import method |
| pip upgrade issue | Yandex mirror examined | Newer version not available from mirror at that time |
| Host ping to Packet Tracer router | `ping 192.168.10.1` | Request timed out |
| Host-to-simulator connectivity | Virtual and simulated network separation examined | No direct reachability established |
| Automation environment | Alternative simulators considered | GNS3 selected for next experiment |
| Successful automated CLI session | Not achieved | Unconfirmed / incomplete |

---

## 13. Lessons Learned

1. Verify that the target device supports the management protocol before attempting automation.
2. A Web GUI does not imply SSH or Telnet support.
3. A connection refusal is different from a timeout.
4. Python libraries are not necessarily standalone command-line applications.
5. Package mirrors can affect software installation and upgrade availability.
6. Network automation depends on actual connectivity to the management interface.
7. A simulated device may be isolated from the host operating system.
8. Packet Tracer is excellent for foundational network learning but has limitations for host-based automation.
9. GNS3 or EVE-NG may be more suitable for automation workflows requiring host connectivity.
10. SSH should be preferred over Telnet for secure management.
11. Management-plane security is an important part of network automation.
12. Troubleshooting should begin with reachability and protocol verification before debugging the Python code.

---

## Conclusion

The initial Netmiko experiment did not produce a successful automated CLI session.

The investigation nevertheless identified the fundamental reasons behind the failure:

- The real router did not accept the attempted Telnet connection.
- The Cisco Packet Tracer router was not directly reachable from the host PC.
- The automation workflow therefore lacked an accessible management endpoint.

The most important outcome was the selection of a more suitable network emulation environment for future automation work.

This case represents an exploratory troubleshooting investigation that established the prerequisites for successful network automation rather than a completed automation deployment.

---

## Future Work

- Install and configure GNS3.
- Establish host-to-emulated-router connectivity.
- Verify IP reachability before attempting Netmiko.
- Configure SSH on the emulated Cisco device.
- Create a secure Python connection profile.
- Execute read-only commands using Netmiko.
- Collect and save CLI output.
- Implement error handling.
- Explore configuration automation.
- Compare Packet Tracer, GNS3, and EVE-NG for network automation workflows.

---

*Documentation note: This case is based on the investigation history. Successful automation is not claimed where the available evidence does not confirm it. Credentials and personal infrastructure identifiers are intentionally omitted.*
