# Case 002: Home Network Performance Troubleshooting

## Investigation Date: June 2026


## Overview

This case documents troubleshooting of slow network performance and connectivity problems on an Ubuntu laptop connected to a home network through both Wi-Fi and Ethernet.

The investigation began with download speeds limited to approximately 2 MB/s and expanded to include Ethernet addressing problems, network interface behavior, routing, TLS connection stalls, physical network infrastructure, and bandwidth limitations.

The objective was to identify the difference between actual throughput limitations, local connectivity problems, and issues affecting specific internet destinations.

## Environment

### Client Device

- Device: Acer Aspire E1-531
- Operating System: Ubuntu
- Network interfaces:
  - Wi-Fi: `wlp3s0b1`
  - Ethernet: `enp2s0f0`
- Wi-Fi IPv4 address during investigation: `192.168.1.247`
- Ethernet IPv4 address during investigation: Initially `169.254.98.44`, later `192.168.1.183`

### Network Infrastructure

- ISP equipment: Optical Network Terminal (ONT)
- Router: TP-Link Archer C6
- Switch: D-Link
- Home LAN: `192.168.1.0/24`
- Router LAN address: `192.168.1.1`
- Wi-Fi network: `Archi`

### Connection Types

- Wi-Fi: 2.4 GHz connection investigated
- Ethernet: Wired connection through the home network
- Both Wi-Fi and Ethernet were active simultaneously during part of the investigation.

## Objectives

- Investigate download speeds limited to approximately 2 MB/s.
- Compare Wi-Fi and Ethernet connectivity.
- Identify why Ethernet initially received an invalid IPv4 configuration.
- Troubleshoot websites hanging during TLS handshake.
- Understand the behavior of simultaneous wired and wireless connections.
- Investigate potential network bottlenecks involving the router, ONT, Ethernet ports, Wi-Fi, and ISP bandwidth.
- Understand the distinction between link speed and actual internet throughput.

## Initial Symptoms

The original symptoms included:

- Download speed around 2 MB/s.
- Ethernet connection displaying a self-assigned IPv4 address.
- Wi-Fi connection functioning with a valid LAN address.
- Some websites failing to load or remaining stuck during TLS handshake.
- Network behavior changing when Wi-Fi or Ethernet was disabled.

The investigation was not limited to one confirmed root cause. Several issues appeared to coexist.

## Investigation

### 1. Initial Wi-Fi Performance

The Ubuntu laptop was connected to the home Wi-Fi network.

The observed download speed was approximately:

```text
2 MB/s
```

This is equivalent to approximately:

```text
16 Mbps
```

The Wi-Fi interface had a valid IPv4 address:

```text
192.168.1.247
```

The initial investigation considered several potential factors:

- 2.4 GHz Wi-Fi limitations.
- Wireless interference.
- Signal strength and distance.
- Router wireless configuration.
- ISP bandwidth limitations.
- Client wireless adapter capabilities.

The actual Wi-Fi throughput was not conclusively measured using a controlled series of speed tests.

### 2. Ethernet IPv4 Configuration Failure

The Ethernet interface initially displayed:

```text
169.254.98.44
```

This is an IPv4 link-local address assigned when normal IPv4 configuration through DHCP is unsuccessful.

The interface also displayed a reported link speed of 100 Mbps.

The presence of a physical Ethernet link did not confirm that the interface had successfully obtained network configuration or internet connectivity.

The investigation considered:

- Ethernet cable problems.
- Router port issues.
- DHCP availability.
- Network interface configuration.
- The possibility of a faulty or incomplete physical connection.

The exact reason for the initial DHCP failure was not established.

### 3. Simultaneous Wi-Fi and Ethernet Connectivity

Both network interfaces were enabled simultaneously.

The Ethernet interface had an invalid or unusable network configuration, while Wi-Fi was connected to the home LAN.

This raised questions about:

- Routing table selection.
- Interface metrics.
- Default gateway selection.
- Route priority.
- NetworkManager connection profiles.
- Traffic being sent through an interface without functional connectivity.

The Ethernet connection was temporarily disabled during troubleshooting to isolate the Wi-Fi connection.

Command used during investigation:

```bash
sudo ip link set enp2s0f0 down
```

This was a temporary diagnostic action intended to remove the Ethernet interface from active networking.

The investigation also considered disabling automatic connection to the wired network through Ubuntu's network settings.

### 4. Ethernet Connectivity Restored

Later in the investigation, the Ethernet connection was tested again with Wi-Fi disabled.

The Ethernet interface obtained the following IPv4 address:

```text
192.168.1.183
```

This indicated that the interface had successfully obtained a LAN address during that test.

The earlier `169.254.98.44` condition was therefore no longer present in that particular configuration.

The exact change that resolved the initial addressing problem was not conclusively documented.

### 5. TLS Handshake Stalls

Some websites remained inaccessible or stalled during the TLS handshake stage.

Other websites were accessible.

The issue was particularly relevant to specific VPN-related websites and services that the user needed to access.

The investigation considered several possible causes:

- DNS resolution problems.
- Network routing.
- ISP-level restrictions.
- Destination-specific connectivity problems.
- VPN service endpoint accessibility.
- TLS connection failures.

The fact that some websites worked while others did not suggested that the problem was not necessarily a complete loss of internet connectivity.

However, ISP blocking or interference was not conclusively established.

No definitive packet capture or controlled network comparison was documented to prove the exact cause.

### 6. DNS and Network Service Troubleshooting

DNS configuration and network service behavior were investigated.

Commands considered or used included:

```bash
sudo systemctl restart systemd-resolved
```

DNS cache flushing was also explored.

These actions were part of troubleshooting possible name-resolution or connection issues.

The investigation did not establish DNS as the definitive cause of the TLS handshake failures.

### 7. Throughput Testing

Several methods of measuring internet throughput were considered.

A command-line test using `curl` was proposed:

```bash
curl -o /dev/null -s -w 'Download Speed: %{speed_download} B/s\n' http://speedtest.tele2.net/100MB.zip
```

Browser-based speed tests were also considered.

The recommended comparison was to test the network under isolated conditions:

1. Wi-Fi enabled, Ethernet disconnected.
2. Ethernet enabled, Wi-Fi disabled.
3. Repeat tests under comparable conditions.

This approach was intended to distinguish wireless performance from wired performance and avoid ambiguity caused by simultaneous interfaces.

The results of a complete controlled test series were not recorded.

### 8. Wi-Fi Performance Factors

The investigation considered the limitations of the 2.4 GHz Wi-Fi band.

Potential contributing factors included:

- Interference from nearby wireless networks.
- Channel overlap.
- Distance from the router.
- Obstacles between the laptop and router.
- Limitations of the laptop's wireless adapter.
- Wireless standards and negotiated connection rates.

Wi-Fi channel selection was discussed, including channels 1, 6, and 11 as commonly used non-overlapping 2.4 GHz channel choices in many regulatory environments.

A 5 GHz-capable adapter was considered as a possible future improvement.

No confirmed hardware upgrade or final wireless optimization result was documented.

### 9. Physical Network Infrastructure and Bottlenecks

The physical network topology was reviewed.

The intended network structure was:

```text
ISP Fiber
    |
    v
ISP ONT
    |
    v
TP-Link Archer C6
    |
    v
D-Link Switch
    |
    +---- Wired Devices
    |
    +---- Wireless Clients
```

The investigation considered the difference between:

- Physical Ethernet link speed.
- Router and switch port capabilities.
- Client network adapter capabilities.
- ISP subscription bandwidth.
- Actual measured throughput.

Gigabit-capable physical links do not guarantee gigabit internet throughput.

Actual internet speed remains limited by the slowest relevant component, including the ISP plan, WAN connection, router performance, network congestion, and client hardware.

The physical port capabilities and ISP bandwidth were reviewed as part of the broader network-performance investigation.

### 10. Router QoS and Bandwidth Management

Quality of Service (QoS) was explored as a networking concept.

Topics included:

- Download bandwidth.
- Upload bandwidth.
- Device priority.
- Bandwidth allocation.
- Latency during network congestion.
- The difference between home-router QoS and enterprise QoS.

The investigation included educational discussion of how bandwidth values should be based on actual measurements rather than assumed speeds.

No confirmed QoS configuration change or measured performance improvement was documented.

## Observations

| Observation | Status |
|---|---|
| Wi-Fi download speed around 2 MB/s | Observed |
| Wi-Fi had a valid LAN IPv4 address | Observed |
| Ethernet initially received `169.254.98.44` | Observed |
| Ethernet later obtained `192.168.1.183` | Observed |
| Ethernet displayed a 100 Mbps link speed during initial testing | Observed |
| Some websites loaded while others stalled during TLS handshake | Observed |
| Simultaneous Wi-Fi and Ethernet complicated troubleshooting | Investigated |
| DNS was the definitive cause of TLS stalls | Not confirmed |
| ISP deliberately blocked the affected websites | Not confirmed |
| Wi-Fi interference was the definitive cause of low throughput | Not confirmed |
| Cable or router port failure was established | Not confirmed |
| QoS improved network throughput | Not confirmed |

## Troubleshooting Lessons

- A physical Ethernet link does not guarantee successful IP configuration.
- An IPv4 address in the `169.254.0.0/16` range can indicate that DHCP configuration failed.
- A valid LAN address does not guarantee internet connectivity.
- Simultaneous wired and wireless connections can complicate route selection and diagnosis.
- Network troubleshooting should isolate interfaces before comparing performance.
- Download speed in MB/s and link speed in Mbps are different units.
- Link speed, ISP bandwidth, and actual throughput are separate measurements.
- TLS handshake failures affecting only specific destinations should not automatically be attributed to DNS or the ISP without further evidence.
- Network performance should be tested using repeatable measurements.
- Physical infrastructure, client hardware, router configuration, and ISP service should be investigated separately.

## Outcome

The investigation identified and documented multiple network conditions:

- Wi-Fi throughput was approximately 2 MB/s during initial observations.
- Ethernet initially had an unusable link-local IPv4 configuration.
- Ethernet later obtained a valid LAN address.
- Some destination websites experienced TLS handshake stalls while others remained accessible.
- The network topology and potential throughput bottlenecks were reviewed.

The exact root cause of the initial low throughput and destination-specific connection failures was not conclusively established.

The case is therefore classified as an exploratory troubleshooting investigation with partial findings rather than a fully resolved performance issue.

## Future Work

- Perform controlled speed tests over Wi-Fi and Ethernet independently.
- Record upload and download throughput.
- Record latency and packet loss.
- Verify negotiated Ethernet link speed.
- Check router and switch port speeds.
- Test using a known-good Ethernet cable.
- Compare performance at different Wi-Fi distances.
- Investigate wireless channel utilization and interference.
- Verify routing tables and interface metrics while both interfaces are enabled.
- Compare DNS resolution and TLS connectivity for affected and unaffected destinations.
- Capture packets during TLS handshake failures if the issue recurs.
- Verify ISP bandwidth independently of local network performance.
- Review router QoS settings only after baseline measurements are collected.

## Conclusion

This investigation demonstrated the importance of separating local network connectivity problems from internet throughput limitations and destination-specific connectivity failures.

The initial Ethernet addressing problem, low Wi-Fi throughput, and TLS handshake stalls were separate symptoms that required independent testing.

Although several observations were documented and Ethernet connectivity was later demonstrated with a valid IPv4 address, the overall performance issue did not have a conclusively verified root cause.

The investigation provides a practical foundation for future network performance troubleshooting using controlled tests, interface isolation, routing analysis, and physical-layer verification.
