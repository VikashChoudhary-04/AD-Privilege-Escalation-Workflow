# Password Reuse

## Purpose

Password reuse analysis identifies cases where the same password or authentication secret is used across multiple accounts, systems, services, or applications.

The objective is to determine whether one discovered credential provides access beyond its original context.

The workflow is:

```text
Discover
   │
   ▼
Correlate
   │
   ▼
Validate
   │
   ▼
Map Privileges
   │
   ▼
Identify Access Path
   │
   ▼
Document
```

Password reuse should be assessed only within authorized systems and accounts.

---

## Position in the Workflow

```text
04-Credentials-and-Authentication
              │
              ├── Credential Discovery
              │
              ├── Password Reuse  ← Current
              │
              ├── Kerberoasting
              ├── AS-REP Roasting
              └── Credential Dumping
              │
              ▼
05-Privilege-Escalation-Paths
```

Credential discovery identifies authentication material.

Password-reuse analysis determines whether that material appears to be valid in additional contexts.

---

## What Is Password Reuse?

Password reuse occurs when the same authentication secret is used for multiple identities or services.

For example:

```text
Password A
    │
    ├── svc-web
    ├── local-admin
    └── backup-service
```

If the same password is used by several accounts, compromise of one authentication context may expose additional access.

However:

```text
Same password observed
        ≠
Confirmed access everywhere
```

Each account and authentication context must be validated independently.

---

## Password-Reuse Workflow

```text
Credential Discovered
        │
        ▼
Identify Credential Owner
        │
        ▼
Identify Other Candidate Accounts
        │
        ▼
Compare Credential Material
        │
        ▼
Perform Controlled Validation
        │
        ▼
Identify Successful Authentication
        │
        ▼
Determine Authorization
        │
        ▼
Map Privileges
        │
        ▼
Identify Potential Attack Path
        │
        ▼
Document Evidence
```

---

## 1. Identify the Original Credential

Start with a credential discovered during:

* Credential discovery
* Service enumeration
* Application configuration review
* Authorized credential testing
* Password auditing
* Other approved assessment activities

Record:

```text
Account:
System:
Credential Source:
Credential Type:
Discovery Location:
Current Status:
```

Example:

```text
Account:
svc-web

Source:
Application configuration

Credential Type:
Password

System:
WEB01

Status:
Requires validation
```

---

## 2. Identify Candidate Accounts

After identifying the original account, determine which other accounts may be relevant.

Useful account categories include:

```text
Domain Users
Service Accounts
Administrative Accounts
Local Accounts
Application Accounts
Backup Accounts
Deployment Accounts
Database Accounts
```

The objective is not to blindly test every account.

Instead, use previously collected enumeration data to identify meaningful relationships.

---

## 3. Use Existing AD Enumeration

The earlier enumeration stages provide useful context.

Relevant information includes:

```text
Users
Groups
Computers
Service Accounts
SPNs
ACLs
GPOs
Delegation
```

For example:

```text
svc-web
   │
   ├── Member of WebAdmins
   ├── SPN: HTTP/web01
   └── Application configuration
          │
          ▼
       Password
```

If the same credential is associated with another account or service, record the relationship.

---

## 4. Distinguish Account Types

Password reuse has different implications depending on the account type.

### Domain Account

```text
EXAMPLE\svc-web
```

The credential may authenticate to domain-controlled services depending on authorization.

### Local Account

```text
WEB01\Administrator
```

The credential applies to a specific local security context unless the same password has been reused elsewhere.

### Service Account

```text
EXAMPLE\svc-database
```

The account may have access to specific services or systems.

### Application Account

```text
app_user
```

The account may exist only within an application or database.

Do not assume that identical usernames represent the same security principal.

---

## 5. Compare Credential Material

When multiple credential sources are available, compare them carefully.

Possible sources include:

```text
Configuration files
Scripts
Service configurations
Scheduled tasks
Password audit results
Credential stores
Authorized credential dumps
```

A finding might look like:

```text
Source A
    │
    └── svc-web → Password X

Source B
    │
    └── backup-service → Password X
```

This suggests possible reuse.

It does not yet prove that both accounts currently accept the same password.

---

## 6. Validate Suspected Reuse

Validation should be controlled and minimal.

Use the following sequence:

```text
Suspected Reuse
      │
      ▼
Confirm Target Account
      │
      ▼
Confirm Account Status
      │
      ▼
Confirm Target System
      │
      ▼
Controlled Authentication Test
      │
      ▼
Observe Result
```

Possible outcomes:

```text
Successful Authentication
Failed Authentication
Account Disabled
Authentication Restricted
Credential Expired
Account Locked
```

A failed authentication attempt should not automatically be interpreted as proof that the password is incorrect. Other controls may be responsible.

---

## 7. Authentication vs Authorization

A successful login does not automatically provide administrative access.

Separate the two concepts:

```text
Credential
   │
   ▼
Authentication
   │
   ▼
Identity Established
   │
   ▼
Authorization
   │
   ▼
Permissions
```

For example:

```text
svc-web
   │
   ├── Authentication: Successful
   │
   └── Authorization:
           └── Application access only
```

The account may authenticate successfully without providing privilege escalation.

---

## 8. Map the Reused Credential

Once reuse is validated, create a relationship map.

```text
Credential
    │
    ├──────────────┐
    ▼              ▼
Account A       Account B
    │              │
    ▼              ▼
System A        System B
    │              │
    ▼              ▼
Privileges A    Privileges B
```

This allows the assessment to determine whether the reuse expands access.

---

## 9. Correlate With Group Membership

For domain accounts, group membership is important.

Example:

```powershell
Get-ADPrincipalGroupMembership svc-web |
    Select-Object Name
```

For another account:

```powershell
Get-ADPrincipalGroupMembership backup-service |
    Select-Object Name
```

Then compare:

```text
svc-web
   │
   └── WebAdmins

backup-service
   │
   └── BackupOperators
```

The second account may have substantially different privileges even though the same password is used.

---

## 10. Correlate With Local Privileges

For Windows systems, determine whether the reused credential provides local administrative access.

For example:

```powershell
whoami /groups
```

and:

```powershell
net localgroup administrators
```

The goal is to establish:

```text
Credential
   │
   ▼
Account
   │
   ▼
Local Group Membership
   │
   ▼
Administrative Access?
```

Do not assume that membership in a similarly named group has the same privileges across different systems.

---

## 11. Correlate With Service Accounts

Service accounts deserve additional attention because they may be:

* Long-lived
* Used by multiple applications
* Assigned elevated permissions
* Associated with SPNs
* Used on multiple systems

Example:

```text
svc-app
   │
   ├── Web Server
   ├── Database
   └── Scheduled Task
```

If the same password is reused across another privileged service account, the relationship may become more significant.

The actual privilege impact must still be established through evidence.

---

## 12. Correlate With SPNs

A reused credential may belong to an account with one or more SPNs.

Example:

```text
svc-sql
   │
   ├── SPN: MSSQLSvc/DB01
   ├── Group: DatabaseAdmins
   └── Credential reused elsewhere
```

This connects password reuse with the Kerberos attack surface.

Record:

```text
Account
SPN
Service
Host
Groups
Privileges
Credential Source
```

This information can later be correlated with Kerberoasting analysis.

---

## 13. Correlate With ACLs

An account may have significant privileges through object permissions rather than group membership.

The relationship may look like:

```text
Reused Credential
       │
       ▼
Account
       │
       ▼
ACL Permission
       │
       ▼
AD Object
       │
       ▼
Potential Privilege Path
```

Relevant permissions may include control over:

* User objects
* Computer objects
* Groups
* GPOs
* Service accounts
* Delegation-related attributes

A reused credential should therefore be correlated with the ACL enumeration already completed.

---

## 14. Local vs Domain Password Reuse

These should be documented separately.

### Local Reuse

```text
WEB01\Administrator
      │
      ├── WEB01
      ├── APP01
      └── DB01
```

The same local administrative password may be configured across multiple systems.

### Domain Reuse

```text
EXAMPLE\svc-web
        │
        ├── Application
        ├── Service
        └── Administrative Group
```

Domain credentials are controlled differently from local accounts.

Do not confuse:

```text
WEB01\admin
```

with:

```text
EXAMPLE\admin
```

They may represent completely different security principals.

---

## 15. Password Reuse Across Services

A single account may be used for multiple services.

For example:

```text
svc-app
   │
   ├── Windows Service
   ├── Scheduled Task
   ├── Web Application
   └── Database Connection
```

If the password is exposed through one service's configuration, it may affect other services using the same account.

Document the relationship:

```text
Credential Source
        │
        ▼
Account
        │
        ├── Service A
        ├── Service B
        └── Application C
```

---

## 16. Determine Credential Scope

For every suspected reuse finding, determine:

```text
Where does it work?
Where does it not work?
Which account owns it?
Which system accepts it?
What permissions does it provide?
```

Example:

```text
Credential:
Password X

Observed:
svc-web on WEB01

Validated:
svc-web on WEB01

Not established:
Other domain accounts
Other servers
Domain Administrator
```

This prevents overstatement in reports.

---

## 17. Identify Potential Attack Paths

Once reuse is validated, correlate it with privileges.

Example:

```text
Password Reuse
      │
      ▼
svc-backup
      │
      ├── BackupOperators
      │
      └── Access to BACKUP01
              │
              ▼
       Sensitive Resources
```

Another example:

```text
Password Reuse
      │
      ▼
Domain Service Account
      │
      ▼
Privileged Group
      │
      ▼
Administrative Resource
```

The second relationship may warrant deeper investigation, but the actual path must be validated rather than assumed.

---

## 18. Avoid Account Lockouts

Password-reuse testing can create account-lockout risk.

Before validation, determine:

* Whether the account has lockout protection
* Whether authentication failures are monitored
* Whether the assessment permits authentication testing
* Whether a safe validation method exists

Avoid unnecessary password guessing.

The goal is to validate **known credential material**, not perform uncontrolled brute-force activity.

---

## 19. Credential Rotation

Password reuse may change during an assessment.

A credential that worked earlier may later become invalid because of:

* Password rotation
* Account disablement
* Service migration
* Credential revocation
* Administrative remediation

Record the time and context of validation.

Example:

```text
Validated:
2026-09-23

Account:
svc-web

System:
WEB01

Result:
Authentication successful
```

This makes the finding reproducible and time-bounded.

---

## 20. Evidence Collection

For every confirmed reuse relationship, record:

```text
Finding:
Original Credential Source:
Original Account:
Reused Account:
Target System:
Authentication Result:
Authorization Result:
Groups:
Privileges:
Validation Time:
Evidence:
Potential Impact:
```

Example:

```text
Finding:
Credential reused across application and service account

Original Source:
Application configuration

Original Account:
svc-web

Reused Context:
Authorized service configuration

Authentication:
Successful

Authorization:
Application-level access

Privilege:
To be correlated with AD permissions
```

Never include actual passwords in the repository.

Use:

```text
PASSWORD=<REDACTED>
```

instead.

---

## Common False Positives

### Same Username

Two accounts may have the same username in different security contexts.

```text
WEB01\admin
EXAMPLE\admin
```

These are not automatically the same account.

### Similar Configuration

Two applications may contain the same placeholder password.

```text
password=changeme
```

This does not establish real credential reuse.

### Stale Credentials

A password stored in an old backup may no longer work.

### Disabled Accounts

A reused credential associated with a disabled account does not provide current access.

### Authentication Without Privilege

Successful authentication does not prove administrative access.

---

## Common Mistakes

### Mistake 1: Brute-Forcing Instead of Validating

If a known credential is available, validate it carefully rather than performing unnecessary guessing.

### Mistake 2: Ignoring Account Type

Local and domain accounts must be analyzed separately.

### Mistake 3: Assuming Successful Login Equals Admin

Authentication and authorization are different.

### Mistake 4: Ignoring Existing Enumeration

Group, ACL, SPN, and delegation data can significantly change the meaning of credential reuse.

### Mistake 5: Testing Outside Scope

Do not use discovered credentials against unrelated systems.

### Mistake 6: Publishing Credentials

Never commit passwords, hashes, tokens, or private keys to the repository.

---

## Password-Reuse Checklist

### Discovery

* [ ] Identified original credential
* [ ] Identified credential source
* [ ] Identified associated account
* [ ] Identified candidate reuse contexts

### Correlation

* [ ] Checked domain accounts
* [ ] Checked local accounts
* [ ] Checked service accounts
* [ ] Checked application accounts
* [ ] Correlated users and groups
* [ ] Correlated SPNs
* [ ] Correlated ACLs
* [ ] Correlated computer access

### Validation

* [ ] Confirmed account status
* [ ] Validated only known credentials
* [ ] Stayed within authorized scope
* [ ] Avoided unnecessary authentication attempts
* [ ] Considered account-lockout risk
* [ ] Recorded validation time

### Privilege Analysis

* [ ] Determined authentication result
* [ ] Determined authorization result
* [ ] Identified group memberships
* [ ] Identified local privileges
* [ ] Identified domain privileges
* [ ] Identified accessible resources
* [ ] Identified potential attack paths

### Documentation

* [ ] Recorded evidence
* [ ] Sanitized credentials
* [ ] Distinguished confirmed access from assumptions
* [ ] Documented limitations
* [ ] Protected sensitive material

---

## Transition to Kerberoasting

After analyzing password reuse, move to service-account authentication and Kerberos service tickets.

```text
Password Reuse
      │
      ▼
Account Context
      │
      ▼
Service Accounts
      │
      ▼
SPNs
      │
      ▼
Kerberos
      │
      ▼
Kerberoasting
```

The next file is:

**`04-Credentials-and-Authentication/Kerberoasting.md`**
