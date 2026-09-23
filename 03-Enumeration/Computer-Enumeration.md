# Computer Enumeration

## Purpose

Computer enumeration is the process of identifying and analyzing computer objects within an Active Directory environment.

After user and group enumeration, the next question is:

```text
Which systems exist, who uses them, and who can administer them?
```

Computer enumeration focuses on:

* Computer names
* IP addresses
* Operating systems
* Domain membership
* Server and workstation roles
* Domain Controllers
* Logged-on users
* Administrative relationships
* Services
* SPNs
* Group memberships
* Delegated permissions
* Potential paths between identities and systems

The objective is to build a relationship between:

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
Effective Access
```

---

## Enumeration Workflow

Use the following workflow:

```text
Identify Computer Objects
        ↓
Identify Hostnames
        ↓
Identify IP Addresses
        ↓
Identify Operating Systems
        ↓
Identify Domain Controllers
        ↓
Identify Server and Workstation Roles
        ↓
Identify Logged-On Users
        ↓
Identify Computer Group Membership
        ↓
Identify Services and SPNs
        ↓
Identify Administrative Relationships
        ↓
Map Users/Groups → Computers
        ↓
Record Findings
        ↓
Move to Share Enumeration
```

---

## 1. Identify Computer Objects

Begin by enumerating computer objects in Active Directory.

PowerShell:

```powershell id="w7k8n2"
Get-ADComputer -Filter *
```

Include additional properties:

```powershell id="r9v5xq"
Get-ADComputer -Filter * -Properties *
```

Useful information includes:

```text
Computer Name
DNS Hostname
Distinguished Name
Operating System
Operating System Version
Enabled/Disabled Status
Last Logon Information
Service Principal Names
Member Of
Description
```

Windows environments may also provide information through:

```cmd id="0lq2f7"
net group "Domain Computers" /domain
```

The exact results depend on the environment and access level.

---

## 2. Identify Hostnames

Record the hostnames associated with computer objects.

Example:

```text id="1xj3x9"
DC01
FILE01
SQL01
WEB01
WS01
WS02
```

Hostname conventions can provide useful clues about system roles.

Examples:

```text id="8t6g1m"
DC-*       → Domain Controller
FILE-*     → File Server
SQL-*      → Database Server
WEB-*      → Web Server
APP-*      → Application Server
WS-*       → Workstation
```

These are only naming conventions.

Always validate the actual role using additional information.

---

## 3. Identify IP Addresses

Correlate computer objects with their network addresses.

Useful sources include:

* DNS
* Network enumeration
* LDAP attributes
* Host discovery
* Service enumeration

Example:

```text id="q5k0a2"
DC01 → 10.10.10.10
FILE01 → 10.10.10.20
SQL01 → 10.10.10.30
```

Keep the AD object name and network identity connected.

```text id="4u5xw7"
Computer Object
      ↓
Hostname
      ↓
DNS Name
      ↓
IP Address
```

This prevents confusion when multiple systems have similar names.

---

## 4. Identify Operating Systems

Determine the operating system where available.

Example:

```text id="x9m2lk"
DC01   → Windows Server
FILE01 → Windows Server
WS01   → Windows Client
```

PowerShell:

```powershell id="c8z5kr"
Get-ADComputer -Filter * -Properties OperatingSystem,OperatingSystemVersion |
    Select-Object Name,OperatingSystem,OperatingSystemVersion
```

Operating system information can help identify:

* Server vs workstation
* Legacy systems
* Specialized systems
* Potentially unsupported platforms

Do not assume that an operating system version alone determines vulnerability.

---

## 5. Identify Domain Controllers

Domain Controllers should be clearly identified.

Useful PowerShell query:

```powershell id="s6z2p8"
Get-ADDomainController -Filter *
```

Record:

```text id="5b8f4v"
Hostname
IP Address
Site
Operating System
Global Catalog Status
FSMO Roles where relevant
```

Domain Controllers are central to the AD environment and should be treated separately from ordinary member systems.

---

## 6. Identify Server and Workstation Roles

Classify computer objects according to their apparent function.

Example:

```text id="2u0k1n"
Domain Controllers
File Servers
Database Servers
Web Servers
Application Servers
Management Servers
Backup Servers
Workstations
Other Systems
```

A simple inventory can look like:

```text id="5f7k2p"
DC01    → Domain Controller
FILE01  → File Server
SQL01   → Database Server
WEB01   → Web Server
WS01    → Workstation
```

Role classification should be based on multiple indicators where possible.

---

## 7. Identify Computer Group Membership

Computer objects can also belong to groups.

Example:

```text id="7v8x4c"
Computer
   ↓
Group
   ↓
Permissions
```

PowerShell:

```powershell id="q4f6v2"
Get-ADComputer WS01 -Properties memberOf
```

Group membership can help identify:

* Administrative groups
* Server categories
* Application-specific groups
* Delegated management groups

Do not focus exclusively on user memberships.

Computer accounts are security principals too.

---

## 8. Understand Computer Accounts

A computer joined to a domain has an AD computer object and its own security identity.

Conceptually:

```text id="d7m2kq"
Computer
   │
   ├── Computer Account
   ├── SID
   ├── Group Membership
   ├── SPNs
   └── Security Permissions
```

This matters because computer accounts can participate in:

* Kerberos authentication
* ACLs
* Group membership
* Delegation
* Resource access
* Service authentication

A computer account should therefore be treated as a security principal rather than simply a hostname.

---

## 9. Identify SPNs on Computers

Computer accounts commonly have SPNs associated with services running on those systems.

Example:

```text id="m4p9c2"
HOST/DC01.corp.example.com
CIFS/FILE01.corp.example.com
```

PowerShell:

```powershell id="n1w7s5"
Get-ADComputer -Filter * -Properties ServicePrincipalName |
    Select-Object Name,ServicePrincipalName
```

SPNs can reveal:

* Services
* Service hosts
* Naming relationships
* Authentication dependencies

At this stage, document the SPNs.

Detailed authentication analysis belongs in:

```text id="p8m3w1"
04-Credentials-and-Authentication/Kerberoasting.md
```

---

## 10. Identify Logged-On Users

Where authorized access permits, identify users currently or recently associated with systems.

Potential sources include:

```text
query user
quser
Get-CimInstance
Get-Process
NetSession
```

Example:

```cmd id="w2x6y4"
query user
```

This information can help establish:

```text id="n3c5k8"
User
  ↓
Logged Onto
  ↓
Computer
```

That relationship can become important during attack-path analysis.

However, visibility into logged-on users depends heavily on privileges, operating system behavior, and collection method.

---

## 11. Identify Administrative Relationships

Determine which users or groups can administer specific computers.

Potential relationships include:

```text id="7x8v1k"
User
  ↓
Group
  ↓
Local Administrators
  ↓
Computer
```

or:

```text id="q3w8r2"
Domain Group
  ↓
Administrative Permission
  ↓
Server
```

This is one of the most important relationships in privilege-escalation analysis.

A user does not need to be a Domain Admin to have significant administrative access to a particular server.

---

## 12. Distinguish Domain Admin from Local Administration

Keep these concepts separate.

### Domain Administration

Provides broad administrative control across the domain or domain resources depending on the exact privilege.

### Local Administration

Provides administrative control over a particular computer.

Example:

```text id="e7r2m4"
CORP\alice
     │
     └── Local Administrator
                │
                ▼
             WEB01
```

Alice may have administrative control over `WEB01` without being a Domain Admin.

This distinction is critical when constructing attack paths.

---

## 13. Identify Management and Administrative Groups

Look for groups associated with system administration.

Examples:

```text id="h5s3c9"
Server-Admins
Workstation-Admins
IT-Admins
Helpdesk
Remote-Management
Backup-Admins
```

These groups may control access to specific computers.

Correlate:

```text id="b2k8v6"
Group
 ↓
Members
 ↓
Computer Access
 ↓
Effective Privilege
```

The actual permissions must be validated.

---

## 14. Identify Services

Where authorized access allows, identify important services running on systems.

Potential service categories include:

```text id="p4m8s2"
Web Services
Database Services
File Services
Remote Management
Backup Services
Application Services
Authentication Services
```

Services may reveal additional relationships between:

```text id="z1c6q9"
Computer
  ↓
Service
  ↓
Account
  ↓
SPN
```

This becomes especially useful when correlating computer and user enumeration.

---

## 15. Identify SMB and RPC Exposure

SMB and RPC are common components of Windows environments.

Relevant ports include:

```text id="4v9m3x"
135  → RPC
139  → NetBIOS Session Service
445  → SMB
```

Where authorized, identify:

* SMB availability
* Accessible shares
* RPC availability
* Host information
* Authentication requirements
* Server identity

Detailed share analysis is covered in:

```text id="7c2q5m"
03-Enumeration/Share-Enumeration.md
```

---

## 16. Identify Remote Management Interfaces

Windows systems may expose management interfaces such as:

```text id="z5n8r3"
WinRM
RDP
WMI
RPC
SMB
```

Common ports include:

```text id="g2m7k4"
3389 → RDP
5985 → WinRM HTTP
5986 → WinRM HTTPS
135  → RPC
445  → SMB
```

Presence of a service does not automatically mean the current user can access it.

Record:

```text id="a7k1p9"
Computer
Service
Port
Authentication Requirement
Observed Access
```

---

## 17. Identify High-Value Systems

Classify systems that may have greater impact if compromised.

Examples include:

```text id="n8v4s6"
Domain Controllers
Certificate Services Servers
Management Servers
Backup Servers
Database Servers
File Servers
Identity Infrastructure
```

At this stage, the goal is identification and prioritization for further authorized analysis.

Do not assume that every high-value system has an exploitable weakness.

---

## 18. Identify Delegation-Related Computer Information

Computer objects can participate in Kerberos delegation configurations.

Potential configurations include:

```text id="w5r3j8"
Unconstrained Delegation
Constrained Delegation
Resource-Based Constrained Delegation
```

During computer enumeration, record relevant delegation attributes where accessible.

Detailed analysis is covered later in:

```text id="q4p7y2"
03-Enumeration/Delegation-Enumeration.md
```

and:

```text id="x8m2c5"
05-Privilege-Escalation-Paths/Delegation-Abuse.md
```

At this stage, identify the configuration rather than attempting abuse.

---

## 19. Build a Computer Inventory

Create a structured inventory.

Example:

```text id="j4c9w7"
Computer:
Hostname:
DNS Name:
IP Address:
Operating System:
Domain:
Site:
Role:
Enabled:
Domain Controller:
Global Catalog:
SPNs:
Groups:
Logged-On Users:
Administrative Groups:
Services:
Remote Management:
Delegation:
Interesting Relationships:
Notes:
```

Example:

```text id="b5k2r9"
Computer: WEB01
IP: 10.10.10.30
Operating System: Windows Server
Role: Web Server

Groups:
- Domain Computers
- Web-Servers

Services:
- HTTP
- SMB
- WinRM

Notes:
- Non-DC server
- Administrative relationships require further analysis
```

---

## 20. Map Users, Groups, and Computers

The most useful result of computer enumeration is the relationship map.

Example:

```text id="6r8p3v"
User
 │
 ▼
Group
 │
 ▼
Computer
 │
 ├── Services
 ├── Shares
 └── Administrative Access
```

A more detailed example:

```text id="2k7m4x"
alice
  │
  ▼
Server-Admins
  │
  ▼
WEB01
  │
  ├── WinRM
  ├── SMB
  └── HTTP
```

This relationship may later become part of an attack path.

---

## 21. Identify Potential Attack Paths

At the enumeration stage, only identify possible relationships.

For example:

```text id="9q3m6f"
User
 ↓
Group
 ↓
Local Admin
 ↓
Server
 ↓
Sensitive Resource
```

or:

```text id="u5v8n2"
User
 ↓
Service Account Relationship
 ↓
SPN
 ↓
Authentication Analysis
```

or:

```text id="h3j7k1"
Computer
 ↓
Delegation Configuration
 ↓
Potential Privilege Relationship
```

These are **potential paths**, not validated vulnerabilities.

Validation happens later.

---

## 22. Assessment Mindset

Ask:

```text
What computers exist?
        ↓
Which are Domain Controllers?
        ↓
Which are servers?
        ↓
Which are workstations?
        ↓
Who can administer them?
        ↓
Which services do they expose?
        ↓
Which accounts are associated with those services?
        ↓
Which users are logged on?
        ↓
Which relationships connect identities to systems?
```

The objective is to understand the **identity-to-system layer** of the AD environment.

---

## Computer Enumeration Checklist

### Computer Discovery

* [ ] Computer objects enumerated
* [ ] Hostnames recorded
* [ ] DNS names recorded
* [ ] IP addresses correlated
* [ ] Operating systems identified
* [ ] Enabled/disabled status recorded

### Domain Infrastructure

* [ ] Domain Controllers identified
* [ ] Global Catalog servers identified
* [ ] AD Sites identified where available
* [ ] High-value systems identified

### Services

* [ ] SPNs identified
* [ ] SMB identified
* [ ] RPC identified
* [ ] RDP identified
* [ ] WinRM identified
* [ ] Other relevant services recorded

### Access Relationships

* [ ] Computer group memberships identified
* [ ] Logged-on users identified where accessible
* [ ] Administrative relationships identified
* [ ] Management groups identified
* [ ] Delegation-related attributes reviewed

### Documentation

* [ ] Computer inventory created
* [ ] User → Computer relationships recorded
* [ ] Group → Computer relationships recorded
* [ ] Service → Computer relationships recorded
* [ ] Potential attack paths documented
* [ ] Evidence sources recorded

---

## Transition to Share Enumeration

Once computer objects and their services are understood, the next step is to examine **network shares and the resources exposed through SMB**.

Move to:

```text id="0x7v3q"
03-Enumeration/Share-Enumeration.md
```

The next stage will connect:

```text
Computer
  ↓
SMB
  ↓
Shares
  ↓
Permissions
  ↓
Files and Resources
  ↓
Users / Groups
  ↓
Potential Access Relationships
```
