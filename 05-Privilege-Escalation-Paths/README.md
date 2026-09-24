# Privilege Escalation Paths

## Purpose

This section converts the information collected during enumeration and credential analysis into concrete Active Directory privilege-escalation paths.

The objective is to determine:

* Which principals have control over other principals or resources
* Which permissions can change security relationships
* Which delegated rights can affect privileged objects
* Which configurations can enable privilege escalation
* How multiple low-impact findings can combine into a meaningful path
* Whether a suspected path can be validated within authorized scope

The core principle is:

```text id="g3w8p2"
Enumeration
    │
    ▼
Relationships
    │
    ▼
Permissions / Credentials
    │
    ▼
Potential Path
    │
    ▼
Validation
    │
    ▼
Confirmed Impact
```

A permission or configuration should not automatically be classified as exploitable.

---

## Position in the Workflow

The complete workflow is:

```text id="n7x4q1"
01-Foundations
      │
      ▼
02-First-5-Minutes
      │
      ▼
03-Enumeration
      │
      ▼
04-Credentials-and-Authentication
      │
      ▼
05-Privilege-Escalation-Paths  ← Current
      │
      ▼
06-Attack-Path-Analysis
      │
      ▼
07-Privilege-Validation
      │
      ▼
08-Reporting
```

Section 03 identified the environment.

Section 04 identified authentication and credential relationships.

Section 05 investigates how those relationships can produce privilege escalation.

---

## Privilege Escalation Model

A useful AD privilege-escalation model is:

```text id="w2m9c6"
Principal
    │
    ▼
Permission / Credential / Configuration
    │
    ▼
Controlled Object
    │
    ▼
New Privilege
    │
    ▼
Next Principal / Resource
    │
    ▼
Higher Privilege
```

For example:

```text id="r6p3x8"
Low-Privilege User
       │
       ▼
Control Over Group
       │
       ▼
Privileged Group Membership
       │
       ▼
Administrative Access
```

The actual path must be established through evidence.

---

## Privilege Escalation Workflow

```text id="m8q4v2"
Review Known Credentials
        │
        ▼
Review Group Relationships
        │
        ▼
Review ACL Relationships
        │
        ▼
Review GPO Control
        │
        ▼
Review Delegation
        │
        ▼
Review RBCD
        │
        ▼
Review ADCS
        │
        ▼
Review Trust Relationships
        │
        ▼
Build Potential Paths
        │
        ▼
Validate Authorized Paths
        │
        ▼
Document Impact
```

The individual technique files provide detailed analysis for each category.

---

## 1. Group Membership Abuse

**File:** `Group-Membership-Abuse.md`

Groups are one of the fundamental privilege-management mechanisms in Active Directory.

A privilege path may exist when a controlled account can modify membership of a group that grants additional privileges.

Conceptually:

```text id="y5k2r7"
Controlled Account
       │
       ▼
Group Membership Control
       │
       ▼
Privileged Group
       │
       ▼
Additional Access
```

Important questions include:

* Who can modify the group?
* Which members are currently present?
* What privileges does the group provide?
* Is the group nested into another privileged group?
* Is the permission direct or inherited?

Do not assume that a group named `Admins` or similar is automatically highly privileged. Establish its actual permissions.

---

## 2. ACL Abuse

**File:** `ACL-Abuse.md`

Active Directory Access Control Lists determine which principals can perform operations against objects.

A path may exist when a controlled principal has a permission that allows modification of a security-sensitive object.

Conceptually:

```text id="k4m8x2"
Principal
    │
    ▼
ACL Permission
    │
    ▼
AD Object
    │
    ▼
Security-Relevant Modification
    │
    ▼
Privilege Increase
```

Relevant objects can include:

* Users
* Groups
* Computers
* GPOs
* Service accounts
* Organizational Units

The important workflow is:

```text
Permission
    │
    ▼
Object
    │
    ▼
Allowed Operation
    │
    ▼
Security Impact
```

An ACL finding should therefore identify the exact permission and affected object.

---

## 3. GPO Abuse

**File:** `GPO-Abuse.md`

Group Policy Objects can influence the configuration of users and computers across an Active Directory environment.

A potential privilege path may exist when an unauthorized principal can modify a GPO that applies to privileged systems or accounts.

Conceptually:

```text id="v7p2m9"
Controlled Principal
       │
       ▼
GPO Modification Rights
       │
       ▼
GPO
       │
       ▼
Affected OU / Computer / User
       │
       ▼
Security-Relevant Configuration
```

Assessment should establish:

* Who can modify the GPO?
* Where is it linked?
* Which systems receive the policy?
* What configuration does it control?
* Does the modification affect a privileged resource?

The existence of GPO write access alone does not establish the final impact.

---

## 4. Delegation Abuse

**File:** `Delegation-Abuse.md`

Delegation allows services to authenticate on behalf of users under defined conditions.

Section 03 identified delegation configurations.

This section examines whether those relationships can create a privilege-escalation path.

Relevant configurations include:

```text id="a4c8w2"
Unconstrained Delegation
Constrained Delegation
Protocol Transition
Delegation Relationships
```

The analysis should connect:

```text
Delegating Principal
        │
        ▼
Delegation Configuration
        │
        ▼
Target Service
        │
        ▼
Target Computer
        │
        ▼
Privilege Impact
```

A configured delegation relationship should not automatically be considered exploitable.

---

## 5. Resource-Based Constrained Delegation

**File:** `RBCD.md`

Resource-Based Constrained Delegation allows a target resource to specify which principals can act on behalf of users to services hosted by that resource.

The key attribute is:

```text
msDS-AllowedToActOnBehalfOfOtherIdentity
```

The important direction is:

```text id="q8m4x7"
Target Computer
      │
      │ RBCD Configuration
      ▼
Allowed Principal
```

The assessment should determine:

* Which target computer has the configuration?
* Which principal is permitted?
* Who controls that principal?
* Who controls the target computer object?
* What services are hosted by the target?
* What privileges result from the relationship?

RBCD should be analyzed as a relationship rather than as an isolated attribute.

---

## 6. ADCS

**File:** `ADCS.md`

Active Directory Certificate Services can introduce privilege-escalation paths when certificate templates, enrollment permissions, issuance settings, or CA configurations allow inappropriate certificate-based authentication.

The assessment should investigate:

```text id="t5n7c3"
Certificate Authority
        │
        ▼
Certificate Template
        │
        ▼
Enrollment / Modification Permissions
        │
        ▼
Certificate Identity
        │
        ▼
Authentication
        │
        ▼
Privilege Impact
```

Important areas include:

* Certificate Authorities
* Certificate templates
* Enrollment permissions
* Template configuration
* Subject/SAN configuration
* EKUs
* CA permissions
* Administrative control

A certificate template being available does not automatically mean that it provides privilege escalation.

---

## 7. Trust Abuse

**File:** `Trust-Abuse.md`

Trust relationships can connect security boundaries between domains or forests.

Section 03 identified:

* Trust direction
* Trust type
* Transitivity
* Trusted domains
* Trusting domains

Section 05 investigates whether those relationships create a meaningful privilege path.

Conceptually:

```text id="z6q2w8"
Domain A
   │
   │ Trust
   ▼
Domain B
   │
   ▼
Resource / Principal
   │
   ▼
Privilege Impact
```

The assessment should establish the exact authentication and authorization relationship before concluding that a trust creates escalation.

---

## Privilege Path Categories

A useful classification is:

| Category         | Primary Relationship                |
| ---------------- | ----------------------------------- |
| Group Abuse      | Principal → Group                   |
| ACL Abuse        | Principal → Object                  |
| GPO Abuse        | Principal → GPO → Computer/User     |
| Delegation Abuse | Principal → Delegation → Service    |
| RBCD             | Principal → Target Computer         |
| ADCS             | Principal → Certificate Template/CA |
| Trust Abuse      | Domain → Trust → Resource           |

This keeps the investigation structured.

---

## Path Construction

Each potential path should be represented as a chain.

```text id="u8m5q2"
SOURCE
  │
  ▼
CONTROL / PERMISSION
  │
  ▼
OBJECT
  │
  ▼
SECURITY-RELEVANT ACTION
  │
  ▼
NEW ACCESS
  │
  ▼
TARGET
```

Example:

```text id="p4x7n8"
User A
  │
  ▼
WriteMember Permission
  │
  ▼
Group B
  │
  ▼
Privileged Group Membership
  │
  ▼
Administrative Resource
```

This format makes the reasoning behind a finding easy to follow.

---

## Direct vs Chained Paths

Not every privilege-escalation path is direct.

### Direct Path

```text id="g6w3r9"
User
  │
  ▼
Permission
  │
  ▼
Privileged Resource
```

### Chained Path

```text id="c5m8x2"
User
  │
  ▼
Object A
  │
  ▼
Group B
  │
  ▼
Computer C
  │
  ▼
Service Account D
  │
  ▼
Privileged Resource
```

Chained paths are especially important in AD because several individually moderate permissions can combine into a significant security relationship.

---

## Credential-to-Privilege Correlation

Credential findings from Section 04 should be connected to privilege relationships.

```text id="v9q4m6"
Recovered Credential
        │
        ▼
Account
        │
        ├── Group Membership
        ├── ACL Permissions
        ├── SPNs
        ├── Delegation
        └── GPO Access
                │
                ▼
        Potential Privilege Path
```

This is where credential discovery becomes privilege-path analysis.

---

## Object-Control Model

For each suspected path, determine:

```text id="h7m2p5"
Who controls the source?
        │
        ▼
What permission exists?
        │
        ▼
What object is controlled?
        │
        ▼
What action is possible?
        │
        ▼
What privilege changes?
        │
        ▼
What resource becomes accessible?
```

This model should be applied consistently across all Section 05 techniques.

---

## Validate the Permission

Before treating a permission as exploitable, confirm:

* The permission exists
* The principal actually possesses it
* The permission applies to the intended object
* The object is active
* The affected resource is in scope
* The proposed operation is technically possible
* The resulting privilege is measurable

For example:

```text id="j4w8q3"
ACL Found
   │
   ▼
Principal Confirmed
   │
   ▼
Object Confirmed
   │
   ▼
Permission Confirmed
   │
   ▼
Operation Confirmed
   │
   ▼
Impact Confirmed
```

---

## Attack-Path Evidence

For every potential path, record:

```text id="q2m6x8"
Source Principal:
Source Privilege:
Permission / Relationship:
Target Object:
Allowed Action:
Resulting Access:
Target Resource:
Validation:
Evidence:
```

Example:

```text id="r8v3k5"
Source Principal:
user-a

Permission:
Modify Group Membership

Target:
group-b

Resulting Access:
Membership in privileged group

Validation:
Authorized test performed

Evidence:
ACL + group membership + resulting authorization
```

Use sanitized account names in public documentation.

---

## Common Path-Analysis Mistakes

### Mistake 1: Looking at Permissions in Isolation

A permission matters only in relation to the object it affects.

### Mistake 2: Ignoring Object Ownership

Ownership can provide different control than ordinary ACL permissions.

### Mistake 3: Ignoring Inheritance

An inherited permission may affect more objects than initially expected.

### Mistake 4: Assuming Group Names Determine Privilege

Determine actual permissions rather than relying on names.

### Mistake 5: Treating Potential Paths as Confirmed

A relationship should be validated before being reported as a confirmed privilege escalation.

### Mistake 6: Ignoring Chained Relationships

Several moderate permissions may combine into a significant path.

### Mistake 7: Ignoring Scope

Only in-scope systems, accounts, and resources should be validated.

---

## Privilege Escalation Checklist

### Group Membership

* [ ] Identified privileged groups
* [ ] Identified group modification permissions
* [ ] Identified nested groups
* [ ] Validated membership impact

### ACLs

* [ ] Enumerated relevant ACLs
* [ ] Identified controlling principals
* [ ] Identified affected objects
* [ ] Identified exact permissions
* [ ] Validated security impact

### GPOs

* [ ] Identified GPO modification rights
* [ ] Identified GPO links
* [ ] Identified affected systems
* [ ] Validated resulting configuration impact

### Delegation

* [ ] Reviewed unconstrained delegation
* [ ] Reviewed constrained delegation
* [ ] Reviewed protocol transition
* [ ] Mapped delegation relationships
* [ ] Validated potential impact

### RBCD

* [ ] Identified RBCD configurations
* [ ] Identified target computers
* [ ] Identified allowed principals
* [ ] Correlated object-control permissions
* [ ] Validated potential impact

### ADCS

* [ ] Enumerated CAs
* [ ] Enumerated certificate templates
* [ ] Reviewed enrollment permissions
* [ ] Reviewed template configuration
* [ ] Correlated certificate permissions with principals
* [ ] Validated potential authentication impact

### Trusts

* [ ] Identified trust relationships
* [ ] Identified trust direction
* [ ] Identified transitivity
* [ ] Identified affected domains
* [ ] Correlated cross-domain permissions
* [ ] Validated potential impact

### Path Analysis

* [ ] Built source-to-target paths
* [ ] Identified direct paths
* [ ] Identified chained paths
* [ ] Correlated credentials
* [ ] Correlated groups
* [ ] Correlated ACLs
* [ ] Correlated delegation
* [ ] Correlated GPOs
* [ ] Distinguished potential paths from confirmed findings

---

## Transition to Attack-Path Analysis

After individual privilege-escalation mechanisms have been investigated, the next stage is to analyze how multiple relationships combine into complete attack paths.

```text id="m6q8v3"
05-Privilege-Escalation-Paths
              │
              ▼
       Individual Paths
              │
              ├── Group
              ├── ACL
              ├── GPO
              ├── Delegation
              ├── RBCD
              ├── ADCS
              └── Trust
              │
              ▼
06-Attack-Path-Analysis
              │
              ├── BloodHound
              ├── Attack-Path-Identification
              └── Attack-Path-Validation
```

The next file is:

**`05-Privilege-Escalation-Paths/Group-Membership-Abuse.md`**
