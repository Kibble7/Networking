# DHCP Snooping

## Operation
Builds and maintains a **DHCP Snooping Binding Database** (MAC, IP, Lease Time, VLAN, Interface) by classifying switch ports:
- **Trusted Ports:** Uplinks toward legitimate DHCP servers (`ip dhcp snooping trust`). Allowed to forward `DHCPOFFER` and `DHCPACK` messages.
- **Untrusted Ports (Default):** All end-host access ports. Drops any inbound server responses (`DHCPOFFER`, `DHCPACK`) and rate-limits `DHCPDISCOVER` floods (`ip dhcp snooping limit rate 15`).