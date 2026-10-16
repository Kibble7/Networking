---

## File: `networking-portfolio/labs/services/network-management.md`


# Network Management: NTP, CDP, LLDP & TFTP Configuration Backup

## Objective
This lab documents the configuration and verification process for synchronizing network clocks via Network Time Protocol (NTP), auditing physical topology connections with CDP and LLDP, and backing up device configurations to a TFTP server.

## Skills Demonstrated
- NTP Master and Client synchronization and verification
- Selective CDP and LLDP neighbor discovery management
- TFTP configuration backup and restoration

## Networking Concepts
- **NTP (UDP 123):** Synchronizes device clocks to a hierarchical **Stratum** reference clock so security logs and debug outputs share an accurate timeline.
- **CDP & LLDP:** Layer 2 discovery protocols that advertise local system capabilities and port details to directly connected neighbors. Security best practice requires disabling discovery protocols on external untrusted WAN/access ports.
- **TFTP (UDP 69):** Provides rapid configuration archival and disaster recovery for `running-config` and `startup-config`.

## Addressing Plan
| Device | Role | Management IP | NTP Role | TFTP Server IP |
|---|---|---|---|---|
| R1 | Branch Router | `10.10.99.1/24` | NTP Master (Stratum 4) | `10.10.99.50` |
| DSW1 | Distribution Switch | `10.10.99.2/24` | NTP Client | `10.10.99.50` |
| ASW1 | Access Switch | `10.10.99.11/24` | NTP Client | `10.10.99.50` |

## Configuration
```ios
! 1. Configure R1 as Authoritative NTP Server and Disable CDP on WAN
R1(config)# ntp master 4
R1(config)# lldp run
R1(config)# interface GigabitEthernet0/0/1
R1(config-if)# no cdp enable
R1(config-if)# no lldp transmit
R1(config-if)# no lldp receive

! 2. Configure DSW1 & ASW1 as NTP Clients with Timestamped Logging
ASW1(config)# ntp server 10.10.99.1
ASW1(config)# service timestamps log datetime msec
ASW1(config)# lldp run

! 3. Archive Running Configuration to TFTP Server
R1# copy running-config startup-config
R1# copy running-config tftp:
Address or name of remote host []? 10.10.99.50
Destination filename [r1-confg]? R1-2026-backup.cfg
```

## Verification
- show ntp status — Confirm Clock is synchronized, stratum 5, reference is 10.10.99.1.

- show ntp associations — Verify ~10.10.99.1 is marked with * (sys.peer).

- show cdp neighbors detail & show lldp neighbors detail — Confirm adjacent device hostnames, management IPs, and port mappings on internal links, and verify zero neighbors on G0/0/1.

## Key Takeaways
This lab demonstrates operational visibility and configuration lifecycle management by pairing accurate NTP time correlation with secure Layer 2 topology discovery and off-box TFTP backups.