# Ethernet & Layer 2 Switching
**Classification:** Networking Fundamentals (Reference / Knowledge)

## IEEE 802.3 Ethernet Frame Format
| Field | Length | Function |
|---|---|---|
| Preamble + SFD | 7 + 1 Bytes | Bit-level clock synchronization (`10101010...`) and Start Frame Delimiter (`10101011`) |
| Destination MAC | 6 Bytes | 48-bit MAC address of receiving interface (Unicast, Multicast, or Broadcast `FFFF.FFFF.FFFF`) |
| Source MAC | 6 Bytes | 48-bit MAC address of sending interface (24-bit OUI + 24-bit NIC ID) |
| EtherType / 802.1Q | 2 Bytes (+4 Bytes if tagged) | Identifies Layer 3 protocol (`0x0800` IPv4, `0x86DD` IPv6, `0x0806` ARP, `0x8100` VLAN Tag) |
| Payload (Data) | 46–1500 Bytes | Encapsulated Layer 3 Packet (padded if under 46 bytes to maintain 64-byte minimum frame) |
| FCS | 4 Bytes | 32-bit Cyclic Redundancy Check (CRC) for frame integrity verification |

## Switch MAC Address Table Operations
1. **Learning:** Inspects the **Source MAC** and ingress port of every incoming frame and records the mapping in the CAM table (default aging timer: 300 seconds).
2. **Flooding:** If the **Destination MAC** is unknown unicast, broadcast, or multicast, floods the frame out all ports in the same VLAN except the ingress port.
3. **Forwarding:** If the **Destination MAC** exists in the CAM table, forwards the frame solely out the associated egress port.
4. **Filtering:** Drops the frame if the Source and Destination MAC addresses reside on the same port.