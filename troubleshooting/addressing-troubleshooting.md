# Addressing, DHCP & NAT Troubleshooting

## Diagnostic Scenarios
1. **Client Assigned `169.254.x.x` (APIPA):**
   - **Cause:** Client broadcast `DHCPDISCOVER` never reached the DHCP server.
   - **Verification:** Check access VLAN assignment on the switchport; if the DHCP server is on a remote subnet, verify `ip helper-address <server_ip>` is configured on the router subinterface/SVI serving the client VLAN.
2. **DHCP IP Conflict:**
   - **Cause:** Static server IPs were assigned inside the dynamic DHCP pool without configuring `ip dhcp excluded-address <low_ip> <high_ip>`.
   - **Verification:** `show ip dhcp conflict`.
3. **NAT Translation Failure:**
   - **Cause:** Reversed `ip nat inside` / `ip nat outside` interface commands, missing `overload` keyword on PAT rule, or ACL wildcard mask fails to match the internal VLAN subnet.
   - **Verification:** `show ip nat translations` and `show ip nat statistics`.