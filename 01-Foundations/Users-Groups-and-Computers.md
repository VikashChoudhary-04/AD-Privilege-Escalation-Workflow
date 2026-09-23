# Users, Groups and Computers

Users, groups, and computers are some of the most important objects in Active Directory.

From a security-assessment perspective, they should not be viewed as isolated objects. Their **membership, permissions, relationships, and attributes** can create paths from one security principal to another.

A useful mental model is:

```text
Users
  │
  ├── Membership
  │
  ▼
Groups
  │
  ├── Permissions
  │
  ▼
Objects
  │
  ├── Access
  │
  ▼
Computers / Services / Other Resources
```

Understanding these relationships is essential before performing AD privilege-path analysis.

## Users

A user object represents an identity in Active Directory.

User accounts can represent:

* Employees
* Administrators
* Service accounts
* Application identities
* Other organizational identities

A user account contains attributes that describe the identity and its relationship with the domain.

Examples include:

* Username
* Display name
* Description
* Group memberships
* Account status
* Logon-related attributes
* Service-related attributes
* Other directory attributes

The exact attributes available depend on the account and environment.

## User Principal Name

A user can have a User Principal Name (UPN), commonly formatted as:

```text
username@domain
```

For example:

```text
alice@corp.example.com
```

The UPN provides a convenient identity format for authentication and directory operations.

## Security Identifier

Active Directory security principals are associated with security identifiers (SIDs).

A SID uniquely identifies a security principal within its security context.

A simplified example:

```text
User
 │
 └── SID
      └── S-1-5-21-...
```

Permissions in Windows are ultimately associated with security identifiers rather than simply the visible account name.

This matters because names can change while the underlying SID identifies the security principal.

## User Groups

A user's effective privileges can depend heavily on group membership.

For example:

```text
User
 │
 ├── Member Of ──► Group A
 │                    │
 │                    └── Permission X
 │
 └── Member Of ──► Group B
                      │
                      └── Permission Y
```

The user can therefore obtain permissions indirectly through group membership.

This is one of the most important concepts in AD privilege analysis.

## Direct vs Indirect Permissions

A user can receive access in multiple ways.

### Direct Permission

```text
User
 │
 └── Permission ──► Object
```

### Group-Based Permission

```text
User
 │
 └── Member Of ──► Group
                     │
                     └── Permission ──► Object
```

### Nested Group Permission

Groups can also contain other groups.

```text
User
 │
 └── Group A
       │
       └── Group B
             │
             └── Permission
```

Therefore, determining effective privileges requires examining group relationships rather than looking only at direct permissions.

## Security Groups

Security groups are used to assign permissions and privileges to multiple security principals.

Instead of granting the same permission separately to many users:

```text
User A ──► Permission
User B ──► Permission
User C ──► Permission
```

an organization can use:

```text
User A ─┐
User B ─┼──► Security Group ──► Permission
User C ─┘
```

This simplifies administration but also creates important security relationships.

## Built-in and Privileged Groups

Active Directory contains built-in groups and administrative groups with varying levels of privilege.

Examples include:

* Domain Admins
* Enterprise Admins
* Administrators
* Account Operators
* Backup Operators
* Server Operators

The exact permissions associated with a group depend on its role and the environment.

A security assessment should identify:

* Who belongs to privileged groups
* Which groups contain other groups
* What permissions those groups have
* Whether privileged membership is justified
* Whether delegated rights create equivalent access

## Domain Admins

`Domain Admins` is a highly privileged security group within a domain.

Membership can provide extensive administrative control over domain resources.

Conceptually:

```text
User
  │
  └── Member Of
          │
          ▼
    Domain Admins
          │
          ▼
   Domain-Level Access
```

However, not every route to domain-level control requires direct membership in `Domain Admins`.

Alternative privilege relationships can involve:

* Delegated permissions
* ACLs
* GPOs
* Delegation
* Compromised privileged credentials
* Other administrative groups

Therefore, an assessment should analyze the complete privilege graph.

## Nested Groups

Active Directory supports nested group membership.

For example:

```text
User
 │
 ▼
Helpdesk
 │
 ▼
Server Operators
 │
 ▼
Administrators
```

The effective permissions of the original user can depend on the entire chain.

This makes nested group enumeration important during privilege analysis.

## Computers

A computer joined to an Active Directory domain is represented by a computer object.

For example:

```text
Computer:
WORKSTATION01$
```

The `$` suffix is commonly used to identify computer accounts in certain Windows and AD contexts.

A computer account has its own identity and security attributes.

## Computer Accounts as Security Principals

Computer accounts are security principals.

This means they can:

* Authenticate
* Belong to groups
* Receive permissions
* Own or control certain directory relationships
* Participate in Kerberos authentication

Conceptually:

```text
Computer Account
       │
       ├── Authentication
       ├── Permissions
       └── Relationships
```

This is important because AD privilege escalation is not limited to user accounts.

## Machine Accounts

Domain-joined computers commonly have corresponding machine accounts in Active Directory.

For example:

```text
WORKSTATION01
      │
      ▼
WORKSTATION01$
```

The computer account has credentials used for machine authentication and secure communication with the domain.

Computer-account permissions can become relevant in delegation-related scenarios.

## Service Accounts

Some applications and services use dedicated AD accounts.

These accounts may have:

* Service Principal Names
* Access to network resources
* Elevated permissions
* Long-lived credentials

Service accounts are therefore particularly relevant to authentication and credential-access analysis.

For example:

```text
Service Account
      │
      ├── SPN
      │
      └── Service
```

This relationship becomes important when studying Kerberos and Kerberoasting.

## Service Principal Names

A Service Principal Name (SPN) identifies a service instance associated with an account.

A simplified example:

```text
MSSQLSvc/db01.corp.example.com:1433
```

SPNs are used by Kerberos to identify services.

During an assessment, identifying accounts associated with SPNs can reveal service accounts and authentication relationships that warrant further investigation.

SPN enumeration is covered later in the workflow.

## Computer Groups

Computer accounts can also participate in groups and permissions.

For example:

```text
Computer Account
       │
       ▼
Group Membership
       │
       ▼
Permissions
```

This means an assessment should not focus exclusively on human user accounts.

## Objects and Permissions

Users and computers can have permissions over other AD objects.

For example:

```text
User
 │
 └── Permission ──► User Object
```

or:

```text
Computer
 │
 └── Permission ──► Computer Object
```

or:

```text
Group
 │
 └── Permission ──► Group Object
```

These relationships are represented through access control mechanisms.

Later sections will examine how such permissions can form privilege-escalation paths.

## Security Identifiers and Access

Windows access control relies heavily on security identifiers.

A simplified model is:

```text
Security Principal
       │
       ▼
      SID
       │
       ▼
Access Control Entry
       │
       ▼
Object Permission
```

An Access Control Entry (ACE) specifies permissions for a security principal.

Multiple ACEs form an Access Control List (ACL).

Understanding this relationship is essential before studying AD ACL abuse.

## Effective Privilege

A user's effective privilege is not necessarily equal to the permissions shown on the user's account.

Consider:

```text
User
 │
 ├── Group A
 │     │
 │     └── Permission X
 │
 ├── Group B
 │     │
 │     └── Group C
 │           │
 │           └── Permission Y
 │
 └── Direct Permission Z
```

The effective access may therefore result from several different paths.

This is why privilege analysis should consider:

* Direct permissions
* Group membership
* Nested groups
* Object ownership
* Delegated rights
* GPO relationships
* Computer-account relationships

## Object Ownership

Active Directory objects have owners.

Ownership can provide significant control over an object depending on the permissions and security mechanisms involved.

Conceptually:

```text
Principal
    │
    └── Owns ──► AD Object
```

Object ownership therefore becomes another relationship worth understanding during ACL analysis.

## Privileged Access Paths

A simplified privilege path could look like:

```text
Low-Privilege User
       │
       ▼
Group Membership
       │
       ▼
Delegated Permission
       │
       ▼
Privileged Object
       │
       ▼
Higher Privilege
```

The path may contain several intermediate objects.

This is why tools and methodologies that represent AD relationships as graphs are useful during attack-path analysis.

## Example Relationship Graph

Consider:

```text
                    User
                     │
                Member Of
                     ▼
                  Group A
                     │
                Member Of
                     ▼
                  Group B
                     │
                Permission
                     ▼
                Computer
                     │
               Administrative
                  Access
                     ▼
              Higher Privilege
```

The user does not need to be directly listed as an administrator for the relationship chain to become security-relevant.

## Enumeration Mindset

When encountering users, groups, and computers, ask:

```text
Who are the users?
        ↓
Which groups exist?
        ↓
Who belongs to those groups?
        ↓
Are groups nested?
        ↓
Which computers exist?
        ↓
Which accounts control or access those computers?
        ↓
What permissions exist between these objects?
        ↓
Do the relationships create a privilege path?
```

## What to Remember

1. Users are security principals representing identities.
2. Computer accounts are also security principals.
3. Groups can provide permissions indirectly through membership.
4. Groups can contain other groups.
5. Nested groups can create complex privilege relationships.
6. Privileged groups can provide extensive administrative access.
7. Service accounts may have SPNs and access to important services.
8. Computer accounts can participate in authentication and security relationships.
9. Permissions can exist between many different AD objects.
10. Effective privilege depends on the complete relationship graph rather than only direct permissions.
11. SIDs identify security principals in Windows access-control mechanisms.
12. ACLs and ownership determine important object-level permissions.

## Security Assessment Perspective

Do not reduce AD privilege analysis to:

```text
"Is this user a Domain Admin?"
```

Instead ask:

```text
"What is this identity connected to?"
          ↓
"What groups does it inherit access from?"
          ↓
"What objects can those groups control?"
          ↓
"What permissions exist on those objects?"
          ↓
"Can those relationships form a path to higher privilege?"
```

Understanding users, groups, and computers provides the foundation for the ACL, GPO, delegation, credential, and attack-path analysis stages that follow.
