# VLAN Segmentation, Voice VLANs & 802.1Q Trunking

## Objective
This lab documents the configuration and verification process for segmenting a Layer 2 switch fabric into departmental VLANs, provisioning a dedicated Voice VLAN with QoS markings, securing 802.1Q trunk links with a non-default Native VLAN, and disabling Dynamic Trunking Protocol (DTP).

## Skills Demonstrated
- VLAN creation and naming
- Access port and Voice VLAN configuration
- 802.1Q trunk configuration and allowed VLAN pruning
- Native VLAN reassignment (`VLAN 999`)
- Layer 2 verification and troubleshooting

## Networking Concepts
VLANs partition a single physical switch infrastructure into distinct broadcast domains. To carry multiple VLANs across inter-switch links, IEEE 802.1Q inserts a 4-byte tag into Ethernet frames. Any untagged traffic traversing an 802.1Q link is assigned to the **Native VLAN**. Reassigning the Native VLAN away from `VLAN 1` and disabling DTP (`switchport nonegotiate`) mitigates VLAN hopping and unauthorized trunk formation.

## Topology
![VLAN & Trunking Topology](../../topologies/images/vlan-trunking-topology.png)
Refer to the [Enterprise Campus Topology](../../topologies/README.md). `ASW1` and `ASW2` connect to `DSW1` via 802.1Q trunks over `GigabitEthernet0/1`.

## Addressing Plan
| Device | Interface | VLAN | Network | Address | Mask/Prefix | Gateway |
|---|---|---|---|---|---|---|
| ASW1 | Vlan99 | 99 | `10.10.99.0` | `10.10.99.11` | `255.255.255.0` (`/24`) | `10.10.99.1` |
| ASW2 | Vlan99 | 99 | `10.10.99.0` | `10.10.99.12` | `255.255.255.0` (`/24`) | `10.10.99.1` |
| PC-A | NIC | 10 | `10.10.10.0` | `10.10.10.10` | `255.255.255.0` (`/24`) | `10.10.10.1` |
| PC-B | NIC | 20 | `10.10.20.0` | `10.10.20.10` | `255.255.255.0` (`/24`) | `10.10.20.1` |
## Verification

- `show vlan brief` — Expected result: VLANs `10, 20, 99, 150, 999` appear as `active`, with `Fa0/1` in VLAN 10, `Fa0/2` in VLAN 20 and 150, and `Fa0/3–24` in VLAN 999.
- `show interfaces trunk` — Expected result: `Gi0/1` shows mode `on`, encapsulation `802.1q`, status `trunking`, Native VLAN `999`, and allowed VLANs `10,20,99,150`.
- `show interfaces FastEthernet0/2 switchport` — Expected result: Operational Mode is `static access`, Access Mode VLAN is `20 (SALES)`, and Voice VLAN is `150 (VOICE)`.

## Configuration
```ios
! 1. Create VLANs on ASW1
vlan 10
 name ENGINEERING
vlan 20
 name SALES
vlan 99
 name MANAGEMENT
vlan 150
 name VOICE
vlan 999
 name NATIVE_UNUSED
exit

! 2. Configure Access Port and Voice VLAN
interface FastEthernet0/1
 description ENGINEERING_HOST
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate

interface FastEthernet0/2
 description SALES_PHONE_AND_PC
 switchport mode access
 switchport access vlan 20
 switchport voice vlan 150
 mls qos trust cos

! 3. Isolate Unused Switchports
interface range FastEthernet0/3 - 24
 description UNUSED_PORTS
 switchport mode access
 switchport access vlan 999
 shutdown

! 4. Configure 802.1Q Trunk Uplink
interface GigabitEthernet0/1
 description TRUNK_TO_DSW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,150
 switchport nonegotiate

! 5. Configure Management SVI
interface Vlan99
 ip address 10.10.99.11 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.99.1 

```


## Troubleshooting

### Problem

Hosts in VLAN 20 on `ASW1` cannot communicate with hosts in VLAN 20 on `ASW2`, while VLAN 10 traffic succeeds.

### Investigation

Executed `show interfaces trunk` on `ASW1`, `DSW1`, and `ASW2`.

### Root Cause

On `DSW1`, the trunk interface was configured with `switchport trunk allowed vlan 10,99`, inadvertently omitting VLAN 20 from the allowed VLAN list.

### Resolution

Executed `switchport trunk allowed vlan add 20,150` on `DSW1` trunk interfaces.

### Verification

Confirmed `show interfaces trunk` listed `10,20,99,150` under `Vlans in spanning tree forwarding state and not pruned`.

## Key Takeaways

This lab demonstrates practical Layer 2 broadcast segmentation, voice/data convergence on a single cable drop, and trunk hardening using explicit VLAN pruning and Native VLAN isolation.
