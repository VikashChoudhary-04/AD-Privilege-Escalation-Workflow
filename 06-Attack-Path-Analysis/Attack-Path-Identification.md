# Attack Path Identification

## Purpose

Attack Path Identification converts individual Active Directory findings into **candidate privilege-escalation chains**.

The objective is not simply to find vulnerabilities. It is to determine how an initial principal can move through the environment using verified relationships.

The core workflow is:

```text id="4x9p2m"
Define Starting Principal
      ↓
Define Target
      ↓
Collect Relevant Relationships
      ↓
Build Candidate Paths
      ↓
Analyze Preconditions
      ↓
Validate Each Relationship
      ↓
Determine Effective Privilege
      ↓
Document Candidate / Confirmed Path
```

---

## Position in the Workflow

This file follows:

**`06-Attack-Path-Analysis/BloodHound.md`**

BloodHound and other enumeration methods help discover relationships.

This file focuses on turning those relationships into structured attack paths.

```text id="x7m4q9"
AD Enumeration
      ↓
BloodHound / Graph Analysis
      ↓
Relationships
      ↓
Attack Path Identification
      ↓
Attack Path Validation
```

---

## What Is an Attack Path?

An attack path is a sequence of relationships connecting an initial security principal to a target resource or privilege.

Example:

```text id="9k2v6s"
User A
  ↓
MemberOf
  ↓
Group B
  ↓
GenericWrite
  ↓
Computer C
  ↓
Local Administrator
```

The path consists of multiple relationships.

Each relationship must be understood and validated independently.

---

## Candidate vs Confirmed Path

This distinction is important.

### Candidate Path

A path discovered through:

* BloodHound
* LDAP
* PowerShell
* Manual enumeration
* Correlation

Example:

```text id="6m8r2p"
User
 ↓
GenericAll
 ↓
Computer
```

This is a candidate until the permission and resulting access are verified.

### Confirmed Path

A path where:

* Source is valid
* Permission is confirmed
* Target is correct
* Required conditions are satisfied
* Resulting privilege is demonstrated

```text id="q5n7x3"
Source
 ↓
Verified Relationship
 ↓
Verified Target
 ↓
Verified Operation
 ↓
Verified Privilege
```

---

# 1. Define the Starting Principal

Every path should begin with a clearly defined principal.

Record:

* Username
* Domain
* Account type
* Enabled state
* Current groups
* Current privileges
* Available credentials
* Accessible hosts

Example:

```text id="m8q3v7"
Initial Principal:
    analyst01@example.local

Current Privilege:
    Standard Domain User
```

Avoid assuming that every domain user has identical access.

---

# 2. Define the Target

Choose the resource or privilege being investigated.

Possible targets include:

* Domain Admins
* Enterprise Admins
* Administrators
* Domain Controller
* High-value server
* Local Administrator
* Privileged service account
* Sensitive application
* Specific AD object

Example:

```text id="r4p8y2"
Target:
    Domain Admins
```

A target should be concrete enough to validate.

---

# 3. Build the Relationship Inventory

Gather relevant relationships from previous sections.

### Membership

```text
User → Group
```

### ACL

```text
Principal → Permission → Object
```

### GPO

```text
Principal → GPO → Target
```

### Delegation

```text
Principal → Delegation → Service
```

### RBCD

```text
Principal → RBCD → Computer
```

### AD CS

```text
Principal → Template → Certificate → Identity
```

### Trust

```text
Domain → Trust → Domain
```

---

# 4. Normalize Relationships

Represent each relationship consistently.

A useful format is:

```text id="8f3m6k"
SOURCE
RELATIONSHIP
TARGET
CONDITION
IMPACT
```

Example:

```text id="5x9p3v"
User A
GenericWrite
Computer B
Permission currently effective
Can modify relevant configuration
```

This makes paths easier to construct.

---

# 5. Identify Direct Paths

A direct path contains very few relationships.

Example:

```text id="n6q4m8"
User
 ↓
MemberOf
 ↓
Privileged Group
```

Another:

```text id="z7p3r5"
User
 ↓
GenericAll
 ↓
Privileged Object
```

Direct paths should still be validated.

---

# 6. Identify Indirect Paths

Indirect paths contain multiple relationships.

Example:

```text id="w4k8s2"
User
 ↓
Group
 ↓
ACL
 ↓
Computer
 ↓
Local Administrator
```

Another:

```text id="j3m7q5"
User
 ↓
GPO Control
 ↓
GPO
 ↓
Server OU
 ↓
Server
 ↓
Administrative Access
```

The additional relationships create additional prerequisites.

---

# 7. Membership-Based Paths

Start with group membership.

Example:

```text id="r8v2n6"
User A
 ↓
MemberOf
 ↓
Group A
 ↓
MemberOf
 ↓
Group B
 ↓
MemberOf
 ↓
Privileged Group
```

Check:

* Direct membership
* Nested membership
* Group scope
* Domain
* Enabled state
* Effective membership

---

# 8. ACL-Based Paths

ACL paths can involve:

* GenericAll
* GenericWrite
* WriteDacl
* WriteOwner
* WriteProperty
* ForceChangePassword
* AddMember
* Object-specific permissions

Example:

```text id="p5x9m3"
User
 ↓
WriteDacl
 ↓
Group
 ↓
Membership Control
 ↓
Privileged Group
```

The exact permission must be confirmed.

---

# 9. Credential-Based Paths

Credentials discovered during earlier phases can create new attack-path starting points.

Example:

```text id="c7n4q8"
Initial User
 ↓
Credential Discovery
 ↓
Service Account
 ↓
Group Membership
 ↓
Privileged Resource
```

For every discovered credential, determine:

* Owner
* Account status
* Credential validity
* Privileges
* Reuse
* Scope

---

# 10. Password-Reuse Paths

Password reuse may connect multiple identities.

Example:

```text id="g2m6r9"
Credential
 ↓
User Account
 ↓
Service Account
 ↓
Privileged Group
```

Do not treat similar passwords as proof of reuse.

The authentication relationship must be validated within the authorized environment.

---

# 11. Kerberos-Based Paths

Kerberos relationships can create paths through:

* SPNs
* Service accounts
* Kerberoasting
* AS-REP Roasting
* Delegation

Example:

```text id="k8p3v6"
User
 ↓
Service Account
 ↓
SPN
 ↓
Authentication
 ↓
Privileged Service
```

The resulting privilege must be established separately.

---

# 12. GPO-Based Paths

GPO paths should include both control and scope.

Example:

```text id="q7x4n8"
User
 ↓
GPO Modification
 ↓
GPO
 ↓
Linked OU
 ↓
Computer
 ↓
Security-Relevant Policy
```

Validate:

* GPO permissions
* Link
* Security filtering
* WMI filtering
* Policy precedence
* Target computer

---

# 13. Delegation-Based Paths

Delegation paths should identify:

* Source
* Delegation type
* SPN
* Target
* User context
* Restrictions

Example:

```text id="v9m2r4"
Controlled Principal
 ↓
Delegation
 ↓
Target Service
 ↓
Target Account
 ↓
Effective Access
```

---

# 14. RBCD Paths

RBCD paths require particular attention to the target computer object.

Example:

```text id="x5q8k3"
User
 ↓
Computer Object Control
 ↓
RBCD Configuration
 ↓
Trusted Principal
 ↓
Target Computer
 ↓
Target Service
```

The complete permission chain must be validated.

---

# 15. AD CS Paths

Certificate-based paths can be represented as:

```text id="p8m4y6"
User
 ↓
Enrollment Permission
 ↓
Certificate Template
 ↓
Certificate
 ↓
Identity Mapping
 ↓
Privileged Identity
```

Check:

* Enrollment
* Template configuration
* EKU
* Subject/SAN
* Approval
* Certificate mapping
* Strong binding
* Resulting authorization

---

# 16. Trust-Based Paths

Cross-domain relationships can create paths such as:

```text id="h6r3q9"
Domain A User
 ↓
Trust
 ↓
Domain B
 ↓
Cross-Domain Group
 ↓
Privileged Resource
```

Validate:

* Trust direction
* Trust type
* Authentication scope
* Cross-domain membership
* ACL
* Effective privilege

---

# 17. Combine Relationship Types

The most interesting paths often combine several relationship types.

Example:

```text id="n3x7m5"
User
 ↓
Password Reuse
 ↓
Service Account
 ↓
MemberOf
 ↓
Privileged Group
```

Another:

```text id="q4v8k2"
User
 ↓
ACL Control
 ↓
Computer
 ↓
RBCD
 ↓
Target Service
```

Another:

```text id="s9p6r3"
User
 ↓
GPO Control
 ↓
Server
 ↓
Credential Discovery
 ↓
Privileged Account
```

Each transition must be independently validated.

---

# 18. Attack Path Preconditions

Every path should list its prerequisites.

Example:

```text id="7m5x9q"
Path:
User → GPO → Server

Prerequisites:
- User can modify GPO
- GPO is linked to target OU
- Security filtering permits application
- Target computer receives policy
- Relevant policy setting is effective
```

This prevents incomplete paths from being reported as confirmed findings.

---

# 19. Account State

Always verify:

* Enabled
* Disabled
* Locked
* Expired
* Password status
* Service availability

A path involving a disabled account may not be practically usable.

---

# 20. Network Reachability

An AD relationship does not necessarily mean the source can reach the target.

Consider:

* Network segmentation
* Firewall rules
* Routing
* Required ports
* VPN boundaries
* Administrative protocols

Example:

```text id="c5x8m2"
AD Permission
      ↓
Target Server
      ↓
Network Reachability
      ↓
Service Availability
```

---

# 21. Authentication Requirements

Determine what authentication mechanism each step requires.

Examples:

* NTLM
* Kerberos
* Certificate authentication
* Local authentication
* SMB
* LDAP
* WinRM
* RDP

A path should not assume authentication is possible simply because authorization appears possible.

---

# 22. Authorization vs Authentication

Keep these concepts separate.

### Authentication

```text
Who are you?
```

### Authorization

```text
What are you allowed to do?
```

Example:

```text id="w6k3p8"
Cross-Domain Trust
      ↓
Authentication
      ↓
Cross-Domain User
      ↓
ACL
      ↓
Authorization
```

Both stages should be validated.

---

# 23. Identify the Effective Privilege

At the end of every path, determine the actual privilege.

Examples:

```text
Local Administrator
Domain Administrator
Service Account Access
GPO Modification
Computer Object Control
Certificate-Based Authentication
Sensitive Server Access
```

Avoid using vague descriptions such as "full control" without explaining what that control means in context.

---

# 24. Path Dependencies

Some paths depend on previous steps.

Example:

```text id="j7q4n8"
Step 1
Credential Discovery
      ↓
Step 2
Account Access
      ↓
Step 3
Group Membership
      ↓
Step 4
Privileged Resource
```

If Step 1 fails, the rest of the path may be unavailable.

Document these dependencies explicitly.

---

# 25. Path Validation Matrix

Use a validation table:

| Step | Source    | Relationship | Target    | Evidence      | Status    |
| ---- | --------- | ------------ | --------- | ------------- | --------- |
| 1    | User A    | MemberOf     | Group A   | AD output     | Confirmed |
| 2    | Group A   | GenericWrite | Object B  | ACL output    | Confirmed |
| 3    | Object B  | Controls     | Service C | Configuration | Confirmed |
| 4    | Service C | Access       | Server D  | Validation    | Confirmed |

Possible status values:

* Candidate
* Verified
* Blocked
* Not Applicable
* Confirmed

---

# 26. Candidate Path Filtering

Remove paths that fail basic conditions.

Examples:

```text id="r5m8x3"
Disabled Source
        ↓
Remove

Nonexistent Target
        ↓
Remove

Read-Only Permission
        ↓
Reassess

Missing Required Configuration
        ↓
Remove / Reclassify
```

This keeps the final attack-path set meaningful.

---

# 27. Avoid Overlapping Paths

Multiple paths may lead to the same target.

Example:

```text id="v8q4k6"
User
 ├── Group Path ───────→ Target
 ├── ACL Path ─────────→ Target
 └── GPO Path ─────────→ Target
```

Document them separately when they represent materially different security conditions.

Avoid duplicating the same underlying relationship in multiple findings.

---

# 28. Attack Path Documentation Format

Use a consistent format:

```text id="y4n7p2"
### Path

User A
  ↓
Relationship
  ↓
Object B
  ↓
Relationship
  ↓
Target

### Preconditions

- Condition 1
- Condition 2

### Validation

- Evidence 1
- Evidence 2

### Result

Effective privilege or access.
```

This format makes paths easy to review.

---

# 29. Evidence Quality

Strong evidence should establish:

1. Source identity
2. Relationship
3. Target
4. Configuration
5. Effective result

For example:

```text id="m6x9r4"
Source
 ↓
ACL
 ↓
Target Object
 ↓
Configuration
 ↓
Validated Result
```

Avoid relying on a screenshot alone when direct configuration evidence is available.

---

# 30. Common False Positives

### Stale Graph Data

The relationship may have changed.

### Disabled Accounts

The source may no longer be usable.

### Missing Scope

The policy or permission may not apply to the intended target.

### Insufficient Permission

The relationship may provide less control than initially assumed.

### Security Restrictions

Delegation, certificate, trust, or authentication controls may block the path.

### Unreachable Target

The target may not be accessible from the current network position.

---

# 31. Common Mistakes

Avoid:

* Starting without defining the initial principal
* Starting without defining the target
* Treating every graph relationship as an attack path
* Ignoring intermediate dependencies
* Ignoring account status
* Ignoring network reachability
* Ignoring authentication requirements
* Ignoring authorization
* Ignoring inheritance
* Ignoring nested groups
* Ignoring security filtering
* Reporting candidate paths as confirmed findings

---

# 32. Assessment Checklist

### Define

* [ ] Identify starting principal
* [ ] Identify target
* [ ] Record current privilege
* [ ] Record available credentials

### Discover

* [ ] Review group membership
* [ ] Review ACLs
* [ ] Review GPOs
* [ ] Review credentials
* [ ] Review Kerberos
* [ ] Review delegation
* [ ] Review RBCD
* [ ] Review AD CS
* [ ] Review trusts

### Construct

* [ ] Build candidate paths
* [ ] Map every relationship
* [ ] Identify prerequisites
* [ ] Identify dependencies
* [ ] Identify target privilege

### Validate

* [ ] Verify source
* [ ] Verify relationship
* [ ] Verify target
* [ ] Verify current configuration
* [ ] Verify network reachability
* [ ] Verify authentication
* [ ] Verify authorization
* [ ] Verify effective privilege

### Document

* [ ] Record complete path
* [ ] Record evidence
* [ ] Record limitations
* [ ] Separate candidate and confirmed paths
* [ ] Sanitize sensitive information

---

## Transition

Continue with:

**`06-Attack-Path-Analysis/Attack-Path-Validation.md`**

The final file in Section 06 will establish a repeatable process for **proving or rejecting candidate attack paths**, separating configuration findings from confirmed privilege escalation.
