# Routing Troubleshooting

## Diagnostic Scenarios
1. **Missing Return Route (Asymmetric Blackhole):**
   - **Symptom:** Ping from `PC-A (10.10.10.10)` to remote branch `10.20.20.10` times out (`. . . .`), even though `R1` has a valid route to `10.20.20.0/24`.
   - **Investigation:** Packet reaches the destination router `R2`, but `R2` lacks a route back to `10.10.10.0/24`.
   - **Verification:** Run an extended ping (`ping` → source `10.10.10.1`) and inspect `show ip route 10.10.10.0` on `R2`.
2. **Static Route Missing from `show ip route`:**
   - **Cause:** A static route is only installed in the RIB if the resolved **exit interface is `up/up`** and the **next-hop IP is reachable** via a connected or local route.
3. **Suboptimal Path / Floating Static Route Always Active:**
   - **Cause:** Primary link failed or the floating static route was configured without an Administrative Distance higher than the primary route.