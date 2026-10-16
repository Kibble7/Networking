# Default Routing (Gateway of Last Resort)

## Core Concepts
A default route (`0.0.0.0/0` in IPv4, `::/0` in IPv6) matches any destination packet that does not match a more specific prefix in the routing table. Ideal for stub or edge routers connecting to an ISP.

```ios
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ipv6 route ::/0 2001:db8:isp::1