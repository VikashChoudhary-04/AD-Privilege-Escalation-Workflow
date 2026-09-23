# SPN Enumeration

## Purpose

Service Principal Name (SPN) enumeration is the process of identifying service accounts and computer accounts associated with registered Kerberos service principals in Active Directory.

An SPN establishes a relationship between:

```text id="r3x8p6"
Service
  ↓
SPN
  ↓
AD Account
  ↓
Kerberos Authentication
```

SPNs are important during Active Directory security assessments because they can reveal:

* Services running in the domain
* Service-host relationships
* Accounts associated with services
* Service naming conventions
* User accounts configured with SPNs
* Computer accounts configured with SPNs
* Potential Kerberos authentication attack surface

The purpose of this stage is **enumeration and relationship mapping**.

Detailed authentication attacks such as Kerberoasting are covered later in:

```text id="4x9m2c"
04-Credentials-and-Authentication/Kerberoasting.md
```

---

## Enumeration Workflow

Use the following workflow:

```text
Identify Kerberos Environment
        ↓
Enumerate SPNs
        ↓
Identify SPN-Associated Accounts
        ↓
Classify User vs Computer Accounts
        ↓
Identify Service Types
        ↓
Identify Service Hosts
        ↓
Review Account Context
        ↓
Identify Interesting Service Accounts
        ↓
Correlate with Groups and Computers
        ↓
Record Findings
        ↓
Move to Trust Enumeration
```

---

## 1. Understand SPNs

An SPN is an identifier used by Kerberos to associate a service instance with an account.

A simplified format is:

```text id="v7g2m4"
SERVICE/HOSTNAME:PORT
```

Examples:

```text id="4y6p8x"
HTTP/web01.corp.example.com
MSSQLSvc/sql01.corp.example.com:1433
CIFS/file01.corp.example.com
```

The exact SPN format depends on the service.

---

## 2. Understand the Account Behind an SPN

An SPN is registered against an AD security principal.

That principal may be:

```text id="x6c3m9"
User Account
Computer Account
```

Example:

```text id="q8v2k5"
MSSQLSvc/sql01.corp.example.com:1433
             │
             ▼
          svc-sql
```

or:

```text id="h4n7p1"
HOST/dc01.corp.example.com
          │
          ▼
         DC01$
```

Do not assume that every SPN belongs to a user/service account.

---

## 3. Enumerate SPNs

PowerShell:

```powershell id="z3k8q5"
Get-ADObject -Filter {ServicePrincipalName -like "*"} `
    -Properties ServicePrincipalName
```

For user accounts:

```powershell id="6v2m9c"
Get-ADUser -Filter {ServicePrincipalName -like "*"} `
    -Properties ServicePrincipalName
```

For computer accounts:

```powershell id="w4p7x1"
Get-ADComputer -Filter {ServicePrincipalName -like "*"} `
    -Properties ServicePrincipalName
```

Other authorized tools can enumerate SPNs through LDAP queries.

Record:

```text id="g8r5m2"
SPN
Account
Account Type
Host
Service
Port
```

---

## 4. Identify Service Classes

The first part of an SPN often identifies the service class.

Common examples include:

```text id="8m4q2p"
HTTP
MSSQLSvc
CIFS
HOST
LDAP
WSMAN
TERMSRV
```

Examples:

```text id="x7n3v5"
HTTP/web01.corp.example.com
```

```text id="p6k2r8"
MSSQLSvc/sql01.corp.example.com:1433
```

```text id="q4m9c7"
CIFS/file01.corp.example.com
```

Service classes provide clues about the systems and applications present in the environment.

---

## 5. Identify Service Hosts

Parse the hostname associated with each SPN.

Example:

```text id="t8v5n2"
MSSQLSvc/sql01.corp.example.com:1433
          │
          ▼
        SQL01
```

Then correlate the host with the computer inventory.

```text id="3x6q9m"
SPN
 ↓
SQL01
 ↓
Computer Object
 ↓
Operating System
 ↓
Groups
 ↓
Services
```

This connects authentication information to the underlying system.

---

## 6. Identify Service Ports

Some SPNs explicitly contain a port.

Example:

```text id="1m7c4x"
MSSQLSvc/sql01.corp.example.com:1433
                                  │
                                  ▼
                                Port
```

Record the port when present.

This helps correlate:

```text id="6p9v2k"
SPN
 ↓
Service
 ↓
Port
 ↓
Network Enumeration
```

An SPN without an explicit port can still represent a service using a standard or service-specific port.

---

## 7. Identify User-Account SPNs

User accounts can be configured to run services.

Example:

```text id="5n8r3q"
svc-sql
   │
   └── MSSQLSvc/sql01.corp.example.com:1433
```

Identify user accounts with SPNs.

Record:

```text id="2k7x5m"
Account:
SPN:
Service:
Host:
Groups:
Description:
Account Status:
```

These accounts are particularly relevant to later Kerberos authentication analysis.

---

## 8. Identify Computer-Account SPNs

Computer accounts commonly possess SPNs.

Example:

```text id="9c4m7v"
DC01$
 ├── HOST/dc01.corp.example.com
 ├── LDAP/dc01.corp.example.com
 └── ...
```

Do not treat every computer SPN as an unusual finding.

Computer SPNs are often expected.

The important task is to distinguish:

```text id="x2v8q4"
Normal Computer SPN
```

from:

```text id="h7m3p9"
Interesting User-Account SPN
```

and then validate the context.

---

## 9. Identify Duplicate SPNs

SPNs should normally uniquely identify a service instance.

Duplicate SPNs can cause Kerberos authentication problems and may indicate configuration issues.

During enumeration, look for:

```text id="4q8n2c"
Same SPN
   ↓
Multiple Accounts
```

Record:

```text id="u5m7x3"
SPN
Account 1
Account 2
Host
Service
```

Do not automatically classify a duplicate as a security vulnerability.

First establish why the duplication exists.

---

## 10. Identify Service Account Naming Patterns

Service accounts often use recognizable naming conventions.

Examples:

```text id="m2p7v9"
svc-*
sql-*
app-*
web-*
backup-*
```

These are only indicators.

A normal user account can also run a service, and a service account may have an unrelated name.

Correlate the naming pattern with:

* SPN
* Description
* Group membership
* Host
* Service
* Account type

---

## 11. Correlate SPNs with Computer Enumeration

Use the computer inventory created earlier.

Example:

```text id="f6v3k9"
SPN:
MSSQLSvc/sql01.corp.example.com:1433

        ↓

Computer:
SQL01

        ↓

Role:
Database Server
```

Then determine:

```text id="j4n8q2"
Service Account
        ↓
SQL01
        ↓
MSSQL
```

This establishes the service dependency.

---

## 12. Correlate SPNs with User Enumeration

Similarly connect SPNs to the user inventory.

Example:

```text id="s7m2x5"
User:
svc-sql

SPN:
MSSQLSvc/sql01.corp.example.com:1433

Groups:
- Domain Users
- SQL-Admins
```

This creates a relationship:

```text id="c9p4v7"
User
 ↓
Service Account
 ↓
SPN
 ↓
Service
 ↓
Computer
```

This relationship becomes important during credential and authentication analysis.

---

## 13. Review Service Account Privileges

Do not stop after finding an SPN.

Determine what groups the associated account belongs to.

For example:

```text id="n5x8m3"
svc-sql
   │
   ├── Domain Users
   └── SQL-Admins
```

Then ask:

```text id="u7q2k6"
What does SQL-Admins control?
```

The SPN itself does not establish privilege.

The account's:

* Group membership
* ACLs
* Delegated rights
* Resource access

determine its broader security context.

---

## 14. Review Account Status

For each user account associated with an SPN, determine:

```text id="e4m9r2"
Enabled
Disabled
Expired
Password Metadata
Account Expiration
```

An SPN associated with a disabled account may still be useful as configuration information, but it should not automatically be treated as an active authentication path.

---

## 15. Review Account Descriptions

Descriptions may reveal the purpose of a service account.

Example:

```text id="p8c4x7"
Account:
svc-backup

Description:
Backup service account
```

Correlate the description with:

```text id="m3v6n9"
SPN
Host
Service
Groups
Account Status
```

Do not assume descriptions are current.

---

## 16. Identify Interesting Service Types

Prioritize understanding services that commonly have dedicated accounts.

Examples:

```text id="2q7m5x"
SQL Server
Web Applications
File Services
Application Servers
Remote Management
Custom Enterprise Applications
```

For each service:

```text id="c8v4p1"
Service
 ↓
SPN
 ↓
Account
 ↓
Computer
```

Document the complete relationship.

---

## 17. Understand Kerberos and SPNs

Kerberos uses SPNs when requesting service tickets.

Simplified flow:

```text id="y4m8c2"
User
 ↓
KDC
 ↓
Service Ticket
 ↓
SPN
 ↓
Service Account / Computer
```

This is why SPNs are closely related to Kerberos authentication.

During later credential analysis, service accounts with SPNs may become relevant to **Kerberoasting**.

At this stage:

```text id="s2p6x8"
Enumerate
   ↓
Understand
   ↓
Document
```

Do not immediately move from discovery to attack execution.

---

## 18. Identify Potential Kerberoasting Candidates

During enumeration, identify user accounts that:

* Have one or more SPNs
* Are enabled
* Represent service accounts or service-like accounts
* Have potentially significant privileges

Example:

```text id="v5n2m7"
svc-sql
   │
   ├── Enabled
   ├── SPN present
   └── SQL-Admins
```

This may justify deeper authentication analysis.

However:

```text id="r8c3q1"
SPN + User Account
       ≠
Confirmed Vulnerability
```

The later Kerberoasting stage evaluates the authentication risk.

---

## 19. Identify Potentially Sensitive Services

Some SPNs may reveal services with significant infrastructure roles.

Examples:

```text id="p7x4m2"
MSSQLSvc
LDAP
CIFS
HTTP
WSMAN
```

The significance depends on:

* Account privileges
* Host role
* Network exposure
* Authentication configuration
* Service configuration

Use SPN enumeration to build context rather than assigning severity from the service name alone.

---

## 20. Useful Enumeration Tools

### PowerShell

```text id="z8m4c1"
Get-ADUser
Get-ADComputer
Get-ADObject
```

### setspn

Windows includes `setspn` for querying and managing SPNs.

Example query:

```cmd id="p6x2n9"
setspn -Q */*
```

Specific account:

```cmd id="c4v8m1"
setspn -L svc-sql
```

Use administrative modification capabilities only when explicitly required and authorized.

For enumeration, querying is sufficient.

### LDAP

```text id="h2q7v5"
ldapsearch
```

### PowerView

Common capabilities include:

```text id="m9x3k6"
Get-DomainUser -SPN
Get-DomainComputer -SPN
Get-DomainObject
```

### BloodHound-Compatible Collection

```text id="v4c8p2"
BloodHound-compatible collectors
```

Graph collection can correlate:

```text id="d6m2q8"
User
 ↓
SPN
 ↓
Computer
 ↓
Service
```

---

## 21. Build an SPN Inventory

Create a structured inventory.

Example:

```text id="q7x4m9"
SPN:
Service Class:
Host:
Port:
Account:
Account Type:
Enabled:
Description:
Groups:
Computer Role:
Potential Authentication Relevance:
Notes:
```

Example:

```text id="b5n8c3"
SPN:
MSSQLSvc/sql01.corp.example.com:1433

Service:
MSSQLSvc

Host:
SQL01

Account:
svc-sql

Account Type:
User

Groups:
- Domain Users
- SQL-Admins

Notes:
- Enabled service account
- Requires Kerberos authentication review
```

---

## 22. Build a Service Relationship Map

Connect SPNs to the broader AD model.

```text id="k4m7p2"
User Account
      │
      ▼
     SPN
      │
      ▼
   Service
      │
      ▼
   Computer
      │
      ▼
Computer Role
```

Example:

```text id="n8v3x5"
svc-sql
   ↓
MSSQLSvc/sql01.corp.example.com:1433
   ↓
MSSQL
   ↓
SQL01
   ↓
Database Server
```

This relationship can later be combined with:

```text id="a6q2m9"
Group Membership
ACLs
Delegation
Credentials
```

to identify potential attack paths.

---

## 23. Separate Normal from Interesting SPNs

Not every SPN is equally useful.

Classify findings as:

```text id="w5c8p3"
Expected
    ↓
Interesting
    ↓
Requires Validation
```

For example:

```text id="z7m2q4"
DC01$
 └── HOST/DC01
```

may be expected.

Whereas:

```text id="x4p9v6"
svc-database
 └── MSSQLSvc/sql01:1433
```

may require deeper investigation because it identifies a user account associated with a service.

---

## 24. Assessment Mindset

Ask:

```text
Which SPNs exist?
        ↓
Which accounts own them?
        ↓
Are those accounts users or computers?
        ↓
Which services do they represent?
        ↓
Which computers host those services?
        ↓
What groups and privileges do the accounts have?
        ↓
Does the relationship require authentication-focused analysis?
```

The goal is to transform:

```text id="j2k7m4"
SPN
```

into:

```text id="v8q3p6"
Account → Service → Host → Privilege Context
```

---

## SPN Enumeration Checklist

### SPN Discovery

* [ ] SPNs enumerated
* [ ] User-account SPNs identified
* [ ] Computer-account SPNs identified
* [ ] Service classes identified
* [ ] Hosts identified
* [ ] Ports recorded where present

### Account Analysis

* [ ] Associated accounts identified
* [ ] Account type recorded
* [ ] Account status reviewed
* [ ] Group membership reviewed
* [ ] Descriptions reviewed
* [ ] Service-account candidates identified

### Service Analysis

* [ ] Service relationships documented
* [ ] Computer relationships documented
* [ ] Duplicate SPNs checked
* [ ] Interesting service types identified
* [ ] High-value service relationships recorded

### Authentication Analysis

* [ ] Kerberos relationship understood
* [ ] Potential Kerberoasting candidates identified
* [ ] Authentication follow-up documented
* [ ] No assumptions made from SPN presence alone

### Documentation

* [ ] SPN inventory created
* [ ] Account → SPN relationships recorded
* [ ] SPN → Host relationships recorded
* [ ] Service relationships recorded
* [ ] Potential authentication paths documented

---

## Transition to Trust Enumeration

SPN enumeration completes the service and Kerberos relationship layer of the current enumeration phase.

The next step is to examine **trust relationships between domains and forests**.

Move to:

```text id="7m4x8q"
03-Enumeration/Trust-Enumeration.md
```

The next stage will connect:

```text
Domain
  ↓
Trust
  ↓
Other Domain / Forest
  ↓
Authentication Relationship
  ↓
Cross-Domain Access
  ↓
Potential Attack Path
```
