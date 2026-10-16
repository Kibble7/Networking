# IPv4 Subnetting — FLSM & VLSM

**Classification:** Networking Fundamentals / Reference & Quick Reference

> A practical IPv4 subnetting reference covering CIDR, subnet masks, usable hosts, subnet increments, FLSM, VLSM, private address ranges, and common subnetting calculations.

---

## 1. IPv4 Address Structure

An IPv4 address contains **32 bits**, divided into four 8-bit octets.

```text
192.168.10.25
   ↓    ↓  ↓
  Octet 1 2 3 4
```

Each octet can contain values from:

```text
0 – 255
```

Example:

```text
192.168.10.25/24
```

The `/24` indicates that the first **24 bits** are the network portion.

```text
Network bits       Host bits
11111111.11111111.11111111.00000000
<-------- 24 ---------><--- 8 --->

192.168.10.25/24
```

---

# 2. CIDR / Prefix Length

CIDR notation indicates how many of the 32 IPv4 bits belong to the network portion.

| CIDR | Subnet Mask | Network Bits | Host Bits |
|---:|---|---:|---:|
| /8 | 255.0.0.0 | 8 | 24 |
| /9 | 255.128.0.0 | 9 | 23 |
| /10 | 255.192.0.0 | 10 | 22 |
| /11 | 255.224.0.0 | 11 | 21 |
| /12 | 255.240.0.0 | 12 | 20 |
| /13 | 255.248.0.0 | 13 | 19 |
| /14 | 255.252.0.0 | 14 | 18 |
| /15 | 255.254.0.0 | 15 | 17 |
| /16 | 255.255.0.0 | 16 | 16 |
| /17 | 255.255.128.0 | 17 | 15 |
| /18 | 255.255.192.0 | 18 | 14 |
| /19 | 255.255.224.0 | 19 | 13 |
| /20 | 255.255.240.0 | 20 | 12 |
| /21 | 255.255.248.0 | 21 | 11 |
| /22 | 255.255.252.0 | 22 | 10 |
| /23 | 255.255.254.0 | 23 | 9 |
| /24 | 255.255.255.0 | 24 | 8 |
| /25 | 255.255.255.128 | 25 | 7 |
| /26 | 255.255.255.192 | 26 | 6 |
| /27 | 255.255.255.224 | 27 | 5 |
| /28 | 255.255.255.240 | 28 | 4 |
| /29 | 255.255.255.248 | 29 | 3 |
| /30 | 255.255.255.252 | 30 | 2 |
| /31 | 255.255.255.254 | 31 | 1 |
| /32 | 255.255.255.255 | 32 | 0 |

---

# 3. Common Subnetting Reference Table

| CIDR | Subnet Mask | Total Addresses | Usable Hosts* | Block Size |
|---:|---|---:|---:|---:|
| /24 | 255.255.255.0 | 256 | 254 | 256 |
| /25 | 255.255.255.128 | 128 | 126 | 128 |
| /26 | 255.255.255.192 | 64 | 62 | 64 |
| /27 | 255.255.255.224 | 32 | 30 | 32 |
| /28 | 255.255.255.240 | 16 | 14 | 16 |
| /29 | 255.255.255.248 | 8 | 6 | 8 |
| /30 | 255.255.255.252 | 4 | 2 | 4 |
| /31 | 255.255.255.254 | 2 | 2† | 2 |
| /32 | 255.255.255.255 | 1 | 1‡ | 1 |

\* Traditional IPv4 subnet calculation reserves the network and broadcast addresses.

† `/31` is commonly used for point-to-point links and does not use a traditional broadcast address.

‡ `/32` represents a single host address, commonly used for loopbacks, host routes, and specific endpoint identification.

---

# 4. Quick Host Requirement Table

| Required Hosts | Minimum CIDR | Usable Hosts | Total Addresses |
|---:|---:|---:|---:|
| 2 | /30 | 2 | 4 |
| 6 | /29 | 6 | 8 |
| 14 | /28 | 14 | 16 |
| 30 | /27 | 30 | 32 |
| 62 | /26 | 62 | 64 |
| 126 | /25 | 126 | 128 |
| 254 | /24 | 254 | 256 |
| 510 | /23 | 510 | 512 |
| 1,022 | /22 | 1,022 | 1,024 |
| 2,046 | /21 | 2,046 | 2,048 |
| 4,094 | /20 | 4,094 | 4,096 |
| 8,190 | /19 | 8,190 | 8,192 |
| 16,382 | /18 | 16,382 | 16,384 |
| 32,766 | /17 | 32,766 | 32,768 |
| 65,534 | /16 | 65,534 | 65,536 |

### Fast Rule

Choose the **smallest subnet that can accommodate the required number of hosts**.

Formula:

```text
2^h - 2 >= required hosts
```

Where:

```text
h = number of host bits
```

---

# 5. Core Subnetting Formulas

### Number of Subnets

When borrowing `b` bits:

```text
Number of subnets = 2^b
```

Example:

```text
/24 → /27

Borrowed bits = 3

2^3 = 8 subnets
```

### Number of Hosts

For traditional IPv4 subnets:

```text
Usable hosts = 2^h - 2
```

Where `h` is the number of host bits.

Example:

```text
/27

32 - 27 = 5 host bits

2^5 - 2
= 32 - 2
= 30 usable hosts
```

---

# 6. Borrowed Bits

Subnetting moves bits from the host portion into the network portion.

Example:

```text
Original:

/24
11111111.11111111.11111111.00000000


Subnetted:

/27
11111111.11111111.11111111.11100000
                                   ↑
                              3 borrowed bits
```

Therefore:

```text
2^3 = 8 subnets
```

Each subnet contains:

```text
2^5 = 32 addresses
```

Traditionally:

```text
32 - 2 = 30 usable hosts
```

---

# 7. Subnet Increment / Magic Number

The subnet increment can be calculated as:

```text
256 - subnet mask value
```

Example:

```text
/26

Mask:
255.255.255.192

256 - 192 = 64
```

Therefore the subnet increment is:

```text
64
```

The networks occur every 64 addresses:

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

---

# 8. Common Magic Numbers

| CIDR | Mask | Interesting Octet | Increment |
|---:|---|---:|---:|
| /25 | 255.255.255.128 | 128 | 128 |
| /26 | 255.255.255.192 | 192 | 64 |
| /27 | 255.255.255.224 | 224 | 32 |
| /28 | 255.255.255.240 | 240 | 16 |
| /29 | 255.255.255.248 | 248 | 8 |
| /30 | 255.255.255.252 | 252 | 4 |

### Memory Trick

```text
/25 → 128
/26 → 64
/27 → 32
/28 → 16
/29 → 8
/30 → 4
```

Each additional subnet bit cuts the block size in half.

---

# 9. Network / Host / Broadcast

For a traditional IPv4 subnet:

```text
Network Address
      ↓
First usable host
      ↓
...
Last usable host
      ↓
Broadcast Address
```

Example:

```text
192.168.10.0/26
```

Block size:

```text
64
```

Therefore:

```text
Network:       192.168.10.0
First Host:    192.168.10.1
Last Host:     192.168.10.62
Broadcast:     192.168.10.63
```

Next subnet:

```text
Network:       192.168.10.64
First Host:    192.168.10.65
Last Host:     192.168.10.126
Broadcast:     192.168.10.127
```

---

# 10. FLSM — Fixed Length Subnet Masking

FLSM means **Fixed Length Subnet Masking**.

Every subnet uses the **same subnet mask**.

Example:

```text
Network: 192.168.1.0/24
```

Suppose four equal-sized networks are required.

Borrow 2 bits:

```text
/24 → /26

2^2 = 4 subnets
```

Result:

| Subnet | Network | Usable Range | Broadcast |
|---|---|---|---|
| 1 | 192.168.1.0/26 | .1 – .62 | .63 |
| 2 | 192.168.1.64/26 | .65 – .126 | .127 |
| 3 | 192.168.1.128/26 | .129 – .190 | .191 |
| 4 | 192.168.1.192/26 | .193 – .254 | .255 |

Every subnet has:

```text
64 total addresses
62 traditional usable hosts
```

### When to Use FLSM

FLSM is useful when:

- Network segments require similar numbers of hosts.
- Simplicity is more important than address efficiency.
- All subnets can use the same prefix length.

---

# 11. VLSM — Variable Length Subnet Masking

VLSM allows different subnet sizes within the same larger address block.

Example requirements:

```text
LAN A → 100 hosts
LAN B → 50 hosts
LAN C → 20 hosts
WAN   → 2 hosts
```

Instead of giving every network the same-size subnet, allocate only what each network needs.

---

# 12. VLSM Design Rule

Always start with the **largest requirement**.

### Example

```text
100 hosts
50 hosts
20 hosts
2 hosts
```

Sort:

```text
100
50
20
2
```

Then choose the smallest subnet that satisfies each requirement.

| Requirement | Prefix | Usable Hosts | Block Size |
|---:|---:|---:|---:|
| 100 | /25 | 126 | 128 |
| 50 | /26 | 62 | 64 |
| 20 | /27 | 30 | 32 |
| 2 | /30 | 2 | 4 |

---

# 13. Complete VLSM Example

Starting network:

```text
192.168.10.0/24
```

Requirements:

```text
LAN A → 100 hosts
LAN B → 50 hosts
LAN C → 20 hosts
WAN   → 2 hosts
```

### Step 1 — LAN A

100 hosts requires `/25`.

```text
192.168.10.0/25
```

Range:

```text
Network:     192.168.10.0
Hosts:       192.168.10.1 – 192.168.10.126
Broadcast:   192.168.10.127
```

Remaining:

```text
192.168.10.128 – 192.168.10.255
```

### Step 2 — LAN B

50 hosts requires `/26`.

```text
192.168.10.128/26
```

Range:

```text
Network:     192.168.10.128
Hosts:       192.168.10.129 – 192.168.10.190
Broadcast:   192.168.10.191
```

Remaining:

```text
192.168.10.192 – 192.168.10.255
```

### Step 3 — LAN C

20 hosts requires `/27`.

```text
192.168.10.192/27
```

Range:

```text
Network:     192.168.10.192
Hosts:       192.168.10.193 – 192.168.10.222
Broadcast:   192.168.10.223
```

Remaining:

```text
192.168.10.224 – 192.168.10.255
```

### Step 4 — WAN Link

2 hosts requires `/30`.

```text
192.168.10.224/30
```

Range:

```text
Network:     192.168.10.224
Hosts:       192.168.10.225 – 192.168.10.226
Broadcast:   192.168.10.227
```

Remaining:

```text
192.168.10.228 – 192.168.10.255
```

---

# 14. VLSM Allocation Summary

| Network | Requirement | Prefix | Network | Usable Hosts | Broadcast |
|---|---:|---:|---|---|---|
| LAN A | 100 | /25 | 192.168.10.0 | .1 – .126 | .127 |
| LAN B | 50 | /26 | 192.168.10.128 | .129 – .190 | .191 |
| LAN C | 20 | /27 | 192.168.10.192 | .193 – .222 | .223 |
| WAN | 2 | /30 | 192.168.10.224 | .225 – .226 | .227 |

Unused address space:

```text
192.168.10.228 – 192.168.10.255
```

This can be reserved for future expansion.

---

# 15. VLSM Rules to Remember

1. Start with the largest host requirement.
2. Choose the smallest subnet that satisfies the requirement.
3. Allocate the subnet from the beginning of the available address space.
4. Move to the next available boundary.
5. Continue from largest to smallest.
6. Never allow subnet ranges to overlap.
7. Leave room for future growth when designing production networks.
8. Document every allocation.

---

# 16. Common Prefixes by Network Type

| Network Type | Typical Prefix | Traditional Usable Hosts |
|---|---:|---:|
| Large LAN | /23 | 510 |
| Standard LAN | /24 | 254 |
| Medium LAN | /25 | 126 |
| Small LAN | /26 | 62 |
| Small VLAN | /27 | 30 |
| Very Small Segment | /28 | 14 |
| Infrastructure Segment | /29 | 6 |
| Point-to-Point IPv4 | /30 | 2 |
| Host/Loopback | /32 | 1 |

> These are common design choices, not mandatory rules. Actual subnet sizes should be based on requirements.

---

# 17. Private IPv4 Address Ranges

Private IPv4 addresses are defined by RFC 1918.

| Range | CIDR | Address Space |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | /8 | Large private networks |
| 172.16.0.0 – 172.31.255.255 | /12 | Private networks |
| 192.168.0.0 – 192.168.255.255 | /16 | Small/home/private networks |

### Quick Memory

```text
10/8
172.16/12
192.168/16
```

---

# 18. Special IPv4 Addresses

| Address / Range | Purpose |
|---|---|
| 0.0.0.0 | Unspecified / default route context |
| 127.0.0.0/8 | Loopback |
| 127.0.0.1 | Common loopback address |
| 169.254.0.0/16 | IPv4 link-local / APIPA |
| 224.0.0.0/4 | IPv4 multicast |
| 255.255.255.255 | Limited broadcast |
| 10.0.0.0/8 | Private |
| 172.16.0.0/12 | Private |
| 192.168.0.0/16 | Private |

---

# 19. Subnetting Decision Process

When given a subnetting question:

### Step 1 — Identify the requirement

Ask:

```text
How many hosts?
How many subnets?
```

### Step 2 — Determine host bits

Find the smallest `h` where:

```text
2^h - 2 >= required hosts
```

### Step 3 — Determine prefix

```text
Prefix = 32 - host bits
```

### Step 4 — Determine subnet mask

Convert the prefix into dotted-decimal notation.

### Step 5 — Determine block size

```text
256 - interesting octet
```

### Step 6 — Find subnet boundaries

Start at zero and increment using the block size.

### Step 7 — Identify

For the required IP:

```text
Network address
First usable host
Last usable host
Broadcast address
```

---

# 20. Fast Subnetting Table

Memorize this:

```text
/24 → 256 addresses → 254 hosts
/25 → 128 addresses → 126 hosts
/26 →  64 addresses →  62 hosts
/27 →  32 addresses →  30 hosts
/28 →  16 addresses →  14 hosts
/29 →   8 addresses →   6 hosts
/30 →   4 addresses →   2 hosts
```

And:

```text
/25 → increment 128
/26 → increment 64
/27 → increment 32
/28 → increment 16
/29 → increment 8
/30 → increment 4
```

---

# 21. Example: Find the Network of an IP

Given:

```text
192.168.50.77/27
```

`/27` gives:

```text
Mask = 255.255.255.224
Increment = 256 - 224
          = 32
```

Subnet boundaries:

```text
0
32
64
96
128
160
192
224
```

`77` falls between:

```text
64 – 95
```

Therefore:

```text
Network:     192.168.50.64
First Host:  192.168.50.65
Last Host:   192.168.50.94
Broadcast:   192.168.50.95
```

---

# 22. Example: Determine the Subnet From Host Requirement

Requirement:

```text
45 hosts
```

Try `/27`:

```text
2^5 - 2 = 30
```

Not enough.

Try `/26`:

```text
2^6 - 2 = 62
```

Enough.

Therefore:

```text
Required subnet = /26
```

---

# 23. FLSM vs VLSM

| Feature | FLSM | VLSM |
|---|---|---|
| Subnet sizes | Same | Different |
| Mask | Same for all subnets | Can vary |
| Address efficiency | Lower | Higher |
| Complexity | Simpler | More complex |
| Best for | Equal-size networks | Different-size networks |
| Common use | Simple designs | Efficient enterprise designs |

### Easy Memory

```text
FLSM = Fixed
VLSM = Variable
```

---

# 24. Common Subnetting Mistakes

### Mistake 1 — Forgetting the network address

The network address identifies the subnet and is not normally assigned to a host.

### Mistake 2 — Forgetting the broadcast address

Traditional IPv4 subnets reserve the last address as the broadcast address.

### Mistake 3 — Choosing a subnet that is too small

Always verify:

```text
2^h - 2 >= required hosts
```

### Mistake 4 — VLSM allocation in the wrong order

Allocate:

```text
Largest → Smallest
```

### Mistake 5 — Overlapping VLSM ranges

Every subnet must have a unique address range.

### Mistake 6 — Confusing block size with usable hosts

For example:

```text
/26

64 total addresses
62 traditional usable hosts
```

Not 64 usable hosts.

---

# 25. Subnetting Exam / Interview Quick Reference

```text
IPv4 = 32 bits

Usable hosts:
2^h - 2

Subnets:
2^b

Prefix:
32 - host bits

Block size:
256 - interesting octet

VLSM:
Largest → Smallest

/24 = 254 hosts
/25 = 126 hosts
/26 = 62 hosts
/27 = 30 hosts
/28 = 14 hosts
/29 = 6 hosts
/30 = 2 hosts
```

---

# 26. Practical Network Design Reminder

Subnetting is not only about calculating addresses.

A good network design considers:

- Current host requirements
- Future growth
- VLAN structure
- Broadcast-domain size
- Routing boundaries
- Point-to-point links
- Address conservation
- Network documentation
- Troubleshooting simplicity
- Security segmentation

The goal is to create an addressing plan that is **efficient, scalable, understandable, and easy to troubleshoot**.
