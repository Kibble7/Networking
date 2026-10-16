# IOS Basics & First-Time Device Config

## CLI Modes
| Prompt | Mode | How to enter | How to leave |
|---|---|---|---|
| `Router>` | User EXEC | Login | `exit` |
| `Router#` | Privileged EXEC | `enable` | `disable` |
| `Router(config)#` | Global config | `configure terminal` | `end` / `Ctrl+Z` |
| `Router(config-if)#` | Interface config | `interface g0/0` | `exit` (one level) |
| `Router(config-line)#` | Line config | `line vty 0 15` | `exit` |
| `Router(config-router)#` | Routing protocol | `router ospf 1` | `exit` |
| `Switch(config-vlan)#` | VLAN config | `vlan 10` | `exit` |

## CLI Navigation & Productivity
```ios
?                              ! Help: list commands in this mode
sh<Tab>                        ! Auto-complete
show ip int br                 ! Abbreviations work if unambiguous
do show ip route               ! Run a privileged command from config mode
no <command>                   ! Remove a command
default interface g0/1         ! Reset an interface to factory defaults
interface range g0/1 - 4       ! Configure several ports at once
terminal length 0              ! Stop the --More-- pager
show history                   ! Recent commands
Ctrl+Shift+6                   ! Abort a ping/traceroute
```

### Output filters
```ios
show running-config | include ip route
show running-config | section ospf
show running-config | begin line vty
show ip interface brief | exclude unassigned
show running-config interface g0/1
```

## First-Time Router / Switch Hardening Baseline
```ios
enable
configure terminal
hostname R1
no ip domain-lookup                              ! Stop DNS lookups on typos
service password-encryption
security passwords min-length 10
enable secret <password>
banner motd #Authorized access only. Activity is monitored.#

username admin privilege 15 secret <password>
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
login block-for 120 attempts 3 within 60         ! Brute-force protection

line console 0
 password <console_password>
 login
 logging synchronous
 exec-timeout 5 0
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
exit

interface g0/0/0
 description LAN to SW1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
```

## Switch Management Address
```ios
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.99.1                  ! Layer 2 switch only
```

## Layer 3 Switch Basics
```ios
ip routing                                       ! Turn the switch into a router
interface g1/0/1
 no switchport                                   ! Make the port a routed port
 ip address 10.0.0.1 255.255.255.252
```

## Saving, Viewing & Resetting
| Command | Purpose |
|---|---|
| `copy running-config startup-config` | Save the config (or `write memory`) |
| `show running-config` | Active configuration (RAM) |
| `show startup-config` | Saved configuration (NVRAM) |
| `erase startup-config` | Wipe the saved config |
| `reload` | Reboot (answer `no` to save prompt if you want to discard changes) |
| `show version` | IOS version, uptime, hardware, config register |
| `show flash:` | Files in flash (IOS images, `vlan.dat`) |
| `show clock` | Current device time |
| `show users` | Who is logged in |
| `show interfaces description` | Interface descriptions at a glance |
| `show inventory` | Hardware model and serial numbers |

**Factory reset a switch:** `erase startup-config`, `delete flash:vlan.dat`, then `reload`.
