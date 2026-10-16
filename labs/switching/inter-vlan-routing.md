---

## File: `networking-portfolio/labs/switching/inter-vlan-routing.md`


# Inter-VLAN Routing (Router-on-a-Stick & Layer 3 Switching)

## Objective
This lab documents the configuration and verification process for routing traffic between segmented VLANs using both **Router-on-a-Stick (802.1Q subinterfaces)** on `R1` and **Multilayer Switch SVIs** on `DSW1`.

## Skills Demonstrated
- Router 802.1Q subinterface provisioning (`encapsulation dot1Q`)
- Dual-stack IPv4 and IPv6 gateway configuration
- Layer 3 Switch SVI routing (`ip routing`, `interface Vlan<id>`)
- Routed switch ports (`no switchport`)
- Inter-VLAN path verification

## Networking Concepts
Because each VLAN is an isolated Layer 2 broadcast domain, hosts in different VLANs require a Layer 3 router or multilayer switch to communicate. **Router-on-a-Stick (ROAS)** terminates multiple VLAN tags on logical subinterfaces of a single router physical port. **Layer 3 Switching** routes between internal Switched Virtual Interfaces (SVIs) in hardware ASICs at wire speed.

## Topology
![Inter-VLAN Routing Topology](../../topologies/images/inter-vlan-routing-topology.png)
Refer to the [Enterprise Campus Topology](../../topologies/README.md). `DSW1 Gi0/0` connects via an 802.1Q trunk to `R1 G0/0/0`.

## Addressing Plan
| Device | Interface | Network | Address | Mask/Prefix | Gateway |
|---|---|---|---|---|---|
| R1 | `G0/0/0.10` | `10.10.10.0/24` | `10.10.10.1` | `/24` | Local |
| R1 | `G0/0/0.10` (IPv6) | `2001:db8:acad:10::/64` | `2001:db8:acad:10::1` | `/64` (`fe80::1`) | Local |
| R1 | `G0/0/0.20` | `10.10.20.0/24` | `10.10.20.1` | `/24` | Local |
| R1 | `G0/0/0.20` (IPv6) | `2001:db8:acad:20::/64` | `2001:db8:acad:20::1` | `/64` (`fe80::1`) | Local |
| R1 | `G0/0/0.99` | `10.10.99.0/24` | `10.10.99.1` | `/24` | Local |
| R1 | `G0/0/0.999` | Native VLAN 999 | Unassigned | — | — |

## Configuration
### Part 1: Router-on-a-Stick (R1)
```ios
ipv6 unicast-routing

interface GigabitEthernet0/0/0
 description TRUNK_TO_DSW1
 no ip address
 no shutdown

interface GigabitEthernet0/0/0.10
 description ENGINEERING_GATEWAY
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
 ipv6 address 2001:db8:acad:10::1/64
 ipv6 address fe80::1 link-local

interface GigabitEthernet0/0/0.20
 description SALES_GATEWAY
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
 ipv6 address 2001:db8:acad:20::1/64
 ipv6 address fe80::1 link-local

interface GigabitEthernet0/0/0.99
 description MANAGEMENT_GATEWAY
 encapsulation dot1Q 99
 ip address 10.10.99.1 255.255.255.0

interface GigabitEthernet0/0/0.999
 description NATIVE_VLAN_999
 encapsulation dot1Q 999 native

```

 ## part 2: Layer 3 Switch Alternative (DSW1)
 ```ios
 ip routing
ipv6 unicast-routing

interface Vlan10
 description ENGINEERING_SVI
 ip address 10.10.10.1 255.255.255.0
 ipv6 address 2001:db8:acad:10::1/64
 no shutdown

interface Vlan20
 description SALES_SVI
 ip address 10.10.20.1 255.255.255.0
 ipv6 address 2001:db8:acad:20::1/64
 no shutdown

! Configure Routed Uplink to R1
interface GigabitEthernet0/0
 no switchport
 ip address 10.10.254.2 255.255.255.252
 no shutdown

 ```
 ## Verification
 ```ios
show ip interface brief — Verify all subinterfaces (G0/0/0.10, .20, .99) or SVIs (Vlan10, Vlan20) report up/up.

show ip route & show ipv6 route — Confirm connected (C) and local (L) routes exist for all VLAN subnets.

show vlans — Verify 802.1Q VLAN tag bindings on router subinterfaces.
```

## Troubleshooting
Problem
In the ROAS deployment, R1 subinterfaces are up/up, but PC-A (10.10.10.10) cannot ping its default gateway 10.10.10.1.

## Investigation
Inspected show interfaces Gi0/0 switchport on DSW1 facing R1.

## Root Cause
DSW1 Gi0/0 was left in switchport mode access instead of switchport mode trunk, dropping all 802.1Q-tagged frames sent to and from R1.

## Resolution
Configured switchport trunk encapsulation dot1q and switchport mode trunk on DSW1 Gi0/0.

## Verification
Verified trunk formation with show interfaces trunk and confirmed inter-VLAN pings between 10.10.10.10 and 10.10.20.10.

## Key Takeaways
This lab demonstrates both branch-level subinterface routing (ROAS) and campus-grade Layer 3 SVI switching, highlighting the requirement for matching 802.1Q tags between switch trunks and router subinterfaces.