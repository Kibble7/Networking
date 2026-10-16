## File: `networking-portfolio/labs/security/device-hardening-and-ssh.md`


# Network Device Hardening & Secure Remote Access (SSHv2)

## Objective
This lab documents the configuration and verification process for hardening Cisco IOS routers and switches and replacing plaintext Telnet with encrypted SSHv2 management.

## Skills Demonstrated
- RSA cryptographic key generation and SSHv2 enforcement
- Local AAA user database creation with privilege levels and hashed secrets
- Plaintext password encryption (`service password-encryption`)
- Console and VTY timeout, login throttling, and legal banner configuration

## Configuration
```ios
! 1. Set Hostname, Domain Name, and Encrypted Privileged Secret
hostname R1
ip domain-name corp.local
enable secret <REDACTED>
service password-encryption
no ip domain-lookup

! 2. Create Local Admin Account and Generate 2048-Bit RSA Keys
username netadmin privilege 15 secret <LAB_PASSWORD>
crypto key generate rsa general-keys modulus 2048

! 3. Harden SSHv2 Parameters and Login Block Protection
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
login block-for 120 attempts 3 within 60
security passwords min-length 10

! 4. Configure Legal Warning Banner
banner motd #
******************************************************************
WARNING: Authorized Access Only. All activity is logged.
******************************************************************
#

! 5. Secure Console and VTY Lines (Disable Telnet)
line console 0
 login local
 exec-timeout 5 0
 logging synchronous

line vty 0 4
 login local
 transport input ssh
 exec-timeout 5 0
 logging synchronous

```

## Verification
- show ip ssh — Confirm SSH Enabled - version 2.0, authentication timeout 60 secs, and retries 3.

- show ssh — Inspect active encrypted SSHv2 administrator sessions.

- show running-config | include secret|password — Verify all credentials use Type 5/8/9 hashes (secret) or Type 7 obfuscation, with zero cleartext passwords exposed.

## Key Takeaways
This lab demonstrates baseline control-plane and management-plane hardening required before placing any enterprise router or switch into production.