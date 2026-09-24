# Domain Admin Validation

## Purpose

Domain Admin validation determines whether a tested identity actually possesses **Domain Admin-level authorization** within the current Active Directory domain.

The goal is to distinguish:

```text id="7q4m2x"
Potential Domain-Level Path
        ↓
Verified Domain Admin Membership / Equivalent Privilege
```

from weaker results such as:

* Local Administrator
* Object-level control
* Server-specific administration
* Membership in a custom administrative group
* Access to a privileged service

The validation should be evidence-driven and use the least invasive method necessary.

---

## Position in the Workflow

This file follows:

**`07-Privilege-Validation/README.md`**

The workflow is:

```text id="m8x3q7"
Attack Path
      ↓
Path Validation
      ↓
Privilege Validation
      ↓
Domain Admin Validation
      ↓
Evidence Collection
      ↓
Reporting
```

---

## What Domain Admin Means

**Domain Admins** is a built-in privileged security group within an Active Directory domain.

Membership generally provides extensive administrative capabilities across the domain, subject to the environment's security configuration.

However:

```text id="p6n4v8"
Domain Admin
≠
Any Administrative Access
```

A user who can administer one server is not automatically a Domain Admin.

---

# 1. Identify the Domain

First establish the domain being assessed.

PowerShell:

```powershell id="q3m7x9"
Get-ADDomain
```

Useful information includes:

* DNS domain name
* NetBIOS name
* Domain SID
* Domain controllers
* Distinguished name
* Parent/child relationships

---

# 2. Enumerate Domain Admins

Retrieve current membership:

```powershell id="n8x4q2"
Get-ADGroupMember "Domain Admins"
```

For recursive membership:

```powershell id="v5m3r8"
Get-ADGroupMember "Domain Admins" -Recursive
```

Recursive enumeration is important because privileged access may be obtained through nested groups.

---

# 3. Identify the Tested Identity

Determine whether the assessed identity is:

* Direct member
* Nested member
* Computer account
* Service account
* Member through another privileged group

Example:

```text id="j7q2m5"
User A
 ↓
MemberOf
 ↓
Group A
 ↓
MemberOf
 ↓
Domain Admins
```

The complete membership chain should be documented.

---

# 4. Verify Direct Membership

For a direct membership relationship:

```powershell id="c8x4p6"
Get-ADGroupMember "Domain Admins" |
    Where-Object {$_.SamAccountName -eq "user01"}
```

This provides direct membership evidence.

---

# 5. Verify Nested Membership

Nested groups require additional analysis.

Example:

```text id="y4m8q3"
User A
 ↓
Group A
 ↓
Group B
 ↓
Domain Admins
```

Use recursive membership:

```powershell id="h6n3x9"
Get-ADGroupMember "Domain Admins" -Recursive
```

Then identify the exact membership chain.

---

# 6. Verify Account Status

A privileged membership is not enough to establish current usable access.

Check:

```powershell id="r9q5m2"
Get-ADUser "user01" -Properties Enabled,LockedOut,AccountExpirationDate
```

Relevant properties include:

* Enabled
* LockedOut
* AccountExpirationDate
* PasswordExpired
* PasswordLastSet

An inactive or disabled identity should be documented accurately.

---

# 7. Verify the Domain

Make sure the privileged group belongs to the intended domain.

In multi-domain environments:

```text id="x5v7m3"
Forest
├── Domain A
│   └── Domain Admins
│
└── Domain B
    └── Domain Admins
```

Membership in Domain A's Domain Admins should not be reported as Domain B's Domain Admin access.

---

# 8. Distinguish Domain Admin from Enterprise Admin

These are different administrative scopes.

### Domain Admins

Administrative group for a specific domain.

```text id="p7m4x9"
Domain A
└── Domain Admins
```

### Enterprise Admins

Forest-level administrative group located in the forest root domain.

```text id="n3q8m5"
Forest
└── Root Domain
    └── Enterprise Admins
```

Do not use these terms interchangeably.

---

# 9. Domain Admin Equivalent Access

Not every route to extensive domain control requires direct Domain Admin membership.

An assessment may identify a principal with another set of permissions that provides comparable administrative control over important domain resources.

Examples may involve:

* Highly privileged custom groups
* Extensive control over domain controllers
* Administrative control over critical AD objects
* Other domain-wide delegated permissions

Such cases should be documented as **effective domain-level administrative access**, with the exact permissions described.

Do not automatically label every broad privilege as Domain Admin membership.

---

# 10. Validate the Source Attack Path

Domain Admin validation should reference the attack path that produced the privilege.

Example:

```text id="w8m3q5"
User A
 ↓
ACL Abuse
 ↓
Group B
 ↓
Membership Control
 ↓
Domain Admins
```

Confirm each relationship:

1. User A exists.
2. User A controls Group B.
3. Group B affects Domain Admin membership.
4. The resulting membership is current.
5. User A receives the resulting privilege.

---

# 11. Validate Group Membership Changes

If an authorized test modifies group membership, verify the resulting directory state.

For example:

```powershell id="m4q7x8"
Get-ADGroupMember "Domain Admins" -Recursive
```

Confirm:

* Expected account
* Correct domain
* Current membership
* Expected nested path

Avoid unnecessary changes in production environments.

---

# 12. Validate Existing Domain Admin Access

If the account is already a Domain Admin, the task is to establish evidence of current privilege rather than create additional access.

Useful evidence includes:

```powershell id="t5x8n3"
whoami
```

and:

```powershell id="v7m2q9"
whoami /groups
```

These commands can help establish the current Windows security context.

---

# 13. Domain Controller Context

A Domain Controller provides domain-level security functionality.

When validating domain-level administrative access, identify the domain controllers:

```powershell id="c6q4m8"
Get-ADDomainController -Filter *
```

Record:

* Hostname
* Domain
* Operating system
* Site
* IP address where authorized

Avoid making unnecessary changes to domain controllers.

---

# 14. Administrative Access vs Domain Admin Membership

These are separate claims.

### Claim A

```text
User can administer SERVER01
```

### Claim B

```text
User is a member of Domain Admins
```

### Claim C

```text
User has another verified path to broad domain-level administrative control
```

Each requires different evidence.

Do not use evidence for Claim A to support Claim B.

---

# 15. Local Administrator Is Not Domain Admin

Consider:

```text id="y9q3m6"
User
 ↓
Local Administrators
 ↓
SERVER01
```

This establishes local administrative privilege on SERVER01.

It does not automatically establish:

```text id="m7x4p2"
Domain Admin
```

The scope must be explicitly documented.

---

# 16. Domain Admin and Custom Groups

Some organizations use custom administrative groups.

Example:

```text id="q5m8x3"
User
 ↓
Enterprise-Server-Admins
 ↓
Administrative Access
```

Determine:

* Group membership
* ACLs
* Delegated permissions
* Domain scope
* Target resources

Describe the actual privilege rather than inferring Domain Admin membership from the group name.

---

# 17. Domain Admin and ACL Control

An ACL path may lead to significant domain-level control.

Example:

```text id="x8n4q6"
User
 ↓
WriteDacl
 ↓
Privileged Group
 ↓
Membership Control
 ↓
Domain Admins
```

Validate every relationship.

The initial ACL permission alone does not prove Domain Admin access.

---

# 18. Domain Admin and GPO

A GPO-based path may affect domain-level administrative resources.

Example:

```text id="j3m7v9"
User
 ↓
GPO Control
 ↓
GPO
 ↓
Domain Controller OU
 ↓
Security-Relevant Policy
```

The effective policy and resulting privilege must be validated.

Do not classify GPO modification alone as Domain Admin access.

---

# 19. Domain Admin and Delegation

Delegation paths may potentially reach privileged domain services.

Example:

```text id="n6q2x8"
Controlled Principal
 ↓
Delegation
 ↓
Target Service
 ↓
Privileged Identity
```

Verify the actual target identity and authorization.

---

# 20. Domain Admin and AD CS

Certificate-based paths may map to privileged domain identities.

Example:

```text id="p8x3m7"
Principal
 ↓
Certificate Template
 ↓
Certificate
 ↓
Privileged Identity
 ↓
Domain-Level Authorization
```

Verify:

* Certificate issuance
* Mapping
* Authentication
* Authorization

---

# 21. Domain Admin and Trusts

A trust relationship does not automatically grant Domain Admin privileges.

Example:

```text id="r4m7q2"
Domain A User
 ↓
Trust
 ↓
Domain B
```

Additional authorization must exist:

```text id="v8x3n5"
Domain B
 ↓
Cross-Domain Group / ACL
 ↓
Domain B Privilege
```

Validate the complete path.

---

# 22. Effective Privilege Validation

For a final privilege determination, ask:

```text id="h6q4m8"
Who is the identity?
        ↓
Which domain?
        ↓
Which groups?
        ↓
Which permissions?
        ↓
Which resources?
        ↓
What is the effective administrative scope?
```

This provides a defensible result.

---

# 23. Evidence Collection

Record:

| Field          | Description                                |
| -------------- | ------------------------------------------ |
| Identity       | Tested principal                           |
| Domain         | Relevant AD domain                         |
| Group          | Domain Admins or relevant privileged group |
| Membership     | Direct/nested/effective                    |
| Source Path    | Attack path producing access               |
| Account Status | Enabled/disabled/etc.                      |
| Scope          | Domain/resource scope                      |
| Validation     | Test performed                             |
| Evidence       | Commands/output/screenshots                |

---

# 24. Evidence for Domain Admin Membership

Strong evidence can include:

```powershell id="w5m9x2"
Get-ADGroupMember "Domain Admins" -Recursive
```

Combined with:

```powershell id="q8x3n6"
Get-ADUser "user01" -Properties Enabled
```

And, where appropriate:

```powershell id="r4m7v8"
whoami /groups
```

Evidence should establish both identity and privilege.

---

# 25. Evidence for Equivalent Domain-Level Control

If the principal is not a Domain Admin but has significant equivalent control, document:

```text id="k6q3x9"
Principal
 ↓
Permission
 ↓
Object / Resource
 ↓
Administrative Capability
 ↓
Scope
```

Explain the exact control instead of assigning an unsupported label.

---

# 26. Common False Positives

### Local Administrator

Administrative access to one computer is not Domain Admin.

### Server Administrator

Control over servers does not automatically imply Domain Admin.

### Privileged-Sounding Group

A custom group name does not establish its actual permissions.

### Authentication

Successful authentication does not establish administrative privilege.

### Trust

A trust does not automatically create Domain Admin access.

### Stale Membership

Historical group membership may no longer be current.

---

# 27. Common Mistakes

Avoid:

* Confusing Domain Admins with Enterprise Admins
* Confusing local admin with domain admin
* Assuming custom groups are equivalent without evidence
* Treating trust as authorization
* Treating GPO control as automatic Domain Admin
* Treating certificate issuance as Domain Admin
* Ignoring nested groups
* Ignoring account status
* Ignoring domain boundaries
* Making unnecessary changes to Domain Admin membership

---

# 28. Assessment Checklist

### Identity

* [ ] Identify tested account
* [ ] Confirm account status
* [ ] Confirm domain
* [ ] Confirm current context

### Domain Admin Membership

* [ ] Enumerate Domain Admins
* [ ] Check direct membership
* [ ] Check nested membership
* [ ] Confirm current membership
* [ ] Confirm correct domain

### Equivalent Privilege

* [ ] Identify custom privileged groups
* [ ] Review effective permissions
* [ ] Review ACLs
* [ ] Review GPO control
* [ ] Review delegation
* [ ] Review AD CS
* [ ] Review trust relationships

### Validation

* [ ] Validate source attack path
* [ ] Confirm resulting identity
* [ ] Confirm authorization
* [ ] Confirm administrative scope
* [ ] Collect evidence
* [ ] Avoid unnecessary modification

### Documentation

* [ ] Record exact privilege
* [ ] Record domain
* [ ] Record membership path
* [ ] Record attack path
* [ ] Record evidence
* [ ] Record limitations

---

## Transition

Continue with:

**`07-Privilege-Validation/Evidence-Collection.md`**

The final file in Section 07 will establish a structured method for collecting and organizing **defensible evidence for validated privileges and attack paths** before moving into the reporting phase.
