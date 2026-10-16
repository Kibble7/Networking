# Switching & Layer 2 Troubleshooting

## Diagnostic Scenarios
1. **Access Port Unreachable:**
   - **Symptom:** Host shows link down or cannot ping gateway.
   - **Checks:** `show interfaces status err-disabled` (Port Security or BPDU Guard violation), `show vlan brief` (port assigned to inactive or wrong VLAN).
   - **Fix:** Remove rogue device, then bounce interface (`shutdown` / `no shutdown`).
2. **802.1Q Trunk Failure:**
   - **Symptom:** Hosts on `ASW1` can ping each other in VLAN 10 but cannot reach `R1` or `ASW2`.
   - **Checks:** `show interfaces trunk` — inspect **Vlans allowed on trunk** and **Vlans in spanning tree forwarding state and not pruned**.
   - **Fix:** Ensure `switchport trunk allowed vlan add <id>` includes the missing VLAN on both switches.
3. **EtherChannel Member Suspended (`s`) or Stand-alone (`I`):**
   - **Symptom:** `show etherchannel summary` displays `Po1(SD)` or member flags `(I)` / `(s)`.
   - **Fix:** Align speed, duplex, trunk mode, native VLAN, and LACP mode (`active`/`passive`) across all physical member ports.