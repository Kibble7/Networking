# Secure Remote Management (SSHv2 & VTY ACLs)

## SSHv2 Implementation Workflow
1. Configure hostname and domain name (`ip domain-name corp.local`).
2. Generate 2048-bit RSA keypair (`crypto key generate rsa general-keys modulus 2048`).
3. Create local user credentials (`username admin privilege 15 secret <LAB_PASSWORD>`).
4. Enforce SSHv2 (`ip ssh version 2`), restrict VTY lines to `transport input ssh` and `login local`, and apply a VTY `access-class` ACL restricting management access to the Management VLAN (`10.10.99.0/24`).