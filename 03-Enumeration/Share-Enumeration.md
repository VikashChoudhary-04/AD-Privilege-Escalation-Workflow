# Share Enumeration

## Purpose

Share enumeration is the process of identifying SMB shares within an Active Directory environment and determining how users, groups, and computers can access them.

Windows environments commonly expose shared resources through **SMB**.

The objective is to understand:

* Which systems expose SMB
* Which shares exist
* Which shares are accessible
* Share permissions
* NTFS permissions where visible
* Administrative shares
* SYSVOL and NETLOGON
* Sensitive or interesting resources
* User/group relationships with shares
* Potential paths from a share to other AD resources

The goal is not simply to list shares.

The important question is:

```text
Who can access what?
```

---

## Enumeration Workflow

Use the following workflow:

```text
Identify SMB Hosts
        ↓
Enumerate Shares
        ↓
Identify Share Types
        ↓
Determine Accessible Shares
        ↓
Review Share Permissions
        ↓
Review NTFS Permissions
        ↓
Identify Administrative Shares
        ↓
Review SYSVOL / NETLOGON
        ↓
Identify Interesting Files and Resources
        ↓
Map Users / Groups → Shares
        ↓
Record Findings
        ↓
Move to GPO Enumeration
```

---

## 1. Identify SMB Hosts

Start with the computer inventory from the previous stage.

Identify systems exposing SMB:

```text
445/tcp → SMB
139/tcp → NetBIOS Session Service
```

Potential SMB hosts may include:

```text
Domain Controllers
File Servers
Application Servers
Workstations
Other Windows Systems
```

Do not assume that every Windows host exposes SMB.

Validate actual service availability.

---

## 2. Enumerate SMB Shares

Once SMB hosts are identified, enumerate their available shares.

Useful tools include:

```text
smbclient
netexec
rpcclient
enum4linux-ng
PowerShell
Windows net commands
```

Example:

```bash
smbclient -L //10.10.10.20 -U 'CORP\user'
```

NetExec:

```bash
nxc smb 10.10.10.20 -u user -p 'password' --shares
```

The exact syntax depends on the tool version and authentication method.

Record:

```text
Host
Share Name
Share Type
Access
Description
Authentication Context
```

---

## 3. Understand Common Share Types

Common Windows shares include:

```text
ADMIN$
C$
IPC$
NETLOGON
SYSVOL
User-created shares
Application shares
Department shares
```

Their purposes differ.

### ADMIN$

Administrative share used for remote administration.

### C$

Administrative share exposing the system drive to authorized administrators.

### IPC$

Used for inter-process communication and certain SMB/RPC interactions.

### SYSVOL

Contains domain-wide files used by Active Directory and Group Policy.

### NETLOGON

Provides domain-related logon scripts and other resources.

Not every share is accessible to every user.

---

## 4. Identify Accessible Shares

Separate shares into categories:

```text
Visible
    ↓
Accessible
    ↓
Readable
    ↓
Writable
    ↓
Administrative
```

For example:

```text
Share        Visible    Read    Write
--------------------------------------
SYSVOL       Yes        Yes     No
NETLOGON     Yes        Yes     No
Finance      Yes        Yes     No
Public       Yes        Yes     Yes
ADMIN$       Yes        No      No
```

The exact permissions depend on the environment.

Do not confuse **share visibility** with **resource access**.

---

## 5. Understand Share Permissions

SMB share permissions determine access at the share layer.

Common permissions include:

```text
Read
Change
Full Control
```

Conceptually:

```text
User
 ↓
Group Membership
 ↓
Share Permission
 ↓
Share Access
```

However, this is only one layer of access control.

---

## 6. Understand NTFS Permissions

Windows file systems can apply NTFS permissions independently from SMB share permissions.

The effective permission may therefore depend on both:

```text
Share Permissions
        +
NTFS Permissions
        =
Effective File Access
```

For example:

```text
Share:
Everyone → Change

NTFS:
Users → Read
```

The resulting effective access must be evaluated from both layers.

Do not report share permissions alone as the final file permission.

---

## 7. Identify Readable Shares

Readable shares may contain:

* Documentation
* Scripts
* Configuration files
* Application data
* Deployment files
* Logs
* Administrative resources
* Backup information
* User-generated files

During authorized assessment, review resources according to the agreed scope.

Avoid indiscriminate collection of unrelated personal or sensitive information.

Record:

```text
Host
Share
Path
Access Level
Relevant Finding
```

---

## 8. Identify Writable Shares

Writable shares deserve particular attention because write access can affect resources used by other systems or users.

Examples may include:

```text
Scripts
Deployment directories
Application directories
Shared configuration
Log locations
Public folders
```

However:

```text
Writable Share ≠ Automatically Exploitable
```

Validate:

```text
Who can write?
        ↓
What can they write?
        ↓
Who consumes the resource?
        ↓
How is the resource used?
        ↓
What security impact exists?
```

Do not assume that every writable directory creates privilege escalation.

---

## 9. Identify Administrative Shares

Common administrative shares include:

```text
ADMIN$
C$
D$
IPC$
```

These are generally associated with administrative or system-management functions.

Example:

```text
C$
 ↓
System Drive
```

Access should be evaluated under the current authentication context.

Record:

```text
Host
Administrative Share
Accessible
Authenticated User
Observed Permission
```

Administrative share visibility alone does not establish administrative access.

---

## 10. Enumerate SYSVOL

`SYSVOL` is a critical Active Directory share.

It contains domain-related files, including Group Policy-related content.

Common path:

```text
\\<domain>\SYSVOL
```

Conceptually:

```text
Domain
  ↓
SYSVOL
  ├── Policies
  ├── Scripts
  └── Other Domain Files
```

SYSVOL can provide information about:

* GPO structure
* Domain scripts
* Policy-related files
* Organizational configuration

SYSVOL enumeration is closely connected to GPO enumeration.

---

## 11. Enumerate NETLOGON

`NETLOGON` is another standard domain share.

Common path:

```text
\\<domain>\NETLOGON
```

It may contain:

* Logon scripts
* Administrative scripts
* Domain-related resources
* Deployment-related files

Record:

```text
File
Path
Permissions
Purpose
Interesting References
```

Do not assume that every script contains credentials or sensitive information.

---

## 12. Review Scripts Carefully

Scripts stored on accessible shares may contain references to:

* Service accounts
* Administrative accounts
* Network paths
* Applications
* Scheduled tasks
* Configuration
* Legacy credentials

Examples of script types:

```text
.bat
.cmd
.ps1
.vbs
```

The presence of a credential-like string should be treated as an observation requiring validation.

For example:

```text
username = svc-backup
```

does not prove:

```text
svc-backup = current valid credential
```

Record evidence and validate through authorized means.

---

## 13. Identify Interesting File Types

Depending on the assessment scope, files that may deserve review include:

```text
Configuration files
Scripts
Backup files
Deployment files
Database configuration
Application configuration
Documentation
Log files
```

Examples:

```text
.config
.xml
.ini
.ps1
.bat
.cmd
.sql
.txt
.csv
```

File extension alone does not determine sensitivity.

Evaluate the actual content and business context.

---

## 14. Search for Credential-Related Artifacts

When authorized, review accessible files for credential-related information.

Potential indicators include:

```text
username
password
passwd
credential
connection string
token
secret
API key
```

Use this as an **indicator search**, not as proof that a credential exists.

If a credential is discovered:

```text
Discovery
   ↓
Validate
   ↓
Determine Scope
   ↓
Determine Account
   ↓
Assess Impact
```

Avoid unnecessarily exposing discovered secrets in reports or repositories.

Store sensitive evidence securely.

---

## 15. Identify Share Permissions by Group

Share access is often assigned through groups.

Example:

```text
CORP\Finance
       │
       ▼
Finance Share
       │
       ▼
Read / Change
```

Record:

```text
Group
Share
Permission
Resource
```

Then correlate with user enumeration:

```text
User
 ↓
Group
 ↓
Share
 ↓
Resource
```

This can reveal access relationships that are not obvious from the user account alone.

---

## 16. Identify Cross-System Share Relationships

A single account may have access to resources across multiple systems.

Example:

```text
User
 ├── FILE01\Finance
 ├── FILE02\Projects
 └── APP01\Application
```

Map these relationships.

This can help determine:

* Resource concentration
* Administrative access
* Shared service accounts
* Cross-server access
* Potential lateral movement paths

At this stage, document the relationship rather than automatically treating it as an attack path.

---

## 17. Identify Share-to-Identity Relationships

A useful relationship model is:

```text
User
 ↓
Group
 ↓
Share
 ↓
File
 ↓
Application / Service
```

For example:

```text
CORP\alice
      ↓
Developers
      ↓
\\FILE01\Projects
      ↓
deployment.ps1
      ↓
Application Deployment
```

This kind of relationship can later become relevant to privilege-escalation or attack-path analysis.

---

## 18. Review Share Names and Descriptions

Share names can provide useful context.

Examples:

```text
Finance
HR
Projects
Software
Deploy
Backup
IT
Public
Development
```

However:

```text
Share Name ≠ Confirmed Function
```

Validate the purpose through:

* Directory contents
* Descriptions
* File types
* Server role
* Group permissions
* Administrative documentation

---

## 19. Useful Enumeration Tools

### SMB Client

```text
smbclient
```

Example:

```bash
smbclient -L //10.10.10.20 -U 'CORP\user'
```

### NetExec

```text
netexec
```

Example:

```bash
nxc smb 10.10.10.20 --shares
```

### RPC

```text
rpcclient
```

### enum4linux-ng

```text
enum4linux-ng
```

### Windows

```cmd
net view
net view \\SERVER
```

### PowerShell

```powershell
Get-SmbShare
Get-SmbShareAccess
```

Availability of these commands depends on the host and permissions.

---

## 20. Build a Share Inventory

Maintain a structured inventory.

Example:

```text
Host:
IP:
Share:
Description:
Share Permissions:
NTFS Permissions:
Accessible:
Readable:
Writable:
Authentication Context:
Interesting Files:
Associated Groups:
Associated Users:
Notes:
```

Example:

```text
Host: FILE01
Share: Projects
Access: Read
Group: CORP\Developers

Interesting Files:
- deployment.ps1
- application.config

Notes:
- Review files within authorized scope
```

---

## 21. Build an SMB Relationship Map

Connect the information gathered so far.

```text
Computer
   ↓
SMB
   ↓
Share
   ↓
Permission
   ↓
Group
   ↓
User
```

Example:

```text
FILE01
  ↓
Projects
  ↓
Developers
  ↓
alice
```

This allows share enumeration to contribute directly to the broader AD relationship model.

---

## 22. Assessment Mindset

Ask:

```text
Which hosts expose SMB?
        ↓
Which shares exist?
        ↓
Which shares can I access?
        ↓
What permissions do I have?
        ↓
What is the difference between share and NTFS permissions?
        ↓
Which resources are interesting?
        ↓
Which users/groups can access them?
        ↓
Does any resource create a meaningful privilege relationship?
```

Avoid the assumption:

```text
Readable = Vulnerable
```

Instead:

```text
Accessible Resource
        ↓
Understand Content
        ↓
Understand Permissions
        ↓
Understand Consumers
        ↓
Determine Security Impact
```

---

## Share Enumeration Checklist

### SMB Discovery

* [ ] SMB hosts identified
* [ ] Port 445 reviewed
* [ ] Port 139 reviewed where relevant
* [ ] SMB service information recorded

### Share Discovery

* [ ] Shares enumerated
* [ ] Share types identified
* [ ] Administrative shares identified
* [ ] SYSVOL identified
* [ ] NETLOGON identified
* [ ] Custom shares identified

### Access

* [ ] Visible shares recorded
* [ ] Accessible shares identified
* [ ] Read permissions identified
* [ ] Write permissions identified
* [ ] Authentication context recorded

### Permissions

* [ ] Share permissions reviewed
* [ ] NTFS permissions reviewed where accessible
* [ ] Effective access considered
* [ ] Group-based access documented

### Content

* [ ] Interesting directories identified
* [ ] Relevant scripts reviewed
* [ ] Configuration files reviewed
* [ ] Credential-related artifacts identified where authorized
* [ ] Sensitive evidence handled securely

### Documentation

* [ ] Share inventory created
* [ ] User → Group → Share relationships recorded
* [ ] Computer → Share relationships recorded
* [ ] Interesting resources documented
* [ ] Potential security implications recorded

---

## Transition to GPO Enumeration

Once SMB shares and their contents are understood, the next step is to examine **Group Policy**, one of the mechanisms through which configuration and permissions are distributed across an AD environment.

Move to:

```text
03-Enumeration/GPO-Enumeration.md
```

The next stage will connect:

```text
Domain
  ↓
GPO
  ↓
OU / Computer / User
  ↓
Policy Configuration
  ↓
Permissions
  ↓
Potential Administrative Impact
```
