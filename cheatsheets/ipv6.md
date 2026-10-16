# IPv6 Cheat Sheet

## Address Types & Prefixes
| Type | Prefix | Notes |
|---|---|---|
| Global Unicast (GUA) | `2000::/3` | Publicly routable Internet addresses |
| Link-Local (LLA) | `fe80::/10` | Auto-created on every interface; non-routable off-link |
| Unique Local (ULA) | `fc00::/7` (`fd00::/8`) | Internal private addressing |
| Multicast | `ff00::/8` | `ff02::1` (All Nodes), `ff02::2` (All Routers), `ff02::5` (OSPFv3 routers), `ff02::6` (OSPFv3 DR/BDR), `ff02::1:2` (DHCPv6 agents) |
| Solicited-Node Multicast | `ff02::1:ffXX:XXXX` | Used by Neighbor Solicitation (replaces ARP broadcast) |
| Loopback / Default | `::1/128` / `::/0` | Localhost loopback / Default route |
| Unspecified | `::/128` | Source address while a host has no address yet |

## Address Structure
- 128 bits = 8 quartets of 16 bits, written in hex.
- A typical host address is a **/64**: 48-bit global prefix + 16-bit subnet ID + 64-bit interface ID.
- No broadcast in IPv6 (multicast replaces it) and no network/broadcast host addresses to subtract.

## Compression Rules
1. Drop leading zeros in each quartet (`00ab` → `ab`, `0000` → `0`).
2. Replace **one** contiguous group of all-zero quartets with `::`.

Example: `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:db8::1`

## EUI-64 Interface ID (from a MAC address)
1. Split the MAC in half: `00:1A:2B | 3C:4D:5E`
2. Insert `FFFE` in the middle: `00:1A:2B:FF:FE:3C:4D:5E`
3. Flip the 7th bit of the first byte (U/L bit): `00` → `02`
4. Result interface ID: `021A:2BFF:FE3C:4D5E`

## Router Configuration
```ios
ipv6 unicast-routing                              ! Required to route IPv6 and send RAs
interface g0/0/0
 ipv6 address 2001:db8:acad:1::1/64               ! Static GUA
 ipv6 address fe80::1 link-local                  ! Easy-to-remember LLA
 no shutdown

interface g0/0/1
 ipv6 address 2001:db8:acad:2::/64 eui-64         ! Prefix + EUI-64 interface ID
 ipv6 enable                                      ! LLA only, no GUA

interface g0/0/2
 ipv6 address autoconfig                          ! Router acts as a SLAAC client
```

## IPv6 Static Routes
```ios
ipv6 route 2001:db8:acad:3::/64 2001:db8:acad:2::2        ! Next-hop (GUA)
ipv6 route 2001:db8:acad:3::/64 g0/0/1                    ! Exit interface (point-to-point)
ipv6 route 2001:db8:acad:3::/64 g0/0/1 fe80::2            ! Fully specified (needed for LLA next-hop)
ipv6 route ::/0 2001:db8:acad:2::2                        ! Default route
ipv6 route 2001:db8:acad:3::/64 2001:db8:acad:4::2 5      ! Floating static (AD 5)
```

## SLAAC vs. DHCPv6
- **SLAAC (`M=0, O=0`):** Prefix + Gateway from Router Advertisement (RA).
- **Stateless DHCPv6 (`M=0, O=1`):** IP from SLAAC; DNS/Domain from DHCPv6 (`ipv6 nd other-config-flag`).
- **Stateful DHCPv6 (`M=1`):** IP + DNS leased from DHCPv6 server (`ipv6 nd managed-config-flag`).

Full DHCPv6 configuration is in [`dhcp.md`](./dhcp.md).

## Neighbor Discovery (ICMPv6)
| Message | ICMPv6 Type | Purpose |
|---|---|---|
| Router Solicitation (RS) | 133 | Host asks "any routers?" (to `ff02::2`) |
| Router Advertisement (RA) | 134 | Router announces prefix, gateway, flags (to `ff02::1`) |
| Neighbor Solicitation (NS) | 135 | "Who has this address?" (like ARP request); also Duplicate Address Detection |
| Neighbor Advertisement (NA) | 136 | "I have it" (like ARP reply) |

## Verify
```ios
show ipv6 interface brief
show ipv6 interface g0/0/0                  ! Shows RA flags, joined multicast groups
show ipv6 route
show ipv6 route static
show ipv6 neighbors                         ! IPv6 equivalent of ARP table
show ipv6 protocols
ping 2001:db8:acad:2::2
traceroute 2001:db8:acad:3::1
```
```text
Windows : ipconfig /all   |  netsh interface ipv6 show neighbors  |  ping -6 <addr>
Linux   : ip -6 addr  |  ip -6 route  |  ip -6 neigh  |  ping -6 <addr>
```
