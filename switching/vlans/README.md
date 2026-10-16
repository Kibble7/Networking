# VLANs & Voice VLANs

## Core Concepts
- **Broadcast Domain Segmentation:** Each VLAN creates an isolated Layer 2 broadcast domain, improving security and reducing unnecessary broadcast flooding.
- **Data vs. Voice VLANs:** Access ports connected to Cisco IP phones use two VLANs:
  - `switchport access vlan <id>` for untagged PC traffic.
  - `switchport voice vlan <id>` for tagged 802.1Q voice frames with IEEE 802.1p CoS priority (`5` for RTP audio).

## Implementation Link
See [Lab: VLAN Segmentation & 802.1Q Trunking](../../labs/switching/vlan-segmentation-and-trunking.md).