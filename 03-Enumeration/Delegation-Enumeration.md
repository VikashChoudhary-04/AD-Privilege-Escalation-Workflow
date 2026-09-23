# Delegation Enumeration

## Purpose

Delegation enumeration identifies Active Directory accounts and computers configured to perform Kerberos delegation.

The objective is to determine:

* Which computers or accounts have delegation configured
* What type of delegation is being used
* Which services are involved
* Which principals are trusted to delegate
* Which resources can be accessed through delegated authentication
* Whether delegation creates a potential privilege-escalation or lateral-movement path

Delegation configuration alone does **not** automatically mean that a system is vulnerable. Each finding should be correlated with account privileges, SPNs, ACLs, service configuration, and the target resource.

---

## Delegation Concepts

Kerberos delegation allows a service to authenticate to another service on behalf of a user.

A simplified relationship is:

```text
User
  │
  │ Authenticates to
  ▼
Service / Computer
  │
  │ Delegates authentication
  ▼
Target Service
```

Delegation can be useful for legitimate multi-tier applications.

For example:

```text
User
  │
  ▼
Web Server
  │
  │ Delegates authentication
  ▼
Database Server
```

During an assessment, the important question is:

```text
Who can delegate?
        │
        ▼
To which service?
        │
        ▼
On which computer?
        │
        ▼
With what privileges?
```

---

## Delegation Types

The main delegation configurations to identify are:

| Type                     | Main Indicator                             | Assessment Focus                                                 |
| ------------------------ | ------------------------------------------ | ---------------------------------------------------------------- |
| Unconstrained Delegation | `TrustedForDelegation` / UAC flag          | Which systems can receive delegated authentication               |
| Constrained Delegation   | `msDS-AllowedToDelegateTo`                 | Which services the account can delegate to                       |
| Protocol Transition      | `TrustedToAuthForDelegation`               | Whether non-Kerberos authentication can be converted to Kerberos |
| RBCD                     | `msDS-AllowedToActOnBehalfOfOtherIdentity` | Which principals can delegate to a target computer               |

These configurations should be analyzed separately because the delegation relationship is represented differently in Active Directory.

---

## Delegation Enumeration Workflow

Use the following workflow:

```text
Identify Delegation Attributes
        │
        ▼
Identify Unconstrained Delegation
        │
        ▼
Identify Constrained Delegation
        │
        ▼
Identify Protocol Transition
        │
        ▼
Identify RBCD
        │
        ▼
Identify Delegated Accounts
        │
        ▼
Identify Affected Services / Computers
        │
        ▼
Correlate SPNs and ACLs
        │
        ▼
Identify Potential Attack Paths
        │
        ▼
Record Findings
        │
        ▼
Move to Credential & Authentication Analysis
```

---

## 1. Identify Delegation-Related Attributes

Several Active Directory attributes are especially useful during delegation enumeration.

### `userAccountControl`

The `userAccountControl` attribute contains account flags.

Delegation-related flags include:

```text
TRUSTED_FOR_DELEGATION
TRUSTED_TO_AUTH_FOR_DELEGATION
```

These flags can indicate that an account or computer has specific delegation capabilities.

---

### `msDS-AllowedToDelegateTo`

This attribute is primarily associated with **constrained delegation**.

It identifies the service principal names to which a principal is allowed to delegate.

Conceptually:

```text
Delegating Account
        │
        │ msDS-AllowedToDelegateTo
        ▼
Allowed Service SPN
```

Example:

```text
WEB01$
   │
   └── MSSQLSvc/DB01.domain.local:1433
```

The presence of this configuration should be recorded and then correlated with the target service and account privileges.

---

### `msDS-AllowedToActOnBehalfOfOtherIdentity`

This attribute is associated with **Resource-Based Constrained Delegation (RBCD)**.

Unlike traditional constrained delegation, the configuration is stored on the **target resource**.

Conceptually:

```text
Target Computer
      │
      │ msDS-AllowedToActOnBehalfOfOtherIdentity
      ▼
Allowed Principal
```

This distinction is important when analyzing RBCD.

---

## 2. Enumerate Unconstrained Delegation

Unconstrained delegation allows a trusted service to perform authentication on behalf of users to other services.

A computer configured for unconstrained delegation should therefore receive particular attention.

### PowerShell

```powershell
Get-ADComputer -Filter * -Properties TrustedForDelegation |
    Where-Object {$_.TrustedForDelegation -eq $true} |
    Select-Object Name, DNSHostName, TrustedForDelegation
```

For user accounts:

```powershell
Get-ADUser -Filter * -Properties TrustedForDelegation |
    Where-Object {$_.TrustedForDelegation -eq $true} |
    Select-Object SamAccountName, TrustedForDelegation
```

### What to Record

For every result, record:

```text
Computer / Account
DNS Name
Delegation Configuration
SPNs
Administrative Privileges
Role / Function
```

Example:

```text
Computer: APP01
DNS: app01.example.local
Delegation: Unconstrained
SPNs: HTTP/app01.example.local
Role: Application Server
```

### Assessment Questions

Ask:

* Is the system expected to use delegation?
* Is it a high-value server?
* Which services are running?
* Which accounts can authenticate to it?
* Does the account have administrative privileges?
* Are there additional controls restricting the configuration?

The goal is to understand the security impact rather than treating the delegation flag itself as a vulnerability.

---

## 3. Enumerate Constrained Delegation

Constrained delegation restricts delegation to specific services.

The primary attribute to examine is:

```text
msDS-AllowedToDelegateTo
```

### PowerShell

```powershell
Get-ADComputer -Filter * -Properties msDS-AllowedToDelegateTo |
    Where-Object {$_.'msDS-AllowedToDelegateTo'} |
    Select-Object Name, DNSHostName, msDS-AllowedToDelegateTo
```

User accounts should also be checked:

```powershell
Get-ADUser -Filter * -Properties msDS-AllowedToDelegateTo |
    Where-Object {$_.'msDS-AllowedToDelegateTo'} |
    Select-Object SamAccountName, msDS-AllowedToDelegateTo
```

### Example Relationship

```text
WEB01$
   │
   │ AllowedToDelegateTo
   ▼
MSSQLSvc/DB01.example.local:1433
```

This tells us that the delegation configuration references the SQL service on `DB01`.

It does **not** by itself establish that an attack is possible.

Further analysis should determine:

```text
Who controls WEB01?
        │
        ▼
What service account is involved?
        │
        ▼
What service is targeted?
        │
        ▼
What privileges exist on DB01?
```

---

## 4. Identify Protocol Transition

Protocol transition allows a service configured for constrained delegation to obtain Kerberos service tickets on behalf of users even when the original authentication did not use Kerberos.

The relevant configuration is commonly represented by:

```text
TrustedToAuthForDelegation
```

### PowerShell

```powershell
Get-ADComputer -Filter * -Properties TrustedToAuthForDelegation |
    Where-Object {$_.TrustedToAuthForDelegation -eq $true} |
    Select-Object Name, DNSHostName, TrustedToAuthForDelegation
```

User accounts can also be examined:

```powershell
Get-ADUser -Filter * -Properties TrustedToAuthForDelegation |
    Where-Object {$_.TrustedToAuthForDelegation -eq $true} |
    Select-Object SamAccountName, TrustedToAuthForDelegation
```

### Correlate With Constrained Delegation

Protocol transition is most useful to analyze together with:

```text
TrustedToAuthForDelegation
        +
msDS-AllowedToDelegateTo
```

The resulting relationship can be represented as:

```text
Delegating Principal
        │
        ├── Protocol Transition
        │
        └── Allowed Services
                │
                ▼
          Target Service
```

This provides a more complete picture than checking either attribute independently.

---

## 5. Enumerate Resource-Based Constrained Delegation

RBCD changes the perspective of delegation analysis.

Instead of asking:

```text
What can this account delegate to?
```

ask:

```text
Which principals are allowed to delegate to this resource?
```

The main attribute is:

```text
msDS-AllowedToActOnBehalfOfOtherIdentity
```

### PowerShell

A broad search can be performed with:

```powershell
Get-ADObject -LDAPFilter "(msDS-AllowedToActOnBehalfOfOtherIdentity=*)" `
    -Properties msDS-AllowedToActOnBehalfOfOtherIdentity |
    Select-Object DistinguishedName, msDS-AllowedToActOnBehalfOfOtherIdentity
```

Computer objects should then be investigated individually.

### Important Direction

For RBCD:

```text
Target Computer
      │
      │ msDS-AllowedToActOnBehalfOfOtherIdentity
      ▼
Principal Allowed to Act on Behalf of Users
```

For example:

```text
SERVER02
   │
   │ RBCD configuration
   ▼
APP01$
```

This means the delegation relationship is configured **on SERVER02**, while `APP01$` is the principal represented in the configuration.

Do not confuse this with:

```text
msDS-AllowedToDelegateTo
```

which identifies the services a delegating principal is configured to access.

---

## 6. Enumerate Delegation With PowerView

PowerView can provide another way to identify delegation configurations during an authorized assessment.

Examples include:

```powershell
Get-DomainComputer -Unconstrained
```

and:

```powershell
Get-DomainComputer -TrustedToAuth
```

These results can then be correlated with:

```powershell
Get-DomainComputer
Get-DomainUser
Get-DomainObject
```

For RBCD-related investigation, search for objects containing:

```text
msDS-AllowedToActOnBehalfOfOtherIdentity
```

The exact command and available properties can vary between PowerView versions, so verify the output against the underlying LDAP attributes rather than assuming a missing field means that no configuration exists.

---

## 7. Correlate Delegation With SPNs

Delegation analysis should be combined with SPN enumeration.

Useful attributes include:

```text
servicePrincipalName
msDS-AllowedToDelegateTo
```

A useful relationship map is:

```text
Account
  │
  ├── SPNs
  │
  ├── Delegation Configuration
  │
  └── Group Membership
          │
          ▼
     Target Service
          │
          ▼
       Computer
```

Questions to answer:

* Which services are associated with the delegating account?
* Which computers host those services?
* Are those computers high-value assets?
* Does the account have administrative privileges?
* Are multiple delegation configurations connected to the same system?

---

## 8. Correlate Delegation With ACLs

Delegation findings should also be correlated with permissions.

For example:

```text
Principal
   │
   ├── Group Membership
   │
   ├── ACL Permissions
   │
   └── Delegation Configuration
            │
            ▼
       Target Computer
```

Useful permissions to investigate include control over:

* Computer objects
* Service accounts
* Group membership
* GPOs
* Delegation-related attributes
* High-value servers

A delegation configuration becomes significantly more important when combined with excessive control over the relevant account or computer object.

---

## 9. Identify High-Value Delegation Relationships

Not every delegation configuration has the same security significance.

Prioritize relationships involving:

* Domain controllers
* Authentication infrastructure
* Administrative servers
* Database servers
* File servers
* High-privilege service accounts
* Accounts with significant group memberships
* Systems that interact with sensitive services

Record the relationship rather than immediately labeling it exploitable.

Example:

```text
Delegating Principal:
    WEB01$

Delegation Type:
    Constrained Delegation

Allowed Service:
    MSSQLSvc/DB01.example.local:1433

Target:
    DB01

Observed Privileges:
    WEB01$ → Web application server
    DB01 → Database server

Next Question:
    Does control of WEB01$ provide a meaningful path
    toward the delegated service?
```

---

## 10. Build the Delegation Map

A useful assessment artifact is a delegation relationship table.

| Delegating Principal | Delegation Type     | Configuration                              | Target            | Relevant Privileges |
| -------------------- | ------------------- | ------------------------------------------ | ----------------- | ------------------- |
| WEB01$               | Unconstrained       | `TrustedForDelegation`                     | Multiple services | Investigate         |
| APP01$               | Constrained         | `msDS-AllowedToDelegateTo`                 | DB01 SQL service  | Investigate         |
| APP02$               | Protocol Transition | `TrustedToAuthForDelegation`               | Defined SPNs      | Investigate         |
| SERVER02             | RBCD                | `msDS-AllowedToActOnBehalfOfOtherIdentity` | SERVER02          | Investigate         |

The exact entries should come from the environment being assessed.

---

## 11. Delegation Attack-Path Model

Once delegation configurations are identified, represent them as relationships:

```text
Initial Access / Controlled Account
            │
            ▼
   Delegating Principal
            │
            │ Delegation
            ▼
      Target Service
            │
            ▼
      Target Computer
            │
            ▼
     Privileged Resource
```

For RBCD:

```text
Controlled Principal
        │
        ▼
RBCD Permission
        │
        ▼
Target Computer
        │
        ▼
Target Service
```

Then correlate the relationship with:

```text
ACLs
+
Group Membership
+
SPNs
+
Service Accounts
+
Computer Roles
```

This turns raw enumeration data into an attack-path hypothesis.

---

## 12. Validate the Finding

A delegation configuration should pass through several validation stages:

```text
Delegation Exists
      │
      ▼
Configuration Identified
      │
      ▼
Affected Principal Identified
      │
      ▼
Target Service Identified
      │
      ▼
Privileges Correlated
      │
      ▼
Potential Path Identified
      │
      ▼
Authorized Validation
```

Avoid conclusions such as:

```text
"Delegation exists → Domain Admin"
```

Instead document the actual relationship:

```text
Principal A
    ↓
Delegation Configuration
    ↓
Service B
    ↓
Computer C
    ↓
Observed Privilege
```

Only move from a potential path to a confirmed finding after appropriate authorized validation.

---

## Assessment Mindset

When enumerating delegation, ask:

### Configuration

* What type of delegation is configured?
* Which AD attribute or flag represents it?
* Is the configuration expected?

### Principal

* Which user or computer is involved?
* What groups is it a member of?
* What permissions does it have?

### Service

* Which SPNs are associated with it?
* Which service is the delegation targeting?
* Which computer hosts the service?

### Target

* Is the target a high-value system?
* What privileges exist on the target?
* Does the relationship create a meaningful path?

### Validation

* Is the configuration actually usable?
* Are additional conditions required?
* Has the relationship been validated within authorized scope?

---

## Delegation Enumeration Checklist

### Delegation Attributes

* [ ] Checked `userAccountControl`
* [ ] Checked `TrustedForDelegation`
* [ ] Checked `TrustedToAuthForDelegation`
* [ ] Checked `msDS-AllowedToDelegateTo`
* [ ] Checked `msDS-AllowedToActOnBehalfOfOtherIdentity`

### Unconstrained Delegation

* [ ] Identified delegated computers
* [ ] Identified delegated user accounts
* [ ] Recorded SPNs
* [ ] Identified system roles
* [ ] Identified associated privileges

### Constrained Delegation

* [ ] Identified delegating principals
* [ ] Enumerated allowed SPNs
* [ ] Identified target services
* [ ] Identified target computers
* [ ] Correlated account privileges

### Protocol Transition

* [ ] Identified principals with protocol transition
* [ ] Correlated with constrained delegation
* [ ] Identified target services
* [ ] Recorded relevant SPNs

### RBCD

* [ ] Searched for `msDS-AllowedToActOnBehalfOfOtherIdentity`
* [ ] Identified target computers
* [ ] Identified principals allowed to act on behalf of users
* [ ] Checked permissions over the affected computer objects
* [ ] Correlated with target services

### Attack-Path Analysis

* [ ] Correlated delegation with ACLs
* [ ] Correlated delegation with group membership
* [ ] Correlated delegation with SPNs
* [ ] Identified high-value targets
* [ ] Distinguished configuration from exploitability
* [ ] Documented potential paths
* [ ] Validated findings within authorized scope

---

## Transition to Credential & Authentication Analysis

After completing delegation enumeration, the assessment should move from **relationship discovery** to **credential and authentication analysis**.

```text
03-Enumeration
      │
      └── Delegation Enumeration
              │
              ▼
04-Credentials-and-Authentication
              │
              ├── Credential Discovery
              ├── Password Reuse
              ├── Kerberoasting
              ├── AS-REP Roasting
              └── Credential Dumping
```

The next stage is:

**`04-Credentials-and-Authentication/README.md`**

This section will examine how authentication mechanisms, credentials, service accounts, and credential exposure can create paths toward higher privileges.
