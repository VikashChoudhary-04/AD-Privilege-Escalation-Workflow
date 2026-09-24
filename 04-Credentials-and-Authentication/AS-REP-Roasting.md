# AS-REP Roasting

## Purpose

AS-REP Roasting is a Kerberos-focused credential-access technique involving Active Directory accounts that do not require Kerberos pre-authentication.

The assessment objective is to:

* Identify accounts configured without Kerberos pre-authentication
* Understand the affected account context
* Determine whether the accounts are enabled and relevant
* Obtain authentication material within authorized scope
* Perform controlled offline password analysis
* Determine whether a credential can be recovered
* Map recovered credentials to privileges and access
* Identify potential privilege-escalation paths

The core workflow is:

```text id="j7x5mv"
Enumerate Accounts
       │
       ▼
Identify Pre-Authentication Configuration
       │
       ▼
Identify Eligible Accounts
       │
       ▼
Assess Account Context
       │
       ▼
Obtain AS-REP Material
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
Attack-Path Analysis
       │
       ▼
Document
```

AS-REP Roasting should only be performed against authorized Active Directory environments.

---

## Position in the Workflow

```text id="z9r2xa"
04-Credentials-and-Authentication
              │
              ├── Credential Discovery
              ├── Password Reuse
              ├── Kerberoasting
              │
              ├── AS-REP Roasting  ← Current
              │
              └── Credential Dumping
              │
              ▼
05-Privilege-Escalation-Paths
```

---

## Kerberos Pre-Authentication

Kerberos normally uses pre-authentication to help verify that a client knows the account's secret before the KDC issues an authentication response.

The simplified flow is:

```text id="7s0e1f"
Client
  │
  │ Authentication Request
  ▼
KDC
  │
  │ Pre-Authentication
  ▼
Authentication Response
```

Some Active Directory accounts can be configured so that Kerberos pre-authentication is not required.

This configuration is represented by:

```text id="j6e9tg"
DONT_REQUIRE_PREAUTH
```

Such accounts can become candidates for AS-REP Roasting.

---

## AS-REP Roasting Workflow

```text id="w8b3r7"
Start
 │
 ▼
Enumerate Domain Accounts
 │
 ▼
Check Pre-Authentication Setting
 │
 ▼
Identify Eligible Accounts
 │
 ▼
Check Account Status
 │
 ▼
Assess Privileges
 │
 ▼
Obtain AS-REP Response
 │
 ▼
Secure Authentication Material
 │
 ▼
Offline Password Analysis
 │
 ▼
Credential Recovery?
 │
 ├── No ──► Document Result
 │
 └── Yes
       │
       ▼
  Validate Credential
       │
       ▼
  Map Privileges
       │
       ▼
  Identify Potential Path
```

---

## 1. Identify Accounts Without Pre-Authentication

The first step is to enumerate accounts where Kerberos pre-authentication is not required.

### PowerShell

Using the Active Directory module:

```powershell id="3p2h4x"
Get-ADUser -Filter * -Properties DoesNotRequirePreAuth |
    Where-Object {$_.DoesNotRequirePreAuth -eq $true} |
    Select-Object SamAccountName, Enabled, DoesNotRequirePreAuth
```

This identifies user accounts with the relevant configuration.

The result should then be investigated rather than immediately classified as a vulnerability.

---

## 2. LDAP-Based Enumeration

The underlying account configuration can also be queried through LDAP.

A useful filter is:

```text id="x1q8ks"
(DONT_REQUIRE_PREAUTH)
```

For example, tools that support LDAP filtering can search for:

```text id="q4y8tw"
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))
```

The exact LDAP syntax and tooling can vary.

The important objective is to identify accounts where the relevant `userAccountControl` bit is set.

---

## 3. Understand `userAccountControl`

Active Directory stores account configuration flags in:

```text id="e6x8q2"
userAccountControl
```

One of those flags represents:

```text id="s2z5mj"
DONT_REQUIRE_PREAUTH
```

Its decimal value is:

```text id="q7b5w4"
4194304
```

Therefore, a bitwise LDAP query can identify accounts with this setting.

Do not rely exclusively on the numeric value in documentation.

Where possible, confirm the resulting account property using a directory-aware tool.

---

## 4. Identify Account Status

An account without Kerberos pre-authentication may still be:

* Disabled
* Expired
* Inactive
* Restricted
* Non-production
* A test account

Check the account status.

```powershell id="q8j4x2"
Get-ADUser -Identity username `
    -Properties Enabled,AccountExpirationDate,PasswordLastSet
```

Record:

```text id="l5m7h2"
Account
Enabled
Expiration
Password Last Set
Description
Groups
```

The configuration is more meaningful when the account is active and used in the environment.

---

## 5. Identify Account Type

Determine what type of account is affected.

Examples include:

```text id="z4y6h0"
Normal User
Service Account
Administrative Account
Application Account
Test Account
```

Example:

```text id="8s3d2v"
Account:
svc-app

Type:
Service Account

Pre-Authentication:
Not required
```

The account type affects the subsequent impact analysis.

---

## 6. Analyze Account Privileges

The presence of `DONT_REQUIRE_PREAUTH` does not determine the final security impact.

Investigate:

* Group membership
* ACL permissions
* Delegation
* SPNs
* Administrative roles
* Access to sensitive systems

For example:

```powershell id="g9q3r4"
Get-ADPrincipalGroupMembership svc-app |
    Select-Object Name
```

Then build the relationship:

```text id="0a7v6q"
AS-REP Eligible Account
        │
        ▼
Account
        │
        ├── Groups
        ├── ACLs
        ├── SPNs
        └── Delegation
```

---

## 7. Obtain AS-REP Material

For an account that does not require pre-authentication, an AS-REP response can be requested without first completing the normal pre-authentication exchange.

During an authorized assessment, Kerberos-aware tools can request the response and save the relevant material for offline analysis.

The conceptual process is:

```text id="3e9j5x"
Eligible Account
      │
      ▼
Authentication Request
      │
      ▼
KDC
      │
      ▼
AS-REP Response
      │
      ▼
Captured Authentication Material
```

The request should be performed only against accounts and domains included in the engagement scope.

---

## 8. Offline Password Analysis

The key property of AS-REP Roasting is that the resulting authentication material can be subjected to offline password analysis.

Conceptually:

```text id="q7f4c2"
AS-REP Material
      │
      ▼
Offline Password Analysis
      │
      ▼
Candidate Password
      │
      ▼
Credential Validation
```

Unlike online password guessing, the analysis does not require repeatedly authenticating against the domain.

Therefore, password strength becomes an important factor.

Record:

```text id="k4z9g2"
Account
AS-REP Material Obtained
Analysis Method
Wordlist / Rules
Result
Time Required
```

Do not store recovered passwords in public documentation.

---

## 9. Credential Recovery

There are several possible outcomes.

### No Password Recovered

```text id="v8x4c3"
AS-REP Material
      │
      ▼
Offline Analysis
      │
      ▼
No Credential Recovered
```

Document the result accurately.

### Password Recovered

```text id="h5w2d6"
AS-REP Material
      │
      ▼
Offline Analysis
      │
      ▼
Credential Recovered
      │
      ▼
Controlled Validation
```

Password recovery should then be treated as a credential finding requiring account and privilege analysis.

---

## 10. Validate the Recovered Credential

If a password is recovered, determine:

```text id="e2n7m4"
Which account?
      │
      ▼
Is it enabled?
      │
      ▼
Where can it authenticate?
      │
      ▼
What privileges does it have?
```

Example:

```text id="p7r8d1"
Recovered Credential
       │
       ▼
svc-app
       │
       ├── Enabled
       ├── WebAdmins
       └── WEB01 access
```

Authentication should be validated only where permitted.

---

## 11. Authentication vs Authorization

A successful authentication does not automatically mean privileged access.

Separate:

```text id="r9j3x5"
Credential
    │
    ▼
Authentication
    │
    ▼
Identity
    │
    ▼
Authorization
    │
    ▼
Permissions
```

For example:

```text id="s2h6v8"
Authentication:
Successful

Authorization:
Application access only
```

This distinction should appear in the final assessment evidence.

---

## 12. Correlate With Group Membership

Group membership can reveal the actual impact of a recovered account.

```powershell id="j3k5m1"
Get-ADPrincipalGroupMembership username |
    Select-Object Name
```

For example:

```text id="m7d2p8"
Recovered Credential
       │
       ▼
svc-backup
       │
       ▼
BackupOperators
```

This may provide access to additional systems or resources.

The actual permissions of the group should be established before reporting impact.

---

## 13. Correlate With ACLs

The account may have significant rights through delegated permissions.

Conceptually:

```text id="c6q8s0"
Recovered Credential
       │
       ▼
Account
       │
       ▼
ACL
       │
       ▼
AD Object
       │
       ▼
Potential Privilege Path
```

Relevant objects may include:

* Users
* Groups
* Computers
* GPOs
* Service accounts
* Delegation-related objects

This connects AS-REP Roasting with the ACL enumeration completed earlier.

---

## 14. Correlate With SPNs

An account without pre-authentication may also have service-related configuration.

Check:

```powershell id="n8s2d7"
Get-ADUser username -Properties ServicePrincipalName |
    Select-Object SamAccountName, ServicePrincipalName
```

This can reveal relationships such as:

```text id="f4q8r1"
Account
 │
 ├── No Kerberos Pre-Authentication
 │
 └── SPN
       │
       ▼
     Service
```

Multiple authentication weaknesses affecting the same account should be documented together.

---

## 15. Correlate With Delegation

Check whether the affected account is involved in delegation.

Relevant properties include:

```text id="m3t7q6"
TrustedForDelegation
TrustedToAuthForDelegation
msDS-AllowedToDelegateTo
```

The combined relationship may look like:

```text id="g8v2j4"
AS-REP Eligible Account
       │
       ├── Delegation
       │
       ├── SPN
       │
       └── Privileged Group
```

This provides more useful context than reporting the pre-authentication setting alone.

---

## 16. Correlate With Password Reuse

If an account's password is recovered, investigate whether the credential is reused elsewhere.

```text id="p5m8z1"
AS-REP Roasting
       │
       ▼
Recovered Password
       │
       ▼
Account
       │
       ▼
Password-Reuse Analysis
       │
       ▼
Additional Access?
```

Do not assume reuse.

It must be independently established.

---

## 17. Identify Potential Attack Paths

The final goal is to determine whether the account creates a meaningful path toward privileged resources.

Example:

```text id="v4q9s6"
No Pre-Authentication
        │
        ▼
svc-backup
        │
        ├── Credential Recovered
        │
        ├── BackupOperators
        │
        └── BACKUP01 Access
                 │
                 ▼
          Sensitive Resources
```

Another example:

```text id="k6h3p2"
No Pre-Authentication
        │
        ▼
Privileged Service Account
        │
        ▼
Administrative Group
        │
        ▼
Privileged Resource
```

The path must be supported by actual enumeration and validation evidence.

---

## 18. Account Password Management

Password management is an important factor.

Investigate:

```powershell id="q5d8w9"
Get-ADUser username `
    -Properties PasswordLastSet,PasswordNeverExpires,Enabled
```

Potential risk indicators include:

```text id="g1r5m8"
Long password age
Password never expires
Manual password management
Shared service-account credentials
Weak password policy for the affected account
```

These indicators should be reported as configuration evidence, not automatically as proof that the account can be compromised.

---

## 19. Managed Service Accounts

If an eligible account is a managed service account or gMSA, record that context.

Managed service accounts are designed to improve service-account password management.

Therefore:

```text id="z2x7v3"
No Pre-Authentication
       +
Managed Account
```

should not be interpreted using exactly the same assumptions as:

```text id="a7m4q9"
No Pre-Authentication
       +
Manually Managed User Account
```

Account type matters.

---

## 20. Evidence Collection

For each AS-REP Roasting assessment, record:

```text id="n6j8r3"
Account:
Account Type:
Enabled:
Pre-Authentication:
Password Last Set:
Groups:
SPNs:
Delegation:
AS-REP Material Obtained:
Offline Analysis:
Credential Recovered:
Credential Validation:
Observed Privileges:
Potential Impact:
```

Never store recovered passwords in the repository.

Use:

```text id="v3q5m9"
PASSWORD=<REDACTED>
```

for sanitized documentation.

---

## Common False Positives

### Disabled Account

An account may have the relevant configuration but be disabled.

### Test Account

A development or laboratory account may intentionally use different authentication settings.

### Low-Privilege Account

Credential recovery does not necessarily result in privilege escalation.

### Strong Password

An eligible account may still have a password that resists the authorized offline analysis.

### Stale Configuration

The account may have been configured historically but no longer be used.

---

## Common Mistakes

### Mistake 1: Treating Configuration as Compromise

```text id="j4s8p1"
DONT_REQUIRE_PREAUTH
        ≠
Password Recovered
```

### Mistake 2: Ignoring Account Privileges

The account context determines the potential impact.

### Mistake 3: Confusing Authentication With Authorization

Successful authentication does not establish administrative access.

### Mistake 4: Testing Outside Scope

Only authorized accounts and systems should be tested.

### Mistake 5: Ignoring Password Management

Password age and management practices provide important context.

### Mistake 6: Publishing Credentials

Never commit recovered passwords, hashes, or authentication material to GitHub.

---

## AS-REP Roasting Checklist

### Enumeration

* [ ] Enumerated domain users
* [ ] Identified accounts without Kerberos pre-authentication
* [ ] Confirmed `DONT_REQUIRE_PREAUTH`
* [ ] Identified account type
* [ ] Checked account status

### Account Analysis

* [ ] Checked groups
* [ ] Checked ACLs
* [ ] Checked SPNs
* [ ] Checked delegation
* [ ] Checked password age
* [ ] Checked password-management context

### AS-REP Analysis

* [ ] Obtained AS-REP material within authorized scope
* [ ] Protected authentication material
* [ ] Performed authorized offline analysis
* [ ] Recorded analysis result
* [ ] Documented whether a credential was recovered

### Validation

* [ ] Identified recovered credential owner
* [ ] Checked account status
* [ ] Validated credential within scope
* [ ] Determined authentication scope
* [ ] Determined authorization
* [ ] Mapped privileges

### Correlation

* [ ] Checked password reuse
* [ ] Checked group membership
* [ ] Checked ACLs
* [ ] Checked SPNs
* [ ] Checked delegation
* [ ] Identified potential attack paths

### Documentation

* [ ] Recorded account configuration
* [ ] Recorded evidence
* [ ] Documented validation
* [ ] Distinguished configuration from confirmed impact
* [ ] Sanitized credentials
* [ ] Protected sensitive material

---

## Transition to Credential Dumping

After analyzing credentials exposed through configuration and Kerberos authentication weaknesses, the workflow moves to authorized recovery of credential material from systems where access has already been obtained.

```text id="q8m4v7"
AS-REP Roasting
      │
      ▼
Authentication Weaknesses
      │
      ▼
Credential Discovery
      │
      ▼
Authorized System Access
      │
      ▼
Credential Stores
      │
      ▼
Credential Dumping
```

The next file is:

**`04-Credentials-and-Authentication/Credential-Dumping.md`**
