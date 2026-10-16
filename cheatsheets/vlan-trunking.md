# VLANs, Trunking, STP & EtherChannel Cheat Sheet

## VLAN & Trunking Quick Reference
- **Access Port:** Carries 1 untagged data VLAN (+ optional tagged Voice VLAN).
- **802.1Q Trunk Port:** Carries multiple tagged VLANs across switches; untagged frames belong to the **Native VLAN**.
- **STP Root Bridge Election:** Lowest Bridge ID (`Priority [Default 32768] + VLAN ID + MAC`).
- **Rapid STP Port Roles:** Root Port (best path to Root), Designated Port (forwarding port per segment), Alternate Port (discarding backup path).
- **PortFast & BPDU Guard:** Enable on host access ports only; immediately puts port in `err-disabled` if a BPDU arrives.
- **VLAN ranges:** Normal `1–1005` (stored in `vlan.dat`), Extended `1006–4094`. VLANs `1`, `1002–1005` cannot be deleted.

## Create VLANs & Assign Access Ports
```ios
vlan 10
 name SALES
vlan 20
 name ENGINEERING
vlan 99
 name MGMT
vlan 999
 name BLACKHOLE                                ! Parking VLAN for unused ports

interface g0/1
 description PC - Sales
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable

interface range g0/10 - 20
 switchport mode access
 switchport access vlan 20

! Access port with an IP phone (data + voice)
interface g0/2
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 150

no vlan 20                                     ! Delete a VLAN (move ports first!)
```

## Trunk Configuration
```ios
interface g0/24
 description Trunk to DIST1
 switchport trunk encapsulation dot1q          ! Only on switches that support ISL too
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99        ! Restrict which VLANs cross the trunk
 switchport trunk allowed vlan add 30          ! Add one VLAN to the existing list
 switchport trunk allowed vlan remove 20       ! Remove one VLAN from the list
 switchport nonegotiate                        ! Disable DTP
```

### DTP Mode Results
| Side A | Side B | Result |
|---|---|---|
| `dynamic auto` | `dynamic auto` | Access |
| `dynamic auto` | `dynamic desirable` | Trunk |
| `dynamic desirable` | `dynamic desirable` | Trunk |
| `trunk` | `dynamic auto` / `desirable` | Trunk |
| `trunk` | `access` | Mismatch - avoid |

Best practice: hard-code `switchport mode access` or `switchport mode trunk`, and add `switchport nonegotiate` on trunks.

## Inter-VLAN Routing

### Router-on-a-Stick (subinterfaces)
```ios
interface g0/0/1
 no shutdown                                   ! Physical port: no IP address
interface g0/0/1.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0/1.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
interface g0/0/1.99
 encapsulation dot1q 99 native
 ip address 192.168.99.1 255.255.255.0
```

### Layer 3 Switch (SVIs)
```ios
ip routing
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```
An SVI only comes up if the VLAN exists and at least one port in that VLAN is up.

## Spanning Tree Protocol (STP)
```ios
spanning-tree mode rapid-pvst                  ! Recommended (802.1w per VLAN)
spanning-tree vlan 1,10,20 root primary        ! Make this switch the root
spanning-tree vlan 1,10,20 root secondary      ! Backup root
spanning-tree vlan 10 priority 4096            ! Manual priority (multiples of 4096)

spanning-tree portfast default                 ! PortFast on all access ports
spanning-tree portfast bpduguard default       ! BPDU Guard on all PortFast ports
spanning-tree guard root                       ! Interface: block superior BPDUs (root guard)
spanning-tree guard loop                       ! Interface: protect against unidirectional links

spanning-tree vlan 10 cost 10                  ! Interface: change path cost
spanning-tree vlan 10 port-priority 64         ! Interface: change port priority
```

| Facts | Value |
|---|---|
| Default bridge priority | `32768` (+ VLAN ID = extended system ID) |
| Hello / Forward delay / Max age | `2s` / `15s` / `20s` |
| Classic port costs | `10 Mbps = 100`, `100 Mbps = 19`, `1 Gbps = 4`, `10 Gbps = 2` |
| Classic STP states | Blocking → Listening → Learning → Forwarding (+ Disabled) |
| Rapid STP states | Discarding → Learning → Forwarding |
| Tie-breakers | Lowest root path cost → lowest sender Bridge ID → lowest sender port priority → lowest port ID |

## EtherChannel (LACP vs PAgP)
| Protocol | Standard | Mode Combinations That Form a Bundle |
|---|---|---|
| **LACP** | IEEE 802.3ad | `active` + `active` OR `active` + `passive` |
| **PAgP** | Cisco Proprietary | `desirable` + `desirable` OR `desirable` + `auto` |
| **Static (`on`)** | Manual | `on` + `on` only |

*All member interfaces must match: Speed, Duplex, Access/Trunk mode, Native VLAN, and Allowed VLANs.*

```ios
interface range g0/1 - 2
 channel-group 1 mode active                   ! LACP (use "desirable" for PAgP, "on" for static)
 no shutdown
interface port-channel 1
 switchport mode trunk                         ! Configure the logical interface, not the members
 switchport trunk allowed vlan 10,20,99

port-channel load-balance src-dst-ip           ! Global: hashing method for member links
```
Layer 3 EtherChannel: `no switchport` on the `port-channel` interface, then give it an IP address.

## VTP (legacy - know it, usually avoid it)
```ios
vtp mode transparent                           ! Safest: ignores and forwards VTP, keeps local VLANs
vtp mode off                                   ! Disabled entirely
show vtp status
```

## Verify
```ios
show vlan brief
show interfaces trunk
show interfaces g0/1 switchport
show interfaces status
show mac address-table dynamic vlan 10
show spanning-tree
show spanning-tree vlan 10
show spanning-tree summary
show spanning-tree root
show spanning-tree interface g0/1 detail
show etherchannel summary
show etherchannel port-channel
show running-config interface g0/1
```
Common trunk problem: **native VLAN mismatch** logs `%CDP-4-NATIVE_VLAN_MISMATCH` and breaks untagged traffic.
