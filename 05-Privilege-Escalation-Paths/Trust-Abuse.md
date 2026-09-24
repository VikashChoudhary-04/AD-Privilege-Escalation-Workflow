# Trust Abuse

## Purpose

Active Directory trust relationships allow authentication and resource access across domains or forests.

Trusts are legitimate components of multi-domain and multi-forest environments, but their configuration can create unintended privilege paths when combined with:

* Excessive trust relationships
* Weak trust direction
* Broad authentication scope
* SID-related authorization behavior
* Cross-domain group membership
* Misconfigured permissions
* Unnecessary administrative access

This file focuses on **enumerating, analyzing, validating, and documenting trust-based privilege escalation paths** during an authorized security assessment.

The core workflow is:

```text
Enumerate Trusts
      ↓
Identify Source and Target
      ↓
Determine Trust Direction
      ↓
Determine Trust Type
      ↓
Analyze Authentication Scope
      ↓
Analyze SID / Group Relationships
      ↓
Map Cross-Domain Access
      ↓
Determine Security Impact
      ↓
Validate Authorized Path
      ↓
Document Evidence
```

---

## Position in the Workflow

Trust abuse is the final major privilege-escalation path in Section 05.

The section has now covered:

* Group Membership Abuse
* ACL Abuse
* GPO Abuse
* Delegation Abuse
* RBCD
* AD CS
* Trust Abuse

Trust analysis is especially important in environments containing:

```text
Domain A
   ↓
Trust
   ↓
Domain B
```

because privileges can sometimes cross administrative boundaries.

---

## Trust Fundamentals

An AD trust establishes a relationship between security principals in different domains or forests.

A simplified model is:

```text
Domain A
   ↓
Trust Relationship
   ↓
Domain B
```

The trust determines how authentication and authorization can operate between those environments.

A trust does **not** automatically mean that all users in one domain have administrative access to another domain.

---

## Trust Direction

Trust direction describes which domain trusts the other domain.

Conceptually:

```text
Domain A
   ───── trusts ─────>
Domain B
```

The direction must be interpreted correctly.

When documenting a trust, record:

* Source domain
* Target domain
* Trust direction
* Trust type
* Transitivity
* Authentication scope

---

## Trust Types

Common AD trust relationships include:

* Parent-child trusts
* Tree-root trusts
* Forest trusts
* External trusts
* Realm trusts
* Shortcut trusts

Each has different authentication and administrative implications.

Do not assume that all trust types provide equivalent access.

---

## 1. Enumerate Domain Trusts

PowerShell:

```powershell
Get-ADTrust -Filter *
```

For a specific domain:

```powershell
Get-ADTrust -Identity "example.local"
```

Useful information includes:

* Name
* Source
* Target
* Direction
* Trust type
* Trust attributes
* Selective authentication
* SID filtering-related configuration

---

## 2. Enumerate Forest Information

Identify the current forest:

```powershell
Get-ADForest
```

Useful properties include:

* Forest name
* Root domain
* Domains
* Sites
* Global catalog servers
* Forest-wide configuration

Then determine whether other forests are connected through trusts.

---

## 3. Enumerate Domains

Identify all domains within the forest:

```powershell
(Get-ADForest).Domains
```

Also identify domain controllers:

```powershell
Get-ADDomainController -Filter *
```

Build a basic map:

```text
Forest
├── Domain A
├── Domain B
└── Domain C
```

Then map external trust relationships separately.

---

## 4. Trust Direction Analysis

For each trust, record:

| Property       | Question                            |
| -------------- | ----------------------------------- |
| Source         | Which domain owns the relationship? |
| Target         | Which domain is trusted?            |
| Direction      | One-way or two-way?                 |
| Type           | What trust type?                    |
| Transitivity   | Does trust extend further?          |
| Authentication | Who can authenticate?               |
| Authorization  | Which resources can be accessed?    |

This prevents incorrect assumptions about cross-domain access.

---

## 5. One-Way Trust

A one-way trust can be represented as:

```text
Domain A
   │
   │ trusts
   ▼
Domain B
```

The direction determines which security principals can be authenticated across the relationship.

Do not interpret a one-way trust as unrestricted access between both domains.

---

## 6. Two-Way Trust

A two-way trust creates mutual trust relationships:

```text
Domain A
   ⇄
Domain B
```

This can expand the authentication surface.

However, authentication capability still does not automatically grant administrative authorization.

Determine:

* Which resources are accessible
* Which groups contain cross-domain members
* Which ACLs allow cross-domain principals
* Whether selective authentication is enabled

---

## 7. Transitive Trust

Transitive trusts can extend authentication relationships beyond the directly connected domains.

Example:

```text
Domain A
   ↓
Domain B
   ↓
Domain C
```

When analyzing a trust path, determine whether access can extend through additional domains.

This is particularly relevant in multi-domain forests.

---

## 8. External Trusts

External trusts connect domains outside the same forest.

For example:

```text
Forest A
   │
External Trust
   │
Forest B
```

These relationships should be reviewed carefully because they cross normal forest boundaries.

Determine:

* Trust direction
* Authentication scope
* SID filtering
* Selective authentication
* Cross-domain groups
* Resource permissions

---

## 9. Forest Trusts

Forest trusts connect separate AD forests.

Example:

```text
Forest A
   ⇄
Forest B
```

A forest trust does not automatically make administrators of one forest administrators of another.

The assessment must identify the exact authorization paths.

---

## 10. Selective Authentication

Selective authentication can restrict which users from a trusted domain may authenticate to specific resources.

When enabled, evaluate:

* Target computers
* Allowed principals
* Authentication permissions
* Cross-domain groups
* Resource ACLs

The model becomes:

```text
Trusted Domain
      ↓
Selective Authentication
      ↓
Specific Resource
      ↓
Specific Principal
```

Therefore, trust existence alone is insufficient to establish access.

---

## 11. SID Filtering

SID filtering is an important security control for certain external or forest trust configurations.

It helps prevent unauthorized SID information from being used across a trust boundary to influence authorization.

When assessing a trust, determine:

* Whether SID filtering is enabled
* Trust type
* Trust direction
* Whether the relationship is intentionally configured
* Whether cross-domain authorization relies on SID history

---

## 12. SID History

`SIDHistory` allows an account to retain references to previous security identifiers.

It can be legitimate during:

* Domain migrations
* Consolidations
* Organizational changes

However, unexpected or improperly controlled SID history can affect authorization across trust boundaries.

The relevant model is:

```text
Account
   ↓
SIDHistory
   ↓
Authorization
   ↓
Resource Access
```

Do not classify SIDHistory as malicious simply because it exists.

Investigate:

* Source SID
* Target account
* Historical domain
* Group or user identity
* Resource permissions
* Trust relationship

---

## 13. Cross-Domain Group Membership

AD groups can contain principals from other domains when the relevant group scope and trust relationships permit it.

Example:

```text
Domain A User
      ↓
Domain B Group
      ↓
Domain B Resource
```

This can create legitimate or unintended privilege paths.

Enumerate:

* Group members
* Group scope
* Domain membership
* Nested groups
* Resource permissions

---

## 14. Cross-Domain ACLs

ACLs can grant permissions to principals from trusted domains.

Example:

```text
Domain A User
      ↓
Cross-Domain ACE
      ↓
Domain B Server
```

When a cross-domain ACE is discovered, determine:

* Principal
* SID
* Object
* Permission
* Inheritance
* Resource
* Effective privilege

A cross-domain ACL is not automatically dangerous.

Its impact depends on the controlled resource.

---

## 15. Trusts and Privileged Groups

Review whether cross-domain principals appear in:

* Domain Admins
* Enterprise Admins
* Server administrators
* Custom privileged groups
* Local Administrators
* Application administration groups

For each relationship, identify:

```text
Principal
   ↓
Group
   ↓
Resource
   ↓
Effective Privilege
```

---

## 16. Trusts and Authentication Paths

Authentication and authorization should be analyzed separately.

Example:

```text
Domain A User
      ↓
Can Authenticate to Domain B
      ↓
Does NOT necessarily mean
      ↓
Can Administer Domain B
```

A successful authentication relationship should therefore be followed by authorization analysis.

---

## 17. Trust and ACL Correlation

Trust analysis should be correlated with ACL findings.

Example:

```text
Domain A User
      ↓
Trusted Relationship
      ↓
Domain B Resource
      ↓
Cross-Domain ACL
      ↓
Administrative Permission
```

This is a more meaningful attack path than simply reporting that a trust exists.

---

## 18. Trust and Group Correlation

Cross-domain group membership can produce indirect paths.

Example:

```text
Domain A User
      ↓
Domain B Group
      ↓
Nested Group
      ↓
Privileged Group
      ↓
Domain B Resource
```

Analyze every membership relationship before determining effective privilege.

---

## 19. Trust and SIDHistory Correlation

A potentially important path can involve:

```text
Domain A Account
      ↓
SIDHistory
      ↓
Historical / Trusted SID
      ↓
Domain B Authorization
      ↓
Resource Access
```

When SIDHistory is involved, determine:

* Who controls the account
* What SID is present
* Which domain issued it
* Which resource recognizes it
* Whether SID filtering applies
* Whether the relationship is legitimate

---

## 20. Trust and Delegation

Trusts can also intersect with Kerberos delegation.

Consider:

```text
Domain A
      ↓
Trust
      ↓
Domain B
      ↓
Delegation Configuration
      ↓
Target Service
```

If delegation and trust relationships overlap, analyze the combined authentication path rather than evaluating each configuration independently.

---

## 21. Trust Attack-Path Construction

A trust-based privilege path should contain verified relationships.

Example:

```text
User A
  ↓
Domain A
  ↓
Trust Relationship
  ↓
Domain B
  ↓
Cross-Domain Group / ACL
  ↓
Privileged Resource
  ↓
Effective Access
```

Every step must be supported by enumeration or validation evidence.

---

## 22. BloodHound-Compatible Trust Analysis

BloodHound-compatible collectors can help visualize:

* Domains
* Trusts
* Users
* Groups
* Computers
* ACLs
* Cross-domain relationships
* Group memberships

Use graph analysis to identify candidate paths:

```text
Domain
  ↓
Trust
  ↓
Principal
  ↓
Group
  ↓
Resource
```

Then validate each relationship against Active Directory.

---

## 23. Determine Effective Trust Impact

For each trust, answer:

### Authentication

Can principals from one side authenticate across the relationship?

### Authorization

What resources can those principals access?

### Scope

Is access broad or limited?

### Direction

Which side trusts which?

### Transitivity

Can the relationship extend to other domains?

### Security Controls

Are selective authentication and SID filtering enabled where applicable?

---

## 24. Validate Trust-Based Paths

Use the following validation sequence:

```text
Trust Identified
      ↓
Direction Confirmed
      ↓
Trust Type Confirmed
      ↓
Authentication Scope Confirmed
      ↓
Principal Identified
      ↓
Cross-Domain Permission Confirmed
      ↓
Target Resource Confirmed
      ↓
Effective Privilege Confirmed
```

This separates a trust configuration from a confirmed privilege path.

---

## 25. Evidence Collection

Record:

| Field                    | Description                               |
| ------------------------ | ----------------------------------------- |
| Source Domain            | Origin domain                             |
| Target Domain            | Destination domain                        |
| Trust Type               | Trust relationship type                   |
| Direction                | Trust direction                           |
| Transitivity             | Whether trust is transitive               |
| Authentication           | Authentication scope                      |
| SID Filtering            | Relevant configuration                    |
| Selective Authentication | Relevant configuration                    |
| Principal                | Cross-domain user/group                   |
| Permission               | Resource permission                       |
| Target                   | Resource affected                         |
| Result                   | Effective access                          |
| Evidence                 | Commands, output, screenshots, graph data |

---

## 26. Common False Positives

### Trust Exists but No Privileged Access

A trust may exist without granting meaningful administrative privileges.

### Read-Only Cross-Domain Access

The principal may only have read permissions.

### Selective Authentication

Authentication may be restricted to specific resources.

### SID Filtering

SID-related authorization may be restricted.

### Legitimate Migration Configuration

SIDHistory or cross-domain permissions may exist for legitimate migration reasons.

### Low-Impact Resource

Cross-domain access may be limited to a non-sensitive application or workstation.

---

## 27. Common Mistakes

Avoid:

* Treating every trust as a vulnerability
* Confusing trust direction
* Assuming two-way trust means administrative access
* Ignoring selective authentication
* Ignoring SID filtering
* Treating SIDHistory as automatically malicious
* Ignoring cross-domain group scope
* Ignoring nested groups
* Ignoring cross-domain ACLs
* Confusing authentication with authorization
* Assuming forest trust means forest compromise
* Reporting trust existence without demonstrating security impact

---

## 28. Assessment Checklist

### Trust Enumeration

* [ ] Enumerate domains
* [ ] Enumerate forests
* [ ] Enumerate trusts
* [ ] Identify trust direction
* [ ] Identify trust type
* [ ] Identify transitivity

### Security Controls

* [ ] Check selective authentication
* [ ] Check SID filtering
* [ ] Review trust attributes
* [ ] Identify authentication scope

### Cross-Domain Analysis

* [ ] Enumerate cross-domain users
* [ ] Enumerate cross-domain groups
* [ ] Review nested groups
* [ ] Review cross-domain ACLs
* [ ] Review privileged groups
* [ ] Review SIDHistory

### Attack Path Analysis

* [ ] Identify source principal
* [ ] Identify target domain
* [ ] Identify target resource
* [ ] Map authentication
* [ ] Map authorization
* [ ] Determine effective privilege

### Validation

* [ ] Confirm trust relationship
* [ ] Confirm direction
* [ ] Confirm authentication scope
* [ ] Confirm cross-domain permission
* [ ] Confirm target resource
* [ ] Confirm effective access
* [ ] Collect evidence

---

## Transition

This completes the individual privilege-escalation-path topics in Section 05.

Next, continue to:

**`06-Attack-Path-Analysis/README.md`**

Section 06 will bring the findings together and focus on **BloodHound, attack-path identification, correlation, and validation of complete AD privilege-escalation chains**.
