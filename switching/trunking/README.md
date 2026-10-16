# IEEE 802.1Q Trunking & Native VLANs

## Core Concepts
- **802.1Q Tagging:** Inserts a 4-byte VLAN tag into Ethernet frames traversing inter-switch or switch-to-router links.
- **Native VLAN:** Carries untagged frames across a trunk link. Best practice mandates reassigning the Native VLAN from `VLAN 1` to an unused, unrouted VLAN (e.g., `VLAN 999`) on both ends of the trunk.
- **DTP Mitigation:** Disable Dynamic Trunking Protocol (`switchport nonegotiate`) after setting explicit `switchport mode trunk` or `switchport mode access`.

## Implementation Link
See [Lab: VLAN Segmentation & 802.1Q Trunking](../../labs/switching/vlan-segmentation-and-trunking.md).