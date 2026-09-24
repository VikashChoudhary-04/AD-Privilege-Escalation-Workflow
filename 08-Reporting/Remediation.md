# Remediation

## Purpose

Remediation converts validated security findings into practical corrective actions.

The objective is not simply to describe what should be changed, but to establish:

```text id="m7x4q9"
Finding
   ↓
Root Cause
   ↓
Corrective Action
   ↓
Security Control
   ↓
Validation
   ↓
Attack Path Removed / Reduced
```

A remediation should address the underlying cause wherever practical rather than only treating the visible symptom.

---

# 1. Remediation Principles

Effective remediation should be:

* Specific
* Actionable
* Proportionate
* Testable
* Aligned with least privilege
* Consistent with the organization's architecture
* Validated after implementation

Avoid recommendations such as:

```text id="q8n3v6"
Fix the vulnerability.
Improve security.
Secure Active Directory.
```

Instead, identify the exact configuration or relationship that should change.

---

# 2. Start With the Root Cause

A finding may have several symptoms but one underlying cause.

Example:

```text id="v5m8x2"
Excessive ACL
      ↓
Unauthorized Group Control
      ↓
Privileged Access
```

The remediation should normally focus on the excessive ACL rather than only removing the resulting group membership.

---

# 3. Remediation Structure

Use the following structure:

```text id="x7q4m9"
Finding
      ↓
Root Cause
      ↓
Recommended Action
      ↓
Expected Secure State
      ↓
Validation Method
```

Example:

```text id="n6m3x8"
Finding:
Excessive permissions on privileged group

Root Cause:
Unnecessary delegated ACL

Action:
Remove the unnecessary permission

Expected State:
Principal cannot modify the privileged group

Validation:
Re-enumerate ACL and test authorization
```

---

# 4. Least Privilege

Apply the principle of least privilege.

Users and systems should receive only the permissions necessary for their intended responsibilities.

Review:

* Group membership
* ACLs
* GPO permissions
* Delegation
* Service accounts
* Certificate enrollment
* Administrative roles
* Cross-domain access

---

# 5. Group Membership Remediation

If excessive group membership is identified:

### Review

* Current members
* Nested groups
* Business requirement
* Administrative responsibility
* Account status

### Correct

Remove unnecessary membership.

Example:

```text id="p8m4x7"
Before:

User
 ↓
Privileged Group

After:

User
 ↓
Required Non-Privileged Group
```

Validate that the user no longer receives the unnecessary privilege.

---

# 6. Nested Group Remediation

Nested groups can make effective privileges difficult to identify.

Example:

```text id="m7q3x9"
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Group
```

Review the complete membership chain.

Remediation may involve:

* Removing unnecessary nested membership
* Redesigning administrative groups
* Separating administrative and standard accounts
* Removing obsolete groups

---

# 7. ACL Remediation

For excessive ACL permissions:

1. Identify the unnecessary permission.
2. Identify the principal receiving it.
3. Confirm the intended administrative requirement.
4. Remove or restrict the permission.
5. Recheck inheritance.
6. Validate the resulting authorization.

Example:

```text id="x5n8m3"
Before:
User → WriteDacl → PrivilegedGroup

After:
User → No unnecessary control
```

Be careful with inherited permissions.

---

# 8. WriteDacl and WriteOwner Remediation

When dangerous object-control permissions are identified, review:

* Direct permissions
* Inherited permissions
* Group-based permissions
* Ownership
* Delegated administration

Remove unnecessary control rights while preserving required administrative functions.

---

# 9. GPO Remediation

For excessive GPO permissions:

Review:

* GPO owners
* Editors
* Delegated permissions
* GPO links
* Affected OUs
* Scope

Restrict modification privileges to authorized administrators.

Example:

```text id="q4m7x8"
Before:
Broad Group → GPO Modification

After:
Authorized Administrators → GPO Modification
```

After remediation, verify that the previous principal cannot modify the GPO.

---

# 10. Delegation Remediation

For unnecessary delegation:

Review:

* Source accounts
* Target services
* Delegation type
* Business requirement
* Scope

Remove unnecessary delegation relationships.

Where delegation is required:

* Limit its scope
* Use dedicated service accounts where appropriate
* Monitor privileged delegation
* Document the intended relationship

---

# 11. RBCD Remediation

For resource-based constrained delegation findings:

Review:

* `msDS-AllowedToActOnBehalfOfOtherIdentity`
* Source computer accounts
* Target computer accounts
* Administrative ownership
* Business requirement

Remove unauthorized delegation relationships.

Then verify:

```text id="w8m3q6"
Source Principal
      ↓
No Unauthorized RBCD
      ↓
Target Computer
```

---

# 12. AD CS Remediation

For certificate-related findings, review:

* Certificate template permissions
* Enrollment permissions
* Template configuration
* Subject/SAN configuration
* Authentication mapping
* CA permissions
* Administrative ownership

Restrict certificate issuance to appropriate principals.

After remediation, verify that the previously identified authentication path is no longer available.

---

# 13. Kerberos-Related Remediation

For Kerberoasting-related exposure:

Review:

* Service accounts
* SPNs
* Password strength
* Password rotation
* Account privileges

Where supported by the environment, consider:

* Strong service-account passwords
* Managed service accounts
* Appropriate account privilege separation
* Monitoring for suspicious Kerberos activity

The remediation should address both credential strength and excessive privilege.

---

# 14. AS-REP Roasting Remediation

For accounts that do not require pre-authentication:

Review the relevant account configuration.

Where appropriate, enable Kerberos pre-authentication.

Also review:

* Account privilege
* Password strength
* Account necessity
* Exposure of privileged accounts

Validate that the original condition no longer exists.

---

# 15. Credential Exposure Remediation

When credentials or credential material are exposed:

1. Identify the affected account.
2. Determine whether the credential is still valid.
3. Rotate or revoke it.
4. Identify other locations where it may have been reused.
5. Review resulting access.
6. Validate that the old credential no longer works.

Never include real credentials in remediation documentation.

---

# 16. Password Reuse Remediation

Where password reuse contributed to an attack path:

Review:

* Account passwords
* Service accounts
* Administrative accounts
* Shared credentials
* Password storage
* Credential management practices

Use unique credentials for separate accounts and services.

After remediation, validate that the previously reused credential no longer provides unauthorized access.

---

# 17. Trust Remediation

For problematic trust relationships:

Review:

* Trust direction
* Trust type
* Scope
* Cross-domain authorization
* SID filtering
* Selective authentication
* Business requirement

Where a trust is unnecessary, consider removing it.

Where it is required, restrict the associated authorization as much as practical.

---

# 18. Privileged Account Remediation

For privileged accounts:

Review:

* Group membership
* Account age
* Enabled state
* Password controls
* Administrative use
* Service dependencies
* Delegated permissions

Consider separating:

```text id="n4x8m6"
Standard User Account
        +
Dedicated Administrative Account
```

This reduces unnecessary exposure of privileged credentials.

---

# 19. Domain Admin Remediation

When unnecessary Domain Admin membership is identified:

Review:

* Direct membership
* Nested membership
* Service accounts
* Administrative groups
* Historical accounts
* Disabled accounts

Remove unnecessary membership.

The expected state should be:

```text id="m7q4x9"
Required Administrators
        ↓
Domain Admins

Unnecessary Principals
        ↓
Removed
```

Validate effective membership after the change.

---

# 20. Local vs Domain Privilege Remediation

Do not solve every privilege issue by granting broader domain permissions.

Example:

```text id="q8m3v6"
Requirement:
Administer SERVER01

Bad Remediation:
Add user to Domain Admins

Preferred Direction:
Grant only the required administrative scope
```

Remediation should minimize privilege scope.

---

# 21. Attack Path Remediation

For chained attack paths, identify which relationship should be removed.

Example:

```text id="x6n4m8"
User
 ↓
Group A
 ↓
ACL
 ↓
Group B
 ↓
Privileged Group
```

Possible remediation:

```text id="p5q8x3"
Remove unnecessary ACL
        ↓
Break Attack Path
```

This is often more effective than modifying the final privileged object alone.

---

# 22. Break the Path at Multiple Layers

Some attack paths have redundant routes.

Example:

```text id="v7m3q9"
             ┌── ACL ────────────┐
User ────────┼── Group Membership ┼──→ Privileged Target
             └── Credential Path ─┘
```

Removing one relationship may not eliminate the entire path.

Validate all known paths after remediation.

---

# 23. Expected Secure State

Every remediation should define the expected final state.

Example:

```text id="r8x4m6"
Before:
user01 can modify PrivilegedGroup.

Expected After:
user01 cannot modify PrivilegedGroup.

Validation:
ACL enumeration confirms the permission is absent.
```

This creates a clear remediation target.

---

# 24. Remediation Validation

Validation should answer:

```text id="n5q7m8"
Did the configuration change?
        ↓
Did the unnecessary permission disappear?
        ↓
Does the original attack path still work?
        ↓
Did another path replace it?
```

A configuration change alone is not always sufficient.

---

# 25. Re-Enumeration

After remediation, re-enumerate the affected object.

Examples:

```powershell id="x3m8q6"
Get-ADGroupMember "Domain Admins" -Recursive
```

For other objects, use the relevant enumeration method.

Compare:

```text id="m4q7v9"
Before
  ↓
Remediation
  ↓
After
```

---

# 26. Revalidate the Attack Path

If the original path was:

```text id="q6n3x8"
User
 ↓
ACL
 ↓
Privileged Group
 ↓
Domain Admins
```

After remediation, verify:

```text id="v8m4q5"
User
 ↓
ACL Removed
 ↓
Path Blocked
```

This is stronger evidence than merely showing that an ACL entry changed.

---

# 27. Regression Checks

A remediation should not introduce another security problem.

Check for:

* Alternative privileged paths
* Broken required administration
* New excessive permissions
* Unexpected inheritance
* Incorrect group nesting
* Service failures
* GPO scope changes

Security changes should be validated within the authorized environment.

---

# 28. Remediation Priority

Organizations may need to address several findings simultaneously.

Consider:

* Validated privilege
* Scope
* Number of affected assets
* Exploit prerequisites
* Exposure
* Root-cause breadth
* Ease of remediation
* Operational dependencies

Document the rationale used by the assessment methodology.

---

# 29. Remediation Tracking

Use a tracking table:

| Finding | Root Cause                | Action            | Owner    | Status      | Validation |
| ------- | ------------------------- | ----------------- | -------- | ----------- | ---------- |
| AD-001  | Excessive ACL             | Remove permission | AD Team  | Open        | Pending    |
| AD-002  | GPO delegation            | Restrict editors  | AD Team  | In Progress | Pending    |
| AD-003  | Certificate configuration | Correct template  | PKI Team | Open        | Pending    |

Keep ownership and status current.

---

# 30. Remediation Status

Possible states:

```text id="j8m4q6"
Open
In Progress
Implemented
Validated
Accepted Risk
Not Applicable
```

A remediation should not be marked **Validated** merely because a configuration change was reported.

The relevant security condition should be rechecked.

---

# 31. Compensating Controls

Sometimes direct remediation cannot be performed immediately.

Possible compensating controls may include:

* Restricting access
* Monitoring privileged activity
* Reducing account scope
* Disabling unnecessary accounts
* Restricting network reachability
* Increasing authentication controls

Clearly distinguish a compensating control from permanent root-cause remediation.

---

# 32. Common Remediation Mistakes

Avoid:

* Removing privileges without understanding dependencies
* Granting broader permissions to fix narrow access requirements
* Removing only one step when multiple attack paths exist
* Ignoring inherited permissions
* Ignoring nested groups
* Changing production systems without authorization
* Treating configuration change as proof of remediation
* Failing to revalidate the original path
* Publishing sensitive remediation details

---

# 33. Remediation Checklist

### Root Cause

* [ ] Root cause identified
* [ ] Affected relationship identified
* [ ] Scope understood

### Corrective Action

* [ ] Specific action defined
* [ ] Least privilege applied
* [ ] Dependencies considered
* [ ] Alternative paths considered

### Validation

* [ ] Configuration rechecked
* [ ] Permission rechecked
* [ ] Original path revalidated
* [ ] Alternative paths reviewed
* [ ] Expected secure state confirmed

### Documentation

* [ ] Owner assigned
* [ ] Status recorded
* [ ] Validation evidence preserved
* [ ] Limitations documented

---

# 34. Final Remediation Workflow

Use this workflow for each finding:

```text id="k7m4x9"
Validated Finding
      ↓
Identify Root Cause
      ↓
Identify Affected Relationship
      ↓
Define Least-Privilege Correction
      ↓
Implement Authorized Change
      ↓
Re-Enumerate
      ↓
Revalidate Original Path
      ↓
Check Alternative Paths
      ↓
Confirm Expected Secure State
      ↓
Record Evidence
      ↓
Mark Remediation Status
```

---

## Section 08 Completion

The **Reporting** workflow is now complete:

```text id="s3f9w2"
08-Reporting/
│
├── README.md
├── Findings.md
├── Attack-Path-Documentation.md
└── Remediation.md
```

Section 08 provides the complete process for turning validated Active Directory security observations into structured findings, documented attack paths, and verifiable remediation actions.

---

# Repository Workflow Completion

With Section 08 complete, the main technical workflow is now:

```text id="p8x4m7"
01-Foundations
        ↓
02-First-5-Minutes
        ↓
03-Enumeration
        ↓
04-Credentials-and-Authentication
        ↓
05-Privilege-Escalation-Paths
        ↓
06-Attack-Path-Analysis
        ↓
07-Privilege-Validation
        ↓
08-Reporting
        ↓
09-Labs
        ↓
References
```

The next section is:

**`09-Labs/README.md`**
