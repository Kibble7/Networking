# Common Ports & Protocols Reference

## Well-Known Ports
| Port | Transport | Protocol | Purpose |
|---|---|---|---|
| `20 / 21` | TCP | FTP | File Transfer Protocol (Data / Control) |
| `22` | TCP | SSH / SCP / SFTP | Encrypted remote CLI management and file transfer |
| `23` | TCP | Telnet | Unencrypted legacy remote CLI (Disable in production) |
| `25` | TCP | SMTP | Simple Mail Transfer Protocol (server to server) |
| `49` | TCP | TACACS+ | Cisco AAA (encrypts the whole payload) |
| `53` | UDP / TCP | DNS | Domain Name System queries (UDP) and zone transfers (TCP) |
| `67 / 68` | UDP | DHCPv4 | Dynamic Host Configuration Protocol (Server `67` / Client `68`) |
| `69` | UDP | TFTP | Trivial File Transfer Protocol (Config/IOS backups) |
| `80` | TCP | HTTP | Unencrypted web traffic |
| `88` | TCP / UDP | Kerberos | Windows / AD authentication |
| `110` | TCP | POP3 | Email retrieval (cleartext) |
| `123` | UDP | NTP | Network Time Protocol clock synchronization |
| `137 - 139` | UDP / TCP | NetBIOS | Legacy Windows name and session services |
| `143` | TCP | IMAP | Email retrieval (cleartext) |
| `161 / 162` | UDP | SNMP | Simple Network Management Protocol (Polling / Traps) |
| `179` | TCP | BGP | Border Gateway Protocol |
| `389` | TCP / UDP | LDAP | Directory queries (Active Directory) |
| `443` | TCP | HTTPS | TLS-encrypted web traffic |
| `445` | TCP | SMB | Windows file sharing |
| `465 / 587` | TCP | SMTPS / Submission | Authenticated, encrypted mail submission |
| `500` | UDP | IKE (ISAKMP) | IPsec VPN key exchange |
| `514` | UDP | Syslog | Centralized event logging |
| `520` | UDP | RIP | Routing Information Protocol |
| `521` | UDP | RIPng | RIP for IPv6 |
| `546 / 547` | UDP | DHCPv6 | DHCPv6 Client (`546`) / Server (`547`) |
| `636` | TCP | LDAPS | LDAP over TLS |
| `993 / 995` | TCP | IMAPS / POP3S | Encrypted email retrieval |
| `1433` | TCP | MS SQL | Microsoft SQL Server |
| `1645 / 1646` or `1812 / 1813` | UDP | RADIUS | Authentication / Accounting (`1812/1813` is current) |
| `1701` | UDP | L2TP | Layer 2 Tunneling Protocol |
| `1723` | TCP | PPTP | Legacy VPN |
| `3306` | TCP | MySQL | MySQL / MariaDB |
| `3389` | TCP | RDP | Remote Desktop Protocol |
| `4500` | UDP | IPsec NAT-T | IPsec through NAT |
| `5060 / 5061` | TCP / UDP | SIP / SIP-TLS | VoIP signaling |
| `5432` | TCP | PostgreSQL | PostgreSQL database |

## IP Protocol Numbers (Layer 3 payload)
| Number | Protocol | Notes |
|---|---|---|
| `1` | ICMP | ping, traceroute, errors |
| `6` | TCP | Reliable, connection-oriented |
| `17` | UDP | Connectionless |
| `47` | GRE | Tunneling |
| `50` | ESP | IPsec encrypted payload |
| `51` | AH | IPsec authentication header |
| `58` | ICMPv6 | Neighbor Discovery, errors, ping |
| `88` | EIGRP | Runs directly over IP |
| `89` | OSPF | Runs directly over IP |
| `112` | VRRP | First-hop redundancy |

## Reserved Multicast Addresses
| Address | Used by |
|---|---|
| `224.0.0.1` / `224.0.0.2` | All hosts / all routers |
| `224.0.0.5` / `224.0.0.6` | OSPF routers / OSPF DR and BDR |
| `224.0.0.9` | RIPv2 |
| `224.0.0.10` | EIGRP |
| `224.0.0.18` | VRRP |
| `224.0.0.102` | HSRP version 2 |
| `ff02::1` / `ff02::2` | IPv6 all nodes / all routers |
| `ff02::5` / `ff02::6` | OSPFv3 routers / DR and BDR |
| `ff02::1:2` | DHCPv6 relay agents and servers |

## ICMP Message Types
| Type | Name | Used by |
|---|---|---|
| `0` | Echo Reply | `ping` response |
| `3` | Destination Unreachable | Codes: net, host, protocol, port unreachable, fragmentation needed |
| `8` | Echo Request | `ping` |
| `11` | Time Exceeded | `traceroute` (TTL expired) |
| ICMPv6 `133 / 134` | Router Solicitation / Advertisement | SLAAC |
| ICMPv6 `135 / 136` | Neighbor Solicitation / Advertisement | IPv6 ARP replacement |

## Layer 2 Identifiers
| Item | Value |
|---|---|
| EtherType IPv4 / ARP / IPv6 | `0x0800` / `0x0806` / `0x86DD` |
| EtherType 802.1Q VLAN tag | `0x8100` |
| EtherType LLDP | `0x88CC` |
| STP BPDU multicast MAC | `0180.c200.0000` |
| CDP multicast MAC | `0100.0ccc.cccc` |
| LLDP multicast MAC | `0180.c200.000e` |
| Broadcast MAC | `ffff.ffff.ffff` |

## TCP vs UDP
| | TCP | UDP |
|---|---|---|
| Connection | 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) | None |
| Reliability | Acknowledgments, retransmission, ordering | Best effort |
| Common use | Web, email, SSH, file transfer | DNS queries, DHCP, VoIP, streaming, syslog |
| Flags to know | `SYN`, `ACK`, `FIN`, `RST`, `PSH` | n/a |
| Port ranges | Well-known `0-1023`, Registered `1024-49151`, Dynamic `49152-65535` | same |
