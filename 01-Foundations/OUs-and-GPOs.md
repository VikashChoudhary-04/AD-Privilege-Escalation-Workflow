# OUs and GPOs

Organizational Units (OUs) and Group Policy Objects (GPOs) are important components of Active Directory administration and security.

OUs provide a way to organize directory objects, while GPOs provide centralized configuration and policy management for users and computers.

From a security-assessment perspective, the important question is not simply:

> "What policies exist?"

It is:

> **"Which policies apply to which objects, who can modify them, and what security impact could those permissions have?"**

## Organizational Units

An Organizational Unit (OU) is a container within Active Directory used to organize objects.

OUs can contain objects such as:

* Users
* Computers
* Groups
* Other OUs

For example:

```text id="a3k9pw"
corp.example.com
│
├── OU=Users
│
├── OU=Workstations
│
├── OU=Servers
│
└── OU=Service-Accounts
```

OUs provide logical organization within a domain.

## Why OUs Matter

OUs are particularly useful for applying Group Policy to specific sets of users or computers.

For example:

```text id="9e2v6s"
OU=Workstations
       │
       ▼
   GPO Applied
       │
       ▼
Workstation Configuration
```

Another OU can have a different policy:

```text id="a0r1d5"
OU=Servers
       │
       ▼
   GPO Applied
       │
       ▼
Server Configuration
```

This allows administrators to apply different configurations to different organizational groups.

## OUs vs Groups

OUs and security groups serve different purposes.

| Component      | Primary Purpose                                                      |
| -------------- | -------------------------------------------------------------------- |
| OU             | Organize objects and provide a scope for management and Group Policy |
| Security Group | Group security principals for permissions and access control         |

For example:

```text id="x6s0bd"
OU
 │
 ├── User A
 ├── User B
 └── User C
```

while:

```text id="7bqj5a"
Security Group
 │
 ├── User A
 ├── User B
 └── User C
```

The OU is primarily an organizational and administrative container. The group is a security principal that can be assigned permissions.

## Group Policy

Group Policy provides centralized management of configuration settings for users and computers.

Policies can control areas such as:

* Security settings
* User configuration
* Computer configuration
* System behavior
* Authentication-related settings
* Software configuration
* Administrative restrictions
* Windows security controls

A simplified model is:

```text id="c1xq57"
Group Policy Object
        │
        ▼
Policy Settings
        │
        ▼
Users / Computers
```

## Group Policy Object

A Group Policy Object (GPO) is a collection of policy settings that can be applied to users and computers.

A GPO contains two major configuration areas:

```text id="qtxh1j"
GPO
│
├── Computer Configuration
│
└── User Configuration
```

### Computer Configuration

Computer Configuration contains settings that apply to computers.

Examples include:

* Security configuration
* Windows settings
* Administrative templates
* Startup scripts
* System-related policies

### User Configuration

User Configuration contains settings that apply to users.

Examples include:

* User settings
* Administrative templates
* Logon scripts
* Desktop configuration
* User-specific restrictions

## GPO Storage

Group Policy information is stored in both Active Directory and the domain's `SYSVOL` structure.

Conceptually:

```text id="8qtrc7"
              GPO
               │
       ┌───────┴───────┐
       │               │
       ▼               ▼
Active Directory     SYSVOL
       │               │
       └───────┬───────┘
               ▼
        Policy Information
```

This distinction is important because GPO information is not represented entirely in one location.

## GPO Linking

A GPO does not automatically apply to every object in a domain.

It is commonly linked to an AD site, domain, or OU.

For example:

```text id="6q4e4p"
Domain
   │
   └── OU=Workstations
           │
           └── GPO-Workstation-Security
```

The link determines the scope in which the policy can apply.

## Scope of a GPO

A simplified hierarchy is:

```text id="x8qz1k"
Site
 │
 ▼
Domain
 │
 ▼
OU
 │
 ▼
Child OU
```

GPOs can be linked at different levels of this hierarchy.

Multiple GPOs can therefore affect the same user or computer.

## Group Policy Processing

When multiple policies apply to an object, Windows processes them according to Group Policy processing rules.

A simplified representation is:

```text id="k5s6pu"
Site
 ↓
Domain
 ↓
Parent OU
 ↓
Child OU
 ↓
Computer / User
```

The actual result depends on factors such as:

* GPO links
* Link order
* Inheritance
* Enforcement
* Block inheritance
* Security filtering
* WMI filtering
* Other Group Policy processing rules

Therefore, simply finding a GPO does not tell you its complete security impact.

## Group Policy Inheritance

GPOs can be inherited through the AD hierarchy.

For example:

```text id="8tw8bq"
Domain
 │
 └── GPO-A
      │
      ▼
    OU=Servers
      │
      └── Child OU
```

Unless inheritance is modified, settings can flow down the organizational hierarchy.

This makes understanding OU structure important during policy analysis.

## Security Filtering

GPOs can use security filtering to determine which users or computers receive the policy.

Conceptually:

```text id="k8y0ac"
GPO
 │
 ▼
Security Filtering
 │
 ├── User A ✓
 ├── User B ✗
 └── Computer C ✓
```

Therefore, a GPO being linked to an OU does not necessarily mean every object within that OU receives every setting from the GPO.

## WMI Filtering

WMI filters can further control whether a GPO applies to a particular computer.

Conceptually:

```text id="8q5pqa"
GPO
 │
 ▼
WMI Filter
 │
 ├── Condition Met ──► Apply
 │
 └── Condition Not Met ──► Do Not Apply
```

This can make policy analysis more complex in larger environments.

## GPO Permissions

GPOs themselves have permissions.

Different principals may have permissions such as:

* Read
* Write
* Modify
* Link-related permissions
* Other administrative permissions

The exact effective control depends on the permissions assigned to the GPO and its related objects.

From a security perspective, an important question is:

```text id="p5f8pp"
Who can modify this GPO?
```

If a lower-privileged principal can make security-relevant changes to a GPO that applies to more privileged systems or users, the relationship may warrant further investigation.

## GPO Abuse Concept

Consider:

```text id="q2c2kd"
Low-Privilege User
        │
        │ Can Modify
        ▼
       GPO
        │
        │ Applies To
        ▼
Privileged Computer / User
```

This creates a potential privilege-escalation path.

The actual security impact depends on:

* What permissions the user has
* What the GPO controls
* Which objects receive the GPO
* Whether the relevant policy settings can affect privilege
* Whether the resulting path can be safely validated

The repository's later **GPO Abuse** section will cover this in detail.

## GPO and Privileged Systems

A GPO may apply to:

* Workstations
* Servers
* Domain Controllers
* Users
* Administrative systems

Therefore, the same type of GPO permission can have very different consequences depending on its scope.

For example:

```text id="b5a4l3"
GPO
 │
 ├── Workstations
 │
 ├── Servers
 │
 └── Domain Controllers
```

A policy affecting a Domain Controller deserves particular attention because of the privileged role of that system.

## Group Policy and Logon Scripts

GPOs can distribute scripts such as:

* Startup scripts
* Shutdown scripts
* Logon scripts
* Logoff scripts

These scripts may interact with:

* User accounts
* Computer accounts
* File shares
* Administrative processes
* Other domain resources

During an assessment, scripts can therefore become a source of configuration and security information.

## GPO and Credentials

Historically, some Group Policy configurations could expose sensitive information.

A well-known example involved storing certain password-related information in Group Policy Preferences.

Modern environments should not rely on this insecure mechanism, but legacy configurations may still exist.

The important assessment principle is:

> **Configuration data distributed through Group Policy should be examined for unintended exposure of sensitive information.**

## OUs and Privilege Analysis

OUs themselves do not automatically grant administrative privileges.

Their security relevance comes from how they interact with:

* GPOs
* Object permissions
* Delegation
* Administrative management
* Group memberships

For example:

```text id="a6x3n7"
OU
 │
 ├── Computer A
 ├── Computer B
 └── Computer C
       │
       ▼
      GPO
       │
       ▼
Configuration
```

The OU establishes the organizational scope; the GPO determines the relevant policy configuration.

## Delegation on OUs

Active Directory supports delegation of administrative permissions.

For example:

```text id="8n7t9u"
OU=Helpdesk
      │
      └── Delegated Permission
               │
               ▼
          Helpdesk Group
```

Delegated permissions can allow a group to perform specific administrative actions without granting complete Domain Admin privileges.

Delegation should therefore be considered during privilege analysis.

## Assessment Workflow

When examining OUs and GPOs, think in this order:

```text id="7p9s6m"
Identify OUs
     ↓
Identify objects within OUs
     ↓
Identify GPOs
     ↓
Identify GPO links
     ↓
Determine GPO scope
     ↓
Examine inheritance and filtering
     ↓
Identify GPO permissions
     ↓
Determine who can modify them
     ↓
Identify privileged targets
     ↓
Evaluate potential attack paths
```

## Example Attack-Path Concept

Consider:

```text id="5ctv5n"
User
 │
 │ Member Of
 ▼
Helpdesk Group
 │
 │ Write Permission
 ▼
GPO
 │
 │ Linked To
 ▼
Server OU
 │
 ├── Server01
 ├── Server02
 └── AdminServer
```

The existence of the relationship does not automatically establish successful privilege escalation.

The assessor must determine:

1. What exactly can the Helpdesk group modify?
2. Which GPO settings can be changed?
3. Which systems receive the GPO?
4. What privileges exist on those systems?
5. Can the relationship be safely validated?

## What to Remember

1. OUs organize Active Directory objects.
2. Security groups and OUs serve different purposes.
3. GPOs provide centralized configuration for users and computers.
4. GPOs contain Computer Configuration and User Configuration.
5. GPO information is stored through Active Directory and `SYSVOL`.
6. GPOs can be linked to sites, domains, and OUs.
7. GPO inheritance can cause policies to affect child OUs.
8. Security and WMI filtering can affect GPO scope.
9. GPOs have their own permissions.
10. The ability to modify a security-relevant GPO can become an important privilege relationship.
11. OUs can have delegated administrative permissions.
12. The security impact of a GPO depends heavily on its scope and the systems or users it affects.

## Security Assessment Perspective

When examining an OU or GPO, avoid stopping at:

```text id="i0zj4t"
"GPO exists."
```

Instead determine:

```text id="7cb2gy"
Who can modify it?
       ↓
What can they modify?
       ↓
Where is it linked?
       ↓
What objects receive it?
       ↓
Are any targets privileged?
       ↓
Does the relationship create a
potential privilege-escalation path?
```

Understanding OUs and GPOs provides the foundation for the GPO enumeration and GPO abuse workflows later in this repository.
