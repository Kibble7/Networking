# Access Control Lists (ACL) Cheat Sheet

## Standard vs. Extended ACLs
| Feature | Standard ACL (`1–99`, `1300–1999`) | Extended ACL (`100–199`, `2000–2699`) |
|---|---|---|
| Match Criteria | Source IPv4 Address only | Source/Dest IP, Protocol (`tcp/udp/icmp/ip`), L4 Ports |
| Optimal Placement | Closest to **Destination** | Closest to **Source** |
| VTY Line Binding | `access-class <acl> in` | Possible, but standard is typical |
| Interface Binding | `ip access-group <acl> {in \| out}` | `ip access-group <acl> {in \| out}` |

## Rules That Always Apply
- ACLs are processed **top-down**; the first match wins.
- There is an invisible **implicit `deny any`** at the end of every ACL.
- One ACL per **interface, per direction, per protocol**.
- Outbound ACLs do **not** filter traffic that the router itself generates.
- `in` = traffic entering the interface; `out` = traffic leaving it.
- Named ACLs can be edited by sequence number; numbered ACLs are best deleted and rewritten (or converted to named).

## Wildcard Mask Shortcuts
- `0.0.0.0` (`host 10.1.1.5`) — Match exact host IP.
- `0.0.0.255` — Match `/24` subnet.
- `255.255.255.255` (`any`) — Match any IP address.
- **Calculation:** `255.255.255.255 - Subnet Mask = Wildcard Mask`.
- Example: `/27` → `255.255.255.224` → wildcard `0.0.0.31`.
- Range example: `192.168.10.0 0.0.1.255` matches `192.168.10.0/24` **and** `192.168.11.0/24`.

## Standard ACL Configuration
```ios
! Numbered
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 deny host 192.168.20.5
access-list 10 permit any

! Named (preferred)
ip access-list standard ADMIN_ONLY
 10 permit host 192.168.99.10
 20 deny any log

interface g0/0/1
 ip access-group 10 out
```

## Extended ACL Configuration
```ios
ip access-list extended WEB_ONLY
 10 remark Allow web and DNS from Sales LAN
 20 permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.10 eq 80
 30 permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.10 eq 443
 40 permit udp 192.168.10.0 0.0.0.255 host 10.1.1.53 eq 53
 50 permit icmp 192.168.10.0 0.0.0.255 any echo
 60 deny ip any any log

interface g0/0/0
 ip access-group WEB_ONLY in
```

### Port & Protocol Keywords
| Keyword | Meaning |
|---|---|
| `eq 80` / `eq www` | Equal to port 80 |
| `neq 23` | Not equal to port 23 |
| `gt 1023` / `lt 1024` | Greater than / less than |
| `range 20 21` | Port range |
| `established` | TCP segments with ACK or RST set (return traffic of an inside-initiated session) |
| `log` | Log each match to syslog |
| `echo` / `echo-reply` | ICMP ping request / reply |

### Typical Patterns
```ios
permit tcp any any established                    ! Allow return TCP traffic
deny   tcp any any eq 23                          ! Block Telnet
permit tcp host 192.168.99.10 any eq 22           ! Allow SSH only from the admin host
permit ip any any                                 ! Allow everything else (put LAST)
```

## Secure VTY Access
```ios
access-list 5 permit 192.168.99.0 0.0.0.255
line vty 0 15
 access-class 5 in
 transport input ssh
```

## Edit ACLs
```ios
ip access-list extended WEB_ONLY
 no 30                                            ! Remove one line by sequence number
 25 permit tcp any host 10.1.1.10 eq 8080         ! Insert a line between 20 and 30
ip access-list resequence WEB_ONLY 10 10          ! Renumber from 10 in steps of 10
no ip access-list extended WEB_ONLY               ! Delete the whole ACL
```

## IPv6 ACL
```ios
ipv6 access-list BLOCK_TELNET_V6
 deny tcp any any eq 23
 permit ipv6 any any

interface g0/0/0
 ipv6 traffic-filter BLOCK_TELNET_V6 in
```
Every IPv6 ACL has implicit `permit icmp any any nd-na`, `permit icmp any any nd-ns` and then `deny ipv6 any any`. The ND permits keep Neighbor Discovery working.

## Verify & Troubleshoot
```ios
show access-lists
show ip access-lists WEB_ONLY
show ip interface g0/0/0 | include access list    ! Is the ACL applied, and which direction?
show running-config | section access-list
clear access-list counters                        ! Reset match counters, then retest
show ipv6 access-list
```
Check `(X matches)` counters to see which line is catching traffic. If the counter on your `permit` line stays at 0, the traffic is hitting a rule above it, the ACL is on the wrong interface/direction, or the traffic never reaches this device.
