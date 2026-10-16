---

## File: `networking-portfolio/labs/services/dhcp-and-ipv6-services.md`


# DHCPv4 Relay, SLAAC, Stateless & Stateful DHCPv6

## Objective
This lab documents the configuration and verification process for IOS DHCPv4 server pools, DHCPv4 relay agent forwarding (`ip helper-address`), IPv6 SLAAC, Stateless DHCPv6, and Stateful DHCPv6.

## Skills Demonstrated
- IOS DHCPv4 address exclusions and pool configuration
- DHCP Relay (`ip helper-address`) across Layer 3 VLAN boundaries
- IPv6 SLAAC router advertisement configuration
- Stateless DHCPv6 (`ipv6 nd other-config-flag`)
- Stateful DHCPv6 (`ipv6 nd managed-config-flag`)

## Networking Concepts
- **DHCPv4 & Relay:** Clients use broadcast DORA messages to acquire IPv4 parameters. When the DHCP server resides in a centralized subnet, the router gateway uses `ip helper-address` to relay broadcast requests as unicast packets.
- **IPv6 Autoconfiguration:** Governed by ICMPv6 Router Advertisement (RA) flags:
  - **SLAAC (`M=0, O=0`):** Host derives `/64` prefix and default gateway purely from the RA.
  - **Stateless DHCPv6 (`M=0, O=1`):** Host uses SLAAC for its IPv6 address but queries a DHCPv6 server for DNS/domain options.
  - **Stateful DHCPv6 (`M=1`):** Host obtains its IPv6 address and DNS options from a stateful DHCPv6 pool.

## Topology
![DHCP and IPv6 Services Topology](../../topologies/images/dhcp-ipv6-topology.png)
`R1` acts as the DHCPv4/v6 server for VLAN 10 (Stateless DHCPv6) and VLAN 20 (Stateful DHCPv6 + Relayed DHCPv4).

## Addressing Plan
| VLAN | IPv4 Pool / Exclusions | IPv6 Prefix | IPv6 Allocation Mode |
|---|---|---|---|
| VLAN 10 (Eng) | `10.10.10.0/24` (Exclude `.1 – .9`) | `2001:db8:acad:10::/64` | Stateless DHCPv6 (`O=1`) |
| VLAN 20 (Sales) | Relayed to `10.10.99.50` | `2001:db8:acad:20::/64` | Stateful DHCPv6 (`M=1`) |

## Configuration
```ios
! 1. Configure IOS DHCPv4 Server Pool for VLAN 10 on R1
ip dhcp excluded-address 10.10.10.1 10.10.10.9
ip dhcp pool ENG_VLAN10_POOL
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.99.50
 domain-name corp.local

! 2. Configure DHCPv4 Relay on VLAN 20 Subinterface
interface GigabitEthernet0/0/0.20
 ip helper-address 10.10.99.50

! 3. Configure Stateless DHCPv6 for VLAN 10 (SLAAC + DNS)
ipv6 unicast-routing
ipv6 dhcp poolIPV6_STATELESS_ENG
 dns-server 2001:db8:acad:99::50
 domain-name corp.local

interface GigabitEthernet0/0/0.10
 ipv6 nd other-config-flag
 ipv6 dhcp server IPV6_STATELESS_ENG

! 4. Configure Stateful DHCPv6 for VLAN 20
ipv6 dhcp pool IPV6_STATEFUL_SALES
 address prefix 2001:db8:acad:20::/64
 dns-server 2001:db8:acad:99::50
 domain-name sales.corp.local

interface GigabitEthernet0/0/0.20
 ipv6 nd managed-config-flag
 ipv6 nd prefix 2001:db8:acad:20::/64 no-autoconfig
 ipv6 dhcp server IPV6_STATEFUL_SALES

```

## Verification
- show ip dhcp binding — Confirm leased IPv4 addresses, client MAC addresses, and lease expiration timers.

- show ip dhcp pool — Verify pool utilization and excluded range counts.

- show ipv6 dhcp pool — Inspect active client count (Active clients: 1 for Stateful DHCPv6).

- show ipv6 dhcp binding — Confirm DUID-to-IPv6 address mappings for VLAN 20 stateful clients.

- show ipv6 interface G0/0/0.10 — Verify Hosts use stateless autoconfig for addresses and Hosts use DHCP to obtain other configuration.

## Key Takeaways
This lab demonstrates automated enterprise address management across both IPv4 and IPv6, contrasting broadcast-to-unicast DHCPv4 relay with RA flag-driven IPv6 autoconfiguration.
