# Active Directory Reconnaissance and Enumeration

## Overview

Following deployment and configuration of the Active Directory environment,
KALI01 was used to perform reconnaissance and enumeration against the lab.

The objective was to identify the Domain Controller, discover exposed
services, determine the Active Directory domain, enumerate available
information, and identify potential attack vectors before attempting
exploitation.

Testing was initially performed without domain credentials. Authenticated
enumeration was introduced later using a standard domain user account.

## 1. Host Discovery

Initial testing confirmed connectivity between KALI01 and the Domain
Controller.

```bash
ping -c 4 192.168.10.10
```

DC01 responded at:

`192.168.10.10`

This established the Domain Controller as the primary target for subsequent
service enumeration.

---

## 2. Port and Service Discovery

Nmap was used to identify services exposed by DC01.

```bash
nmap -sC -sV 192.168.10.10
```

The scan identified several services associated with an Active Directory
Domain Controller.

| Port | Service | Purpose |
|---|---|---|
| 53 | DNS | Domain name resolution |
| 88 | Kerberos | Active Directory authentication |
| 135 | MSRPC | Microsoft RPC |
| 139 | NetBIOS | Windows networking |
| 389 | LDAP | Active Directory directory service |
| 445 | SMB | Windows file and resource sharing |
| 464 | Kerberos | Kerberos password operations |
| 636 | LDAPS | LDAP over TLS |
| 3268 | Global Catalog | Forest-wide LDAP queries |
| 3269 | Global Catalog SSL | Encrypted Global Catalog |

Service enumeration also identified:

- Hostname: `DC01`
- Domain: `lab.local`

The combination of Kerberos, LDAP, SMB, DNS, and Global Catalog services
confirmed that the host was operating as an Active Directory Domain
Controller.

---

## 3. DNS Enumeration

DNS was queried to verify the Active Directory domain and discover
domain-related records.

Active Directory relies heavily on DNS for locating services such as Domain
Controllers, Kerberos servers, and LDAP services.

Enumeration confirmed the use of:

`lab.local`

as the Active Directory domain.

This information was subsequently used during LDAP and Kerberos enumeration.

---

## 4. SMB Enumeration

SMB was examined to determine supported protocol versions and security
configuration.

Nmap SMB scripts were used against TCP port 445.

```bash
nmap --script smb-protocols,smb2-security-mode,smb2-time \
-p 445 192.168.10.10
```

The server supported modern SMB versions, including SMB 2.x and SMB 3.x.

SMBv1 was not advertised.

The scan also reported:

```text
Message signing enabled and required
```

### Security Observation

Requiring SMB signing protects the integrity of SMB communication and
restricts attack techniques that depend on a target accepting unsigned SMB
sessions.

The absence of SMBv1 also removes exposure to weaknesses associated with the
legacy protocol.

---

## 5. Anonymous SMB and RPC Enumeration

Anonymous access was tested to determine whether information about the domain
could be obtained without valid credentials.

SMB share enumeration was attempted using:

```bash
smbclient -L //192.168.10.10 -N
```

The server accepted the initial anonymous connection, but useful anonymous
share enumeration was not obtained.

RPC enumeration was subsequently tested:

```bash
rpcclient -U "" -N 192.168.10.10
```

Commands such as:

```text
enumdomusers
enumdomgroups
querydominfo
```

returned:

```text
NT_STATUS_ACCESS_DENIED
```

### Security Observation

Anonymous RPC domain enumeration was restricted.

An unauthenticated attacker could therefore not simply retrieve the domain
user and group lists through a null RPC session.

---

## 6. LDAP Enumeration

LDAP was tested to determine what Active Directory information could be
obtained without credentials.

An anonymous directory query was attempted against:

```text
DC=lab,DC=local
```

The Domain Controller returned an error indicating that a successful bind
was required.

Anonymous enumeration of directory objects was therefore restricted.

### RootDSE Enumeration

A RootDSE query was also performed:

```bash
ldapsearch -x \
-H ldap://192.168.10.10 \
-s base -b "" \
defaultNamingContext rootDomainNamingContext namingContexts
```

Unlike the directory query, the RootDSE request succeeded.

The response disclosed naming contexts including:

```text
defaultNamingContext: DC=lab,DC=local
rootDomainNamingContext: DC=lab,DC=local
```

Additional configuration, schema, and DNS naming contexts were also exposed.

### Security Observation

Although anonymous directory enumeration required authentication, basic
Active Directory metadata remained discoverable through RootDSE.

This provided sufficient information to determine the LDAP base DN and
confirm the structure of the domain.

---

## 7. Kerberos User Enumeration

With the domain name and Domain Controller identified, Kerberos was tested
for username enumeration.

Kerbrute was used with a candidate username list:

```bash
kerbrute userenum \
-d lab.local \
--dc 192.168.10.10 \
users.txt
```

The enumeration identified valid domain principals including:

```text
administrator@lab.local
sarahmiller@lab.local
```

### Security Observation

This demonstrated that valid domain usernames could be identified through
differences in Kerberos responses without first obtaining a valid domain
password.

The discovered usernames provided potential targets for subsequent
authentication testing.

---

## 8. AS-REP Roasting Assessment

The identified accounts were tested to determine whether Kerberos
pre-authentication had been disabled.

Impacket's `GetNPUsers` was used:

```bash
impacket-GetNPUsers lab.local/ \
-usersfile users.txt \
-dc-ip 192.168.10.10 \
-no-pass
```

Neither identified account was vulnerable.

The tool reported that the accounts did not have:

```text
UF_DONT_REQUIRE_PREAUTH
```

configured.

No AS-REP hashes were obtained.

### Security Observation

Kerberos pre-authentication was enabled for the tested accounts.

This prevented an unauthenticated attacker from requesting AS-REP material
for those accounts and performing an offline password-guessing attack
against it.

The unsuccessful attack was retained as an assessment result because it
demonstrates the effect of the existing security configuration.

---

## 9. Authenticated SMB Enumeration

After unauthenticated enumeration had been exhausted, a standard domain
account was used to examine what additional information was available to an
authenticated user.

Authentication was performed using:

`LAB\sarahmiller`

SMB enumeration identified standard Windows shares including:

```text
ADMIN$
C$
IPC$
NETLOGON
SYSVOL
```

The presence of a share did not imply that the standard user had unrestricted
access to it. Access remained controlled by the permissions associated with
the authenticated account.

`SYSVOL` and `NETLOGON` are expected domain resources and are relevant during
Active Directory assessments because they can contain domain-wide scripts,
policies, and configuration information.

---

## 10. Enumeration Summary

The reconnaissance phase established the following:

| Test | Result |
|---|---|
| Domain Controller discovered | `DC01` |
| Active Directory domain identified | `lab.local` |
| AD services identified | Successful |
| SMBv1 | Not advertised |
| SMB signing | Required |
| Anonymous RPC enumeration | Restricted |
| Anonymous LDAP directory enumeration | Restricted |
| RootDSE enumeration | Successful |
| Kerberos username enumeration | Successful |
| `administrator` identified | Yes |
| `sarahmiller` identified | Yes |
| AS-REP roastable accounts identified | No |
| Authenticated SMB enumeration | Successful |

## Enumeration Flow

```text
KALI01
   |
   v
192.168.10.10
   |
   v
DC01 discovered
   |
   +--------------------+
   |                    |
   v                    v
Services identified   lab.local identified
   |                    |
   +---------+----------+
             |
             v
    Anonymous Enumeration
             |
     +-------+-------+
     |               |
     v               v
    RPC             LDAP
  Restricted      Restricted
                     |
                     v
              RootDSE successful
                     |
                     v
            Kerberos Enumeration
                     |
             +-------+-------+
             |               |
             v               v
       administrator     sarahmiller
             |               |
             +-------+-------+
                     |
                     v
              AS-REP Roasting
                     |
                     X
              Pre-auth required
```

## Findings

The initial assessment did not identify an immediately exploitable
unauthenticated path to domain compromise.

Several defensive configurations limited common Active Directory attacks:

- SMB signing was required
- SMBv1 was not advertised
- Anonymous RPC enumeration was restricted
- Anonymous LDAP directory enumeration was restricted
- Kerberos pre-authentication was enabled for the tested accounts

At the same time, Kerberos responses allowed valid usernames to be
identified, and authenticated access provided additional visibility into
domain resources.

These findings establish a security baseline for the environment before
controlled vulnerabilities and attack scenarios are introduced during the
attack-simulation phase.

## Next Steps

The remaining enumeration stage will use BloodHound to map Active Directory
objects, group memberships, permissions, and potential privilege
relationships.

Following completion of Active Directory mapping, the project will proceed
to controlled attack simulation.
