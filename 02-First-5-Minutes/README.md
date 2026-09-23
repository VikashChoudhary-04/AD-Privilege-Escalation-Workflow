# First 5 Minutes

The first few minutes after obtaining access to an Active Directory environment should be used to establish the current position, identity, host, domain, network context, and available privileges.

The objective is not to immediately attempt exploitation.

The objective is to answer:

```text
Who am I?
   ↓
Where am I?
   ↓
What domain am I in?
   ↓
What system am I on?
   ↓
What can I access?
   ↓
What should I investigate next?
```

## Objective

The first five minutes should establish a reliable baseline of the current environment.

At minimum, determine:

* Current username
* Current privileges
* Hostname
* Operating system
* Domain
* Domain Controller
* Network configuration
* Logged-on users
* Domain groups
* Local groups
* Accessible network resources
* Available security tooling
* Current authentication context

## First 5 Minutes Workflow

```text
01. Identify Current Identity
        ↓
02. Identify Host
        ↓
03. Identify Domain
        ↓
04. Identify Domain Controller
        ↓
05. Identify Network Context
        ↓
06. Identify Current Privileges
        ↓
07. Identify Users and Groups
        ↓
08. Identify Accessible Resources
        ↓
09. Identify Security Controls
        ↓
10. Record Initial Findings
```

---

## 1. Identify Current Identity

The first question should be:

> **Which account am I currently operating as?**

Determine:

* Username
* Domain
* User SID
* Group memberships
* Current logon context

Conceptually:

```text
Current Session
      ↓
Identity
      ↓
User + Domain + SID
```

The identity determines the context in which subsequent enumeration is performed.

### Why It Matters

A domain user, local user, service account, and administrative account can have very different visibility and privileges.

Do not assume that the account used to obtain the initial foothold represents the maximum privilege available in the environment.

---

## 2. Identify the Host

Determine the system on which the current session exists.

Collect information such as:

* Hostname
* Operating system
* Architecture
* System role
* Domain membership

Conceptually:

```text
Current Session
      ↓
Host
      ├── Hostname
      ├── OS
      ├── Architecture
      └── Domain Membership
```

This establishes whether the foothold is on:

* A workstation
* A member server
* A Domain Controller
* Another domain-joined system

---

## 3. Identify the Domain

Determine the Active Directory domain associated with the current host and identity.

For example:

```text
Domain:
corp.example.com
```

Record the domain name because it becomes the foundation for later enumeration.

```text
Domain
  ↓
Users
Groups
Computers
GPOs
Trusts
Kerberos
LDAP
```

---

## 4. Identify the Domain Controller

Determine which Domain Controller or Controllers are associated with the domain.

Useful information includes:

* Hostname
* FQDN
* IP address
* Available services

Conceptually:

```text
Current Host
      ↓
Domain
      ↓
Domain Controller
      ↓
AD Services
```

Knowing the Domain Controller provides an important reference point for later LDAP, Kerberos, SMB, DNS, and domain enumeration.

---

## 5. Identify Network Context

Determine the current network configuration.

Collect information such as:

* IP address
* Network interface
* Subnet
* Default gateway
* DNS servers
* Routes
* Reachable networks

A simplified view:

```text
Host
 │
 ├── IP
 ├── Subnet
 ├── Gateway
 ├── DNS
 └── Routes
```

### Why It Matters

Network information can reveal:

* Domain network ranges
* Additional subnets
* Potentially reachable systems
* DNS infrastructure
* Segmentation boundaries

Network visibility can significantly affect the subsequent enumeration strategy.

---

## 6. Identify Current Privileges

Determine what the current identity is actually allowed to do.

Consider:

* Local administrative privileges
* Domain group membership
* Token privileges
* Effective permissions
* Service-related permissions
* Access to administrative resources

Conceptually:

```text
Identity
   ↓
Groups
   ↓
Privileges
   ↓
Effective Access
```

Do not equate group membership alone with complete control.

The actual security context depends on the complete set of permissions and relationships.

---

## 7. Identify Users and Groups

Establish an initial understanding of the identity structure.

Look for:

* Current user
* Local users
* Domain users
* Local groups
* Domain groups
* Privileged groups
* Nested group relationships

Conceptually:

```text
Users
  ↓
Groups
  ↓
Nested Groups
  ↓
Effective Privileges
```

At this stage, the goal is orientation rather than exhaustive enumeration.

Detailed enumeration belongs in:

```text
03-Enumeration/
```

---

## 8. Identify Accessible Resources

Determine what resources the current identity can access.

Potential resources include:

* SMB shares
* Administrative shares
* Network services
* Domain resources
* File systems
* Internal applications

Conceptually:

```text
Current Identity
       ↓
Accessible Resources
       │
       ├── SMB
       ├── Files
       ├── Services
       └── Applications
```

The objective is to understand the current attack surface without immediately attempting exploitation.

---

## 9. Identify Security Controls

Establish an initial understanding of security controls affecting the host.

Depending on the environment, consider:

* Windows Defender
* Endpoint protection
* Firewall configuration
* Application control
* PowerShell restrictions
* Logging
* Network segmentation
* Security monitoring

The purpose is not to bypass controls during the first five minutes.

Instead, understand the environment in which the assessment is taking place.

---

## 10. Identify Authentication Context

Determine how the current session is authenticated.

Relevant questions include:

```text
Is the account local or domain-based?
        ↓
Is Kerberos being used?
        ↓
Is NTLM involved?
        ↓
What credentials or tickets are available?
        ↓
What resources can the current identity authenticate to?
```

The answers can influence later enumeration and attack-path analysis.

---

## 11. Check for Domain Controller Indicators

A host may be a Domain Controller.

Indicators can include the presence of services and components associated with:

* Active Directory Domain Services
* Kerberos
* LDAP
* DNS
* SYSVOL
* NETLOGON

If the current host is a Domain Controller, immediately adjust the assessment context.

```text
Member Host
    ↓
Domain Enumeration

vs.

Domain Controller
    ↓
Domain-Level Infrastructure Analysis
```

The distinction is important because the security impact of actions on a DC is substantially different from actions on an ordinary workstation.

---

## 12. Record Initial Findings

Do not rely entirely on memory.

Record:

```text
Identity:
Domain:
Hostname:
Operating System:
IP Address:
Domain Controller:
Current Privileges:
Groups:
Network:
Accessible Resources:
Security Controls:
Interesting Findings:
```

This initial snapshot becomes the baseline for the rest of the assessment.

---

# First 5 Minutes Checklist

Use this checklist after obtaining a foothold:

```text
[ ] Current username identified
[ ] Domain identified
[ ] User SID identified
[ ] Hostname identified
[ ] Operating system identified
[ ] Architecture identified
[ ] Domain membership confirmed
[ ] Domain Controller identified
[ ] IP address identified
[ ] Network/subnet identified
[ ] DNS configuration identified
[ ] Routes reviewed
[ ] Current privileges identified
[ ] Local groups reviewed
[ ] Domain groups reviewed
[ ] Logged-on users reviewed
[ ] Accessible SMB resources identified
[ ] Authentication context considered
[ ] Security controls noted
[ ] Initial findings documented
```

---

# Decision Point

After the initial five minutes, the next step should be determined by what has been discovered.

A simplified decision tree is:

```text
Initial Foothold
      │
      ▼
Identify Identity
      │
      ▼
Identify Host
      │
      ▼
Identify Domain
      │
      ▼
Identify Domain Controller
      │
      ▼
Assess Privileges
      │
      ├───────────────┐
      │               │
      ▼               ▼
Interesting        Nothing
Relationship       Obvious
Found              Yet
      │               │
      └───────┬───────┘
              ▼
       Systematic Enumeration
              │
              ▼
       03-Enumeration/
```

The absence of an obvious privilege-escalation path during the first five minutes is normal.

The purpose of this stage is to establish context before deeper enumeration.

---

# What Not to Do

The first five minutes should not become a random sequence of commands.

Avoid:

```text
Run every tool available
        ↓
Collect everything
        ↓
Hope something interesting appears
```

Instead:

```text
Establish Context
       ↓
Identify Relevant Questions
       ↓
Perform Targeted Enumeration
       ↓
Build Relationships
       ↓
Investigate Potential Paths
```

This keeps the workflow efficient and reduces unnecessary activity.

---

# Transition to Enumeration

Once the initial context has been established, move to:

```text
03-Enumeration/
```

The next stage expands the initial snapshot into a systematic examination of:

* Domains
* Users
* Groups
* Computers
* Shares
* GPOs
* ACLs
* SPNs
* Trusts
* Delegation

The goal is to transform:

```text
"I have access to this machine."
```

into:

```text
"I understand my identity,
the domain,
the available objects,
the important relationships,
and the potential privilege paths
that require investigation."
```

## Final Principle

The first five minutes are about **orientation, not exploitation**.

A good initial assessment should answer:

```text
Who am I?
Where am I?
What domain am I in?
Who controls the domain?
What can I access?
What privileges do I have?
What relationships should I investigate next?
```

Those answers provide the baseline for the systematic Active Directory enumeration workflow.
