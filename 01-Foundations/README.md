# AD Foundations

This section covers the Active Directory concepts, components, protocols, and relationships required to understand the privilege-escalation workflow.

The objective is not to provide a complete administration guide. Instead, each topic focuses on the parts of Active Directory that are relevant to security assessment and privilege-escalation analysis.

## Learning Path

```text
Active Directory Basics
        ↓
Domains, Forests & Trees
        ↓
Domain Controllers
        ↓
Users, Groups & Computers
        ↓
Organizational Units & GPOs
        ↓
LDAP
        ↓
Kerberos
        ↓
SMB
```

## Topics

| Topic                                                        | Purpose                                                                                 |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| [Active Directory Basics](./Active-Directory-Basics.md)      | Understand what Active Directory is and how its main components fit together            |
| [Domain Controllers](./Domain-Controllers.md)                | Understand the role of Domain Controllers and why they are central to an AD environment |
| [Domains, Forests & Trees](./Domains-Forests-and-Trees.md)   | Understand AD hierarchy and relationships between domains                               |
| [Users, Groups & Computers](./Users-Groups-and-Computers.md) | Understand the main security principals and objects used within AD                      |
| [OUs & GPOs](./OUs-and-GPOs.md)                              | Understand organizational structure and centralized configuration through Group Policy  |
| [LDAP](./LDAP.md)                                            | Understand directory queries and how AD information can be accessed                     |
| [Kerberos](./Kerberos.md)                                    | Understand AD authentication and the tickets involved in Kerberos-based authentication  |
| [SMB](./SMB.md)                                              | Understand Windows file and network sharing and its relevance during AD enumeration     |

## Why These Concepts Matter

Active Directory privilege escalation is largely about understanding **relationships and permissions**.

For example:

```text
User
 ↓
Group Membership
 ↓
Permissions
 ↓
Object Relationship
 ↓
Delegated Rights
 ↓
Potential Privilege-Escalation Path
```

A strong understanding of the underlying AD structure makes later enumeration results much easier to interpret.

## Foundation Goals

By completing this section, you should be able to explain:

* What Active Directory is
* What a domain is
* What a Domain Controller does
* How forests and trees are organized
* What users, groups, and computers represent
* What Organizational Units are
* What Group Policy is
* What LDAP is used for
* How Kerberos authentication works at a high level
* What SMB provides
* How these components interact during an AD security assessment

## Prerequisite for the Next Stage

Before moving to `02-First-5-Minutes/`, you should be comfortable answering:

```text
What is the domain?

Who is the Domain Controller?

What users and groups exist?

What computers belong to the domain?

How does authentication work?

How are permissions and relationships represented?

How can these relationships create privilege-escalation paths?
```

These concepts form the foundation for the enumeration and attack-path analysis stages that follow.
