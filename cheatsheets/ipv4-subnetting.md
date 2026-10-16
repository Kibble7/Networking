# IPv4 & Subnetting Cheat Sheet

## CIDR Prefix to Subnet Mask Reference
| CIDR | Subnet Mask | Wildcard Mask | Total IPs | Usable Hosts ($2^h - 2$) | Block Size (interesting octet) |
|---|---|---|---|---|---|
| `/8` | `255.0.0.0` | `0.255.255.255` | 16,777,216 | 16,777,214 | 1 (1st octet) |
| `/16` | `255.255.0.0` | `0.0.255.255` | 65,536 | 65,534 | 1 (2nd octet) |
| `/24` | `255.255.255.0` | `0.0.0.255` | 256 | 254 | 1 (3rd octet) |
| `/25` | `255.255.255.128` | `0.0.0.127` | 128 | 126 | 128 |
| `/26` | `255.255.255.192` | `0.0.0.63` | 64 | 62 | 64 |
| `/27` | `255.255.255.224` | `0.0.0.31` | 32 | 30 | 32 |
| `/28` | `255.255.255.240` | `0.0.0.15` | 16 | 14 | 16 |
| `/29` | `255.255.255.248` | `0.0.0.7` | 8 | 6 | 8 |
| `/30` | `255.255.255.252` | `0.0.0.3` | 4 | 2 (P2P WAN) | 4 |
| `/31` | `255.255.255.254` | `0.0.0.1` | 2 | 2 (P2P only, RFC 3021) | 2 |
| `/32` | `255.255.255.255` | `0.0.0.0` | 1 | 1 (Host Route) | 1 |

## Octet Bit Values
| Bit position | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|
| Mask value when bits are on | 128 | 192 | 224 | 240 | 248 | 252 | 254 | 255 |
| Number of network bits in octet | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |

## Rapid Subnetting Process
1. **Find Block Size:** $256 - \text{Mask Octet} = \text{Increment}$.
2. **Network Address:** Multiple of the Block Size less than or equal to the target IP.
3. **Broadcast Address:** $(\text{Next Subnet Network Address}) - 1$.
4. **Usable Host Range:** $(\text{Network Address} + 1)$ through $(\text{Broadcast Address} - 1)$.

### Worked Example 1: `192.168.10.77/27`
- Mask `255.255.255.224` → block size $256 - 224 = 32$
- Subnets: `.0`, `.32`, `.64`, `.96` → `.77` falls in the `.64` subnet
- Network `192.168.10.64`, broadcast `192.168.10.95`
- Usable hosts `192.168.10.65` – `192.168.10.94` (30 hosts)

### Worked Example 2: `172.16.45.200/20`
- Mask `255.255.240.0` → interesting octet is the 3rd, block size $256 - 240 = 16$
- Subnets in 3rd octet: `32`, `48` → `45` falls in `32`
- Network `172.16.32.0`, broadcast `172.16.47.255`
- Usable hosts `172.16.32.1` – `172.16.47.254` (4,094 hosts)

## Quick Planning Formulas
- **Hosts per subnet:** $2^h - 2$ where $h$ = host bits.
- **Number of subnets:** $2^n$ where $n$ = bits borrowed.
- **VLSM rule:** allocate the **largest** subnet first, then the next largest, so blocks stay aligned.
- **Pick the prefix:** find the smallest $h$ where $2^h - 2 \ge$ required hosts (e.g. 50 hosts → $h = 6$ → `/26`).

## Special IPv4 Ranges
- **RFC 1918 Private:** `10.0.0.0/8`, `172.16.0.0/12` (`172.16.0.0–172.31.255.255`), `192.168.0.0/16`
- **APIPA (Link-Local):** `169.254.0.0/16` (host could not reach a DHCP server)
- **Loopback:** `127.0.0.0/8` (`127.0.0.1`)
- **Multicast:** `224.0.0.0/4` (`224.0.0.5` OSPF routers, `224.0.0.6` OSPF DR/BDR, `224.0.0.9` RIPv2, `224.0.0.10` EIGRP)
- **Limited broadcast:** `255.255.255.255`
- **Default route:** `0.0.0.0/0`
- **Documentation (TEST-NET):** `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`

## Classful Reference (for exam context)
| Class | First Octet | Default Mask | Use |
|---|---|---|---|
| A | 1–126 | `/8` | Large networks |
| B | 128–191 | `/16` | Medium networks |
| C | 192–223 | `/24` | Small networks |
| D | 224–239 | n/a | Multicast |
| E | 240–255 | n/a | Experimental |

## Verify Addressing
```ios
show ip interface brief
show interfaces g0/0 | include Internet address
show ip route connected
```
```text
Windows : ipconfig /all
Linux   : ip addr show
```
