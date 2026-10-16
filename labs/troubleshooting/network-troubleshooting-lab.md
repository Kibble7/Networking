

## File: `networking-portfolio/labs/troubleshooting/network-troubleshooting-lab.md`


# Multi-Layer Network Troubleshooting & Verification Lab

## Objective
This lab documents a structured, multi-ticket troubleshooting scenario isolating and resolving interconnected Layer 2, Layer 3, Services, and Security faults across the enterprise campus topology.

## Skills Demonstrated
- Bottom-up and divide-and-conquer fault isolation
- 802.1Q trunk and VLAN mismatch resolution
- Static routing and next-hop reachability repair
- DHCP Relay and Extended ACL troubleshooting

---

## Ticket 1: Sales Hosts Failing to Acquire DHCPv4 Addresses
### Problem
Workstations in VLAN 20 (`Sales`) receive `169.254.x.x` APIPA addresses and cannot reach `R1` (`10.10.20.1`).
### Investigation
1. Checked `show interfaces status` on `ASW1` → `Fa0/2` was `connected` in VLAN 20.
2. Checked `show ip interface brief` on `R1` → `G0/0/0.20` (`10.10.20.1`) was `up/up`.
3. Checked `show running-config interface G0/0/0.20` on `R1` → Observed `encapsulation dot1Q 200` and `ip helper-address 10.10.99.99`.
### Root Cause
Compound fault:
1. Subinterface `G0/0/0.20` was tagging VLAN `200` instead of VLAN `20`.
2. `ip helper-address` pointed to `.99` instead of the DHCP server at `10.10.99.50`.
### Resolution
```ios
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
 no ip helper-address 10.10.99.99
 ip helper-address 10.10.99.50
```

 ## Verification
Renewed the client DHCP lease (ipconfig /renew) and confirmed 10.10.20.10/24 was assigned and recorded in show ip dhcp binding.

## Ticket 2: Engineering Hosts Cannot Reach Remote HQ (10.20.20.0/24)
### Problem
PC-A (10.10.10.10) can ping its gateway (10.10.10.1) and the Internet (203.0.113.1), but pings to 10.20.20.10 fail with Destination host unreachable from 203.0.113.1.

### Investigation
- Executed traceroute 10.20.20.10 from PC-A and saw traffic forwarded to the ISP (203.0.113.1) instead of R2 (10.0.12.2).

- Checked show ip route on R1 → No static route existed for 10.20.20.0/24, so traffic matched the 0.0.0.0/0 default route to the ISP.

- Checked show running-config | include ip route on R1 → Found ip route 10.20.20.0 255.255.255.0 10.0.12.9.

### Root Cause
The static route next-hop (10.0.12.9) was outside the /30 WAN subnet (10.0.12.0/30), rendering the next-hop unreachable and preventing the route from entering the RIB.

### Resolution
```ios 
no ip route 10.20.20.0 255.255.255.0 10.0.12.9
ip route 10.20.20.0 255.255.255.0 10.0.12.2
```

### Verification
Confirmed S 10.20.20.0/24 [1/0] via 10.0.12.2 appeared in show ip route and end-to-end pings succeeded.

### Key Takeaways
This lab demonstrates disciplined CLI verification—confirming Layer 2 VLAN tags before troubleshooting Layer 3 DHCP relay, and verifying next-hop reachability when static routes fail to install in the routing table.