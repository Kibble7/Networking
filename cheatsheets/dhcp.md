# DHCPv4, DHCPv6 & SLAAC Cheat Sheet

## DHCPv4 DORA Packet Flow
| Step | Message | Source IP : Port | Dest IP : Port | Frame Type |
|---|---|---|---|---|
| 1 | `DHCPDISCOVER` | `0.0.0.0 : 68` | `255.255.255.255 : 67` | Broadcast |
| 2 | `DHCPOFFER` | `<Server_IP> : 67` | `255.255.255.255 : 68` (or Unicast) | Broadcast/Unicast |
| 3 | `DHCPREQUEST` | `0.0.0.0 : 68` | `255.255.255.255 : 67` | Broadcast |
| 4 | `DHCPACK` | `<Server_IP> : 67` | `255.255.255.255 : 68` (or Unicast) | Broadcast/Unicast |

Lease renewal: the client tries to renew at **50%** of the lease (T1) and rebinds with any server at **87.5%** (T2).

## Essential CLI Configuration
```ios
! IOS DHCPv4 Server
ip dhcp excluded-address 10.10.10.1 10.10.10.9       ! Reserve static addresses (gateway, printers)
ip dhcp pool LAN_POOL
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 10.10.99.50
 domain-name lab.local
 lease 7                                              ! Days (or: lease 0 8 0 = 8 hours)
 ! option 150 ip 10.10.99.60                          ! TFTP server for IP phones

! DHCPv4 Relay Agent (on Client Gateway Interface)
interface G0/0/0.20
 ip helper-address 10.10.99.50

! Router interface as a DHCP client
interface g0/0/1
 ip address dhcp
 no shutdown
```
`service dhcp` is on by default; `no service dhcp` disables the server and relay.

`ip helper-address` relays UDP broadcasts for: DHCP (`67/68`), TFTP (`69`), DNS (`53`), TIME (`37`), NetBIOS (`137/138`) and TACACS (`49`).

## Verify DHCPv4
```ios
show ip dhcp binding                     ! Active leases (IP, MAC, expiry)
show ip dhcp pool                        ! Pool size, leased addresses
show ip dhcp server statistics           ! DISCOVER/OFFER/REQUEST/ACK counts
show ip dhcp conflict                    ! Addresses withdrawn due to duplicate IP
clear ip dhcp binding *                  ! Release all leases
show ip interface g0/0/0 | include Helper
```
```text
Windows : ipconfig /release   then   ipconfig /renew   |   ipconfig /all
Linux   : sudo dhclient -r && sudo dhclient   |   nmcli device show
```

## SLAAC vs. DHCPv6 Summary
| Mode | RA Flags | Address from | DNS from | Router command |
|---|---|---|---|---|
| **SLAAC** | `M=0, O=0` | SLAAC (prefix in RA + EUI-64/random) | Not provided | Default behavior |
| **Stateless DHCPv6** | `M=0, O=1` | SLAAC | DHCPv6 server | `ipv6 nd other-config-flag` |
| **Stateful DHCPv6** | `M=1` | DHCPv6 server | DHCPv6 server | `ipv6 nd managed-config-flag` |

Message flow: client sends **RS** to `ff02::2`, router answers with **RA** to `ff02::1`; DHCPv6 clients then use UDP `546` (client) and `547` (server/relay).
DHCPv6 messages: `SOLICIT` → `ADVERTISE` → `REQUEST` → `REPLY` (stateless clients use `INFORMATION-REQUEST` → `REPLY`).

## DHCPv6 CLI Configuration
```ios
ipv6 unicast-routing

! Stateless DHCPv6 (SLAAC address + DHCPv6 for DNS)
ipv6 dhcp pool STATELESS_POOL
 dns-server 2001:db8:acad:99::5
 domain-name lab.local
interface g0/0/0
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server STATELESS_POOL

! Stateful DHCPv6 (addresses + DNS from the server)
ipv6 dhcp pool STATEFUL_POOL
 address prefix 2001:db8:acad:2::/64 lifetime infinite infinite
 dns-server 2001:db8:acad:99::5
 domain-name lab.local
interface g0/0/1
 ipv6 address 2001:db8:acad:2::1/64
 ipv6 nd managed-config-flag
 ipv6 nd prefix default no-autoconfig                ! Clear the A flag so hosts do not use SLAAC
 ipv6 dhcp server STATEFUL_POOL

! DHCPv6 relay (client-facing interface, server on another network)
interface g0/0/2
 ipv6 dhcp relay destination 2001:db8:acad:99::5

! Router interface as a client
interface g0/0/3
 ipv6 address autoconfig                             ! SLAAC
 ! ipv6 address dhcp                                 ! Stateful DHCPv6
```

## Verify DHCPv6
```ios
show ipv6 interface g0/0/0               ! Look for the RA flags (managed / other-config)
show ipv6 dhcp pool
show ipv6 dhcp binding
show ipv6 dhcp interface
show ipv6 neighbors
```
