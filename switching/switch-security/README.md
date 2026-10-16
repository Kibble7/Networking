# Layer 2 Switch Security

## Threat Mitigation Controls
- **MAC Flooding Mitigation:** Port Security limits the maximum number of MAC addresses learned on an access port (`switchport port-security maximum <n>`).
- **Rogue DHCP Server Mitigation:** DHCP Snooping inspects DHCP messages and drops `DHCPOFFER` / `DHCPACK` packets arriving on untrusted access ports.
- **VLAN Hopping Mitigation:** Disable DTP (`switchport nonegotiate`), explicitly set `switchport mode access`, and change the trunk Native VLAN to an unused ID.

## Implementation Link
See [Lab: Switch Security](../../labs/security/switch-security.md).