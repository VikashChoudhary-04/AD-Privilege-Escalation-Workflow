# Attack Path Documentation

## Purpose

Attack-path documentation explains how multiple Active Directory relationships can be combined to move from an initial principal to a validated security outcome.

The objective is to make the chain understandable and reproducible:

```text id="f7m3q9"
Starting Principal
       ↓
Relationship
       ↓
Permission / Credential / Configuration
       ↓
Intermediate Principal
       ↓
Additional Relationship
       ↓
Target Privilege
       ↓
Validation
       ↓
Evidence
```

An attack path should describe what was **actually validated**, not merely what a graph or enumeration tool suggests.

---

# 1. Attack Path vs Finding

These concepts should remain distinct.

### Finding

Describes a security weakness.

### Attack Path

Describes how one or more weaknesses and relationships combine into a security outcome.

Example:

```text id="m4x8q2"
Finding AD-001
Weak ACL on Group A

Finding AD-002
Group A controls Group B

Attack Path
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Group
```

One attack path may contain multiple findings.

---

# 2. Define the Starting Principal

Every attack path must have a clear starting point.

Possible starting principals include:

* Domain user
* Service account
* Computer account
* Compromised account
* Authorized test account

Document:

```text id="q7n3v8"
Principal:
user01

Domain:
CORP.LOCAL

Account Type:
Domain User
```

Use sanitized identities in public documentation.

---

# 3. Define the Target

The target should represent the security outcome being investigated.

Examples:

* Privileged group
* Domain Admins
* Domain Controller
* Sensitive server
* Protected AD object
* GPO
* Certificate authority
* Forest/domain resource

Example:

```text id="x5m9q4"
Target:
Domain Admins

Target Type:
Privileged Security Group
```

---

# 4. Establish the Initial State

Record the privileges already available to the starting principal.

Examples:

```text id="v8q4m6"
Initial Access:
Standard domain user

Existing Group Membership:
Domain Users

Known Privileges:
No administrative group membership
```

This establishes a baseline.

---

# 5. Map the First Relationship

Identify the first meaningful transition.

Examples:

```text id="n6x3q8"
User
 ↓
MemberOf
 ↓
Group
```

or:

```text id="r4m8v2"
User
 ↓
WriteDacl
 ↓
Group
```

or:

```text id="p7q5x9"
User
 ↓
Credential Access
 ↓
Privileged Account
```

The relationship should be explicitly identified.

---

# 6. Document Every Transition

Do not skip intermediate steps.

Weak documentation:

```text id="j8m4q6"
User → Domain Admin
```

Better documentation:

```text id="c5x8n3"
User
 ↓
Group A
 ↓
WriteDacl
 ↓
Group B
 ↓
Membership Control
 ↓
Domain Admins
```

Each transition should be independently understood.

---

# 7. Attack Path Components

A complete path may contain:

### Identity

The principal performing the action.

### Relationship

The permission or relationship connecting two objects.

### Target

The object affected by the relationship.

### Preconditions

Conditions required for the transition.

### Result

The privilege or access obtained.

---

# 8. Common Relationship Types

Attack paths may include:

* MemberOf
* GenericAll
* GenericWrite
* WriteDacl
* WriteOwner
* ForceChangePassword
* AddMember
* Owns
* Controls
* GPO-related permissions
* Delegation relationships
* Certificate-related relationships
* Trust relationships

Only include relationships that are relevant to the validated path.

---

# 9. Group Membership Path

Example:

```text id="w7m3q9"
User
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

Document:

* Group names
* Group nesting
* Scope
* Current membership
* Resulting authorization

---

# 10. ACL-Based Path

Example:

```text id="m6q8x4"
User
 ↓
WriteDacl
 ↓
Privileged Group
 ↓
Membership Modification
 ↓
Privileged Authorization
```

Document:

* Source principal
* ACL permission
* Target object
* Resulting capability
* Validation

The ACL relationship alone does not prove the final privilege.

---

# 11. GPO-Based Path

Example:

```text id="x8v4m7"
User
 ↓
GPO Modification Permission
 ↓
GPO
 ↓
Linked OU
 ↓
Affected Systems
```

Document:

* GPO
* Permissions
* Link
* Scope
* Affected objects
* Validation result

---

# 12. Credential-Based Path

Example:

```text id="q4n7m8"
Initial User
 ↓
Credential Discovery
 ↓
Privileged Account
 ↓
Authentication
 ↓
Privileged Access
```

Document:

* Credential source
* Account
* Account status
* Authentication context
* Resulting authorization

Do not publish actual credentials.

---

# 13. Kerberos-Based Path

For Kerberos-related paths, document the relevant relationship.

Example:

```text id="v6m3q9"
Account
 ↓
SPN / Kerberos Relationship
 ↓
Credential Material
 ↓
Account Authentication
 ↓
Authorization
```

Record only the evidence necessary to establish the finding.

---

# 14. Delegation Path

Example:

```text id="p8x5m3"
Principal
 ↓
Delegation Configuration
 ↓
Target Service
 ↓
Authentication Context
 ↓
Authorized Resource
```

Document:

* Source
* Target
* Delegation type
* Required conditions
* Resulting identity
* Validation

---

# 15. RBCD Path

Example:

```text id="k7q4x9"
Controlled Principal
 ↓
RBCD Configuration
 ↓
Target Computer
 ↓
Service Authentication
 ↓
Resulting Access
```

Document the relevant computer accounts and configuration without exposing unnecessary secrets.

---

# 16. AD CS Path

Example:

```text id="m9v4q6"
Principal
 ↓
Certificate Enrollment
 ↓
Certificate Template
 ↓
Certificate
 ↓
Identity Mapping
 ↓
Authentication
 ↓
Authorization
```

Record:

* Template
* Enrollment permission
* Certificate configuration
* Identity mapping
* Authentication result
* Final authorization

---

# 17. Trust-Based Path

Example:

```text id="x3m8q5"
Source Domain
 ↓
Trust
 ↓
Cross-Domain Authorization
 ↓
Target Group / ACL
 ↓
Protected Resource
```

Document:

* Source domain
* Target domain
* Trust direction
* Trust type
* Authorization relationship
* Final access

A trust relationship by itself is not proof of privilege.

---

# 18. Attack Path Preconditions

Each transition should identify required conditions.

Example:

```text id="n5q8x4"
Preconditions:
- Starting account is enabled
- ACL is currently present
- Target group is active
- Network connectivity exists
- Required authentication is possible
```

This prevents the path from being presented as universally exploitable.

---

# 19. Authentication vs Authorization

Keep these concepts separate.

```text id="w4m7q9"
Authentication
     ↓
Who are you?

Authorization
     ↓
What are you allowed to do?
```

A successful login does not automatically prove administrative privilege.

---

# 20. Effective Privilege

Document the final effective privilege.

Examples:

```text id="v8q3m6"
Local Administrator
```

```text id="r6x4n9"
Object-Level Control
```

```text id="m7q5x8"
Domain-Level Administrative Access
```

```text id="p4n8v3"
Domain Admins Membership
```

Use the most precise description supported by evidence.

---

# 21. Candidate vs Confirmed Path

Clearly distinguish path status.

### Candidate

Potential relationship identified.

### Verified

Relevant relationship or configuration confirmed.

### Confirmed

The resulting security consequence was validated.

### Blocked

A required condition prevented the path.

### Unverified

Available evidence was insufficient.

Example:

```text id="x9m4q7"
Path Status:
Confirmed

Reason:
All relationships were verified and the resulting authorization was validated.
```

---

# 22. Attack Path Evidence

Each transition should have supporting evidence where practical.

| Step | Relationship       | Evidence          | Status    |
| ---- | ------------------ | ----------------- | --------- |
| 1    | MemberOf           | AD output         | Verified  |
| 2    | WriteDacl          | ACL output        | Verified  |
| 3    | Membership Control | Validation output | Confirmed |
| 4    | Privileged Access  | Runtime context   | Confirmed |

This creates traceability.

---

# 23. BloodHound Path Documentation

BloodHound can provide a visual representation of the path.

Example:

```text id="q5m8x3"
User
 ↓
Group
 ↓
ACL
 ↓
Privileged Group
```

Record:

* Collection date
* Starting node
* Target node
* Relevant edges
* Query or analysis used

BloodHound should support, not replace, manual validation.

---

# 24. Attack Path Dependencies

Some paths depend on other conditions.

Example:

```text id="m8q3v6"
Path A
  ↓
Requires Group Membership
  ↓
Group Membership
  ↓
Requires ACL Permission
```

Document these dependencies so reviewers understand the complete chain.

---

# 25. Multiple Paths to the Same Target

Several paths may reach the same privilege.

Example:

```text id="x7n4m9"
                 ┌── ACL Abuse ───────┐
User ────────────┼── Group Membership ┼──→ Privileged Target
                 └── Credential Path ──┘
```

Document separate paths when their prerequisites or root causes differ.

---

# 26. Shortest Path vs Most Relevant Path

A graph may contain many possible paths.

Do not automatically treat the shortest path as the only meaningful path.

Consider:

* Validation status
* Preconditions
* Reliability
* Scope
* Evidence quality
* Root cause

The report should focus on paths that are relevant and supported by evidence.

---

# 27. Blocked Attack Paths

Blocked paths should be documented when they materially affect the assessment.

Example:

```text id="n6m4x8"
Path:
User → Group → Privileged Resource

Status:
Blocked

Reason:
Required permission was not present.
```

This helps distinguish attempted analysis from successful validation.

---

# 28. Attack Path Summary Format

A concise report format:

```text id="p9x3m7"
Path ID:
AP-001

Starting Principal:
user01

Target:
Domain Admins

Status:
Confirmed

Path:
user01
  ↓
WriteDacl
  ↓
PrivilegedGroup
  ↓
Membership Control
  ↓
Domain Admins

Impact:
Validated domain-level administrative authorization.

Evidence:
- ACL output
- Group membership
- Runtime context
- Validation output
```

---

# 29. Mapping Paths to Findings

Use explicit references.

Example:

```text id="v4q8m6"
AP-001
 ├── AD-001 Excessive ACL
 ├── AD-002 Privileged Group Control
 └── AD-003 Domain-Level Impact
```

This allows reviewers to move between:

```text
Finding → Attack Path → Evidence
```

---

# 30. Attack Path Documentation Checklist

### Starting Point

* [ ] Starting principal identified
* [ ] Initial privilege documented
* [ ] Domain identified

### Path

* [ ] Target identified
* [ ] Every relationship documented
* [ ] Intermediate objects identified
* [ ] Preconditions documented
* [ ] Dependencies documented

### Validation

* [ ] Each relationship verified
* [ ] Final privilege validated
* [ ] Authentication and authorization distinguished
* [ ] Path status assigned

### Evidence

* [ ] Evidence mapped to path
* [ ] BloodHound data timestamped
* [ ] Direct validation included
* [ ] Sensitive information sanitized

### Reporting

* [ ] Path ID assigned
* [ ] Related findings linked
* [ ] Impact documented
* [ ] Limitations documented

---

# 31. Final Attack Path Review

Before including a path in the final report, verify:

```text id="q7m5x8"
Starting Identity
      ↓
Current Relationship
      ↓
Required Preconditions
      ↓
Intermediate Object
      ↓
Next Relationship
      ↓
Target
      ↓
Validated Authorization
      ↓
Evidence
```

Every important transition should be supported by evidence.

---

## Transition

The final file in Section 08 is:

**`08-Reporting/Remediation.md`**

It will define how to translate validated Active Directory findings into practical remediation actions and how to verify that those remediations actually remove the identified attack path.
