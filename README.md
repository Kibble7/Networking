# Enterprise Networking Portfolio

Professional networking portfolio demonstrating CCNA-level network architecture, protocol analysis, CLI configuration workflows, security hardening, and structured troubleshooting methodologies.

## Repository Navigation

- **[Fundamentals](./fundamentals/README.md)** — Core reference documentation for OSI & TCP/IP models, Ethernet, IPv4, IPv6, Subnetting, CIDR, ARP, and TCP vs. UDP.
- **[Switching](./switching/README.md)** — Layer 2 & Layer 3 switching architecture: VLANs, 802.1Q Trunking, Inter-VLAN Routing, Rapid STP, EtherChannel, and Switch Security.
- **[Routing](./routing/README.md)** — Path selection and static routing architectures: Static, Default, Floating Static, Host Routes, and Routing Troubleshooting.
- **[Network Services](./network-services/README.md)** — Infrastructure services: DHCPv4, DHCP Relay, DHCPv6, SLAAC, NAT/PAT, NTP, CDP/LLDP, and TFTP Backup.
- **[Security](./security/README.md)** — Defense-in-depth infrastructure controls: Standard/Extended/Named ACLs, Port Security, DHCP Snooping, Device Hardening, and SSHv2.
- **[Troubleshooting](./troubleshooting/README.md)** — Structured OSI/TCP-IP fault isolation methodologies and verification workflows.
- **[Hands-On Labs](./labs/README.md)** — Complete step-by-step configuration, verification, and troubleshooting lab documentation.
- **[Topologies](./topologies/README.md)** — Network architecture diagrams, Layer 2/Layer 3 boundaries, and traffic flows.
- **[Cheat Sheets](./cheatsheets/README.md)** — Concise operational reference guides and CLI command tables.
- **[Documentation & Indexes](./docs/networking-skills-matrix.md)** — [Networking Skills Matrix](./docs/networking-skills-matrix.md) and [Lab Index](./docs/lab-index.md).

---

## Networking Skills

### Network Fundamentals (Reference / Knowledge)
- IPv4 Addressing, Private RFC 1918 Ranges, Loopback & APIPA
- IPv6 Addressing (Global Unicast, Link-Local, Multicast, EUI-64)
- Subnetting (FLSM & VLSM)
- Classless Inter-Domain Routing (CIDR) & Route Summarization
- Ethernet Frame Structure (IEEE 802.3) & MAC Learning/Forwarding
- Address Resolution Protocol (ARP) & IPv6 Neighbor Discovery (NDP)
- TCP/IP Protocol Suite & Encapsulation
- OSI 7-Layer Reference Model

### Switching (Configuration & Verification Documented)
- VLAN Segmentation, Access Ports & Voice VLANs (`mls qos trust cos`)
- IEEE 802.1Q Trunking, Native VLAN Security & DTP Mitigation
- Inter-VLAN Routing (Router-on-a-Stick & Layer 3 Switch SVIs)
- Spanning Tree Protocol (STP), Rapid PVST+ (802.1w), Root Bridge Election
- EtherChannel Link Aggregation & IEEE 802.3ad LACP

### Routing (Configuration & Verification Documented)
- IPv4 & IPv6 Static Routing (Next-Hop, Exit-Interface, Fully Specified)
- Default Routing (`0.0.0.0/0` and `::/0`)
- Floating Static Routes & Administrative Distance Tuning
- Host Routes (`/32` and `/128`) & Routing Table Evaluation (Longest-Prefix Match)
- Dynamic Routing Concepts (Administrative Distance, Metrics, OSPF Overview)

### Network Services (Configuration & Verification Documented)
- DHCPv4 Server Pools, Excluded Addresses & DHCP Relay (`ip helper-address`)
- DHCPv6 (Stateless & Stateful)
- SLAAC (Stateless Address Autoconfiguration & Router Advertisement Flags)
- Static NAT, Dynamic NAT & PAT (NAT Overload)
- DNS Resolution Concepts
- NTP Time Synchronization, CDP/LLDP Discovery & TFTP Configuration Backup

### Security (Configuration & Verification Documented)
- Standard, Extended & Named IPv4 ACLs
- VTY Line Access Control (`access-class`)
- Layer 2 Port Security (Sticky MAC, Violation Modes)
- DHCP Snooping & Trusted Interface Enforcement
- Rapid STP Edge Protection (`PortFast` & `BPDU Guard`)
- Secure Remote Management via SSHv2
- Network Device Hardening (Secret Hashing, Encryption, Banners, Exec Timeouts)

### Troubleshooting (Configuration & Verification Documented)
- End-to-End Connectivity Testing (`ping`, `traceroute`, Extended Ping)
- IPv4/IPv6 Addressing, Subnet Mask & Default Gateway Troubleshooting
- Layer 2 VLAN, 802.1Q Trunk, STP & EtherChannel Troubleshooting
- Layer 3 Next-Hop, Return Path & Administrative Distance Troubleshooting
- Systematic CLI Configuration Verification