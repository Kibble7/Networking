## File: `networking-portfolio/labs/services/nat-and-pat.md`


# Static NAT, Dynamic NAT & PAT (NAT Overload)

## Objective
This lab documents the configuration and verification process for translating private RFC 1918 IPv4 addresses to public routable IPv4 addresses using Static NAT, Dynamic NAT Pools, and Port Address Translation (PAT / NAT Overload).

## Skills Demonstrated
- Inside/Outside NAT interface designation (`ip nat inside`, `ip nat outside`)
- Static 1-to-1 NAT mapping for internal server publishing
- Dynamic NAT pool creation and ACL binding
- PAT (NAT Overload) using exit interface and pool overload
- NAT translation table inspection and troubleshooting

## Networking Concepts
RFC 1918 private addresses (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) are dropped by public Internet routers. **Static NAT** provides permanent bidirectional 1-to-1 mapping for DMZ servers. **Dynamic NAT** maps internal hosts to an available public IP from a pool. **PAT (`overload`)** maps multiple private Inside Local addresses to a single Inside Global public IP by tracking unique Layer 4 TCP/UDP port numbers.

## Topology
![NAT and PAT Topology](../../topologies/images/nat-pat-topology.png)
`R1` translates traffic arriving on internal subinterfaces (`ip nat inside`) before forwarding out `G0/0/1` (`ip nat outside`) toward the ISP (`203.0.113.1/30`).

## Addressing Plan
| Translation Type | Inside Local | Inside Global | Mechanism |
|---|---|---|---|
| Static NAT (Web Server) | `10.10.99.50` | `203.0.113.10` | Permanent 1:1 Mapping |
| Dynamic NAT Pool | `10.10.20.0/24` | `203.0.113.20 – .25` | Pool `PUBLIC_POOL` |
| PAT (Overload) | `10.10.10.0/24` | `203.0.113.2` (`G0/0/1`) | Port Multiplexing |

## Configuration
```ios
! 1. Designate Inside and Outside Interfaces
interface GigabitEthernet0/0/0.10
 ip nat inside
interface GigabitEthernet0/0/0.20
 ip nat inside
interface GigabitEthernet0/0/0.99
 ip nat inside
interface GigabitEthernet0/0/1
 ip nat outside

! 2. Configure Static NAT for Internal Server
ip nat inside source static 10.10.99.50 203.0.113.10

! 3. Configure Dynamic NAT Pool for VLAN 20
ip access-list standard ACL_NAT_VLAN20
 permit 10.10.20.0 0.0.0.255
exit
ip nat pool PUBLIC_POOL 203.0.113.20 203.0.113.25 netmask 255.255.255.240
ip nat inside source list ACL_NAT_VLAN20 pool PUBLIC_POOL

! 4. Configure PAT (NAT Overload) for VLAN 10 using Outside Interface IP
ip access-list standard ACL_PAT_VLAN10
 permit 10.10.10.0 0.0.0.255
exit
ip nat inside source list ACL_PAT_VLAN10 interface GigabitEthernet0/0/1 overload
```

## Verification
- show ip nat translations — Verify active Pro, Inside global, Inside local, Outside local, and Outside global address/port mappings.

- show ip nat statistics — Confirm total active translations, inside/outside interfaces, hit/miss counters, and pool allocation percentage.

## Troubleshooting
Problem
Hosts in VLAN 10 (10.10.10.0/24) can ping R1's outside interface (203.0.113.2) but cannot ping the ISP router (203.0.113.1), and show ip nat translations is empty for VLAN 10.

## Investigation
Executed show ip nat statistics and inspected running-config on G0/0/0.10.

## Root Cause
ip nat inside was omitted from subinterface GigabitEthernet0/0/0.10, causing packets from VLAN 10 to be routed to the ISP with an untranslated private source IP (10.10.10.10).

## Resolution
Applied ip nat inside under interface GigabitEthernet0/0/0.10.

## Verification
Generated traffic from PC-A and verified PAT port translations appeared in show ip nat translations.

## Key Takeaways
This lab demonstrates how edge routers conserve public IPv4 space via PAT while selectively publishing internal services via Static NAT.