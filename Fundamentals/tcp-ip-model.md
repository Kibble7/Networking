# TCP/IP Protocol Model
**Classification:** Networking Fundamentals (Reference / Knowledge)

## Architecture Mapping
| TCP/IP Layer | OSI Equivalent | Core Protocols | Primary Addressing |
|---|---|---|---|
| Application | Layers 5, 6, 7 | SSH, HTTP/S, DNS, DHCP, NTP, TFTP | Process / URL / Hostname |
| Transport | Layer 4 | TCP, UDP | 16-bit Port Numbers (`0–65535`) |
| Internet | Layer 3 | IPv4, IPv6, ICMP, ICMPv6 | 32-bit IPv4 / 128-bit IPv6 Logical Addresses |
| Network Access | Layers 1, 2 | Ethernet (802.3), WLAN (802.11), ARP | 48-bit MAC Hardware Addresses |