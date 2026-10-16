# Routing Table Verification & Troubleshooting

## Routing Table Codes (`show ip route`)
- `L` — Local interface address (`/32` host route)
- `C` — Directly connected subnet (`AD = 0`)
- `S` — Static route (`AD = 1`)
- `S*` — Candidate default static route (`0.0.0.0/0`)
- `O` — OSPF dynamic route (`AD = 110`)

See [Routing Troubleshooting Guide](../../troubleshooting/routing-troubleshooting.md).