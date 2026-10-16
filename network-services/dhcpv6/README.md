# Stateless & Stateful DHCPv6

## Router Advertisement (RA) Flag Control
| Method | M Flag (Managed) | O Flag (Other) | Address Source | DNS/Domain Source |
|---|---|---|---|---|
| SLAAC Only | `0` | `0` | RA Prefix + EUI-64/Random | RA (RDNSS) |
| Stateless DHCPv6 | `0` | `1` (`ipv6 nd other-config-flag`) | RA Prefix (SLAAC) | DHCPv6 Server |
| Stateful DHCPv6 | `1` (`ipv6 nd managed-config-flag`) | `0` or `1` | DHCPv6 Server Pool | DHCPv6 Server |