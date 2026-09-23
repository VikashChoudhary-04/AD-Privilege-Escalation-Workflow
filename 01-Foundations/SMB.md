# SMB

Server Message Block (SMB) is a network protocol used by Windows systems for sharing files, directories, printers, and other network resources.

SMB is deeply integrated into Windows environments and is commonly encountered during Active Directory security assessments.

From an assessment perspective, SMB can provide information about:

* Domain infrastructure
* Shared resources
* Users and groups
* File permissions
* Administrative access
* Scripts and configuration files
* Potentially sensitive information

## What SMB Provides

SMB can provide network access to resources such as:

* File shares
* Directories
* Printers
* Named pipes
* Administrative shares

A simplified model is:

```text id="c6q4m1"
Client
  │
  │ SMB
  ▼
Windows Server
  │
  ├── File Shares
  ├── Administrative Shares
  ├── Named Pipes
  └── Other Resources
```

## SMB in an Active Directory Environment

SMB is commonly used between domain-joined systems.

For example:

```text id="j5n8p3"
Domain User
    │
    │ Authentication
    ▼
Windows Server
    │
    │ SMB
    ▼
Network Share
```

Access to the share depends on the user's authentication context and the permissions assigned to the resource.

## SMB Port

SMB commonly operates over:

```text id="r2v6k9"
TCP 445
```

Older environments may also use SMB over NetBIOS-related services, including:

```text id="q8m4x2"
TCP 139
```

Modern Windows environments primarily use direct-hosted SMB over TCP port `445`.

## SMB Authentication

SMB can use Windows authentication mechanisms to determine who is requesting access.

In an Active Directory environment, authenticated access can involve domain credentials and Kerberos or NTLM depending on the circumstances and configuration.

Conceptually:

```text id="w3s7p5"
Client
  │
  │ Authentication
  ▼
SMB Server
  │
  ▼
Access Decision
  │
  ├── Allowed
  └── Denied
```

The authentication mechanism and resulting permissions depend on the environment.

## SMB Shares

A share provides network access to a resource.

For example:

```text id="m7x2c4"
\\SERVER01\Shared
```

The share name identifies the network resource.

An assessment may discover shares such as:

```text id="r4k8v1"
\\SERVER01\Public
\\SERVER01\Projects
\\SERVER01\IT
```

The names themselves can provide useful information about the environment.

## Share Permissions vs NTFS Permissions

Access to a Windows file through SMB can be affected by two major permission layers:

1. Share permissions
2. NTFS permissions

Conceptually:

```text id="q6p9t3"
SMB Share Permission
        │
        ▼
    Share Access
        │
        ▼
   NTFS Permission
        │
        ▼
   Effective Access
```

The resulting access is determined by the interaction between these permission systems.

## Administrative Shares

Windows systems commonly expose administrative shares for administrative operations.

Examples include:

```text id="k3v8m2"
C$
ADMIN$
IPC$
```

These shares are intended for administrative use.

Access generally requires appropriate privileges.

For example:

```text id="a5n7q4"
\\SERVER01\C$
```

may provide access to the system drive when the authenticated identity has the necessary administrative permissions.

## IPC$

`IPC$` is a special SMB share used for inter-process communication and certain remote management and authentication operations.

It is commonly encountered during Windows and Active Directory enumeration.

Conceptually:

```text id="u2r5m8"
Client
  │
  ▼
IPC$
  │
  ├── Named Pipes
  └── Remote Communication
```

The presence of `IPC$` does not itself indicate a vulnerability.

## Named Pipes

SMB can transport named-pipe communication.

Named pipes are used by Windows services and management mechanisms for communication between systems and processes.

Conceptually:

```text id="p7x3m9"
SMB
 │
 └── Named Pipe
       │
       └── Windows Service / RPC
```

This is one reason SMB can expose more functionality than simple file sharing.

## SMB and RPC

Windows Remote Procedure Call (RPC) mechanisms can operate through SMB-related infrastructure.

This can support administrative and management operations.

A simplified relationship is:

```text id="y4k8s2"
Remote Client
     │
     ▼
    SMB
     │
     ▼
Named Pipes / RPC
     │
     ▼
Windows Service
```

This becomes relevant when enumerating Windows systems remotely.

## SMB and Domain Controllers

Domain Controllers commonly provide SMB services.

For example:

```text id="x6q2v8"
Domain Controller
      │
      ├── SMB
      ├── LDAP
      ├── Kerberos
      └── DNS
```

SMB access to a Domain Controller can therefore expose domain-related resources and administrative interfaces depending on permissions.

## SYSVOL

One particularly important AD-related resource is `SYSVOL`.

Domain Controllers use `SYSVOL` to distribute domain-wide files, including Group Policy-related data and scripts.

Conceptually:

```text id="z3w7p4"
Domain Controller
       │
       ▼
     SYSVOL
       │
       ├── Group Policy
       ├── Scripts
       └── Other Domain Files
```

SMB can provide access to the `SYSVOL` share for domain users according to the environment's permissions.

## NETLOGON

Domain Controllers also commonly expose the `NETLOGON` share.

`NETLOGON` can contain domain-related scripts and files used during domain operations.

Conceptually:

```text id="v8m5q2"
Domain Controller
       │
       ├── SYSVOL
       │
       └── NETLOGON
```

These shares can therefore be relevant during AD enumeration.

## SMB Enumeration

SMB enumeration can reveal information such as:

* Hostname
* Domain name
* Operating system
* SMB dialects
* Available shares
* Share permissions
* User information
* Group information
* Domain-related resources

The exact information available depends on the target's configuration and the authentication context.

## Null Sessions

A null session refers to certain forms of unauthenticated SMB access.

Historically, Windows environments exposed more information through null sessions than modern configurations typically allow.

Conceptually:

```text id="p4y8r3"
Unauthenticated Client
        │
        ▼
       SMB
        │
        ▼
Limited Information
```

Modern systems commonly restrict such access.

Therefore:

> **Do not assume that null-session enumeration is available. Test the actual configuration and document the result.**

## Authenticated SMB Enumeration

Authenticated domain credentials can provide significantly more visibility.

For example:

```text id="s2n7x5"
Domain User
     │
     ▼
Authenticated SMB
     │
     ├── Shares
     ├── Files
     ├── Directories
     └── Other Resources
```

A low-privileged account may still have access to information that is useful for further assessment.

## SMB and Sensitive Files

Accessible shares can contain files such as:

* Configuration files
* Scripts
* Documentation
* Backups
* Deployment files
* Log files
* Application data

Some files may contain sensitive information such as:

* Passwords
* API keys
* Connection strings
* Internal hostnames
* Service account information

The presence of sensitive information depends entirely on the organization's configuration and file-handling practices.

## SMB and Credential Exposure

Scripts and configuration files stored on accessible shares can sometimes contain credentials.

For example:

```text id="k6r9m1"
SMB Share
   │
   ▼
Script / Configuration
   │
   ▼
Credential
   │
   ▼
Further Authentication
```

Credentials discovered in this way must be validated carefully and only within the authorized scope.

## SMB and Lateral Movement

SMB can be used for legitimate remote administration and resource access.

During an authorized assessment, SMB relationships can also help identify potential lateral-movement paths.

For example:

```text id="f5p8w2"
Compromised Identity
       │
       ▼
SMB Authentication
       │
       ▼
Remote System
       │
       ▼
Accessible Resource
```

The actual security impact depends on the privileges associated with the identity and the permissions configured on the target.

## SMB and Administrative Access

Administrative SMB shares such as `C$` can be useful for remote administration.

If a principal has appropriate administrative privileges:

```text id="n7v3q6"
Administrator
     │
     ▼
\\SERVER01\C$
     │
     ▼
System File Access
```

This can be evidence of administrative control over the target system.

However, the existence of an administrative share alone does not establish that the current user has access to it.

## SMB Signing

SMB signing provides integrity protection for SMB communications.

Whether SMB signing is required or merely supported can have security implications depending on the environment and the attack scenario.

During an assessment, useful information includes:

* Whether SMB signing is enabled
* Whether signing is required
* Which systems require it
* Whether configuration is consistent across important systems

SMB signing should be evaluated in the context of the actual attack surface rather than treated as an isolated finding.

## SMB Versions

Modern Windows environments support newer SMB protocol versions, while older environments may support legacy versions.

The protocol version can affect:

* Security features
* Performance
* Compatibility
* Available protections

Legacy SMB configurations should therefore be identified during network enumeration.

## SMB and NTLM

SMB can use NTLM authentication in certain circumstances.

This is relevant because NTLM has a different security model from Kerberos.

A simplified relationship is:

```text id="d8q4m7"
SMB
 │
 ├── Kerberos
 │
 └── NTLM
```

The actual authentication mechanism depends on the environment, protocol negotiation, service configuration, and authentication context.

Understanding which authentication mechanisms are available can be important when evaluating Windows and AD attack paths.

## SMB and LDAP Relationship

SMB and LDAP provide different types of information.

LDAP primarily exposes directory information:

```text id="w6p2c8"
LDAP
 │
 └── Directory Objects
```

SMB primarily exposes network resources:

```text id="r9m4x1"
SMB
 │
 └── Network Resources
```

Together they can provide complementary visibility:

```text id="v5k8q3"
              AD Assessment
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        LDAP                 SMB
          │                   │
     AD Objects          Network Resources
          │                   │
          └─────────┬─────────┘
                    ▼
             Relationship Map
```

## SMB Enumeration Workflow

A conceptual SMB workflow is:

```text id="e8p3n6"
Identify Target
      ↓
Identify SMB
      ↓
Determine Authentication Context
      ↓
Enumerate Shares
      ↓
Review Share Permissions
      ↓
Inspect Accessible Resources
      ↓
Identify Sensitive Information
      ↓
Map Relevant Relationships
      ↓
Investigate Potential Attack Paths
```

The specific commands and tools belong in the later enumeration section.

## Assessment Questions

When encountering SMB, ask:

```text id="k7q2v5"
Is SMB available?
        ↓
Which SMB versions are supported?
        ↓
Is signing required?
        ↓
Can unauthenticated access retrieve information?
        ↓
What shares exist?
        ↓
What can the current account access?
        ↓
Are SYSVOL and NETLOGON accessible?
        ↓
Are sensitive files exposed?
        ↓
Does any discovered information lead
to another identity or privilege?
```

## What to Remember

1. SMB is widely used for Windows network resource sharing.
2. SMB commonly operates over TCP `445`.
3. TCP `139` may be encountered in older NetBIOS-based configurations.
4. SMB access is controlled by authentication and permissions.
5. Share permissions and NTFS permissions both influence file access.
6. Administrative shares such as `C$` and `ADMIN$` are intended for administrative use.
7. `IPC$` supports certain inter-process and remote communication mechanisms.
8. Domain Controllers commonly expose `SYSVOL` and `NETLOGON`.
9. Accessible shares can contain useful configuration and security information.
10. Scripts and configuration files can sometimes expose sensitive credentials.
11. SMB can use authentication mechanisms including Kerberos and NTLM depending on the environment.
12. SMB signing configuration can be relevant to security assessment.
13. SMB and LDAP provide complementary information during AD enumeration.

## Security Assessment Perspective

Do not stop at:

```text id="j3x8m6"
"Port 445 is open."
```

Instead ask:

```text id="w5n2q9"
What SMB service is running?
        ↓
What shares are available?
        ↓
What can my current identity access?
        ↓
What information is exposed?
        ↓
Does that information identify
another credential, system, or relationship?
        ↓
Does it create a potential attack path?
```

SMB therefore becomes an important source of information during the enumeration and privilege-escalation stages of an Active Directory assessment.
