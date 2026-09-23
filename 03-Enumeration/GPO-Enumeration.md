# GPO Enumeration

## Purpose

Group Policy Object (GPO) enumeration is the process of identifying, analyzing, and mapping Group Policy within an Active Directory environment.

GPOs can control configuration across:

* Users
* Computers
* Servers
* Workstations
* Organizational Units
* Domain-wide environments

Because GPOs can influence security configuration and system behavior, their permissions and scope are important during privilege-escalation analysis.

The goal is to determine:

* Which GPOs exist
* What each GPO controls
* Where each GPO is linked
* Which users and computers are affected
* Who can modify or manage each GPO
* Which groups have delegated GPO permissions
* What security-relevant configuration exists
* Whether a GPO relationship creates a potential privilege path

---

## Enumeration Workflow

Use the following workflow:

```text id="n4x7c2"
Identify GPOs
        ↓
Identify GPO GUIDs
        ↓
Identify GPO Links
        ↓
Identify OUs and Scope
        ↓
Review GPO Permissions
        ↓
Identify GPO Administrators
        ↓
Review Computer Configuration
        ↓
Review User Configuration
        ↓
Review SYSVOL Content
        ↓
Identify Security-Relevant Settings
        ↓
Map GPO → OU → Users / Computers
        ↓
Identify Potential Attack Paths
        ↓
Record Findings
        ↓
Move to ACL Enumeration
```

---

## 1. Understand GPO Structure

A GPO consists of two closely related parts:

```text id="q7c4p1"
Group Policy Object
       │
       ├── Active Directory Information
       │
       └── SYSVOL Files
```

The Active Directory portion contains information about the GPO object and its relationships.

The SYSVOL portion contains policy files and configuration data.

Both should be considered during enumeration.

---

## 2. Identify GPOs

Start by enumerating all GPOs in the domain.

PowerShell:

```powershell id="h8y4n6"
Get-GPO -All
```

Useful information includes:

```text
GPO Name
GPO GUID
Creation Time
Modification Time
Status
Owner
```

A GPO can be identified by its GUID.

Example:

```text id="6r0x3b"
GPO Name:
Workstation Security Policy

GUID:
{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}
```

Record the name and GUID together.

---

## 3. Identify GPO GUIDs

The GPO GUID provides a unique identifier for the policy object.

Example:

```text id="2k6m8q"
{12345678-1234-1234-1234-123456789ABC}
```

The GUID is important because the corresponding policy files are stored within SYSVOL using the GPO identifier.

Conceptually:

```text id="j3w5c8"
GPO
 │
 ├── AD Object
 │     └── GUID
 │
 └── SYSVOL
       └── Policies
             └── {GPO-GUID}
```

This relationship is useful when correlating directory information with filesystem content.

---

## 4. Identify GPO Links

A GPO does not automatically affect every object in the domain.

It is applied through links.

GPOs can be linked to:

* Site
* Domain
* Organizational Unit (OU)

Conceptually:

```text id="6g0r4t"
GPO
 │
 └── Link
      │
      ├── Site
      ├── Domain
      └── OU
```

Identify:

```text id="v3w5h2"
GPO
Linked Location
Link Order
Enforced
Enabled
```

PowerShell:

```powershell id="5p7n2x"
Get-GPInheritance -Target "DC=corp,DC=example,DC=com"
```

For an OU:

```powershell id="g4m8k1"
Get-GPInheritance -Target "OU=Servers,DC=corp,DC=example,DC=com"
```

The exact distinguished name depends on the environment.

---

## 5. Understand GPO Scope

The impact of a GPO depends on where it is linked and which objects fall within its scope.

A simplified relationship is:

```text id="9h2q6w"
GPO
 ↓
OU
 ↓
Computers / Users
 ↓
Applied Configuration
```

For example:

```text id="u8m5c1"
Server Security GPO
        ↓
Servers OU
        ↓
WEB01
SQL01
FILE01
```

Do not assume that every computer in the domain receives every GPO.

---

## 6. Identify Organizational Units

GPO enumeration requires understanding the OUs to which policies are linked.

Identify:

```text id="s4k7d3"
OU Name
Distinguished Name
Parent OU
Child OUs
Linked GPOs
Users
Computers
```

Example:

```text id="e1y5q8"
corp.example.com
│
├── Users
│
├── Workstations
│
└── Servers
      ├── Web
      └── Database
```

Different OUs may receive different policy configurations.

---

## 7. Review GPO Permissions

Determine who can manage each GPO.

Potential permissions include:

```text id="c7v3m9"
Read
Write
Modify
Create
Delete
Link
Apply Group Policy
```

The exact permissions exposed depend on the directory ACLs.

The most important question is:

```text id="p1k8r5"
Who can modify this GPO?
```

For example:

```text id="6v2n7q"
GPO
 ↓
Modify Permission
 ↓
Group
 ↓
Users
```

This relationship can become important when analyzing potential privilege paths.

---

## 8. Distinguish Read from Modify

Do not treat GPO access as a single permission.

A principal may be able to:

```text id="9w3k6m"
Read GPO
```

without being able to:

```text id="m4p8x2"
Modify GPO
```

Likewise, permissions related to applying a GPO are different from permissions that allow modification.

Therefore record permissions separately.

Example:

```text id="f6j9t3"
Principal:
CORP\IT-Admins

Permissions:
Read
Write
Modify
```

---

## 9. Identify GPO Owners

Identify the owner of each GPO where accessible.

Example:

```text id="w7c5q2"
GPO:
Workstation Policy

Owner:
CORP\Domain Admins
```

Ownership can be relevant because it is one part of the broader authorization model.

However:

```text id="k2m5p7"
Owner ≠ Automatically Full Effective Control
```

Validate the actual permissions associated with the owner and other principals.

---

## 10. Review Computer Configuration

GPOs can contain computer-side settings.

Potential areas include:

```text id="p5d8x4"
Security Settings
Windows Settings
Administrative Templates
Scripts
Services
Scheduled Tasks
Registry Settings
Software Configuration
```

The exact categories depend on the GPO.

Focus on settings that can influence:

* Authentication
* Local administrators
* Services
* Scheduled tasks
* Security controls
* System configuration
* Software deployment

---

## 11. Review User Configuration

GPOs can also contain user-side settings.

Potential areas include:

```text id="z4n6r8"
Administrative Templates
Windows Settings
Logon / Logoff Scripts
Software Configuration
Registry Settings
Desktop Configuration
```

Determine which users or OUs receive these policies.

For privilege analysis, pay particular attention to configurations that may change:

* User privileges
* Local settings
* Authentication behavior
* Application configuration
* Administrative access

---

## 12. Review SYSVOL Policy Files

GPO-related files are stored in SYSVOL.

A common structure is:

```text id="c6k2m8"
\\<domain>\SYSVOL\<domain>\Policies\
```

Within the policy directory, GPO GUIDs identify individual policies.

Example:

```text id="p3r7w1"
Policies
│
├── {GPO-GUID-1}
├── {GPO-GUID-2}
└── {GPO-GUID-3}
```

Review authorized policy files to correlate them with the corresponding AD GPO objects.

---

## 13. Identify Policy Scripts

GPOs can deploy scripts.

Examples include:

```text id="x4q8s2"
Startup Scripts
Shutdown Scripts
Logon Scripts
Logoff Scripts
```

Scripts may provide information about:

* Administrative workflows
* Service accounts
* Network resources
* Deployment processes
* System configuration

If credential-like information is discovered, treat it as sensitive evidence and validate it rather than assuming it is current.

---

## 14. Review Security Configuration

GPOs may define important security settings.

Examples include:

```text id="t6m2v9"
Password Policies
Account Policies
Audit Policies
User Rights Assignment
Security Options
Windows Firewall
Defender Configuration
Restricted Groups
```

Record the configuration and its scope.

For example:

```text id="8p4w1c"
GPO:
Server Security Policy

Linked To:
Servers OU

Relevant Settings:
User Rights Assignment
Windows Firewall
Security Options
```

Avoid interpreting an individual setting without considering inheritance and policy precedence.

---

## 15. Review Restricted Groups

Restricted Groups can manage membership of local or domain groups through Group Policy.

Conceptually:

```text id="m7q3d1"
GPO
 ↓
Restricted Group
 ↓
Group Membership
 ↓
Computer
```

This can be highly relevant to privilege analysis because policy-controlled group membership may determine local administrative access.

Document:

```text id="s9c2f6"
GPO
Group
Members
Affected Computers
Effective Scope
```

---

## 16. Review Local Group Management

GPOs can also influence local group membership through various policy mechanisms.

For example:

```text id="v5r8k2"
GPO
 ↓
Local Administrators
 ↓
Computer
```

If a domain group is added to local Administrators on a set of servers, that creates a significant identity-to-computer relationship.

Record:

```text id="u4m9x7"
Domain Group
Local Group
Computer Scope
GPO
```

Do not assume effective access until the actual policy scope and membership are validated.

---

## 17. Identify GPOs Affecting High-Value Systems

Prioritize GPOs linked to:

```text id="y6p2q8"
Domain Controllers
Servers
Administrative Workstations
Management Systems
Backup Systems
Certificate Infrastructure
Other High-Value Hosts
```

Example:

```text id="8k4n5r"
Domain Controllers OU
       ↑
       │
Domain Controller Security GPO
```

These policies can have broader security implications than policies affecting ordinary workstations.

---

## 18. Understand GPO Inheritance

GPO application can involve inheritance.

A simplified structure:

```text id="q7m2v9"
Site
 ↓
Domain
 ↓
Parent OU
 ↓
Child OU
 ↓
Computer/User
```

Policies can be affected by:

* Link order
* Inheritance
* Block inheritance
* Enforced links
* Security filtering
* WMI filtering

Therefore, identifying a linked GPO does not automatically establish its final effective configuration.

---

## 19. Review Security Filtering

A GPO may be linked to an OU but restricted to particular security principals.

Conceptually:

```text id="m8k4c2"
GPO
 ↓
Security Filtering
 ↓
User / Group / Computer
 ↓
Policy Application
```

Determine:

```text id="5q3x8m"
Who can read the GPO?
Who can apply the GPO?
Which objects are in scope?
```

This distinction is important when determining whether a discovered policy affects a particular account or machine.

---

## 20. Review WMI Filtering

Some GPOs use WMI filters to determine which systems receive a policy.

Conceptually:

```text id="r6w2p9"
GPO
 ↓
WMI Filter
 ↓
Computer Characteristics
 ↓
Policy Applied / Not Applied
```

Possible criteria may include:

* Operating system
* Version
* Hardware characteristics
* Computer properties

Record the filter and understand its scope before concluding that a GPO affects a particular host.

---

## 21. Identify Potential GPO Abuse Paths

During enumeration, look for relationships such as:

```text id="c3x7m5"
User
 ↓
Group
 ↓
GPO Modification Permission
 ↓
GPO
 ↓
Affected Computer
 ↓
Administrative Impact
```

For example:

```text id="7q2n8v"
CORP\IT-Admins
       ↓
Modify GPO
       ↓
Server Security GPO
       ↓
Servers OU
       ↓
WEB01 / SQL01 / FILE01
```

This is a **potential attack path**, not a validated vulnerability.

Validation belongs in the privilege-escalation phase.

---

## 22. Useful Enumeration Tools

### PowerShell

```text id="m5q9x3"
Get-GPO
Get-GPOReport
Get-GPInheritance
Get-GPPermission
```

Examples:

```powershell id="x2v7n5"
Get-GPO -All
```

```powershell id="j6c4r8"
Get-GPOReport -All -ReportType Html -Path .\gpo-report.html
```

### Windows Group Policy Tools

```text id="k7p2m4"
gpresult
Group Policy Management Console
```

Example:

```cmd id="n4w8q1"
gpresult /r
```

### LDAP

```text id="r5x3m7"
ldapsearch
```

### SMB

```text id="u2k9c6"
smbclient
netexec
```

Useful for accessing or reviewing authorized SYSVOL content.

### Graph-Based Collection

```text id="p8m4x1"
BloodHound-compatible collectors
```

Graph collection can help connect:

```text
GPO
 ↓
OU
 ↓
Computer
 ↓
User / Group
```

---

## 23. Build a GPO Inventory

Maintain a structured inventory.

Example:

```text id="f7m3q9"
GPO Name:
GUID:
Owner:
Linked To:
Link Order:
Enforced:
Security Filtering:
WMI Filter:
Computer Configuration:
User Configuration:
SYSVOL Path:
Modify Permissions:
Apply Permissions:
Affected Users:
Affected Computers:
Interesting Settings:
Potential Relationships:
Notes:
```

Example:

```text id="h5c8w2"
GPO: Server Security Policy
GUID: {XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}
Linked To: Servers OU

Affected Systems:
- WEB01
- SQL01
- FILE01

Interesting Settings:
- Local Administrators configuration
- Security Options

Notes:
- Review GPO permissions and inheritance
```

---

## 24. Build a GPO Relationship Map

Connect the information into a graph.

```text id="v8q4m6"
GPO
 │
 ├── Linked To ──► OU
 │                  │
 │                  ├── Computer
 │                  └── User
 │
 ├── Modified By ─► Group
 │                    │
 │                    └── User
 │
 └── Contains ────► Policy Configuration
```

This allows you to identify relationships such as:

```text id="a6m2r9"
User
 ↓
Group
 ↓
GPO Permission
 ↓
GPO
 ↓
OU
 ↓
Computer
```

These relationships can become important during attack-path analysis.

---

## 25. Assessment Mindset

Ask:

```text
Which GPOs exist?
        ↓
Where are they linked?
        ↓
Which users and computers receive them?
        ↓
Who can modify them?
        ↓
What security-relevant settings do they contain?
        ↓
What is their effective scope?
        ↓
Do any permissions create a meaningful privilege relationship?
```

Do not stop at:

```text
"GPO exists."
```

Instead determine:

```text
GPO
 ↓
Permission
 ↓
Scope
 ↓
Affected Object
 ↓
Effective Impact
```

---

## GPO Enumeration Checklist

### GPO Discovery

* [ ] GPOs enumerated
* [ ] GPO names recorded
* [ ] GPO GUIDs recorded
* [ ] Owners identified
* [ ] Creation/modification information recorded

### Scope

* [ ] GPO links identified
* [ ] Linked OUs identified
* [ ] Domain-level links identified
* [ ] Site-level links identified where relevant
* [ ] Link order reviewed
* [ ] Inheritance reviewed
* [ ] Enforced settings reviewed
* [ ] Security filtering reviewed
* [ ] WMI filtering reviewed

### Configuration

* [ ] Computer configuration reviewed
* [ ] User configuration reviewed
* [ ] Security settings reviewed
* [ ] Restricted Groups reviewed
* [ ] Local group configuration reviewed
* [ ] Scripts reviewed
* [ ] SYSVOL content correlated

### Permissions

* [ ] GPO read permissions identified
* [ ] GPO modification permissions identified
* [ ] Apply permissions identified
* [ ] Delegated administrators identified

### Relationships

* [ ] GPO → OU relationships recorded
* [ ] GPO → Computer relationships recorded
* [ ] GPO → User relationships recorded
* [ ] Group → GPO relationships recorded
* [ ] Potential GPO attack paths documented

### Documentation

* [ ] GPO inventory created
* [ ] Relevant settings recorded
* [ ] Evidence sources recorded
* [ ] Scope and inheritance documented
* [ ] Potential privilege relationships identified

---

## Transition to ACL Enumeration

GPO enumeration shows **how policy is applied and who can control it**.

The next step is to examine **Active Directory access control lists (ACLs)** directly.

Move to:

```text id="w8p4z2"
03-Enumeration/ACL-Enumeration.md
```

The next stage will connect:

```text
Principal
  ↓
ACL
  ↓
AD Object
  ↓
Permission
  ↓
Effective Control
  ↓
Potential Attack Path
```
