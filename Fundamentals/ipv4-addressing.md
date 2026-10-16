# IPv4 Addressing
**Classification:** Networking Fundamentals (Reference / Knowledge)

## Structure & Transmission Types
- **32-Bit Address:** Divided into 4 dotted-decimal octets (`0–255`), segmented by a subnet mask into a **Network Prefix** and **Host Identifier**.
- **Unicast:** One-to-one communication targeting a single host interface.
- **Broadcast:** One-to-all communication within a local subnet (all host bits set to `1`, or limited broadcast `255.255.255.255`).
- **Multicast:** One-to-many communication targeting subscribed group members (`224.0.0.0/4`).

## Reserved & Private IPv4 Ranges
| Block | Range | Scope & Purpose |
|---|---|---|
| `10.0.0.0/8` | `10.0.0.0 – 10.255.255.255` | RFC 1918 Private Enterprise Space |
| `172.16.0.0/12` | `172.16.0.0 – 172.31.255.255` | RFC 1918 Private Enterprise Space |
| `192.168.0.0/16` | `192.168.0.0 – 192.168.255.255` | RFC 1918 Private LAN Space |
| `127.0.0.0/8` | `127.0.0.1 – 127.255.255.254` | Host Loopback Stack Verification |
| `169.254.0.0/16` | `169.254.0.1 – 169.254.255.254` | APIPA / Link-Local (Assigned when DHCP fails) |