# Networking Quick Reference (Master Summary)

A one-page summary. Follow the links for the full detail and configuration.

## OSI vs TCP/IP
| OSI Layer | TCP/IP Layer | PDU | Examples | Devices |
|---|---|---|---|---|
| 7 Application | Application | Data | HTTP, DNS, DHCP, SSH | - |
| 6 Presentation | Application | Data | TLS, encoding, compression | - |
| 5 Session | Application | Data | Session setup / teardown | - |
| 4 Transport | Transport | Segment / Datagram | TCP, UDP | Firewalls (stateful) |
| 3 Network | Internet | Packet | IP, ICMP, OSPF | Routers, L3 switches |
| 2 Data Link | Network Access | Frame | Ethernet, 802.1Q, STP, ARP | Switches, APs |
| 1 Physical | Network Access | Bits | Copper, fiber, Wi-Fi | Hubs, cables, repeaters |

## Key Numbers to Memorize
| Topic | Value |
|---|---|
| Administrative Distance | Connected 0, Static 1, eBGP 20, EIGRP 90, OSPF 110, RIP 120 |
| OSPF timers | Hello 10 / Dead 40 (broadcast, P2P) |
| OSPF multicast | `224.0.0.5` all routers, `224.0.0.6` DR/BDR |
| STP timers | Hello 2 / Forward delay 15 / Max age 20 |
| STP default priority | 32768 (+ VLAN ID) |
| STP port cost | 10M = 100, 100M = 19, 1G = 4, 10G = 2 |
| HSRP | Priority 100 default, Hello 3 / Hold 10 |
| CDP / LLDP timers | 60 / 180 and 30 / 120 |
| ARP / MAC aging | MAC table 300 s default |
| DHCP DORA | Discover, Offer, Request, Ack (UDP 67/68) |
| Private IPv4 | `10/8`, `172.16/12`, `192.168/16` |
| APIPA | `169.254.0.0/16` |
| IPv6 link-local / ULA / GUA | `fe80::/10`, `fc00::/7`, `2000::/3` |
| Ethernet MTU | 1500 bytes (payload), minimum frame 64 bytes |

## Topic Map
| I need to... | Go to |
|---|---|
| Configure a new device | [IOS Basics](./ios-basics.md) |
| Calculate a subnet or plan VLSM | [IPv4 & Subnetting](./ipv4-subnetting.md) |
| Work with IPv6 addresses or routes | [IPv6](./ipv6.md) |
| Build VLANs, trunks, STP, EtherChannel | [VLANs & Switching](./vlan-trunking.md) |
| Configure static routes, OSPF, HSRP | [Routing](./routing.md) |
| Filter traffic | [ACLs](./acl.md) |
| Share one public IP | [NAT / PAT](./nat.md) |
| Hand out addresses | [DHCP](./dhcp.md) |
| Harden switches and device access | [Network Security](./network-security.md) |
| Set up NTP, syslog, SNMP, backups | [Network Services](./network-services.md) |
| Fix something that is broken | [Troubleshooting](./troubleshooting-commands.md) |
| Look up a port or protocol number | [Ports & Protocols](./common-ports-protocols.md) |

## Top 25 Show Commands
```ios
show running-config
show ip interface brief
show interfaces <id>
show interfaces status
show interfaces trunk
show vlan brief
show mac address-table
show spanning-tree
show etherchannel summary
show ip route
show ipv6 route
show ip protocols
show ip ospf neighbor
show ip arp
show ipv6 neighbors
show ip dhcp binding
show ip nat translations
show access-lists
show port-security
show cdp neighbors detail
show ntp status
show logging
show version
show ip ssh
show standby brief
```

## Wireless Basics
| Topic | Facts |
|---|---|
| 2.4 GHz | Longer range, more interference; non-overlapping channels `1`, `6`, `11` |
| 5 GHz | Shorter range, more channels, less interference |
| 6 GHz | Wi-Fi 6E / 7, most spectrum, shortest range |
| SSID / BSSID | Network name / AP radio MAC |
| Architectures | Autonomous AP, Lightweight AP + WLC (CAPWAP), Cloud-managed |
| Security | WPA2-AES minimum, WPA3 preferred, 802.1X for enterprise |

## Pre-Change Checklist
1. Save a copy of the running config (`copy running-config flash:backup`).
2. Know the rollback command (`reload in 10` is a safety net on remote changes, cancel with `reload cancel`).
3. Make one change at a time and verify with `show` commands.
4. Save with `copy running-config startup-config` once confirmed.
5. Document what changed and why.
