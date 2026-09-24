# Group Membership Abuse

## Purpose

Group membership is one of the primary mechanisms used by Active Directory to assign permissions and privileges.

Group membership abuse occurs when a principal has unauthorized or excessive control over membership of a security-relevant group and that control can be used to obtain additional access.

The objective is to determine:

* Which groups are security-relevant
* Who can modify their membership
* What permissions control that membership
* Which accounts can be added or removed
* What privileges the group provides
* Whether modifying membership creates a privilege-escalation path
* Whether the path can be validated within authorized scope

The core workflow is:

```text id="q7m4x2"
Identify Group
      │
      ▼
Identify Current Members
      │
      ▼
Identify Group Privileges
      │
      ▼
Identify Membership Control
      │
      ▼
Identify Controlling Principal
      │
      ▼
Validate Permission
      │
      ▼
Assess Resulting Access
      │
      ▼
Document Path
```

---

## Position in the Workflow

```text id="m3x8p6"
05-Privilege-Escalation-Paths
              │
              ├── Group Membership Abuse  ← Current
              ├── ACL Abuse
              ├── GPO Abuse
              ├── Delegation Abuse
              ├── RBCD
              ├── ADCS
              └── Trust Abuse
```

---

## Why Group Membership Matters

Active Directory groups commonly determine access to:

* File shares
* Applications
* Servers
* Administrative functions
* GPOs
* Other groups
* Domain resources

A simplified relationship is:

```text id="f8q2v5"
User
 │
 ▼
Group
 │
 ▼
Permissions
 │
 ▼
Resource
```

If a principal can control membership of that group, it may be able to influence the permissions inherited through that group.

---

## Group Membership Abuse Model

The fundamental path is:

```text id="w6r3n8"
Controlled Principal
        │
        ▼
Can Modify Group Membership
        │
        ▼
Security-Relevant Group
        │
        ▼
New Member
        │
        ▼
Group Permissions
        │
        ▼
Additional Access
```

The critical question is not:

```text
"Can I see the group?"
```

It is:

```text
"Who can change its membership, and what happens if membership changes?"
```

---

## Group Types

Active Directory contains several types of groups.

Common categories include:

```text id="p5m7c2"
Security Groups
Distribution Groups
Domain Local Groups
Global Groups
Universal Groups
Built-in Groups
Custom Administrative Groups
```

For privilege-escalation analysis, focus primarily on security groups that provide meaningful permissions.

---

## 1. Enumerate Groups

Using PowerShell:

```powershell id="x4q8m7"
Get-ADGroup -Filter * |
    Select-Object Name,GroupScope,GroupCategory
```

For a specific group:

```powershell id="r2v6k9"
Get-ADGroup -Identity "Domain Admins" -Properties *
```

Record:

```text id="h7m3q8"
Group Name
Group Scope
Group Category
Description
Distinguished Name
```

Do not assume that a group's name alone establishes its privileges.

---

## 2. Enumerate Group Members

For a specific group:

```powershell id="n8p4x2"
Get-ADGroupMember -Identity "Domain Admins"
```

For nested membership:

```powershell id="q5m7r3"
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

This helps establish:

```text id="z3w8k5"
Group
 │
 ├── User
 ├── User
 ├── Group
 │    └── Nested User
 └── Computer
```

Nested groups can create indirect privilege relationships.

---

## 3. Identify Privileged Groups

Important groups may include built-in or organization-specific administrative groups.

Examples include:

```text id="m6q4v9"
Domain Admins
Enterprise Admins
Administrators
Account Operators
Backup Operators
Server Operators
Print Operators
Custom Administrative Groups
```

The actual security significance depends on the permissions assigned in the environment.

Do not assume every similarly named custom group has the same privileges as a built-in group.

---

## 4. Enumerate Nested Groups

Nested groups can create indirect privilege paths.

Example:

```text id="c8x2m5"
User A
   │
   ▼
Group A
   │
   ▼
Group B
   │
   ▼
Privileged Group C
```

A user may therefore receive privileges without being directly listed as a member of the final privileged group.

Use:

```powershell id="v7p3n6"
Get-ADGroupMember -Identity "GroupName" -Recursive
```

Then document the complete relationship.

---

## 5. Identify Group Membership Control

The key question is:

```text id="q9m4x7"
Who can modify the group?
```

Potential control mechanisms include:

* Write Members
* GenericWrite
* GenericAll
* WriteDacl
* Ownership
* Delegated administrative permissions

The relationship may look like:

```text id="r5x8m2"
Principal
   │
   ▼
ACL Permission
   │
   ▼
Group
   │
   ▼
Membership
```

This is where group analysis connects directly with ACL analysis.

---

## 6. Inspect Group ACLs

Retrieve the group's security descriptor:

```powershell id="k3q7v5"
Get-Acl "AD:\CN=GroupName,OU=Groups,DC=example,DC=local"
```

The exact distinguished name depends on the environment.

For broader AD object ACL analysis, use an appropriate directory-security enumeration method.

The goal is to identify principals with permissions that can modify the group.

---

## 7. Identify Membership-Modification Permissions

Relevant permissions may include:

```text id="m8p4x6"
Write Members
GenericWrite
GenericAll
WriteDacl
WriteOwner
```

The exact meaning of a permission depends on the object and security descriptor.

Do not report:

```text
"GenericWrite = Domain Admin"
```

Instead document:

```text
Principal
   │
   ▼
GenericWrite
   │
   ▼
Group Object
   │
   ▼
Security-Relevant Modification
```

---

## 8. Direct Membership Control

A direct relationship is:

```text id="w3m7q9"
User A
   │
   │ Membership Control
   ▼
Privileged Group
```

If the permission allows authorized membership modification, the path may be:

```text id="p6x2v8"
User A
   │
   ▼
Modify Group Membership
   │
   ▼
Privileged Group
   │
   ▼
Additional Access
```

The resulting access must be validated.

---

## 9. Indirect Membership Control

Control can also occur through another group.

Example:

```text id="g5q8m3"
User A
   │
   ▼
Group A
   │
   ▼
Permission Over Group B
   │
   ▼
Privileged Group B
```

The assessment should identify every relationship in the chain.

---

## 10. Group Ownership

Ownership can provide significant control over an AD object.

Conceptually:

```text id="x7m2q5"
Principal
   │
   ▼
Owner of Group
   │
   ▼
Security Descriptor Control
   │
   ▼
Potential Membership Control
```

Ownership does not necessarily mean immediate membership modification.

The exact permissions available through ownership must be established.

---

## 11. WriteDacl and Group Control

A principal with appropriate control over a group's DACL may be able to alter permissions on the group.

Conceptually:

```text id="v8q4n6"
Principal
   │
   ▼
WriteDacl
   │
   ▼
Group Security Descriptor
   │
   ▼
Permission Modification
   │
   ▼
Membership Control
```

This is a chained path and should be documented as such.

---

## 12. GenericAll and Group Objects

`GenericAll` represents extensive control over an object.

When applied to a group, it may allow security-relevant modifications.

The workflow remains:

```text id="h2m6x9"
GenericAll
    │
    ▼
Group Object
    │
    ▼
Available Operations
    │
    ▼
Membership Modification?
    │
    ▼
Privilege Impact
```

Do not treat the permission as the final finding without establishing the resulting capability.

---

## 13. Built-in Administrative Groups

Built-in groups deserve particular attention because their privileges are defined by Active Directory and Windows security architecture.

Examples include:

```text id="f3p7m8"
Domain Admins
Enterprise Admins
Administrators
Account Operators
Backup Operators
Server Operators
```

However, always verify the actual security context.

For example:

```text id="q8m5x2"
Group
 │
 ▼
Members
 │
 ▼
Nested Groups
 │
 ▼
Assigned Rights
 │
 ▼
Effective Privileges
```

---

## 14. Custom Administrative Groups

Organizations often create custom groups.

Examples:

```text id="s6v4p9"
Server-Admins
Infrastructure-Admins
Database-Admins
Backup-Admins
Application-Admins
Helpdesk-Admins
```

These groups can be security-relevant even when they are not built-in groups.

Investigate:

* Group members
* Nested groups
* Assigned ACLs
* GPO permissions
* Resource access
* Delegated rights

---

## 15. Identify Effective Privileges

Membership should be translated into actual permissions.

Example:

```text id="n4x7q2"
User
 │
 ▼
Group
 │
 ▼
Nested Group
 │
 ▼
Resource ACL
 │
 ▼
Effective Access
```

For example:

```text id="k6m3v8"
User A
  │
  ▼
Helpdesk-Admins
  │
  ▼
Computer Management Permission
  │
  ▼
WORKSTATION01
```

This is more useful than simply stating that the user belongs to `Helpdesk-Admins`.

---

## 16. Validate the Membership Path

A potential path should pass through:

```text id="y5q8m3"
Permission Exists
      │
      ▼
Correct Principal
      │
      ▼
Correct Group
      │
      ▼
Membership Can Be Modified
      │
      ▼
Membership Change
      │
      ▼
Expected Privilege
      │
      ▼
Access Confirmed
```

Only the necessary validation should be performed.

---

## 17. Analyze Privilege Changes

Before and after validation, document the security context.

Example:

```text id="r3m7x5"
Before:
User A
   └── Standard User

After Authorized Validation:
User A
   └── Member of Security-Relevant Group
```

Then establish the resulting access:

```text id="p8q2v6"
Group Membership
      │
      ▼
Assigned Permissions
      │
      ▼
Observed Access
```

Do not assume that membership automatically produces administrative privileges without validating the group's effective rights.

---

## 18. Group Membership and GPOs

Groups may receive privileges indirectly through GPOs.

Example:

```text id="c5n8m2"
User
 │
 ▼
Group
 │
 ▼
GPO Security Filtering
 │
 ▼
Computer Configuration
 │
 ▼
Resulting Access
```

This is why group membership should be correlated with the GPO enumeration already completed.

---

## 19. Group Membership and ACLs

A group may have permissions over sensitive objects.

```text id="m4x7q9"
User
 │
 ▼
Group
 │
 ▼
ACL
 │
 ▼
AD Object
 │
 ▼
Security-Relevant Operation
```

This creates a relationship between:

```text
Group Membership
+
ACL Permissions
```

and potentially a privilege-escalation path.

---

## 20. Group Membership and Delegation

Groups may contain accounts involved in delegation.

For example:

```text id="w6p3m8"
User
 │
 ▼
Service Group
 │
 ▼
Delegated Service Account
 │
 ▼
Target Service
```

This relationship should be documented when it materially affects the attack path.

---

## 21. Group Membership and Service Accounts

Service accounts may belong to privileged groups.

Example:

```text id="x2q5v7"
Service Account
      │
      ▼
Privileged Group
      │
      ▼
Additional Access
```

If the service account credential is discovered through Section 04, the findings can combine:

```text id="g8m4p2"
Credential
   │
   ▼
Service Account
   │
   ▼
Privileged Group
   │
   ▼
Resource Access
```

This is a complete credential-to-privilege relationship.

---

## 22. Nested Group Attack Paths

Nested membership can create paths that are easy to overlook.

Example:

```text id="h7x3m5"
User A
  │
  ▼
Group A
  │
  ▼
Group B
  │
  ▼
Group C
  │
  ▼
Privileged Resource
```

When documenting such a path, record every intermediate group.

Do not simplify:

```text
User A → Group C
```

if the actual relationship is:

```text
User A → Group A → Group B → Group C
```

The complete chain is useful evidence.

---

## 23. Evidence Collection

For each group-membership finding, record:

```text id="q5m8x4"
Source Principal:
Group:
Group Scope:
Current Members:
Nested Groups:
Controlling Permission:
Permission Holder:
Security-Relevant Privilege:
Validation:
Resulting Access:
Evidence:
```

Example:

```text id="r7p3v6"
Source Principal:
user-a

Controlled Object:
Server-Admins

Control:
Membership modification

Resulting Relationship:
user-a → Server-Admins

Observed Impact:
Administrative access to in-scope server resources
```

Use sanitized names in public documentation.

---

## Common False Positives

### Group Visibility

Being able to enumerate a group does not mean that its membership can be modified.

### Group Membership

Membership in a group does not automatically mean Domain Administrator access.

### Similar Group Names

Custom groups may have names similar to built-in administrative groups without having equivalent privileges.

### Nested Groups

A nested relationship may exist without granting the expected permission because of other security controls.

### Stale Membership

An account may be listed as a member but be disabled or otherwise unable to authenticate.

---

## Common Mistakes

### Mistake 1: Treating Enumeration as Control

```text
Can Read Group
      ≠
Can Modify Group
```

### Mistake 2: Ignoring Nested Groups

Indirect membership may be responsible for the actual privilege.

### Mistake 3: Ignoring ACLs

Group membership control is often granted through object permissions.

### Mistake 4: Assuming Group Names Determine Privilege

Always verify effective permissions.

### Mistake 5: Skipping Validation

A suspected permission should be confirmed before reporting a privilege escalation.

### Mistake 6: Ignoring Chained Paths

A group can connect credentials, ACLs, GPOs, and administrative resources.

---

## Group Membership Abuse Checklist

### Group Enumeration

* [ ] Enumerated relevant groups
* [ ] Identified group scope
* [ ] Identified group category
* [ ] Enumerated members
* [ ] Enumerated nested membership

### Privilege Analysis

* [ ] Identified security-relevant groups
* [ ] Identified actual group permissions
* [ ] Identified resource access
* [ ] Identified custom administrative groups

### Membership Control

* [ ] Identified group ACLs
* [ ] Identified membership-control permissions
* [ ] Identified permission holders
* [ ] Checked ownership
* [ ] Checked DACL control
* [ ] Checked indirect control paths

### Validation

* [ ] Confirmed permission
* [ ] Confirmed affected group
* [ ] Performed authorized validation
* [ ] Confirmed resulting membership
* [ ] Confirmed resulting authorization
* [ ] Documented evidence

### Correlation

* [ ] Correlated with credentials
* [ ] Correlated with ACLs
* [ ] Correlated with GPOs
* [ ] Correlated with service accounts
* [ ] Correlated with delegation
* [ ] Identified complete attack path

---

## Transition to ACL Abuse

Group membership analysis often reveals that the underlying control comes from an Active Directory ACL.

The workflow therefore continues:

```text id="n6q3w8"
Group Membership Abuse
          │
          ▼
Who Can Modify the Group?
          │
          ▼
ACL / Object Permissions
          │
          ▼
ACL Abuse
```

The next file is:

**`05-Privilege-Escalation-Paths/ACL-Abuse.md`**
