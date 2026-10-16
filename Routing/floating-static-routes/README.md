# Floating Static Routes

## Core Concepts
A floating static route provides automatic backup path failover by configuring a higher **Administrative Distance (AD)** than the primary route (e.g., AD `5` vs. default static AD `1` or OSPF AD `110`). The backup route remains hidden from the active RIB until the primary route's interface goes down.

```ios
! Primary Static Route (Default AD = 1)
ip route 10.10.20.0 255.255.255.0 10.0.12.2
! Backup Floating Static Route (AD = 5)
ip route 10.10.20.0 255.255.255.0 10.0.13.2 5