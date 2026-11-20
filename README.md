# Active Directory Security Home Lab

An enterprise-style Active Directory home lab built to develop practical
experience in Windows domain administration, penetration testing, attack
detection, and defensive hardening.

The environment consists of a Windows Server 2025 Domain Controller,
a domain-joined Windows 11 workstation, and a Kali Linux attacker machine
running on an isolated VMware network.

## Lab Architecture

| System | Operating System | Role | IP Address |
|---|---|---|---|
| DC01 | Windows Server 2025 | Domain Controller / DNS | 192.168.10.10 |
| CLIENT01 | Windows 11 | Domain Workstation | 192.168.10.20 |
| KALI01 | Kali Linux | Security Testing | 192.168.10.30 |

**Domain:** `lab.local`  
**Network:** `192.168.10.0/24`  
**Virtualization:** VMware Workstation Pro  
**Network Type:** Isolated host-only network (VMnet2)

## Project Objectives

- Build and administer a Windows Active Directory domain
- Configure users, groups, OUs, DNS, and Group Policy
- Perform Active Directory reconnaissance and enumeration
- Simulate common Active Directory attacks
- Investigate attack activity through Windows logging
- Apply defensive controls and hardening
- Retest attacks to validate mitigations

## Technologies

- Windows Server 2025
- Windows 11
- Kali Linux
- Active Directory Domain Services
- Group Policy
- PowerShell
- VMware Workstation Pro

## Security Tools

- Nmap
- Impacket
- Kerbrute
- BloodHound / SharpHound
- NetExec
- Wireshark
- Hashcat
- John the Ripper

## Network Architecture

                  VMnet2
              192.168.10.0/24
                      |
          +-----------+-----------+
          |           |           |
        DC01       CLIENT01      KALI01
     .10 / AD DS      .20          .30
        DNS        Windows 11     Kali
                                   |
                              Security Testing

## Active Directory Enumeration

Initial reconnaissance from KALI01 identified the Domain Controller and
the primary services associated with Active Directory.

### Service Discovery

Nmap identified services including:

| Port | Service | Purpose |
|---|---|---|
| 53 | DNS | Active Directory DNS |
| 88 | Kerberos | Domain authentication |
| 135 | MSRPC | Windows RPC |
| 389 | LDAP | Active Directory directory service |
| 445 | SMB | Windows file/service sharing |
| 464 | Kerberos | Password operations |
| 636 | LDAPS | LDAP over TLS |
| 3268 | Global Catalog | AD Global Catalog |

Service enumeration identified the host as `DC01` and the Active
Directory domain as `lab.local`.

### SMB Enumeration

SMB enumeration determined that:

- SMBv1 was not enabled
- SMB 2.x and 3.x were supported
- SMB signing was enabled and required
- Anonymous RPC domain enumeration was restricted

Authenticated SMB enumeration using a standard domain user exposed
standard domain shares including `NETLOGON` and `SYSVOL`.

### LDAP Enumeration

Anonymous LDAP directory searches were rejected because an authenticated
bind was required.

RootDSE queries remained available and revealed the domain naming
contexts, including:

`DC=lab,DC=local`

### Kerberos User Enumeration

Kerberos enumeration identified valid domain accounts, including:

- `administrator@lab.local`
- `sarahmiller@lab.local`

### AS-REP Roasting

The discovered accounts were tested for AS-REP roasting.

Both accounts required Kerberos pre-authentication and therefore were
not vulnerable.

This unsuccessful attack is retained as part of the assessment because
it demonstrates the effect of correctly configured Kerberos
pre-authentication.

## Attack Path

KALI01
   |
   v
DC01 discovered
   |
   v
lab.local identified
   |
   v
Kerberos user enumeration
   |
   +-- administrator
   |
   +-- sarahmiller
           |
           v
    AS-REP Roasting
           |
           X
    Pre-authentication required

## Attack Simulation

Work in progress.

Planned scenarios include:

- Kerberoasting
- Password attacks
- Credential attacks
- Privilege escalation
- Lateral movement
- Active Directory ACL abuse

## Detection & Monitoring

Work in progress.

The lab will use Windows auditing, Event Viewer, and PowerShell logging
to correlate simulated attacks with defensive telemetry.

## Hardening

Following attack simulation, discovered weaknesses will be remediated
and the attacks repeated to verify that the mitigations are effective.

## Disclaimer

This project was created in an isolated virtual lab for educational and
authorized security testing purposes.
