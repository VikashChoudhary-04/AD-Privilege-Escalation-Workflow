# Evidence Collection

## Purpose

Evidence collection turns a validated privilege or attack path into a **documented, reproducible, and defensible result**.

The objective is not to collect as much data as possible. It is to collect enough relevant evidence to demonstrate:

```text
Finding
  ↓
Source Identity
  ↓
Relationship / Weakness
  ↓
Validated Path
  ↓
Resulting Privilege
  ↓
Scope
  ↓
Evidence
```

Evidence should be collected with the least invasive method appropriate for the authorized assessment.

---

## Position in the Workflow

This file is the final stage of Section 07:

```text
Attack Path Validation
        ↓
Privilege Validation
        ↓
Evidence Collection
        ↓
Reporting
```

Evidence collected here feeds directly into:

**`08-Reporting/`**

---

# 1. What Makes Good Evidence?

Useful evidence should be:

* Relevant
* Current
* Reproducible
* Understandable
* Sufficient to support the finding
* Collected from the correct security context
* Sanitized before publication

A useful evidence chain is:

```text
Identity
   ↓
Relationship
   ↓
Permission
   ↓
Action / Validation
   ↓
Result
```

---

# 2. Evidence Hierarchy

Not all evidence has the same value.

### Strong Evidence

Examples:

* Current Active Directory object state
* Current group membership
* Current ACL configuration
* Current GPO configuration
* Current delegation configuration
* Current account state
* Controlled validation result
* Runtime security context

### Supporting Evidence

Examples:

* BloodHound graph
* Screenshots
* Tool output
* Enumeration results
* Command output
* Notes from manual verification

### Weak Evidence

Examples:

* Historical screenshots
* Unverified tool findings
* Stale scan results
* Assumptions based only on naming
* Inferred privilege without validation

Use multiple sources when practical.

---

# 3. Record the Tested Identity

Every privilege finding should identify the principal involved.

Record:

```text
Username:
Domain:
Account Type:
Enabled:
Authentication Context:
```

Example:

```text
Identity: user01
Domain: CORP.LOCAL
Account Type: Domain User
Enabled: Yes
```

Avoid publishing unnecessary sensitive information.

---

# 4. Record the Target

Identify the resource affected by the privilege.

Examples:

* Domain
* Domain Controller
* User
* Group
* Computer
* GPO
* Certificate Template
* Service
* File Share

Example:

```text
Target:
DC01.CORP.LOCAL

Target Type:
Domain Controller
```

---

# 5. Record the Relationship

Document the relationship that creates the privilege.

Examples:

```text
User → MemberOf → Group
```

```text
User → GenericAll → Group
```

```text
User → WriteDacl → Object
```

```text
User → Controls → GPO
```

```text
Principal → Delegation → Service
```

The relationship should be specific rather than simply described as "privileged access."

---

# 6. Record the Attack Path

Represent the validated path in a concise form.

Example:

```text
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

For a credential-based path:

```text
Initial User
  ↓
Credential Discovery
  ↓
Privileged Account
  ↓
Authenticated Access
```

The path should identify each meaningful transition.

---

# 7. Record Preconditions

Some paths only work when specific conditions exist.

Examples:

* Account enabled
* Service reachable
* Required group membership exists
* Permission remains present
* Certificate template is available
* Delegation configuration is active
* Network path is accessible

Record these conditions.

Example:

```text
Preconditions:
- user01 is enabled
- target service is reachable
- required ACL remains present
```

---

# 8. Capture Command Output

Where appropriate, preserve relevant command output.

Example:

```powershell id="m6q8x4"
Get-ADGroupMember "Domain Admins" -Recursive
```

Keep only the output necessary to demonstrate the finding.

Avoid collecting unrelated sensitive directory information.

---

# 9. Capture Runtime Context

When privilege depends on the current Windows security context:

```cmd id="q4x7m8"
whoami
```

and:

```cmd id="v5n3q9"
whoami /groups
```

This can demonstrate the identity and groups associated with the current session.

Runtime context should be interpreted alongside directory configuration.

---

# 10. Capture Directory State

For relevant AD objects, document current configuration.

Examples:

```powershell id="x7m4p8"
Get-ADUser "user01" -Properties *
```

```powershell id="n5q8v3"
Get-ADGroupMember "Domain Admins"
```

Only collect properties relevant to the finding when possible.

---

# 11. Capture ACL Evidence

When an attack path depends on permissions, record the relevant ACL relationship.

The evidence should show:

```text
Principal
   ↓
Permission
   ↓
Target Object
```

For example:

```text
user01
   ↓
WriteDacl
   ↓
PrivilegedGroup
```

Do not claim that an ACL proves a privilege unless the resulting authorization has been established.

---

# 12. Capture GPO Evidence

For GPO-based paths, record:

* GPO name
* GPO identifier
* Controlling principal
* Relevant permissions
* Link location
* Scope
* Affected computers/users
* Validation result

Example:

```text
GPO:
Workstation Security Policy

Controlled By:
user01

Linked To:
Workstations OU
```

The evidence should establish both control and scope.

---

# 13. Capture Delegation Evidence

For delegation-related findings, record:

* Source principal
* Target service/computer
* Delegation configuration
* Relevant account
* Required preconditions
* Resulting privilege

Example:

```text
Source:
Computer01$

Delegation:
Configured

Target:
Service01

Result:
Validated access to the intended resource
```

Do not infer final privilege solely from a delegation setting.

---

# 14. Capture AD CS Evidence

For certificate-related paths, document only the relevant configuration.

Potential evidence includes:

* Certificate Authority
* Certificate template
* Enrollment permissions
* Template configuration
* Identity mapping
* Authentication result
* Resulting authorization

The evidence chain should be:

```text
Template / Configuration
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

# 15. Capture Trust Evidence

For cross-domain or forest paths, record:

* Source domain
* Target domain
* Trust direction
* Trust type
* Relevant group/ACL relationship
* Resulting authorization

Example:

```text
Domain A
   ↓
Trust
   ↓
Domain B
   ↓
Authorized Group
   ↓
Target Resource
```

A trust relationship by itself is not sufficient evidence of privilege.

---

# 16. BloodHound Evidence

BloodHound is useful for visualizing relationships and identifying candidate paths.

Capture:

* Starting node
* Target node
* Relevant relationships
* Path structure
* Collection date

Example:

```text
User
 ↓
Group
 ↓
ACL Relationship
 ↓
Privileged Group
```

BloodHound evidence should normally be supported by direct validation of important relationships.

---

# 17. Timestamp Evidence

Record when evidence was collected.

Example:

```text
Evidence Date:
2026-09-24

Evidence Time:
11:30 IST
```

Timestamps help distinguish current findings from historical data.

---

# 18. Preserve Evidence Integrity

Do not alter raw evidence unnecessarily.

For important evidence:

```text
Raw Output
     ↓
Preserved
     ↓
Working Copy
     ↓
Sanitized Report Evidence
```

Keep original evidence protected when the engagement requires formal evidence handling.

---

# 19. Sanitize Sensitive Information

Before publishing screenshots, command output, or repository examples, remove or replace:

* Real usernames
* Passwords
* Tokens
* API keys
* Hashes
* Private keys
* Internal IP addresses where appropriate
* Sensitive hostnames
* Internal domain names
* Personal information

Example:

```text
Real:
vikash-admin@internal.example

Sanitized:
admin@corp.local
```

Do not publish real credentials.

---

# 20. Screenshots

Screenshots can provide useful supporting evidence.

A useful screenshot should show:

* Relevant command
* Relevant output
* Identity/context where necessary
* Enough surrounding information to establish meaning

Avoid screenshots containing unrelated sensitive information.

Prefer command output or structured evidence when it communicates the finding more clearly.

---

# 21. Evidence Naming

Use consistent names.

Example:

```text
finding-01-group-membership.txt
finding-01-acl-validation.txt
finding-01-runtime-context.txt
finding-01-bloodhound-path.png
```

A predictable naming convention makes evidence easier to correlate with findings.

---

# 22. Evidence-to-Finding Mapping

Every evidence item should map to a finding.

Example:

| Evidence                  | Supports                  |
| ------------------------- | ------------------------- |
| Group membership output   | Privileged membership     |
| ACL output                | Permission relationship   |
| BloodHound graph          | Attack-path visualization |
| `whoami /groups`          | Runtime context           |
| GPO configuration         | GPO control               |
| Certificate configuration | AD CS path                |
| Validation output         | Resulting authorization   |

---

# 23. Evidence Matrix

Use a structured matrix during an assessment.

| Step | Identity | Relationship    | Target        | Evidence         | Status    |
| ---- | -------- | --------------- | ------------- | ---------------- | --------- |
| 1    | user01   | MemberOf        | GroupA        | AD output        | Verified  |
| 2    | GroupA   | Controls        | GroupB        | ACL output       | Verified  |
| 3    | GroupB   | MemberOf        | Domain Admins | Group output     | Confirmed |
| 4    | user01   | Runtime context | Session       | `whoami /groups` | Confirmed |

Possible statuses:

* Candidate
* Verified
* Confirmed
* Blocked
* Unverified
* Not Applicable

---

# 24. Evidence for Failed Validation

Failed validation is also useful evidence.

Example:

```text
Expected:
Privileged Access

Observed:
Access Denied
```

Document:

* Expected result
* Test performed
* Actual result
* Blocking condition

This prevents a candidate path from being incorrectly reported as confirmed.

---

# 25. Evidence Limitations

Every finding should identify important limitations.

Examples:

```text
Limitation:
BloodHound data was collected before the final ACL change.
```

```text
Limitation:
The account could authenticate but the final administrative action was not performed.
```

```text
Limitation:
The privilege was verified through directory membership but not through a live administrative session.
```

Clear limitations improve the accuracy of the report.

---

# 26. Reproducibility

A reviewer should be able to understand how the result was obtained.

Record:

```text
Environment
Identity
Command / Method
Expected Result
Observed Result
Conclusion
```

Avoid relying exclusively on screenshots or undocumented tool output.

---

# 27. Minimum Evidence Set

For a validated privilege path, aim to capture:

* [ ] Starting identity
* [ ] Target resource
* [ ] Relevant relationship
* [ ] Preconditions
* [ ] Attack path
* [ ] Resulting privilege
* [ ] Scope
* [ ] Validation method
* [ ] Validation result
* [ ] Timestamp
* [ ] Limitations

---

# 28. Common Evidence Mistakes

Avoid:

* Collecting excessive unrelated data
* Publishing credentials
* Using stale enumeration as current proof
* Treating BloodHound alone as final validation
* Omitting the tested identity
* Omitting the target
* Omitting privilege scope
* Mixing evidence from different environments
* Failing to record timestamps
* Publishing unsanitized screenshots
* Claiming more privilege than the evidence demonstrates

---

# 29. Final Evidence Checklist

### Identity

* [ ] Principal identified
* [ ] Domain identified
* [ ] Account state checked
* [ ] Authentication context recorded

### Relationship

* [ ] Relevant relationship identified
* [ ] Permissions documented
* [ ] Group nesting verified
* [ ] Preconditions recorded

### Validation

* [ ] Candidate path validated
* [ ] Resulting privilege confirmed
* [ ] Scope confirmed
* [ ] Failed conditions documented

### Evidence

* [ ] Command output captured
* [ ] Relevant configuration captured
* [ ] Runtime context captured where necessary
* [ ] BloodHound evidence correlated
* [ ] Timestamp recorded
* [ ] Evidence mapped to finding

### Security

* [ ] Credentials removed
* [ ] Secrets removed
* [ ] Sensitive identifiers sanitized
* [ ] Raw evidence protected
* [ ] No unnecessary destructive action performed

---

## Section 07 Completion

The **Privilege Validation** workflow is now complete:

```text
07-Privilege-Validation/
│
├── README.md
├── Account-Privilege-Validation.md
├── Domain-Admin-Validation.md
└── Evidence-Collection.md
```

The section establishes a complete process for moving from a validated attack path to a documented privilege determination supported by evidence.

---

## Transition

The next workflow stage is:

**`08-Reporting/`**

Begin with:

**`08-Reporting/README.md`**

This section will convert validated technical findings and evidence into structured penetration-testing documentation, including findings, attack-path documentation, and remediation.
