# Network Address Translation (NAT & PAT)

## Address Terminology
- **Inside Local:** Private RFC 1918 IP assigned to an internal host (`10.10.10.10`).
- **Inside Global:** Public routable IP representing the internal host to the outside world (`203.0.113.5`).
- **Outside Global:** Public IP assigned to an external Internet server (`198.51.100.20`).
- **Outside Local:** How the external host's IP appears to the inside network (typically identical to Outside Global).

## Translation Types
- **Static NAT:** 1-to-1 permanent mapping for internal servers requiring inbound reachability.
- **Dynamic NAT:** Many-to-many mapping from an internal ACL to a pool of public IPv4 addresses.
- **PAT (NAT Overload):** Many-to-one mapping multiplexing thousands of internal hosts onto a single public IPv4 address using unique Layer 4 source TCP/UDP port numbers.