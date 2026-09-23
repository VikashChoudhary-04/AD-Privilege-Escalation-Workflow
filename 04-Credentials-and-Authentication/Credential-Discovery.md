# Credential Discovery

## Purpose

Credential discovery focuses on identifying authentication material exposed through systems, applications, configuration files, scripts, services, and other authorized sources.

The objective is not simply to find passwords.

The objective is to determine:

* Where authentication material is stored
* What type of credential was discovered
* Which account it belongs to
* Whether the credential is still valid
* Where the account can authenticate
* What privileges the account has
* Whether the credential creates a potential privilege-escalation path

Use the following mindset:

```text
Discover
   │
   ▼
Identify
   │
   ▼
Validate
   │
   ▼
Correlate
   │
   ▼
Document
```

Credential discovery should be performed only against systems and data included in the authorized assessment scope.

---

## Position in the Workflow

```text
03-Enumeration
      │
      ▼
04-Credentials-and-Authentication
      │
      ├── Credential Discovery  ← Current
      ├── Password Reuse
      ├── Kerberoasting
      ├── AS-REP Roasting
      └── Credential Dumping
      │
      ▼
05-Privilege-Escalation-Paths
```

Enumeration tells us **what exists**.

Credential discovery investigates **what authentication material may provide access to those resources**.

---

## Credential Discovery Workflow

Use this workflow when investigating a system:

```text
Identify System
      │
      ▼
Identify User Context
      │
      ▼
Review Common Credential Sources
      │
      ├── Files
      ├── Scripts
      ├── Services
      ├── Scheduled Tasks
      ├── Environment Variables
      ├── Registry
      ├── Application Configuration
      ├── Credential Stores
      └── Backups
      │
      ▼
Identify Credential Material
      │
      ▼
Identify Associated Account
      │
      ▼
Determine Credential Type
      │
      ▼
Validate Carefully
      │
      ▼
Map Account Privileges
      │
      ▼
Identify Potential Access Path
      │
      ▼
Document Evidence
```

---

## 1. Establish the Current Context

Before searching for credentials, establish the current system and user context.

### Windows

```powershell
whoami
whoami /all
hostname
```

Useful environment information:

```powershell
$env:USERNAME
$env:USERDOMAIN
$env:COMPUTERNAME
$env:USERPROFILE
```

### Linux

```bash
whoami
id
hostname
pwd
```

Environment information:

```bash
echo "$USER"
echo "$HOME"
echo "$HOSTNAME"
```

The purpose is to understand:

```text
Current User
      │
      ▼
Current System
      │
      ▼
Available Permissions
      │
      ▼
Credential Sources Accessible
```

Do not assume that a credential source exists simply because it is common on another system.

---

## 2. Search Configuration Files

Configuration files are one of the most common places where applications accidentally expose authentication material.

Look for:

* Database connection strings
* API credentials
* Service account usernames
* Password fields
* Connection parameters
* Authentication tokens
* Application secrets
* Backup credentials

Common Windows locations include:

```text
C:\inetpub\
C:\ProgramData\
C:\Users\
C:\Windows\Temp\
Application installation directories
```

Linux systems may contain application configuration under locations such as:

```text
/etc/
/opt/
/srv/
/var/www/
/home/
```

The exact locations depend on the application and operating system.

---

## 3. Search for Credential-Related Strings

When performing an authorized assessment, targeted searches can help locate suspicious configuration entries.

### Windows PowerShell

For example:

```powershell
Get-ChildItem -Path C:\ -File -Recurse -ErrorAction SilentlyContinue |
    Select-String -Pattern 'password|passwd|pwd|credential|secret|token' `
    -ErrorAction SilentlyContinue
```

A narrower search is generally preferable to scanning an entire filesystem unnecessarily.

Example:

```powershell
Get-ChildItem "C:\inetpub\wwwroot" -File -Recurse -ErrorAction SilentlyContinue |
    Select-String -Pattern 'password|connectionstring|secret' `
    -ErrorAction SilentlyContinue
```

### Linux

```bash
grep -RniE 'password|passwd|pwd|credential|secret|token' \
    /etc /opt /srv /var/www 2>/dev/null
```

Use targeted directories where possible.

The goal is to reduce:

* Noise
* Processing overhead
* Accidental access to unrelated data
* Unnecessary exposure of sensitive information

---

## 4. Inspect Scripts

Scripts frequently contain authentication material because administrators use them to automate tasks.

Potential locations include:

```text
.ps1
.bat
.cmd
.vbs
.py
.sh
```

Search for patterns such as:

```text
password
username
credential
ConvertTo-SecureString
net use
runas
Invoke-Command
connection strings
database authentication
```

Example:

```powershell
Get-ChildItem C:\Scripts -File -Recurse -ErrorAction SilentlyContinue |
    Select-String -Pattern 'password|credential|username|ConvertTo-SecureString'
```

A script containing a username does not necessarily contain a password.

Record the actual evidence rather than assuming what a script does.

---

## 5. Inspect Service Configurations

Windows services may run under dedicated service accounts.

First enumerate services:

```powershell
Get-CimInstance Win32_Service |
    Select-Object Name, StartName, State, PathName
```

Pay particular attention to:

```text
Name
StartName
PathName
```

The `StartName` field can identify the account used to run a service.

Example:

```text
Service:
    WebApp

StartName:
    EXAMPLE\svc-web

PathName:
    C:\Program Files\WebApp\app.exe
```

This establishes an account relationship:

```text
Service
   │
   ▼
Service Account
   │
   ▼
Account Privileges
```

The service account should then be correlated with AD enumeration.

---

## 6. Inspect Scheduled Tasks

Scheduled tasks can execute commands under privileged accounts.

Enumerate tasks:

```powershell
Get-ScheduledTask |
    Select-Object TaskName, TaskPath, State
```

For additional information:

```powershell
Get-ScheduledTask |
    ForEach-Object {
        [PSCustomObject]@{
            TaskName = $_.TaskName
            TaskPath = $_.TaskPath
            UserId   = $_.Principal.UserId
            RunLevel = $_.Principal.RunLevel
        }
    }
```

The important relationship is:

```text
Scheduled Task
      │
      ▼
Execution Account
      │
      ▼
Privileges
      │
      ▼
Script / Binary
```

Then inspect the referenced script or executable for exposed credentials.

---

## 7. Inspect Environment Variables

Applications sometimes expose credentials through environment variables.

### Windows

```powershell
Get-ChildItem Env:
```

Search selectively:

```powershell
Get-ChildItem Env: |
    Where-Object {
        $_.Name -match 'PASS|PWD|SECRET|TOKEN|KEY|CRED'
    }
```

### Linux

```bash
env
```

Targeted searches:

```bash
env | grep -Ei 'pass|pwd|secret|token|key|cred'
```

Environment variables should be treated as potentially sensitive.

Do not automatically treat every variable containing the word `KEY` or `TOKEN` as a credential.

---

## 8. Inspect Application Configuration

Web applications and enterprise software frequently use configuration files containing connection information.

Common examples include:

```text
web.config
app.config
applicationHost.config
.env
settings.json
appsettings.json
database configuration files
```

Look for:

```text
Connection Strings
Database Users
Database Passwords
API Keys
Service Credentials
Authentication Settings
```

Example relationship:

```text
Application
     │
     ▼
Configuration
     │
     ▼
Database Credential
     │
     ▼
Database Account
     │
     ▼
Database Privileges
```

The application credential may not correspond to a domain account, so account type must be established before correlating it with AD.

---

## 9. Inspect Registry Locations

Windows applications may store configuration information in the registry.

Useful areas can include:

```text
HKLM\Software\
HKCU\Software\
Application-specific registry keys
```

For example:

```powershell
Get-ChildItem HKLM:\Software -ErrorAction SilentlyContinue
```

Registry enumeration should be targeted to the application or service being assessed.

The objective is to identify:

* Application configuration
* Service configuration
* Credential references
* Stored secrets

Do not assume that a registry value containing `Password` contains a plaintext password; it may be encrypted, encoded, masked, or simply a configuration label.

---

## 10. Inspect Credential Stores

Operating systems and applications may maintain dedicated credential stores.

Examples include:

* Windows Credential Manager
* Browser credential stores
* Application-specific credential stores
* SSH configuration and key material
* Cloud CLI credential files
* Kerberos credential caches

For Windows, identify stored credentials where authorized:

```cmd
cmdkey /list
```

This can show stored credential targets.

The output should then be interpreted carefully.

A stored credential target does not necessarily reveal the underlying password.

---

## 11. Inspect User Profiles

User profile directories can contain application configuration and authentication artifacts.

Windows:

```text
C:\Users\<username>\
```

Linux:

```text
/home/<username>/
```

Potential areas include:

```text
Configuration files
SSH files
Application data
Command history
Development files
Backup files
Scripts
Cloud CLI configuration
```

The important question is:

```text
Does this file contain authentication material?
```

rather than:

```text
Does this file look interesting?
```

---

## 12. Inspect Command History

Command history may expose credentials when administrators place authentication material directly in commands.

### PowerShell

Depending on configuration and PowerShell version:

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath `
    -ErrorAction SilentlyContinue
```

### Linux

```bash
cat ~/.bash_history 2>/dev/null
```

Other shells may maintain different history files.

Look for commands involving:

```text
Passwords
Database connections
Remote authentication
Cloud credentials
API tokens
Administrative commands
```

Be careful not to reproduce discovered secrets in notes or public repositories.

---

## 13. Inspect Backup and Temporary Files

Backup and temporary files may contain older configuration data.

Examples:

```text
.bak
.old
.backup
.tmp
.save
.zip
.tar
config copies
database dumps
```

Examples of potentially interesting locations include:

```text
C:\Backup\
C:\Temp\
C:\Windows\Temp\
/tmp/
/var/backups/
```

An old configuration file may contain credentials that are no longer valid.

Therefore:

```text
Credential Found
      │
      ▼
Current?
      │
 ┌────┴────┐
No         Yes
 │           │
 ▼           ▼
Document   Validate
as stale   Context
```

---

## 14. Identify Credential Type

After finding authentication material, classify it.

Possible categories include:

| Credential Type    | Example                  |
| ------------------ | ------------------------ |
| Username           | `EXAMPLE\svc-web`        |
| Plaintext password | Password string          |
| Password hash      | NTLM hash                |
| Kerberos ticket    | TGT / service ticket     |
| API key            | Application token        |
| Access token       | Bearer-style token       |
| SSH key            | Private key              |
| Connection string  | Database authentication  |
| Cloud credential   | Provider-specific secret |

Classification determines the next stage of analysis.

---

## 15. Identify the Associated Account

Never stop at:

```text
Credential discovered
```

Determine the owner.

For an AD account:

```text
Credential
    │
    ▼
Domain Account
    │
    ├── Groups
    ├── SPNs
    ├── ACLs
    └── Privileges
```

Useful AD queries include:

```powershell
Get-ADUser -Identity svc-web -Properties *
```

For computer accounts:

```powershell
Get-ADComputer -Identity WEB01$ -Properties *
```

If the credential belongs to an application-specific local account, document that separately.

---

## 16. Determine Whether the Credential Is Current

A discovered credential may be:

* Active
* Expired
* Disabled
* Rotated
* Stale
* Application-specific
* Stored only for historical purposes

For an AD account, inspect account state:

```powershell
Get-ADUser -Identity svc-web `
    -Properties Enabled,PasswordLastSet,AccountExpirationDate
```

For a computer:

```powershell
Get-ADComputer -Identity WEB01$ `
    -Properties Enabled,PasswordLastSet
```

The presence of a password in a file is therefore not sufficient evidence of current access.

---

## 17. Map Account Privileges

Once the account is identified, map its privileges.

### Groups

```powershell
Get-ADPrincipalGroupMembership svc-web |
    Select-Object Name
```

### User Information

```powershell
Get-ADUser svc-web -Properties MemberOf,Description,Enabled
```

Then correlate with information collected during enumeration:

```text
Credential
    │
    ▼
Account
    │
    ├── Groups
    ├── SPNs
    ├── ACLs
    ├── Delegation
    └── Accessible Systems
```

This is where credential discovery becomes relevant to privilege escalation.

---

## 18. Determine Authentication Scope

A credential should be mapped to where it can authenticate.

Consider:

```text
Domain
Local System
Remote System
Application
Database
File Share
Web Application
Service
```

For example:

```text
Credential
    │
    ▼
svc-web
    │
    ├── Web Server
    ├── Database
    └── File Share
```

Do not assume that domain credentials provide administrative access everywhere in the domain.

Authentication and authorization are separate concepts.

---

## 19. Validate Carefully

Credential validation should be controlled.

Use the following sequence:

```text
Credential Identified
        │
        ▼
Account Identified
        │
        ▼
Account Status Checked
        │
        ▼
Authentication Scope Identified
        │
        ▼
Authorized Validation
        │
        ▼
Privilege Verified
        │
        ▼
Evidence Collected
```

Avoid unnecessary authentication attempts.

Repeated failed authentication can:

* Trigger account lockouts
* Generate security alerts
* Affect production systems
* Distort assessment results

---

## 20. Map the Potential Attack Path

After validation, combine the credential with the existing AD map.

Example:

```text
Discovered Credential
        │
        ▼
svc-web
        │
        ├── Member of WebAdmins
        │
        ├── SPN: HTTP/web01
        │
        └── Access to WEB01
                 │
                 ▼
          Server Permissions
                 │
                 ▼
        Potential Next Step
```

The final relationship should be evidence-based.

Do not automatically classify the credential as a privilege-escalation finding.

---

## 21. Credential Discovery Evidence

For each significant discovery, record:

```text
Finding:
Source:
System:
File / Location:
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

For example:

```text
Finding:
Service credential exposed in application configuration

Source:
C:\inetpub\wwwroot\application\web.config

Credential Type:
Application / database credential

Associated Account:
svc-app

Account Status:
Enabled

Validation:
Validated within authorized application scope

Observed Privileges:
Database access

Potential Impact:
Requires further privilege and access-path analysis
```

Never place real passwords, hashes, tokens, or private keys in a public GitHub repository.

Use sanitized examples such as:

```text
USERNAME=svc-example
PASSWORD=<REDACTED>
TOKEN=<REDACTED>
```

---

## Common False Positives

Credential discovery produces many false positives.

Examples include:

```text
password=
```

where the value is:

```text
empty
placeholder
encrypted
masked
sample data
documentation text
```

Other examples:

* Test credentials
* Disabled accounts
* Expired passwords
* Example configuration files
* Development-only secrets
* Non-production credentials
* Tokens that have already expired

Always validate context.

---

## Common Mistakes

### Mistake 1: Treating Every Password String as Valid

```text
password found
      ≠
valid credential
```

### Mistake 2: Ignoring Account Context

A credential has little meaning without knowing which account owns it.

### Mistake 3: Ignoring Privileges

A valid low-privilege credential may not provide meaningful escalation.

### Mistake 4: Testing Outside Scope

Do not use discovered credentials against systems that are outside the authorized assessment.

### Mistake 5: Storing Secrets in Documentation

Never commit real secrets to GitHub, reports, screenshots, or public notes.

### Mistake 6: Assuming Domain Administrator Access

A domain account is not automatically a Domain Admin.

### Mistake 7: Ignoring Credential Rotation

Configuration files frequently contain stale credentials.

---

## Credential Discovery Checklist

### Initial Context

* [ ] Identified current user
* [ ] Identified hostname
* [ ] Identified domain
* [ ] Identified available permissions

### Files and Configuration

* [ ] Reviewed application configuration
* [ ] Reviewed scripts
* [ ] Reviewed connection strings
* [ ] Reviewed backup files
* [ ] Reviewed temporary files
* [ ] Reviewed relevant user directories

### Services and Tasks

* [ ] Enumerated services
* [ ] Identified service accounts
* [ ] Enumerated scheduled tasks
* [ ] Identified task execution accounts
* [ ] Inspected referenced scripts and binaries

### Credential Stores

* [ ] Reviewed relevant credential stores
* [ ] Reviewed environment variables
* [ ] Reviewed command history
* [ ] Reviewed application-specific credential locations
* [ ] Reviewed SSH/cloud credential locations where applicable

### Validation

* [ ] Identified credential type
* [ ] Identified associated account
* [ ] Checked account status
* [ ] Determined whether credential is current
* [ ] Determined authentication scope
* [ ] Validated only within authorized scope

### Correlation

* [ ] Checked group membership
* [ ] Checked privileges
* [ ] Checked SPNs
* [ ] Checked ACLs
* [ ] Checked delegation
* [ ] Identified potential attack paths
* [ ] Documented evidence
* [ ] Sanitized sensitive information

---

## Transition to Password Reuse

Once exposed credentials have been identified and their account context established, investigate whether the same authentication material appears across multiple accounts, systems, or services.

```text
Credential Discovery
        │
        ▼
Credential Identified
        │
        ▼
Associated Account
        │
        ▼
Authentication Scope
        │
        ▼
Possible Reuse
        │
        ▼
Password Reuse Analysis
```

The next file is:

**`04-Credentials-and-Authentication/Password-Reuse.md`**
