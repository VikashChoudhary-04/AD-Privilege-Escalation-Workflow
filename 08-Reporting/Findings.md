# Findings

## Purpose

A finding converts a validated security weakness into a structured, evidence-based report item.

For this workflow, every finding should connect:

```text id="q4m7x9"
Security Weakness
      ↓
Affected Object
      ↓
Attack Path
      ↓
Validated Result
      ↓
Impact
      ↓
Evidence
      ↓
Remediation
```

The goal is to accurately communicate what was identified and validated without overstating the result.

---

# 1. Finding Structure

Use a consistent structure for every finding:

```text id="v8n3m6"
Finding ID
Title
Severity
Status

Affected Assets

Description

Technical Details

Attack Path

Impact

Evidence

Root Cause

Remediation

Validation

Limitations
```

Consistency makes findings easier to review and compare.

---

# 2. Finding ID

Assign a unique identifier.

Example:

```text id="m5q8x2"
AD-001
AD-002
AD-003
```

Use the same identifier across:

* Report
* Evidence
* Attack-path documentation
* Remediation
* Screenshots

---

# 3. Finding Title

The title should identify the actual security issue.

Good examples:

```text id="x7m4p9"
Excessive ACL Permissions on a Privileged Group

Overly Permissive GPO Delegation

Risky Certificate Template Configuration

Excessive Delegation Permissions
```

Avoid vague titles such as:

```text id="q8n3v6"
Active Directory Vulnerability
AD Security Issue
Critical Problem
```

---

# 4. Severity

Severity should reflect the assessment methodology being used.

Possible classifications include:

```text id="h6m9x3"
Critical
High
Medium
Low
Informational
```

The methodology should be applied consistently.

Consider:

* Privilege gained
* Scope
* Exploitability
* Preconditions
* Number of affected resources
* Business sensitivity
* Existing controls

Do not assign severity solely because a finding involves Active Directory.

---

# 5. Finding Status

A finding should have a clear validation state.

Recommended states:

| Status         | Meaning                                                   |
| -------------- | --------------------------------------------------------- |
| Candidate      | Potential issue identified but not sufficiently validated |
| Verified       | Relevant configuration or relationship confirmed          |
| Confirmed      | Security consequence validated                            |
| Blocked        | Expected path could not proceed                           |
| Unverified     | Insufficient evidence to determine final result           |
| Not Applicable | Condition does not apply to the tested environment        |

Only appropriately validated results should be presented as confirmed security findings.

---

# 6. Affected Assets

Identify the assets involved.

Examples:

* Domain
* Domain Controller
* User
* Group
* Computer
* GPO
* Certificate Template
* Service
* Share

Example:

```text id="r5x8m2"
Affected Domain:
CORP.LOCAL

Affected Object:
PrivilegedGroup

Affected Principal:
user01
```

Use sanitized values in public documentation.

---

# 7. Description

The description should provide a concise explanation of the issue.

A useful pattern:

```text id="n7m4q8"
The principal has [permission/access] over [target], which allows [validated consequence].
```

Keep the description factual.

---

# 8. Technical Details

Explain how the issue works.

Include:

* Relevant configuration
* Security relationship
* Required permissions
* Preconditions
* Validation method
* Result

Example:

```text id="c8v3m6"
user01 has a delegated permission over PrivilegedGroup.
The permission allows modification of the relevant group relationship.
The resulting authorization was validated during the assessment.
```

---

# 9. Attack Path

Document the complete validated chain.

Example:

```text id="p6q9x4"
user01
   ↓
WriteDacl
   ↓
PrivilegedGroup
   ↓
Membership Control
   ↓
Domain Admins
```

If multiple paths exist, document them separately.

---

# 10. Direct vs Chained Findings

### Direct Finding

A single weakness produces the result.

```text id="m8x3q7"
User
 ↓
Excessive Permission
 ↓
Privileged Resource
```

### Chained Finding

Several relationships must be combined.

```text id="v4n7q2"
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Group
 ↓
Domain-Level Privilege
```

For chained paths, document every required relationship.

---

# 11. Preconditions

Document conditions required for the finding to be exploitable or usable.

Examples:

* Account must be enabled
* Required service must be reachable
* Permission must still exist
* User must authenticate
* Certificate template must be available
* Delegation must be configured

Example:

```text id="x5m8q3"
Prerequisites:
- Valid domain account
- Target service reachable
- Delegated permission present
```

---

# 12. Impact

Impact describes what the validated weakness allows.

Possible impacts include:

* Unauthorized privilege
* Access to protected resources
* Domain-level administrative access
* Unauthorized policy modification
* Credential exposure
* Lateral movement
* Persistence
* Access to sensitive systems

Only describe consequences supported by the assessment evidence.

---

# 13. Scope of Impact

Specify the scope.

Examples:

```text id="w7q4m9"
Single Computer
Single User
Single Group
Organizational Unit
Domain
Multiple Domains
Forest
```

Scope should not be inferred solely from the name of a privileged object.

---

# 14. Evidence

Every finding should reference supporting evidence.

Example:

```text id="j8m3x6"
Evidence:
- AD group membership output
- ACL enumeration output
- Runtime security context
- BloodHound attack-path graph
```

Evidence should establish the important parts of the finding.

---

# 15. Evidence Chain

A strong finding connects evidence in sequence:

```text id="n6q8v3"
Identity
   ↓
Permission
   ↓
Target
   ↓
Validation
   ↓
Result
```

Example:

```text id="r3m7x9"
user01
   ↓
WriteDacl
   ↓
PrivilegedGroup
   ↓
Membership modification
   ↓
Validated privileged access
```

---

# 16. Root Cause

Identify why the weakness exists.

Examples:

### Excessive ACL

```text id="q7m4x8"
Root Cause:
Unnecessary delegated permissions were assigned to a non-privileged principal.
```

### GPO

```text id="x3n8v5"
Root Cause:
GPO permissions allow unnecessary modification by a broad group.
```

### AD CS

```text id="m9q6x2"
Root Cause:
Certificate template configuration permits inappropriate enrollment or identity mapping.
```

Root cause should focus on the underlying configuration rather than only the observed symptom.

---

# 17. Remediation

Remediation should directly address the root cause.

Example:

```text id="v8m3q7"
Remove unnecessary ACL permissions from the affected principal.
Restrict administrative group modification to authorized administrators.
Review related nested group memberships.
```

Avoid generic recommendations such as:

```text id="p5x7n2"
Fix the configuration.
Improve security.
Patch the system.
```

---

# 18. Remediation Validation

Define how the fix can be verified.

Example:

```text id="k4q8m6"
Before:
user01 can modify PrivilegedGroup.

After:
user01 no longer has the relevant permission.

Validation:
Re-enumerate the ACL and verify that the permission has been removed.
```

---

# 19. Finding Example: ACL Abuse

A sanitized example:

```text id="w9m4x7"
Finding ID:
AD-001

Title:
Excessive ACL Permissions on a Privileged Group

Severity:
High

Status:
Confirmed

Affected Asset:
PrivilegedGroup

Description:
A non-privileged account has excessive permissions over a privileged security group.

Attack Path:
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
The validated permissions allow the assessed principal to obtain privileged domain authorization through the affected group.

Evidence:
- ACL enumeration
- Group membership output
- Runtime security context
- Validation output

Root Cause:
Excessive delegated permissions.

Remediation:
Remove unnecessary permissions and restrict modification of the privileged group to authorized administrators.

Validation:
Recheck the ACL and confirm that the principal no longer has the relevant permission.
```

---

# 20. Finding Example: GPO Permissions

Example structure:

```text id="x6q3m8"
Finding ID:
AD-002

Title:
Overly Permissive GPO Delegation

Affected Asset:
Security Policy GPO

Description:
A principal has unnecessary modification permissions over a GPO affecting sensitive systems.

Attack Path:
Principal
  ↓
GPO Modification
  ↓
Affected OU
  ↓
Policy Application

Impact:
Unauthorized policy modification may affect systems within the GPO's effective scope.

Evidence:
- GPO ACL
- GPO link
- Scope information
- Validation output

Remediation:
Restrict GPO modification permissions and review delegated administration.
```

---

# 21. Finding Example: AD CS

Example structure:

```text id="q8m5v3"
Finding ID:
AD-003

Title:
Risky Certificate Template Configuration

Affected Asset:
Certificate Template

Description:
The certificate template configuration permits an authentication path that can result in unauthorized privileged identity use under the validated conditions.

Attack Path:
Principal
  ↓
Certificate Enrollment
  ↓
Certificate
  ↓
Identity Mapping
  ↓
Privileged Authorization

Evidence:
- Template configuration
- Enrollment permissions
- Certificate details
- Authentication validation

Remediation:
Review template configuration, enrollment permissions, and identity mapping controls.
```

The exact impact should depend on what was actually validated.

---

# 22. Finding Example: Delegation

Example:

```text id="m7x4q9"
Finding ID:
AD-004

Title:
Excessive Delegation Permissions

Description:
A delegation configuration permits a principal to perform an authentication operation against a privileged service under the tested conditions.

Attack Path:
Source Principal
  ↓
Delegation
  ↓
Target Service
  ↓
Privileged Identity

Evidence:
- Delegation configuration
- Source account
- Target service
- Validation result

Remediation:
Review delegation requirements and restrict unnecessary delegation relationships.
```

Do not describe delegation alone as proof of domain compromise.

---

# 23. Finding Example: Trust Relationship

Example:

```text id="v5n8m2"
Finding ID:
AD-005

Title:
Excessive Cross-Domain Authorization

Description:
A cross-domain trust combined with an authorization relationship permits the assessed principal to access a protected resource in another domain.

Attack Path:
Source Domain
  ↓
Trust
  ↓
Cross-Domain Group
  ↓
Target Resource

Evidence:
- Trust configuration
- Group membership
- ACL
- Access validation

Remediation:
Review the trust and remove unnecessary cross-domain authorization.
```

The trust itself should not be presented as the vulnerability unless its configuration is the actual issue.

---

# 24. Finding Quality Test

Before finalizing a finding, ask:

```text id="n4q7x8"
Can another reviewer determine:

What is wrong?
        ↓
Where is it?
        ↓
Who can use it?
        ↓
What conditions are required?
        ↓
What did the assessment validate?
        ↓
What evidence proves it?
        ↓
How should it be fixed?
```

If any important question cannot be answered, improve the finding.

---

# 25. Common Reporting Mistakes

Avoid:

* Reporting enumeration as exploitation
* Calling every privileged group Domain Admin
* Treating BloodHound paths as automatically confirmed
* Omitting prerequisites
* Omitting affected assets
* Overstating impact
* Using vague titles
* Mixing multiple unrelated root causes
* Providing remediation unrelated to the root cause
* Publishing sensitive evidence
* Ignoring blocked or unverified paths

---

# 26. Finding Checklist

### Identification

* [ ] Finding ID assigned
* [ ] Specific title written
* [ ] Severity methodology applied
* [ ] Status defined

### Scope

* [ ] Affected domain identified
* [ ] Affected objects identified
* [ ] Affected principals identified
* [ ] Scope documented

### Technical Analysis

* [ ] Weakness described
* [ ] Preconditions documented
* [ ] Attack path documented
* [ ] Result validated
* [ ] Impact accurately described

### Evidence

* [ ] Evidence attached
* [ ] Evidence traceable
* [ ] Runtime context included where relevant
* [ ] Sensitive information sanitized

### Remediation

* [ ] Root cause identified
* [ ] Corrective action documented
* [ ] Remediation validation defined

---

## Transition

The next file is:

**`08-Reporting/Attack-Path-Documentation.md`**

It will focus specifically on documenting complete Active Directory attack chains, their dependencies, validation status, evidence, and relationship to individual findings.
