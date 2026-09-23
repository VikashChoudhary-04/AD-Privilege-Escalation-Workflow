# Active Directory Basics

Active Directory (AD) is Microsoft's directory service for managing identities, computers, resources, authentication, authorization, and administrative policies within a Windows-based environment.

From a security-assessment perspective, Active Directory is important because many security relationships are represented through **objects, groups, permissions, authentication mechanisms, and delegated rights**.

Understanding these relationships is essential for identifying potential privilege-escalation paths.

## What Active Directory Provides

Active Directory provides centralized management of resources such as:

* Users
* Groups
* Computers
* Servers
* Printers
* Organizational Units
* Group Policies
* Service accounts
* Other directory objects

Instead of every computer independently managing identities and permissions, a domain environment can use centralized directory services.

## Basic AD Architecture

A simplified Active Directory environment can be viewed as:

```text
                    Active Directory
                           │
                           ▼
                        Domain
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
        Users            Groups          Computers
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                    Domain Controller
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
            LDAP                    Kerberos
```

This is a simplified model. Real environments can contain multiple domains, Domain Controllers, organizational units, trusts, services, and other components.

## Directory Objects

Active Directory stores information as directory objects.

Common objects include:

| Object              | Description                                                        |
| ------------------- | ------------------------------------------------------------------ |
| User                | Represents an identity used by a person or service                 |
| Group               | Contains users or other security principals                        |
| Computer            | Represents a domain-joined computer                                |
| Organizational Unit | Provides logical organization for objects                          |
| Group Policy Object | Contains policy settings that can be applied to users or computers |
| Domain Controller   | A server providing core AD domain services                         |

Understanding these objects is important because security permissions can exist between them.

For example:

```text
User
  │
  └── Member of ──► Group
                       │
                       └── Has permissions on ──► Computer
```

A seemingly ordinary user may therefore have indirect privileges through group membership or delegated permissions.

## Security Principals

A security principal is an identity to which permissions can be assigned.

Common security principals include:

* User accounts
* Computer accounts
* Security groups
* Service accounts

Permissions can be assigned directly to a principal or inherited through group membership.

For example:

```text
User
  │
  └── Member Of
          │
          ▼
       Group
          │
          └── Permission
                    │
                    ▼
                  Object
```

This relationship is one of the fundamental concepts behind AD privilege analysis.

## Domain

An Active Directory domain is a logical administrative and security boundary containing directory objects such as users, groups, and computers.

A domain commonly has a DNS name such as:

```text
corp.example.com
```

The domain provides a central identity and authentication context for domain-joined systems.

During an assessment, identifying the domain is one of the first important pieces of information because it establishes the environment in which users, computers, groups, and permissions are being evaluated.

## Domain Controller

A Domain Controller (DC) is a server that provides core Active Directory domain services.

A DC commonly provides services related to:

* Directory storage
* Authentication
* Authorization
* Kerberos
* LDAP
* Domain replication
* Group Policy processing

A simplified authentication flow is:

```text
User
  │
  │ Authentication request
  ▼
Domain Controller
  │
  ├──► Kerberos
  │
  └──► Directory Services
```

The Domain Controller is therefore a particularly important component when analyzing an AD environment.

## Active Directory Database

A Domain Controller maintains the Active Directory database, commonly stored in:

```text
C:\Windows\NTDS\ntds.dit
```

The database contains directory information such as:

* User accounts
* Group information
* Computer accounts
* Password-related data
* Security-related attributes
* Other directory objects

Access to sensitive Domain Controller data can have significant security implications, which is why Domain Controllers are high-value systems during security assessments.

## Authentication vs Authorization

These concepts should be kept separate.

### Authentication

Authentication answers:

> **Who are you?**

Examples include:

* Password-based authentication
* Kerberos authentication
* Certificate-based authentication

### Authorization

Authorization answers:

> **What are you allowed to do?**

For example:

```text
Authentication
      ↓
Identity established
      ↓
Authorization
      ↓
Permissions determined
```

An attacker may obtain valid authentication without immediately having administrative authorization.

Privilege escalation often involves discovering a relationship that provides additional authorization.

## Groups and Privileges

Groups are particularly important in Active Directory because permissions can be assigned to groups rather than individual users.

For example:

```text
User
  ↓
Member of Group A
  ↓
Group A has permission on Object B
  ↓
User inherits access to Object B
```

This creates an important principle:

> **A user's effective privileges can be greater than the permissions assigned directly to the user account.**

Therefore, security assessments should examine both direct and indirect permissions.

## Organizational Structure

Active Directory environments can organize objects using Organizational Units (OUs).

For example:

```text
Domain
│
├── Users
│
├── Workstations
│
├── Servers
│
└── Service Accounts
```

OUs can be used to organize objects and apply Group Policy.

Their configuration can therefore affect the privileges and behavior of users and computers.

## Group Policy

Group Policy provides centralized configuration and management of users and computers.

Policies can control areas such as:

* Security settings
* User configuration
* Computer configuration
* Authentication-related settings
* Software configuration
* Administrative restrictions

Because policies can affect many systems or users simultaneously, improperly configured Group Policy can become relevant during privilege-escalation analysis.

Detailed GPO enumeration and abuse are covered later in the repository.

## LDAP

LDAP (Lightweight Directory Access Protocol) is used to interact with directory services.

In an Active Directory environment, LDAP can be used to query information about objects such as:

* Users
* Groups
* Computers
* Organizational Units
* Domain information
* Attributes and permissions

Conceptually:

```text
Security Assessment
        │
        ▼
      LDAP
        │
        ▼
Directory Information
        │
        ├── Users
        ├── Groups
        ├── Computers
        └── Other Objects
```

LDAP becomes particularly important during enumeration.

## Kerberos

Kerberos is a major authentication protocol used by Active Directory.

At a high level, Kerberos uses tickets to authenticate users and access services.

A simplified model is:

```text
User
 │
 ▼
Authentication
 │
 ▼
Kerberos
 │
 ├── Ticket Granting Ticket
 │
 └── Service Tickets
```

Understanding Kerberos is important for concepts such as:

* Service Principal Names
* Kerberoasting
* AS-REP Roasting
* Delegation
* Ticket-based authentication

These topics are covered in greater detail later.

## SMB

Server Message Block (SMB) is commonly used for network file and resource sharing in Windows environments.

SMB may expose resources such as:

* File shares
* Administrative shares
* Shared documents
* Scripts
* Configuration files

During an assessment, accessible shares can sometimes reveal information relevant to further enumeration or credential discovery.

## The AD Security Model

A useful way to think about Active Directory from an assessment perspective is as a graph of relationships:

```text
                 ┌──────────┐
                 │   User   │
                 └────┬─────┘
                      │
                 Member Of
                      │
                      ▼
                 ┌──────────┐
                 │  Group   │
                 └────┬─────┘
                      │
                 Permission
                      │
                      ▼
                 ┌──────────┐
                 │  Object  │
                 └────┬─────┘
                      │
                Delegated Right
                      │
                      ▼
                 ┌──────────┐
                 │  Target  │
                 └──────────┘
```

Privilege escalation can occur when these relationships create a path from a lower-privileged principal to a more privileged object or account.

## What to Remember

The most important concepts from this section are:

1. Active Directory centrally manages identities and resources.
2. A domain contains objects such as users, groups, and computers.
3. Domain Controllers provide core AD services.
4. Security principals can receive permissions directly or through group membership.
5. Authentication determines identity; authorization determines access.
6. LDAP provides access to directory information.
7. Kerberos provides ticket-based authentication within AD.
8. SMB provides network resource and file-sharing functionality.
9. Group Policy can centrally affect users and computers.
10. AD privilege analysis is largely about understanding relationships between identities, objects, permissions, and delegated rights.

## Security Assessment Perspective

When approaching an unfamiliar AD environment, avoid thinking only in terms of:

```text
"What command should I run?"
```

Instead, think:

```text
"What identity do I currently have?"
             ↓
"What objects can I see?"
             ↓
"What relationships exist?"
             ↓
"What permissions do those relationships provide?"
             ↓
"Does any relationship create a path to higher privileges?"
```

That mindset forms the foundation for the enumeration and attack-path analysis stages of this workflow.
