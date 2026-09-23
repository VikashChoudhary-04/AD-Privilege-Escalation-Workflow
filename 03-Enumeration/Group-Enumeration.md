# Group Enumeration

## Purpose

Group enumeration is the process of identifying Active Directory groups and understanding how membership in those groups affects access and privileges.

A user account by itself does not always reveal its effective privileges. Access may be inherited through:

* Direct group membership
* Nested groups
* Privileged groups
* Delegated groups
* Resource-specific groups
* Group-based ACLs
* GPO permissions

The goal is to build a clear relationship between:

```text
Users
  ↓
Groups
  ↓
Nested Groups
  ↓
Permissions
  ↓
Resources
  ↓
Effective Privilege
```

Group enumeration should therefore focus on **relationships**, not just group names.

---

## Enumeration Workflow

Use the following workflow:

```text id="4b6r8s"
Identify Domain Groups
        ↓
Classify Groups
        ↓
Enumerate Group Membership
        ↓
Identify Nested Groups
        ↓
Identify Privileged Groups
        ↓
Identify Delegated Groups
        ↓
Review Group Scope
        ↓
Review Group Permissions
        ↓
Map Users → Groups → Resources
        ↓
Identify Interesting Relationships
        ↓
Record Findings
        ↓
Move to Computer Enumeration
```

---

## 1. Identify Domain Groups

Begin by identifying the groups present in the domain.

Useful information includes:

```text id="l8b5l4"
Group Name
Group SID
Description
Group Scope
Group Type
Distinguished Name
Members
Member Of
```

On Windows:

```cmd id="p4u8rj"
net group /domain
```

PowerShell:

```powershell id="qf6g7v"
Get-ADGroup -Filter *
```

Detailed attributes:

```powershell id="6q0x3s"
Get-ADGroup -Filter * -Properties *
```

The available results depend on the account and directory permissions.

---

## 2. Understand Group Types

AD groups can serve different purposes.

Common group types include:

```text id="7kz6mv"
Security Groups
Distribution Groups
```

Security groups are particularly relevant to authorization because they can be assigned permissions to:

* Files
* Folders
* Shares
* Applications
* GPOs
* AD objects
* Administrative functions

Distribution groups primarily support messaging and distribution use cases.

For privilege analysis, prioritize groups that participate in **access control**.

---

## 3. Understand Group Scope

Common AD security group scopes include:

```text id="z2kq4p"
Domain Local
Global
Universal
```

Their intended use differs.

### Domain Local

Generally used to assign permissions to resources within its domain.

Conceptually:

```text
Users
  ↓
Global Group
  ↓
Domain Local Group
  ↓
Resource Permission
```

### Global

Commonly used to group users from the same domain according to role or function.

Example:

```text
CORP\Developers
CORP\Administrators
CORP\Finance
```

### Universal

Can be used across domains within a forest.

Universal groups become particularly relevant in multi-domain environments.

Understanding scope helps explain how permissions flow between users, groups, domains, and resources.

---

## 4. Enumerate Group Membership

For each relevant group, identify its members.

PowerShell:

```powershell id="l0f8yd"
Get-ADGroupMember "Domain Admins"
```

For all groups:

```powershell id="qj0qcs"
Get-ADGroup -Filter * | ForEach-Object {
    Get-ADGroupMember $_
}
```

Windows:

```cmd id="t10w0k"
net group "Domain Admins" /domain
```

Record:

```text id="f6xv7k"
Group
Members
Member Type
Source
```

Member types may include:

* User
* Group
* Computer

Do not assume that only user accounts can provide effective access.

---

## 5. Identify Nested Groups

Groups can contain other groups.

Example:

```text id="e6z8jq"
User
 ↓
Helpdesk
 ↓
IT-Operations
 ↓
Server-Admins
 ↓
Administrative Access
```

A direct membership query may only reveal:

```text
User → Helpdesk
```

while the effective relationship is much larger.

Nested membership is therefore one of the most important things to investigate.

---

## 6. Map Nested Group Relationships

Represent nested groups as a graph.

Example:

```text id="2xqv5j"
Developers
    │
    ▼
Application-Admins
    │
    ▼
Server-Admins
    │
    ▼
Privileged Access
```

For each group, identify:

```text
Member Of
Members
Nested Members
Nested Parent Groups
```

PowerShell can help inspect relationships:

```powershell id="8t3c5d"
Get-ADGroupMember "Application-Admins"
```

and:

```powershell id="g3lq4p"
Get-ADGroup -Identity "Application-Admins" -Properties memberOf
```

Graph-based tools can also help visualize nested relationships.

---

## 7. Identify Privileged Groups

Identify groups associated with significant administrative privileges.

Examples include:

```text id="v5u0gx"
Domain Admins
Enterprise Admins
Administrators
Schema Admins
Account Operators
Backup Operators
Server Operators
Print Operators
```

The exact groups and their effective privileges depend on the environment.

Do not assume:

```text
Group Name = Complete Privilege
```

Instead validate:

```text
Group
 ↓
Membership
 ↓
Assigned Permissions
 ↓
Scope
 ↓
Effective Access
```

Some groups may provide broad administrative capabilities, while others provide narrower rights.

---

## 8. Identify Delegated Groups

Not every security-relevant group is a built-in privileged group.

Organizations frequently create custom groups for delegated administration.

Examples:

```text id="k5e0hl"
Helpdesk-Admins
Server-Operators
SQL-Admins
Backup-Admins
Workstation-Admins
GPO-Admins
DNS-Admins
```

These names are only examples.

Look for custom groups that have permissions over:

* Users
* Groups
* Computers
* OUs
* GPOs
* Servers
* Shares
* Applications

Custom delegation can create privilege relationships that are not obvious from built-in group membership.

---

## 9. Review Group Descriptions

Group descriptions may provide useful context.

Examples:

```text id="m0h4x5"
Server administration team
Helpdesk password reset permissions
Application deployment administrators
Backup operators
```

Record:

```text id="2qf3gk"
Group:
Description:
Scope:
Type:
Members:
Notes:
```

Descriptions are evidence about intended use, not proof of current effective access.

---

## 10. Identify Groups Containing Groups

Search specifically for groups whose members include other groups.

Conceptually:

```text id="yq8qvl"
Group A
  │
  ├── User 1
  ├── User 2
  └── Group B
           │
           ├── User 3
           └── Group C
```

This creates a membership chain.

The chain should be recorded rather than flattened into a simple list.

Example:

```text id="7i4k4p"
User: jsmith

Direct:
    Helpdesk

Nested:
    Helpdesk
       ↓
    IT-Operations
       ↓
    Server-Admins
```

This provides much better visibility into effective privilege.

---

## 11. Identify Groups with Interesting Membership

Prioritize groups containing:

* Administrative accounts
* Service accounts
* Computer accounts
* Other privileged groups
* Large numbers of users
* Unexpected members
* Cross-domain members

Example:

```text id="jfl3m0"
Server-Admins
│
├── alice
├── bob
├── svc-backup
└── Workstation-Admins
```

Each relationship should be investigated according to the assessment scope.

---

## 12. Review Group SIDs

Every security group has a SID.

Example:

```text id="9n4dve"
S-1-5-21-AAAAAAAA-BBBBBBBB-CCCCCCCC-1101
```

Well-known groups may have recognizable RID patterns in many environments, but do not rely on a RID alone to determine current privilege.

Record the actual SID returned by the environment.

SIDs are useful when correlating:

* Group membership
* ACLs
* Access tokens
* File permissions
* Security events
* Object ownership

---

## 13. Review Group-Controlled Access

Groups can control access to many resources.

Examples:

```text id="1l9kq5"
Group
 │
 ├── File Permissions
 ├── SMB Share Permissions
 ├── GPO Permissions
 ├── AD Object Permissions
 ├── Application Access
 └── Administrative Rights
```

Therefore, group enumeration should eventually connect to ACL enumeration.

For example:

```text id="q9y1cq"
User
 ↓
Group
 ↓
ACL
 ↓
Object
 ↓
Permission
```

This relationship can later reveal potential privilege paths.

Detailed ACL analysis is covered in:

```text id="qyt1d0"
03-Enumeration/ACL-Enumeration.md
```

---

## 14. Identify Groups Associated with GPO Management

Some groups may have permissions over Group Policy Objects.

Potential permissions of interest include the ability to:

* Read a GPO
* Modify a GPO
* Create links
* Manage GPOs
* Delegate GPO administration

At this stage, identify the group relationships.

Detailed GPO enumeration is covered in:

```text id="o7o9y5"
03-Enumeration/GPO-Enumeration.md
```

---

## 15. Identify Cross-Domain Group Membership

In multi-domain forests, groups may contain principals from other domains.

Example:

```text id="d9a4m8"
CORP\Domain-Admins
        │
        └── DEV\developer
```

Cross-domain membership can be significant because it may create access relationships across administrative boundaries.

Record:

```text id="6v8h3a"
Source Domain
Target Domain
Group
Member
Relationship
```

Do not assume that cross-domain membership automatically grants broad administrative access. Validate the actual permissions and trust relationships.

---

## 16. Identify Built-in vs Custom Groups

Separate groups into categories.

Example:

```text id="7t7jz7"
Built-in / Default
    │
    ├── Domain Admins
    ├── Administrators
    └── Backup Operators

Custom
    │
    ├── IT-Admins
    ├── SQL-Admins
    └── Helpdesk
```

Custom groups deserve attention because delegated privileges are often implemented through organization-specific groups.

---

## 17. Useful Enumeration Tools

### Windows

```text id="xyf0vi"
net group /domain
net localgroup
whoami /groups
```

### PowerShell

```text id="9y3c6s"
Get-ADGroup
Get-ADGroupMember
Get-ADPrincipalGroupMembership
Get-ADObject
```

### LDAP

```text id="w7gk9v"
ldapsearch
```

### SMB/RPC

```text id="5t3k1m"
rpcclient
enum4linux-ng
netexec
```

### PowerView / SharpView

```text id="c8c2sn"
Get-DomainGroup
Get-DomainGroupMember
Get-DomainUser
```

### Graph-Based Collection

```text id="y2q2f6"
BloodHound-compatible collectors
```

Different tools expose different levels of group membership and relationship information.

---

## 18. Build a Group Inventory

Maintain a structured inventory.

Example:

```text id="1h2b0o"
Group:
SID:
Type:
Scope:
Description:
Direct Members:
Nested Groups:
Member Of:
Privileged:
Delegated Permissions:
Associated Resources:
Notes:
```

Example:

```text id="p9n5ub"
Group: Server-Admins
Scope: Global
Type: Security

Members:
- alice
- bob
- Workstation-Admins

Member Of:
- IT-Operations

Notes:
- Custom administrative group
- Requires ACL/resource validation
```

---

## 19. Build a User-to-Group Map

Create a simple relationship map.

Example:

```text id="ojz2kp"
alice
 ├── Domain Users
 ├── Helpdesk
 └── IT-Operations

bob
 ├── Domain Users
 └── Server-Admins

svc-backup
 ├── Domain Users
 └── Backup-Admins
```

Then expand nested memberships:

```text id="k6d6b9"
alice
  ↓
Helpdesk
  ↓
IT-Operations
  ↓
Server-Admins
```

This allows later attack-path analysis to work from actual relationships rather than isolated observations.

---

## 20. Effective Privilege

The important question is not:

```text
"What groups is this user directly in?"
```

It is:

```text
"What effective access does this user receive?"
```

A simplified model is:

```text id="y8c8ut"
Direct Membership
        +
Nested Membership
        +
ACLs
        +
Delegated Rights
        +
Resource Permissions
        =
Effective Privilege
```

This is why group enumeration must eventually connect with:

* ACL enumeration
* GPO enumeration
* Computer enumeration
* Share enumeration
* Attack-path analysis

---

## 21. Assessment Mindset

Ask:

```text id="qz1g6q"
What groups exist?
        ↓
Which groups are privileged?
        ↓
Who belongs to them?
        ↓
Which groups contain other groups?
        ↓
Which groups have delegated permissions?
        ↓
Which groups control resources?
        ↓
Which users inherit access through those relationships?
```

Do not immediately classify every administrative-looking group as exploitable.

First establish:

```text
Observed Relationship
        ↓
Permission
        ↓
Scope
        ↓
Effective Access
        ↓
Potential Attack Path
```

---

## Group Enumeration Checklist

### Group Discovery

* [ ] Domain groups enumerated
* [ ] Group SIDs recorded
* [ ] Group types identified
* [ ] Group scopes identified
* [ ] Descriptions reviewed

### Membership

* [ ] Group members enumerated
* [ ] User memberships recorded
* [ ] Group-to-group memberships identified
* [ ] Nested groups identified
* [ ] Cross-domain members identified

### Privilege

* [ ] Privileged groups identified
* [ ] Custom administrative groups identified
* [ ] Delegated groups identified
* [ ] Resource-related groups identified
* [ ] GPO-related groups identified

### Relationship Mapping

* [ ] User → Group relationships recorded
* [ ] Group → Group relationships recorded
* [ ] Group → Resource relationships identified
* [ ] Potential effective privilege paths documented

### Documentation

* [ ] Group inventory created
* [ ] Important memberships recorded
* [ ] Nested relationships documented
* [ ] Evidence sources recorded
* [ ] Follow-up ACL/GPO investigation identified

---

## Transition to Computer Enumeration

Once users and groups are understood, the next step is to identify the **computers and systems those identities interact with**.

Move to:

```text id="4y8f7k"
03-Enumeration/Computer-Enumeration.md
```

The next stage will focus on:

```text
Computers
  ↓
Hostnames
  ↓
Operating Systems
  ↓
Domain Membership
  ↓
Roles
  ↓
Logged-On Users
  ↓
Services
  ↓
Administrative Relationships
  ↓
Potential Privilege Paths
```

Understanding computers completes another major part of the AD relationship model:

```text
Users ↔ Groups ↔ Computers
```
