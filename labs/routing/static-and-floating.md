---

## File: `networking-portfolio/labs/routing/static-and-floating-routing.md`

```markdown
# Static, Default, Floating Static & Host Routing

## Objective
This lab documents the configuration and verification process for IPv4 and IPv6 static routing, default gateway routing, redundant WAN failover using floating static routes, and `/32` and `/128` host routes.

## Skills Demonstrated
- Next-hop and fully specified static routes (IPv4 & IPv6)
- Default route (`0.0.0.0/0` and `::/0`) configuration
- Floating static route failover via Administrative Distance tuning
- Host route (`/32` and `/128`) traffic steering
- Routing table inspection and failover verification

## Networking Concepts
- **Static & Default Routes:** Provide deterministic, zero-overhead path control. Default routes (`0.0.0.0/0` and `::/0`) act as the Gateway of Last Resort for unknown external prefixes.
- **Floating Static Routes:** Assigned an Administrative Distance higher than the primary route (e.g., `AD 5` vs. default static `AD 1`). They remain inactive until the primary route's interface fails.
- **Host Routes (`/32` & `/128`):** Evaluated first due to the **Longest-Prefix Match** rule, allowing granular traffic engineering to specific servers or loopback interfaces.

## Topology
![Static and Floating Routing Topology](../../topologies/images/static-floating-routing-topology.png)
`R1` connects to `ISP` via `G0/0/1`, and connects to remote `R2` (`10.20.20.0/24`) via a primary link (`S0/1/0` - `10.0.12.0/30`) and a backup link (`S0/1/1` - `10.0.13.0/30`).

## Addressing Plan
| Device | Interface | Network | Address | Mask/Prefix | Next-Hop |
|---|---|---|---|---|---|
| R1 | `G0/0/1` | `203.0.113.0/30` | `203.0.113.2` | `/30` | `203.0.113.1` (ISP) |
| R1 | `S0/1/0` (Primary) | `10.0.12.0/30` | `10.0.12.1` | `/30` | `10.0.12.2` (R2) |
| R1 | `S0/1/1` (Backup) | `10.0.13.0/30` | `10.0.13.1` | `/30` | `10.0.13.2` (R2) |
| R2 | `G0/0/0` (HQ LAN) | `10.20.20.0/24` | `10.20.20.1` | `/24` | Local |
| R2 | `Loopback100` (Auth Server) | `10.20.20.250/32` | `10.20.20.250` | `/32` | Dedicated Path |

## Configuration
```ios
! R1 Configuration
ipv6 unicast-routing

! 1. Default Routes toward ISP (Gateway of Last Resort)
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ipv6 route ::/0 2001:db8:isp::1

! 2. Primary Static Route to HQ LAN (Default AD = 1)
ip route 10.20.20.0 255.255.255.0 10.0.12.2
ipv6 route 2001:db8:acad:200::/64 2001:db8:12::2

! 3. Backup Floating Static Route to HQ LAN (AD = 5)
ip route 10.20.20.0 255.255.255.0 10.0.13.2 5
ipv6 route 2001:db8:acad:200::/64 2001:db8:13::2 5

! 4. Explicit /32 and /128 Host Routes for Critical Server over Backup Link
ip route 10.20.20.250 255.255.255.255 Serial0/1/1 10.0.13.2
ipv6 route 2001:db8:acad:200::250/128 Serial0/1/1 2001:db8:13::2
```

## Verification
   show ip route static — Expected result:

  - S* 0.0.0.0/0 [1/0] via 203.0.113.1

  - S 10.20.20.0/24 [1/0] via 10.0.12.2 (Floating route AD 5 does not appear while S0/1/0 is up)

  - S 10.20.20.250/32 [1/0] via 10.0.13.2, Serial0/1/1

- Failover Test: Execute shutdown on R1 Serial0/1/0 and verify show ip route 10.20.20.0 immediately installs S 10.20.20.0/24 [5/0] via 10.0.13.2.

## Troubleshooting
Problem
Traffic from R1 to 10.20.20.10 routes over the primary link, but return traffic from R2 traverses the slower backup link.

## Investigation
Checked show ip route on R2.

## Root Cause
On R2, the static route on the primary link was accidentally configured with an AD of 10, while the backup link had the default AD of 1, causing asymmetric routing.

## Resolution
Removed the 10 AD from the primary static route on R2 and set the backup route AD to 5.

## Verification
Executed traceroute in both directions and confirmed symmetric routing over 10.0.12.0/30.

## Key Takeaways
This lab demonstrates how Longest-Prefix Match (/32 > /24 > /0) and Administrative Distance (AD 1 < AD 5) govern router forwarding decisions and automated WAN failover.