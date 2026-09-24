# References

## Purpose

This section contains supporting resources for the **AD Privilege Escalation Workflow**.

Unlike the numbered directories, `References/` is not a separate assessment phase.

Instead, it supports the complete workflow:

```text
Foundations
     ↓
First 5 Minutes
     ↓
Enumeration
     ↓
Credentials & Authentication
     ↓
Privilege Escalation Paths
     ↓
Attack Path Analysis
     ↓
Privilege Validation
     ↓
Reporting
     ↓
Labs

     ↑
 References
 support all stages
```

Use references to understand technologies, verify behavior, research security relationships, and confirm tool usage.

---

## Reference Priority

Prefer authoritative sources whenever possible.

A practical order is:

```text
Official Documentation
        ↓
Security Standards / Frameworks
        ↓
Project Documentation
        ↓
Security Research
        ↓
Training Resources
        ↓
Community Material
```

Community resources can be useful, but important technical conclusions should be verified against reliable documentation or controlled testing.

---

## Microsoft Active Directory Documentation

Microsoft documentation should be the primary reference for Windows and Active Directory behavior.

Useful areas include:

* Active Directory Domain Services
* Domain controllers
* Domains and forests
* Users and groups
* Organizational Units
* Group Policy
* LDAP
* Kerberos
* SMB
* Access control
* Security descriptors
* Authentication
* Delegation
* Active Directory Certificate Services

Official documentation:

```text
https://learn.microsoft.com/
```

---

## Active Directory Domain Services

Active Directory Domain Services documentation provides information about:

* Directory architecture
* Domain structure
* Forest structure
* Replication
* Authentication
* Directory objects
* Administrative boundaries

Reference:

```text
https://learn.microsoft.com/windows-server/identity/ad-ds/
```

---

## Active Directory Security Groups

Use Microsoft documentation when researching:

* Domain Users
* Domain Admins
* Enterprise Admins
* Administrators
* Account Operators
* Backup Operators
* Server Operators
* Protected Users
* Group scopes

Understanding the actual purpose and scope of a group is preferable to assuming privilege from its name.

---

## Group Policy

Microsoft Group Policy documentation is useful for understanding:

* GPO architecture
* GPO processing
* Linking
* Security filtering
* Delegation
* Administrative templates
* Policy precedence

Reference:

```text
https://learn.microsoft.com/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview
```

---

## LDAP

LDAP is fundamental to Active Directory enumeration.

Useful topics include:

* Distinguished Names
* LDAP filters
* Directory searches
* Object attributes
* Schema
* Authentication

Reference:

```text
https://learn.microsoft.com/windows/win32/adsi/ldap-adspath
```

---

## Kerberos

Microsoft Kerberos documentation should be used when studying:

* Ticket Granting Tickets
* Service tickets
* SPNs
* Pre-authentication
* Delegation
* Authentication flows
* Kerberos configuration

Reference:

```text
https://learn.microsoft.com/windows-server/security/kerberos/kerberos-authentication-overview
```

---

## SMB

SMB documentation is useful for understanding:

* File shares
* Authentication
* Access control
* SMB versions
* Administrative shares
* Network file access

Reference:

```text
https://learn.microsoft.com/windows-server/storage/file-server/smb-overview
```

---

## Windows Access Control

Access control is essential when analyzing Active Directory ACL relationships.

Important concepts include:

* Security descriptors
* DACLs
* SACLs
* ACEs
* Ownership
* Inheritance
* Access tokens
* Security identifiers

Reference:

```text
https://learn.microsoft.com/windows/win32/secauthz/access-control
```

---

## Active Directory Certificate Services

Use Microsoft documentation when studying:

* Enterprise Certificate Authorities
* Certificate templates
* Enrollment
* Authentication certificates
* Certificate mapping
* PKI architecture

Reference:

```text
https://learn.microsoft.com/windows-server/identity/ad-cs/
```

---

## PowerShell Active Directory Module

The Active Directory PowerShell module is useful throughout the workflow.

Common commands include:

```powershell
Get-ADDomain
Get-ADForest
Get-ADUser
Get-ADGroup
Get-ADGroupMember
Get-ADComputer
Get-ADOrganizationalUnit
Get-ADDomainController
Get-ADTrust
```

Reference:

```text
https://learn.microsoft.com/powershell/module/activedirectory/
```

Always check command documentation before relying on environment-specific behavior.

---

## BloodHound

BloodHound models Active Directory and related identity relationships as a graph.

It is useful for:

* Group membership analysis
* ACL analysis
* Session relationships
* Delegation analysis
* Attack-path discovery
* Privilege relationship visualization

Official project:

```text
https://github.com/SpecterOps/BloodHound
```

Documentation:

```text
https://bloodhound.specterops.io/
```

BloodHound relationships should be manually validated before being reported as confirmed attack paths.

---

## SharpHound

SharpHound is the official BloodHound data collector.

It can collect information about relationships such as:

* Groups
* Computers
* Sessions
* ACLs
* Trusts
* Containers
* Active Directory configuration

Official project:

```text
https://github.com/SpecterOps/SharpHound
```

Collection should remain within the authorized assessment scope.

---

## Certipy

Certipy is commonly used for assessing Active Directory Certificate Services.

It can assist with:

* CA discovery
* Certificate template enumeration
* Enrollment analysis
* Certificate-related attack-path research

Official project:

```text
https://github.com/ly4k/Certipy
```

Tool findings should be correlated with the actual AD CS configuration.

---

## Impacket

Impacket provides Python implementations of numerous network protocols used in Windows and Active Directory environments.

It is useful for understanding and testing protocols such as:

* SMB
* MSRPC
* LDAP-related workflows
* Kerberos
* NTLM

Official project:

```text
https://github.com/fortra/impacket
```

Use individual utilities only within authorized environments.

---

## NetExec

NetExec can assist with authorized Windows and Active Directory enumeration and validation.

Official project:

```text
https://github.com/Pennyw0rth/NetExec
```

Use it as a supporting assessment tool rather than treating automated output as final evidence.

---

## PowerView

PowerView provides PowerShell-based Active Directory enumeration capabilities.

It can assist with:

* Domain enumeration
* User enumeration
* Group enumeration
* Computer enumeration
* ACL analysis
* Trust analysis

PowerView is associated with the PowerSploit project.

Reference:

```text
https://github.com/PowerShellMafia/PowerSploit
```

Verify important findings through current directory state where possible.

---

## Microsoft Sysinternals

Sysinternals utilities can help with Windows system analysis.

Official documentation:

```text
https://learn.microsoft.com/sysinternals/
```

Useful tools vary according to the assessment objective.

---

## MITRE ATT&CK

MITRE ATT&CK provides a knowledge base of adversary behaviors and techniques.

It can help map findings to broader security concepts involving:

* Credential Access
* Discovery
* Lateral Movement
* Privilege Escalation
* Persistence
* Defense Evasion

Reference:

```text
https://attack.mitre.org/
```

ATT&CK mappings should describe the behavior observed rather than being added only for completeness.

---

## OWASP

OWASP primarily focuses on application security, but its documentation and testing philosophy can still provide useful guidance for:

* Structured testing
* Evidence collection
* Risk communication
* Reporting

Reference:

```text
https://owasp.org/
```

Use AD-specific references for Active Directory behavior.

---

## NIST

NIST publications can provide broader guidance for:

* Identity management
* Access control
* Authentication
* Risk management
* Security testing
* Incident response

Reference:

```text
https://www.nist.gov/
```

Relevant publications should be selected according to the assessment context.

---

## Microsoft Security Guidance

Microsoft security guidance can help when developing remediation recommendations.

Useful topics include:

* Identity protection
* Privileged access
* Credential protection
* Active Directory security
* Windows security
* Authentication

Reference:

```text
https://learn.microsoft.com/security/
```

---

## Windows Security Documentation

Reference:

```text
https://learn.microsoft.com/windows/security/
```

Useful areas include:

* Authentication
* Credential protection
* Security policies
* Access control
* Identity protection

---

## PortSwigger Web Security Academy

PortSwigger Web Security Academy primarily focuses on web application security rather than Active Directory.

It remains useful for general penetration-testing skills such as:

* Structured methodology
* Authentication testing
* Authorization testing
* Evidence-driven validation

Reference:

```text
https://portswigger.net/web-security
```

Use AD-focused labs and documentation for Active Directory-specific techniques.

---

## TryHackMe

TryHackMe provides controlled training environments covering Windows and Active Directory concepts.

Reference:

```text
https://tryhackme.com/
```

Training labs can be useful for practicing:

* Enumeration
* Windows authentication
* Active Directory concepts
* Privilege relationships
* Attack-path reasoning

Lab behavior should not automatically be assumed to represent every production environment.

---

## GTFOBins and LOLBAS

These projects document legitimate binaries and system utilities that can have security relevance.

### LOLBAS

Windows-focused:

```text
https://lolbas-project.github.io/
```

### GTFOBins

Unix/Linux-focused:

```text
https://gtfobins.github.io/
```

LOLBAS is generally more directly relevant to Windows and Active Directory environments.

---

## CVE Research

When a workflow encounters a product-specific vulnerability, verify it through reliable vulnerability databases.

### CVE

```text
https://www.cve.org/
```

### NIST National Vulnerability Database

```text
https://nvd.nist.gov/
```

Confirm:

* Affected versions
* Preconditions
* Patch status
* Vendor guidance

Do not assume a CVE applies solely because a service or product was detected.

---

## Vendor Documentation

For third-party software integrated with Active Directory, prefer the vendor's own documentation.

Examples may include:

* Identity platforms
* VPN solutions
* Certificate products
* Backup systems
* Endpoint management
* Security products

Vendor-specific behavior should not be inferred from unrelated environments.

---

## Security Research

High-quality security research can help explain complex Active Directory attack paths.

When using research material:

1. Identify the original researcher where possible.
2. Check publication date.
3. Determine the affected technology/version.
4. Verify important behavior independently.
5. Compare with current vendor documentation.

Active Directory security evolves, so older research may require additional validation.

---

## Tool Documentation

Before using any assessment tool, review its official documentation.

Confirm:

```text
Tool Purpose
     ↓
Required Privilege
     ↓
Collection Method
     ↓
Network Impact
     ↓
Output
     ↓
Limitations
```

Do not execute unfamiliar options in sensitive environments without understanding their effect.

---

## Documentation vs Validation

Documentation explains expected behavior.

Validation determines what is true in the assessed environment.

Use both:

```text
Documentation
     +
Enumeration
     +
Manual Verification
     +
Controlled Validation
     =
Defensible Conclusion
```

---

## Version Awareness

Active Directory environments vary significantly.

Record relevant versions where necessary:

```text
Windows Server Version
Domain Functional Level
Forest Functional Level
Tool Version
Certificate Services Configuration
Client Operating System
```

A technique documented for one configuration may behave differently in another.

---

## Reference Notes

When documenting research during an assessment, consider recording:

| Field         | Purpose                        |
| ------------- | ------------------------------ |
| Topic         | What was researched            |
| Source        | Documentation or research      |
| URL           | Reference location             |
| Date Accessed | When it was reviewed           |
| Relevance     | Why it matters                 |
| Validation    | Whether behavior was confirmed |

This makes later reporting easier.

---

## Reference Checklist

### Source Quality

* [ ] Prefer official documentation
* [ ] Identify original research
* [ ] Check publication date
* [ ] Check affected versions
* [ ] Verify important claims

### Tools

* [ ] Use official project repositories
* [ ] Review tool documentation
* [ ] Understand collection behavior
* [ ] Record relevant tool versions
* [ ] Validate automated findings

### Reporting

* [ ] Reference relevant vendor guidance
* [ ] Use appropriate security standards
* [ ] Avoid unsupported claims
* [ ] Preserve useful source information
* [ ] Recheck references before final reporting

---

## Repository Reference Philosophy

The workflow should follow:

```text
Learn From Documentation
        ↓
Enumerate the Environment
        ↓
Form a Hypothesis
        ↓
Validate the Relationship
        ↓
Confirm Effective Privilege
        ↓
Collect Evidence
        ↓
Report the Result
```

References support the analysis.

They do not replace validation.

---

## Repository Structure

The completed repository structure is:

```text
AD-Privilege-Escalation-Workflow/
│
├── README.md
│
├── 01-Foundations/
├── 02-First-5-Minutes/
├── 03-Enumeration/
├── 04-Credentials-and-Authentication/
├── 05-Privilege-Escalation-Paths/
├── 06-Attack-Path-Analysis/
├── 07-Privilege-Validation/
├── 08-Reporting/
├── 09-Labs/
└── References/
    └── README.md
```

The numbered directories represent the practical workflow.

`References/` remains separate because it supports every stage rather than functioning as another assessment step.

---

## Final Workflow

```text
Understand Active Directory
          ↓
Establish Initial Context
          ↓
Enumerate Relationships
          ↓
Analyze Credentials & Authentication
          ↓
Identify Privilege Escalation Paths
          ↓
Analyze Attack Paths
          ↓
Validate Effective Privilege
          ↓
Collect Evidence
          ↓
Report Findings
          ↓
Practice in Authorized Labs
```

This creates a methodology-focused approach to Active Directory privilege escalation rather than a collection of isolated commands or techniques.
