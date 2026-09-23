# LDAP

LDAP (Lightweight Directory Access Protocol) is a protocol used to access and interact with directory services.

Active Directory Domain Services (AD DS) provides LDAP interfaces that allow authorized clients and applications to query and manage directory information.

From a security-assessment perspective, LDAP is important because it can expose information about the structure and relationships of an Active Directory environment.

## What LDAP Provides

LDAP provides a standardized way to interact with directory information.

In an Active Directory environment, LDAP can be used to access information about:

* Users
* Groups
* Computers
* Organizational Units
* Domains
* Group Policy-related objects
* Service accounts
* Security-related attributes
* Other directory objects

A simplified model is:

```text id="xv9y8r"
Security Assessment
        │
        ▼
       LDAP
        │
        ▼
Active Directory Directory
        │
 ┌──────┼──────┬─────────┐
 ▼      ▼      ▼         ▼
Users  Groups Computers  OUs
```

## Directory Service

A directory service stores information about objects in a structured way.

Unlike a traditional database that is primarily designed for arbitrary application data, a directory service is optimized for information about identities, resources, and organizational relationships.

Active Directory uses a hierarchical directory structure.

For example:

```text id="v4p1d8"
Domain
 │
 ├── Users
 ├── Groups
 ├── Computers
 └── Organizational Units
```

LDAP provides a protocol for interacting with this directory.

## LDAP and Active Directory

Active Directory Domain Services supports LDAP as one of its primary directory-access protocols.

Conceptually:

```text id="f5x3sc"
Client
  │
  │ LDAP
  ▼
Domain Controller
  │
  ▼
Active Directory
```

The Domain Controller provides directory services while LDAP defines how clients communicate with those services.

## LDAP Ports

LDAP commonly uses:

```text id="o3c2jj"
TCP 389
```

LDAP can also operate over TLS using:

```text id="9q6h4k"
TCP 636
```

Port `636` is commonly associated with **LDAPS**, or LDAP over TLS.

The availability and configuration of these services depend on the environment.

## LDAP and Domain Controllers

LDAP services are commonly provided by Domain Controllers.

For example:

```text id="0f3h3u"
LDAP Client
     │
     ▼
  DC01
     │
     ▼
Directory Information
```

Therefore, identifying a Domain Controller often leads to identifying an LDAP service as well.

## LDAP Directory Structure

LDAP represents directory information in a hierarchical structure.

A simplified example is:

```text id="zkgp6n"
DC=corp,DC=example,DC=com
│
├── OU=Users
│   ├── CN=Alice
│   └── CN=Bob
│
├── OU=Computers
│   ├── CN=WS01
│   └── CN=WS02
│
└── OU=Groups
    ├── CN=IT
    └── CN=Helpdesk
```

The notation describes a distinguished name hierarchy.

## Distinguished Name

A Distinguished Name (DN) uniquely identifies an object within the LDAP directory hierarchy.

For example:

```text id="l8v2pq"
CN=Alice,OU=Users,DC=corp,DC=example,DC=com
```

This identifies an object named `Alice` located within the `Users` OU of the `corp.example.com` domain.

The components commonly include:

* `CN` — Common Name
* `OU` — Organizational Unit
* `DC` — Domain Component

## Relative Distinguished Name

The Relative Distinguished Name (RDN) is the component that identifies an object relative to its parent directory location.

For example:

```text id="w3d4x4"
CN=Alice
```

can be the RDN of the object represented by:

```text id="5rj8nt"
CN=Alice,OU=Users,DC=corp,DC=example,DC=com
```

## LDAP Attributes

Directory objects contain attributes.

For example, a user object may contain attributes such as:

```text id="7u2f4v"
sAMAccountName
userPrincipalName
displayName
memberOf
description
objectSid
```

A computer object can contain attributes such as:

```text id="p8m4yq"
sAMAccountName
dNSHostName
operatingSystem
servicePrincipalName
memberOf
```

The exact attributes available depend on the object and environment.

## LDAP Object Classes

LDAP objects are associated with object classes that describe the type of object and its available attributes.

For example:

```text id="6d8xk9"
User Object
    │
    ├── Object Class
    │
    └── Attributes
```

Different object types have different schemas and attributes.

This allows directory clients to distinguish between users, computers, groups, and other objects.

## LDAP Queries

LDAP clients can search the directory using filters.

A conceptual filter might look like:

```text id="q9t1n6"
(objectClass=user)
```

This represents a request for objects matching the specified condition.

Another example:

```text id="2z3r0k"
(objectClass=computer)
```

The exact query syntax and available attributes depend on the directory schema.

## Search Base

An LDAP search normally uses a search base that defines where the query begins.

For example:

```text id="3z4m8j"
DC=corp,DC=example,DC=com
```

A query can then search within that directory subtree.

Conceptually:

```text id="j8s2v6"
Search Base
     │
     ▼
Domain
     │
 ┌───┼────┬──────┐
 ▼   ▼    ▼      ▼
Users Groups Computers OUs
```

## LDAP Scope

LDAP searches can use different scopes.

Common concepts include:

* Base
* One Level
* Subtree

### Base

Search only the specified directory object.

### One Level

Search objects directly below the specified location.

### Subtree

Search the specified object and objects below it.

For example:

```text id="5j2k9p"
Domain
 │
 ├── OU=Users
 │    ├── Alice
 │    └── Bob
 │
 └── OU=Computers
      ├── WS01
      └── WS02
```

A subtree search from the domain can potentially include objects throughout the hierarchy.

## LDAP Authentication

LDAP operations can occur under different authentication contexts.

A client may perform queries using:

* An authenticated domain account
* Another supported authentication mechanism
* Anonymous access, if the environment permits it

The information available depends on the permissions and configuration of the directory service.

A security assessment should therefore distinguish between:

```text id="e2s5p6"
Unauthenticated Visibility
        vs
Authenticated Visibility
```

## Anonymous LDAP Access

Some LDAP services may permit limited anonymous access.

If enabled, an unauthenticated client may be able to retrieve certain directory information.

However, modern Active Directory environments commonly restrict anonymous access.

Therefore:

> **Do not assume that LDAP permits anonymous enumeration. Test and document the actual configuration.**

## Authenticated LDAP Enumeration

Authenticated access can expose significantly more directory information depending on the account's permissions.

For example:

```text id="t7j4k1"
Authenticated User
       │
       ▼
      LDAP
       │
       ▼
Directory Information
       │
 ┌─────┼─────┬────────┐
 ▼     ▼     ▼        ▼
Users Groups Computers OUs
```

This makes low-privileged domain credentials valuable for understanding the environment even when they do not provide administrative access.

## LDAP and Security Relationships

LDAP can expose information that helps identify relationships between objects.

For example:

```text id="q2v5g0"
User
 │
 └── memberOf ──► Group
                      │
                      └── Permissions
```

Attributes such as group membership can therefore help construct an initial understanding of the privilege graph.

## LDAP and Group Membership

Group membership can be represented through directory attributes.

For example:

```text id="9m3d5v"
User
 │
 └── memberOf
       │
       ├── Group A
       └── Group B
```

Groups can also contain other groups, creating nested relationships.

These relationships become important during privilege analysis.

## LDAP and Computer Enumeration

Computer objects can provide useful information such as:

* Hostname
* DNS hostname
* Operating system
* Group membership
* Service Principal Names
* Other attributes

For example:

```text id="x7w9y1"
Computer Object
      │
      ├── dNSHostName
      ├── operatingSystem
      └── servicePrincipalName
```

This can help an assessor understand the systems present within the domain.

## LDAP and SPNs

Service Principal Names can be stored as attributes associated with directory objects.

Conceptually:

```text id="w8n2q0"
Service Account
      │
      └── servicePrincipalName
              │
              ▼
          Kerberos Service
```

This relationship becomes important when performing SPN enumeration and investigating Kerberos authentication.

## LDAP and Group Policy

Some Group Policy-related information is represented within Active Directory.

This means LDAP queries can help identify:

* GPO objects
* GPO relationships
* Organizational structure
* Policy-related attributes

However, complete GPO analysis may also require information stored in `SYSVOL`.

## LDAP and ACL Information

Active Directory objects contain security-related attributes that describe access control.

Conceptually:

```text id="7d4z5x"
AD Object
    │
    ▼
Security Descriptor
    │
    ▼
Access Control Entries
    │
    ▼
Permissions
```

LDAP can therefore be part of the process of discovering object-level permissions.

## LDAP and Attack-Path Analysis

LDAP information can provide building blocks for understanding AD relationships.

For example:

```text id="2r5m9j"
LDAP
 │
 ├── Users
 ├── Groups
 ├── Computers
 ├── OUs
 ├── SPNs
 └── Other Attributes
          │
          ▼
Relationship Mapping
          │
          ▼
Attack-Path Analysis
```

LDAP does not itself determine whether an attack path exists. It provides directory information that can be analyzed alongside permissions, authentication, and other relationships.

## LDAP Enumeration Workflow

A useful conceptual workflow is:

```text id="6d2s8a"
Identify Domain
      ↓
Identify Domain Controller
      ↓
Identify LDAP Service
      ↓
Determine Authentication Context
      ↓
Identify Search Base
      ↓
Enumerate Directory Objects
      ↓
Enumerate Relevant Attributes
      ↓
Map Relationships
      ↓
Investigate Potential Privilege Paths
```

The actual commands and tools used for this process belong in the later `03-Enumeration/` section.

## Assessment Questions

When examining LDAP, ask:

```text id="x4w1r7"
Is LDAP available?
        ↓
Is LDAP protected with TLS?
        ↓
What authentication is required?
        ↓
What can an unauthenticated client see?
        ↓
What can the current account see?
        ↓
What directory objects are exposed?
        ↓
What relationships can be identified?
        ↓
Do any relationships require further investigation?
```

## What to Remember

1. LDAP is a protocol for interacting with directory services.
2. Active Directory Domain Services provides LDAP interfaces.
3. LDAP commonly uses TCP `389`.
4. LDAP over TLS commonly uses TCP `636`.
5. Domain Controllers commonly provide LDAP services.
6. LDAP represents directory information hierarchically.
7. Distinguished Names identify objects within the directory hierarchy.
8. Directory objects contain attributes.
9. LDAP filters can be used to search for specific object types or attributes.
10. The information available through LDAP depends on authentication and permissions.
11. LDAP can expose relationships between users, groups, computers, and other objects.
12. LDAP information can contribute to AD attack-path analysis.

## Security Assessment Perspective

LDAP should be viewed as an **information source**, not as an attack by itself.

The important question is:

```text id="3r2x6c"
"What does the directory reveal about
the identities, objects, permissions,
and relationships in this environment?"
```

That information can then be combined with authentication, ACL, GPO, delegation, and other security data to identify and validate potential privilege-escalation paths.
