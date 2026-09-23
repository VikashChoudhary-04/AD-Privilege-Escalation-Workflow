# Domain Enumeration

## Purpose

Domain enumeration is the process of identifying and understanding the Active Directory domain associated with the current assessment environment.

After network enumeration identifies potential hosts and services, domain enumeration focuses on the **directory environment** itself.

The goal is to determine:

* Domain name
* Domain SID
* Domain controllers
* Forest context
* Directory naming context
* Global Catalog availability
* DNS relationship with Active Directory
* Current authentication context
* Important domain configuration
* Relevant privileged groups
* Initial relationships between domain objects

The objective is not simply to collect names. It is to build a reliable understanding of **how the domain is organized and where privilege relationships may exist**.

---

## Enumeration Workflow

Use the following workflow:

```text
Identify Current Domain
        ↓
Identify Domain SID
        ↓
Identify Domain Controllers
        ↓
Identify Forest Context
        ↓
Identify LDAP Naming Context
        ↓
Identify Global Catalog
        ↓
Identify Domain DNS Structure
        ↓
Identify Important Domain Groups
        ↓
Identify Authentication Context
        ↓
Identify Domain Configuration
        ↓
Record Relationships
        ↓
Move to User Enumeration
```

---

## 1. Identify the Current Domain

First determine which domain the current host or account belongs to.

Useful sources include:

* Current user information
* Host configuration
* DNS suffix
* Domain membership
* LDAP responses
* Kerberos information
* SMB authentication context
* Environment variables
* Directory queries

On Windows, useful commands include:

```cmd
whoami
whoami /user
whoami /groups
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
systeminfo
```

PowerShell:

```powershell
$env:USERDOMAIN
$env:USERDNSDOMAIN
$env:COMPUTERNAME
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
```

The exact commands available will depend on the privileges and environment.

Record:

```text
Domain:
DNS Domain:
Current User:
Current Computer:
```

---

## 2. Identify the Domain SID

The **Security Identifier (SID)** uniquely identifies a security principal.

An AD domain also has a domain SID.

Individual domain users and groups generally have RIDs appended to the domain SID.

Conceptually:

```text
Domain SID
    │
    ├── User RID
    ├── Group RID
    └── Computer RID
```

For example:

```text
Domain SID: S-1-5-21-AAAAAAAA-BBBBBBBB-CCCCCCCC
User SID:   S-1-5-21-AAAAAAAA-BBBBBBBB-CCCCCCCC-1105
```

The exact values will differ between environments.

Identifying the domain SID can help correlate users, groups, and computers during later enumeration.

Useful sources include:

```cmd
whoami /user
```

and directory/security-account information obtained through authorized enumeration tools.

---

## 3. Identify Domain Controllers

Domain Controllers (DCs) provide the core services required by the AD environment.

A domain may have multiple DCs.

Identify:

* DC hostname
* IP address
* DNS name
* Available AD services
* Global Catalog status
* Site information where available

Common services include:

| Service                 | Port | Purpose                        |
| ----------------------- | ---: | ------------------------------ |
| DNS                     |   53 | Name resolution                |
| Kerberos                |   88 | Authentication                 |
| LDAP                    |  389 | Directory access               |
| SMB                     |  445 | File/RPC-related AD operations |
| LDAPS                   |  636 | LDAP over TLS                  |
| Global Catalog          | 3268 | Forest-wide directory queries  |
| Global Catalog over TLS | 3269 | Secure GC queries              |

A host exposing several of these services is a strong candidate for a Domain Controller, but identification should be based on multiple indicators rather than a single open port.

---

## 4. Validate the Domain Controller

Do not assume that every host exposing an AD-related service is a DC.

Correlate information from:

* DNS
* LDAP
* Kerberos
* SMB
* Global Catalog
* Domain records
* Host naming
* Directory responses

For example:

```text
DNS
 │
 ├── identifies domain infrastructure
 │
 ├── points toward DCs
 │
 └── resolves AD services
        │
        ▼
Domain Controller
        │
        ├── Kerberos
        ├── LDAP
        ├── SMB
        └── Global Catalog
```

Multiple independent indicators provide greater confidence in the result.

---

## 5. Understand the LDAP Naming Context

Active Directory stores directory information inside a hierarchical namespace.

The domain DNS name is commonly represented as a distinguished name (DN).

For example:

```text
corp.example.com
```

can correspond to:

```text
DC=corp,DC=example,DC=com
```

This becomes an important LDAP search base.

Example:

```text
Search Base:
DC=corp,DC=example,DC=com
```

The naming context determines where directory queries begin.

Understanding the naming context is useful when performing:

* LDAP searches
* User enumeration
* Group enumeration
* Computer enumeration
* GPO enumeration
* ACL enumeration
* SPN enumeration

---

## 6. Understand RootDSE

**RootDSE** is a special LDAP entry that provides information about the LDAP server and directory configuration.

It can expose information such as:

* Default naming context
* Configuration naming context
* Schema naming context
* Supported LDAP capabilities
* Server information
* Root domain-related information

Conceptually:

```text
LDAP Server
     │
     └── RootDSE
          ├── Default Naming Context
          ├── Configuration Naming Context
          ├── Schema Naming Context
          └── Server Capabilities
```

RootDSE can therefore provide an efficient starting point for understanding the directory structure.

---

## 7. Identify the Forest Context

A domain does not necessarily exist in isolation.

Determine whether the current domain belongs to a larger AD forest.

Record, where available:

```text
Current Domain:
Forest:
Parent Domain:
Child Domains:
Other Known Domains:
```

For example:

```text
Forest
│
├── corp.example.com
│
├── dev.example.com
│
└── europe.example.com
```

The presence of multiple domains can introduce additional:

* Trust relationships
* Administrative boundaries
* Authentication paths
* Resource relationships
* Privilege relationships

Detailed trust enumeration is covered later in:

```text
03-Enumeration/Trust-Enumeration.md
```

---

## 8. Identify Global Catalog Services

The **Global Catalog (GC)** provides a searchable representation of objects across the forest.

Common ports:

```text
3268  → Global Catalog
3269  → Global Catalog over TLS
```

Determine:

* Whether a DC is a GC server
* Which hosts expose GC
* Whether forest-wide searches may be possible

Conceptually:

```text
Forest
│
├── Domain A
│    ├── Users
│    ├── Groups
│    └── Computers
│
├── Domain B
│    ├── Users
│    ├── Groups
│    └── Computers
│
└── Global Catalog
       │
       └── Searchable forest-wide object information
```

The Global Catalog becomes particularly useful when the assessment involves multiple domains.

---

## 9. Identify Domain DNS Structure

Active Directory depends heavily on DNS.

Determine:

* AD DNS domain
* DNS servers
* DNS suffix
* Domain Controller records
* SRV records
* Relevant AD service records

AD-related DNS records can help locate services such as:

```text
_ldap._tcp
_kerberos._tcp
_gc._tcp
```

Conceptually:

```text
DNS
 │
 ├── Domain Name
 │
 ├── Domain Controllers
 │
 ├── LDAP Services
 │
 ├── Kerberos Services
 │
 └── Global Catalog Services
```

DNS enumeration should therefore be considered part of AD enumeration rather than a completely separate activity.

---

## 10. Identify Important Domain Groups

Identify important security groups within the domain.

Examples include:

```text
Domain Admins
Enterprise Admins
Administrators
Account Operators
Backup Operators
Server Operators
Print Operators
Remote Desktop Users
Remote Management Users
```

The exact groups and memberships vary between environments.

At this stage, the objective is **identification**, not exploitation.

Record:

```text
Group
Description
Members
Nested Groups
Interesting Permissions
```

Detailed group enumeration will be performed in:

```text
03-Enumeration/Group-Enumeration.md
```

---

## 11. Identify the Current Authentication Context

The same host can produce very different enumeration results depending on the authentication context.

Determine whether enumeration is being performed:

* Unauthenticated
* As a local user
* As a domain user
* As a service account
* With elevated privileges
* Through delegated access

Useful information includes:

```text
Current User:
Domain:
User SID:
Group Membership:
Authentication Protocol:
Local/Domain Context:
Available Network Resources:
```

For example:

```text
LOCAL\user
```

and:

```text
CORP\user
```

represent different security contexts.

Always record the context under which enumeration was performed.

---

## 12. Identify Domain Configuration

Where authorized access permits, identify high-level domain configuration such as:

* Domain functional level
* Forest functional level
* Domain controllers
* Sites
* Subnets
* Password policy
* Account lockout policy
* Authentication-related configuration
* Domain-wide security settings

Do not assume that a discovered configuration applies universally without verifying its scope.

For example:

```text
Domain Policy
      │
      ├── Password Requirements
      ├── Account Lockout
      └── Authentication Settings
```

Some settings may instead be affected by:

* Fine-Grained Password Policies
* GPOs
* OU-specific configuration
* Local security policy
* Application-specific controls

These distinctions become important during later analysis.

---

## 13. Identify AD Sites and Subnets

AD Sites represent logical network locations used by Active Directory for:

* Replication
* Service discovery
* Client/DC selection
* Network topology

Where accessible, identify:

```text
Site
Subnet
Associated DCs
```

Example:

```text
HQ-Site
│
├── 10.10.10.0/24
│
└── DC01

Branch-Site
│
├── 10.20.20.0/24
│
└── DC02
```

This can help explain why certain hosts communicate with specific Domain Controllers.

---

## 14. Useful Enumeration Tools

Different tools expose different parts of the directory.

### Windows Built-in Tools

```text
whoami
systeminfo
ipconfig
nltest
net
dsquery
PowerShell ActiveDirectory module
```

### LDAP Tools

```text
ldapsearch
ldapwhoami
```

### SMB/RPC Enumeration

```text
netexec
smbclient
rpcclient
enum4linux-ng
```

### PowerShell-Based Enumeration

```text
PowerView
SharpView
ActiveDirectory module
```

### Graph-Based Enumeration

```text
BloodHound-compatible collectors
```

Tool output should be correlated rather than treated as automatically correct.

---

## 15. Build a Domain Inventory

Create a structured inventory as enumeration progresses.

Example:

```text
Domain:
Forest:

Domain Controllers:
- DC01
- DC02

Domain SID:
- S-1-5-21-...

Global Catalog:
- DC01
- DC02

DNS:
- DC01
- DC02

LDAP:
- DC01
- DC02

Kerberos:
- DC01
- DC02

Important Groups:
- Domain Admins
- Enterprise Admins
- Administrators

Current Context:
- CORP\user
```

This inventory becomes the foundation for later relationship mapping.

---

## 16. Map Domain Relationships

The information gathered so far should be connected.

Example:

```text
                 Forest
                   │
                   ▼
             corp.example.com
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      DC01        DC02       Users
        │          │           │
   ┌────┼────┐     │           ▼
   ▼    ▼    ▼     ▼         Groups
 LDAP Kerb  SMB   GC            │
                                ▼
                            Privileges
```

The objective is to understand how:

```text
Users
  ↓
Groups
  ↓
Computers
  ↓
Services
  ↓
Permissions
  ↓
Potential Privilege Paths
```

are connected.

---

## 17. Record Findings

Maintain an enumeration record.

At minimum capture:

```text
Domain:
Forest:
Domain SID:
Domain Controllers:
IP Addresses:
DNS Servers:
LDAP Servers:
Kerberos Servers:
Global Catalog Servers:
Important Groups:
Current Authentication Context:
Domain Configuration:
AD Sites:
Interesting Relationships:
```

Also record:

* Source of the information
* Authentication context
* Tool used
* Timestamp where relevant
* Confidence level
* Follow-up questions

This prevents rediscovering the same information later.

---

## 18. Assessment Mindset

Do not treat domain enumeration as a list of commands.

Ask:

```text
What domain am I in?
        ↓
What forest does it belong to?
        ↓
Which systems control the domain?
        ↓
Which directory services are available?
        ↓
What authentication context do I have?
        ↓
What important objects and groups exist?
        ↓
How are these objects related?
        ↓
Where should I enumerate next?
```

The objective is to gradually transform raw directory information into a model of the environment.

---

## Domain Enumeration Checklist

### Domain Identity

* [ ] Domain identified
* [ ] DNS domain identified
* [ ] Domain SID identified
* [ ] Forest identified
* [ ] Naming context identified

### Domain Controllers

* [ ] Domain Controllers identified
* [ ] DC IP addresses recorded
* [ ] LDAP services identified
* [ ] Kerberos services identified
* [ ] SMB services identified
* [ ] Global Catalog identified

### DNS

* [ ] DNS servers identified
* [ ] AD DNS records reviewed
* [ ] Relevant SRV records identified
* [ ] Domain/DC relationships documented

### Directory Structure

* [ ] RootDSE information reviewed
* [ ] Naming contexts identified
* [ ] AD Sites identified where accessible
* [ ] Subnets identified where accessible

### Security Context

* [ ] Current user identified
* [ ] User SID recorded
* [ ] Domain groups identified
* [ ] Authentication context documented
* [ ] Important privileged groups identified

### Documentation

* [ ] Domain inventory created
* [ ] DC inventory created
* [ ] Important relationships recorded
* [ ] Enumeration sources recorded
* [ ] Follow-up questions documented

---

## Transition to User Enumeration

Once the domain structure is understood, move from **domain-level information** to **individual directory objects**.

The next stage is:

```text
03-Enumeration/User-Enumeration.md
```

Focus on:

```text
Users
  ↓
Usernames
  ↓
SIDs
  ↓
Descriptions
  ↓
Group Membership
  ↓
SPNs
  ↓
Account Status
  ↓
Potentially Interesting Accounts
```

The goal is to determine **which user accounts exist, how they are related to groups and services, and which accounts require deeper investigation**.
