# Kerberoasting

## Purpose

Kerberoasting is an Active Directory credential-access technique that targets accounts associated with Kerberos Service Principal Names (SPNs).

The assessment objective is to:

* Identify accounts associated with service principals
* Understand which services those accounts represent
* Determine whether the accounts are suitable for authorized Kerberos ticket analysis
* Assess the strength and management of associated service-account passwords
* Correlate service accounts with groups, ACLs, delegation, and privileges
* Determine whether a service account could create a privilege-escalation path

The workflow is:

```text id="2n3r6x"
Enumerate SPNs
      │
      ▼
Identify Service Accounts
      │
      ▼
Identify Services
      │
      ▼
Assess Account Context
      │
      ▼
Request / Obtain Service Ticket
      │
      ▼
Perform Authorized Offline Analysis
      │
      ▼
Assess Credential Strength
      │
      ▼
Map Account Privileges
      │
      ▼
Identify Potential Path
      │
      ▼
Document
```

Kerberoasting should be performed only against authorized Active Directory environments.

---

## Position in the Workflow

```text id="u9q2xk"
04-Credentials-and-Authentication
              │
              ├── Credential Discovery
              ├── Password Reuse
              │
              ├── Kerberoasting  ← Current
              │
              ├── AS-REP Roasting
              └── Credential Dumping
              │
              ▼
05-Privilege-Escalation-Paths
```

Kerberoasting builds on the earlier **SPN Enumeration** performed in Section 03.

---

## Kerberos and SPNs

Kerberos uses tickets to authenticate users and services.

A simplified service-authentication flow is:

```text id="a8n6pk"
User
 │
 │ Requests service ticket
 ▼
Domain Controller / KDC
 │
 │ Issues ticket
 ▼
Service
```

A **Service Principal Name (SPN)** identifies a service instance associated with an account.

Conceptually:

```text id="n9xk5j"
Service Account
      │
      ▼
SPN
      │
      ▼
Service
      │
      ▼
Host
```

Example:

```text id="v7j3nd"
svc-sql
   │
   └── MSSQLSvc/db01.example.local:1433
```

The SPN identifies the SQL service, while the account controls the service identity.

---

## Why SPNs Matter

Kerberos service tickets are encrypted in a way that involves the account associated with the service.

If an account uses a password that can be guessed offline, an attacker who obtains an appropriate service-ticket representation may be able to perform offline password analysis.

The important relationship is:

```text id="x5f8bh"
SPN
 │
 ▼
Service Account
 │
 ▼
Service Ticket
 │
 ▼
Offline Password Analysis
 │
 ▼
Potential Credential Recovery
```

The security significance depends heavily on:

* Password strength
* Password age
* Account privileges
* Service-account management
* Exposure of the service
* Existing AD permissions

---

## Kerberoasting Workflow

```text id="y4p2o8"
Start
 │
 ▼
Enumerate SPNs
 │
 ▼
Filter Relevant Service Accounts
 │
 ▼
Review Account Properties
 │
 ▼
Identify Service / Host
 │
 ▼
Assess Privileges
 │
 ▼
Obtain Service Ticket
 │
 ▼
Save Evidence Securely
 │
 ▼
Offline Password Analysis
 │
 ▼
Credential Validation
 │
 ▼
Privilege Correlation
 │
 ▼
Document
```

---

## 1. Enumerate SPNs

Start by identifying accounts with SPNs.

### PowerShell

```powershell id="3i0v3p"
Get-ADUser -Filter * -Properties ServicePrincipalName |
    Where-Object {$_.ServicePrincipalName} |
    Select-Object SamAccountName, ServicePrincipalName
```

For computer accounts:

```powershell id="8bq6v4"
Get-ADComputer -Filter * -Properties ServicePrincipalName |
    Where-Object {$_.ServicePrincipalName} |
    Select-Object Name, ServicePrincipalName
```

A broader directory query can also be used:

```powershell id="a2w5jk"
Get-ADObject -LDAPFilter "(servicePrincipalName=*)" `
    -Properties servicePrincipalName |
    Select-Object DistinguishedName, servicePrincipalName
```

The purpose is to establish:

```text id="h6g5z1"
Account
   │
   └── SPN
          │
          └── Service / Host
```

---

## 2. Identify Relevant Service Accounts

Not every SPN represents a user-managed service account.

Common service-related account types include:

* Domain user service accounts
* Managed service accounts
* Group Managed Service Accounts
* Computer accounts
* Application-specific accounts

Record the account type before proceeding.

Example:

```text id="8j73sl"
Account:
svc-web

Type:
Domain user

SPN:
HTTP/web01.example.local
```

Compare that with:

```text id="uw4i4b"
Account:
WEB01$

Type:
Computer account

SPN:
HOST/web01.example.local
```

These have different security contexts.

---

## 3. Identify the Service

SPNs contain information about the service and host.

Examples:

```text id="y65v4m"
HTTP/web01.example.local
MSSQLSvc/db01.example.local:1433
CIFS/files01.example.local
LDAP/dc01.example.local
```

Record:

```text id="1h8kyo"
Account
Service Type
Hostname
Port, if present
SPN
```

Example:

| Account   | Service | Host  | SPN                                |
| --------- | ------- | ----- | ---------------------------------- |
| `svc-web` | HTTP    | WEB01 | `HTTP/web01.example.local`         |
| `svc-sql` | MSSQL   | DB01  | `MSSQLSvc/db01.example.local:1433` |

---

## 4. Identify the Account Context

Once an SPN is identified, investigate the associated account.

```powershell id="v5x2sp"
Get-ADUser -Identity svc-sql -Properties `
    Enabled,MemberOf,PasswordLastSet,ServicePrincipalName,Description
```

Review:

* Account status
* Group membership
* Password age
* Description
* SPNs
* Delegation settings
* Administrative privileges

The important relationship is:

```text id="sh6zbb"
SPN
 │
 ▼
Service Account
 │
 ├── Groups
 ├── ACLs
 ├── Delegation
 └── Privileges
```

---

## 5. Check Password Management

Service-account password management is important because Kerberoasting depends on the ability to recover or guess the underlying password.

Inspect:

```powershell id="3j8pqq"
Get-ADUser -Identity svc-sql `
    -Properties PasswordLastSet,PasswordNeverExpires,Enabled
```

Potential indicators include:

```text id="y4x0pv"
Long password age
Password never expires
Manual password management
Shared service credentials
Weak organizational password practices
```

These indicators should be treated as assessment evidence rather than automatic proof of compromise.

---

## 6. Prioritize Service Accounts by Context

A useful prioritization model is based on account context.

### Lower-Impact Example

```text id="2hpkcq"
svc-test
   │
   └── Test application
```

### Higher-Impact Context

```text id="9z4wcb"
svc-backup
   │
   ├── BackupOperators
   ├── Multiple servers
   └── Sensitive resources
```

The second example deserves deeper investigation because the account has a broader privilege context.

This is not because of the SPN alone.

The significance comes from:

```text id="xy4zpj"
SPN
+
Account Privileges
+
Credential Strength
+
Accessible Resources
```

---

## 7. Obtain a Service Ticket

During an authorized assessment, a service ticket can be requested for an identified SPN.

Using native Windows tooling:

```cmd id="5a0qzq"
klist
```

Tickets can be requested through Kerberos-aware tooling appropriate to the assessment environment.

For example, from an authorized Windows domain context:

```cmd id="y6g1sr"
setspn -Q MSSQLSvc/db01.example.local:1433
```

The exact ticket-request mechanism depends on the assessment environment and tooling.

The important workflow is:

```text id="k2h7n5"
SPN Identified
      │
      ▼
Authorized Ticket Request
      │
      ▼
Service Ticket Obtained
      │
      ▼
Secure Evidence Handling
```

---

## 8. Understand the Offline Analysis Model

Kerberoasting is significant because password analysis can occur offline.

The conceptual flow is:

```text id="h4q3b7"
Service Ticket
      │
      ▼
Ticket Representation
      │
      ▼
Offline Password Guessing
      │
      ▼
Candidate Password
      │
      ▼
Credential Validation
```

The password-guessing stage does not require repeatedly authenticating to the domain.

This is why password strength and service-account password management matter.

---

## 9. Offline Password Analysis

If authorized by the engagement, the service-ticket material can be subjected to offline password analysis.

The assessment should record:

```text id="0js4x5"
Account
SPN
Ticket Obtained
Analysis Method
Wordlist / Rules
Result
Time Required
```

Do not publish recovered passwords.

Use:

```text id="k8v9hy"
Password:
<REDACTED>
```

in documentation.

---

## 10. Credential Validation

If a password is recovered, validate it carefully.

First determine:

```text id="crx8l1"
Which account?
      │
      ▼
Is the account enabled?
      │
      ▼
What authentication scope?
      │
      ▼
What privileges?
```

Then perform only authorized validation.

A recovered password does not automatically mean:

```text id="yr1v7q"
Domain Administrator
```

It may belong to:

```text id="b6k2q9"
Low-privilege service account
```

or:

```text id="5g6qpc"
Privileged service account
```

The account context determines the impact.

---

## 11. Correlate With Group Membership

Check the account's group memberships.

```powershell id="9ndh5r"
Get-ADPrincipalGroupMembership svc-sql |
    Select-Object Name
```

For deeper account information:

```powershell id="y0y0g4"
Get-ADUser svc-sql -Properties MemberOf,Description
```

Map the result:

```text id="f7t8f3"
Recovered Credential
       │
       ▼
svc-sql
       │
       ├── DatabaseAdmins
       ├── SQL service
       └── DB01
```

This establishes the potential security impact.

---

## 12. Correlate With ACLs

An account's privileges may come from ACLs rather than group membership.

For example:

```text id="0z8j9m"
svc-sql
   │
   ▼
ACL Permission
   │
   ▼
Computer / User / Group Object
   │
   ▼
Potential Privilege Path
```

Use the ACL information collected during Section 03.

This is especially important when an apparently ordinary service account has delegated control over sensitive AD objects.

---

## 13. Correlate With Delegation

Service accounts may also be associated with delegation configurations.

Check for:

```text id="c3z1rs"
TrustedForDelegation
TrustedToAuthForDelegation
msDS-AllowedToDelegateTo
```

This creates a combined relationship:

```text id="h5r4dc"
Service Account
      │
      ├── SPN
      ├── Credential
      └── Delegation
              │
              ▼
         Target Service
```

The combined evidence may reveal a more meaningful path than any single finding.

---

## 14. Correlate With Password Reuse

The previous workflow examined password reuse.

Kerberoasting can reveal another credential that should then be checked against known reuse patterns.

```text id="p8o1hk"
Kerberoasting
      │
      ▼
Recovered Credential
      │
      ▼
Account
      │
      ▼
Check Existing Credential Relationships
      │
      ▼
Potential Reuse
```

Do not assume that a recovered service-account password is reused elsewhere.

It must be independently established.

---

## 15. Identify Potential Attack Paths

A Kerberoasting finding becomes more significant when the associated account has meaningful privileges.

Example:

```text id="7zj8h4"
SPN
 │
 ▼
svc-backup
 │
 ├── Password recovered
 │
 ├── BackupOperators
 │
 └── Access to BACKUP01
          │
          ▼
     Sensitive Resources
```

Another example:

```text id="g4c5sa"
SPN
 │
 ▼
svc-admin
 │
 ▼
Privileged Group
 │
 ▼
Administrative Resource
```

The documented path should contain only relationships supported by evidence.

---

## 16. Managed Service Accounts

Modern Active Directory environments may use:

* Managed Service Accounts
* Group Managed Service Accounts (gMSA)

These accounts are designed to improve service-account password management.

When an SPN belongs to a managed service account, record the account type.

Example:

```text id="c7wh3e"
gMSA
 │
 ├── SPN
 ├── Service
 └── Managed Password
```

Do not automatically apply the same password-risk assumptions used for manually managed service accounts.

---

## 17. Computer Accounts With SPNs

Computer accounts commonly possess SPNs.

For example:

```text id="x9k4yp"
WEB01$
   │
   ├── HOST/web01
   ├── RestrictedKrbHost/web01
   └── Other system SPNs
```

The presence of an SPN on a computer account is normal in many environments.

Therefore:

```text id="u8v3h7"
Computer SPN
     ≠
Automatically Vulnerable
```

Focus on account type and context.

---

## 18. Duplicate SPNs

Duplicate SPNs can cause Kerberos authentication problems and may indicate configuration issues.

Search for a specific SPN:

```cmd id="o6m5e8"
setspn -Q HTTP/web01.example.local
```

For duplicate SPNs:

```cmd id="4k6s2x"
setspn -X
```

Record:

```text id="w2m3pm"
SPN
Associated Accounts
Affected Service
Observed Behavior
```

Duplicate SPNs are primarily a configuration issue and should not automatically be classified as a credential-compromise finding.

---

## 19. Evidence Collection

For every Kerberoasting assessment, record:

```text id="t0j4w6"
Account:
SPN:
Service:
Host:
Account Type:
Account Status:
Password Last Set:
Password Management:
Groups:
Delegation:
Ticket Obtained:
Offline Analysis:
Credential Recovered:
Validation:
Privileges:
Potential Impact:
```

Keep sensitive ticket material and recovered credentials out of public repositories.

---

## 20. Common False Positives

### Computer Account SPNs

Computer accounts commonly have SPNs.

### Managed Service Accounts

Managed accounts may have stronger automated password management.

### Strong Passwords

A service account may have an SPN while using a strong, regularly managed password.

### Low-Privilege Service Accounts

Credential recovery does not necessarily provide meaningful privilege escalation.

### Disabled Accounts

A recovered password for a disabled account does not provide current authentication.

### Stale Assessment Data

Account configuration may change during or after an assessment.

---

## Common Mistakes

### Mistake 1: Treating Every SPN as Vulnerable

SPNs are normal components of Kerberos service authentication.

### Mistake 2: Ignoring Account Privileges

The account behind the SPN determines the potential impact.

### Mistake 3: Confusing SPN With Username

An SPN identifies a service instance; it is not itself the account credential.

### Mistake 4: Assuming Password Recovery Equals Escalation

A recovered password must be mapped to actual privileges.

### Mistake 5: Testing Outside Scope

Do not use recovered credentials against systems outside the authorized scope.

### Mistake 6: Publishing Recovered Secrets

Never commit passwords, ticket material, or other sensitive authentication data to GitHub.

---

## Kerberoasting Checklist

### SPN Enumeration

* [ ] Enumerated user SPNs
* [ ] Enumerated computer SPNs
* [ ] Identified service types
* [ ] Identified service hosts
* [ ] Checked for duplicate SPNs

### Account Analysis

* [ ] Identified account type
* [ ] Checked account status
* [ ] Checked password age
* [ ] Checked password-management context
* [ ] Checked group membership
* [ ] Checked ACL permissions
* [ ] Checked delegation configuration

### Ticket Analysis

* [ ] Identified relevant SPN
* [ ] Obtained ticket within authorized scope
* [ ] Protected ticket material
* [ ] Performed authorized offline analysis
* [ ] Recorded analysis results

### Credential Validation

* [ ] Identified recovered credential owner
* [ ] Checked account status
* [ ] Validated credential within scope
* [ ] Determined authentication scope
* [ ] Determined authorization
* [ ] Mapped privileges

### Correlation

* [ ] Correlated with password reuse
* [ ] Correlated with group membership
* [ ] Correlated with ACLs
* [ ] Correlated with delegation
* [ ] Identified potential attack paths
* [ ] Distinguished configuration from confirmed vulnerability

### Documentation

* [ ] Recorded SPN
* [ ] Recorded account
* [ ] Recorded service and host
* [ ] Recorded evidence
* [ ] Sanitized sensitive credentials
* [ ] Documented limitations

---

## Transition to AS-REP Roasting

After analyzing service accounts and Kerberos service tickets, the workflow moves to accounts that may use Kerberos without pre-authentication.

```text id="k1r5v2"
Kerberoasting
      │
      ▼
Kerberos Service Accounts
      │
      ▼
Authentication Configuration
      │
      ▼
Accounts Without Pre-Authentication
      │
      ▼
AS-REP Roasting
```

The next file is:

**`04-Credentials-and-Authentication/AS-REP-Roasting.md`**
