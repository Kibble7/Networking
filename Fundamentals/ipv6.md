# IPv6 Addressing
**Classification:** Networking Fundamentals (Reference / Knowledge)

## 128-Bit Format & Compression Rules
Written as 8 hexadecimal quartets separated by colons (e.g., `2001:0db8:0000:0010:0000:0000:0000:0001`).
1. **Rule 1 (Leading Zeros):** Omit leading zeros in any quartet (`0db8` → `db8`, `0001` → `1`).
2. **Rule 2 (Double Colon `::`):** Compress a single contiguous sequence of all-zero quartets into `::` **once** per address (`2001:db8:0:10::1`).

## Address Classifications
| Type | Prefix | Routing Scope | Notes |
|---|---|---|---|
| Global Unicast (GUA) | `2000::/3` | Globally Routable | Standard `/64` subnet prefix |
| Link-Local (LLA) | `fe80::/10` | Local Link Only | Required on every IPv6 interface; used as next-hop |
| Unique Local (ULA) | `fc00::/7` (`fd00::/8`) | Internal Organization | Private internal routing equivalent |
| Multicast | `ff00::/8` | Scoped Group | Replaces broadcast (`ff02::1` All Nodes, `ff02::2` All Routers) |
| Loopback | `::1/128` | Host Internal | Local TCP/IPv6 verification |