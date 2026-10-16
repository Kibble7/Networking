# OSI 7-Layer Model
**Classification:** Networking Fundamentals (Reference / Knowledge)

## Layer Breakdown
| Layer | Name | PDU | Core Responsibility | Protocols & Hardware |
|---|---|---|---|---|
| 7 | Application | Data | Network interface for user applications | HTTP/HTTPS, DNS, DHCP, SSH, TFTP, NTP |
| 6 | Presentation | Data | Data formatting, encryption, compression | TLS/SSL, ASCII, UTF-8, JSON |
| 5 | Session | Data | Session establishment, maintenance, teardown | RPC, NetBIOS, SOCKS |
| 4 | Transport | Segment / Datagram | End-to-end delivery, port multiplexing, flow control | TCP, UDP |
| 3 | Network | Packet | Logical addressing and path determination | IPv4, IPv6, ICMP, OSPF, Routers, L3 Switches |
| 2 | Data Link | Frame | MAC addressing, VLAN tagging, error detection | Ethernet (802.3), 802.1Q, STP, L2 Switches |
| 1 | Physical | Bit | Binary signal transmission across physical media | Cat6 UTP, Single/Multimode Fiber, RJ-45, SFP |

## Encapsulation & Decapsulation
- **Encapsulation (Top-Down):** Application Data → L4 Header added (Segment) → L3 IP Header added (Packet) → L2 MAC Header + FCS Trailer added (Frame) → L1 Bits transmitted.
- **Decapsulation (Bottom-Up):** Receiving NIC verifies L2 FCS trailer, strips L2 header, passes Packet to L3 IP stack, strips L3 header, passes Segment to L4 port socket, and delivers payload to the Application.