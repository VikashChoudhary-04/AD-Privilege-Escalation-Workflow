# Delegation Abuse

## Purpose

Kerberos delegation allows a service or computer to act on behalf of a user when accessing another service.

Delegation is a legitimate Active Directory feature, but overly broad or incorrectly configured delegation can create privilege escalation paths.

This file focuses on identifying, analyzing, validating, and documenting **delegation-based privilege escalation paths** during an authorized security assessment.

The core workflow is:

```text
Enumerate Delegation
      ↓
Identify Delegated Principal
      ↓
Identify Delegation Type
      ↓
Identify Allowed Target
      ↓
Identify Protected Users / Services
      ↓
Map Delegation Relationship
      ↓
Assess Security Impact
      ↓
Validate Authorized Path
      ↓
Document Evidence
```

---

## Position in the Workflow

Delegation abuse belongs in the **Privilege Escalation Paths** phase.

The previous files examined:

* Group membership control
* ACL-based object control
* GPO control

Delegation introduces another important question:

> Can one account or computer authenticate to another service on behalf of a user?

The answer depends on the configured delegation type and the specific objects involved.

---

## Kerberos Delegation Fundamentals

Kerberos normally authenticates a user to a service using service tickets.

Delegation changes this relationship by allowing a service to obtain or use Kerberos authentication material in a way that permits access to another service on behalf of a user.

Simplified:

```text id="5plh8f"
User
  ↓
Service A
  ↓
Kerberos Delegation
  ↓
Service B
```

The security impact depends on:

* Delegation type
* Delegated account
* Source computer/service
* Target service
* User identity
* Target privileges
* Whether delegation is constrained
* Whether the target account is protected

---

## Delegation Types

The main delegation models to understand are:

1. Unconstrained Delegation
2. Constrained Delegation
3. Resource-Based Constrained Delegation (RBCD)

They should be analyzed separately because their control models are different.

---

# 1. Unconstrained Delegation

Unconstrained delegation allows a computer or service account to receive Kerberos authentication material that can potentially be used when accessing other services.

This creates a broad trust relationship.

Conceptually:

```text id="q5qjnp"
User Authentication
      ↓
Delegated Computer
      ↓
Kerberos Credentials
      ↓
Other Services
```

The security impact can be significant when the delegated computer is trusted with authentication from privileged users.

---

## 1.1 Identify Unconstrained Delegation

Computer objects can be inspected for the relevant delegation configuration.

PowerShell:

```powershell id="f3ozm7"
Get-ADComputer -Filter * -Properties TrustedForDelegation |
    Where-Object {$_.TrustedForDelegation -eq $true}
```

Also inspect service accounts where relevant.

The important attribute is:

```text
TrustedForDelegation
```

---

## 1.2 Identify Delegated Computers

Record:

* Computer name
* Operating system
* OU
* IP address
* Domain
* Service role
* Delegation configuration

Then determine which users may authenticate to the computer.

A domain controller or other highly privileged system should be treated differently from an ordinary workstation.

---

## 1.3 Assess Privileged User Exposure

The important question is not simply:

> "Does this computer have unconstrained delegation?"

Instead determine:

> "Can privileged users authenticate to this delegated computer?"

Potentially relevant accounts include:

* Domain administrators
* Enterprise administrators
* Server administrators
* Service administrators
* Other privileged groups

Membership and authentication paths should be verified rather than assumed.

---

## 1.4 Security Impact

A delegated system may become a high-value target when privileged authentication occurs there.

The relationship can be represented as:

```text id="t4kgga"
Privileged User
      ↓
Authentication
      ↓
Unconstrained Delegated Computer
      ↓
Delegation Material
      ↓
Potential Access to Other Services
```

The exact impact must be validated in the authorized environment.

---

# 2. Constrained Delegation

Constrained delegation restricts a service to specific services to which it may delegate authentication.

This is narrower than unconstrained delegation.

Conceptually:

```text id="2t9v5v"
Service Account
      ↓
Constrained Delegation
      ↓
Allowed Service
```

The security assessment should identify exactly which services are permitted.

---

## 2.1 Identify Constrained Delegation

Useful PowerShell:

```powershell id="f3q5cv"
Get-ADObject -LDAPFilter "(|(msDS-AllowedToDelegateTo=*)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" `
    -Properties msDS-AllowedToDelegateTo,msDS-AllowedToActOnBehalfOfOtherIdentity
```

For computer objects:

```powershell id="qf4e7u"
Get-ADComputer -Filter * `
    -Properties TrustedToAuthForDelegation,msDS-AllowedToDelegateTo
```

For users:

```powershell id="m4v9bw"
Get-ADUser -Filter * `
    -Properties TrustedToAuthForDelegation,msDS-AllowedToDelegateTo
```

These properties should be interpreted according to the object type and delegation configuration.

---

## 2.2 `msDS-AllowedToDelegateTo`

This attribute identifies service principal names to which constrained delegation is permitted.

Example:

```text id="qz9a4v"
Service Account
      ↓
msDS-AllowedToDelegateTo
      ↓
HTTP/server01.example.local
```

The actual SPNs should be recorded during enumeration.

---

## 2.3 Identify the Target Service

For every delegation relationship, identify:

* Source account
* Source computer
* Source service
* Target SPN
* Target computer
* Target service
* Target account
* Privilege level

Example:

```text id="a8j8p1"
WEB01$
   ↓
Delegation
   ↓
MSSQLSvc/DB01
   ↓
DB01
```

This provides the context required for impact analysis.

---

## 2.4 Protocol Transition

Some constrained delegation configurations allow protocol transition.

This can involve the:

```text
TrustedToAuthForDelegation
```

property.

Protocol transition changes how authentication can be established before delegation occurs.

Do not treat the presence of this property as a complete attack path by itself. Validate:

* Source account
* Delegation configuration
* Allowed SPNs
* Target service
* User context
* Effective permissions

---

## 2.5 Constrained Delegation Attack-Path Model

A simplified model is:

```text id="5g8k2m"
Compromised / Controlled Principal
          ↓
Delegation Rights
          ↓
Allowed Service
          ↓
Target Server
          ↓
Service Access
```

The security impact depends heavily on the identity under which the delegated authentication is performed.

---

# 3. Resource-Based Constrained Delegation

Resource-Based Constrained Delegation (RBCD) changes the control model.

Instead of the source service defining which targets it can delegate to, the **target resource** specifies which principals are trusted to act on behalf of users.

The important attribute is:

```text
msDS-AllowedToActOnBehalfOfOtherIdentity
```

Conceptually:

```text id="k9h7xe"
Target Computer
      ↓
msDS-AllowedToActOnBehalfOfOtherIdentity
      ↓
Trusted Principal
```

---

## 3.1 Identify RBCD Configuration

Computer objects can be queried for the attribute:

```powershell id="x9n4h2"
Get-ADComputer -Filter * `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
```

You can identify objects where the attribute is populated:

```powershell id="5o6n0y"
Get-ADComputer -Filter * `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity |
    Where-Object {
        $_.'msDS-AllowedToActOnBehalfOfOtherIdentity'
    }
```

---

## 3.2 Identify the Trusted Principal

The attribute contains security descriptor information.

The assessment should determine:

* Which principal is trusted
* Whether it is a computer account
* Whether it is a service account
* Who controls that principal
* What privileges the target computer provides

The relationship is:

```text id="d5e1xw"
Trusted Principal
      ↓
RBCD Permission
      ↓
Target Computer
```

---

## 3.3 RBCD and ACL Abuse

RBCD is closely connected to ACL abuse.

For example:

```text id="8x4y7c"
User
 ↓
Write Permission on Computer Object
 ↓
RBCD Attribute
 ↓
Trusted Principal
 ↓
Target Computer
```

Therefore, when analyzing computer ACLs, check whether security-relevant delegation attributes can be modified.

Do not assume that having generic object control automatically means a complete RBCD path. The exact permissions and environment must be verified.

---

# 4. Delegation Enumeration Workflow

Use a structured process.

```text id="9f6n1c"
1. Enumerate Users
        ↓
2. Enumerate Computers
        ↓
3. Enumerate SPNs
        ↓
4. Identify Delegation Configuration
        ↓
5. Identify Source Principal
        ↓
6. Identify Target Service
        ↓
7. Identify Target Computer
        ↓
8. Map Privileges
        ↓
9. Validate Path
```

---

# 5. Identify Delegated Accounts

Delegation may involve:

* Computer accounts
* User service accounts
* Managed service accounts
* Group Managed Service Accounts
* Application service identities

Record:

* Account name
* Account type
* SPNs
* Delegation attributes
* Group memberships
* ACLs
* Owner
* Service role

---

# 6. Delegation and SPNs

SPNs are central to Kerberos service authentication.

Useful enumeration:

```powershell id="h8f2dk"
setspn -Q */*
```

For a specific account:

```powershell id="8o6j7s"
setspn -L svc-web
```

For a computer:

```powershell id="7c2k5z"
setspn -L WEB01$
```

Correlate SPNs with delegation configuration.

---

# 7. Delegation and Privilege Levels

Delegation becomes more security-sensitive when the target service provides significant privileges.

Consider:

```text id="h3q4tc"
Delegated Identity
      ↓
Target Service
      ↓
Target Account
      ↓
Effective Privilege
```

For example, access to a service running under a highly privileged account may have a different impact from access to a low-privileged application service.

Do not infer privilege solely from the hostname or service name.

---

# 8. Delegation and Protected Accounts

Some accounts are specifically protected against delegation.

Relevant controls include:

* Protected Users group
* `Account is sensitive and cannot be delegated`
* Other domain security controls

When evaluating a path, determine whether the user can actually be delegated.

---

# 9. Protected Users

Membership in the **Protected Users** group changes several authentication behaviors and can prevent certain delegation scenarios.

Check relevant membership:

```powershell id="q1x8yn"
Get-ADGroupMember "Protected Users"
```

Also inspect the target account's properties.

A delegation path involving a protected account should be validated against the actual domain configuration rather than assumed to work.

---

# 10. Account Sensitivity

A user account may have the:

```text
Account is sensitive and cannot be delegated
```

setting.

This is an important control when evaluating whether Kerberos delegation can involve that account.

The assessment should record this property when relevant.

---

# 11. Delegation and Domain Controllers

Domain controllers require special attention because they are highly privileged systems.

Identify:

* Domain controllers
* Delegation settings
* Authentication exposure
* Service accounts
* Administrative logons

A delegated service on a domain controller should be carefully reviewed within the authorized scope.

---

# 12. Delegation and Service Accounts

Service accounts commonly appear in delegation configurations.

Review:

* SPNs
* Group membership
* Password management
* Delegation rights
* Target services
* ACLs
* Managed account status

Correlate delegation with the credential findings from Section 04.

For example:

```text id="p8n2av"
Service Account
      ↓
SPN
      ↓
Kerberos Delegation
      ↓
Target Service
      ↓
Privileged Resource
```

---

# 13. Delegation and ACL Correlation

Delegation should never be analyzed in isolation.

Correlate:

* GenericAll
* GenericWrite
* WriteDacl
* WriteOwner
* WriteProperty
* Computer object permissions
* Service account permissions

Example:

```text id="8h7g4s"
User
 ↓
Computer Object Control
 ↓
Delegation Attribute
 ↓
RBCD Configuration
 ↓
Target Computer
```

This is particularly important when constructing multi-step attack paths.

---

# 14. Delegation and Group Membership

The effective privilege of a delegated account can depend on group membership.

For every relevant account, inspect:

```powershell id="x2b7r9"
Get-ADPrincipalGroupMembership svc-web
```

Consider:

* Direct groups
* Nested groups
* Administrative groups
* Resource permissions
* Domain-wide privileges

---

# 15. Attack Path Analysis

A delegation path should be expressed as verified relationships.

### Unconstrained Delegation

```text id="1k9v4a"
Privileged User
      ↓
Authenticates to
      ↓
Delegated Computer
      ↓
Delegation Exposure
      ↓
Potential Access to Services
```

### Constrained Delegation

```text id="3m5p7r"
Controlled Service
      ↓
Delegation Configuration
      ↓
Allowed SPN
      ↓
Target Service
      ↓
Effective Access
```

### RBCD

```text id="7c6w2e"
Controlled Principal
      ↓
Trusted by Target
      ↓
RBCD Configuration
      ↓
Target Computer
      ↓
Effective Service Access
```

---

# 16. Validate Delegation Paths

Use the following validation chain:

```text id="4j8n6q"
Delegation Found
      ↓
Source Confirmed
      ↓
Delegation Type Confirmed
      ↓
Target Confirmed
      ↓
Allowed SPN / RBCD Relationship Confirmed
      ↓
User Delegation Restrictions Checked
      ↓
Effective Privilege Confirmed
      ↓
Evidence Collected
```

The goal is to distinguish:

```text
Configured Delegation
```

from:

```text
Confirmed Security-Relevant Delegation Path
```

---

# 17. Evidence Collection

Record:

| Field             | Description                                |
| ----------------- | ------------------------------------------ |
| Source            | Delegated user/computer/service            |
| Type              | Unconstrained/constrained/RBCD             |
| Attribute         | Relevant delegation attribute              |
| SPN               | Relevant service principal                 |
| Target            | Target service/computer                    |
| Trusted Principal | RBCD trusted principal where applicable    |
| User              | Relevant delegated identity                |
| Privilege         | Effective target privilege                 |
| Restrictions      | Protected/sensitive account controls       |
| Validation        | Verification performed                     |
| Evidence          | Command output, screenshots, or graph data |

Avoid storing unnecessary credentials or authentication material.

---

# 18. Common False Positives

### Delegation Exists but No Sensitive Target

The delegation relationship may point only to a low-impact service.

### Protected Account

The intended user may not be delegable.

### Sensitive Account Setting

The account may explicitly prevent delegation.

### Wrong Service

An SPN may exist but not represent the expected target service.

### Disabled Account

A source or target account may be disabled.

### Unreachable Target

The configured service may no longer exist or be operational.

### Permission Does Not Exist

An assumed RBCD or delegation path may lack the necessary controlling permission.

---

# 19. Common Mistakes

Avoid:

* Treating every delegation configuration as exploitable
* Confusing constrained delegation with unconstrained delegation
* Ignoring RBCD
* Ignoring SPNs
* Ignoring Protected Users
* Ignoring the sensitive-account setting
* Ignoring target service privileges
* Assuming computer compromise automatically provides domain-level privilege
* Treating delegation configuration as proof of successful privilege escalation
* Modifying delegation settings without authorization

---

# 20. Assessment Checklist

### Enumeration

* [ ] Enumerate computers
* [ ] Enumerate service accounts
* [ ] Enumerate SPNs
* [ ] Identify unconstrained delegation
* [ ] Identify constrained delegation
* [ ] Identify RBCD
* [ ] Record relevant delegation attributes

### Source Analysis

* [ ] Identify source account
* [ ] Identify source computer
* [ ] Identify service
* [ ] Identify owner
* [ ] Review group membership
* [ ] Review ACLs

### Target Analysis

* [ ] Identify target SPN
* [ ] Identify target computer
* [ ] Identify target service
* [ ] Identify service account
* [ ] Determine effective privilege

### Restrictions

* [ ] Check Protected Users
* [ ] Check sensitive-account setting
* [ ] Check account status
* [ ] Check delegation restrictions

### Validation

* [ ] Confirm delegation type
* [ ] Confirm source
* [ ] Confirm target
* [ ] Confirm SPN
* [ ] Confirm effective relationship
* [ ] Confirm resulting access
* [ ] Collect evidence

---

## Transition

Once delegation-based paths have been analyzed, continue to:

**`05-Privilege-Escalation-Paths/RBCD.md`**

The next file focuses specifically on **Resource-Based Constrained Delegation**, including its relationship with computer-object permissions and attack-path analysis.
