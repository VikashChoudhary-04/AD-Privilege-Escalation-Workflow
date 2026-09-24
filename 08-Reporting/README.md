# Reporting

## Purpose

The Reporting phase converts validated technical results into clear, evidence-based security findings.

The objective is to communicate:

```text
Validated Technical Result
        ↓
Security Finding
        ↓
Business / Technical Impact
        ↓
Evidence
        ↓
Attack Path
        ↓
Remediation
```

A report should describe what was actually validated without overstating the severity, scope, or impact.

---

## Position in the Workflow

Reporting is the final major stage of the workflow:

```text
Foundations
    ↓
First 5 Minutes
    ↓
Enumeration
    ↓
Credentials & Authentication
    ↓
Privilege Escalation Paths
    ↓
Attack Path Analysis
    ↓
Privilege Validation
    ↓
Reporting
```

The reporting phase uses evidence collected during **Section 07**.

---

## Reporting Principles

A good report should be:

* Accurate
* Evidence-based
* Reproducible
* Clear
* Concise
* Actionable
* Properly scoped
* Sanitized

The report should distinguish between:

```text
Candidate
Verified
Confirmed
Blocked
Unverified
Not Applicable
```

Do not report a candidate relationship as a confirmed compromise.

---

# 1. Start With Validated Findings

Only create a formal finding when sufficient evidence exists.

Example:

```text
Candidate Path
      ↓
Validation
      ↓
Evidence
      ↓
Finding
```

Enumeration results alone may identify potential weaknesses but do not necessarily establish exploitable privilege.

---

# 2. Define the Finding

Each finding should answer:

```text
What is wrong?
Where is it?
Who can abuse it?
What can it affect?
How was it validated?
What evidence supports it?
How should it be fixed?
```

---

# 3. Finding Structure

A practical finding can contain:

| Field             | Purpose                        |
| ----------------- | ------------------------------ |
| Finding ID        | Unique identifier              |
| Title             | Concise description            |
| Severity          | Risk classification            |
| Affected Assets   | Systems/objects involved       |
| Description       | What was identified            |
| Technical Details | How it works                   |
| Attack Path       | How access can progress        |
| Impact            | Resulting security consequence |
| Evidence          | Supporting proof               |
| Remediation       | Recommended corrective action  |
| References        | Supporting resources           |
| Limitations       | Important constraints          |

---

# 4. Finding IDs

Use consistent identifiers.

Example:

```text id="x4m7q9"
AD-001
AD-002
AD-003
```

The identifier should remain consistent across:

* Finding
* Evidence
* Screenshots
* Attack-path documentation
* Remediation
* Final report

---

# 5. Finding Titles

Titles should describe the technical issue.

Good examples:

```text id="p8v3m6"
Excessive ACL Permissions on Privileged Group

Weak GPO Permissions Allow Unauthorized Policy Modification

Unrestricted Certificate Enrollment Enables Privileged Authentication
```

Avoid vague titles such as:

```text
Active Directory Issue
Security Problem
Critical AD Bug
```

---

# 6. Affected Assets

Identify exactly what is affected.

Examples:

* Domain
* Domain Controller
* User account
* Security group
* Computer
* GPO
* Certificate Template
* File Share

Example:

```text id="q5n8x3"
Affected Domain:
CORP.LOCAL

Affected Object:
PrivilegedGroup

Affected Account:
user01
```

Sanitize real environments before publication.

---

# 7. Technical Description

Explain the underlying issue.

A useful structure is:

```text id="m7x4p9"
Current Configuration
        ↓
Security Weakness
        ↓
Available Relationship
        ↓
Validated Consequence
```

Keep the description factual.

---

# 8. Attack Path Documentation

For AD privilege escalation, attack paths are often central to the finding.

Example:

```text id="v6q3m8"
Initial User
    ↓
ACL Permission
    ↓
Privileged Group
    ↓
Group Membership Control
    ↓
Domain Admins
```

Document each transition.

---

# 9. Impact

Impact should describe the validated consequence.

Possible areas include:

* Unauthorized privilege
* Unauthorized access
* Domain-level administrative control
* Access to sensitive systems
* Credential exposure
* Persistence opportunities
* Lateral movement opportunities
* Policy manipulation

Do not claim an impact that was not demonstrated or reasonably established from the validated configuration.

---

# 10. Severity

Severity should be based on the assessment methodology being used.

Consider factors such as:

* Privilege gained
* Scope
* Number of affected systems
* Exploitability
* Preconditions
* Business sensitivity
* Existing security controls
* Required attacker access

The methodology should be stated consistently throughout the report.

---

# 11. Evidence

Each finding should contain evidence sufficient to support the claim.

Examples:

```text id="k8m4x7"
Directory Configuration
Group Membership
ACL Output
GPO Configuration
BloodHound Path
Runtime Context
Validation Result
```

Evidence should be directly connected to the finding.

---

# 12. Evidence References

Use predictable references.

Example:

```text id="n5q8v2"
Evidence:
- finding-01-acl-validation.txt
- finding-01-group-membership.txt
- finding-01-runtime-context.txt
```

This allows reviewers to trace the finding back to the original evidence.

---

# 13. Attack Path vs Finding

These are related but different.

### Finding

Describes a security weakness.

### Attack Path

Describes how multiple relationships can be combined.

Example:

```text id="w7m3q9"
Finding A:
Weak ACL on Group A

Finding B:
Group A controls Group B

Attack Path:
User → Group A → Group B → Privileged Group
```

Several findings may contribute to one attack path.

---

# 14. Avoid Duplicate Findings

Multiple observations may originate from the same root cause.

Example:

```text id="r4x8m6"
Weak ACL
   ├── Group Membership Abuse
   ├── Privileged Access
   └── Domain-Level Impact
```

These may need to be consolidated rather than reported as unrelated vulnerabilities.

---

# 15. Root Cause

Where possible, identify the underlying cause.

Examples:

* Excessive permissions
* Poor group delegation
* Insecure GPO permissions
* Weak certificate template configuration
* Excessive delegation
* Inadequate privilege separation
* Unnecessary trust relationships
* Poor account management

Root-cause analysis makes remediation more effective.

---

# 16. Remediation Link

Every finding should connect to a corrective action.

Example:

```text id="c7m5x9"
Finding
  ↓
Root Cause
  ↓
Recommended Change
  ↓
Validation
```

Avoid recommendations that simply say:

```text
"Fix the vulnerability."
```

The remediation should explain what should change.

---

# 17. Validation After Remediation

Where possible, define how the fix can be verified.

Example:

```text id="x9q4m7"
Before:
User controls privileged group.

Remediation:
Remove unnecessary permission.

After:
User no longer controls privileged group.
```

This creates a measurable remediation condition.

---

# 18. Common AD Reporting Categories

Findings may involve:

### Identity and Access

* Excessive group membership
* Privileged account exposure
* Password reuse
* Weak account controls

### ACLs

* Excessive object permissions
* Privileged group control
* Insecure delegation of rights

### GPOs

* Excessive GPO permissions
* Insecure policy configuration

### Authentication

* Kerberoasting exposure
* AS-REP Roasting exposure
* Credential exposure

### Delegation

* Excessive delegation
* RBCD configuration issues

### AD CS

* Risky certificate templates
* Excessive enrollment permissions
* Weak identity mapping

### Trusts

* Excessive cross-domain trust relationships
* Inappropriate cross-domain authorization

---

# 19. Report Candidate Paths Carefully

A candidate attack path can be useful information without being a confirmed finding.

Example:

```text id="h8m3q5"
Status:
Candidate

Reason:
Relationship identified during enumeration but final authorization was not validated.
```

This prevents overstatement.

---

# 20. Document Blocked Paths

Blocked attack paths can also provide useful assessment context.

Example:

```text id="v4n7x2"
Status:
Blocked

Reason:
Required account was disabled.
```

This demonstrates that the path was investigated but did not result in the expected privilege.

---

# 21. Document Limitations

Examples:

* Testing window was limited
* Some systems were unavailable
* Certain actions were prohibited
* Credential validation was not permitted
* Production changes were not allowed
* Evidence was based on configuration rather than live exploitation

Limitations should be explicit.

---

# 22. Executive Summary

The executive summary should communicate:

* Assessment objective
* Scope
* Major validated security themes
* Overall observations
* Important limitations
* Recommended priorities

Avoid excessive technical detail in this section.

---

# 23. Technical Findings

The technical section contains the detailed findings.

A typical structure:

```text id="m3q8x6"
Finding ID
Title
Severity

Affected Assets

Description

Technical Details

Attack Path

Impact

Evidence

Remediation

Validation

References
```

---

# 24. Attack Path Summary

A consolidated attack path section can help reviewers understand relationships between findings.

Example:

```text id="n7v4m9"
Initial Access
      ↓
Credential Exposure
      ↓
Domain User
      ↓
ACL Abuse
      ↓
Privileged Group
      ↓
Domain-Level Privilege
```

Only include transitions supported by evidence.

---

# 25. Remediation Summary

Create a remediation table.

| Finding | Root Cause                      | Recommended Action            | Validation            |
| ------- | ------------------------------- | ----------------------------- | --------------------- |
| AD-001  | Excessive ACL                   | Remove unnecessary permission | Recheck ACL           |
| AD-002  | Weak GPO delegation             | Restrict GPO permissions      | Recheck GPO ACL       |
| AD-003  | Risky certificate configuration | Correct template settings     | Revalidate enrollment |

---

# 26. Report Quality Review

Before finalizing the report, verify:

### Accuracy

* [ ] Every claim is supported
* [ ] Findings are current
* [ ] Scope is correct
* [ ] Privilege is not overstated

### Evidence

* [ ] Evidence is traceable
* [ ] Evidence is relevant
* [ ] Evidence is timestamped
* [ ] Sensitive data is sanitized

### Attack Paths

* [ ] Starting identity is clear
* [ ] Every relationship is documented
* [ ] Preconditions are identified
* [ ] Final privilege is validated

### Remediation

* [ ] Root cause is identified
* [ ] Remediation is actionable
* [ ] Validation method is defined

---

# 27. Reporting Workflow

Use the following process:

```text id="q8m5x3"
Collect Validated Results
        ↓
Group Related Observations
        ↓
Identify Root Causes
        ↓
Create Findings
        ↓
Document Attack Paths
        ↓
Attach Evidence
        ↓
Describe Impact
        ↓
Develop Remediation
        ↓
Define Validation
        ↓
Quality Review
        ↓
Final Report
```

---

# 28. Reporting Checklist

### Findings

* [ ] Finding IDs assigned
* [ ] Titles are specific
* [ ] Affected assets identified
* [ ] Technical descriptions written
* [ ] Impact documented
* [ ] Severity methodology applied consistently

### Attack Paths

* [ ] Starting identity identified
* [ ] Relationships documented
* [ ] Preconditions documented
* [ ] Resulting privilege validated
* [ ] Candidate paths clearly separated from confirmed paths

### Evidence

* [ ] Evidence mapped to findings
* [ ] Screenshots sanitized
* [ ] Credentials removed
* [ ] Sensitive information protected
* [ ] Evidence references are consistent

### Remediation

* [ ] Root cause identified
* [ ] Corrective action documented
* [ ] Validation method defined

### Final Review

* [ ] Scope verified
* [ ] Dates verified
* [ ] Findings reviewed
* [ ] Technical claims checked
* [ ] Limitations documented
* [ ] Report sanitized

---

## Transition

The next file is:

**`08-Reporting/Findings.md`**

It will define the practical structure for writing individual Active Directory security findings from the validated evidence collected during the workflow.
