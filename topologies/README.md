# Network Topology Documentation

## Enterprise Campus & WAN Edge Architecture
*Image Reference:* `![Enterprise Campus Topology](./images/enterprise-campus-topology.png)`

```text
                       [ ISP / WAN Edge (203.0.113.0/30) ]
                                       |
                                    G0/0/1 (.2) [NAT Outside]
                         +---------------------------+
                         |     R1 (Branch Edge)      |
                         +---------------------------+
                    G0/0/0 (.10, .20, .99, .999) |   \ S0/1/0 (Primary WAN)
                                    802.1Q Trunk |    \ S0/1/1 (Backup Floating Route)
                         +---------------------------+      \
                         |  DSW1 (Distribution Sw)   |    [ R2 - Remote HQ ]
                         +---------------------------+    (10.20.20.0/24)
                           Po1 (LACP: G0/1 + G0/2)
                                    802.1Q Trunk |
              +----------------------------------+----------------------------------+
              | Gi0/1                                                               | Gi0/1
+---------------------------+                                         +---------------------------+
|    ASW1 (Access Switch)   |                                         |    ASW2 (Access Switch)   |
+---------------------------+                                         +---------------------------+
  Fa0/1 (VLAN 10)  Fa0/2 (VLAN 20 + Voice 150)                          Fa0/1 (VLAN 10)  Fa0/24 (Trusted DHCP)
       |                    |                                                |                    |
   [ PC-A ]        [ IP Phone + PC-B ]                                   [ PC-C ]         [ SRV-DHCP/TFTP ]
 (10.10.10.10)        (10.10.20.10)                                    (10.10.10.11)       (10.10.99.50)

 





## Network Segments & VLAN Boundaries

| **VLAN ID** | **Name**        | **IPv4 Subnet**  | **IPv6 Prefix**         | **Layer 3 Default Gateway** | **Layer 2 Boundary**               |
| ----------- | --------------- | ---------------- | ----------------------- | --------------------------- | ---------------------------------- |
| `10`        | `ENGINEERING`   | `10.10.10.0/24`  | `2001:db8:acad:10::/64` | `10.10.10.1` (`fe80::1`)    | `ASW1`, `ASW2`, `DSW1`             |
| `20`        | `SALES`         | `10.10.20.0/24`  | `2001:db8:acad:20::/64` | `10.10.20.1` (`fe80::1`)    | `ASW1`, `ASW2`, `DSW1`             |
| `99`        | `MANAGEMENT`    | `10.10.99.0/24`  | `2001:db8:acad:99::/64` | `10.10.99.1` (`fe80::1`)    | Switch SVIs & `SRV-DHCP/TFTP`      |
| `150`       | `VOICE`         | `10.10.150.0/24` | —                       | `10.10.150.1`               | Tagged Voice on `ASW1 Fa0/2`       |
| `999`       | `NATIVE_UNUSED` | Unrouted         | Unrouted                | None (L2 Isolation)         | Trunk Native VLAN & Shutdown Ports |



## Traffic Flow Analysis

1. **Intra-VLAN Traffic (`PC-A` → `PC-C` in VLAN 10):** Switched purely at Layer 2 across `ASW1 → Po1 (DSW1) → ASW2` using 802.1Q VLAN tag `10` without crossing `R1`.
2. **Inter-VLAN Traffic (`PC-A` VLAN 10 → `PC-B` VLAN 20):** Frame travels to default gateway (`R1 G0/0/0.10` or `DSW1 Vlan10` SVI), decrements IP TTL, rewrites L2 MAC header, and re-tags frame with `VLAN 20`.
3. **Outbound Internet Traffic (`PC-A` → `203.0.113.1`):** Routed to `R1`, inspected by outbound ACLs, translated via PAT (`G0/0/1` `203.0.113.2`), and forwarded to the ISP.