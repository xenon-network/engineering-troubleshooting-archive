# Case 005: Home Network Design and VLAN Configuration in Cisco Packet Tracer

**Investigation Date:** June 2026

## Overview

This case documents the initial development of a simulated home network in Cisco Packet Tracer, including physical topology design, switch configuration, VLAN fundamentals, router subinterfaces, and attempts to expand the lab toward DHCP, wireless connectivity, and network automation.

The project was intended to reproduce a home network in a controlled simulation while introducing enterprise networking concepts such as VLAN segmentation, 802.1Q trunking, routing, and network device management.

The lab was partially completed. Basic topology construction and initial VLAN/router configuration were performed, but end-to-end connectivity testing and several planned features remained unfinished.

---

## Objectives

- Build a simulated home network topology.
- Understand the relationship between routers, switches, and end devices.
- Configure switch access ports and VLANs.
- Introduce 802.1Q trunking.
- Configure router subinterfaces using Router-on-a-Stick.
- Understand broadcast domains and IP addressing.
- Explore DHCP and wireless connectivity.
- Investigate the limitations of Packet Tracer as a simulation environment.
- Explore the possibility of automating network device configuration using Python.

---

## Environment

| Component | Description |
|---|---|
| Simulator | Cisco Packet Tracer |
| Router | Home-Gateway / simulated Cisco router |
| Switch | Home-Switch |
| End devices | PCs, laptop, smartphones |
| Configuration interface | Cisco-style CLI and graphical device interfaces |
| Network design | Home network with potential VLAN segmentation |

The lab used simulated equipment rather than physical networking hardware.

---

## 1. Initial Topology Design

The first topology contained:

- One router
- One switch
- Two wired end devices
- Additional wireless devices planned for later stages

The intended physical architecture was:

```text
                    ┌───────────────┐
                    │ Home-Gateway  │
                    │    Router     │
                    └───────┬───────┘
                            │
                       Trunk Link
                            │
                    ┌───────┴───────┐
                    │ Home-Switch   │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
              Main PC               Laptop
```

During topology review, a direct connection between the main PC and router was identified as inconsistent with the intended switched architecture.

The topology was adjusted so that the end devices connected to the switch, while the router used a single physical connection to the switch for the planned VLAN configuration.

### Design principle

In a Router-on-a-Stick architecture, multiple VLANs share a physical router connection through logical subinterfaces. The switch-to-router connection therefore needs to carry traffic for multiple VLANs.

---

## 2. VLAN Configuration

The initial VLAN exercise focused on creating VLAN 10 and assigning an access port.

Example configuration:

```cisco
enable
configure terminal

vlan 10
 name Engineering
exit

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

end
write memory
```

The switch was subsequently configured with a trunk link intended to connect to the router.

### VLAN concepts explored

- VLANs create separate Layer 2 broadcast domains.
- Access ports normally belong to a single VLAN.
- Trunk ports carry traffic for multiple VLANs.
- VLAN IDs identify logically separated networks.
- VLAN segmentation is independent of whether devices use similar IP address patterns.

An important conceptual question during the lab was whether VLANs were necessary when devices appeared to use addresses from the same IP subnet.

This led to examining the distinction between:

- Layer 2 segmentation
- Layer 3 addressing
- Broadcast domains
- Routing between networks

VLANs are not simply a method of assigning different IP address ranges. They provide logical separation at Layer 2, while IP addressing and routing operate at Layer 3.

---

## 3. Router-on-a-Stick Configuration

The router was configured with a logical subinterface on its physical GigabitEthernet interface.

Example configuration:

```cisco
enable
configure terminal

interface GigabitEthernet0/0
 no shutdown
exit

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address <VLAN10_GATEWAY> <SUBNET_MASK>
exit

end
```

The subinterface was intended to serve as the default gateway for devices belonging to VLAN 10.

The configuration introduced the following concepts:

- Physical interfaces
- Logical subinterfaces
- IEEE 802.1Q VLAN tagging
- Gateway addresses
- Router-on-a-Stick
- Interface status verification

The router's interface configuration was inspected using:

```cisco
show ip interface brief
```

The output showed the configured subinterface and its operational status during the lab.

---

## 4. End-Device Addressing

The next planned step was to configure an end device in the same VLAN as the router subinterface.

The intended configuration consisted of:

- An IPv4 address from the VLAN subnet
- A subnet mask
- The router subinterface address as the default gateway

Example:

```text
IPv4 Address:      <PC_ADDRESS>
Subnet Mask:       <SUBNET_MASK>
Default Gateway:    <VLAN10_GATEWAY>
```

The purpose was to establish basic Layer 3 communication between the PC and its default gateway.

### Planned verification

```text
ping <VLAN10_GATEWAY>
```

However, the available progress record does not confirm a successful end-to-end ping test. Therefore, this milestone is documented as a planned verification step rather than a confirmed result.

---

## 5. Wireless Network Experiments

The project was also expanded toward wireless connectivity.

Experiments involved:

- Wireless interface configuration
- SSID configuration
- Wireless authentication settings
- DHCP versus static addressing
- Connecting simulated laptops and smartphones
- Router GUI configuration

Some wireless configuration attempts encountered limitations or inconsistencies in the Packet Tracer environment.

In particular, configuration options available through a device's GUI were not necessarily available through its CLI.

Wireless connection attempts also involved troubleshooting devices that retained automatically assigned addresses instead of receiving the intended network configuration.

### Observation

Packet Tracer models different device types with different levels of functionality. A configuration procedure that works for one simulated device may not be supported by another.

The simulator should not automatically be treated as equivalent to the behavior of real Cisco hardware.

---

## 6. DHCP and Network Addressing

DHCP configuration was considered as a way to automate address assignment for network clients.

The planned configuration included:

- DHCP address pools
- Network and subnet mask
- Default gateway option
- Automatic IP assignment to end devices
- DHCP lease verification

A DHCP-related verification command discussed during the project was:

```cisco
show ip dhcp binding
```

The lab also explored changing addressing schemes to make the simulated environment more representative of the existing home network.

However, the DHCP implementation and complete client verification were not confirmed as finished in the available progress record.

---

## 7. Network Automation Exploration

The project was extended beyond manual configuration into Python network automation.

Netmiko was introduced as a possible tool for connecting to network devices and executing commands programmatically.

A read-only command was selected as an initial automation target:

```python
show running-config
```

The intended workflow was:

1. Establish a connection to the network device.
2. Authenticate.
3. Execute a read-only command.
4. Collect the output.
5. Display or save the result.

The automation attempt encountered connection problems, including connection refusal and timeout errors.

The investigation raised questions about:

- Device reachability
- Management interfaces
- Telnet availability
- VTY configuration
- Simulation limitations
- Differences between simulated devices and real hardware

The automation experiment was not considered successfully completed.

---

## 8. Challenges and Troubleshooting

### Challenge 1 — Physical topology inconsistencies

**Problem:** The original diagram did not fully match the intended switched network architecture.

**Investigation:** The connections were reviewed before proceeding with logical configuration.

**Action:** The design was adjusted to place end devices behind the switch and use the router connection for the planned VLAN architecture.

**Lesson:** Validate the physical or simulated topology before troubleshooting higher network layers.

---

### Challenge 2 — Understanding VLANs versus IP subnets

**Problem:** The purpose of VLANs was questioned when devices appeared to have addresses from similar IP ranges.

**Investigation:** Layer 2 segmentation and Layer 3 addressing were examined separately.

**Outcome:** This became an important conceptual learning point in the project. VLAN configuration should be driven by the segmentation design, not merely by changing the third octet of an IP address.

---

### Challenge 3 — CLI and GUI capability differences

**Problem:** Some wireless configuration commands were unavailable on the simulated devices.

**Investigation:** Device-specific GUI and CLI capabilities were compared.

**Outcome:** The lab demonstrated that Packet Tracer device models have different supported feature sets.

**Lesson:** Confirm the capabilities of the selected simulated device before assuming that a configuration command is supported.

---

### Challenge 4 — Wireless connectivity and addressing

**Problem:** Wireless clients did not consistently obtain the expected network addresses.

**Investigation:** SSID settings, authentication, interface configuration, connection behavior, and addressing were examined.

**Outcome:** The wireless portion remained incomplete in the documented progress.

**Lesson:** Troubleshoot connectivity in layers rather than assuming an IP configuration issue is necessarily the root cause.

---

### Challenge 5 — Network automation connection failures

**Problem:** Python-based management connections did not successfully communicate with the simulated router.

**Investigation:** Connection errors and the differences between simulated device management and real network devices were examined.

**Outcome:** The automation workflow remained experimental.

**Lesson:** A successful automation script requires a reachable management plane, supported protocols, correct credentials, and a compatible target device.

---

## 9. Project Status

| Milestone | Status |
|---|---|
| Packet Tracer installation | Completed |
| Initial topology construction | Completed |
| Device cabling | Completed |
| Router CLI navigation | Completed |
| Physical router interface activation | Completed |
| VLAN 10 creation | Completed |
| Access port assignment | Completed |
| Router subinterface creation | Completed |
| 802.1Q encapsulation | Completed |
| Router subinterface IP configuration | Completed |
| Trunk configuration attempt | Completed |
| Full end-to-end gateway ping verification | Not confirmed |
| VLAN 20 implementation | Not confirmed |
| Inter-VLAN routing verification | Not completed |
| Complete DHCP implementation and validation | Not confirmed |
| Wireless connectivity | Experimental / incomplete |
| Netmiko automation | Experimental / incomplete |

---

## 10. Lessons Learned

1. A network topology should be validated before configuration begins.
2. Access ports and trunk ports serve different purposes.
3. VLANs separate Layer 2 broadcast domains.
4. IP addressing and VLAN membership are related but distinct concepts.
5. Router subinterfaces allow multiple logical networks to share one physical router connection.
6. Configuration commands should be verified with operational output rather than assumed successful.
7. Packet Tracer has device-specific limitations.
8. GUI and CLI functionality may differ between simulated devices.
9. Network automation depends on management-plane connectivity, not just the correctness of Python code.
10. In troubleshooting, planned, attempted, and verified actions must be documented separately.

---

## Conclusion

This project established the foundation for a more advanced network engineering lab.

The work covered initial topology design, switch configuration, VLAN concepts, trunking, router subinterfaces, addressing, and introductory network automation.

Although the complete network was not verified as operational, the project provided practical experience with Cisco-style CLI configuration and exposed several areas that require further investigation.

The next stage is to complete a clean, reproducible topology with verified gateway connectivity, expand the VLAN configuration, implement DHCP, and test inter-VLAN routing before introducing more advanced automation.

---

## Future Work

- Complete and verify VLAN 10 connectivity.
- Add additional VLANs with a clearly defined segmentation purpose.
- Implement inter-VLAN routing.
- Configure and verify DHCP.
- Test wireless connectivity using supported Packet Tracer devices.
- Capture and inspect ARP and DHCP traffic using Simulation Mode.
- Verify the network using `show vlan brief`, `show interfaces trunk`, and `show ip interface brief`.
- Explore Python automation against a supported network emulator or lab environment.
- Document the final topology and configuration files.

---

*Documentation note: This case records the work and learning process based on the available project history. Unverified connectivity and unfinished configurations are intentionally not presented as successful outcomes.*
