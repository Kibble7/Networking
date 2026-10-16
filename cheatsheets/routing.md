# Routing Cheat Sheet

## Administrative Distance (AD) Reference
| Route Source | Default AD |
|---|---|
| Connected Interface (`C`) | `0` |
| Static Route (`S`) | `1` |
| Floating Static Route | `> 1` (User configured, e.g., `5` or `200`) |
| eBGP | `20` |
| EIGRP (Internal) | `90` |
| OSPF | `110` |
| IS-IS | `115` |
| RIP | `120` |
| EIGRP (External) | `170` |
| iBGP | `200` |
| Unreachable / unknown | `255` |

## Path Selection Order of Operations
1. **Longest-Prefix Match:** A `/32` host route beats a `/24` route, which beats a `/16` route, regardless of AD or metric.
2. **Lowest Administrative Distance (AD):** Evaluated only when multiple routing sources advertise the **exact same prefix length**.
3. **Lowest Metric:** Evaluated within a single routing protocol (e.g., OSPF cost); equal metrics result in Equal-Cost Multi-Path (ECMP) load balancing.

## Reading `show ip route`
| Code | Meaning |
|---|---|
| `C` / `L` | Connected network / local interface address (`/32`) |
| `S` / `S*` | Static route / static default route (candidate default) |
| `O` | OSPF intra-area |
| `O IA` | OSPF inter-area |
| `O E1` / `O E2` | OSPF external type 1 / type 2 |
| `D` / `D EX` | EIGRP internal / EIGRP external |
| `B` | BGP |
| `R` | RIP |

Entry format: `O  10.2.2.0/24 [110/2] via 10.1.1.2, 00:05:12, G0/0/0` → `[AD/metric]` via next hop, age, exit interface.
`Gateway of last resort is ...` shows the active default route.

## Static Routes (IPv4)
```ios
ip route 192.168.2.0 255.255.255.0 10.0.0.2              ! Next-hop IP (recursive lookup)
ip route 192.168.2.0 255.255.255.0 g0/0/1                ! Exit interface (point-to-point links)
ip route 192.168.2.0 255.255.255.0 g0/0/1 10.0.0.2       ! Fully specified (best on Ethernet)
ip route 0.0.0.0 0.0.0.0 10.0.0.2                        ! Default route
ip route 192.168.2.0 255.255.255.0 10.0.0.6 5            ! Floating static (AD 5, backup path)
ip route 10.1.1.5 255.255.255.255 10.0.0.2               ! Host route
```
IPv6 static routes are in [`ipv6.md`](./ipv6.md).

## OSPFv2 (Single Area Quick Config)
```ios
router ospf 1                                   ! Process ID is locally significant
 router-id 1.1.1.1                              ! Manual router ID (best practice)
 network 10.0.0.0 0.0.0.255 area 0              ! Enable OSPF on matching interfaces
 passive-interface g0/0/0                       ! Advertise LAN, send no Hellos on it
 passive-interface default                      ! Or: passive everywhere...
 no passive-interface g0/0/1                    ! ...then enable only on links to neighbors
 default-information originate                  ! Advertise a default route to neighbors
 auto-cost reference-bandwidth 10000            ! Set on ALL routers (value in Mbps)
 maximum-paths 4                                ! Number of ECMP paths (default 4)

! Alternative: enable OSPF per interface (no network statement)
interface g0/0/1
 ip ospf 1 area 0
 ip ospf cost 10                                ! Override calculated cost
 ip ospf priority 100                           ! DR/BDR election (0 = never DR)
 ip ospf network point-to-point                 ! No DR/BDR election on P2P links
 ip ospf hello-interval 10
 ip ospf dead-interval 40

clear ip ospf process                           ! Restart OSPF (needed after changing router-id)
```

| OSPF Facts | Value |
|---|---|
| Cost formula | Reference bandwidth ÷ interface bandwidth (default reference `100 Mbps`) |
| Hello / Dead (broadcast and point-to-point) | `10s` / `40s` |
| Hello / Dead (NBMA) | `30s` / `120s` |
| Multicast addresses | `224.0.0.5` (all OSPF routers), `224.0.0.6` (DR/BDR) |
| Router ID order | Manual `router-id` → highest loopback IP → highest active interface IP |
| DR election | Highest priority, then highest router ID (non-preemptive) |
| Neighbor states | Down → Init → 2-Way → ExStart → Exchange → Loading → Full |
| Must match to become neighbors | Area ID, subnet + mask, Hello/Dead timers, authentication, area type, unique router ID |

## OSPFv3 (IPv6)
```ios
ipv6 unicast-routing
ipv6 router ospf 1
 router-id 1.1.1.1                              ! Still a 32-bit ID; must be set manually if no IPv4 address
interface g0/0/1
 ipv6 ospf 1 area 0
```

## EIGRP (Legacy / Optional for CCNA, common in real networks)
```ios
router eigrp 100                                ! AS number must match on neighbors
 eigrp router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255
 passive-interface g0/0/0
 no auto-summary
```
Default metric uses **bandwidth + delay**. Feasible Successor = backup route that meets the feasibility condition. Verify with `show ip eigrp neighbors` and `show ip eigrp topology`.

## First-Hop Redundancy: HSRP
```ios
interface g0/0/0
 ip address 192.168.1.2 255.255.255.0
 standby version 2
 standby 1 ip 192.168.1.254                     ! Virtual gateway IP given to hosts
 standby 1 priority 110                         ! Default 100; higher = Active
 standby 1 preempt                              ! Take over when priority is higher
 standby 1 track g0/0/1 20                      ! Lower priority by 20 if uplink fails
```

| FHRP Facts | Value |
|---|---|
| HSRP | Cisco proprietary; Active / Standby; Hello `3s`, Hold `10s` |
| VRRP | Open standard; Master / Backup |
| GLBP | Cisco proprietary; load balances across gateways |
| Verify | `show standby brief`, `show standby` |

## Verify Routing
```ios
show ip route
show ip route 10.2.2.5                          ! Which route matches this destination?
show ip route static
show ip route ospf
show ip protocols                               ! Active protocols, router ID, networks, passive interfaces
show ip ospf neighbor
show ip ospf interface brief
show ip ospf interface g0/0/1
show ip ospf database
show ipv6 ospf neighbor
show ip eigrp neighbors
show standby brief
```
