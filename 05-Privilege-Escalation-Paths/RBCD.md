# Resource-Based Constrained Delegation (RBCD)

## Purpose

Resource-Based Constrained Delegation (RBCD) is a Kerberos delegation mechanism in which the **target resource** defines which principals are trusted to act on behalf of users.

In an Active Directory security assessment, RBCD is important because excessive control over a computer or service object can sometimes allow an attacker-controlled principal to become trusted by that resource.

The core workflow is:

```text
Identify Target
      ↓
Enumerate RBCD Configuration
      ↓
Identify Trusted Principal
      ↓
Identify Who Can Modify RBCD
      ↓
Validate Required Permissions
      ↓
Map Kerberos Delegation Path
      ↓
Determine Target Privilege
      ↓
Validate Authorized Path
      ↓
Document Evidence
```

---

## Position in the Workflow

RBCD is part of:

```text
05-Privilege-Escalation-Paths
```

It follows the general delegation analysis from:

**`Delegation-Abuse.md`**

RBCD deserves a dedicated workflow because its security model is different from traditional constrained delegation.

The central question is:

> Who can make a target resource trust a principal for delegated authentication, and what does that target allow?

---

## RBCD Fundamentals

The primary Active Directory attribute associated with RBCD is:

```text
msDS-AllowedToActOnBehalfOfOtherIdentity
```

This attribute is associated with the **target resource**.

Conceptually:

```text
Target Computer
      ↓
msDS-AllowedToActOnBehalfOfOtherIdentity
      ↓
Trusted Principal
```

This differs from traditional constrained delegation, where the source account specifies the services to which it can delegate.

---

## Traditional vs Resource-Based Delegation

### Traditional Constrained Delegation

```text
Source Account
      ↓
Defines Allowed Target Services
      ↓
Target Service
```

### RBCD

```text
Target Resource
      ↓
Defines Trusted Source Principal
      ↓
Source Principal
```

The distinction is important when analyzing permissions and attack paths.

---

## RBCD Attack-Path Model

A simplified RBCD path is:

```text
Controlled Principal
        ↓
Ability to Influence RBCD
        ↓
Target Computer
        ↓
RBCD Configuration
        ↓
Delegated Authentication
        ↓
Target Service
        ↓
Effective Privilege
```

Every step must be independently verified.

---

## 1. Identify Computer Objects

Start by enumerating computer accounts.

PowerShell:

```powershell
Get-ADComputer -Filter * -Properties DNSHostName,OperatingSystem
```

Useful information includes:

* Computer name
* DNS hostname
* Operating system
* OU
* Enabled status
* Group membership
* SPNs
* Delegation settings

---

## 2. Enumerate Existing RBCD Configuration

Query the RBCD attribute:

```powershell
Get-ADComputer -Filter * `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
```

To identify populated configurations:

```powershell
Get-ADComputer -Filter * `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity |
    Where-Object {
        $_.'msDS-AllowedToActOnBehalfOfOtherIdentity'
    }
```

The presence of this attribute does not automatically indicate a vulnerability.

You must determine whether the trusted principal and its controlling permissions are appropriate.

---

## 3. Inspect a Specific Computer

For a specific computer:

```powershell
Get-ADComputer "SERVER01" `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
```

Record:

* Computer name
* Distinguished Name
* RBCD configuration
* Owner
* ACL
* Group memberships
* SPNs
* Operating system
* Enabled state

---

## 4. Understand the RBCD Security Descriptor

The RBCD attribute contains security descriptor information defining which principals are trusted.

The important relationship is:

```text
Target Computer
      ↓
RBCD Security Descriptor
      ↓
Allowed Principal
```

The assessment goal is to translate the security descriptor into understandable relationships:

```text
Trusted Principal
      ↓
Can Act on Behalf of Users
      ↓
Against Target Resource
```

---

## 5. Identify the Trusted Principal

The trusted principal may be:

* Computer account
* User account
* Service account
* Other security principal

Determine:

* Principal name
* Object type
* Owner
* Group membership
* ACL
* Who controls the principal
* Whether the principal is authorized

This is critical because the security impact depends on whether an unauthorized party can control the trusted principal.

---

## 6. Identify Who Can Modify RBCD

This is one of the most important steps.

Ask:

> Who can modify the target computer object's RBCD configuration?

Review permissions on the computer object.

Potentially relevant permissions include:

* GenericAll
* GenericWrite
* WriteDacl
* WriteOwner
* WriteProperty
* Specific control over the RBCD attribute

Example:

```powershell
Get-Acl "AD:\CN=SERVER01,OU=Servers,DC=example,DC=local"
```

The exact distinguished name depends on the environment.

---

## 7. RBCD and GenericAll

GenericAll provides broad control over an AD object.

If a principal has GenericAll over a target computer, investigate whether that control permits security-relevant changes to the computer object's delegation configuration.

The relationship may be:

```text
User
 ↓
GenericAll
 ↓
Computer Object
 ↓
RBCD Configuration
```

Do not stop at identifying GenericAll.

Validate the exact resulting permission and impact.

---

## 8. RBCD and GenericWrite

GenericWrite can permit modification of multiple attributes.

However, GenericWrite should not automatically be interpreted as unrestricted control over every security-sensitive attribute.

Determine:

* Which attributes are writable
* Whether the RBCD attribute can actually be modified
* Whether additional permissions are required
* What resulting relationship is created

---

## 9. RBCD and WriteDacl

WriteDacl allows modification of the object's DACL.

This may create a chained path:

```text
User
 ↓
WriteDacl
 ↓
Computer Object
 ↓
Grant Required Permission
 ↓
RBCD Configuration
 ↓
Target Access
```

The important distinction is that WriteDacl may be an **indirect control path**, rather than the final privilege itself.

---

## 10. RBCD and WriteOwner

Ownership can affect control over an object's security descriptor.

A possible conceptual chain is:

```text
User
 ↓
WriteOwner
 ↓
Computer Object Ownership
 ↓
Security Descriptor Control
 ↓
Required Permission
 ↓
RBCD
```

Ownership should not be treated as immediate administrative access.

Validate the entire chain.

---

## 11. Computer Object Creation

Computer-account creation permissions can be relevant to RBCD attack paths.

Check whether a principal is allowed to create computer objects in the domain or within a particular OU.

A common permission of interest is:

```text
ms-DS-MachineAccountQuota
```

Query:

```powershell
Get-ADDomain -Properties ms-DS-MachineAccountQuota
```

This setting controls the default number of computer accounts that a non-administrative domain user can add to the domain.

Its presence alone does not establish an RBCD vulnerability.

The complete environment must be evaluated.

---

## 12. Machine Account Control

When assessing computer-account creation, determine:

* Who can create computers
* Where they can be created
* Whether the created computer is controlled by the assessed principal
* Whether the account is enabled
* What SPNs are associated with it
* Whether the resulting account can participate in the intended delegation path

This should be analyzed as an attack-path prerequisite rather than treated as automatic privilege escalation.

---

## 13. RBCD and ACL Enumeration

RBCD should be correlated directly with ACL analysis.

For every interesting computer object:

```text
Computer Object
      ↓
ACL Enumeration
      ↓
Who Can Modify Object?
      ↓
Which Attributes Can They Change?
      ↓
Can RBCD Be Influenced?
```

This makes RBCD a natural continuation of:

**`ACL-Abuse.md`**

---

## 14. RBCD and SPNs

Kerberos services are identified using SPNs.

Inspect computer SPNs:

```powershell
setspn -L SERVER01$
```

Inspect a service account:

```powershell
setspn -L svc-web
```

The target service should be identified before determining the security impact of an RBCD configuration.

---

## 15. Identify the Target Service

A computer can host multiple services.

Examples include:

* CIFS
* HTTP
* LDAP
* MSSQLSvc
* HOST
* RestrictedKrbHost

The relevant service must be identified accurately.

The path should therefore be represented as:

```text
Trusted Principal
      ↓
RBCD
      ↓
Target Computer
      ↓
Target Service
      ↓
Effective Access
```

---

## 16. Identify the Target Privilege

The presence of RBCD does not itself establish a particular privilege level.

Determine:

* Which account runs the target service
* What local privileges it has
* What domain permissions it has
* Which resources it can access
* Whether it belongs to privileged groups
* Whether the target is a domain controller

For example:

```text
Target Computer
      ↓
Target Service
      ↓
Service Account
      ↓
Group Membership
      ↓
Effective Privilege
```

---

## 17. RBCD on Domain Controllers

Domain controllers require special attention.

If an RBCD relationship involves a domain controller, determine:

* Target computer
* Trusted principal
* RBCD permissions
* Relevant SPNs
* Service account context
* Domain privileges
* Authentication restrictions

Do not classify a path solely because the target hostname appears to be a domain controller.

Validate the actual delegation relationship and resulting access.

---

## 18. Existing RBCD vs Vulnerable RBCD

These are different findings.

### Existing RBCD

```text
Target
 ↓
Configured Trusted Principal
```

This may be intentional.

### Potentially Unsafe RBCD

```text
Unauthorized Principal
 ↓
Can Control Trusted Principal
 ↓
Trusted by Target
```

### Confirmed Attack Path

```text
Controlled Principal
 ↓
RBCD Relationship
 ↓
Target Service
 ↓
Confirmed Security-Relevant Access
```

The third state requires validation.

---

## 19. RBCD Attack-Path Construction

Build the path from verified relationships.

Example:

```text
User A
   ↓
Controls Computer Account A
   ↓
Computer A Trusted by Computer B
   ↓
RBCD on Computer B
   ↓
Kerberos Delegation Relationship
   ↓
Service on Computer B
   ↓
Effective Privilege
```

Do not skip intermediate relationships.

---

## 20. BloodHound-Compatible Analysis

BloodHound-compatible collectors can help identify relationships involving:

* Computer objects
* ACLs
* Object control
* Delegation
* Group membership
* SPNs

Depending on the collector and version, graph relationships may expose RBCD-related paths.

Use the graph to discover candidates, then manually verify:

```text
Graph Relationship
      ↓
AD Object
      ↓
Exact Attribute / Permission
      ↓
Effective Configuration
      ↓
Security Impact
```

---

## 21. Validate RBCD Configuration

A complete validation process is:

```text
Target Identified
      ↓
RBCD Attribute Identified
      ↓
Trusted Principal Identified
      ↓
Principal Control Confirmed
      ↓
Target Service Identified
      ↓
Required Permissions Confirmed
      ↓
Delegation Relationship Confirmed
      ↓
Effective Access Confirmed
```

This prevents configuration-only findings from being reported as confirmed privilege escalation.

---

## 22. Controlled Validation

Validation should be performed only within the authorized environment.

Prefer the least invasive method capable of proving the relationship.

For example:

* Verify the ACE
* Verify the RBCD attribute
* Verify the trusted principal
* Verify the target SPN
* Verify the target service
* Verify the resulting authorized access

Avoid unnecessary changes to production computer objects.

---

## 23. Evidence Collection

Record:

| Field             | Description                                |
| ----------------- | ------------------------------------------ |
| Target            | Computer/resource                          |
| RBCD Attribute    | `msDS-AllowedToActOnBehalfOfOtherIdentity` |
| Trusted Principal | Principal permitted by RBCD                |
| Controller        | Principal controlling the trusted identity |
| Permission        | Permission enabling configuration          |
| SPN               | Relevant target service                    |
| Service           | Target service                             |
| Privilege         | Effective service privilege                |
| Validation        | Verification performed                     |
| Evidence          | Commands, output, screenshots, graph data  |

Do not store unnecessary credentials or authentication material.

---

## 24. Common False Positives

### Legitimate RBCD

RBCD may be intentionally configured for applications or services.

### Trusted Administrative Principal

The trusted principal may already be an authorized administrative identity.

### No Control Over Trusted Principal

A user may discover an RBCD relationship but have no ability to control the trusted principal.

### No Relevant Target Privilege

The target service may not provide meaningful additional access.

### Disabled Principal

The trusted account may be disabled.

### Protected User

Authentication restrictions may prevent the expected delegation behavior.

### Incorrect SPN

The assumed target service may not correspond to the actual configured SPN.

---

## 25. Common Mistakes

Avoid:

* Treating every populated RBCD attribute as vulnerable
* Confusing RBCD with traditional constrained delegation
* Ignoring the target object's ACL
* Ignoring control over the trusted principal
* Assuming GenericWrite always provides complete RBCD control
* Assuming GenericAll automatically proves the final attack path
* Ignoring SPNs
* Ignoring service-account privileges
* Ignoring Protected Users and account restrictions
* Assuming computer-account creation automatically creates privilege escalation
* Making unnecessary production changes

---

## 26. Assessment Checklist

### Target Enumeration

* [ ] Enumerate computer objects
* [ ] Identify domain controllers
* [ ] Identify important servers
* [ ] Enumerate SPNs
* [ ] Identify existing RBCD configuration

### RBCD Analysis

* [ ] Inspect `msDS-AllowedToActOnBehalfOfOtherIdentity`
* [ ] Identify trusted principal
* [ ] Identify target computer
* [ ] Identify target service
* [ ] Identify target SPN

### Permission Analysis

* [ ] Enumerate target ACL
* [ ] Identify GenericAll
* [ ] Identify GenericWrite
* [ ] Identify WriteDacl
* [ ] Identify WriteOwner
* [ ] Identify relevant attribute permissions
* [ ] Determine who controls the trusted principal

### Computer Account Analysis

* [ ] Check machine account creation controls
* [ ] Check `ms-DS-MachineAccountQuota`
* [ ] Identify who can create computer accounts
* [ ] Determine resulting object ownership/control
* [ ] Verify SPNs

### Impact Analysis

* [ ] Identify target service account
* [ ] Determine service privileges
* [ ] Check group membership
* [ ] Check domain permissions
* [ ] Check authentication restrictions
* [ ] Determine effective access

### Validation

* [ ] Confirm RBCD configuration
* [ ] Confirm trusted principal
* [ ] Confirm controlling permissions
* [ ] Confirm target service
* [ ] Confirm delegation relationship
* [ ] Confirm resulting access
* [ ] Collect evidence

---

## Transition

After RBCD analysis, continue to:

**`05-Privilege-Escalation-Paths/ADCS.md`**

The next file covers **Active Directory Certificate Services (AD CS)** and certificate-based privilege escalation paths.
