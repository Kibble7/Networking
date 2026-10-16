---

## File: `networking-portfolio/labs/security/switch-security.md`


# Layer 2 Switch Security: Port Security & DHCP Snooping

## Objective
This lab documents the configuration and verification process for mitigating Layer 2 MAC flooding, unauthorized device attachment, and rogue DHCP servers using Port Security and DHCP Snooping.

## Skills Demonstrated
- Port Security configuration with Sticky MAC learning and violation modes
- DHCP Snooping global and per-VLAN activation
- Trusted vs. untrusted DHCP Snooping port classification and rate limiting
- Layer 2 security verification and `err-disabled` recovery

## Networking Concepts
- **Port Security:** Restricts an access port to a defined number of source MAC addresses. Using `mac-address sticky` dynamically learns the MAC and writes it to `running-config`. When the maximum is exceeded, the port executes `protect`, `restrict`, or `shutdown` (`err-disabled`).
- **DHCP Snooping:** Acts as a Layer 2 firewall between untrusted host ports and trusted DHCP server uplinks, dropping rogue `DHCPOFFER` / `DHCPACK` messages on untrusted interfaces.

## Configuration
```ios
! 1. Enable DHCP Snooping on ASW1
ip dhcp snooping
ip dhcp snooping vlan 10,20
no ip dhcp snooping information option

! Trust the Uplink Trunk Toward the Authorized DHCP Server/Relay
interface GigabitEthernet0/1
 description TRUSTED_UPLINK_TO_DSW1
 ip dhcp snooping trust

! Rate-Limit DHCP Packets on Untrusted Access Ports
interface range FastEthernet0/1 - 24
 ip dhcp snooping limit rate 15

! 2. Configure Port Security on Access Ports
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 ```
 ## Verification
- show port-security interface FastEthernet0/1 — Verify Port Security : Enabled, Port Status : Secure-up, Violation Mode : Restrict, and Maximum MAC Addresses : 2.

- show port-security address — Inspect Sticky MAC entries bound to each interface.

- show ip dhcp snooping — Confirm trusted interface Gi0/1 (Trusted: yes) and rate limits on Fa0/1–24.

- show ip dhcp snooping binding — Verify the snooping database records leased MAC-to-IP bindings.

## Troubleshooting
### Problem
An employee replaces a desktop on Fa0/2 with a different laptop, and the switch port immediately goes dark (down/down).

### Investigation
Ran show interfaces status and observed Fa0/2 in err-disabled state. show port-security interface Fa0/2 showed Security Violation Count : 1 and Port Status : Secure-shutdown.

### Root Cause
Fa0/2 had sticky MAC maximum 1 with violation shutdown. Connecting a new MAC triggered an err-disabled shutdown.

### Resolution
Cleared the old sticky MAC (no switchport port-security mac-address sticky <old_mac>), then bounced the port with shutdown followed by no shutdown.

### Key Takeaways
This lab demonstrates practical Layer 2 access-layer defense against CAM table overflow and rogue DHCP spoofing attacks.