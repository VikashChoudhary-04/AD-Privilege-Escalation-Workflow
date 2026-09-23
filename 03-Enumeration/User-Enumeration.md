# User Enumeration

## Purpose

User enumeration is the process of identifying and analyzing user accounts within an Active Directory environment.

After domain enumeration establishes the domain structure, user enumeration focuses on **who exists in the domain, how accounts are configured, and how those accounts relate to groups, services, and privileges**.

The goal is to determine:

* Which user accounts exist
* Which accounts are enabled or disabled
* Usernames and naming conventions
* SIDs and account identifiers
* Group memberships
* Account descriptions
* Service-related accounts
* SPNs associated with users
* Administrative or privileged accounts
* Potentially interesting account relationships

User enumeration should produce a structured inventory rather than simply a list of usernames.

---

## Enumeration Workflow

Use the following workflow:

```text
Identify Enumeration Context
        ↓
Identify User Accounts
        ↓
Collect Account Attributes
        ↓
Identify Enabled/Disabled Accounts
        ↓
Identify Group Membership
        ↓
Identify Privileged Users
        ↓
Identify Service Accounts
        ↓
Identify SPNs
        ↓
Review Descriptions and Metadata
        ↓
Identify Interesting Relationships
        ↓
Record Findings
        ↓
Move to Group Enumeration
```

---

## 1. Identify the Enumeration Context

Before enumerating users, determine the context under which the queries are being performed.

Record:

```text
Current User:
Domain:
Authentication Context:
Local/Domain:
Privileges:
Available Directory Access:
```

Enumeration results can differ significantly depending on whether access is:

* Unauthenticated
* Authenticated as a normal domain user
* Authenticated as a service account
* Performed with administrative privileges

Always record the context alongside the results.

---

## 2. Identify User Accounts

The first objective is to determine which user objects exist in the domain.

Common information includes:

```text
Username
Display Name
Distinguished Name
User SID
UPN
Description
Account Status
```

Example:

```text
Username:       jsmith
Display Name:   John Smith
UPN:            jsmith@corp.example.com
SID:            S-1-5-21-...
Status:         Enabled
```

Possible enumeration sources include:

### Windows

```cmd
whoami
net user /domain
```

### PowerShell

```powershell
Get-ADUser -Filter *
```

Specific attributes:

```powershell
Get-ADUser -Filter * -Properties *
```

### LDAP

```text
ldapsearch
```

### SMB/RPC Enumeration

```text
rpcclient
enum4linux-ng
netexec
```

The exact results depend on the permissions and configuration of the environment.

---

## 3. Understand User Object Attributes

A user object contains much more information than the username.

Useful attributes may include:

| Attribute              | Purpose                                                     |
| ---------------------- | ----------------------------------------------------------- |
| `sAMAccountName`       | Traditional logon name                                      |
| `userPrincipalName`    | UPN used for authentication                                 |
| `objectSid`            | Security identifier                                         |
| `distinguishedName`    | LDAP location                                               |
| `displayName`          | Display name                                                |
| `description`          | Administrative description                                  |
| `memberOf`             | Direct group memberships                                    |
| `servicePrincipalName` | SPNs associated with the account                            |
| `userAccountControl`   | Account-control flags                                       |
| `pwdLastSet`           | Password-change timestamp                                   |
| `lastLogon`            | Logon information                                           |
| `accountExpires`       | Account expiration                                          |
| `adminCount`           | Indicator associated with protected administrative accounts |

Not every attribute will be available in every environment or to every account.

---

## 4. Identify Enabled and Disabled Accounts

Determine whether accounts are currently usable.

Common categories:

```text
Enabled
Disabled
Expired
Locked
Never Used
Inactive
```

Account status should be interpreted carefully.

For example:

```text
Enabled ≠ Active User
```

An account may be enabled but unused or obsolete.

Likewise:

```text
Disabled ≠ Irrelevant
```

Disabled accounts may still provide useful information for understanding historical naming, organizational structure, or relationships.

Record status where available.

---

## 5. Identify Username Conventions

Organizations frequently follow predictable username conventions.

Examples:

```text
firstname.lastname
firstinitiallastname
lastname.firstname
employeeID
service-name
```

For example:

```text
john.smith
jsmith
smith.john
```

Understanding naming conventions helps correlate information discovered through different sources.

Record the observed convention rather than assuming one.

---

## 6. Identify User SIDs

Every domain user has a unique security identifier.

Example:

```text
S-1-5-21-AAAAAAAA-BBBBBBBB-CCCCCCCC-1105
```

The final portion is the Relative Identifier (RID).

Conceptually:

```text
Domain SID
    +
User RID
    =
User SID
```

SIDs are useful when correlating:

* Users
* Groups
* ACLs
* Ownership
* File permissions
* Process tokens
* Security events

Do not rely only on usernames when building an object inventory.

---

## 7. Identify Group Membership

A user's effective access is heavily influenced by group membership.

For each interesting user, identify:

```text
Direct Groups
Nested Groups
Privileged Groups
Resource Groups
Administrative Groups
```

Example:

```text
jsmith
 │
 ├── Domain Users
 │
 ├── IT
 │
 └── Server Operators
```

Nested memberships must also be considered.

For example:

```text
User
 ↓
Group A
 ↓
Group B
 ↓
Privileged Group
```

The user may inherit permissions through the entire chain.

Detailed group analysis is covered in:

```text
03-Enumeration/Group-Enumeration.md
```

---

## 8. Identify Privileged Users

Identify accounts associated with privileged groups or administrative roles.

Examples include users belonging to:

```text
Domain Admins
Enterprise Admins
Administrators
Account Operators
Backup Operators
Server Operators
Schema Admins
```

The exact groups present will vary between environments.

Do not automatically treat every member of a named group as having identical effective access.

Validate:

* Group membership
* Nested membership
* Scope
* Delegated permissions
* ACLs
* Logon restrictions
* Resource-specific permissions

The purpose at this stage is to **identify candidates for deeper analysis**.

---

## 9. Identify Service Accounts

Service accounts deserve separate attention because they may be associated with:

* Windows services
* Scheduled tasks
* Applications
* Databases
* Web applications
* Network services
* Automation systems

Possible indicators include:

```text
service
svc-*
sql-*
app-*
backup-*
```

Naming conventions are only indicators and should not be treated as proof.

For each potential service account, record:

```text
Account
Description
Enabled/Disabled
Group Membership
SPNs
Password-related metadata
Associated Service
```

Service accounts can become particularly important when analyzing authentication and SPNs.

---

## 10. Identify Service Principal Names

A **Service Principal Name (SPN)** associates a service instance with an account.

Examples of service classes include:

```text
HTTP
MSSQLSvc
CIFS
HOST
LDAP
```

An SPN may be associated with a user account or computer account.

Example:

```text
MSSQLSvc/sql01.corp.example.com:1433
        │
        ▼
Service Account
```

Identify users that have SPNs associated with their accounts.

Useful PowerShell query:

```powershell
Get-ADUser -Filter {ServicePrincipalName -like "*"} `
    -Properties ServicePrincipalName
```

SPN enumeration becomes particularly important when moving into authentication-focused analysis.

Detailed Kerberoasting analysis belongs in:

```text
04-Credentials-and-Authentication/Kerberoasting.md
```

At the enumeration stage, focus on **identifying and documenting SPNs**, not blindly attempting attacks.

---

## 11. Review Account Descriptions

Descriptions can contain useful administrative information.

Examples of potentially relevant metadata:

```text
Department
Role
Application
Service
Owner
Location
Administrative Function
Legacy Information
```

Example:

```text
Description:
SQL backup service account
```

or:

```text
Description:
Temporary administrator account
```

Descriptions are informational and should be verified against other directory data.

Do not assume that an old description reflects the account's current role.

---

## 12. Review Account-Control Information

Active Directory user objects contain account-control information.

The `userAccountControl` attribute represents various account settings.

These may indicate conditions such as:

* Account disabled
* Password requirements
* Password expiration behavior
* Delegation-related settings
* Other account-control flags

Example:

```powershell
Get-ADUser jsmith -Properties userAccountControl
```

Interpret these values using the relevant AD documentation or tooling rather than manually guessing the meaning of numeric flags.

---

## 13. Review Password Metadata

Where permitted, collect high-level password-related metadata such as:

```text
Password Last Set
Password Expiration
Account Expiration
```

Example:

```powershell
Get-ADUser jsmith -Properties pwdLastSet,accountExpires
```

This can help identify:

* Old accounts
* Service accounts
* Long-lived accounts
* Potentially stale accounts
* Accounts with unusual lifecycle patterns

Metadata alone does not establish whether an account is vulnerable.

---

## 14. Identify Inactive or Stale Accounts

Look for accounts showing signs of inactivity.

Potential indicators include:

```text
Old last-logon information
Old password-change date
Expired account
Disabled account
Unused service account
Former employee naming
```

However, AD logon attributes have limitations.

For example, `lastLogon` is not necessarily replicated between Domain Controllers in the same way as some other attributes.

Therefore:

```text
Old timestamp ≠ Definitively unused account
```

Correlate multiple indicators before classifying an account as stale.

---

## 15. Identify Administrative Naming Patterns

Organizations sometimes use predictable names for administrative accounts.

Examples:

```text
admin
administrator
adm-*
admin-*
svc-*
```

Do not assume the username alone indicates privilege.

Instead correlate:

```text
Username
      ↓
Group Membership
      ↓
ACLs
      ↓
Delegated Rights
      ↓
Resources
```

A user called `administrator` is not automatically equivalent to every other administrative account.

---

## 16. Identify Interesting User Relationships

User enumeration becomes more useful when relationships are recorded.

Example:

```text
User
 │
 ├── Member Of ──► Group
 │                  │
 │                  └── Access To ──► Computer
 │
 ├── Has SPN ─────► Service
 │
 └── Owns ────────► Object
```

Other relationships worth recording include:

* User → Group
* User → Computer
* User → SPN
* User → Service
* User → Object ownership
* User → Delegated permissions
* User → Administrative role

These relationships become important during attack-path analysis.

---

## 17. Useful Enumeration Tools

### Windows Built-in Tools

```text
whoami
net user /domain
net group /domain
nltest
```

### PowerShell

```text
Get-ADUser
Get-ADGroupMember
Get-ADObject
Get-ADComputer
```

### LDAP

```text
ldapsearch
ldapwhoami
```

### SMB/RPC

```text
rpcclient
enum4linux-ng
netexec
```

### PowerView / SharpView

```text
Get-DomainUser
Get-DomainGroupMember
Get-DomainUser -SPN
Get-DomainObject
```

### Graph-Based Collection

```text
BloodHound-compatible collectors
```

Different tools may expose different attributes or relationships. Correlate results instead of treating one tool's output as the complete directory view.

---

## 18. Build a User Inventory

Create a structured inventory.

Example:

```text
Username:
Display Name:
UPN:
SID:
Status:
Description:
Groups:
Privileged Groups:
SPNs:
Password Last Set:
Last Logon:
Account Expiration:
Interesting Relationships:
Notes:
```

For example:

```text
Username: jsmith
UPN: jsmith@corp.example.com
Status: Enabled
Groups:
- Domain Users
- IT

SPNs:
- None

Notes:
- Standard domain account
```

For an interesting account:

```text
Username: svc-sql
UPN: svc-sql@corp.example.com
Status: Enabled
Groups:
- Domain Users
- SQL Operators

SPNs:
- MSSQLSvc/sql01.corp.example.com:1433

Notes:
- Service account
- SPN requires authentication-focused review
```

Use fictional examples when documenting the workflow.

---

## 19. Separate Facts from Assumptions

User enumeration can generate many misleading indicators.

For example:

```text
Old password date
        ↓
Possible stale account
```

is an observation, not proof.

Likewise:

```text
SPN exists
        ↓
Service account
```

is not necessarily true because computer accounts can also have SPNs.

Use three categories:

```text
Observed
    ↓
Potentially Interesting
    ↓
Validated
```

This keeps the enumeration process evidence-driven.

---

## 20. Assessment Mindset

Ask:

```text
Who are the users?
        ↓
Which accounts are active?
        ↓
Which accounts are privileged?
        ↓
Which accounts are service-related?
        ↓
Which users have SPNs?
        ↓
Which accounts belong to important groups?
        ↓
Which relationships require deeper investigation?
```

The goal is not to attack every discovered account.

The goal is to identify **accounts and relationships that can explain how privileges are assigned within the domain**.

---

## User Enumeration Checklist

### Account Discovery

* [ ] Domain users enumerated
* [ ] Usernames recorded
* [ ] UPNs recorded
* [ ] User SIDs recorded
* [ ] Distinguished Names recorded

### Account Status

* [ ] Enabled accounts identified
* [ ] Disabled accounts identified
* [ ] Expired accounts identified where available
* [ ] Potentially stale accounts identified

### Attributes

* [ ] Descriptions reviewed
* [ ] Account-control information reviewed
* [ ] Password metadata reviewed
* [ ] Logon metadata reviewed
* [ ] Account expiration reviewed

### Groups and Privileges

* [ ] Direct group memberships identified
* [ ] Nested memberships considered
* [ ] Privileged users identified
* [ ] Administrative accounts identified

### Service Accounts

* [ ] Service-account candidates identified
* [ ] SPNs enumerated
* [ ] Service relationships recorded
* [ ] Authentication-related follow-up identified

### Documentation

* [ ] User inventory created
* [ ] Interesting accounts documented
* [ ] Relationships recorded
* [ ] Observations separated from assumptions
* [ ] Enumeration context recorded

---

## Transition to Group Enumeration

Once users have been identified, the next step is to understand the **groups that control access and privilege**.

Move to:

```text
03-Enumeration/Group-Enumeration.md
```

The next stage will focus on:

```text
Groups
  ↓
Group Membership
  ↓
Nested Groups
  ↓
Privileged Groups
  ↓
Delegated Groups
  ↓
Group Relationships
  ↓
Effective Privilege
```

Understanding group relationships is essential because AD privileges are often inherited through **nested memberships and delegated permissions**, rather than being assigned directly to individual users.
