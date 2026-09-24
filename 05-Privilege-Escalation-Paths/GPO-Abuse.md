# GPO Abuse

## Purpose

Group Policy Objects (GPOs) are a major security control in Active Directory. When an account or group has excessive permissions over a GPO, that control can potentially be used to modify policies applied to users or computers.

This file focuses on identifying, analyzing, validating, and documenting **GPO-based privilege escalation paths** during an authorized security assessment.

The core workflow is:

```text
Identify GPO
      ↓
Enumerate GPO Permissions
      ↓
Identify Controlling Principal
      ↓
Determine GPO Scope
      ↓
Identify Linked Users/Computers
      ↓
Determine Effective Impact
      ↓
Validate Authorized Path
      ↓
Document Evidence
```

---

## Position in the Workflow

GPO abuse belongs in the **Privilege Escalation Paths** phase.

The previous phase identified credentials and authentication weaknesses. ACL analysis then identified principals that control AD objects.

GPO abuse extends that analysis by asking:

> Can a principal who controls a GPO influence a user or computer with greater privileges?

A GPO permission alone does not automatically represent privilege escalation. The GPO's **scope, links, inheritance, security filtering, and affected systems** must also be evaluated.

---

## GPO Fundamentals

A Group Policy Object contains policy settings that can be applied to:

* Users
* Computers
* Security settings
* Administrative settings
* Scripts
* Software configuration
* System configuration
* Windows components

A GPO generally has two important components:

```text
GPO
├── Group Policy Container (GPC)
│   └── Active Directory object
│
└── Group Policy Template (GPT)
    └── SYSVOL contents
```

The GPC is stored in Active Directory, while the GPT is stored in the domain's SYSVOL structure.

Both components should be considered when assessing GPO security.

---

## GPO Abuse Model

A typical GPO-based privilege escalation path can be represented as:

```text
Low-Privilege Principal
        ↓
GPO Permission
        ↓
GPO Modification
        ↓
GPO Linked to Target
        ↓
Policy Applied to Target
        ↓
Security-Relevant Effect
        ↓
Higher Privilege / Access
```

The important question is not simply:

> "Can this user modify the GPO?"

Instead ask:

> "What systems or users are affected by this GPO, and what security-relevant changes can the user make?"

---

## 1. Enumerate GPOs

Start by identifying available GPOs.

PowerShell:

```powershell
Get-GPO -All
```

Useful information includes:

* GPO name
* Display name
* GPO ID
* Creation time
* Modification time
* Domain
* Status

For a specific GPO:

```powershell
Get-GPO -Name "Example Policy"
```

---

## 2. Enumerate GPO Permissions

GPO permissions determine who can read, modify, or control the object.

Example:

```powershell
Get-GPPermission -Name "Example Policy" -All
```

This helps identify principals associated with the GPO and their permission levels.

Pay particular attention to:

* Domain Admins
* Enterprise Admins
* SYSTEM
* Administrators
* Domain users
* Custom administrative groups
* Individual user accounts
* Service accounts
* Unexpected groups

---

## 3. Identify Modification Rights

Not every GPO permission provides the same level of control.

Determine whether the principal can:

* Read the GPO
* Edit the GPO
* Modify security filtering
* Modify related permissions
* Modify policy settings
* Modify the underlying GPO files
* Modify the GPO's AD security descriptor

The exact effective permission should always be verified rather than inferred from a group or ACE name alone.

---

## 4. GPO Permission Types

When analyzing a GPO, distinguish between:

### Read

Allows the principal to read the GPO.

Read access alone generally does not provide the ability to modify policy.

### Edit

Allows modification of policy settings.

This can become security-sensitive when the GPO applies to privileged systems or users.

### Edit Settings, Delete, Modify Security

Broader administrative permissions can provide additional control over the GPO.

These permissions should be analyzed together with:

* GPO links
* Security filtering
* OU structure
* Target computers
* Target users
* Group membership

---

## 5. Determine the GPO Scope

A GPO becomes significantly more important when it is linked to sensitive parts of the domain.

Find where the GPO is linked.

Useful command:

```powershell
Get-GPInheritance -Target "OU=Servers,DC=example,DC=local"
```

For domain-level policy:

```powershell
Get-GPInheritance -Target "DC=example,DC=local"
```

The exact distinguished name depends on the environment.

---

## 6. GPO Links

A GPO can be linked to:

* Domain
* Site
* OU

For example:

```text
Domain
└── Servers OU
    ├── Web Servers
    ├── Database Servers
    └── Management Servers
```

If a GPO is linked to the `Servers` OU, its effective policy may affect multiple systems within that scope.

Therefore:

```text
GPO Permission
+
GPO Link
+
Target Object
=
Potential Attack Path
```

---

## 7. Identify Affected Computers

After identifying a GPO, determine which computers may receive it.

Consider:

* OU placement
* Nested OUs
* Security filtering
* WMI filters
* Block inheritance
* Enforced links
* Group membership

A GPO linked to a low-impact workstation is different from the same GPO linked to a sensitive server.

---

## 8. Identify Affected Users

GPOs can also contain user-side policies.

Determine:

* Which users are in the target OU
* Which security groups receive the policy
* Whether security filtering limits application
* Whether privileged users are affected

Example:

```text
GPO
 ↓
Finance OU
 ↓
Finance Users
```

Compare this with:

```text
GPO
 ↓
Domain-level Link
 ↓
Broad User Population
```

The effective scope matters.

---

## 9. Security Filtering

Security filtering controls which security principals can receive a GPO.

Common principals include:

* Authenticated Users
* Domain Computers
* Domain Users
* Custom groups
* Administrative groups

Do not assume that a GPO linked to an OU affects every object in that OU.

Check security filtering before determining impact.

---

## 10. WMI Filters

A GPO may use a WMI filter to further restrict application.

For example, a policy could be limited to systems matching certain characteristics.

Therefore:

```text
GPO Link
      ↓
Security Filtering
      ↓
WMI Filtering
      ↓
Effective Target
```

All relevant conditions should be considered when determining whether a GPO reaches a target system.

---

## 11. Computer-Side Policy

Computer-side policies can affect the security configuration of machines.

Potentially relevant settings include:

* Local group membership
* Security policies
* Windows services
* Scheduled tasks
* Startup scripts
* Firewall configuration
* Registry settings
* User rights assignments
* Software configuration

The security impact depends on exactly what the GPO can modify and which systems receive it.

---

## 12. User-Side Policy

User-side policies can affect:

* Logon scripts
* User configuration
* Environment settings
* Application configuration
* Registry settings
* Desktop configuration
* Group membership-related mechanisms

As with computer policies, determine the actual target population before assessing impact.

---

## 13. GPO and Local Group Membership

One important area is local group configuration.

A GPO may control membership of local groups on computers.

For example:

```text
GPO
 ↓
Computer OU
 ↓
Local Administrators Configuration
 ↓
Target Computer
```

If a low-privileged principal can legitimately modify the applicable GPO, the resulting policy change may affect local administrative access.

The assessment should verify:

1. Who can modify the GPO?
2. Which computers receive it?
3. Which local group is affected?
4. Which account or group would gain membership?
5. What privilege would that membership provide?

---

## 14. GPO and Scripts

GPOs may distribute:

* Startup scripts
* Shutdown scripts
* Logon scripts
* Logoff scripts

When assessing a GPO, determine whether the modifying principal can change scripts that execute on privileged systems.

The important relationship is:

```text
GPO Control
      ↓
Script Configuration
      ↓
Target System
      ↓
Script Execution Context
```

Execution context and target scope must be validated before concluding that privilege escalation exists.

---

## 15. GPO and Scheduled Tasks

Group Policy Preferences and related policy mechanisms may configure scheduled tasks.

Assess:

* Task creation
* Task modification
* Execution account
* Trigger
* Target computers
* Stored configuration
* Privilege level

A scheduled task running under a privileged context can be security-relevant if an unauthorized principal can control its configuration.

---

## 16. GPO and Registry Settings

GPOs can configure registry values across affected systems.

During assessment, identify:

* Registry path
* Value
* Data
* Target computers
* Execution context
* Security impact

Do not treat every registry modification as privilege escalation.

The security impact depends on what the specific setting controls.

---

## 17. GPO and User Rights Assignment

Security policies can define Windows user rights.

Examples include rights related to:

* Local logon
* Remote logon
* Service execution
* Batch jobs
* Scheduled tasks
* System shutdown
* Backup and restore

When such settings are modified, determine:

```text
Policy
 ↓
Assigned Principal
 ↓
Target System
 ↓
Effective User Right
 ↓
Security Impact
```

---

## 18. GPO Permissions vs GPO Links

These are separate concepts.

A principal may have permission to modify a GPO but the GPO may not be linked to a useful target.

Conversely, a GPO may be linked to highly privileged systems but the assessed principal may only have read access.

Therefore:

```text
GPO Control ≠ Automatic Privilege Escalation
```

The complete path must be established.

---

## 19. GPO Inheritance

GPO processing can involve inheritance.

Consider:

```text
Domain
   ↓
Parent OU
   ↓
Child OU
   ↓
Computer
```

A policy may originate at a higher level and apply to objects below it.

Check for:

* Inheritance
* Block Inheritance
* Enforced links
* Link order
* Conflicting policies

Effective policy should be determined from the actual processing hierarchy.

---

## 20. GPO Precedence

Multiple GPOs may apply to the same target.

Therefore, a GPO modification should be evaluated against:

* Other linked GPOs
* Link order
* Enforced settings
* Conflicting settings
* Security filtering
* WMI filtering

A configuration that appears exploitable in isolation may not produce the expected effective configuration.

---

## 21. GPO and SYSVOL

The Group Policy Template is stored within SYSVOL.

A typical domain structure includes:

```text
\\example.local\SYSVOL\
```

GPO data is associated with a GUID.

During an assessment, inspect only the information necessary to establish:

* GPO existence
* Policy configuration
* Script references
* File permissions
* Modification rights

Do not unnecessarily alter shared policy files.

---

## 22. GPO and ACL Abuse

GPO abuse frequently overlaps with ACL abuse.

The chain may look like:

```text
User
 ↓
GPO ACL
 ↓
GPO Modification
 ↓
Linked OU
 ↓
Privileged Computer
 ↓
Security-Relevant Policy
 ↓
Privilege
```

Therefore, ACL enumeration should be correlated with GPO enumeration.

---

## 23. GPO and OU Relationships

OU permissions can also affect GPO-based attack paths.

Consider:

```text
User
 ↓
OU Control
 ↓
GPO Link / Object Placement
 ↓
Target Computers
```

An OU permission does not automatically provide GPO modification rights, so each permission must be evaluated independently.

---

## 24. BloodHound-Compatible Analysis

BloodHound-compatible collectors can help visualize relationships between:

* Users
* Groups
* Computers
* OUs
* GPOs
* ACLs

Depending on collector and version, relationships may expose GPO control or related object-control paths.

Use graph analysis to answer:

```text
Who controls the GPO?
        ↓
Where is it linked?
        ↓
Who receives it?
        ↓
What security-relevant setting can be changed?
        ↓
What privilege follows?
```

The graph should support manual validation rather than replace it.

---

## 25. Attack Path Construction

A GPO-based path should be represented as a sequence of verified relationships.

Example:

```text
User A
  ↓
Can Modify GPO
  ↓
GPO-Servers
  ↓
Linked to Server OU
  ↓
Server01
  ↓
Policy Applies
  ↓
Security-Relevant Configuration
  ↓
Higher Access
```

Every arrow should be supported by enumeration or validation evidence.

---

## 26. Direct vs Chained GPO Paths

### Direct Path

```text
User
 ↓
GPO Control
 ↓
Privileged Target
```

### Chained Path

```text
User
 ↓
GPO Control
 ↓
Target Computer
 ↓
Local Privilege
 ↓
Credential / Access Discovery
 ↓
Additional AD Privilege
```

Chained paths should be documented step by step rather than treating the final privilege as directly granted by the GPO.

---

## 27. Validate the GPO Path

Use a structured validation process:

```text
GPO Identified
      ↓
Permission Confirmed
      ↓
Principal Confirmed
      ↓
GPO Link Confirmed
      ↓
Security Filtering Confirmed
      ↓
Target Confirmed
      ↓
Policy Setting Confirmed
      ↓
Effective Result Confirmed
```

Only after the complete chain is established should the finding be classified as a confirmed privilege escalation path.

---

## 28. Evidence Collection

For every GPO-related finding, record:

| Field      | Description                                  |
| ---------- | -------------------------------------------- |
| GPO        | Name and GUID                                |
| Principal  | User/group controlling the GPO               |
| Permission | Exact permission                             |
| Link       | Domain/site/OU link                          |
| Scope      | Affected users/computers                     |
| Filtering  | Security/WMI filtering                       |
| Policy     | Relevant configuration                       |
| Target     | Affected system or user                      |
| Result     | Observed security impact                     |
| Validation | How the path was confirmed                   |
| Evidence   | Commands, output, screenshots, or graph data |

Avoid collecting unnecessary sensitive information.

---

## 29. Common False Positives

### Read-Only Access

A principal may be able to read a GPO without modifying it.

### GPO Not Linked

A principal may control a GPO that is not linked to a relevant target.

### Security Filtering

The GPO may not actually apply to the expected target.

### WMI Filtering

A WMI filter may prevent the policy from applying.

### Policy Precedence

Another GPO may override the relevant setting.

### Low-Impact Target

The GPO may affect only systems without meaningful privilege implications.

### Disabled Objects

A target user or computer may be disabled or otherwise outside the practical attack path.

---

## 30. Common Mistakes

Avoid:

* Treating GPO read access as modification access
* Assuming every linked GPO affects every object
* Ignoring security filtering
* Ignoring WMI filters
* Ignoring inheritance
* Ignoring GPO precedence
* Assuming GPO modification automatically means Domain Admin
* Failing to identify the execution context of scripts or tasks
* Ignoring the target computer's actual privilege model
* Modifying production policies without authorization

---

## 31. Assessment Checklist

### GPO Enumeration

* [ ] Enumerate GPOs
* [ ] Record GPO names and GUIDs
* [ ] Identify linked OUs
* [ ] Identify domain/site links
* [ ] Review inheritance

### Permission Analysis

* [ ] Identify GPO owners
* [ ] Enumerate GPO permissions
* [ ] Identify modification rights
* [ ] Review ACLs
* [ ] Check custom groups
* [ ] Check unexpected principals

### Scope Analysis

* [ ] Identify target OUs
* [ ] Identify target computers
* [ ] Identify target users
* [ ] Review security filtering
* [ ] Review WMI filtering
* [ ] Review GPO precedence

### Security Impact

* [ ] Review computer-side settings
* [ ] Review user-side settings
* [ ] Review local group configuration
* [ ] Review scripts
* [ ] Review scheduled tasks
* [ ] Review registry settings
* [ ] Review user rights
* [ ] Determine effective result

### Validation

* [ ] Confirm principal
* [ ] Confirm GPO permission
* [ ] Confirm GPO link
* [ ] Confirm effective scope
* [ ] Confirm policy setting
* [ ] Confirm security impact
* [ ] Collect evidence

---

## Transition

Once GPO control and its effective security impact have been established, continue to:

**`05-Privilege-Escalation-Paths/Delegation-Abuse.md`**

This moves the workflow from policy-based privilege paths to **delegation-based privilege paths**, including how delegated Kerberos permissions can affect access to other AD resources.
