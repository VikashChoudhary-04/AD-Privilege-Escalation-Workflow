# Credentials and Authentication

## Purpose

This section focuses on identifying, analyzing, and validating authentication mechanisms and credential exposure within an Active Directory environment.

The objective is to determine:

* Where credentials may be exposed
* Which accounts use weak or reused authentication secrets
* Which service accounts are associated with Kerberos service principals
* Which accounts do not require Kerberos pre-authentication
* Whether credential material can be recovered from authorized systems
* How discovered credentials relate to existing privilege-escalation paths

Credential discovery should be treated as a structured process rather than simply collecting passwords.

The important question is:

```text
What authentication material is available?
            │
            ▼
Which account does it belong to?
            │
            ▼
What privileges does that account have?
            │
            ▼
Where can that account authenticate?
            │
            ▼
Does it create a validated privilege path?
```

---

## Position in the Workflow

The overall AD privilege-escalation workflow is:

```text
01-Foundations
      │
      ▼
02-First-5-Minutes
      │
      ▼
03-Enumeration
      │
      ▼
04-Credentials-and-Authentication
      │
      ▼
05-Privilege-Escalation-Paths
      │
      ▼
06-Attack-Path-Analysis
      │
      ▼
07-Privilege-Validation
      │
      ▼
08-Reporting
```

Enumeration establishes what exists in the environment.

This section investigates how authentication and credential exposure may provide access to those resources.

---

## Authentication Attack Surface

An Active Directory environment contains multiple sources of authentication material.

```text
                    Active Directory
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
      Users             Services           Computers
        │                  │                  │
        ▼                  ▼                  ▼
   Passwords             SPNs            Machine Accounts
        │                  │                  │
        └────────────┬─────┴──────────────────┘
                     ▼
             Authentication
                     │
                     ▼
             Access / Privileges
```

During an assessment, investigate authentication material without assuming that every credential exposure is exploitable.

---

## Credential & Authentication Workflow

Use the following workflow:

```text
Review Known Accounts
        │
        ▼
Identify Credential Exposure
        │
        ▼
Check Password Reuse
        │
        ▼
Identify Service Accounts / SPNs
        │
        ▼
Assess Kerberos Authentication
        │
        ├── Kerberoasting
        │
        └── AS-REP Roasting
        │
        ▼
Assess Credential Material
        │
        └── Credential Dumping
        │
        ▼
Map Discovered Credentials
        │
        ▼
Correlate Privileges
        │
        ▼
Validate Authentication Paths
        │
        ▼
Move to Privilege-Escalation Paths
```

---

## 1. Credential Discovery

**File:** `Credential-Discovery.md`

Credential discovery focuses on identifying locations where authentication material may be exposed.

Potential sources include:

* Configuration files
* Application files
* Scripts
* Environment variables
* Scheduled tasks
* Service configurations
* Command histories
* Credential stores
* Registry locations
* Memory, where authorized
* Files containing connection strings
* Backups and deployment artifacts

The workflow should be:

```text
Identify Source
      │
      ▼
Locate Credential Material
      │
      ▼
Identify Associated Account
      │
      ▼
Determine Credential Type
      │
      ▼
Assess Validity
      │
      ▼
Map Account Privileges
```

Finding a password-like string is not equivalent to finding a valid credential.

Always determine:

* Which account owns it
* Whether it is current
* Where it can authenticate
* What privileges it provides

---

## 2. Password Reuse

**File:** `Password-Reuse.md`

Password reuse examines whether the same authentication secret is used across multiple accounts or services.

A simplified relationship is:

```text
Credential
    │
    ├── Account A
    │
    ├── Service B
    │
    └── System C
```

The assessment should establish whether reuse exists and whether it expands access.

Important questions include:

* Is the same password used by multiple accounts?
* Is a service account using the same password elsewhere?
* Does a discovered credential authenticate to another system?
* Does the associated account have additional privileges?

Password reuse should be validated carefully because stale, disabled, or application-specific credentials may produce misleading results.

---

## 3. Kerberoasting

**File:** `Kerberoasting.md`

Kerberoasting focuses on service accounts associated with Kerberos Service Principal Names (SPNs).

The basic relationship is:

```text
User / Service Account
          │
          │ SPN
          ▼
    Kerberos Service
          │
          ▼
Service Ticket
```

The assessment goal is to identify service accounts whose Kerberos authentication configuration warrants further security analysis.

Important enumeration data includes:

* Account name
* SPNs
* Service type
* Associated host
* Group memberships
* Privileges
* Password-management context

A service account having an SPN does not automatically mean that it is vulnerable.

The account's password strength and privileges must also be considered.

---

## 4. AS-REP Roasting

**File:** `AS-REP-Roasting.md`

AS-REP Roasting focuses on accounts configured without Kerberos pre-authentication.

The relevant condition is:

```text
DONT_REQUIRE_PREAUTH
```

The workflow is:

```text
Identify Account
      │
      ▼
Check Pre-Authentication Setting
      │
      ▼
Identify Eligible Accounts
      │
      ▼
Assess Account Privileges
      │
      ▼
Perform Authorized Validation
```

Important information to record includes:

* Account name
* Pre-authentication configuration
* Group memberships
* Privileges
* Account status
* Password-management context

Again, configuration alone does not establish a successful compromise.

---

## 5. Credential Dumping

**File:** `Credential-Dumping.md`

Credential dumping focuses on recovering authentication material from systems where the assessor has authorized access.

Potential credential material may include:

* Password hashes
* Kerberos tickets
* Cached credentials
* Secrets stored by applications
* Service credentials
* Authentication tokens

The workflow is:

```text
Authorized System Access
        │
        ▼
Identify Credential Stores
        │
        ▼
Acquire Credential Material
        │
        ▼
Identify Associated Account
        │
        ▼
Assess Credential Validity
        │
        ▼
Map Privileges
        │
        ▼
Identify Potential Access Paths
```

Credential dumping should always be performed within the defined assessment scope.

---

## Credential-to-Privilege Mapping

Finding credentials is only the beginning.

Every credential should be mapped to its corresponding account and privileges.

```text
Credential
    │
    ▼
Account
    │
    ├── Group Membership
    │
    ├── Local Privileges
    │
    ├── Domain Privileges
    │
    ├── ACL Permissions
    │
    └── Service Access
            │
            ▼
       Target Resource
```

For example:

```text
Credential
    │
    ▼
svc-web
    │
    ├── Member of: WebAdmins
    │
    ├── SPN: HTTP/web01.example.local
    │
    └── Access: Application Server
```

This provides much more useful information than simply recording:

```text
Password discovered
```

---

## Authentication Path Analysis

Discovered authentication material should be correlated with the enumeration performed in Section 03.

```text
Credentials
     │
     ▼
Users / Service Accounts
     │
     ├── Groups
     ├── ACLs
     ├── SPNs
     ├── Computers
     └── Services
             │
             ▼
      Potential Path
             │
             ▼
      Privileged Resource
```

This creates a bridge between credential discovery and privilege escalation.

---

## Account Context

For every discovered credential or authentication weakness, establish the account context.

| Question                   | Example                    |
| -------------------------- | -------------------------- |
| Which account?             | `svc-web`                  |
| Account type?              | Service account            |
| Domain?                    | `example.local`            |
| Groups?                    | `WebAdmins`                |
| SPNs?                      | `HTTP/web01.example.local` |
| Authentication method?     | Kerberos                   |
| Accessible systems?        | Web server                 |
| Administrative privileges? | To be validated            |
| Potential path?            | To be investigated         |

This prevents isolated findings from being mistaken for complete attack paths.

---

## Credential Validation Workflow

Use a controlled validation process:

```text
Credential Identified
        │
        ▼
Identify Account
        │
        ▼
Verify Account Status
        │
        ▼
Verify Credential Validity
        │
        ▼
Determine Authentication Scope
        │
        ▼
Determine Privileges
        │
        ▼
Identify Accessible Resources
        │
        ▼
Document Evidence
```

Do not unnecessarily reuse discovered credentials against unrelated systems.

Validation should remain within authorized scope.

---

## Correlation With Previous Enumeration

Section 03 provides the environment map.

Section 04 adds authentication information to that map.

```text
03-Enumeration
      │
      ├── Users
      ├── Groups
      ├── Computers
      ├── Shares
      ├── GPOs
      ├── ACLs
      ├── SPNs
      ├── Trusts
      └── Delegation
              │
              ▼
04-Credentials-and-Authentication
              │
              ├── Credentials
              ├── Password Reuse
              ├── Kerberos Tickets
              ├── Authentication Weaknesses
              └── Credential Material
                      │
                      ▼
              Privilege Relationships
```

The combined data can then be used to identify potential escalation paths.

---

## Assessment Mindset

When investigating credentials and authentication, ask:

### Credential

* What authentication material was discovered?
* Where was it found?
* What type of credential is it?
* Is it current?

### Account

* Which account does it belong to?
* Is the account enabled?
* What groups is it a member of?
* Does it have administrative privileges?

### Authentication

* Where can the account authenticate?
* Which authentication protocol is involved?
* Is Kerberos involved?
* Are there configuration weaknesses?

### Access

* What systems can the account access?
* What services can it access?
* Does it have local or domain-level privileges?

### Attack Path

* Does the credential connect to an existing path?
* Does it provide access to a higher-privileged account?
* Does it create a new path toward a privileged resource?

---

## Evidence Collection

For every significant authentication finding, record:

```text
Finding:
Account:
Credential / Authentication Type:
Source:
Domain:
Groups:
SPNs:
Accessible Systems:
Observed Privileges:
Validation Performed:
Evidence:
Potential Impact:
```

Avoid storing real passwords or sensitive credential material in a public repository.

For lab documentation, use sanitized examples.

---

## Section Checklist

### Credential Discovery

* [ ] Reviewed authorized credential sources
* [ ] Identified exposed authentication material
* [ ] Identified associated accounts
* [ ] Validated credential context

### Password Reuse

* [ ] Identified possible password reuse
* [ ] Validated within authorized scope
* [ ] Mapped reused credentials to accounts
* [ ] Checked resulting privileges

### Kerberoasting

* [ ] Enumerated SPNs
* [ ] Identified service accounts
* [ ] Recorded associated services
* [ ] Assessed account privileges
* [ ] Performed authorized validation where appropriate

### AS-REP Roasting

* [ ] Identified accounts without Kerberos pre-authentication
* [ ] Recorded account privileges
* [ ] Assessed account importance
* [ ] Performed authorized validation where appropriate

### Credential Dumping

* [ ] Identified authorized credential stores
* [ ] Recovered credential material where permitted
* [ ] Identified associated accounts
* [ ] Validated account context
* [ ] Protected sensitive evidence

### Correlation

* [ ] Correlated credentials with users
* [ ] Correlated credentials with groups
* [ ] Correlated credentials with computers
* [ ] Correlated credentials with SPNs
* [ ] Correlated credentials with ACLs
* [ ] Identified potential privilege paths
* [ ] Distinguished potential paths from confirmed findings

---

## Transition to Privilege Escalation Paths

Once credentials and authentication weaknesses have been analyzed, move from **credential discovery** to **privilege-path analysis**.

```text
04-Credentials-and-Authentication
              │
              ▼
      Credential / Access
              │
              ▼
      Account Privileges
              │
              ▼
05-Privilege-Escalation-Paths
              │
              ├── Group Membership Abuse
              ├── ACL Abuse
              ├── GPO Abuse
              ├── Delegation Abuse
              ├── RBCD
              ├── ADCS
              └── Trust Abuse
```

The next file is:

**`04-Credentials-and-Authentication/Credential-Discovery.md`**
