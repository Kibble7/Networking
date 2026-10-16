# Address Resolution Protocol (ARP)
**Classification:** Networking Fundamentals (Reference / Knowledge)

## IPv4 ARP Resolution Workflow
1. **Destination Check:** Host compares target IP against its own IP and subnet mask.
   - **Same Subnet:** ARPs directly for the target host's IP.
   - **Remote Subnet:** ARPs for the **Default Gateway IP**.
2. **ARP Cache Lookup:** Checks local ARP table (`show ip arp` or `arp -a`).
3. **ARP Request (Broadcast):** Encapsulates ARP query in an Ethernet frame with Destination MAC `FFFF.FFFF.FFFF`.
4. **ARP Reply (Unicast):** Owner of the target IP responds via unicast frame containing its MAC address.

## IPv6 Neighbor Discovery (NDP)
IPv6 eliminates broadcast ARP in favor of ICMPv6 Neighbor Solicitation (Type 135) sent to the **Solicited-Node Multicast** address (`ff02::1:ffXX:XXXX`) and Neighbor Advertisement (Type 136) unicast replies.