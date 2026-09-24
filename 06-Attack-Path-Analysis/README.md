# Attack Path Analysis

## Purpose

Attack Path Analysis brings together the information collected during the earlier stages of the Active Directory assessment.

Instead of analyzing individual findings in isolation, this section focuses on answering:

> How can the current principal move from its existing level of access to a more privileged resource or identity?

An AD privilege-escalation path is usually a **chain of relationships**, not a single vulnerability.

The core workflow is:

```text id="qj7k4p"
Collect Findings
      ↓
Normalize Objects and Relationships
      ↓
Build Attack Graph
      ↓
Identify Candidate Paths
      ↓
Prioritize Relevant Paths
      ↓
Validate Each Relationship
      ↓
Confirm Effective Privilege
      ↓
Document Complete Path
```

---

## Position in the Workflow

Section 06 follows:

* Foundations
* First 5 Minutes
* Enumeration
* Credentials and Authentication
* Privilege Escalation Paths

The earlier sections identify individual security relationships.

This section connects them.

```text id="5m0p7r"
Enumeration
     ↓
Credentials
     ↓
ACLs / GPOs / Delegation / AD CS / Trusts
     ↓
Attack Path Analysis
     ↓
Validated Privilege Path
```

---

## What Is an Attack Path?

An attack path is a sequence of relationships that connects an initial principal to a target privilege or resource.

Example:

```text id="e4k7v2"
User A
  ↓
MemberOf
  ↓
Group B
  ↓
GenericAll
  ↓
Computer C
  ↓
Local Administrator
  ↓
Privileged Access
```

Each relationship must be understood individually.

A graph showing a path does not by itself prove that the path is exploitable.

---

## Attack Path vs Individual Finding

An individual finding describes one security condition.

Example:

```text id="8m3n6c"
User A
   ↓
WriteDacl
   ↓
Object B
```

An attack path connects that condition with additional relationships:

```text id="3f8p5w"
User A
   ↓
WriteDacl
   ↓
Object B
   ↓
Privilege-Relevant Permission
   ↓
Object C
   ↓
Higher Access
```

The second representation provides much more useful security context.

---

## Attack Graph Model

An Active Directory environment can be represented as a graph:

```text id="7x2n9v"
              ┌───────────┐
              │   User    │
              └─────┬─────┘
                    │
                 MemberOf
                    │
                    ▼
              ┌───────────┐
              │   Group   │
              └─────┬─────┘
                    │
                 ACL / Access
                    │
                    ▼
              ┌───────────┐
              │ Computer  │
              └─────┬─────┘
                    │
                 Service
                    │
                    ▼
              ┌───────────┐
              │ Privilege │
              └───────────┘
```

Objects are nodes.

Relationships are edges.

Privilege escalation occurs when a sequence of valid relationships connects the starting principal to a higher-value target.

---

## 1. Define the Starting Principal

Before analyzing paths, identify the starting context.

Record:

* Username
* Domain
* Account type
* Group membership
* Current privileges
* Local privileges
* Accessible systems
* Credentials available
* Service access

Example:

```text id="7j3f8q"
Initial Principal:
    user01@example.local

Current Context:
    Domain User

Known Access:
    Workstation01
    FileShare01
```

The starting point must be clearly defined.

---

## 2. Define the Target

An attack path must have a meaningful destination.

Possible targets include:

* Domain Admin
* Enterprise Admin
* Privileged group
* Domain controller
* High-value server
* Sensitive application
* Local Administrator
* Service account
* Other protected resource

Do not assume that every path must end at Domain Admin.

A path to a sensitive server or privileged application can also be security-relevant.

---

## 3. Build the Object Inventory

Create a normalized inventory of relevant objects.

### Users

```text
User
├── Username
├── Domain
├── Groups
├── SPNs
├── Delegation
└── ACL Relationships
```

### Groups

```text
Group
├── Members
├── Scope
├── Nested Groups
└── ACL Relationships
```

### Computers

```text
Computer
├── Hostname
├── OS
├── OU
├── SPNs
├── Delegation
└── ACL Relationships
```

### Other Objects

Also consider:

* GPOs
* OUs
* Certificate templates
* CAs
* Domains
* Trusts
* Service accounts

---

## 4. Normalize Relationships

Different tools may describe the same relationship differently.

Normalize findings into categories such as:

```text id="9f1c6x"
Membership
ACL Control
GPO Control
Credential Access
Kerberos
Delegation
Certificate Enrollment
Trust
Local Privilege
Remote Access
```

This makes relationships easier to compare.

---

## 5. Identify High-Value Nodes

Not every AD object has equal security significance.

High-value nodes may include:

* Domain Controllers
* Domain Admins
* Enterprise Admins
* Administrators
* Server administrators
* Critical servers
* Certificate Authorities
* Highly privileged service accounts
* Sensitive applications

The purpose is not to assign arbitrary scores, but to understand which nodes materially affect the environment.

---

## 6. Identify Candidate Paths

Candidate paths can be discovered using:

* BloodHound-compatible graph analysis
* LDAP enumeration
* PowerShell
* ACL analysis
* Group membership analysis
* Manual correlation

Example:

```text id="h8g3y4"
User
 ↓
Group
 ↓
Computer
 ↓
Local Admin
```

Another:

```text id="n5q2t8"
User
 ↓
ACL Control
 ↓
GPO
 ↓
Server
 ↓
Administrative Access
```

These are candidates until each relationship is verified.

---

## 7. BloodHound

BloodHound is a graph-based AD relationship analysis platform.

It helps visualize relationships between:

* Users
* Groups
* Computers
* Domains
* OUs
* GPOs
* ACLs
* Sessions
* Trusts
* Delegation

The important principle is:

```text id="3k9w6r"
BloodHound
    ↓
Find Relationships
    ↓
Analyst
    ↓
Validate Relationships
```

BloodHound should support analysis rather than replace manual verification.

---

## 8. Data Collection

BloodHound-compatible collectors can gather AD relationship data.

Common collection categories include:

* Group membership
* Local groups
* Sessions
* ACLs
* Trusts
* Object properties
* GPO relationships
* Computer information

The exact collection methods depend on the collector and environment.

Collect only what is authorized and necessary.

---

## 9. Graph Nodes

Important graph nodes include:

### User

Represents an AD user account.

### Group

Represents an AD security group.

### Computer

Represents a domain-joined machine.

### Domain

Represents an AD domain.

### GPO

Represents a Group Policy Object.

### OU

Represents an organizational unit.

### Certificate Template

Represents an AD CS certificate template where supported by the collector.

---

## 10. Graph Relationships

Examples of relationship categories include:

```text id="5x7r2m"
MemberOf
AdminTo
HasSession
GenericAll
GenericWrite
WriteDacl
WriteOwner
ForceChangePassword
AddMember
Owns
Contains
AllowedToDelegate
TrustedBy
```

Exact relationship names can vary by BloodHound version and data collector.

Always verify the underlying AD permission or configuration before reporting a relationship as exploitable.

---

## 11. Membership Paths

Group membership is one of the simplest attack-path relationships.

Example:

```text id="g8v4y1"
User
 ↓
MemberOf
 ↓
Group
 ↓
MemberOf
 ↓
Privileged Group
```

Nested groups must be included when calculating effective privilege.

---

## 12. ACL Paths

ACL relationships can create direct or indirect control.

Example:

```text id="7y5n4c"
User
 ↓
GenericAll
 ↓
Computer
```

Or:

```text id="p4m7k8"
User
 ↓
WriteDacl
 ↓
Object
 ↓
Additional Permission
 ↓
Security-Relevant Operation
```

The exact permission and object type must be validated.

---

## 13. Credential Paths

Credentials discovered in Section 04 can become graph edges.

Example:

```text id="9k2x6m"
Credential
 ↓
Account
 ↓
Group Membership
 ↓
Resource
```

A credential should therefore be correlated with:

* Account status
* Group membership
* Password reuse
* SPNs
* Delegation
* ACLs

---

## 14. Kerberos Paths

Kerberos-related relationships can include:

* SPNs
* Kerberoasting candidates
* AS-REP Roasting candidates
* Delegation
* Service accounts

Example:

```text id="w4r6t2"
User
 ↓
Service Account
 ↓
SPN
 ↓
Kerberos Authentication
 ↓
Privileged Service
```

Each step must be verified.

---

## 15. Delegation Paths

Delegation can create paths such as:

```text id="m3q7s8"
Controlled Principal
      ↓
Delegation
      ↓
Target Service
      ↓
Target Account
      ↓
Effective Privilege
```

Analyze:

* Delegation type
* Source
* Target
* SPN
* Account restrictions
* Target privilege

---

## 16. GPO Paths

GPO-based paths may look like:

```text id="x7j2n5"
User
 ↓
GPO Control
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

* GPO permission
* GPO link
* Security filtering
* WMI filtering
* Effective policy
* Target privilege

---

## 17. AD CS Paths

Certificate-based paths can be represented as:

```text id="z4c8m1"
User
 ↓
Certificate Enrollment
 ↓
Certificate Template
 ↓
Certificate
 ↓
Identity Mapping
 ↓
Privileged Identity
```

The complete certificate-to-identity path must be confirmed.

---

## 18. Trust Paths

Cross-domain paths may look like:

```text id="v6p3y9"
Domain A User
      ↓
Trust
      ↓
Domain B
      ↓
Cross-Domain Group / ACL
      ↓
Privileged Resource
```

Analyze:

* Trust direction
* Trust type
* Authentication scope
* Group membership
* ACLs
* SID-related controls

---

## 19. Chained Attack Paths

Real environments may contain several relationship types in a single path.

Example:

```text id="n7k5s3"
User
 ↓
Password Reuse
 ↓
Service Account
 ↓
Group Membership
 ↓
ACL Control
 ↓
Computer
 ↓
Delegation
 ↓
Target Service
```

Each transition should be independently validated.

Do not report a chain simply because a graph visualization connects the objects.

---

## 20. Path Validation

For every candidate path, ask:

```text id="x9m4p7"
Is the Source Valid?
        ↓
Is the Relationship Valid?
        ↓
Is the Permission Effective?
        ↓
Is the Target Reachable?
        ↓
Does the Operation Work?
        ↓
Does the Resulting Privilege Exist?
```

A path is confirmed only when its relevant relationships and resulting access are demonstrated.

---

## 21. Direct vs Indirect Paths

### Direct

```text id="d6k8q1"
User
 ↓
Privileged Group
```

### Indirect

```text id="h5r3x7"
User
 ↓
Group
 ↓
ACL
 ↓
Computer
 ↓
Local Admin
```

Indirect paths often require more careful validation because each additional relationship introduces another dependency.

---

## 22. Shortest Path vs Practical Path

A graph may identify a short path, but the shortest path is not automatically the only relevant path.

A practical path may depend on:

* Credentials
* Network reachability
* Account status
* Service availability
* Security controls
* Delegation restrictions
* Human approval
* Environmental configuration

Therefore, analyze the **actual prerequisites** for each path.

---

## 23. Path Preconditions

Document prerequisites explicitly.

Example:

```text id="g4x6m2"
Path:
User → GPO → Server

Prerequisites:
- User can modify GPO
- GPO is linked to Server OU
- Security filtering permits application
- Target server is online
- Policy setting affects target
```

This makes the analysis reproducible.

---

## 24. Effective Privilege

Do not stop when the path reaches a computer or group.

Determine what privilege actually results.

For example:

```text id="r7n3v5"
Computer
 ↓
Local Administrators
 ↓
Local Administrator Privilege
```

or:

```text id="s2k8y6"
Group
 ↓
Domain Admins
 ↓
Domain-Level Administrative Privilege
```

The final authorization context matters.

---

## 25. Attack Path Validation Table

A useful working table is:

| Step | Source    | Relationship | Target    | Verified |
| ---- | --------- | ------------ | --------- | -------- |
| 1    | User A    | MemberOf     | Group A   | Yes      |
| 2    | Group A   | GenericWrite | Object B  | Yes      |
| 3    | Object B  | Controls     | Service C | Yes      |
| 4    | Service C | Access       | Server D  | Yes      |
| 5    | Server D  | Privilege    | Target E  | Yes      |

This makes gaps in the path visible.

---

## 26. Evidence Correlation

For each relationship, associate evidence.

Example:

```text id="f7q2w9"
Relationship:
User A → GenericWrite → Object B

Evidence:
- LDAP enumeration
- PowerShell ACL output
- BloodHound graph
```

Use multiple sources where practical.

Graph evidence is useful for discovery; directory data and controlled validation establish the actual configuration.

---

## 27. Avoiding False Paths

Common reasons a graph path may fail include:

* Disabled account
* Removed group membership
* Stale collector data
* Incorrect permissions
* Inheritance differences
* Security filtering
* Protected accounts
* Network restrictions
* Service unavailable
* Certificate mapping protections
* Delegation restrictions

Always validate current state before reporting.

---

## 28. Path Prioritization

When several candidate paths exist, organize them based on objective factors such as:

* Current validity
* Number of prerequisites
* Required privileges
* Target sensitivity
* Reliability of evidence
* Scope
* Business impact

This is analysis, not a numerical security score.

Document why a path requires fewer or more dependencies without assigning arbitrary rankings.

---

## 29. Attack Path Documentation

Each confirmed path should contain:

```text id="w5j7k3"
Starting Principal
        ↓
Step 1
        ↓
Step 2
        ↓
Step 3
        ↓
Target Privilege
```

Then document:

* Preconditions
* Evidence
* Validation
* Security impact
* Remediation

---

## 30. Common Mistakes

Avoid:

* Treating graph edges as proof of exploitation
* Ignoring stale data
* Ignoring disabled accounts
* Ignoring inheritance
* Ignoring nested groups
* Ignoring security filtering
* Ignoring authentication restrictions
* Ignoring target privileges
* Ignoring network reachability
* Stopping before confirming effective privilege
* Reporting incomplete chains as confirmed vulnerabilities

---

## 31. Assessment Checklist

### Starting Point

* [ ] Identify current principal
* [ ] Record current privileges
* [ ] Record available credentials
* [ ] Identify accessible systems

### Graph Construction

* [ ] Enumerate users
* [ ] Enumerate groups
* [ ] Enumerate computers
* [ ] Enumerate ACLs
* [ ] Enumerate GPOs
* [ ] Enumerate delegation
* [ ] Enumerate AD CS
* [ ] Enumerate trusts

### Path Analysis

* [ ] Identify candidate paths
* [ ] Map every relationship
* [ ] Identify prerequisites
* [ ] Identify target privilege
* [ ] Check account status
* [ ] Check security controls

### Validation

* [ ] Verify each relationship
* [ ] Verify current configuration
* [ ] Verify target
* [ ] Verify resulting privilege
* [ ] Collect evidence
* [ ] Document complete path

---

## Transition

This section establishes the methodology for analyzing complete AD attack paths.

Continue with:

**`06-Attack-Path-Analysis/BloodHound.md`**

The next file focuses specifically on using **BloodHound-compatible data and graph analysis** to discover, understand, and validate Active Directory attack paths.
