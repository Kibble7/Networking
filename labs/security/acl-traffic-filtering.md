---

## File: `networking-portfolio/labs/security/acl-traffic-filtering.md`

# Access Control Lists (Standard, Extended, Named & VTY)

## Objective
This lab documents the configuration and verification process for enforcing Layer 3 and Layer 4 packet filtering using Standard, Extended, and Named IPv4 ACLs, as well as restricting administrative VTY access via `access-class`.

## Skills Demonstrated
- Wildcard mask calculation and ACE ordering
- Extended Named ACL filtering by protocol (`tcp`, `icmp`, `ip`) and port (`eq 22`, `eq 80`, `eq 443`)
- Standard Named ACL enforcement on VTY management lines
- ACL placement and hit counter verification

## Networking Concepts
ACLs filter packets sequentially top-to-bottom and conclude with an implicit `deny ip any any`. **Extended ACLs** inspect source/destination IPs and Layer 4 ports and should be placed **closest to the traffic source** to prevent unwanted traffic from consuming bandwidth. **Standard ACLs** inspect only the source IP and should be placed **closest to the destination** (or applied to VTY lines via `access-class`).

## Topology
![ACL Traffic Filtering Topology](../../topologies/images/acl-filtering-topology.png)
Refer to the [Enterprise Campus Topology](../../topologies/README.md). Policy requirements:
1. Sales (`10.10.20.0/24`) is prohibited from SSHing to the Management subnet (`10.10.99.0/24`), can access HTTPS (`443`) on `10.10.99.50`, and is permitted all other traffic.
2. Only the Management subnet (`10.10.99.0/24`) is permitted VTY remote access to `R1`.

## Configuration
```ios
! 1. Extended Named ACL Applied Inbound on Sales Gateway (Closest to Source)
ip access-list extended ACL_SALES_INBOUND
 remark Block Sales from SSH to Management Subnet
 deny tcp 10.10.20.0 0.0.0.255 10.10.99.0 0.0.0.255 eq 22 log
 remark Permit Sales HTTPS access to Internal Server
 permit tcp 10.10.20.0 0.0.0.255 host 10.10.99.50 eq 443
 remark Permit all other legitimate IPv4 traffic
 permit ip 10.10.20.0 0.0.0.255 any
exit

interface GigabitEthernet0/0/0.20
 ip access-group ACL_SALES_INBOUND in

! 2. Standard Named ACL Restricting VTY Lines on R1
ip access-list standard ACL_VTY_MGMT_ONLY
 remark Permit only Management VLAN 99
 permit 10.10.99.0 0.0.0.255
 deny any log
exit

line vty 0 4
 access-class ACL_VTY_MGMT_ONLY in

```

## Verification
- show access-lists — Confirm ACE sequence numbers and verify matches counters increment when test traffic is generated:

```ios
Extended IP access list ACL_SALES_INBOUND
    10 deny tcp 10.10.20.0 0.0.0.255 10.10.99.0 0.0.0.255 eq 22 log (4 matches)
    20 permit tcp 10.10.20.0 0.0.0.255 host 10.10.99.50 eq 443 (12 matches)
    30 permit ip 10.10.20.0 0.0.0.255 any (28 matches)
```

- show ip interface GigabitEthernet0/0/0.20 — Confirm Inbound access list is ACL_SALES_INBOUND.

## Troubleshooting
### Problem
After applying an ACL to block a single host (10.10.20.99), all traffic from the entire Sales VLAN drops.

### Investigation
Reviewed show access-lists and found only 10 deny ip host 10.10.20.99 any.

### Root Cause
Every ACL ends with an implicit deny ip any any. Without an explicit permit ip any any at the end of the ACL, all non-matching traffic is silently dropped.

### Resolution
Added sequence 20 permit ip any any inside the named ACL configuration mode.

### Key Takeaways
This lab demonstrates least-privilege network segmentation using Extended ACLs at ingress boundaries and VTY control-plane restriction via access-class.