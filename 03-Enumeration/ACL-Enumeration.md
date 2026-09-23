# ACL Enumeration

## Purpose

Access Control List (ACL) enumeration is the process of identifying permissions assigned to users, groups, and other security principals over Active Directory objects.

ACLs are critical to Active Directory privilege analysis because a user does not necessarily need to belong to a highly privileged group to have significant control over an object.

A simplified relationship is:

```text
Principal
   ↓
ACL
   ↓
AD Object
   ↓
Permission
   ↓
Effective Control
```

The objective is to identify:

* Who has permissions over AD objects
* Which objects they control
* What type of permissions they have
* Whether permissions are inherited
* Whether permissions are delegated
* Whether a user or group can modify another security principal
* Whether permissions create potential privilege-escalation paths

ACL enumeration should focus on **relationships and effective control**, not simply dumping large permission lists.

---

## Enumeration Workflow

Use the following workflow:

```text
Identify Important AD Objects
        ↓
Identify Security Principals
        ↓
Enumerate Object ACLs
        ↓
Identify Explicit Permissions
        ↓
Identify Inherited Permissions
        ↓
Identify Dangerous / Interesting Rights
        ↓
Trace Group Membership
        ↓
Determine Effective Control
        ↓
Map Principal → Permission → Object
        ↓
Identify Potential Attack Paths
        ↓
Record Evidence
        ↓
Move to SPN Enumeration
```

---

## 1. Understand AD ACLs

An Active Directory object can have an ACL containing multiple Access Control Entries (ACEs).

Conceptually:

```text
AD Object
   │
   └── DACL
        │
        ├── User A → Permission
        ├── Group B → Permission
        └── Group C → Permission
```

The ACL determines which security principals can perform particular operations on the object.

Examples of objects with ACLs include:

* Users
* Groups
* Computers
* OUs
* GPOs
* Domain objects
* Service-related objects

---

## 2. Understand ACEs

An **Access Control Entry (ACE)** represents an individual permission entry.

An ACE can specify:

```text
Principal
Access Type
Permission
Object / Property
Inheritance
Scope
```

Example:

```text
CORP\Helpdesk
    ↓
Write Property
    ↓
User Object
```

The exact effect depends on the object, property, inheritance, and directory configuration.

---

## 3. Identify Important Security Principals

Start by identifying principals that appear in ACLs.

These may include:

```text
Users
Groups
Computers
Service Accounts
Built-in Principals
Authenticated Users
Domain Users
Other Domain Principals
```

Record the actual identity rather than relying only on a display name.

Useful identifiers include:

```text
SID
Distinguished Name
SAM Account Name
```

---

## 4. Identify Important AD Objects

Prioritize ACL analysis for objects that can influence privilege.

Examples include:

```text
Users
Groups
Computers
OUs
GPOs
Domain Object
Certificate-related Objects
Service-related Objects
```

A useful starting model is:

```text
Domain
│
├── Users
├── Groups
├── Computers
├── OUs
└── GPOs
```

Not every object requires the same depth of ACL analysis.

---

## 5. Enumerate Object ACLs

PowerShell can retrieve ACL information for AD objects.

Example:

```powershell
Get-Acl "AD:\CN=Alice,CN=Users,DC=corp,DC=example,DC=com"
```

For a broader enumeration workflow, PowerView can also be used to inspect object permissions.

The exact commands depend on the tool version and environment.

Record:

```text
Object
Principal
Permission
Inheritance
Source
```

---

## 6. Distinguish Explicit and Inherited Permissions

Permissions can be:

```text
Explicit
Inherited
```

Conceptually:

```text
Parent Object
     ↓
Inherited Permission
     ↓
Child Object
```

For example:

```text
OU
 ↓
Inherited ACL
 ↓
User / Computer
```

Therefore, an ACL should not be interpreted without understanding its inheritance.

Record:

```text
Explicit Permission:
Inherited Permission:
Inherited From:
```

---

## 7. Understand Object-Level vs Property-Level Permissions

AD permissions can apply to:

* Entire objects
* Specific properties
* Child objects
* Object creation
* Object deletion
* Group membership
* Password-related operations
* Extended rights

For example:

```text
Principal
   ↓
Write Property
   ↓
Specific Attribute
   ↓
Object
```

A permission such as `WriteProperty` should therefore not automatically be interpreted as unrestricted control.

Determine **what property or object the permission applies to**.

---

## 8. Identify Interesting Permission Categories

During authorized assessment, pay attention to permissions that may allow meaningful modification or control.

Examples include:

```text
GenericAll
GenericWrite
WriteDACL
WriteOwner
WriteProperty
CreateChild
DeleteChild
Extended Rights
AddMember
```

The actual security impact depends on:

* Target object
* Permission scope
* Property
* Principal
* Object type
* Inheritance
* Group membership

Therefore:

```text
Interesting Permission
        ≠
Confirmed Privilege Escalation
```

It is a lead requiring validation.

---

## 9. Understand GenericAll

`GenericAll` generally represents broad control over an object.

Conceptually:

```text
Principal
   ↓
GenericAll
   ↓
Object
```

Because it represents broad permissions, it deserves careful investigation.

However, the exact practical impact depends on the object being controlled.

Examples:

```text
User Object
Group Object
Computer Object
OU
```

should not be treated as identical targets.

---

## 10. Understand GenericWrite

`GenericWrite` generally represents broad write access to an object or its properties.

Conceptually:

```text
Principal
   ↓
GenericWrite
   ↓
Object
```

The exact security implications depend on:

* Target object
* Writable properties
* Existing configuration
* Authentication behavior
* Delegation
* Group membership

Validate the actual permission rather than assuming unrestricted control.

---

## 11. Understand WriteDACL

`WriteDACL` allows a principal to modify an object's discretionary ACL.

Conceptually:

```text
Principal
   ↓
WriteDACL
   ↓
Object ACL
   ↓
Permissions Can Be Changed
```

This can be particularly important because control over an object's ACL can potentially lead to additional permissions.

During enumeration, record:

```text
Principal
Target Object
WriteDACL
Inherited / Explicit
```

Detailed abuse belongs in the privilege-escalation phase.

---

## 12. Understand WriteOwner

`WriteOwner` relates to changing the ownership of an object.

Conceptually:

```text
Principal
   ↓
WriteOwner
   ↓
Object Ownership
```

Ownership and ACL control are related but distinct concepts.

Do not assume that changing ownership automatically provides every possible permission.

Record the relationship for later validation.

---

## 13. Identify WriteProperty Permissions

`WriteProperty` can be particularly interesting because the impact depends on **which property is writable**.

Example:

```text
Principal
   ↓
WriteProperty
   ↓
Specific Attribute
   ↓
User / Computer / Group
```

Always determine:

```text
Which object?
Which property?
Who can write it?
Is the permission inherited?
```

A generic `WriteProperty` observation without this context is incomplete.

---

## 14. Identify Extended Rights

Active Directory contains extended rights that provide specialized operations.

Examples may relate to:

* Password changes
* Replication
* Object-specific actions

Some extended rights can have significant security implications.

Record:

```text
Principal
Extended Right
Target Object
Scope
Inheritance
```

Do not classify an extended right as dangerous solely from its name. Validate its actual target and effect.

---

## 15. Identify Group Membership Control

ACLs can control membership-related operations on groups.

Conceptually:

```text
Principal
   ↓
Permission
   ↓
Group
   ↓
Membership
   ↓
User
```

For example, a principal may have permissions that allow modification of a group's membership.

This can become important if the affected group has significant privileges.

The relationship should be documented as:

```text
Principal → Can Modify → Group
```

Then:

```text
Group → Provides Access → Resource
```

This creates a potential privilege path.

---

## 16. Identify User Object Control

User objects can also have delegated permissions.

Potential relationships include:

```text
Principal
   ↓
User Object
   ↓
Write / Extended Permission
```

Investigate whether the permission affects:

* User attributes
* Group memberships
* Authentication-related properties
* Account control
* Other security-sensitive properties

Do not assume that every writable user attribute results in privilege escalation.

---

## 17. Identify Computer Object Control

Computer objects can also have delegated ACLs.

Example:

```text
Principal
   ↓
Computer Object
   ↓
Permission
```

Potentially relevant permissions may include:

```text
GenericAll
GenericWrite
WriteDACL
WriteOwner
WriteProperty
```

Computer object permissions can become particularly relevant when analyzing:

* Delegation
* Resource-based constrained delegation
* Machine account relationships
* Administrative access

Detailed exploitation is intentionally separated into the later privilege-escalation sections.

---

## 18. Identify OU-Level Permissions

Organizational Units can have ACLs affecting their child objects.

Conceptually:

```text
Principal
   ↓
OU Permission
   ↓
Child Objects
   ├── Users
   ├── Groups
   └── Computers
```

Potential permissions include:

```text
Create Child
Delete Child
Write Property
GenericWrite
GenericAll
```

Inheritance can make OU permissions particularly important.

Always determine:

```text
OU
 ↓
Permission
 ↓
Inheritance
 ↓
Affected Objects
```

---

## 19. Identify GPO-Related ACLs

GPO permissions were introduced during GPO enumeration, but ACL enumeration should correlate them directly with AD object permissions.

Example:

```text
Principal
   ↓
GPO Permission
   ↓
GPO
   ↓
OU
   ↓
Computer
```

This allows you to connect:

```text
Identity
     ↓
GPO Control
     ↓
Policy Scope
     ↓
System
```

Potential GPO abuse is covered later in:

```text
05-Privilege-Escalation-Paths/GPO-Abuse.md
```

---

## 20. Trace Group-Based ACLs

A principal may receive permissions indirectly through group membership.

Example:

```text
User
 ↓
Group A
 ↓
Group B
 ↓
ACL
 ↓
Computer
```

Therefore:

```text
ACL Principal ≠ Always Direct User
```

When an ACL references a group, trace:

```text
Group
 ↓
Members
 ↓
Nested Groups
 ↓
Effective Users
```

This is where user, group, and ACL enumeration converge.

---

## 21. Determine Effective Control

The central question is:

```text
What can this principal actually control?
```

Use the following model:

```text
Direct Permissions
        +
Inherited Permissions
        +
Group Membership
        +
Nested Groups
        +
Object Scope
        +
Property Scope
        =
Effective Access
```

Avoid declaring a privilege path based on a single ACL entry without checking these factors.

---

## 22. Identify High-Impact ACL Relationships

Examples of relationships worth investigating include:

```text
User
 ↓
WriteDACL
 ↓
Privileged Group
```

```text
User
 ↓
GenericAll
 ↓
Computer
```

```text
Group
 ↓
GenericWrite
 ↓
User
```

```text
Principal
 ↓
Modify GPO
 ↓
GPO
 ↓
Servers OU
```

These are **potential attack paths**.

They must be validated against the actual environment.

---

## 23. Use BloodHound for Relationship Analysis

Graph-based tools can make ACL relationships easier to understand.

A graph may represent:

```text
User
 ↓
MemberOf
 ↓
Group
 ↓
GenericAll
 ↓
Computer
```

or:

```text
User
 ↓
WriteDACL
 ↓
Group
 ↓
MemberOf
 ↓
Domain Admins
```

The advantage of graph analysis is that multiple relationships can be combined into a single attack-path view.

However, graph output should still be validated against actual directory permissions.

---

## 24. Useful Enumeration Tools

### PowerShell

```text
Get-Acl
Get-ADObject
Get-ADUser
Get-ADGroup
Get-ADComputer
```

### PowerView

Common capabilities include:

```text
Get-DomainObjectAcl
Find-InterestingDomainAcl
Get-DomainUser
Get-DomainGroup
Get-DomainComputer
```

### LDAP

```text
ldapsearch
```

### BloodHound-Compatible Collection

```text
BloodHound-compatible collectors
```

Graph tools are particularly useful for correlating ACLs with:

* Group membership
* Computer access
* GPO relationships
* Delegation
* Domain privileges

---

## 25. Build an ACL Inventory

Create a structured inventory.

Example:

```text
Target Object:
Object Type:
Principal:
Principal Type:
Permission:
Property / Extended Right:
Explicit / Inherited:
Inherited From:
Object Scope:
Group Membership:
Potential Impact:
Validation Status:
Notes:
```

Example:

```text
Target: Server-Admins
Type: Group

Principal: CORP\IT-Operations
Permission: WriteDACL
Scope: Explicit

Notes:
- Requires validation
- Group has administrative relationships
```

---

## 26. Build a Permission Graph

Represent important ACL relationships visually.

```text
Principal
    │
    │ Permission
    ▼
AD Object
    │
    │ Membership / Link / Ownership
    ▼
Privileged Object
```

Example:

```text
alice
  │
  └── WriteDACL
          │
          ▼
    Server-Admins
          │
          └── MemberOf
                  │
                  ▼
            Administrative Group
```

This makes potential attack paths easier to identify.

---

## 27. Separate Enumeration from Abuse

ACL enumeration identifies **what is possible**.

Privilege validation determines **whether the permission actually produces the expected security impact**.

Keep the stages separate:

```text
Enumeration
    ↓
Interesting ACL
    ↓
Permission Analysis
    ↓
Attack-Path Hypothesis
    ↓
Controlled Validation
```

Do not modify ACLs simply because an interesting permission has been discovered.

---

## 28. Assessment Mindset

Ask:

```text
Who has access?
        ↓
To which object?
        ↓
What exact permission?
        ↓
Is it explicit or inherited?
        ↓
Is it direct or through a group?
        ↓
What property or object does it affect?
        ↓
What effective control does it provide?
        ↓
What other object does that control connect to?
```

The objective is to transform:

```text
Raw ACL
```

into:

```text
Principal → Permission → Object → Impact
```

---

## ACL Enumeration Checklist

### Object Discovery

* [ ] Important AD objects identified
* [ ] Users reviewed
* [ ] Groups reviewed
* [ ] Computers reviewed
* [ ] OUs reviewed
* [ ] GPOs reviewed

### Permission Discovery

* [ ] ACLs enumerated
* [ ] ACEs identified
* [ ] Explicit permissions identified
* [ ] Inherited permissions identified
* [ ] Inheritance source recorded
* [ ] Property-level permissions reviewed

### Interesting Rights

* [ ] GenericAll reviewed
* [ ] GenericWrite reviewed
* [ ] WriteDACL reviewed
* [ ] WriteOwner reviewed
* [ ] WriteProperty reviewed
* [ ] Extended rights reviewed
* [ ] Group membership control reviewed
* [ ] Object creation/deletion permissions reviewed

### Relationship Mapping

* [ ] Principal → Object relationships recorded
* [ ] Group-based permissions traced
* [ ] Nested groups considered
* [ ] OU inheritance considered
* [ ] GPO relationships correlated
* [ ] Potential attack paths documented

### Validation

* [ ] Effective access considered
* [ ] Scope verified
* [ ] Permission target verified
* [ ] Potential impact documented
* [ ] No unnecessary changes made

---

## Transition to SPN Enumeration

ACL enumeration reveals **who can control which objects**.

The next step is to examine **Service Principal Names (SPNs)** and their relationship with Kerberos authentication.

Move to:

```text id="7c5m9x"
03-Enumeration/SPN-Enumeration.md
```

The next stage will connect:

```text
Service
  ↓
SPN
  ↓
Account
  ↓
Kerberos
  ↓
Authentication Relationship
  ↓
Potential Credential Attack Surface
```
