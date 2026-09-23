# Domain Controllers

A Domain Controller (DC) is a Windows Server system that provides core services for an Active Directory domain.

It stores and processes directory information, authenticates domain identities, participates in replication, and provides services that allow domain-joined systems to operate within the domain.

From a security-assessment perspective, the Domain Controller is one of the most important systems in an Active Directory environment because compromise of highly privileged access to the domain can affect identities, systems, and resources throughout the environment.

## Role of a Domain Controller

A Domain Controller commonly provides:

* Active Directory Domain Services (AD DS)
* Authentication
* Authorization-related directory information
* LDAP directory services
* Kerberos authentication
* DNS integration
* Group Policy distribution
* Active Directory replication

A simplified view is:

```text
                    Domain Controller
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
      AD DS             Kerberos            LDAP
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                           ▼
                    Domain Environment
```

## Domain Controller vs Domain

These concepts should not be confused.

### Domain

A **domain** is a logical Active Directory security and administrative boundary containing objects such as:

* Users
* Groups
* Computers
* Servers
* Organizational Units

### Domain Controller

A **Domain Controller** is a server that provides services for that domain.

For example:

```text
Domain:
corp.example.com

        │
        ├── DC01
        ├── DC02
        ├── Users
        ├── Groups
        └── Computers
```

A domain can have multiple Domain Controllers.

## Why Multiple Domain Controllers Exist

Organizations commonly deploy multiple Domain Controllers for:

* Availability
* Fault tolerance
* Load distribution
* Geographic distribution
* Directory replication

If one DC becomes unavailable, another DC may continue providing authentication and directory services.

A simplified environment might look like:

```text
                 Domain
            corp.example.com
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
        DC01                  DC02
          │                     │
          └────── Replication ──┘
```

## Active Directory Domain Services

Active Directory Domain Services (AD DS) is the core Windows Server role that provides the directory service for an Active Directory domain.

AD DS manages information about directory objects and makes that information available to authorized systems and users.

Important objects include:

* User accounts
* Computer accounts
* Groups
* Organizational Units
* Group Policy-related objects
* Service-related accounts
* Other directory objects

## Authentication

A Domain Controller participates in authenticating users and computers within the domain.

A simplified authentication flow is:

```text
User / Computer
      │
      │ Authentication
      ▼
Domain Controller
      │
      ├── Kerberos
      │
      └── Directory Services
      │
      ▼
Authenticated Identity
```

The exact authentication process depends on the protocol and circumstances, but the Domain Controller is commonly central to domain authentication.

## Kerberos and the Domain Controller

Kerberos is the primary authentication protocol used by modern Active Directory environments.

A Domain Controller commonly provides the Key Distribution Center (KDC) functionality required for Kerberos authentication.

The KDC contains two logical components:

```text
Key Distribution Center
        │
        ├── Authentication Service
        │
        └── Ticket Granting Service
```

At a high level:

```text
User
  │
  │ Authentication
  ▼
Authentication Service
  │
  ▼
Ticket Granting Ticket
  │
  ▼
Ticket Granting Service
  │
  ▼
Service Ticket
  │
  ▼
Target Service
```

Understanding this process is important for later topics such as:

* Service Principal Names
* Kerberoasting
* AS-REP Roasting
* Delegation
* Ticket-based authentication

## LDAP and the Domain Controller

Domain Controllers provide LDAP directory services that allow authorized clients to query and interact with Active Directory data.

Conceptually:

```text
Client
  │
  │ LDAP Query
  ▼
Domain Controller
  │
  ▼
Active Directory Database
  │
  ├── Users
  ├── Groups
  ├── Computers
  └── Other Objects
```

During enumeration, LDAP can therefore provide information about the structure and relationships of the domain.

## Active Directory Database

The Active Directory database is commonly stored in:

```text
C:\Windows\NTDS\ntds.dit
```

The database contains directory information including objects and their attributes.

The database is highly sensitive because it contains information necessary for the operation of the domain directory.

Direct access to the database is not normally required for ordinary directory queries. Domain Controllers expose directory services through mechanisms such as LDAP.

## SYSVOL

Domain Controllers also maintain the `SYSVOL` directory.

`SYSVOL` is used to store domain-wide files that need to be available to domain members, including Group Policy-related files and logon scripts.

A simplified structure is:

```text
Domain Controller
│
├── NTDS
│   └── ntds.dit
│
└── SYSVOL
    ├── Group Policy files
    └── Scripts
```

`SYSVOL` therefore becomes relevant during enumeration because it can contain configuration and policy-related information.

## Group Policy and Domain Controllers

Domain Controllers play an important role in distributing Group Policy information throughout the domain.

Group Policy can define settings affecting:

* Users
* Computers
* Security configuration
* Authentication behavior
* Administrative controls
* Software and system configuration

Conceptually:

```text
Group Policy
      │
      ▼
Domain Controller
      │
      ▼
Domain Members
      │
      ├── Users
      └── Computers
```

Misconfigured Group Policy can therefore have security implications across multiple systems.

## Replication

Active Directory uses replication to synchronize directory information between Domain Controllers.

For example:

```text
        DC01
         │
         │ Directory Changes
         ▼
        DC02
```

In an environment with multiple DCs, changes made on one Domain Controller can be replicated to other Domain Controllers.

Replication helps maintain consistency and availability throughout the domain.

## Global Catalog

A Domain Controller can also provide Global Catalog functionality.

The Global Catalog contains a searchable, partial representation of objects from the forest.

It helps clients locate objects across domains within an Active Directory forest.

Conceptually:

```text
                 Forest
                   │
       ┌───────────┼───────────┐
       │           │           │
     Domain A    Domain B    Domain C
       │           │           │
       └───────────┼───────────┘
                   │
             Global Catalog
```

The Global Catalog becomes particularly useful when dealing with environments containing multiple domains.

## DNS and Domain Controllers

DNS is closely integrated with Active Directory.

AD environments commonly use DNS to locate services such as Domain Controllers.

For example, clients can use DNS records to discover appropriate domain services.

A simplified relationship is:

```text
Client
  │
  │ DNS Query
  ▼
DNS
  │
  ▼
Domain Controller
  │
  ▼
AD Services
```

Because of this integration, DNS enumeration can provide useful information during an assessment.

## Domain Controller Discovery

One of the early objectives during an AD assessment is identifying the Domain Controller or Domain Controllers.

Useful information includes:

* Hostname
* IP address
* Domain name
* DNS name
* Available services
* Operating system
* LDAP availability
* Kerberos availability
* SMB availability

The exact enumeration methods depend on the environment and the access available to the assessor.

The important point is to establish:

```text
Current Position
      ↓
Domain
      ↓
Domain Controller
      ↓
Other Domain Systems
```

## Privileged Accounts

Active Directory contains accounts and groups with varying levels of administrative privilege.

Examples include:

* Domain Admins
* Enterprise Admins
* Built-in administrative accounts
* Other delegated administrative groups

Membership in a highly privileged group can provide extensive control over domain resources.

However, privilege should not be assessed solely by looking for obvious administrator group membership.

Effective privileges can also result from:

* Nested groups
* ACLs
* Delegation
* GPO permissions
* Computer-account permissions
* Service permissions
* Other object relationships

This is why later enumeration and attack-path analysis are essential.

## Domain Controllers as High-Value Targets

From an assessment perspective, Domain Controllers are high-value systems because they provide core domain services and maintain critical directory information.

Potential security impact depends on the access obtained.

For example:

```text
Low-Privilege User
       │
       ▼
Intermediate Access
       │
       ▼
Privileged Account
       │
       ▼
Domain-Level Control
```

The exact path depends on the environment and its configuration.

The objective of an assessment is to identify and validate such relationships rather than assume that a Domain Controller is directly exploitable.

## Domain Controller Security Perspective

When examining a Domain Controller, consider:

```text
Identity
   ↓
Authentication
   ↓
Directory Access
   ↓
Permissions
   ↓
Delegation
   ↓
Administrative Relationships
```

Each layer can reveal information about how control is distributed within the domain.

## What to Remember

1. A Domain Controller provides core Active Directory services.
2. A domain can contain multiple Domain Controllers.
3. Domain Controllers participate in authentication and directory services.
4. Kerberos authentication is closely associated with Domain Controllers through KDC functionality.
5. LDAP provides access to directory information.
6. `ntds.dit` contains the Active Directory database.
7. `SYSVOL` stores domain-wide files such as Group Policy-related data and scripts.
8. Active Directory uses replication to synchronize directory information between Domain Controllers.
9. DNS is tightly integrated with Active Directory service discovery.
10. Domain Controllers are high-value systems because they provide critical domain services and maintain sensitive directory information.
11. Effective privilege can arise from relationships beyond direct administrator-group membership.

## Assessment Mindset

When a Domain Controller is identified, don't immediately ask:

```text
"How do I compromise the DC?"
```

Instead ask:

```text
"What does this DC tell me about the domain?"
            ↓
"What services does it expose?"
            ↓
"What identities and objects can I enumerate?"
            ↓
"What permissions and relationships exist?"
            ↓
"Is there a validated path to higher privilege?"
```

This approach keeps the assessment focused on understanding the Active Directory environment and its privilege relationships.
