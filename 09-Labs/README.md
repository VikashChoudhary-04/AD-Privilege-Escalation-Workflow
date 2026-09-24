# Labs

## Purpose

The Labs section provides a practical environment for applying the Active Directory privilege escalation workflow.

The objective is to move from:

```text id="m7q4x9"
Theory
   ↓
Enumeration
   ↓
Attack Path Analysis
   ↓
Validation
   ↓
Reporting
```

into repeatable hands-on practice.

Labs should reinforce the workflow rather than become a collection of unrelated walkthroughs.

---

# 1. Role of the Labs Section

The main workflow explains **how to approach an assessment**.

The Labs section demonstrates how that workflow is applied to controlled environments.

A typical lab should follow:

```text id="x8n3m6"
Scope
 ↓
Initial Access
 ↓
Enumeration
 ↓
Identify Weakness
 ↓
Construct Attack Path
 ↓
Validate Path
 ↓
Validate Privilege
 ↓
Collect Evidence
 ↓
Document Finding
 ↓
Remediation
```

---

# 2. Authorized Lab Environment

All activities in this section should be performed only against:

* Intentionally vulnerable environments
* Personally controlled systems
* Authorized training platforms
* Authorized assessment environments

Do not apply lab techniques to systems without explicit authorization.

---

# 3. Lab Objectives

Each lab should have clearly defined objectives.

Examples:

* Enumerate an Active Directory domain
* Identify users and groups
* Analyze ACL relationships
* Identify Kerberos attack paths
* Investigate delegation
* Analyze AD CS configuration
* Identify GPO weaknesses
* Construct an attack path
* Validate effective privilege
* Document evidence
* Produce a remediation recommendation

---

# 4. Recommended Lab Structure

Each lab should follow a consistent structure.

```text id="q6m8v3"
Lab Name

Objective

Environment

Scope

Initial Access

Enumeration

Potential Weaknesses

Attack Path

Validation

Privilege Validation

Evidence

Finding

Remediation

Lessons Learned
```

This mirrors the main repository workflow.

---

# 5. Lab Metadata

Record basic information.

Example:

```text id="v4m7x9"
Lab:
Active Directory ACL Abuse

Platform:
Authorized AD Lab

Difficulty:
Intermediate

Focus:
ACL Enumeration and Privilege Escalation

Status:
Completed
```

Use accurate platform and difficulty information.

---

# 6. Environment

Document the relevant lab infrastructure.

Example:

```text id="m8x3q6"
Domain:
CORP.LOCAL

Domain Controller:
DC01

Windows Client:
CLIENT01

Attacker Machine:
Linux VM
```

Use sanitized values where appropriate.

---

# 7. Scope

Clearly define what is included.

Example:

```text id="n5q8m4"
In Scope:
- DC01
- CLIENT01
- CORP.LOCAL

Out of Scope:
- Host systems
- External networks
```

This reinforces proper assessment boundaries.

---

# 8. Initial Access

Document how the lab begins.

Possible starting points:

* Standard domain account
* Authorized test credentials
* Initial foothold
* Controlled machine access

Example:

```text id="x7m4q8"
Starting Identity:
user01

Initial Privilege:
Standard Domain User
```

Do not publish real credentials.

---

# 9. Enumeration

Apply the enumeration workflow from:

**`03-Enumeration/`**

Relevant areas may include:

* Network enumeration
* Domain enumeration
* User enumeration
* Group enumeration
* Computer enumeration
* Share enumeration
* GPO enumeration
* ACL enumeration
* SPN enumeration
* Trust enumeration
* Delegation enumeration

Record only the information relevant to the lab objective.

---

# 10. Credential and Authentication Analysis

Apply the workflow from:

**`04-Credentials-and-Authentication/`**

Possible lab topics:

* Credential discovery
* Password reuse
* Kerberoasting
* AS-REP Roasting
* Credential dumping

The lab should explain why a discovered credential or authentication weakness matters.

---

# 11. Privilege Escalation Path

Apply the workflow from:

**`05-Privilege-Escalation-Paths/`**

Possible paths include:

```text id="p9m4x7"
Group Membership
ACL Abuse
GPO Abuse
Delegation
RBCD
AD CS
Trust Relationships
```

The path should be constructed from observed relationships.

---

# 12. Attack Path Analysis

Use:

**`06-Attack-Path-Analysis/`**

Identify:

* Starting principal
* Target
* Intermediate objects
* Relationships
* Preconditions
* Dependencies
* Candidate paths

Example:

```text id="w6q8m3"
User
 ↓
Group
 ↓
ACL
 ↓
Privileged Group
 ↓
Domain-Level Privilege
```

---

# 13. Path Validation

Do not stop at identifying a potential path.

Apply the validation process:

```text id="r7x3m9"
Candidate
   ↓
Relationship Verification
   ↓
Precondition Verification
   ↓
Authorization Validation
   ↓
Confirmed / Blocked
```

This is one of the most important purposes of the lab section.

---

# 14. Privilege Validation

Apply:

**`07-Privilege-Validation/`**

Determine exactly what was obtained.

Examples:

* Local Administrator
* Object-level control
* Server-level administrative access
* Domain-level administrative access
* Domain Admin membership

Do not report a broader privilege than the evidence supports.

---

# 15. Evidence Collection

For each lab, preserve relevant evidence.

Examples:

```text id="q4m8x7"
Enumeration Output
ACL Output
Group Membership
GPO Configuration
BloodHound Graph
Validation Output
Runtime Context
```

Evidence should be sufficient to reproduce the conclusion.

---

# 16. Finding Documentation

Apply:

**`08-Reporting/Findings.md`**

Each significant validated weakness should be converted into a structured finding.

Example:

```text id="v8n3m6"
Finding:
Excessive ACL Permissions

Status:
Confirmed

Affected Object:
PrivilegedGroup

Impact:
Validated unauthorized administrative control
```

---

# 17. Remediation

Every appropriate lab should consider how the weakness could be fixed.

Use:

**`08-Reporting/Remediation.md`**

Document:

* Root cause
* Corrective action
* Expected secure state
* Validation method

---

# 18. Lessons Learned

Each lab should finish with practical observations.

Examples:

* Which enumeration technique revealed the relationship?
* Which assumption initially appeared misleading?
* Which precondition mattered?
* What evidence confirmed the path?
* What prevented a false positive?
* Which remediation would break the path?

The purpose is to improve methodology, not simply record commands.

---

# 19. Lab Difficulty

Use difficulty consistently.

Example:

| Difficulty | Description                                           |
| ---------- | ----------------------------------------------------- |
| Easy       | Single weakness with a straightforward path           |
| Medium     | Multiple enumeration steps or relationships           |
| Hard       | Multiple dependencies or chained attack paths         |
| Advanced   | Complex AD relationships requiring extensive analysis |

Difficulty should describe the complexity of the lab, not the perceived importance of the vulnerability.

---

# 20. Lab Completion Criteria

A lab should not be considered complete merely because access was obtained.

A complete lab should ideally include:

* [ ] Scope identified
* [ ] Initial identity documented
* [ ] Enumeration performed
* [ ] Weakness identified
* [ ] Attack path constructed
* [ ] Path validated
* [ ] Privilege validated
* [ ] Evidence collected
* [ ] Finding documented
* [ ] Remediation considered
* [ ] Lessons learned recorded

---

# 21. Lab Naming

Use descriptive names.

Examples:

```text id="m3q7x9"
01-ACL-Abuse
02-Kerberoasting
03-GPO-Abuse
04-RBCD
05-ADCS
```

The numbering is for organization within the Labs section and does not represent the main assessment workflow.

---

# 22. Lab Documentation Example

A completed lab can follow this high-level format:

```text id="x6m4q8"
# Lab Name

## Objective

## Environment

## Scope

## Initial Access

## Enumeration

## Identified Weakness

## Attack Path

## Path Validation

## Privilege Validation

## Evidence

## Finding

## Remediation

## Lessons Learned
```

Keep each lab focused on its primary learning objective.

---

# 23. Avoid Copying the Main Workflow

The Labs section should not duplicate every command from the main sections.

Instead:

```text id="n8q5m3"
Main Workflow
    ↓
Explains Method

Lab
    ↓
Demonstrates Method
```

Link back to relevant workflow files where appropriate.

---

# 24. Lab-to-Workflow Mapping

A lab can explicitly identify which sections it exercises.

Example:

| Lab Activity         | Workflow Section                                    |
| -------------------- | --------------------------------------------------- |
| Domain Enumeration   | `03-Enumeration/Domain-Enumeration.md`              |
| ACL Analysis         | `03-Enumeration/ACL-Enumeration.md`                 |
| ACL Abuse            | `05-Privilege-Escalation-Paths/ACL-Abuse.md`        |
| BloodHound Analysis  | `06-Attack-Path-Analysis/BloodHound.md`             |
| Path Validation      | `06-Attack-Path-Analysis/Attack-Path-Validation.md` |
| Privilege Validation | `07-Privilege-Validation/`                          |
| Finding              | `08-Reporting/Findings.md`                          |
| Remediation          | `08-Reporting/Remediation.md`                       |

This keeps practical exercises connected to the overall methodology.

---

# 25. Evidence and Privacy

Even lab environments may contain sensitive information.

Do not publish:

* Real credentials
* Private keys
* Tokens
* Unnecessary personal information
* Credentials reused outside the lab
* Sensitive infrastructure details

Use sanitized examples when publishing the repository.

---

# 26. Lab Reproducibility

A good lab should provide enough information for the exercise to be understood again.

Document:

* Environment
* Starting point
* Required tools
* Relevant configuration
* Important commands
* Validation method
* Expected result

Do not depend entirely on screenshots.

---

# 27. Common Lab Documentation Mistakes

Avoid:

* Recording only successful exploitation
* Skipping enumeration reasoning
* Skipping path validation
* Treating tool output as proof
* Omitting preconditions
* Claiming Domain Admin without validation
* Publishing credentials
* Ignoring remediation
* Copying unrelated commands
* Writing walkthroughs without explaining why each step matters

---

# 28. Lab Checklist

### Preparation

* [ ] Authorized environment
* [ ] Scope defined
* [ ] Objective defined
* [ ] Environment documented

### Assessment

* [ ] Initial access documented
* [ ] Enumeration performed
* [ ] Weakness identified
* [ ] Attack path constructed
* [ ] Preconditions identified

### Validation

* [ ] Path validated
* [ ] Resulting privilege validated
* [ ] False positives considered
* [ ] Evidence collected

### Reporting

* [ ] Finding documented
* [ ] Impact documented
* [ ] Remediation documented
* [ ] Remediation validation defined
* [ ] Lessons learned recorded

---

# 29. Recommended Lab Progression

A practical progression can move from individual relationships to complete attack chains:

```text id="q7m4x8"
Basic Enumeration
       ↓
Group / ACL Analysis
       ↓
Kerberos Attacks
       ↓
GPO / Delegation
       ↓
RBCD
       ↓
AD CS
       ↓
Trust Relationships
       ↓
Multi-Step Attack Paths
       ↓
Complete Validation + Reporting
```

The exact order can be adjusted according to the lab environment.

---

## Transition

The Labs section is intentionally designed as a practical container rather than another theoretical workflow stage.

The repository's final supporting section is:

**`References/README.md`**

This will document the authoritative resources, documentation, standards, tools, and references used to support the workflow.
