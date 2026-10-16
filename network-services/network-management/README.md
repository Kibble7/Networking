
# Network Management (NTP, CDP, LLDP, TFTP)

## Core Protocols
- **NTP (UDP 123):** Synchronizes system clocks across routers and switches to a reference Stratum time source for accurate syslog and certificate timestamp correlation.
- **CDP (Cisco Discovery Protocol):** Cisco-proprietary Layer 2 discovery protocol advertising device ID, IP address, IOS version, port ID, and Native VLAN.
- **LLDP (IEEE 802.1AB):** Vendor-neutral Layer 2 neighbor discovery standard.
- **TFTP (UDP 69):** Lightweight file transfer protocol used to archive `running-config` / `startup-config` and IOS images to a management server.