# Privilege Validation

## Purpose

Privilege Validation is the stage where a confirmed attack path is translated into a clearly defined **effective privilege or access level**.

The objective is to answer:

> What privilege does the assessed principal actually have after following the validated attack path?

This prevents a common reporting mistake: identifying a technically interesting relationship without proving what security impact it produces.

The core workflow is:

```text id="j4m8q2"
Validated Attack Path
      ↓
Identify Resulting Identity
      ↓
Identify Effective Permissions
      ↓
Validate Target Access
      ↓
Determine Privilege Level
      ↓
Collect Evidence
      ↓
Document Result
```

---

## Position in the Workflow

Section 07 follows the attack-path analysis performed in Section 06.

The overall progression is:

```text id="7n3x6p"
Enumeration
      ↓
Credential Analysis
      ↓
Privilege Escalation Paths
      ↓
Attack Path Analysis
      ↓
Privilege Validation
      ↓
Reporting
```

The distinction is important:

```text id="q8m5v3"
Potential Relationship
        ↓
Validated Attack Path
        ↓
Effective Privilege
```

---

## What Is Privilege Validation?

Privilege validation determines the actual authorization available after an attack path succeeds.

Examples include:

* Local Administrator
* Member of a privileged domain group
* Control over a sensitive AD object
* Access to a privileged service account
* Domain Administrator-level access
* Enterprise-level administrative access
* Access to a sensitive server

The exact result depends on the environment.

---

## Configuration vs Effective Privilege

A configuration does not automatically equal privilege.

For example:

```text id="v6q2m8"
User
 ↓
GenericAll
 ↓
Computer
```

This establishes object control.

But the assessment must continue:

```text id="n4x8p3"
Computer
 ↓
What Can Be Controlled?
 ↓
What Access Results?
 ↓
What Privilege Exists?
```

This distinction makes findings more accurate.

---

# 1. Define the Validated Path

Start with the attack path established in Section 06.

Example:

```text id="w7m3q9"
User A
 ↓
Group Membership
 ↓
Privileged Group
```

Record:

* Starting principal
* Intermediate objects
* Relationships
* Target
* Validation evidence

---

# 2. Identify the Resulting Identity

Determine which identity actually holds the resulting privilege.

It may be:

* Original user
* Another domain user
* Service account
* Computer account
* Local account
* Certificate-mapped identity

Example:

```text id="p5x8r2"
Certificate
 ↓
Mapped Identity
 ↓
Administrator Account
```

The identity must be verified rather than inferred.

---

# 3. Determine Effective Group Membership

For a domain account, inspect group membership.

Example:

```powershell id="m9q4v6"
Get-ADPrincipalGroupMembership "user01"
```

Also inspect nested memberships where relevant.

Example:

```text id="c7x2n8"
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Group
```

The effective privilege may come from a nested group rather than direct membership.

---

# 4. Identify Privileged Groups

Common high-impact groups include:

* Domain Admins
* Enterprise Admins
* Administrators
* Account Operators
* Backup Operators
* Server Operators
* Print Operators
* DNSAdmins
* Custom privileged groups

Group names alone are not sufficient.

Determine what permissions the group actually has in the environment.

---

# 5. Validate Local Administrator Access

Local administrative access is distinct from domain-level administrative access.

Example:

```text id="h8m3q5"
User
 ↓
Local Administrators
 ↓
SERVER01
 ↓
Local Administrative Privilege
```

Validate:

* Target computer
* User/group membership
* Local group membership
* Effective authorization
* Current account status

---

# 6. Validate Administrative Access on a Computer

Where authorized, determine whether the principal can perform an administrative operation on the target.

Possible validation methods include:

* Inspecting local group membership
* Querying Windows security configuration
* Checking administrative access through an authorized management protocol
* Performing a non-destructive administrative test

The goal is to prove authorization without unnecessary system modification.

---

# 7. Domain-Level Privilege

Domain-level privileges affect multiple resources across the AD environment.

Examples include:

```text id="t6n4p9"
Domain Administrator
      ↓
Broad Domain Administrative Control
```

or:

```text id="v8q2m5"
Privileged Domain Group
      ↓
Specific Administrative Permissions
```

Do not assume every privileged-looking group provides Domain Admin-equivalent access.

---

# 8. Domain Admin Validation

When a validated path appears to reach Domain Admin-level access, confirm:

* Account identity
* Group membership
* Domain
* Domain Admins membership
* Enabled state
* Current configuration

Example:

```powershell id="c5r8x3"
Get-ADGroupMember "Domain Admins"
```

For a specific account:

```powershell id="p7m4q9"
Get-ADPrincipalGroupMembership "admin01"
```

The objective is to establish the authorization state.

---

# 9. Enterprise-Level Privilege

Enterprise Admins is a forest-level administrative group.

When relevant, determine:

* Forest
* Root domain
* Enterprise Admins membership
* Account identity
* Current enabled state

Example:

```powershell id="y3n7q5"
Get-ADGroupMember "Enterprise Admins"
```

Enterprise-level privilege should not be inferred simply because a user has administrative access to one domain.

---

# 10. Privileged Service Accounts

Some attack paths result in control of service accounts rather than membership in a privileged group.

Validate:

* Account identity
* SPNs
* Group membership
* Delegation
* ACLs
* Services using the account
* Resource permissions

Example:

```text id="m6q2x8"
User
 ↓
Service Account
 ↓
Privileged Service
 ↓
Sensitive Resource
```

The service account's actual permissions determine impact.

---

# 11. Computer Account Privilege

Computer accounts are security principals.

A computer account may have:

* Group memberships
* SPNs
* ACL permissions
* Delegation relationships
* Resource access

Do not automatically equate computer-account control with Domain Admin.

Determine its effective privileges.

---

# 12. Object Control

Some paths result in control over an AD object rather than a privileged identity.

Examples:

* User object
* Group object
* Computer object
* GPO
* OU
* Certificate template

Document the exact control:

```text id="r5x8m3"
Principal
 ↓
Object
 ↓
Permission
 ↓
Security-Relevant Operation
```

This may itself constitute a security finding even if no administrative group is reached.

---

# 13. GPO Privilege Validation

For GPO-based paths, determine:

* Who can modify the GPO
* Which systems receive it
* What policy is applied
* What identity receives the resulting access
* What effective privilege follows

Example:

```text id="q9m4v6"
GPO
 ↓
Target Computer
 ↓
Local Group / Policy
 ↓
Effective Privilege
```

---

# 14. Delegation Privilege Validation

For delegation paths, validate:

* Source identity
* Delegation configuration
* Target service
* Target account
* Authentication context
* Resulting authorization

Example:

```text id="x7p3n5"
Delegated Identity
 ↓
Target Service
 ↓
Target Account
 ↓
Effective Permission
```

---

# 15. RBCD Privilege Validation

For RBCD paths:

```text id="h4m8q2"
Trusted Principal
 ↓
RBCD
 ↓
Target Computer
 ↓
Target Service
 ↓
Target Identity
 ↓
Effective Privilege
```

Determine the actual service-account context and target authorization.

---

# 16. AD CS Privilege Validation

For certificate-based paths:

```text id="s6v3x9"
Certificate
 ↓
Mapped AD Identity
 ↓
Authentication
 ↓
Authorization
 ↓
Effective Privilege
```

Verify the identity mapping and resulting permissions.

Do not report certificate issuance alone as Domain Admin access.

---

# 17. Trust-Based Privilege Validation

For cross-domain paths:

```text id="w8q5m2"
Source Identity
 ↓
Trust
 ↓
Target Domain
 ↓
Cross-Domain Authorization
 ↓
Target Resource
 ↓
Effective Privilege
```

Validate both authentication and authorization.

---

# 18. Effective Permission Analysis

Effective permission may be influenced by:

* Direct ACEs
* Group membership
* Nested groups
* Inherited ACEs
* Deny ACEs
* Object ownership
* GPO processing
* Delegation restrictions
* Certificate mapping
* Trust controls

Therefore:

```text id="n7m3x8"
Configured Permission
      ↓
Effective Permission
```

must be established.

---

# 19. Authentication vs Authorization

Always separate:

### Authentication

The identity successfully proves who it is.

```text id="v5q8m2"
Identity
 ↓
Authentication
```

### Authorization

The identity is allowed to perform the requested operation.

```text id="j3x7p9"
Authenticated Identity
 ↓
Authorization
 ↓
Resource Access
```

Both are required for a complete privilege-validation result.

---

# 20. Privilege Scope

Record the scope of the resulting privilege.

Possible scopes include:

### Object

```text id="s8m4q2"
One AD Object
```

### Computer

```text id="p6x3n7"
One Computer
```

### OU

```text id="w9q5m4"
OU / Child Objects
```

### Domain

```text id="h2v7x8"
Entire Domain
```

### Forest

```text id="c4m9p6"
Multiple Domains / Forest
```

This prevents overstatement.

---

# 21. Least-Impact Validation

Validation should use the smallest action capable of proving the privilege.

For example:

```text id="f7q3m8"
Read Effective Permission
      ↓
Sufficient Evidence
      ↓
Stop
```

If a non-destructive check establishes administrative authorization, additional system modification may be unnecessary.

---

# 22. Evidence Collection

Record:

| Field          | Description                      |
| -------------- | -------------------------------- |
| Principal      | Identity holding privilege       |
| Target         | Resource                         |
| Privilege      | Effective permission             |
| Scope          | Object/computer/domain/forest    |
| Source Path    | Attack path producing access     |
| Authentication | How identity was established     |
| Authorization  | Permission verified              |
| Validation     | Test performed                   |
| Evidence       | Output/screenshots/configuration |

---

# 23. Evidence Hierarchy

Prefer evidence that directly establishes the claim.

### Strong Evidence

* Current AD membership
* Current ACL
* Current GPO configuration
* Current certificate mapping
* Controlled authorization test

### Supporting Evidence

* BloodHound graph
* LDAP discovery
* Tool output
* Historical data

Graph data is useful, but current directory configuration should be preferred when proving present privilege.

---

# 24. Privilege Validation Example

Example path:

```text id="x6q4m8"
User A
 ↓
WriteDacl
 ↓
Group B
 ↓
Membership Control
 ↓
Domain Admins
```

Validation:

```text id="v9m3p5"
1. Confirm User A exists.
2. Confirm User A has WriteDacl on Group B.
3. Confirm Group B can affect the target membership path.
4. Confirm the relevant membership relationship.
5. Confirm the resulting privileged authorization.
```

Each step should have evidence.

---

# 25. Failed Privilege Validation

A path may be valid up to a certain point but fail at the final privilege stage.

Example:

```text id="q3n7x8"
User
 ↓
ACL
 ↓
Computer
 ↓
Expected Admin Access
        ✕
```

Possible reasons:

* Wrong target
* Missing local group membership
* Security restriction
* Account disabled
* Service unavailable
* Network restriction

Document the failure rather than forcing the path into a confirmed finding.

---

# 26. Common False Positives

### Privileged Group Name

A custom group may sound privileged without actually providing the expected rights.

### Administrative Object Control

Object control does not automatically mean domain-wide administration.

### Authentication Without Authorization

Successful authentication does not prove administrative access.

### Local vs Domain Privilege

Local Administrator on one computer is not equivalent to Domain Administrator.

### Stale Membership

Historical membership may no longer be current.

---

# 27. Common Mistakes

Avoid:

* Equating object control with Domain Admin
* Equating local admin with domain admin
* Treating certificate issuance as privilege
* Treating trust as authorization
* Treating delegation as automatic privilege
* Ignoring nested groups
* Ignoring inherited permissions
* Ignoring deny permissions
* Ignoring account status
* Performing unnecessary destructive validation

---

# 28. Assessment Checklist

### Identity

* [ ] Identify resulting principal
* [ ] Confirm account status
* [ ] Confirm domain
* [ ] Confirm group membership

### Authorization

* [ ] Confirm effective permission
* [ ] Confirm target
* [ ] Confirm privilege scope
* [ ] Check inherited permissions
* [ ] Check nested groups
* [ ] Check relevant security controls

### Validation

* [ ] Confirm authentication
* [ ] Confirm authorization
* [ ] Confirm security-relevant operation
* [ ] Confirm effective privilege
* [ ] Use least-impact validation

### Evidence

* [ ] Record source path
* [ ] Record resulting identity
* [ ] Record target
* [ ] Record effective privilege
* [ ] Record scope
* [ ] Collect supporting evidence
* [ ] Sanitize sensitive information

---

## Transition

Continue with:

**`07-Privilege-Validation/Domain-Admin-Validation.md`**

The next file focuses specifically on validating **Domain Admin-level access**, distinguishing true domain-level administrative privilege from local, object-level, or narrowly scoped administrative access.
