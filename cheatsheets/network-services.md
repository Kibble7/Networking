# Network Services Cheat Sheet (NTP, Syslog, SNMP, CDP/LLDP, Backups, QoS)

## NTP (Time Synchronization)
```ios
ntp server 10.10.99.1                     ! Sync to an NTP server
ntp server 10.10.99.1 prefer              ! Preferred source
ntp master 3                              ! Make this device an authoritative source (lab use)
clock timezone PST -8                     ! Display offset
clock set 10:30:00 16 October 2026        ! Manual set (privileged EXEC)
service timestamps log datetime msec      ! Timestamps on logs
service timestamps debug datetime msec
```
```ios
show ntp status                           ! "Clock is synchronized", stratum < 16
show ntp associations                     ! Server list; * = selected source
show clock detail
```

## Syslog
```ios
logging host 10.10.99.60                  ! Send logs to a syslog server (UDP 514)
logging trap informational                ! Severity level sent to the server (0-6)
logging source-interface loopback0
logging buffered 16384                    ! Keep logs in RAM
logging console warnings                  ! Limit console noise
logging synchronous                       ! (line mode) stop logs from interrupting typing
```
| Level | Keyword | Meaning |
|---|---|---|
| 0 | Emergency | System unusable |
| 1 | Alert | Immediate action needed |
| 2 | Critical | Critical condition |
| 3 | Error | Error condition |
| 4 | Warning | Warning condition |
| 5 | Notification | Normal but significant |
| 6 | Informational | Informational |
| 7 | Debugging | Debug output |

A level includes itself and everything more severe (lower number). Remember: **"Every Awesome Cisco Engineer Will Need Ice cream Daily"**.
```ios
show logging
show logging | include %LINK
```

## SNMP
```ios
! SNMPv2c (legacy, plain-text community)
snmp-server community <ro_string> RO
snmp-server community <rw_string> RW        ! Avoid RW whenever possible
snmp-server location Vancouver-IDF1
snmp-server contact netops@example.com

! SNMPv3 (use this - authentication + encryption)
snmp-server group ADMINS v3 priv
snmp-server user snmpuser ADMINS v3 auth sha <auth_password> priv aes 128 <priv_password>
snmp-server host 10.10.99.60 version 3 priv snmpuser
snmp-server enable traps
```
| Fact | Value |
|---|---|
| Ports | UDP `161` (manager polls agent), UDP `162` (agent sends traps/informs) |
| Operations | `Get`, `Set`, `GetNext`/`GetBulk`, `Trap`, `Inform` |
| Data structure | MIB (OIDs) |
| Security levels (v3) | `noAuthNoPriv`, `authNoPriv`, `authPriv` |

```ios
show snmp
show snmp user
show snmp group
```

## CDP & LLDP (Neighbor Discovery)
```ios
cdp run                                   ! Global (on by default)
no cdp run
interface g0/1
 no cdp enable                            ! Disable on one interface

lldp run                                  ! Global (off by default on most IOS)
interface g0/1
 lldp transmit
 lldp receive
```
| | CDP | LLDP |
|---|---|---|
| Standard | Cisco proprietary | IEEE 802.1AB (open) |
| Default timers | 60s send / 180s hold | 30s send / 120s hold |
| Layer | 2 | 2 |
```ios
show cdp neighbors
show cdp neighbors detail
show cdp interface
show lldp neighbors
show lldp neighbors detail
```
Security tip: turn CDP/LLDP off on ports that face users or the Internet.

## Backup, Restore & IOS Images
```ios
copy running-config startup-config                          ! Save to NVRAM
copy running-config tftp://10.10.99.70/R1-confg             ! Back up config to TFTP
copy tftp://10.10.99.70/R1-confg running-config             ! Restore (merges into running config)
copy tftp://10.10.99.70/<ios-image.bin> flash:              ! Upload a new IOS image
copy running-config flash:backup-config
verify /md5 flash:<ios-image.bin>                           ! Check image integrity
boot system flash:<ios-image.bin>                           ! Choose the boot image
show flash:
show boot
```
| Protocol | Port | Notes |
|---|---|---|
| TFTP | UDP `69` | No authentication; lab and trusted networks only |
| FTP | TCP `20/21` | Needs `ip ftp username/password` |
| SCP / SFTP | TCP `22` | Encrypted; preferred (`ip scp server enable`) |

## QoS Concepts (CCNA-level)
| Concept | Meaning |
|---|---|
| Classification & marking | Identify traffic and tag it (DSCP at Layer 3, CoS at Layer 2) |
| Queuing | Decide order of transmission (CBWFQ, LLQ for voice) |
| Policing | Drop or re-mark traffic above a rate |
| Shaping | Buffer and delay traffic above a rate |
| Congestion avoidance | Drop selected packets early (WRED) |

| Traffic | Common DSCP | Notes |
|---|---|---|
| Voice | `EF` (46) | Low latency, jitter and loss needed |
| Video / interactive | `AF41` (34) | |
| Call signaling | `CS3` (24) | |
| Best effort | `0` (default) | |

Voice targets: one-way delay under `150 ms`, jitter under `30 ms`, loss under `1%`.
