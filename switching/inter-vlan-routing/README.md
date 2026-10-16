# Inter-VLAN Routing

## Architectural Models
1. **Legacy Inter-VLAN Routing:** Separate physical router interface per VLAN (does not scale).
2. **Router-on-a-Stick (ROAS):** Single physical 802.1Q trunk between switch and router using logical router subinterfaces (`G0/0/0.10`, `G0/0/0.20`).
3. **Layer 3 Switching (SVIs):** Hardware-switched internal routing on a multilayer switch using `ip routing` and virtual `interface Vlan<id>` gateways.

## Implementation Link
See [Lab: Inter-VLAN Routing](../../labs/switching/inter-vlan-routing.md).