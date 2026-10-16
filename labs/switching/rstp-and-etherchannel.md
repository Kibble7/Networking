---

## File: `networking-portfolio/labs/switching/rstp-and-etherchannel.md`


# Rapid STP, PortFast, BPDU Guard & LACP EtherChannel

## Objective
This lab documents the configuration and verification process for deterministic Rapid PVST+ Root Bridge placement, edge port hardening with PortFast and BPDU Guard, and link aggregation using IEEE 802.3ad LACP EtherChannel.

## Skills Demonstrated
- Rapid PVST+ (`rapid-pvst`) deployment
- Root Bridge primary/secondary election tuning
- PortFast and BPDU Guard edge protection
- IEEE 802.3ad LACP EtherChannel bundling (`active`/`passive`)
- Layer 2 redundancy verification

## Networking Concepts
- **Rapid PVST+ (802.1w):** Eliminates Layer 2 loops per VLAN while achieving sub-second convergence via Proposal/Agreement handshakes. Placing the Root Bridge on the distribution switch (`DSW1`) ensures optimal traffic paths.
- **PortFast & BPDU Guard:** Bypasses Learning/Listening delays on host access ports while immediately disabling (`err-disabled`) any access port that receives a BPDU.
- **LACP EtherChannel:** Combines multiple physical links into a single logical `Port-channel` interface so Spanning Tree treats the bundle as one logical link, providing load balancing and automatic link failover without STP recalculation.

## Topology
![RSTP and EtherChannel Topology](../../topologies/images/rstp-etherchannel-topology.png)
`DSW1` and `ASW1` are interconnected via redundant trunk links `Gi0/1` and `Gi0/2` bundled into `Port-channel1`.

## Addressing Plan
| Device | Interface | Channel Group | Protocol | Mode | Trunk Native VLAN | Allowed VLANs |
|---|---|---|---|---|---|---|
| DSW1 | `Gi0/1 - 2` | `Po1` | LACP | `active` | `999` | `10,20,99,150` |
| ASW1 | `Gi0/1 - 2` | `Po1` | LACP | `passive` | `999` | `10,20,99,150` |

## Configuration
```ios
! 1. Configure Rapid PVST+ and Root Bridge on DSW1
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,99,150 root primary

! 2. Configure Rapid PVST+ and Edge Protection on ASW1
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 24
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

! 3. Configure LACP EtherChannel on DSW1 (Active)
interface range GigabitEthernet0/1 - 2
 shutdown
 channel-group 1 mode active
 no shutdown

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,150

! 4. Configure LACP EtherChannel on ASW1 (Passive)
interface range GigabitEthernet0/1 - 2
 shutdown
 channel-group 1 mode passive
 no shutdown

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,150
 ```

 ## Verification
- show etherchannel summary — Expected result: Group 1 displays Po1(SU) (Layer 2, In Use) with protocol LACP and member ports Gi0/1(P) and Gi0/2(P).

- show spanning-tree vlan 10 — Expected result: DSW1 lists This bridge is the root (Priority 24586), and ASW1 lists Po1 as its Root port in FWD state with reduced port cost.

- show spanning-tree interface FastEthernet0/1 detail — Confirm The port is in the portfast mode and Bpdu guard is enabled.

## Troubleshooting
Problem
Po1 reports Po1(SD) (Down) and member interfaces display flag (I) (stand-alone).

## Investigation
Inspected show running-config interface Gi0/1 on both switches.

## Root Cause
Both DSW1 and ASW1 were configured with channel-group 1 mode passive, preventing either side from initiating LACP negotiation packets.

## Resolution
Changed DSW1 interfaces Gi0/1 - 2 to channel-group 1 mode active.

## Verification
Confirmed show etherchannel summary transitioned to Po1(SU) with both members in (P) bundled state.

## Key Takeaways
This lab demonstrates resilient Layer 2 campus design by aligning the RSTP Root Bridge with the Layer 3 distribution gateway, protecting edge ports against rogue switches, and aggregating uplinks with LACP.