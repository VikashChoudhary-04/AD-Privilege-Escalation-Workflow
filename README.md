# AD Privilege Escalation Workflow

A practical, workflow-driven guide to enumerating, analyzing, and validating Active Directory privilege-escalation paths in authorized security assessments and lab environments.

## Objective

The goal of this repository is to provide a structured methodology for assessing Active Directory environments from an initial foothold toward higher privileges.

Instead of functioning as a collection of commands, this repository focuses on:

* What to enumerate
* Why the information matters
* What relationships or weaknesses to look for
* How findings can lead to potential privilege-escalation paths
* How to validate an identified path safely
* How to document evidence and remediation

## Workflow

The overall workflow follows a progressive assessment methodology:

```text
Foundations
    ↓
First 5 Minutes
    ↓
Enumeration
    ↓
Credentials & Authentication
    ↓
Privilege-Escalation Paths
    ↓
Attack-Path Analysis
    ↓
Privilege Validation
    ↓
Reporting
```

## Repository Structure

| Directory                            | Purpose                                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `01-Foundations/`                    | Core Active Directory concepts and protocols required to understand the workflow                              |
| `02-First-5-Minutes/`                | Initial checks to establish the current position and environment                                              |
| `03-Enumeration/`                    | Systematic enumeration of domains, users, groups, computers, shares, GPOs, ACLs, SPNs, trusts, and delegation |
| `04-Credentials-and-Authentication/` | Authentication mechanisms and common credential-access opportunities                                          |
| `05-Privilege-Escalation-Paths/`     | Common AD privilege-escalation paths and abuse scenarios                                                      |
| `06-Attack-Path-Analysis/`           | Mapping and analyzing relationships and potential attack paths                                                |
| `07-Privilege-Validation/`           | Validating obtained privileges and collecting supporting evidence                                             |
| `08-Reporting/`                      | Documenting findings, attack paths, impact, and remediation                                                   |
| `09-Labs/`                           | Practical lab exercises and documented learning progress                                                      |
| `References/`                        | External references, documentation, and useful resources                                                      |

## Core Areas

### Active Directory Foundations

* Active Directory architecture
* Domains
* Domain Controllers
* Forests and trees
* Users
* Groups
* Computers
* Organizational Units
* Group Policy
* LDAP
* Kerberos
* SMB

### Enumeration

* Network and host discovery
* Domain discovery
* User enumeration
* Group enumeration
* Computer enumeration
* SMB share enumeration
* GPO enumeration
* ACL enumeration
* SPN enumeration
* Trust enumeration
* Delegation enumeration

### Credentials & Authentication

* Credential discovery
* Password reuse
* Kerberoasting
* AS-REP Roasting
* Credential dumping
* Authentication-related weaknesses

### Privilege-Escalation Paths

* Group membership abuse
* ACL abuse
* GPO abuse
* Delegation abuse
* Resource-Based Constrained Delegation
* Active Directory Certificate Services
* Trust abuse

### Attack-Path Analysis

* BloodHound
* Relationship analysis
* Attack-path identification
* Attack-path validation

### Validation & Reporting

* Current privilege validation
* Domain-level privilege validation
* Evidence collection
* Attack-path documentation
* Finding documentation
* Remediation

## Methodology

Each workflow should answer four questions:

```text
What can I discover?
        ↓
What does it mean?
        ↓
What path could it create?
        ↓
How can I validate it?
```

The purpose is to develop a repeatable methodology rather than rely on memorized commands.

## Scope

This repository focuses on Active Directory privilege escalation and related enumeration techniques within:

* Authorized penetration tests
* Security assessments
* CTFs and training environments
* Purpose-built Active Directory labs
* Personal virtualized lab environments

All techniques should be performed only against systems for which appropriate authorization has been obtained.

## Learning Approach

The recommended progression is:

```text
Understand AD
     ↓
Identify the environment
     ↓
Enumerate relationships
     ↓
Identify credentials and authentication weaknesses
     ↓
Identify privilege relationships
     ↓
Map potential attack paths
     ↓
Validate the path
     ↓
Document the result
```

## Related Repositories

This repository is part of a privilege-escalation workflow series covering different environments:

* Linux Privilege Escalation Workflow
* Windows Privilege Escalation Workflow
* AD Privilege Escalation Workflow

Each repository focuses on the methodology and privilege-escalation considerations specific to its environment.

## Disclaimer

This repository is intended for educational purposes and authorized security testing only.

Do not use the techniques documented here against systems, accounts, networks, or organizations without explicit authorization.
