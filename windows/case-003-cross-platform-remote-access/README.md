# Case 003: Cross-Platform Remote Access and File Sharing Between Windows and Ubuntu

**Date:** July 2026  
**Investigation concluded:** July 30, 2026  
**Status:** Resolved  
**Environment:** Windows 11 and Ubuntu Linux  
**Case type:** Network configuration, remote desktop, SMB file sharing, cross-platform administration

## 1. Problem Overview

The objective was to establish reliable bidirectional file sharing and remote desktop access between two computers running different operating systems:

- Windows 11 PC (XeSPC)
- Ubuntu Linux PC (XePC)

The goal was to remotely control either computer from the other, access shared files over the local network, and avoid unnecessary third-party software or unstable configurations.

During troubleshooting, multiple remote desktop technologies were tested, removed, or replaced. Several configuration problems occurred, including connection failures, missing graphical settings, desktop environment changes, and confusion between different remote access protocols.

The final configuration provided working SMB file sharing and remote desktop access in both directions.

## 2. Hardware and Network Environment

### Windows PC — XeSPC

- Operating system: Windows 11
- Local IP address recorded: `192.168.1.217`
- Remote desktop software: TightVNC Server
- Firewall management: SimpleWall

### Ubuntu PC — XePC

- Hardware: Acer Aspire E1-531
- CPU: Intel Pentium B960
- RAM: 8 GB
- Graphics: Intel HD Graphics 2000
- Operating system: Ubuntu 26.04 LTS
- Desktop environment: GNOME 50
- Display server: Wayland
- Local IP address recorded: `192.168.1.247`

The systems were connected to the same local network.

*IP addresses represent the configuration recorded during troubleshooting and may change depending on DHCP or network configuration.*

## 3. Initial Objectives

The required functionality was:

1. Access Windows files from Ubuntu and vice versa using SMB.
2. Remotely control Ubuntu from Windows.
3. Remotely control Windows from Ubuntu.
4. Maintain access without repeatedly starting remote desktop servers manually.
5. Avoid unnecessary third-party applications and conflicting services.
6. Preserve the normal local desktop environment.

## 4. SMB File Sharing

Bidirectional SMB file sharing was configured between the computers.

The final historical status confirmed stable access to shared folders in both directions:

- Windows → Ubuntu
- Ubuntu → Windows

SMB was used for file access independently of the remote desktop configuration.

## 5. Remote Desktop Troubleshooting

### 5.1 Initial compatibility problems

Several remote desktop technologies were investigated:

- RDP
- xRDP
- VNC
- GNOME Remote Desktop
- TigerVNC
- x11vnc
- TightVNC
- Remmina

One important compatibility issue was identified during troubleshooting:

The Windows App was being used to attempt a connection to a VNC server. However, the Windows App was not a suitable VNC client for this configuration.

The connection failed with error:

`0x204`

The failure was initially approached as a server, network, firewall, or port problem, although client/server protocol compatibility was also an important factor.

## 6. Ubuntu Remote Desktop Configuration Issues

The Ubuntu system experienced several problems during configuration.

### Remote Desktop settings disappeared

The graphical Remote Desktop section was no longer available in Ubuntu Settings during troubleshooting.

Restarting the GNOME Remote Desktop service did not restore the missing graphical settings.

### Desktop environment changed unexpectedly

An attempt to start TigerVNC with an XFCE session resulted in changes to the local desktop appearance.

The graphical interface, application icons, settings windows, and file manager appearance changed significantly.

This created additional troubleshooting work and required restoring the expected desktop environment.

### Multiple server implementations caused confusion

Different remote desktop services and configurations were tested during the investigation.

The coexistence of several implementations made it difficult to determine:

- Which server was actually running
- Which port was listening
- Which display session was being shared
- Which protocol the client expected
- Whether a connection was targeting the physical desktop or a separate virtual session

Several third-party remote desktop implementations were subsequently removed.

## 7. Final Remote Access Configuration

The final configuration used different remote desktop software depending on the direction of the connection.

### Windows → Ubuntu

- Server: Ubuntu's built-in GNOME Remote Desktop VNC functionality
- Client: TightVNC Viewer on Windows
- Port: `5900`
- Ubuntu IP recorded: `192.168.1.247`

Connection target:

`192.168.1.247:5900`

### Ubuntu → Windows

- Server: TightVNC Server on Windows
- Client: Remmina on Ubuntu
- Port: `5900`
- Windows IP recorded: `192.168.1.217`

Connection target:

`192.168.1.217:5900`

The final status recorded both directions of remote control as operational.

## 8. Firewall and Network Configuration

Firewall configuration was investigated as part of the connection troubleshooting.

On Ubuntu, UFW was used to add an inbound rule for the VNC port:

```bash
sudo ufw allow 5900/tcp
sudo ufw reload
```

Port accessibility and running services were investigated using commands such as:

```bash
ss -tlnp
```

The distinction between a firewall rule and an active listening service became an important part of the troubleshooting process.

Opening a port alone does not guarantee that a remote connection will succeed. A compatible server must also be running and listening on the correct interface and port.

## 9. Failed and Abandoned Approaches

The following approaches were tested or investigated and were not retained in the final configuration:

- xRDP
- x11vnc
- TigerVNC with XFCE
- RemoteRipple
- Attempts to restore the missing graphical Remote Desktop settings through repeated service restarts
- Attempts to use the Windows App as a VNC client

These approaches either introduced additional configuration complexity, failed to achieve the required result, or were abandoned in favor of the final configuration.

They were removed from the final software stack where applicable.

## 10. Final Result

By July 30, 2026, the recorded configuration was:

| Function | Connection | Software | Status |
|---|---|---|---|
| File sharing | Windows ↔ Ubuntu | SMB | Working |
| Remote desktop | Windows → Ubuntu | TightVNC Viewer → GNOME VNC | Working |
| Remote desktop | Ubuntu → Windows | Remmina → TightVNC Server | Working |

The unnecessary remote desktop software stack was removed.

The final setup provided bidirectional file sharing and remote control between both computers.

## 11. Lessons Learned

### Protocol compatibility matters

A remote desktop client cannot necessarily connect to every remote desktop server.

RDP and VNC are different protocols. A connection failure does not automatically mean the server is offline or the firewall is blocking traffic.

### Avoid running multiple remote desktop implementations simultaneously

Installing and configuring multiple remote desktop servers can introduce conflicts involving:

- Display sessions
- Ports
- Authentication
- Desktop environments
- Startup processes
- Network interfaces

A simpler configuration is easier to diagnose and maintain.

### Verify the listening service before changing the firewall

Check whether a service is running and listening on the expected port before repeatedly modifying firewall rules.

### Separate file sharing from remote control

SMB and remote desktop serve different purposes.

SMB provides file access, while VNC or RDP provides interactive desktop control. Troubleshooting one service should not unnecessarily alter the other.

### Be cautious when launching alternate desktop environments

Starting a remote session with a different desktop environment can affect the local graphical session or user experience. The behavior of the selected remote session should be understood before launching it.

## 12. AI-Assisted Troubleshooting Review

AI assistance was used extensively to explore possible configurations and troubleshoot connection failures.

Some recommendations led to unnecessary complexity, repeated attempts, or incorrect assumptions about protocol compatibility and system behavior.

The investigation demonstrated the importance of:

- Verifying commands against actual system output
- Confirming which service is running
- Checking the protocol supported by the client
- Avoiding claims that a solution is guaranteed
- Reconsidering assumptions when repeated steps do not change the outcome
- Removing abandoned configurations rather than leaving multiple competing services installed

The final working configuration was simpler than several of the intermediate configurations proposed during troubleshooting.

## 13. Conclusion

A functional cross-platform administration setup was established between Windows 11 and Ubuntu Linux.

The final system supported bidirectional SMB file sharing and remote desktop access using a minimal set of retained tools.

The main technical lesson was that successful remote access depends on the correct combination of server, client, protocol, port, authentication, and desktop session—not simply opening a firewall port.

This case documents the troubleshooting process, the abandoned approaches, and the final working configuration as a practical cross-platform networking and administration project.
