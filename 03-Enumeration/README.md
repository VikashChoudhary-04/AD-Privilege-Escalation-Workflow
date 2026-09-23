# AD Enumeration

Enumeration is the process of systematically collecting information about an Active Directory environment and the relationships between its objects.

The objective is not simply to collect as much information as possible.

The objective is to build an accurate picture of:

```text
Who exists?
    ↓
What systems exist?
    ↓
What groups exist?
    ↓
What resources exist?
    ↓
What permissions exist?
    ↓
What relationships exist?
    ↓
Which relationships require further investigation?
```

## Objective

The enumeration stage expands the initial information collected during `02-First-5-Minutes/` into a structured understanding of the Active Directory environment.

The major areas are:

* Network
* Domain
* Users
* Groups
* Computers
* Shares
* Group Policy
* Access Control
* Service Principal Names
* Trusts
* Delegation

## Enumeration Workflow

```text
Initial Foothold
       ↓
Network Enumeration
       ↓
Domain Enumeration
       ↓
User Enumeration
       ↓
Group Enumeration
       ↓
Computer Enumeration
       ↓
Share Enumeration
       ↓
GPO Enumeration
       ↓
ACL Enumeration
       ↓
SPN Enumeration
       ↓
Trust Enumeration
       ↓
Delegation Enumeration
       ↓
Relationship Mapping
       ↓
Potential Attack Paths
```

The exact order can change depending on the environment.

Enumeration should be iterative rather than strictly linear.

## Topics

| Topic                                                 | Purpose                                                                                                        |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| [Network Enumeration](./Network-Enumeration.md)       | Identify hosts, network ranges, services, DNS information, and reachable infrastructure                        |
| [Domain Enumeration](./Domain-Enumeration.md)         | Identify the domain, Domain Controllers, domain configuration, and general AD structure                        |
| [User Enumeration](./User-Enumeration.md)             | Identify users, account attributes, authentication-related information, and potentially interesting identities |
| [Group Enumeration](./Group-Enumeration.md)           | Identify security groups, privileged groups, nested memberships, and group relationships                       |
| [Computer Enumeration](./Computer-Enumeration.md)     | Identify domain-joined systems, roles, operating systems, and computer relationships                           |
| [Share Enumeration](./Share-Enumeration.md)           | Identify SMB shares, permissions, accessible resources, and potentially sensitive information                  |
| [GPO Enumeration](./GPO-Enumeration.md)               | Identify Group Policy Objects, links, scope, permissions, and security-relevant configurations                 |
| [ACL Enumeration](./ACL-Enumeration.md)               | Identify object permissions, delegated rights, ownership, and potentially dangerous access relationships       |
| [SPN Enumeration](./SPN-Enumeration.md)               | Identify service accounts and services associated with Kerberos Service Principal Names                        |
| [Trust Enumeration](./Trust-Enumeration.md)           | Identify domain and forest trust relationships and their security context                                      |
| [Delegation Enumeration](./Delegation-Enumeration.md) | Identify Kerberos delegation configurations and principals involved in delegation                              |

## Information Sources

AD enumeration can use multiple information sources.

```text
                 Enumeration
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
     LDAP            SMB           DNS
       │              │              │
       └──────────────┼──────────────┘
                      │
                      ▼
                 AD Objects
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        Users       Groups      Computers
          │           │           │
          └───────────┼───────────┘
                      ▼
                 Relationships
```

Other sources can include:

* Windows local information
* Kerberos
* Group Policy
* Security descriptors
* DNS records
* Network services
* Accessible files and shares

## Authenticated vs Unauthenticated Enumeration

The amount of information available depends on the current authentication context.

A useful distinction is:

```text
Unauthenticated
      │
      ▼
Limited Visibility

Authenticated
      │
      ▼
Expanded Visibility
```

A low-privileged domain account may provide substantially more information than an unauthenticated connection.

Therefore, document the authentication context used during enumeration.

## Passive vs Active Enumeration

Enumeration can also be considered in terms of activity.

### Passive / Low-Interaction

Examples include:

* Reviewing information already available to the current session
* Examining local configuration
* Reviewing existing directory information
* Inspecting accessible resources

### Active

Examples include:

* Network discovery
* Service enumeration
* LDAP queries
* SMB enumeration
* DNS queries
* Authentication attempts

The assessment should use the least intrusive approach that answers the question being investigated.

## Enumeration as Relationship Mapping

The most important purpose of enumeration is to identify relationships.

For example:

```text
User
 │
 ├── Member Of
 ▼
Group
 │
 ├── Has Permission On
 ▼
Computer
 │
 ├── Runs
 ▼
Service
```

Each relationship can provide context for later privilege-path analysis.

## Users

When enumerating users, consider:

* Username
* Domain
* Group membership
* Account status
* Description
* Service-related attributes
* Authentication-related attributes
* Privileged memberships

The objective is not to label every account as interesting.

Instead, identify identities that require further investigation.

## Groups

Groups can reveal:

* Administrative roles
* Privileged access
* Nested memberships
* Delegated permissions
* Resource access

For example:

```text
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Permission
```

Nested relationships can therefore be significant.

## Computers

Computer enumeration should establish:

* Hostname
* IP address
* Operating system
* Domain membership
* Server/workstation role
* Services
* Group membership
* Delegation-related configuration

A domain may contain:

```text
Workstations
Servers
File Servers
Application Servers
Domain Controllers
```

These systems can have very different security importance.

## Shares

SMB share enumeration can identify:

* Share names
* Permissions
* Accessible files
* Scripts
* Configuration data
* Potentially sensitive information

For example:

```text
SMB Share
   ↓
Configuration File
   ↓
Credential
   ↓
New Identity
   ↓
Further Enumeration
```

Any discovered credential must be handled according to the authorized assessment scope.

## GPOs

GPO enumeration should identify:

* GPO names
* GPO links
* Scope
* Security filtering
* Permissions
* Relevant configuration
* Objects affected by the policy

The key question is:

```text
Who can modify the policy,
and who receives it?
```

## ACLs

Access Control Lists are one of the most important areas of AD privilege analysis.

ACL enumeration can reveal relationships such as:

```text
User
 │
 └── Write Permission
        ↓
      Object
```

or:

```text
Group
 │
 └── Administrative Permission
        ↓
      Computer
```

These relationships can become potential privilege-escalation paths.

## SPNs

SPN enumeration identifies services associated with Kerberos principals.

For example:

```text
SPN
 │
 └── Service Account
        │
        └── Service
```

This can identify service accounts that require further authentication and credential-security analysis.

## Trusts

Trust enumeration identifies relationships between domains or forests.

For example:

```text
Domain A
   │
   │ Trust
   ▼
Domain B
```

Important information includes:

* Trusted domain
* Trust direction
* Trust type
* Transitivity
* Cross-domain relationships

A trust should not automatically be interpreted as privilege.

Its actual security significance depends on the configuration and permissions.

## Delegation

Delegation enumeration identifies configurations where one principal or service can act on behalf of another identity.

Important areas include:

* Unconstrained delegation
* Constrained delegation
* Resource-Based Constrained Delegation

Conceptually:

```text
Principal
    │
    ▼
Delegation Relationship
    │
    ▼
Target Service / Computer
```

Delegation relationships can become important during privilege-path analysis.

## Enumeration Notes

Do not treat enumeration as a checklist that is completed once.

Instead use an iterative process:

```text
Discover
   ↓
Analyze
   ↓
Identify Question
   ↓
Enumerate Further
   ↓
Discover New Relationship
   ↓
Analyze Again
```

For example:

```text
User discovered
     ↓
Group membership discovered
     ↓
Group permission discovered
     ↓
Target computer discovered
     ↓
Service discovered
     ↓
New account discovered
```

Each discovery can lead to another enumeration branch.

## Record Findings

Maintain a structured record of important findings.

A useful format is:

```text
Object:
Type:
Domain:
Host:
Owner:
Groups:
Permissions:
Related Objects:
Interesting Attributes:
Evidence:
Next Investigation:
```

This makes later attack-path analysis much easier.

## Avoid Blind Enumeration

A common mistake is:

```text
Run Every Tool
      ↓
Collect Huge Output
      ↓
Search Manually
      ↓
Lose Context
```

A better approach is:

```text
Question
   ↓
Relevant Enumeration
   ↓
Relevant Data
   ↓
Relationship
   ↓
Next Question
```

The goal is **useful information**, not maximum output.

## Enumeration Decision Model

Use this general model:

```text
What do I already know?
        ↓
What important information is missing?
        ↓
Which source can provide it?
        ↓
Can I obtain it with my current privileges?
        ↓
What relationship does it reveal?
        ↓
Does that relationship require validation?
```

## From Enumeration to Privilege Escalation

Enumeration should eventually produce a relationship map.

For example:

```text
Initial User
     │
     ├── Member Of
     ▼
   Group A
     │
     ├── Member Of
     ▼
   Group B
     │
     ├── Write Access
     ▼
    GPO
     │
     ├── Applies To
     ▼
Privileged Computer
```

This is where enumeration transitions into attack-path analysis.

The relationship itself does not automatically establish successful privilege escalation.

It identifies a path that requires further investigation and validation.

## Enumeration Checklist

```text
[ ] Network identified
[ ] Domain identified
[ ] Domain Controllers identified
[ ] DNS information collected
[ ] Users enumerated
[ ] Groups enumerated
[ ] Nested groups reviewed
[ ] Computers enumerated
[ ] SMB shares enumerated
[ ] Accessible resources reviewed
[ ] GPOs enumerated
[ ] GPO scope reviewed
[ ] GPO permissions reviewed
[ ] ACLs enumerated
[ ] Interesting permissions identified
[ ] SPNs enumerated
[ ] Service accounts identified
[ ] Trusts enumerated
[ ] Delegation enumerated
[ ] Relationships documented
[ ] Potential attack paths identified
```

## Transition to the Next Stage

Enumeration should produce enough information to answer:

```text
What identities exist?
        ↓
What systems exist?
        ↓
What resources exist?
        ↓
What permissions exist?
        ↓
What authentication relationships exist?
        ↓
What delegation and trust relationships exist?
        ↓
What potential privilege paths exist?
```

The next stages of this repository use these findings to investigate:

```text
Credentials & Authentication
          ↓
Privilege-Escalation Paths
          ↓
Attack-Path Analysis
          ↓
Privilege Validation
```

## Final Principle

The goal of AD enumeration is not:

```text
"Collect everything."
```

The goal is:

```text
"Understand enough of the environment
to identify meaningful security relationships."
```

A successful enumeration process transforms raw directory and network information into a structured understanding of the Active Directory privilege graph.
