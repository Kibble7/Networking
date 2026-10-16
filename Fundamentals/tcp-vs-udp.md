# Transport Layer: TCP vs. UDP
**Classification:** Networking Fundamentals (Reference / Knowledge)

## Operational Comparison
| Characteristic | TCP (RFC 9293) | UDP (RFC 768) |
|---|---|---|
| Connection Orientation | Connection-oriented (3-Way Handshake: `SYN → SYN/ACK → ACK`) | Connectionless |
| Sequencing | Reorders out-of-order segments via Sequence Numbers | No sequencing |
| Reliability | Positive Acknowledgments (`ACK`) and retransmission timers | Best-effort (handled by upper layers if needed) |
| Flow Control | Dynamic Windowing | None |
| Header Size | 20–60 Bytes | 8 Bytes |
| Common Protocols | SSH (`22`), HTTP (`80`), HTTPS (`443`), BGP (`179`) | DHCP (`67/68`), TFTP (`69`), NTP (`123`), SNMP (`161`) |