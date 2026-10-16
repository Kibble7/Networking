# Classless Inter-Domain Routing (CIDR)
**Classification:** Networking Fundamentals (Reference / Knowledge)

## Purpose
CIDR replaces rigid Class A/B/C boundaries with variable prefix lengths (`/0` to `/32`), enabling hierarchical **Route Summarization (Supernetting)** to reduce router CPU/memory overhead and prevent routing table bloat.

## Route Summarization Calculation
Given four contiguous networks: `172.16.4.0/24`, `172.16.5.0/24`, `172.16.6.0/24`, `172.16.7.0/24`:
1. Express the third octet in binary:
   - `4` = `00000100`
   - `5` = `00000101`
   - `6` = `00000110`
   - `7` = `00000111`
2. Count matching left-to-right bits: First 6 bits (`000001`) of the third octet match.
3. Combine with first two octets: $16 + 6 = /22$.
4. **Summary Route:** `172.16.4.0/22` (`255.255.252.0`).