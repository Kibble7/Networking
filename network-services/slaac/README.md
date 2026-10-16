# SLAAC (Stateless Address Autoconfiguration)

## Operation
1. Host enables IPv6 interface and generates a Link-Local address (`fe80::/10`).
2. Host sends an **ICMPv6 Router Solicitation (RS - Type 133)** to `ff02::2` (All-Routers multicast).
3. Router replies with an **ICMPv6 Router Advertisement (RA - Type 134)** to `ff02::1` (All-Nodes multicast) containing the `/64` prefix and prefix length.
4. Host appends a 64-bit interface ID (via EUI-64 or RFC 4941 random privacy generation) and performs Duplicate Address Detection (DAD) via Neighbor Solicitation before binding the address.