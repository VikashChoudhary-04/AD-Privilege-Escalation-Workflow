# Credential Dumping

## Purpose

Credential dumping is the authorized recovery and analysis of authentication material from systems where the assessor already has appropriate access.

The objective is to determine:

* What credential material is present
* Where it is stored
* Which account it belongs to
* Whether it can be recovered safely
* Whether the recovered material is current
* What privileges the associated account has
* Whether the credential creates an additional privilege-escalation path

Credential dumping should be treated as a controlled assessment activity.

The workflow is:

```text id="c8m4q2"
Authorized Access
       │
       ▼
Identify Credential Stores
       │
       ▼
Select Appropriate Collection Method
       │
       ▼
Acquire Credential Material
       │
       ▼
Identify Associated Account
       │
       ▼
Validate Credential Context
       │
       ▼
Map Privileges
       │
       ▼
Identify Potential Path
       │
       ▼
Document Evidence
```

Only systems and credential stores included in the authorized scope should be examined.

---

## Position in the Workflow

```text id="m7f3z9"
04-Credentials-and-Authentication
              │
              ├── Credential Discovery
              ├── Password Reuse
              ├── Kerberoasting
              ├── AS-REP Roasting
              │
              └── Credential Dumping  ← Current
              │
              ▼
05-Privilege-Escalation-Paths
```

Credential dumping is the final technique in Section 04.

---

## What Is Credential Dumping?

Credential dumping involves recovering authentication material from a system or credential store.

Depending on the operating system and configuration, this may include:

* Password hashes
* Cached credentials
* Kerberos tickets
* Credential Manager entries
* Application secrets
* Service credentials
* Authentication tokens
* Private keys
* Other security-sensitive authentication material

The important relationship is:

```text id="w4n7q1"
System Access
     │
     ▼
Credential Store
     │
     ▼
Authentication Material
     │
     ▼
Account
     │
     ▼
Privileges
```

Finding credential material does not automatically establish a privilege-escalation vulnerability.

---

## Credential Dumping Workflow

```text id="y2r5v8"
Confirm Authorized Access
        │
        ▼
Identify Operating System
        │
        ▼
Identify Current Privileges
        │
        ▼
Identify Relevant Credential Stores
        │
        ▼
Select Minimal Collection Method
        │
        ▼
Acquire Credential Material
        │
        ▼
Protect Evidence
        │
        ▼
Identify Associated Accounts
        │
        ▼
Validate Account Context
        │
        ▼
Map Privileges
        │
        ▼
Correlate With AD Enumeration
        │
        ▼
Identify Potential Attack Path
        │
        ▼
Document
```

---

## 1. Confirm Authorized Access

Before collecting credential material, establish:

```text id="q6k4w1"
Target System
Assessment Scope
Current User
Current Privileges
Approved Collection Method
Evidence Requirements
```

Example:

```powershell id="r8p5x2"
whoami
whoami /all
hostname
```

The objective is to verify that the current session has the permissions required for the approved collection method.

---

## 2. Identify the Operating System

Credential storage differs between operating systems.

### Windows

```powershell id="z4c8m3"
Get-ComputerInfo |
    Select-Object WindowsProductName, WindowsVersion
```

### Linux

```bash id="p7m3k5"
uname -a
cat /etc/os-release
```

Understanding the operating system determines which credential stores are relevant.

---

## 3. Identify Current Privileges

On Windows:

```cmd id="h3v7n2"
whoami /priv
```

and:

```cmd id="x6q2m9"
whoami /groups
```

On Linux:

```bash id="a9k4r6"
id
```

Record:

```text id="f5w8j3"
User
Groups
Privileges
Security Context
```

Do not assume that administrator-level access on one system provides equivalent access throughout the domain.

---

## 4. Windows Credential Stores

Windows systems may contain authentication material in several locations.

Potential sources include:

```text id="n4p7c2"
LSA-related secrets
SAM database
Credential Manager
Windows cached domain credentials
Kerberos tickets
Application credential stores
Browser credential stores
Service configurations
User profile data
```

Each source represents a different type of authentication material.

---

## 5. Linux Credential Sources

Linux systems can contain authentication material in:

```text id="m2q8v5"
/etc/passwd
/etc/shadow
SSH keys
SSH configuration
Shell history
Application configuration
Credential caches
Cloud CLI configuration
Environment variables
Service configuration
```

For example:

```bash id="t6j3w9"
ls -la ~/.ssh/
```

and:

```bash id="g8r2k5"
ls -la ~/
```

Access to a file does not necessarily mean that its contents contain usable credentials.

---

## 6. Credential Material Types

Classify recovered material before assessing it.

| Type                     | Example                  | Meaning                          |
| ------------------------ | ------------------------ | -------------------------------- |
| Password Hash            | NTLM hash                | Password representation          |
| Kerberos Ticket          | TGT / service ticket     | Kerberos authentication material |
| Cached Credential        | Domain logon cache       | Cached authentication data       |
| LSA Secret               | Service-related secret   | Protected Windows secret         |
| Credential Manager Entry | Stored target credential | Application/system credential    |
| SSH Private Key          | Private key              | Key-based authentication         |
| Access Token             | Application token        | Application/API authentication   |
| Application Secret       | API/database secret      | Application authentication       |

Correct classification is necessary before determining impact.

---

## 7. Windows SAM

The Windows Security Account Manager contains information associated with local accounts.

Conceptually:

```text id="s3y7q8"
Local Account
     │
     ▼
SAM
     │
     ▼
Credential Material
```

The SAM should only be accessed through authorized credential-assessment procedures.

The assessment goal is to determine:

* Which local accounts exist
* Whether credential material can be recovered
* Whether accounts are reused across systems
* Whether recovered credentials provide meaningful access

Do not expose recovered hashes in public documentation.

---

## 8. LSA-Related Secrets

Windows Local Security Authority components may protect secrets associated with:

* Service accounts
* Cached authentication information
* Application credentials
* Other system secrets

The conceptual workflow is:

```text id="p4k8r2"
Authorized Privileged Access
          │
          ▼
Protected Credential Store
          │
          ▼
Secret Material
          │
          ▼
Associated Account
```

The exact available material depends on:

* Windows version
* Security configuration
* Running services
* Credential protection mechanisms
* Current privileges

Therefore, absence of a recovered secret does not necessarily mean that no credentials exist.

---

## 9. Windows Credential Manager

Credential Manager may contain stored credentials for applications and services.

A basic enumeration command is:

```cmd id="y8m3q7"
cmdkey /list
```

The output can identify stored credential targets.

However:

```text id="v4x6p2"
Stored Target
     ≠
Recovered Password
```

Record what is actually observable.

---

## 10. Kerberos Tickets

Kerberos tickets are authentication material rather than plaintext passwords.

Current tickets can be viewed with:

```cmd id="r5k2m8"
klist
```

Example conceptual structure:

```text id="j7p4c9"
Account
   │
   ▼
Kerberos Ticket
   │
   ├── Service
   ├── Realm
   └── Validity
```

When tickets are found, record:

* Account
* Ticket type
* Service
* Start time
* End time
* Renewal information

Treat ticket material as sensitive evidence.

---

## 11. Cached Domain Credentials

Windows may maintain cached domain authentication information to support certain logon scenarios.

The important distinction is:

```text id="q9w3k7"
Cached Credential
      ≠
Current Plaintext Password
```

Cached material may require offline analysis and may not provide direct authentication.

Document:

```text id="z4m6r8"
Associated Account
Credential Type
Collection Method
Analysis Result
Validation Result
```

---

## 12. Service Credentials

Services can run under domain or local accounts.

Enumerate services:

```powershell id="m5x8c1"
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State, PathName
```

The relationship is:

```text id="d7q3v9"
Service
   │
   ▼
Service Account
   │
   ▼
Credential Material
```

A recovered service credential should then be correlated with:

* AD account
* Group membership
* SPNs
* Delegation
* ACLs
* Accessible systems

---

## 13. Scheduled Task Credentials

Scheduled tasks may execute under privileged accounts.

Enumerate:

```powershell id="x8p4m6"
Get-ScheduledTask |
    Select-Object TaskName, TaskPath, State
```

Then inspect the execution principal:

```powershell id="n3q7w5"
Get-ScheduledTask |
    ForEach-Object {
        [PSCustomObject]@{
            TaskName = $_.TaskName
            UserId   = $_.Principal.UserId
            RunLevel = $_.Principal.RunLevel
        }
    }
```

A useful relationship is:

```text id="f2c8r5"
Scheduled Task
      │
      ▼
Execution Account
      │
      ▼
Privileges
```

If credentials are exposed through task configuration or referenced files, investigate them within scope.

---

## 14. Application Credential Stores

Applications may maintain their own credential stores.

Potential examples include:

```text id="b6w4p8"
Web applications
Database clients
Development tools
Cloud CLI tools
Backup software
Monitoring software
Deployment systems
```

The assessment workflow is:

```text id="c5n7r2"
Application
     │
     ▼
Credential Store
     │
     ▼
Credential
     │
     ▼
Associated Identity
     │
     ▼
Application / System Access
```

Always determine whether the credential is:

* Current
* Encrypted
* Expired
* Application-specific
* Domain-based

---

## 15. SSH Keys

On Linux and other Unix-like systems, SSH private keys may provide authentication.

Inspect authorized locations:

```bash id="p3x7m9"
ls -la ~/.ssh/
```

Potential files include:

```text id="v8q2k5"
id_rsa
id_ed25519
id_ecdsa
authorized_keys
config
known_hosts
```

The presence of a private key does not automatically establish usable access.

Consider:

```text id="z6m4r1"
Private Key
    │
    ▼
Passphrase?
    │
    ▼
Authorized Key
    │
    ▼
Target Account
    │
    ▼
Target System
```

Only validate against authorized targets.

---

## 16. Environment Variables

Applications may expose authentication material through environment variables.

Windows:

```powershell id="q2m7v4"
Get-ChildItem Env:
```

Linux:

```bash id="h5x8c3"
env
```

Search selectively for likely credential-related names:

```text id="j9p4w6"
PASSWORD
PASS
TOKEN
SECRET
API_KEY
ACCESS_KEY
CREDENTIAL
```

Do not assume that every variable with a security-related name contains a valid credential.

---

## 17. Command History

Command history can reveal authentication material that administrators previously entered.

PowerShell:

```powershell id="a6k3p9"
Get-Content (Get-PSReadLineOption).HistorySavePath `
    -ErrorAction SilentlyContinue
```

Linux:

```bash id="w5q7m2"
cat ~/.bash_history 2>/dev/null
```

Look for:

```text id="c9r4x7"
Remote authentication
Database commands
Cloud authentication
Administrative commands
Credential parameters
Tokens
```

Treat history as sensitive data.

---

## 18. Browser and Application Credentials

Browsers and applications may maintain local credential databases.

Potential material includes:

* Saved login information
* Session tokens
* Cookies
* Application tokens
* Credential databases

These should only be collected where explicitly permitted by the engagement.

The assessment should distinguish:

```text id="v2n8p5"
Credential Material
      │
      ├── Password
      ├── Token
      ├── Cookie
      └── Session Material
```

Each type has a different security meaning.

---

## 19. Identify the Credential Owner

After recovering credential material, establish the associated identity.

```text id="m7x3q8"
Credential Material
        │
        ▼
Account
        │
        ├── User
        ├── Service Account
        ├── Computer Account
        └── Local Account
```

For AD users:

```powershell id="c8v4n6"
Get-ADUser username -Properties *
```

For computers:

```powershell id="p2r7m5"
Get-ADComputer COMPUTER$ -Properties *
```

The account identity is essential for privilege analysis.

---

## 20. Validate Credential Status

A recovered credential may be:

* Valid
* Invalid
* Expired
* Disabled
* Rotated
* Restricted

For AD users:

```powershell id="k4m8x2"
Get-ADUser username `
    -Properties Enabled,PasswordLastSet,AccountExpirationDate
```

Do not repeatedly authenticate just to establish validity.

Use a controlled validation procedure.

---

## 21. Map Privileges

The recovered credential should then be mapped to privileges.

```text id="z7q3v5"
Credential
    │
    ▼
Account
    │
    ├── Groups
    ├── ACLs
    ├── SPNs
    ├── Delegation
    └── Accessible Systems
```

For example:

```powershell id="r6p2w9"
Get-ADPrincipalGroupMembership svc-backup |
    Select-Object Name
```

Then correlate with:

* Group privileges
* Computer permissions
* GPO permissions
* Service access
* Administrative roles

---

## 22. Correlate With Password Reuse

A recovered credential should be checked against known credential relationships.

```text id="x3m7k4"
Credential Dumping
       │
       ▼
Recovered Credential
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

Do not assume that the credential is reused elsewhere.

Establish the relationship through evidence.

---

## 23. Correlate With Kerberos

If the recovered account has SPNs, connect the finding to the Kerberos analysis.

```text id="m8q4v2"
Recovered Credential
       │
       ▼
Service Account
       │
       ├── SPN
       └── Kerberos Service
```

This can reveal relationships between:

* Credential dumping
* Service accounts
* Kerberoasting
* Delegation

---

## 24. Correlate With Delegation

Check whether the account is involved in delegation.

Relevant properties include:

```text id="n5r8c4"
TrustedForDelegation
TrustedToAuthForDelegation
msDS-AllowedToDelegateTo
msDS-AllowedToActOnBehalfOfOtherIdentity
```

The combined relationship may look like:

```text id="b7x3m9"
Recovered Credential
       │
       ▼
Service Account
       │
       ├── Delegation
       ├── SPN
       └── Privileged Group
```

This may create a meaningful path that would not be visible from credential dumping alone.

---

## 25. Identify Potential Attack Paths

The final objective is to determine whether recovered credential material creates additional access.

Example:

```text id="h4p7w2"
Credential Material
       │
       ▼
svc-backup
       │
       ├── BackupOperators
       │
       └── BACKUP01
               │
               ▼
        Sensitive Resources
```

Another example:

```text id="q8m5x3"
Recovered Credential
       │
       ▼
Privileged Service Account
       │
       ▼
Administrative Group
       │
       ▼
High-Value Computer
```

The actual path must be supported by evidence.

---

## 26. Minimize Credential Exposure

Credential dumping creates sensitive evidence.

Use these principles:

```text id="r3k6v8"
Collect Only What Is Required
        │
        ▼
Store Securely
        │
        ▼
Restrict Access
        │
        ▼
Sanitize Evidence
        │
        ▼
Delete Sensitive Copies When Required
```

Do not place the following in a public repository:

```text id="w6n2q9"
Plaintext Passwords
Password Hashes
Private Keys
Session Cookies
Authentication Tokens
Kerberos Ticket Material
API Secrets
```

For documentation:

```text id="c4m8r2"
PASSWORD=<REDACTED>
HASH=<REDACTED>
TOKEN=<REDACTED>
PRIVATE_KEY=<REDACTED>
```

---

## 27. Evidence Collection

For each credential-dumping activity, record:

```text id="p9x3m6"
Target System:
Operating System:
Current User:
Privileges:
Credential Store:
Collection Method:
Credential Type:
Associated Account:
Account Status:
Authentication Scope:
Groups:
Privileges:
Validation:
Evidence:
Potential Impact:
```

Example:

```text id="v5r7q2"
Target:
WORKSTATION01

Credential Type:
Local account hash

Associated Account:
local-admin

Validation:
Credential material recovered during authorized assessment

Observed Context:
Local administrative account

Potential Impact:
Requires correlation with password reuse across assessed systems
```

Do not include the actual hash in the public repository.

---

## 28. Common False Positives

### Stored Credential Target

A credential target does not necessarily expose a usable password.

### Cached Credential

Cached authentication material is not equivalent to a plaintext password.

### Kerberos Ticket

A ticket is authentication material, not necessarily a recoverable password.

### Private Key

A private key may be protected by a passphrase or have no authorized target.

### Disabled Account

Credential material associated with a disabled account may not provide current access.

### Stale Credential

Recovered material may have already been rotated.

---

## Common Mistakes

### Mistake 1: Dumping Everything

Collect only the credential material necessary for the assessment objective.

### Mistake 2: Ignoring Authorization

Credential recovery must remain within the approved scope.

### Mistake 3: Treating Every Hash as a Password

A hash is credential material, not plaintext authentication.

### Mistake 4: Assuming Credential Recovery Equals Privilege Escalation

Always map the credential to an account and its actual privileges.

### Mistake 5: Ignoring Credential Rotation

Recovered material may already be stale.

### Mistake 6: Publishing Sensitive Material

Never commit real credentials, hashes, tokens, or keys to GitHub.

### Mistake 7: Ignoring Detection

Credential dumping can generate significant security telemetry. Assessment activities should account for the engagement's detection and evidence requirements.

---

## Credential Dumping Checklist

### Authorization

* [ ] Confirmed target is in scope
* [ ] Confirmed current access
* [ ] Confirmed required privileges
* [ ] Confirmed approved collection method

### Credential Stores

* [ ] Reviewed relevant Windows credential stores
* [ ] Reviewed relevant Linux credential stores
* [ ] Reviewed service credentials
* [ ] Reviewed scheduled-task context
* [ ] Reviewed application credential stores
* [ ] Reviewed relevant Kerberos tickets
* [ ] Reviewed SSH keys where authorized

### Credential Analysis

* [ ] Identified credential type
* [ ] Identified associated account
* [ ] Checked account status
* [ ] Determined whether credential is current
* [ ] Protected recovered material

### Correlation

* [ ] Checked group membership
* [ ] Checked ACLs
* [ ] Checked SPNs
* [ ] Checked delegation
* [ ] Checked password reuse
* [ ] Checked accessible systems
* [ ] Identified potential attack paths

### Validation

* [ ] Validated only within authorized scope
* [ ] Distinguished authentication from authorization
* [ ] Recorded evidence
* [ ] Documented limitations
* [ ] Avoided unnecessary authentication attempts

### Evidence Protection

* [ ] Sanitized credentials
* [ ] Protected hashes
* [ ] Protected tickets
* [ ] Protected private keys
* [ ] Protected tokens
* [ ] Removed sensitive material from public documentation

---

## Section 04 Completion

Credential dumping completes the primary **Credentials & Authentication** workflow.

The complete section now follows:

```text id="s8m4q7"
Credential Discovery
        │
        ▼
Password Reuse
        │
        ▼
Kerberoasting
        │
        ▼
AS-REP Roasting
        │
        ▼
Credential Dumping
        │
        ▼
Credential / Account Mapping
        │
        ▼
Privilege Relationships
```

The combined information from Sections 03 and 04 should now provide:

```text id="x5q8m2"
Users
Groups
Computers
ACLs
GPOs
SPNs
Trusts
Delegation
        │
        +
Credentials
Authentication Weaknesses
Service Accounts
Credential Material
        │
        ▼
Potential Privilege Paths
```

The next workflow stage is:

**`05-Privilege-Escalation-Paths/README.md`**

This section will turn the relationships discovered so far into concrete privilege-escalation paths, covering:

* Group Membership Abuse
* ACL Abuse
* GPO Abuse
* Delegation Abuse
* RBCD
* ADCS
* Trust Abuse
