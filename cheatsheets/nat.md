# NAT & PAT Cheat Sheet

## NAT Terminology Matrix
| Term | Whose address | Seen from | Example |
|---|---|---|---|
| **Inside Local** | Inside host's real (private) IP | Inside LAN | `10.10.10.10` |
| **Inside Global** | Inside host's translated (public) IP | Outside / Internet | `203.0.113.2` |
| **Outside Local** | Outside host's IP as seen by inside devices | Inside LAN | `198.51.100.1` |
| **Outside Global** | Outside host's real public IP | Outside / Internet | `198.51.100.1` |

Remember: **Inside / Outside** = which side the host is on. **Local / Global** = which side is *viewing* the address. Outside Local usually equals Outside Global unless you translate the destination too.

## NAT Types
| Type | Mapping | Use case |
|---|---|---|
| Static NAT | 1:1, permanent | Servers that must be reachable from the Internet |
| Dynamic NAT | Many private → pool of public (1:1 while in use) | Rare today; limited by pool size |
| PAT (NAT Overload) | Many private → 1 public IP, distinguished by **port** | Standard home/office Internet sharing |

## Mark Inside and Outside Interfaces (required for all NAT types)
```ios
interface g0/0/0
 description LAN
 ip nat inside
interface g0/0/1
 description WAN
 ip nat outside
```

## Quick Configuration Patterns
```ios
! Static NAT (1:1)
ip nat inside source static <inside_local_ip> <inside_global_ip>

! Dynamic NAT (Pool)
ip nat pool <POOL_NAME> <start_ip> <end_ip> netmask <mask>
access-list <ACL_ID> permit <inside_network> <wildcard>
ip nat inside source list <ACL_ID> pool <POOL_NAME>

! PAT / NAT Overload (Interface)
access-list <ACL_ID> permit <inside_network> <wildcard>
ip nat inside source list <ACL_ID> interface <wan_interface> overload

! PAT with a pool
ip nat inside source list <ACL_ID> pool <POOL_NAME> overload

! Static PAT / port forwarding (Internet:8080 -> inside web server:80)
ip nat inside source static tcp <inside_local_ip> 80 <inside_global_ip> 8080
```

### Complete PAT Example
```ios
access-list 1 permit 192.168.10.0 0.0.0.255
interface g0/0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
interface g0/0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
ip nat inside source list 1 interface g0/0/1 overload
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

## Verify & Troubleshoot
```ios
show ip nat translations                  ! Active mappings (Inside local -> Inside global, with ports)
show ip nat translations verbose
show ip nat statistics                    ! Interface bindings, hits, misses, pool usage
clear ip nat translation *                ! Wipe dynamic entries, then retest
debug ip nat                              ! Live NAT events (use briefly; turn off with "undebug all")
```

| Symptom | Likely cause |
|---|---|
| No translations appear | Missing `ip nat inside` / `ip nat outside`, or ACL does not match the inside source |
| `Misses` increase, `Hits` do not | ACL wildcard wrong, or traffic is not routed out the NAT outside interface |
| Only one host works at a time | `overload` keyword is missing |
| Pool exhausted | Dynamic NAT pool too small - use PAT |
| Return traffic never arrives | Upstream has no route back to the inside global address |
