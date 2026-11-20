# 06 - Active Directory Attack Simulation

## Objective

The objective of this phase is to perform controlled attack simulations against the Active Directory environment created in the previous stages of the lab.

The attacks are performed exclusively within the isolated VMware lab environment. The purpose is to understand how common Active Directory weaknesses can be discovered and exploited, evaluate their potential impact, and later identify appropriate detection and mitigation techniques.

Each attack follows the same general methodology:

**Configuration → Attack Hypothesis → Exploitation → Impact → Evidence**

---

## Lab Environment

The attack environment consists of:

- **DC01** - Windows Server Domain Controller
- **CLIENT01** - Windows 11 domain-joined workstation
- **KALI01** - Kali Linux attacker machine
- **Domain:** `lab.local`
- **Network:** Isolated VMware VMnet2 network

KALI01 represents an attacker who has obtained access to the internal corporate network.

---

# 1. Password Attacks

## Configuration

Several domain user accounts were created during the Active Directory configuration phase.

These accounts provide a realistic environment for testing authentication-related attacks.

## Attack Hypothesis

If domain accounts use weak or predictable passwords, an attacker with network access may be able to discover valid credentials through password-based attacks.

## Testing

Usernames discovered during the enumeration phase can be used to test authentication against exposed domain services.

The objective is to determine whether any accounts use weak credentials without generating excessive authentication attempts.

## Impact

Successful credential discovery could allow an attacker to authenticate as a legitimate domain user.

This could provide an initial foothold within the Active Directory environment and enable further enumeration or privilege escalation.

## Evidence

Evidence collected during testing should include:

- Identified usernames
- Authentication attempts
- Successful or unsuccessful login results
- Relevant Windows authentication events

---

# 2. AS-REP Roasting

## Configuration

AS-REP Roasting targets Active Directory accounts that have Kerberos pre-authentication disabled.

## Attack Hypothesis

If a domain account does not require Kerberos pre-authentication, an unauthenticated attacker may be able to request authentication material for that account.

The resulting material can potentially be attacked offline.

## Enumeration

The domain was queried to identify accounts that may have Kerberos pre-authentication disabled.

The following accounts were discovered during Kerberos user enumeration:

- `sarahmiller`
- `administrator`

## Result

AS-REP Roasting was tested against the discovered accounts.

No accounts were found with Kerberos pre-authentication disabled.

Therefore, the attack was unsuccessful.

## Security Significance

This represents a secure configuration.

Requiring Kerberos pre-authentication prevents unauthenticated attackers from obtaining AS-REP material that could be subjected to offline password attacks.

---

# 3. Kerberoasting

## Configuration

Kerberoasting targets domain accounts associated with Service Principal Names (SPNs).

Service accounts may use passwords that remain unchanged for long periods, making them potentially valuable targets.

## Attack Hypothesis

If a service account has an SPN configured, an authenticated domain user may be able to request a Kerberos service ticket associated with that account.

The ticket can then potentially be subjected to offline password analysis.

## Testing

The Active Directory environment will be queried for accounts associated with SPNs.

Any discovered service accounts will be evaluated to determine whether Kerberos service tickets can be obtained.

## Impact

If the password of a privileged service account is recovered, the attacker could potentially gain access to additional systems or privileges within the domain.

## Evidence

Document:

- Discovered SPNs
- Associated service accounts
- Kerberos ticket requests
- Offline password-testing results
- Privileges associated with affected accounts

---

# 4. Credential Attacks

## Configuration

Windows systems may temporarily store authentication material in memory or expose credentials through insecure configuration.

## Attack Hypothesis

If an attacker compromises a domain-connected machine, locally available authentication material may allow access to additional accounts or systems.

## Testing

Credential exposure will be evaluated on compromised lab systems.

Testing will focus on identifying authentication material available to the current user or system context.

## Impact

Recovered credentials could allow an attacker to impersonate another user and potentially move deeper into the domain.

Privileged credentials would represent a particularly significant security risk.

## Evidence

Document:

- Compromised host
- Account context
- Credential material discovered
- Privileges associated with affected accounts
- Subsequent access obtained

Sensitive credentials should not be committed to the public repository.

---

# 5. Privilege Escalation

## Configuration

Domain users, groups, services and permissions were configured during the Active Directory setup phase.

## Attack Hypothesis

Misconfigured permissions or excessive privileges may allow a standard domain user to obtain permissions beyond those originally assigned.

## Testing

Potential privilege escalation paths identified during enumeration will be investigated.

Particular attention will be given to:

- Group memberships
- Administrative permissions
- Service accounts
- Active Directory permissions
- Misconfigured systems

## Impact

Successful privilege escalation could allow a standard domain user to gain administrative control over a workstation, server or the wider domain.

## Evidence

Document the complete privilege escalation path:

`Initial User → Misconfiguration → Elevated Account/Privilege`

---

# 6. Lateral Movement

## Configuration

CLIENT01 and DC01 exist within the same Active Directory environment.

Domain credentials may provide access to multiple systems depending on account permissions.

## Attack Hypothesis

If credentials obtained from one system are valid elsewhere in the domain, an attacker may be able to move from the initially compromised host to another machine.

## Testing

Recovered credentials and discovered permissions will be evaluated against other lab systems.

The objective is to determine whether compromise of one account or workstation can lead to access to additional systems.

## Impact

Lateral movement can transform the compromise of a single workstation into a broader domain compromise.

The severity depends on the privileges and systems accessible using the compromised credentials.

## Evidence

Document the movement path:

`KALI01 → Initial Account → CLIENT01 → Additional Credentials/Privileges → Target System`

---

# 7. Active Directory ACL / Permission Abuse

## Configuration

Active Directory uses Access Control Lists (ACLs) to determine which users and groups can modify domain objects.

## Attack Hypothesis

If a low-privileged user has excessive permissions over another user, group or organizational unit, those permissions may provide an unintended privilege escalation path.

## Testing

Permissions identified during Active Directory enumeration will be reviewed for potentially dangerous relationships.

Examples include permissions allowing an account to:

- Modify another user
- Reset another user's password
- Modify group membership
- Control a privileged group
- Modify an Active Directory object

## Impact

Improper ACL configuration can allow apparently low-privileged accounts to indirectly obtain administrative privileges.

These relationships may not be obvious when examining group membership alone.

## Evidence

BloodHound and other enumeration results should be used to document discovered permission relationships and potential attack paths.

---

# Attack Path Documentation

Successful attacks should be represented as complete attack chains rather than isolated vulnerabilities.

For example:

`Unauthenticated Attacker`
↓
`Internal Network Access`
↓
`Domain Enumeration`
↓
`Valid Domain User`
↓
`Credential or Permission Weakness`
↓
`Privilege Escalation`
↓
`Lateral Movement`
↓
`Privileged Domain Access`

This makes it possible to demonstrate how several individually small weaknesses can combine into a significant security risk.

---

# Results

| Attack | Result | Security Impact |
|---|---|---|
| Password Attacks | Pending | Credential compromise |
| AS-REP Roasting | Not Vulnerable | Kerberos pre-authentication enabled |
| Kerberoasting | Pending | Service account compromise |
| Credential Attacks | Pending | Credential exposure |
| Privilege Escalation | Pending | Elevated privileges |
| Lateral Movement | Pending | Access to additional systems |
| ACL Abuse | Pending | Active Directory privilege escalation |

The table will be updated as each attack simulation is completed.

---

# Next Steps

Following the attack simulation phase, the lab will move into detection and monitoring.

The next phase will focus on:

- Advanced Windows auditing
- Windows Event Logs
- PowerShell logging
- Authentication monitoring
- Detection of simulated attacks
- Identification of relevant Windows Event IDs

The weaknesses identified during this phase will subsequently be remediated during the Active Directory hardening phase.
