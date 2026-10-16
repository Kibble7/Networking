# Spanning Tree Protocol (STP & Rapid PVST+)

## Core Concepts
- **Loop Prevention:** Elects a single **Root Bridge** (lowest Bridge ID = Priority + Extended System ID + MAC) and blocks redundant Layer 2 paths to prevent broadcast storms.
- **Rapid STP (IEEE 802.1w / Rapid PVST+):** Replaces legacy 802.1D timers (30–50 second convergence) with active Proposal/Agreement handshakes for sub-second convergence.
  - Port Roles: `Root Port`, `Designated Port`, `Alternate Port`, `Backup Port`.
  - Port States: `Discarding`, `Learning`, `Forwarding`.
- **PortFast & BPDU Guard:** `PortFast` immediately transitions access ports to `Forwarding`. `BPDU Guard` shuts down (`err-disable`) any PortFast-enabled interface that receives a BPDU.

## Implementation Link
See [Lab: Rapid STP & LACP EtherChannel](../../labs/switching/rstp-and-etherchannel.md).