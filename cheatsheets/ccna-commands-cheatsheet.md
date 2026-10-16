# CCNA Command Cheat Sheet (One File)

Cisco IOS / IOS-XE. Replace `<values>` with your own. `R(config)#` = global config, `R(config-if)#` = interface, and so on.

---

## 1. CLI Basics
```ios
enable / disable / configure terminal / end / exit    ! Change modes (Ctrl+Z = end)
?  |  <Tab>  |  do show run                          ! Help, complete, run show from config mode
no <command>  |  default interface g0/1              ! Remove a command / reset an interface
interface range g0/1 - 4                              ! Configure several ports
show run | include <x>   | section <x>   | begin <x>  ! Output filters
copy running-config startup-config                    ! Save (or: write memory)
erase startup-config  |  delete flash:vlan.dat  |  reload      ! Factory reset a switch
show version | show inventory | show flash: | show clock | show users | show history
```

## 2. Initial Device Config & SSH
```ios
hostname R1
no ip domain-lookup
enable secret <pw>
service password-encryption
security passwords min-length 10
banner motd #Authorized access only#
username admin privilege 15 secret <pw>
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
login block-for 120 attempts 3 within 60
line console 0
 password <pw>
 login
 logging synchronous
 exec-timeout 5 0
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
interface g0/0/0
 description LAN
 ip address 192.168.1.1 255.255.255.0
 no shutdown
! Switch management
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.99.1
```

## 3. VLANs, Trunks & Inter-VLAN Routing
```ios
vlan 10
 name SALES
interface g0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 150                            ! Optional IP phone VLAN
 spanning-tree portfast
 spanning-tree bpduguard enable
interface g0/24
 switchport mode trunk
 switchport trunk encapsulation dot1q                 ! Only on switches that support ISL
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99               ! add / remove / except keywords exist
 switchport nonegotiate                               ! Turn DTP off

! Router-on-a-stick
interface g0/0/1
 no shutdown
interface g0/0/1.10
 encapsulation dot1q 10                               ! "native" keyword for native VLAN subif
 ip address 192.168.10.1 255.255.255.0

! Layer 3 switch
ip routing
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface g1/0/1
 no switchport                                        ! Routed port
 ip address 10.0.0.1 255.255.255.252
```

## 4. STP & EtherChannel
```ios
spanning-tree mode rapid-pvst
spanning-tree vlan 1,10 root primary                  ! or: root secondary / priority 4096
spanning-tree portfast default
spanning-tree portfast bpduguard default
spanning-tree guard root                              ! Interface
spanning-tree vlan 10 cost 10                         ! Interface
spanning-tree vlan 10 port-priority 64                ! Interface

interface range g0/1 - 2
 channel-group 1 mode active                          ! LACP: active/passive  PAgP: desirable/auto  Static: on
interface port-channel 1
 switchport mode trunk
port-channel load-balance src-dst-ip
```

## 5. IPv4 Routing
```ios
ip route <net> <mask> <next-hop>                      ! Static route
ip route <net> <mask> <exit-int> <next-hop>           ! Fully specified
ip route 0.0.0.0 0.0.0.0 <next-hop>                   ! Default route
ip route <net> <mask> <next-hop> 5                    ! Floating static (AD 5)

router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 passive-interface g0/0/0                             ! or: passive-interface default + no passive-interface g0/0/1
 default-information originate
 auto-cost reference-bandwidth 10000
interface g0/0/1
 ip ospf 1 area 0                                     ! Per-interface alternative to network
 ip ospf cost 10
 ip ospf priority 100                                 ! 0 = never DR
 ip ospf network point-to-point
 ip ospf hello-interval 10
 ip ospf dead-interval 40
clear ip ospf process

router eigrp 100                                      ! Legacy / optional
 eigrp router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255
 no auto-summary

! HSRP
interface g0/0/0
 standby version 2
 standby 1 ip 192.168.1.254
 standby 1 priority 110
 standby 1 preempt
```

## 6. IPv6
```ios
ipv6 unicast-routing
interface g0/0/0
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 address fe80::1 link-local
 ipv6 address 2001:db8:acad:2::/64 eui-64
 ipv6 enable                                          ! Link-local only
 ipv6 address autoconfig                              ! SLAAC client
ipv6 route 2001:db8:acad:3::/64 <next-hop>
ipv6 route 2001:db8:acad:3::/64 g0/0/1 fe80::2        ! Fully specified (LLA next hop)
ipv6 route ::/0 <next-hop>
ipv6 router ospf 1                                    ! OSPFv3
 router-id 1.1.1.1
interface g0/0/1
 ipv6 ospf 1 area 0
```

## 7. DHCP
```ios
! DHCPv4 server
ip dhcp excluded-address 10.10.10.1 10.10.10.9
ip dhcp pool LAN_POOL
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.99.50
 domain-name lab.local
 lease 7
interface g0/0/0.20
 ip helper-address 10.10.99.50                        ! Relay
interface g0/0/1
 ip address dhcp                                      ! Router as client

! DHCPv6
ipv6 dhcp pool STATELESS_POOL
 dns-server 2001:db8:acad:99::5
 domain-name lab.local
interface g0/0/0
 ipv6 nd other-config-flag                            ! Stateless (SLAAC + DHCPv6 info)
 ipv6 dhcp server STATELESS_POOL
ipv6 dhcp pool STATEFUL_POOL
 address prefix 2001:db8:acad:2::/64 lifetime infinite infinite
 dns-server 2001:db8:acad:99::5
interface g0/0/1
 ipv6 nd managed-config-flag                          ! Stateful
 ipv6 nd prefix default no-autoconfig
 ipv6 dhcp server STATEFUL_POOL
interface g0/0/2
 ipv6 dhcp relay destination 2001:db8:acad:99::5
```

## 8. ACLs
```ios
! Standard (close to destination)
access-list 10 permit 192.168.10.0 0.0.0.255
ip access-list standard ADMIN_ONLY
 10 permit host 192.168.99.10
 20 deny any log

! Extended (close to source)
ip access-list extended WEB_ONLY
 10 remark Allow web from Sales
 20 permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.10 eq 80
 30 permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.10 eq 443
 40 permit icmp any any echo
 50 deny ip any any log
interface g0/0/0
 ip access-group WEB_ONLY in                          ! in | out
line vty 0 15
 access-class 10 in                                   ! Restrict management access

no 30                                                 ! Remove a line in named ACL
ip access-list resequence WEB_ONLY 10 10

! IPv6
ipv6 access-list BLOCK_TELNET_V6
 deny tcp any any eq 23
 permit ipv6 any any
interface g0/0/0
 ipv6 traffic-filter BLOCK_TELNET_V6 in
```

## 9. NAT / PAT
```ios
interface g0/0/0
 ip nat inside
interface g0/0/1
 ip nat outside
ip nat inside source static <inside_local> <inside_global>                  ! Static NAT
ip nat pool POOL1 <start_ip> <end_ip> netmask <mask>                        ! Dynamic NAT
ip nat inside source list 1 pool POOL1
access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface g0/0/1 overload                       ! PAT
ip nat inside source static tcp <inside_local> 80 <inside_global> 8080      ! Port forward
```

## 10. Layer 2 Security & Device Hardening
```ios
! Port security
interface g0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict          ! protect | restrict | shutdown (default)
errdisable recovery cause psecure-violation

! DHCP snooping + DAI
ip dhcp snooping
ip dhcp snooping vlan 10,20
ip arp inspection vlan 10,20
interface g0/24
 ip dhcp snooping trust
 ip arp inspection trust
interface g0/1
 ip dhcp snooping limit rate 15

! Unused ports
interface range g0/10 - 20
 switchport mode access
 switchport access vlan 999
 shutdown

! AAA
aaa new-model
aaa authentication login default group radius local
radius server RAD1
 address ipv4 10.10.99.50 auth-port 1812 acct-port 1813
 key <secret>
```

## 11. Network Services

```ios
ntp server 10.10.99.1                                 ! Client
ntp master 3                                          ! Lab authoritative source
clock set 10:30:00 16 October 2026
logging host 10.10.99.60
logging trap informational
service timestamps log datetime msec
snmp-server community <string> RO                     ! v2c (legacy)
snmp-server group ADMINS v3 priv
snmp-server user snmpuser ADMINS v3 auth sha <pw> priv aes 128 <pw>
snmp-server host 10.10.99.60 version 3 priv snmpuser
cdp run  |  no cdp enable                             ! Global | per interface
lldp run                                              ! Interface: lldp transmit / lldp receive
copy running-config tftp://10.10.99.70/R1-confg       ! Backup
copy tftp://10.10.99.70/R1-confg running-config       ! Restore (merges)
```

## 12. Verify & Troubleshoot (Show Commands)
```ios
! Interfaces & Layer 2
show ip interface brief | show interfaces <id> | show interfaces status
show interfaces status err-disabled | show interfaces counters errors
show vlan brief | show interfaces trunk | show interfaces <id> switchport
show mac address-table [dynamic] | show spanning-tree [vlan 10 | summary | root]
show etherchannel summary

! Layer 3
show ip route [<ip> | static | ospf] | show ipv6 route
show ip arp | show ipv6 neighbors | show ipv6 interface brief
show ip protocols | show ip ospf neighbor | show ip ospf interface brief
show ip eigrp neighbors | show standby brief

! Services & security
show ip dhcp binding | show ip dhcp pool | show ipv6 dhcp binding
show ip nat translations | show ip nat statistics
show access-lists | show ip interface <id> | include access list
show port-security [interface <id>] | show ip dhcp snooping [binding] | show ip arp inspection
show ntp status | show logging | show cdp neighbors detail | show lldp neighbors
show ip ssh | show snmp

! Tools
ping <ip> | ping <ip> source <int> repeat 100 | traceroute <ip> | ping ipv6 <addr>
terminal monitor | debug <feature> | undebug all
```
```text
Windows : ipconfig /all | ipconfig /release /renew | ping | tracert | nslookup | arp -a | netstat -ano
Linux   : ip addr | ip route | ip neigh | ss -tulpn | ping -c 4 | traceroute | dig
```

---

## Key Numbers

### Subnetting
| CIDR | Mask | Wildcard | Block | Usable Hosts |
|---|---|---|---|---|
| `/24` | `255.255.255.0` | `0.0.0.255` | 256 | 254 |
| `/25` | `255.255.255.128` | `0.0.0.127` | 128 | 126 |
| `/26` | `255.255.255.192` | `0.0.0.63` | 64 | 62 |
| `/27` | `255.255.255.224` | `0.0.0.31` | 32 | 30 |
| `/28` | `255.255.255.240` | `0.0.0.15` | 16 | 14 |
| `/29` | `255.255.255.248` | `0.0.0.7` | 8 | 6 |
| `/30` | `255.255.255.252` | `0.0.0.3` | 4 | 2 |

Block size = 256 − mask octet. Network = largest multiple of block ≤ IP. Broadcast = next network − 1. Hosts = 2^h − 2.
Private: `10/8`, `172.16/12`, `192.168/16`. APIPA `169.254/16`. IPv6: GUA `2000::/3`, LLA `fe80::/10`, ULA `fc00::/7`, multicast `ff00::/8`.

### Administrative Distance & Timers
| Item | Value |
|---|---|
| AD | Connected 0, Static 1, eBGP 20, EIGRP 90, OSPF 110, RIP 120, EIGRP ext 170, iBGP 200 |
| Path choice | Longest prefix → lowest AD → lowest metric |
| OSPF | Hello 10 / Dead 40; `224.0.0.5` all routers, `224.0.0.6` DR/BDR; cost = ref BW ÷ BW (ref 100 Mbps) |
| STP | Hello 2 / Forward 15 / Max age 20; priority 32768 + VLAN; cost 100M=19, 1G=4, 10G=2 |
| HSRP | Priority 100, Hello 3 / Hold 10 |
| CDP / LLDP | 60 / 180 and 30 / 120 |

### EtherChannel & Port Security
| EtherChannel | Forms bundle with |
|---|---|
| LACP (802.3ad) | `active`+`active` or `active`+`passive` |
| PAgP (Cisco) | `desirable`+`desirable` or `desirable`+`auto` |
| Static | `on`+`on` only |

| Violation | Drops | Logs | Err-disables |
|---|---|---|---|
| `protect` | Yes | No | No |
| `restrict` | Yes | Yes | No |
| `shutdown` | Yes | Yes | Yes |

### Ports
| Port | Protocol | Port | Protocol |
|---|---|---|---|
| `20/21` TCP | FTP | `123` UDP | NTP |
| `22` TCP | SSH / SCP / SFTP | `161/162` UDP | SNMP |
| `23` TCP | Telnet | `179` TCP | BGP |
| `25` TCP | SMTP | `389` TCP/UDP | LDAP |
| `53` UDP/TCP | DNS | `443` TCP | HTTPS |
| `67/68` UDP | DHCPv4 | `514` UDP | Syslog |
| `69` UDP | TFTP | `546/547` UDP | DHCPv6 |
| `80` TCP | HTTP | `1812/1813` UDP | RADIUS |
| `49` TCP | TACACS+ | `3389` TCP | RDP |

### Syslog Levels
`0` Emergency · `1` Alert · `2` Critical · `3` Error · `4` Warning · `5` Notification · `6` Informational · `7` Debugging
