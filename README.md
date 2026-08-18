# Enterprise Multi-Site Network Architecture

##  Executive Summary
This project demonstrates the end-to-end design, configuration, and deployment of a highly available, secure, and scalable enterprise network infrastructure. Built utilizing Cisco Packet Tracer, the architecture spans a Layer 3 Core and three distinct physical sites (IT Department, Office 1, and Office 2). The design prioritizes defense-in-depth security, seamless gateway redundancy, and centralized auditing to meet modern enterprise business requirements.

## Business Objectives & Solutions
* **Maximum Uptime & Redundancy:** Eliminated single points of failure at the distribution layer using **HSRP (Hot Standby Router Protocol)** and logically aggregated physical links using **EtherChannel (Port-Channels)** to prevent network loops while maximizing bandwidth.
* **Zero-Trust Internal Security:** Hardened user-facing access layers via **Port Security (Sticky MAC)**, strict **VTY Access Control Lists (ACLs)**, **DHCP Snooping**, and **Dynamic ARP Inspection (DAI)** to prevent Man-in-the-Middle (MitM) attacks. Administrative SSH access is completely restricted to dedicated IT management subnets.
* **Public & Guest Isolation:** Engineered a dedicated Guest WiFi network isolated by **Extended ACLs**, permitting internet access while completely blocking horizontal movement into corporate `10.x.x.x` infrastructure.
* **Centralized Compliance Auditing:** Standardized network time via an **NTP Server** and centralized all switch/router trap alerts to a dedicated **Syslog Server** for real-time security monitoring and hardware diagnostics.

##  Technical Architecture & Protocols
### Routing & Switching Edge
* **OSPF Area 0 (Single-Area):** Configured across the Core and Distribution layers utilizing `/30` transit links and strict `router-id` loopbacks to prevent rogue gateway elections. Passive interfaces were deployed on all user-facing SVIs to secure the routing table.
* **NAT Overload (PAT):** Deployed a dedicated Edge Router (`R1`) translating all internal corporate traffic to a simulated ISP using an overload rule to conserve public IP space.
* **Layer 2 Topology:** Enforced **PVST+ (Per-VLAN Spanning Tree)** to guarantee loop-free topology scaling alongside **BPDU Guard** to prevent rogue switches from hijacking the spanning tree root.

### Subnet & VLAN Allocation
The network utilizes a modular IP schema to allow for rapid scalability across physical buildings. 

| VLAN ID | Network Name | IP Subnet | Active HSRP Gateway | Gateway Preemption |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | IT Office | `10.0.10.0/24` | DSW-A1 (`10.0.10.1`) |  Enabled |
| **VLAN 20** | Office 1 | `10.0.20.0/24` | DSW-B1 (`10.0.20.1`) |  Enabled |
| **VLAN 30** | Office 2 | `10.0.30.0/24` | DSW-C1 (`10.0.30.1`) |  Enabled |
| **VLAN 40** | Corp WiFi | `10.0.40.0/24` | DSW-A1 (`10.0.40.1`) |  Enabled |
| **VLAN 50** | Guest WiFi | `10.0.50.0/24` | DSW-B1 (`10.0.50.1`) |  Enabled |
| **VLAN 99** | IT Mgmt | `10.0.99.0/24` | DSW-A1 (`10.0.99.1`) |  Enabled |
| **VLAN 199**| Office 1 Mgmt| `10.0.29.0/24` | DSW-B1 (`10.0.29.1`) |  Enabled |
| **VLAN 299**| Office 2 Mgmt| `10.0.39.0/24` | DSW-C1 (`10.0.39.1`) |  Enabled |

*(Note: Layer 3 Core transits utilize strict `10.0.100.x /30` point-to-point networks).*

## Security Hardening Implementations
1. **Management Plane Protection:** Telnet is globally disabled. Management access is enforced via **SSHv2** using RSA 2048-bit encryption keys.
2. **Password Encryption:** Global `service password-encryption` enabled to eliminate plain-text credentials in the configuration files, paired with Type 5 secure hashing for the administrative account.
3. **Guest Network Containment:** The `GUEST_ISOLATION` Extended ACL is applied inbound on the Guest WiFi VLAN, dropping all packets destined for private address space while allowing DHCP, DNS, and HTTP/HTTPS outbound.
4. **Layer 2 Threat Mitigation:** Deployed **DHCP Snooping** and **Dynamic ARP Inspection (DAI)** across all access switches, establishing explicit trust boundaries on distribution uplinks and server ports to neutralize rogue DHCP servers and ARP spoofing.

##  How to Navigate This Repository
* `/Configurations/` - Contains the raw Cisco IOS `running-config` text files for all Core Switches (CSW), Distribution Switches (DSW), Access Switches (ASW), and Edge Routers.
* `/Documentation/` - Contains the master `Connection layout.txt` file detailing physical port mappings, IP assignments, and infrastructure documentation.
* `/PacketTracer/` - Contains the raw `.pkt` project file to boot the fully simulated environment.
