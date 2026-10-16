# Layer 2 Port Security

## Violation Modes
| Mode | Discards Offending Traffic | Sends Syslog / SNMP Trap | Increments Violation Counter | Shuts Port (`err-disabled`) |
|---|---|---|---|---|
| `protect` | Yes | No | No | No |
| `restrict` | Yes | Yes | Yes | No |
| `shutdown` (Default) | Yes | Yes | Yes | Yes |