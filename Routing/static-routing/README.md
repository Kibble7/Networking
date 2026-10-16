# Static Routing

## Static Route Types
- **Next-Hop Static Route:** Specifies only the next-hop IP address (`ip route 10.10.20.0 255.255.255.0 10.0.0.2`). Requires a recursive lookup to resolve the exit interface.
- **Directly Attached Static Route:** Specifies only the outgoing interface. Recommended only on point-to-point serial interfaces.
- **Fully Specified Static Route:** Specifies both the outgoing interface and the next-hop IP (`ip route 10.10.20.0 255.255.255.0 G0/0/1 10.0.0.2`), eliminating recursive lookups on multi-access Ethernet networks (mandatory for IPv6 link-local next-hops).

## Implementation Link
See [Lab: Static, Default, Host & Floating Static Routing](../../labs/routing/static-and-floating-routing.md).