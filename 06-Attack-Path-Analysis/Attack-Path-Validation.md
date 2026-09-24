# Attack Path Validation

## Purpose

Attack Path Validation is the final stage of Section 06.

The purpose is to determine whether a candidate Active Directory attack path is actually valid in the current authorized environment.

A graph relationship, ACL, GPO configuration, delegation setting, certificate template, or trust relationship may indicate a potential path without proving that the complete path works.

The core workflow is:

```text id="v6q2m8"
Candidate Path
      ↓
Verify Source
      ↓
Verify Every Relationship
      ↓
Verify Preconditions
      ↓
Verify Target
      ↓
Validate Security-Relevant Operation
      ↓
Confirm Effective Privilege
      ↓
Collect Evidence
      ↓
Classify Result
```

---

## Position in the Workflow

This file completes:

```text id="3x7n5p"
06-Attack-Path-Analysis
```

The section progression is:

```text
README.md
    ↓
BloodHound.md
    ↓
Attack-Path-Identification.md
    ↓
Attack-Path-Validation.md
```

The objective is to move from:

```text
Potential Path
```

to one of:

```text
Confirmed
Blocked
Unverified
Not Applicable
```

---

# 1. Candidate Path

Begin with a clearly defined candidate.

Example:

```text id="m8r4y2"
User A
 ↓
GenericWrite
 ↓
Computer B
 ↓
Local Administrator
```

Before validation, this is only a hypothesis.

Record where the candidate came from:

* BloodHound
* LDAP
* PowerShell
* Manual ACL analysis
* GPO analysis
* AD CS analysis
* Delegation analysis
* Trust analysis

---

# 2. Define the Starting Context

Record:

* Username
* Domain
* Account type
* Enabled state
* Current groups
* Current privileges
* Available credentials
* Network position

Example:

```text id="p5x9k3"
Principal:
    user01@example.local

Current Context:
    Standard Domain User
```

Do not validate a path using a different identity without explicitly documenting the change.

---

# 3. Define the Target

The target should be explicit.

Examples:

* Domain Admins
* Enterprise Admins
* Domain Controller
* Local Administrator
* Privileged Server
* Service Account
* Certificate-Based Identity
* Sensitive Application

Example:

```text id="q7m3v8"
Target:
    SERVER01
```

---

# 4. Break the Path Into Individual Steps

Never validate a complex path as a single operation.

Example:

```text id="x4n8r2"
User
 ↓
Group Membership
 ↓
ACL
 ↓
Computer
 ↓
Local Privilege
```

Break it into:

```text id="h5k7p3"
Step 1:
User → Group

Step 2:
Group → ACL

Step 3:
ACL → Computer

Step 4:
Computer → Privilege
```

Each step requires independent evidence.

---

# 5. Verify the Source

Confirm that the starting account:

* Exists
* Is enabled
* Has the expected identity
* Has the expected group membership
* Is within scope

Example:

```powershell id="w8q3m6"
Get-ADUser "user01" -Properties Enabled,MemberOf
```

Do not rely solely on previously collected data.

---

# 6. Verify the Relationship

For each edge, identify the exact relationship.

Examples:

```text id="n6v4x8"
MemberOf
GenericAll
GenericWrite
WriteDacl
WriteOwner
WriteProperty
GPO Control
Delegation
Trust
Certificate Enrollment
```

Then verify the underlying AD configuration.

---

# 7. Verify ACL Relationships

For ACL-based paths, inspect the object's security descriptor.

Example:

```powershell id="j4p7s2"
Get-Acl "AD:\CN=SERVER01,OU=Servers,DC=example,DC=local"
```

Verify:

* Principal
* Permission
* Allow/Deny
* Inheritance
* Object scope
* Effective control

Do not treat a generic relationship name as sufficient evidence.

---

# 8. Verify Group Membership

For membership paths:

```powershell id="y8m3q5"
Get-ADGroupMember "Example-Group"
```

For a user's effective group relationships:

```powershell id="r6x2n9"
Get-ADPrincipalGroupMembership "user01"
```

Also inspect nested groups where relevant.

---

# 9. Verify GPO Relationships

For GPO paths, verify:

* GPO exists
* Principal can modify it
* GPO is linked
* Link applies to target
* Security filtering permits application
* WMI filtering does not exclude target
* Policy actually affects the target

Example:

```powershell id="k3v8m5"
Get-GPO -Name "Example Policy"
```

And:

```powershell id="b7q2x4"
Get-GPPermission -Name "Example Policy" -All
```

---

# 10. Verify Delegation

For delegation paths, verify:

* Source
* Delegation type
* SPN
* Target
* Account restrictions
* Target service
* Effective privilege

Do not classify a delegation configuration as exploitable without establishing the relevant authentication and authorization conditions.

---

# 11. Verify RBCD

For RBCD paths, verify:

```text id="p8m5r3"
Target Computer
      ↓
RBCD Attribute
      ↓
Trusted Principal
      ↓
Who Controls Trusted Principal?
      ↓
Target Service
      ↓
Effective Privilege
```

Inspect:

```powershell id="t6x4q9"
Get-ADComputer "SERVER01" `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity
```

Also inspect the target computer's ACL.

---

# 12. Verify AD CS

For AD CS paths, verify:

* CA
* Template
* Enrollment permission
* Template configuration
* EKU
* Subject/SAN behavior
* Approval requirements
* Certificate mapping
* Strong-binding protections
* Resulting identity
* Effective privilege

The path is:

```text id="c7m2x5"
Principal
 ↓
Enrollment
 ↓
Template
 ↓
Certificate
 ↓
Identity Mapping
 ↓
Authentication
 ↓
Authorization
```

---

# 13. Verify Trusts

For cross-domain paths, verify:

* Trust exists
* Trust direction
* Trust type
* Transitivity
* Authentication scope
* Selective authentication
* SID filtering where relevant
* Cross-domain membership
* Resource ACL

Do not assume that trust existence proves access.

---

# 14. Verify Credentials

For credential-based paths, determine:

* Credential owner
* Credential validity
* Account status
* Authentication method
* Scope
* Privileges

A discovered credential should not automatically be considered usable.

---

# 15. Verify Network Reachability

A logical AD path may fail because the required target cannot be reached.

Check:

* Routing
* Firewall
* Network segmentation
* Required service ports
* VPN requirements
* Administrative protocols

Example:

```text id="z5n7q3"
Authorization
      ↓
Target Resource
      ↓
Network Reachability
      ↓
Required Service
```

---

# 16. Verify Service Availability

A target may exist but the required service may not be running.

Verify:

* Host availability
* Service availability
* Authentication endpoint
* Required protocol
* Current configuration

This is especially important for:

* SMB
* LDAP
* Kerberos
* WinRM
* RDP
* SQL Server
* Web services

---

# 17. Verify Authentication

Determine which identity is authenticated at each stage.

Examples:

```text id="r3m8x6"
User
 ↓
Kerberos
 ↓
Service
```

or:

```text id="v7p4q2"
Certificate
 ↓
Certificate Mapping
 ↓
AD Account
```

The authenticated identity must match the expected attack path.

---

# 18. Verify Authorization

Authentication alone is not enough.

Determine:

```text id="k8x3m5"
Authenticated Identity
      ↓
Group Membership
      ↓
ACL
      ↓
Effective Permission
```

The resulting authorization must correspond to the claimed privilege.

---

# 19. Validate the Security-Relevant Operation

The operation should be validated with the least invasive method capable of proving the path.

Examples include:

* Confirming effective group membership
* Confirming object modification rights
* Confirming GPO application
* Confirming certificate mapping
* Confirming access to the target resource
* Confirming local administrative privilege

Avoid unnecessary changes to production AD.

---

# 20. Validate Without Over-Exploitation

The objective of an assessment is to prove the security condition, not maximize impact.

For example, if an authorized test can establish that a principal has effective administrative access, further destructive actions may be unnecessary.

Prefer:

```text id="q4v8n2"
Minimum Action
      ↓
Sufficient Evidence
      ↓
Stop
```

---

# 21. Complete Path Validation

A complete path should resemble:

```text id="x6m3p9"
Initial Principal
      ↓
Verified Relationship
      ↓
Verified Object
      ↓
Verified Permission
      ↓
Verified Operation
      ↓
Verified Target
      ↓
Verified Effective Privilege
```

If any critical relationship cannot be established, classify the path accordingly.

---

# 22. Validation Matrix

Use a table such as:

| Step | Source     | Relationship | Target     | Evidence              | Status    |
| ---- | ---------- | ------------ | ---------- | --------------------- | --------- |
| 1    | User A     | MemberOf     | Group A    | AD query              | Confirmed |
| 2    | Group A    | GenericWrite | Object B   | ACL output            | Confirmed |
| 3    | Object B   | Controls     | Computer C | Configuration         | Confirmed |
| 4    | Computer C | Admin Access | Target     | Controlled validation | Confirmed |

This makes incomplete paths easy to identify.

---

# 23. Path Status

Use clear status categories.

### Candidate

A potential path has been identified but not fully validated.

### Verified

The relevant configuration and relationships have been confirmed.

### Confirmed

The complete security-relevant path and resulting privilege have been demonstrated.

### Blocked

A security control or missing prerequisite prevents the path.

### Unverified

Available evidence is insufficient to determine whether the path works.

### Not Applicable

The path does not apply to the current environment or scope.

---

# 24. Blocked Paths

A blocked path is still useful information.

Example:

```text id="h7q3n8"
Candidate:
User → Delegation → Target

Validation:
Protected User restriction prevents delegation.

Result:
Blocked
```

Document the control responsible for blocking the path.

This can demonstrate the effectiveness of an existing security control.

---

# 25. Failed Validation

A failed validation does not necessarily mean the original enumeration was wrong.

Possible causes include:

* Stale data
* Configuration changes
* Network restrictions
* Account status
* Missing prerequisites
* Security controls
* Incorrect assumptions
* Tool interpretation

Document the exact reason when known.

---

# 26. Evidence Standards

Strong evidence should answer:

### Who?

```text
Which principal starts the path?
```

### What?

```text
What permission or relationship exists?
```

### Where?

```text
Which object or service is affected?
```

### How?

```text
How does the relationship create access?
```

### Result?

```text
What privilege is actually obtained?
```

---

# 27. Evidence Types

Useful evidence can include:

* PowerShell output
* LDAP query results
* AD object properties
* ACL output
* GPO configuration
* Certificate-template configuration
* Trust configuration
* BloodHound graph data
* Controlled validation results
* Screenshots where appropriate

Store evidence securely.

---

# 28. Evidence Sanitization

Before placing evidence in a public repository:

Replace:

```text
REALDOMAIN.LOCAL
```

with:

```text
EXAMPLE.LOCAL
```

Replace:

```text
real-user
```

with:

```text
user01
```

Replace real addresses with:

```text
10.10.10.10
```

Never publish:

* Passwords
* NTLM hashes
* Private keys
* Session tokens
* API keys
* Internal secrets
* Real customer infrastructure

---

# 29. Common False Positives

### Graph-Only Relationship

The graph identifies a relationship but direct validation fails.

### Stale Permission

The permission was removed after collection.

### Disabled Account

The source or target is disabled.

### Missing Scope

The GPO or certificate template does not apply to the expected principal.

### Security Control

Delegation, certificate, trust, or authentication controls block the path.

### Insufficient Authorization

Authentication succeeds but the expected privilege is absent.

---

# 30. Common Mistakes

Avoid:

* Treating discovery as validation
* Validating only the first step
* Ignoring intermediate relationships
* Ignoring account status
* Ignoring network reachability
* Ignoring authentication
* Ignoring authorization
* Over-exploiting production systems
* Collecting unnecessary sensitive data
* Publishing real assessment evidence

---

# 31. Assessment Checklist

### Preparation

* [ ] Define scope
* [ ] Define starting principal
* [ ] Define target
* [ ] Record current privileges

### Relationship Validation

* [ ] Verify users
* [ ] Verify groups
* [ ] Verify ACLs
* [ ] Verify GPOs
* [ ] Verify credentials
* [ ] Verify delegation
* [ ] Verify RBCD
* [ ] Verify AD CS
* [ ] Verify trusts

### Environment Validation

* [ ] Verify account status
* [ ] Verify network reachability
* [ ] Verify service availability
* [ ] Verify authentication
* [ ] Verify authorization
* [ ] Verify security controls

### Result

* [ ] Confirm effective privilege
* [ ] Identify blocked paths
* [ ] Identify unverified paths
* [ ] Collect evidence
* [ ] Document limitations

### Reporting

* [ ] Record complete attack path
* [ ] Record prerequisites
* [ ] Record evidence
* [ ] Sanitize evidence
* [ ] Separate candidate and confirmed paths

---

# Section 06 Completion

The attack-path analysis workflow is now complete:

```text id="m4x8q2"
Attack Path Analysis
        ↓
BloodHound
        ↓
Attack Path Identification
        ↓
Attack Path Validation
        ↓
Confirmed Security Path
```

The next phase is **Privilege Validation**.

Continue with:

**`07-Privilege-Validation/README.md`**

Section 07 will focus on proving the **actual privilege obtained**, validating privileged identities, confirming Domain Admin-level access where applicable, and collecting defensible evidence without unnecessary exploitation.
