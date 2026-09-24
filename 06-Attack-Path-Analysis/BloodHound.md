# BloodHound

## Purpose

BloodHound is a graph-based tool for analyzing relationships within Active Directory environments.

It helps security assessors visualize relationships between:

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
* Other AD security relationships

BloodHound is primarily an **attack-path discovery and relationship-analysis tool**.

It should not be treated as proof that a displayed path is automatically exploitable.

The core workflow is:

```text id="w5j8q2"
Collect AD Data
      ↓
Import Data
      ↓
Understand Nodes
      ↓
Understand Relationships
      ↓
Find Candidate Paths
      ↓
Verify Relationships
      ↓
Validate Effective Privilege
      ↓
Document Attack Path
```

---

## Position in the Workflow

BloodHound belongs to:

```text id="5v8r3k"
06-Attack-Path-Analysis
```

The previous file established the general attack-path methodology.

This file focuses specifically on using BloodHound to discover and analyze those relationships.

---

## What BloodHound Does

BloodHound converts Active Directory relationships into a graph.

Conceptually:

```text id="3j7m5x"
AD Objects
    ↓
Collection
    ↓
Graph Database
    ↓
Relationships
    ↓
Path Analysis
```

Instead of manually reading thousands of individual relationships, the graph makes it easier to answer questions such as:

* What can this user access?
* Which groups can this account reach?
* Which computers can this user administer?
* Which objects does this principal control?
* What relationships connect an initial account to a privileged target?

---

## What BloodHound Does Not Prove

A graph relationship does not necessarily prove:

* The account is enabled
* The permission is currently effective
* The target is reachable
* The service is running
* The configuration has not changed
* The path works in the current environment
* The resulting privilege is actually obtainable

Therefore:

```text id="k6q4w9"
BloodHound
    ↓
Candidate Path
    ↓
Manual Verification
    ↓
Confirmed Path
```

---

## 1. BloodHound Architecture

A simplified architecture is:

```text id="n4r8x2"
Active Directory
      ↓
Collector
      ↓
Collected Data
      ↓
BloodHound
      ↓
Graph Database
      ↓
Graph Queries
      ↓
Attack Paths
```

Different BloodHound generations and collectors may use different components and data formats.

Always follow the documentation for the specific version being used.

---

## 2. Data Collection

BloodHound-compatible collectors gather AD information.

Common data categories include:

* Domain objects
* Users
* Groups
* Computers
* Group memberships
* ACLs
* Sessions
* Trusts
* GPO relationships
* Local group membership
* SPNs
* Delegation-related properties

Collect only the data required for the authorized assessment.

---

## 3. Collector Types

Common collection approaches include:

* SharpHound
* BloodHound-compatible collectors
* LDAP-based collection
* Remote collection
* Session collection
* Local group collection

The appropriate collector depends on:

* Operating system
* Current privileges
* Network position
* Domain configuration
* Assessment scope

---

## 4. SharpHound

SharpHound is a BloodHound-compatible data collector for Windows environments.

A basic example may look like:

```powershell id="m9f2v4"
SharpHound.exe -c All
```

Collection options depend on the version and environment.

Other collection categories can be selected when full collection is unnecessary.

Examples include:

```text id="q4x7n1"
All
Default
Session
ACL
Group
Trusts
ObjectProps
```

Always confirm available collection methods in the version being used.

---

## 5. Collection Scope

Do not automatically collect everything.

Consider:

* Assessment authorization
* Network size
* Data sensitivity
* Required analysis
* Collection noise
* Runtime
* Storage requirements

A targeted collection can be preferable when only a specific attack path is being investigated.

---

## 6. Sensitive Data Considerations

BloodHound data can contain information about:

* Users
* Computers
* Groups
* Sessions
* Administrative relationships
* Security permissions
* Internal infrastructure

Treat collected data as sensitive assessment material.

Do not publish:

* Real usernames
* Internal hostnames
* Domain information
* Session information
* Credentials
* Private infrastructure details

in a public GitHub repository.

Use sanitized examples instead.

---

# Graph Fundamentals

## 7. Nodes

A node represents an AD object or security entity.

Common nodes include:

```text id="2h7m8p"
User
Group
Computer
Domain
OU
GPO
Certificate Template
```

Each node contains properties describing the object.

---

## 8. Edges

An edge represents a relationship.

Examples include:

```text id="7x3k9m"
MemberOf
AdminTo
HasSession
GenericAll
GenericWrite
WriteDacl
WriteOwner
ForceChangePassword
Owns
Contains
TrustedBy
```

Exact relationships vary by BloodHound version and collector.

---

## 9. User Nodes

User nodes can contain information such as:

* Username
* Domain
* Enabled status
* Group membership
* SPNs
* Delegation properties
* Administrative relationships

The user node is often the starting point for attack-path analysis.

---

## 10. Group Nodes

Group nodes represent AD security groups.

Important properties include:

* Group scope
* Members
* Nested groups
* Privileged membership
* ACL relationships

Example:

```text id="g3q8w5"
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

---

## 11. Computer Nodes

Computer nodes represent domain-joined systems.

Useful relationships include:

* Local administrator access
* Sessions
* Group membership
* Delegation
* SPNs
* ACL control
* GPO relationships

A computer node should be analyzed together with its local and domain privilege context.

---

## 12. Domain Nodes

Domain nodes provide the broader AD security boundary.

They help identify:

* Domain membership
* Trust relationships
* Domain administrators
* Cross-domain relationships

Example:

```text id="h7v2m9"
Domain A
   ↓
Trust
   ↓
Domain B
```

---

## 13. OU Nodes

Organizational Units help explain:

* Object placement
* GPO links
* Inheritance
* Administrative delegation

Example:

```text id="j5q3x8"
OU
 ↓
GPO
 ↓
Computer
```

---

## 14. GPO Nodes

GPO relationships can expose:

* GPO control
* GPO links
* Target OUs
* Policy scope

Example:

```text id="w2n6r4"
User
 ↓
GPO Control
 ↓
GPO
 ↓
OU
 ↓
Computer
```

The actual policy must still be inspected to determine security impact.

---

## 15. ACL Relationships

ACL edges are particularly important for privilege escalation.

Common relationships include:

```text id="8r4k2p"
GenericAll
GenericWrite
WriteDacl
WriteOwner
WriteProperty
ForceChangePassword
AddMember
Owns
```

The exact effect depends on:

* Object type
* Permission
* Inheritance
* Existing groups
* Other security controls

---

# Attack Path Discovery

## 16. Starting Node

Begin with the principal currently controlled in the assessment.

Example:

```text id="q6y8t2"
USER01@EXAMPLE.LOCAL
```

Then determine reachable objects.

---

## 17. Target Node

Common target nodes include:

* Domain Admins
* Enterprise Admins
* Domain Controllers
* Administrators
* High-value servers
* Sensitive service accounts

The target should be defined before interpreting a path.

---

## 18. Shortest Path Analysis

A graph can identify a short path between two nodes.

Conceptually:

```text id="p8x5n4"
Initial User
    ↓
Group
    ↓
Computer
    ↓
Target
```

A short graph path is useful for discovery but is not automatically the most practical path.

Verify every edge.

---

## 19. Shortest Path to a Privileged Group

A common analysis question is:

```text id="m4j7s2"
What relationships connect User A to Domain Admins?
```

The graph may reveal:

```text id="x9v3k6"
User A
 ↓
Group A
 ↓
Group B
 ↓
Domain Admins
```

Validate:

* Current membership
* Group nesting
* Account status
* Scope
* Effective authorization

---

## 20. Shortest Path to a Computer

Another useful query is:

```text id="y5r8n1"
What path connects User A to Server01?
```

The path may involve:

```text id="v6k2p7"
User
 ↓
Group
 ↓
AdminTo
 ↓
Server
```

Confirm that the administrative relationship is still valid.

---

## 21. Attack Paths Through ACLs

A graph may identify:

```text id="g8w4m3"
User
 ↓
GenericAll
 ↓
Computer
```

Manual validation should determine:

* Exact ACE
* Inheritance
* Object type
* Effective permission
* Target service
* Resulting privilege

---

## 22. Attack Paths Through Groups

Example:

```text id="s7p4x9"
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

Check nested membership carefully.

A group can contain another group, so effective privileges may be several levels deep.

---

## 23. Attack Paths Through Sessions

A session relationship can identify a potentially useful host.

Conceptually:

```text id="r6x8m2"
User
 ↓
HasSession
 ↓
Computer
```

The existence of a session does not itself provide administrative access.

Analyze the user's privileges on that computer separately.

---

## 24. Attack Paths Through Delegation

Delegation relationships can be represented as:

```text id="k3m7q1"
Principal
 ↓
Delegation
 ↓
Target Service
 ↓
Target Account
```

Verify:

* Delegation type
* Source
* Target
* SPN
* Restrictions
* Effective privilege

---

## 25. Attack Paths Through AD CS

Where AD CS data is available:

```text id="c4y8n6"
User
 ↓
Certificate Enrollment
 ↓
Certificate Template
 ↓
Certificate
 ↓
Identity
 ↓
Privilege
```

Certificate mapping and modern security controls must be validated separately.

---

## 26. Attack Paths Through Trusts

Cross-domain paths may look like:

```text id="t5v9p2"
Domain A User
 ↓
Trust
 ↓
Domain B
 ↓
Group / ACL
 ↓
Privileged Resource
```

Verify:

* Trust direction
* Trust type
* Authentication scope
* Authorization
* Cross-domain permissions

---

# Query and Analysis Workflow

## 27. Start Broad

Begin with broad relationship discovery:

```text id="e3q7x9"
Who am I?
      ↓
What groups am I in?
      ↓
What can those groups access?
      ↓
What computers can I reach?
      ↓
What objects can I control?
```

Then narrow the investigation.

---

## 28. Identify Interesting Nodes

Look for:

* Privileged groups
* Domain controllers
* High-value computers
* Administrative accounts
* Service accounts
* Certificate authorities
* Sensitive GPOs

This reduces unnecessary graph complexity.

---

## 29. Follow the Edges

For each interesting relationship:

```text id="w8m4r6"
Source
 ↓
Relationship
 ↓
Target
```

Ask:

1. Is the relationship current?
2. Is it effective?
3. Can the source actually use it?
4. What does the target provide?

---

## 30. Validate Outside the Graph

BloodHound should not be the sole source of truth.

Validate important relationships with:

* PowerShell
* LDAP
* Active Directory administrative tools
* Windows security configuration
* GPO inspection
* Certificate configuration
* Controlled testing

Example:

```text id="j6p2s8"
BloodHound
    ↓
GenericAll Relationship
    ↓
Get-Acl / AD Inspection
    ↓
ACE Confirmed
```

---

## 31. Stale Data

BloodHound data represents the environment at collection time.

Between collection and validation:

* Users may be disabled
* Groups may change
* ACLs may change
* Computers may be removed
* Sessions may end
* GPOs may change
* Trusts may change

Therefore:

```text id="u7x3n5"
Old Graph Data
      ↓
Current AD Verification
```

is required for important findings.

---

## 32. Graph Data and Evidence

Use BloodHound for:

* Discovery
* Relationship visualization
* Path identification
* Attack-chain reasoning

Use direct AD evidence for:

* Permission verification
* Current configuration
* Account status
* GPO configuration
* Certificate configuration
* Final validation

---

## 33. Attack Path Documentation

For every confirmed path, record:

```text id="a9k5m7"
Starting Principal
      ↓
Relationship 1
      ↓
Object
      ↓
Relationship 2
      ↓
Object
      ↓
Final Privilege
```

Then record evidence for every important edge.

---

## 34. Common False Positives

### Stale Relationship

The graph may contain outdated information.

### Disabled Account

The source or target may no longer be active.

### Incorrect Scope

A relationship may not apply to the current environment or target.

### Missing Security Control

The graph may not model every environmental control.

### Read Access

A relationship may provide visibility but not control.

### Authentication Restriction

Delegation, certificate, or trust paths may be blocked by security settings.

---

## 35. Common Mistakes

Avoid:

* Treating every graph edge as exploitable
* Using stale data as current evidence
* Ignoring account status
* Ignoring inheritance
* Ignoring security filtering
* Ignoring delegation restrictions
* Ignoring certificate mapping
* Ignoring trust controls
* Focusing only on shortest paths
* Publishing real BloodHound data
* Treating BloodHound output as a substitute for validation

---

## 36. Assessment Checklist

### Collection

* [ ] Confirm authorization
* [ ] Select appropriate collector
* [ ] Define collection scope
* [ ] Collect required AD data
* [ ] Protect collected data

### Graph Analysis

* [ ] Identify starting principal
* [ ] Identify target
* [ ] Review groups
* [ ] Review ACLs
* [ ] Review computers
* [ ] Review sessions
* [ ] Review GPOs
* [ ] Review delegation
* [ ] Review trusts
* [ ] Review AD CS where supported

### Validation

* [ ] Confirm relationship
* [ ] Confirm current state
* [ ] Confirm permissions
* [ ] Confirm target
* [ ] Confirm effective privilege
* [ ] Collect evidence

### Documentation

* [ ] Record complete path
* [ ] Record prerequisites
* [ ] Record evidence
* [ ] Record limitations
* [ ] Sanitize sensitive data

---

## Transition

Continue with:

**`06-Attack-Path-Analysis/Attack-Path-Identification.md`**

The next file will focus on systematically turning the relationships discovered during enumeration and BloodHound analysis into **specific candidate privilege-escalation paths**.
