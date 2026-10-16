# EtherChannel & LACP (IEEE 802.3ad)

## Core Concepts
- Bundles up to 8 physical Ethernet interfaces into a single logical `Port-channel` interface for increased bandwidth, load balancing, and redundancy.
- **LACP Modes:**
  - `active`: Unconditionally initiates LACP negotiation.
  - `passive`: Responds to LACP packets but does not initiate (`passive` + `passive` fails to bundle).
- **Bundle Consistency Requirements:** All physical member ports must match in speed, duplex, access/trunk mode, native VLAN, and allowed VLAN list.

## Implementation Link
See [Lab: Rapid STP & LACP EtherChannel](../../labs/switching/rstp-and-etherchannel.md).