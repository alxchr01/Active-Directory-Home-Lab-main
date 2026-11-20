# Group Policy Configuration

## Overview

Group Policy was configured to centrally enforce security settings across the
`lab.local` domain.

The objective was to establish a basic security baseline for domain users and
workstations before beginning penetration testing. Policies were configured
on the Domain Controller and subsequently applied to CLIENT01.

The configuration focused on authentication security, account protection,
Windows security controls, and auditing.

## Group Policy Management

Group Policy was administered from DC01 using the Group Policy Management
Console (GPMC).

Policies were linked to the appropriate domain or Organizational Units so
that settings could be centrally applied to users and computers within the
Active Directory environment.

This provides centralized control over security configuration rather than
requiring each workstation to be configured individually.

## Password Policy

Domain password policies were configured to establish minimum requirements
for user credentials.

The policy provides centralized control over settings such as:

- Password complexity requirements
- Minimum password length
- Password history
- Minimum and maximum password age

These controls reduce the likelihood of weak credentials being used within
the domain and provide a baseline for later password-security testing.

## Account Lockout Policy

An account lockout policy was configured to limit repeated authentication
attempts against domain accounts.

The configured policy included:

| Setting | Value |
|---|---|
| Account lockout threshold | 10 failed attempts |
| Account lockout duration | 10 minutes |
| Reset account lockout counter | 10 minutes |

After ten unsuccessful authentication attempts, the affected account is
temporarily locked.

This configuration limits repeated online password guessing while allowing
accounts to recover automatically after the configured lockout period.

The policy is particularly relevant to later security testing because
password attacks must account for the domain's lockout threshold.

## Windows Defender

Microsoft Defender was retained as part of the Windows security baseline.

Defender provides endpoint protection against malicious files and other
potentially unwanted activity on Windows systems within the lab.

The defensive configuration is retained during later attack simulation so
that the interaction between offensive techniques and endpoint security
controls can be observed.

## Windows Firewall

Windows Firewall was enabled on the Windows systems.

During initial network testing, CLIENT01 did not respond to ICMP requests
from KALI01 and several Windows services appeared as filtered during network
scanning.

This demonstrated that network connectivity alone does not imply that every
service on a domain workstation is remotely accessible.

Firewall behavior is considered during later reconnaissance and attack
simulation.

## Audit Policy

Windows auditing provides visibility into security-relevant activity within
the domain.

Audit configuration forms the foundation for later investigation of events
such as:

- Successful authentication
- Failed authentication
- Account lockouts
- Account and group changes
- Privileged activity
- Process execution
- Kerberos authentication activity

The resulting Windows Event Logs are examined during the detection and
monitoring phase of the project.

## Policy Application

After configuring Group Policy on DC01, policies were applied to the
domain-joined CLIENT01 workstation.

Policy processing can be manually refreshed using:

```powershell
gpupdate /force
