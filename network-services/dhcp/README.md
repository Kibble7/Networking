# DHCPv4 & DHCP Relay

## DORA Process
1. **DHCPDISCOVER (Broadcast):** Client broadcasts (`0.0.0.0` → `255.255.255.255`, UDP `68` → `67`) to locate a DHCP server.
2. **DHCPOFFER (Unicast/Broadcast):** Server offers an available IP lease, mask, default gateway, and DNS server.
3. **DHCPREQUEST (Broadcast):** Client formally requests the offered IP parameters.
4. **DHCPACK (Unicast/Broadcast):** Server confirms the lease binding.

## DHCP Relay (`ip helper-address`)
Because routers drop Layer 2 broadcasts by default, `ip helper-address <dhcp_server_ip>` configured on the router's LAN gateway interface intercepts broadcast DHCPDISCOVER packets and forwards them as **unicast** packets to a centralized DHCP server on another subnet.