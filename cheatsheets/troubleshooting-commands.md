# Troubleshooting Command Reference

Organized by operational purpose: **Command → Purpose → What to Look For**.

## Troubleshooting Method (bottom-up)
1. **Physical (L1):** cable, link light, speed/duplex, power.
2. **Data link (L2):** VLAN, trunk, STP, MAC table, port security.
3. **Network (L3):** IP address, mask, gateway, routing, ARP, ACL, NAT.
4. **Transport / Application (L4+):** port reachable? firewall? DNS? service running?

Quick isolation order from a host: `ping 127.0.0.1` (stack) → own IP → default gateway → remote IP → remote name (DNS).

## 1. Physical & Interface Layer
| Command | Purpose | What to Look For |
|---|---|---|
| `show ip interface brief` | Quick summary of all interface IPv4 states | Both `Status` (L1) and `Protocol` (L2) must be `up/up`; `administratively down` means `shutdown` is applied. |
| `show interfaces <id>` | Deep L1/L2 interface diagnostics | Check duplex/speed, `CRC` errors, `runts/giants`, and input/output queue drops. |
| `show interfaces status` | Switch port state, VLAN, and speed summary | Look for `err-disabled` status caused by Port Security or BPDU Guard. |
| `show interfaces status err-disabled` | Identify exact cause of `err-disabled` state | Look under `Reason` column (`psecure-violation`, `bpduguard`, `channel-misconfig`). |
| `show errdisable recovery` | Auto-recovery settings for err-disabled ports | Which causes recover automatically and the interval. |
| `show interfaces counters errors` | Per-port error counters (switch) | Rising `FCS-Err`, `Align-Err`, `Runts` indicate a bad cable, NIC or duplex mismatch. |
| `show controllers <id>` | Hardware-level interface detail | Cable type, clocking (serial), transceiver state. |
| `show interfaces transceiver` | SFP optical levels (where supported) | Tx/Rx power within thresholds. |

## 2. VLANs, Trunks, STP & EtherChannel
| Command | Purpose | What to Look For |
|---|---|---|
| `show vlan brief` | Inspect local switch VLAN database | Confirm VLAN ID exists, is `active`, and access ports are assigned to the right VLAN. |
| `show interfaces trunk` | Verify 802.1Q trunk links | Confirm `802.1q` encapsulation, matching `Native vlan`, and VLANs listed under `forwarding state and not pruned`. |
| `show interfaces <id> switchport` | Detailed L2 port mode and Voice VLAN check | Verify `Operational Mode` (`static access` vs `trunk`) and `Voice VLAN` assignment. |
| `show mac address-table dynamic` | Inspect learned Layer 2 MAC addresses | Confirm host MAC is learned on the expected port and VLAN ID. |
| `show mac address-table address <mac>` | Find which port a MAC lives on | Port and VLAN for that host. |
| `show spanning-tree vlan <id>` | Verify STP Root Bridge and port roles/states | Identify `Root ID` vs `Bridge ID`, and confirm ports are `FWD` or expected `BLK`. |
| `show spanning-tree summary` | STP mode and per-VLAN counts | Mode (`rapid-pvst`), blocking/forwarding counts, PortFast/BPDU Guard defaults. |
| `show etherchannel summary` | Verify LACP/PAgP link aggregation status | Look for `(SU)` on the Port-channel and `(P)` on all physical members; `(I)` or `(s)` indicates misconfiguration. |

## 3. IPv4/IPv6 Addressing, ARP & Routing
| Command | Purpose | What to Look For |
|---|---|---|
| `show ip arp` / `show ipv6 neighbors` | Verify Layer 3 to Layer 2 MAC resolution | Confirm the default gateway IP resolves to a valid MAC (`ARPA` / `REACH`), not `Incomplete`. |
| `show ip route` / `show ipv6 route` | Inspect active Routing Information Base (RIB) | Verify `S*` default route, `C` connected subnets, and expected `[AD/Metric]` next-hops. |
| `show ip route <dest_ip>` | Check longest-prefix match for a specific IP | Shows the exact route entry and exit interface the router will use for that destination. |
| `show ip protocols` | Routing protocol summary | Router ID, advertised networks, passive interfaces, AD. |
| `show ip ospf neighbor` | OSPF adjacency table | State should be `FULL` (or `2WAY` with DROTHER on broadcast segments). |
| `show ip ospf interface brief` | OSPF-enabled interfaces | Area, cost, state (`DR`, `BDR`, `P2P`), neighbor count. |
| `show ip eigrp neighbors` | EIGRP adjacencies | Neighbor listed with sane `Uptime` and `SRTT`. |
| `show standby brief` | HSRP state | One `Active`, one `Standby`, expected virtual IP. |
| `show running-config \| section <keyword>` | Pull one part of the config | Quick check of OSPF, ACL, NAT, VTY configuration. |

## 4. Network Services (DHCP, NAT, NTP, Discovery)
| Command | Purpose | What to Look For |
|---|---|---|
| `show ip dhcp binding` | List active DHCPv4 client leases | Verify client MAC address received an IP from the intended subnet pool. |
| `show ip dhcp pool` | Check DHCPv4 pool capacity | Confirm `Leased addresses` has not exhausted total pool range. |
| `show ip nat translations` | View active NAT/PAT mappings | Verify `Inside local` private IPs translate to `Inside global` public IPs with L4 ports. |
| `show ip nat statistics` | Check NAT interface bindings and hit counters | Verify `Outside interfaces`, `Inside interfaces`, and rising `Hits` count. |
| `show ntp status` | Verify system clock synchronization | Look for `Clock is synchronized` and valid `stratum` level (`< 16`). |
| `show cdp neighbors detail` | Discover directly connected Cisco devices | Inspect neighbor hostname, management IPv4 address, remote port ID, and Native VLAN. |
| `show lldp neighbors detail` | Discover directly connected non-Cisco/Cisco devices | Neighbor name, port, management address. |
| `show logging` | Review device log buffer | Link flaps, STP changes, security violations, OSPF adjacency changes. |

## 5. Security (ACLs, Port Security, DHCP Snooping, SSH)
| Command | Purpose | What to Look For |
|---|---|---|
| `show access-lists` | View all ACL rules and match counters | Check `(X matches)` on `permit`/`deny` lines to confirm which rule is catching traffic. |
| `show ip interface <id>` | Verify ACL direction and DHCP relay | Confirm `Outgoing` / `Inbound access list` names and `Helper addresses` configured on the interface. |
| `show port-security interface <id>` | Inspect port security status and violations | Check `Port Status` (`Secure-up` vs `Secure-shutdown`) and `Security Violation Count`. |
| `show ip dhcp snooping` | Verify DHCP snooping trusted ports | Ensure uplink trunks show `Trusted: yes` and access ports show `Trusted: no`. |
| `show ip arp inspection` | Verify DAI status and drops | Forwarded vs dropped ARP packets per VLAN. |
| `show ip ssh` | Verify SSH server version and timeouts | Confirm `SSH Enabled - version 2.0`. |

## 6. Device Health
| Command | Purpose | What to Look For |
|---|---|---|
| `show version` | Platform, IOS, uptime, last reload reason | Unexpected reloads, config register `0x2102`. |
| `show processes cpu sorted` | CPU usage by process | Sustained high CPU (loops, floods, ARP/broadcast storms). |
| `show memory statistics` | Memory use | Large `Free` shrinking over time suggests a leak. |
| `show environment all` | Power, fans, temperature (where supported) | Anything not `OK`. |

## 7. Connectivity Tools on the Device
```ios
ping 10.1.1.1
ping 10.1.1.1 source g0/0/0 repeat 100 size 1500 df-bit    ! Source-based, MTU test
ping                                                      ! Extended ping (guided prompts)
traceroute 10.1.1.1
ping ipv6 2001:db8:acad:1::1
traceroute ipv6 2001:db8:acad:1::1
ssh -l admin 10.1.1.1                                      ! Test SSH to another device
telnet 10.1.1.1 443                                        ! Quick TCP port test
```
Ping result characters: `!` success, `.` timeout, `U` destination unreachable, `N` network unreachable, `M` could not fragment.

### Debugging (use sparingly)
```ios
terminal monitor                 ! Show debug output on a Telnet/SSH session
debug ip ospf adj                ! Example: watch OSPF adjacency
debug ip packet 100              ! Limit with an ACL first!
undebug all                      ! Stop ALL debugging (also: "no debug all")
```
Debugging can overload a production device. Prefer `show` commands, and always use ACL-limited or specific debugs.

## 8. Host-Side Commands

### Windows (Command Prompt / PowerShell)
| Command | Purpose |
|---|---|
| `ipconfig /all` | IP, mask, gateway, DNS, DHCP server, MAC |
| `ipconfig /release` + `ipconfig /renew` | Refresh DHCP lease |
| `ipconfig /flushdns` / `ipconfig /displaydns` | Clear / view DNS cache |
| `ping <host>` / `ping -t <host>` | Connectivity / continuous ping |
| `tracert <host>` / `pathping <host>` | Path and per-hop loss |
| `nslookup <name>` | DNS lookup |
| `arp -a` | ARP cache |
| `netstat -ano` | Connections, listening ports and PIDs |
| `route print` | Local routing table |
| `netsh interface ip show config` | Interface IP configuration |
| `Test-NetConnection <host> -Port 443` | PowerShell TCP port test |
| `Get-NetIPConfiguration` | PowerShell IP summary |

### Linux
| Command | Purpose |
|---|---|
| `ip addr` / `ip -br a` | Interface addresses |
| `ip route` | Routing table |
| `ip neigh` | ARP / neighbor table |
| `ss -tulpn` | Listening ports and owning processes |
| `ping -c 4 <host>` | Connectivity |
| `traceroute <host>` / `mtr <host>` | Path analysis |
| `dig <name>` / `nslookup <name>` | DNS lookup |
| `sudo tcpdump -i eth0 port 53` | Capture packets (example: DNS) |
| `nmap -sS -p 1-1024 <host>` | Port scan (only on systems you own or are authorized to test) |
| `curl -I https://<host>` | HTTP header test |

## 9. Symptom → Likely Cause
| Symptom | Likely cause | First command |
|---|---|---|
| Interface `down/down` | Cable, remote end shut, SFP | `show interfaces <id>` |
| Interface `up/down` | Encapsulation / clocking / keepalive mismatch | `show interfaces <id>` |
| Port `err-disabled` | Port security, BPDU Guard, EtherChannel misconfig | `show interfaces status err-disabled` |
| Host gets `169.254.x.x` | DHCP unreachable (VLAN, relay, pool exhausted) | `show ip dhcp pool` |
| Can ping gateway, not Internet | Missing default route, NAT, upstream issue | `show ip route`, `show ip nat translations` |
| Can ping IP, not name | DNS problem | `nslookup`, `ipconfig /all` |
| Different VLANs cannot talk | No inter-VLAN routing, wrong gateway, ACL | `show ip route`, `show access-lists` |
| OSPF stuck in `INIT` / `2WAY` / `EXSTART` | Hello mismatch, ACL blocking, MTU mismatch, DR rules | `show ip ospf neighbor`, `show ip ospf interface` |
| Intermittent loss, slow links | Duplex mismatch, bad cable, errors | `show interfaces counters errors` |
| Network-wide slowdown | STP loop / broadcast storm | `show spanning-tree summary`, `show processes cpu sorted` |
