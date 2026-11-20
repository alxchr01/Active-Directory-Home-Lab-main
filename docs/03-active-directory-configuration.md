# Active Directory Configuration

## Overview

After establishing the virtual network infrastructure, DC01 was configured
as the Domain Controller for the lab.

Active Directory Domain Services (AD DS) and DNS were deployed to create a
functional Windows domain environment. Organizational Units, users, groups,
and a domain-joined Windows 11 workstation were then configured to provide
a realistic environment for later security testing.

## Active Directory Domain Services

The Active Directory Domain Services role was installed on DC01 using
Windows Server Manager.

After installation, the server was promoted to a Domain Controller and a new
Active Directory forest was created.

| Setting | Value |
|---|---|
| Domain Controller | `DC01` |
| Forest | `lab.local` |
| Domain | `lab.local` |
| NetBIOS Domain | `LAB` |
| DNS Server | `DC01` |
| DC Address | `192.168.10.10` |

DNS was installed alongside Active Directory to provide name resolution and
service discovery for the domain.

## Active Directory Structure

Organizational Units (OUs) were created to organize domain objects and
represent the structure of a small enterprise environment.

The hierarchy separates users and other resources into logical administrative
units rather than placing all objects in the default Active Directory
containers.

This structure also provides a foundation for applying targeted Group Policy
settings later in the project.

## Domain Users

Multiple domain user accounts were created to represent employees within the
lab environment.

One of the standard user accounts used throughout the project is:

`LAB\sarahmiller`

The account represents an ordinary domain user and is later used for
authentication testing, Active Directory enumeration, and security
assessment from KALI01.

Administrative accounts remain separate from standard user accounts,
allowing differences in permissions and access to be observed during testing.

## Security Groups

Security groups were created to organize users according to their roles and
provide group-based access control.

Users were assigned to the appropriate groups rather than being granted
permissions individually where possible.

This reflects the role-based permission model commonly used in Active
Directory environments and provides additional relationships for later
security analysis.

## DNS Configuration

Active Directory-integrated DNS was configured on DC01.

CLIENT01 was configured to use the Domain Controller as its DNS server:

`192.168.10.10`

Using the Domain Controller for DNS allows the workstation to discover
Active Directory services, including LDAP and Kerberos, through the DNS
records registered by the domain.

The domain could subsequently be resolved as:

`lab.local`

## Joining CLIENT01 to the Domain

After configuring the Windows 11 workstation to use DC01 for DNS, CLIENT01
was joined to the `lab.local` domain.

Domain administrator credentials were used to authorize the join operation.

After restarting CLIENT01, the computer appeared as a domain computer within
Active Directory.

The workstation could then authenticate users against DC01 rather than relying
only on local Windows accounts.

## Domain Authentication Test

Domain authentication was verified by signing into CLIENT01 using a standard
domain account.

For example:

`LAB\sarahmiller`

Successful authentication demonstrated that:

- CLIENT01 could locate the Domain Controller
- DNS resolution was functioning
- Kerberos/domain authentication was operational
- The workstation had successfully joined `lab.local`
- Domain user accounts could authenticate from the client

## Result

At the end of this stage, the lab contained a functioning Active Directory
environment with:

- A Windows Server 2025 Domain Controller
- The `lab.local` Active Directory forest and domain
- Active Directory-integrated DNS
- Organizational Units
- Multiple domain users
- Security groups
- Administrative and standard user accounts
- A domain-joined Windows 11 workstation
- Successful domain-user authentication from CLIENT01

The completed domain provided the foundation for Group Policy configuration
and subsequent Active Directory security testing.
