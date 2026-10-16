# Network Device Hardening

## Control & Management Plane Checklist
- Use `enable secret` (scrypt/MD5 Type 5/8/9) instead of reversible `enable password`.
- Apply `service password-encryption` to obfuscate plaintext passwords in `running-config`.
- Configure legal warning banners (`banner motd #Authorized Access Only#`).
- Enforce `exec-timeout 5 0` and `logging synchronous` on Console and VTY lines.
- Disable unused services (`no ip http server`, `no cdp run` on external WAN ports, `no ip domain-lookup` to prevent CLI mistype stalls).