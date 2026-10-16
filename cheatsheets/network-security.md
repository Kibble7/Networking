# Network Security Cheat Sheet

## Layer 2 Defense Matrix
| Attack Vector | Mitigation Feature | Key Configuration Command |
|---|---|---|
| MAC Table Flooding / Rogue Host | Port Security | `switchport port-security` + `mac-address sticky` |
| Rogue DHCP Server / DHCP Starvation | DHCP Snooping | `ip dhcp snooping` + `ip dhcp snooping trust` (uplinks) |
| ARP Spoofing / Poisoning | Dynamic ARP Inspection (DAI) | `ip arp inspection vlan <id>` + `ip arp inspection trust` (uplinks) |
| IP Spoofing | IP Source Guard | `ip verify source` (needs DHCP snooping) |
| Rogue Switch / BPDU Injection | BPDU Guard | `spanning-tree bpduguard enable` |
| STP Root Hijack | Root Guard | `spanning-tree guard root` |
| VLAN Hopping (DTP / Double Tag) | DTP Off + Native VLAN | `switchport nonegotiate` + `switchport trunk native vlan 999` |
| Cleartext Credential Sniffing | SSHv2 + Secret Hashing | `transport input ssh` + `enable secret <password>` |
| Unused Port Abuse | Shutdown + Parking VLAN | `shutdown` + `switchport access vlan 999` |

## Port Security
```ios
interface g0/1
 switchport mode access                          ! Must be a static access (or trunk) port
 switchport port-security                        ! Enable
 switchport port-security maximum 2              ! Max MAC addresses (default 1)
 switchport port-security mac-address sticky     ! Learn MACs dynamically and save to running-config
 switchport port-security mac-address 0011.2233.4455   ! Or hard-code one
 switchport port-security violation restrict     ! protect | restrict | shutdown (default)
 switchport port-security aging time 10          ! Minutes
 switchport port-security aging type inactivity
```

| Violation Mode | Drops violating frames | Logs / counts | Port shuts down |
|---|---|---|---|
| `protect` | Yes | No | No |
| `restrict` | Yes | Yes (syslog + counter) | No |
| `shutdown` (default) | Yes | Yes | Yes (`err-disabled`) |

Recover an `err-disabled` port: fix the cause, then `shutdown` → `no shutdown` on the interface.
Auto-recover: `errdisable recovery cause psecure-violation` and `errdisable recovery interval 300`.

## DHCP Snooping
```ios
ip dhcp snooping
ip dhcp snooping vlan 10,20
interface g0/24                                  ! Uplink / trunk toward the real DHCP server
 ip dhcp snooping trust
interface g0/1                                   ! Access port (untrusted by default)
 ip dhcp snooping limit rate 15                  ! Max DHCP packets/sec before err-disabled
```
The snooping binding table (`show ip dhcp snooping binding`) is what DAI and IP Source Guard use.

## Dynamic ARP Inspection (DAI)
```ios
ip arp inspection vlan 10,20
ip arp inspection validate src-mac dst-mac ip
interface g0/24
 ip arp inspection trust                         ! Trust uplinks only
```

## Harden Unused & Access Ports
```ios
interface range g0/10 - 20
 switchport mode access
 switchport access vlan 999                      ! Blackhole VLAN
 switchport nonegotiate
 shutdown
```
Also consider `no cdp enable` / `no lldp transmit` on ports facing users or the Internet.

## Secure Device Access
```ios
service password-encryption                      ! Weak (type 7) obfuscation for line passwords
enable secret <password>                         ! Hashed enable password (never use "enable password")
username admin privilege 15 secret <password>
security passwords min-length 10
login block-for 120 attempts 3 within 60
login on-failure log
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
 access-class 5 in                               ! Limit who may connect (see acl.md)
no ip http server                                ! Turn off unencrypted web management
ip http secure-server                            ! Use HTTPS if web management is needed
```

## AAA (Authentication, Authorization, Accounting)
```ios
aaa new-model
aaa authentication login default local           ! Local users only
aaa authentication login default group radius local   ! RADIUS first, local as fallback
aaa authorization exec default group radius local
aaa accounting exec default start-stop group radius

radius server RAD1
 address ipv4 10.10.99.50 auth-port 1812 acct-port 1813
 key <shared_secret>
```
| Protocol | Transport | Notes |
|---|---|---|
| RADIUS | UDP `1812` (auth) / `1813` (accounting) | Open standard; encrypts only the password; combines authN and authZ |
| TACACS+ | TCP `49` | Cisco; encrypts the whole payload; separates authN, authZ and accounting |

## Wireless & Other Security Basics
| Topic | Guidance |
|---|---|
| Wi-Fi encryption | Use **WPA2 (AES-CCMP)** or **WPA3 (SAE)**; avoid WEP and TKIP |
| Enterprise Wi-Fi | WPA2/WPA3-Enterprise with **802.1X** and a RADIUS server |
| Management plane | Out-of-band management VLAN, SSH only, ACL-limited |
| Native VLAN | Not VLAN 1; unused VLAN (e.g. `999`) on both ends of every trunk |
| Logging | Send logs to a central syslog server and keep clocks synced with NTP |

## Verify
```ios
show port-security
show port-security interface g0/1
show port-security address
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip ssh
show login failures
show users
show spanning-tree inconsistentports
```
