# Kerberos

Kerberos is a network authentication protocol used extensively by Active Directory.

It provides ticket-based authentication, allowing users and services to authenticate without repeatedly sending passwords across the network.

From a security-assessment perspective, understanding Kerberos is essential because several important Active Directory security concepts depend on its authentication model, including:

* Service Principal Names (SPNs)
* Kerberoasting
* AS-REP Roasting
* Delegation
* Kerberos tickets
* Service authentication

## Why Kerberos Matters in Active Directory

Active Directory uses Kerberos as its primary authentication protocol for domain authentication.

A simplified model is:

```text id="g7v8f2"
User
  │
  │ Authentication
  ▼
Domain Controller
  │
  ▼
Kerberos
  │
  ├── Ticket Granting Ticket
  │
  └── Service Tickets
```

Instead of continuously sending a user's password to every service, Kerberos uses tickets to prove identity.

## Key Kerberos Components

The main components to understand are:

* Client
* Key Distribution Center (KDC)
* Authentication Service (AS)
* Ticket Granting Service (TGS)
* Target Service

A simplified architecture is:

```text id="b7p4s8"
                    Domain Controller
                           │
                    Key Distribution
                       Center (KDC)
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       Authentication              Ticket Granting
          Service                     Service
             AS                           TGS
```

The KDC is commonly provided by the Domain Controller.

## Key Distribution Center

The Key Distribution Center (KDC) is the Kerberos component responsible for issuing tickets.

It consists logically of:

```text id="z5m3q1"
KDC
│
├── Authentication Service (AS)
│
└── Ticket Granting Service (TGS)
```

The KDC uses information associated with identities and services to issue and validate Kerberos tickets.

## Authentication Service

The Authentication Service (AS) handles the initial authentication process.

A simplified flow is:

```text id="m9r5v2"
Client
  │
  │ Authentication Request
  ▼
Authentication Service
  │
  ▼
Ticket Granting Ticket
```

The resulting Ticket Granting Ticket is commonly abbreviated as:

```text id="q4n8z6"
TGT
```

## Ticket Granting Ticket

The Ticket Granting Ticket (TGT) allows an authenticated user to request service tickets without repeatedly performing the initial authentication process.

Conceptually:

```text id="d8x4r2"
User
  │
  ▼
Authentication
  │
  ▼
TGT
  │
  ├──► Service Ticket A
  ├──► Service Ticket B
  └──► Service Ticket C
```

The TGT is therefore an important part of the Kerberos authentication workflow.

## Ticket Granting Service

The Ticket Granting Service (TGS) issues service tickets.

A simplified flow is:

```text id="v2n7c9"
Client
  │
  │ TGT + Service Request
  ▼
Ticket Granting Service
  │
  ▼
Service Ticket
```

The service ticket can then be presented to the target service.

## Service Tickets

A service ticket is used to authenticate to a particular service.

For example:

```text id="x4q7m2"
User
  │
  ▼
TGT
  │
  ▼
TGS
  │
  ▼
Service Ticket
  │
  ▼
Target Service
```

The service ticket is associated with the service being accessed.

## Kerberos Authentication Flow

A simplified end-to-end process is:

```text id="f6p8r1"
             ┌─────────────────────┐
             │        User         │
             └──────────┬──────────┘
                        │
                        │ 1. Authentication
                        ▼
             ┌─────────────────────┐
             │ Authentication     │
             │ Service (AS)        │
             └──────────┬──────────┘
                        │
                        │ 2. TGT
                        ▼
             ┌─────────────────────┐
             │        TGT          │
             └──────────┬──────────┘
                        │
                        │ 3. Service Request
                        ▼
             ┌─────────────────────┐
             │ Ticket Granting     │
             │ Service (TGS)       │
             └──────────┬──────────┘
                        │
                        │ 4. Service Ticket
                        ▼
             ┌─────────────────────┐
             │   Target Service    │
             └─────────────────────┘
```

This is a simplified conceptual model rather than a complete protocol trace.

## Kerberos and Passwords

Kerberos is designed so that the user's password does not need to be repeatedly transmitted to every service.

The user's long-term secret is used during the authentication process to establish trust and obtain tickets.

After authentication, tickets can be used to request access to services.

This ticket-based model is one of the defining characteristics of Kerberos.

## Service Principal Names

A Service Principal Name (SPN) identifies a service instance for Kerberos authentication.

A simplified example is:

```text id="h7w5q3"
MSSQLSvc/db01.corp.example.com:1433
```

An SPN is associated with an account that represents the service.

For example:

```text id="q1c8z4"
Service Account
      │
      └── SPN
            │
            ▼
        SQL Service
```

SPNs are important because Kerberos uses them to determine which service a ticket is intended for.

## Why SPNs Matter to Security Assessments

During an assessment, SPNs can help identify:

* Service accounts
* Services running in the domain
* Accounts associated with network services
* Potential Kerberos attack surfaces

SPN enumeration therefore becomes an important part of AD enumeration.

## Kerberoasting

Kerberoasting is an attack technique that abuses the Kerberos service-ticket mechanism to obtain material that can be subjected to offline password-cracking attempts for certain service accounts.

The conceptual relationship is:

```text id="v4j2q6"
Authenticated Domain User
          │
          ▼
       SPN Found
          │
          ▼
Service Ticket Requested
          │
          ▼
Ticket Material Obtained
          │
          ▼
Offline Password Analysis
```

The feasibility and impact depend on factors such as:

* Service-account configuration
* Password strength
* Encryption types
* Account privileges
* Environment configuration

The detailed workflow belongs in the later `04-Credentials-and-Authentication/` section.

## AS-REP Roasting

Kerberos normally performs a pre-authentication process before issuing an AS response.

If an account is configured so that Kerberos pre-authentication is not required, the authentication exchange can expose material that may be subjected to offline password analysis.

Conceptually:

```text id="m4s8v3"
User Account
      │
      ▼
Pre-Authentication Disabled
      │
      ▼
AS Response
      │
      ▼
Offline Password Analysis
```

This technique is commonly called **AS-REP Roasting**.

The detailed assessment procedure belongs in the credential and authentication section.

## Kerberos Delegation

Kerberos delegation allows a service to act on behalf of a user when accessing another service.

A simplified concept is:

```text id="j2w6n4"
User
  │
  ▼
Service A
  │
  │ Delegation
  ▼
Service B
```

Delegation can be useful for legitimate enterprise applications.

However, improperly configured delegation can create security risks.

Important delegation concepts include:

* Unconstrained delegation
* Constrained delegation
* Resource-Based Constrained Delegation (RBCD)

These concepts become important later when analyzing privilege-escalation paths.

## Unconstrained Delegation

In an unconstrained delegation configuration, a trusted service can potentially obtain and use delegated user authentication information in a broad manner.

A simplified model is:

```text id="b4n7s1"
User
  │
  ▼
Trusted Service
  │
  ▼
Delegated Authentication
```

Because of the potential security impact, systems configured for unconstrained delegation deserve attention during an assessment.

## Constrained Delegation

Constrained delegation restricts delegation to specified services.

Conceptually:

```text id="p5v9c3"
Service A
   │
   │ Allowed Delegation
   ▼
Service B
```

The configuration limits which services can be accessed through the delegation relationship.

Misconfiguration can still create security-relevant attack paths.

## Resource-Based Constrained Delegation

Resource-Based Constrained Delegation (RBCD) changes the model by placing control over delegation on the target resource.

A simplified representation is:

```text id="s6f4x8"
Target Computer
      │
      │ Defines trusted principals
      ▼
Delegation Relationship
      │
      ▼
Source Principal
```

RBCD is particularly relevant to AD privilege-escalation analysis because permissions on computer objects can determine whether a principal can establish a delegation relationship.

The detailed abuse path belongs in the later privilege-escalation section.

## Kerberos and Time

Kerberos relies on time-sensitive authentication mechanisms.

Significant clock differences between systems can cause authentication problems.

Therefore, when troubleshooting Kerberos authentication during an assessment, time synchronization can be relevant.

Conceptually:

```text id="e9q2k5"
Client Time
     │
     │ Synchronization
     ▼
Domain Controller Time
```

## Kerberos and DNS

Kerberos relies heavily on correct service naming and domain infrastructure.

DNS helps clients locate domain services and resolve hostnames used in authentication.

For example:

```text id="x3p7m1"
Client
  │
  ▼
DNS
  │
  ▼
Domain Controller / Service
  │
  ▼
Kerberos
```

Incorrect DNS configuration can therefore affect Kerberos authentication.

## Kerberos Encryption

Kerberos supports different encryption types.

The security properties of the authentication process depend partly on:

* Encryption type
* Account configuration
* Password strength
* Domain configuration
* Protocol implementation

During security assessments, the supported encryption mechanisms can be relevant when evaluating Kerberos-related attack techniques.

## Kerberos Tickets and Security

Tickets represent authenticated access within the Kerberos model.

A simplified relationship is:

```text id="c5y7q9"
Identity
   │
   ▼
Authentication
   │
   ▼
Ticket
   │
   ▼
Service Access
```

If an attacker obtains or can forge certain Kerberos tickets under specific conditions, the resulting access can be significant.

This is why protecting privileged credentials and Kerberos infrastructure is important.

## Golden Tickets

A Golden Ticket is a forged Kerberos Ticket Granting Ticket created using the domain's Kerberos service account secret.

At a high level:

```text id="r6t3m8"
Compromised Domain Secret
          │
          ▼
   Forged TGT
          │
          ▼
Domain Authentication
```

This requires highly privileged access to sensitive domain authentication material.

Golden Ticket attacks are therefore generally associated with post-compromise scenarios rather than ordinary low-privilege enumeration.

The technique is included here as a conceptual foundation; detailed post-exploitation procedures are outside the initial foundation stage.

## Silver Tickets

A Silver Ticket is a forged Kerberos service ticket associated with a particular service.

Conceptually:

```text id="w4p6z2"
Compromised Service Secret
          │
          ▼
   Forged Service Ticket
          │
          ▼
     Target Service
```

Unlike a Golden Ticket, a Silver Ticket is associated with a specific service rather than a domain-wide TGT.

Again, this is a post-compromise technique requiring appropriate secrets and privileges.

## Kerberos Attack Surface

The Kerberos attack surface can be viewed as:

```text id="q7m5r1"
                 Kerberos
                    │
       ┌────────────┼─────────────┐
       │            │             │
       ▼            ▼             ▼
     SPNs        Tickets      Delegation
       │            │             │
       ▼            ▼             ▼
Kerberoasting   Ticket Abuse   Delegation Abuse
       │
       ▼
AS-REP Roasting
```

Not every Kerberos feature represents a vulnerability.

The assessment objective is to identify **misconfigurations, weak credentials, excessive privileges, or unsafe relationships**.

## Kerberos Enumeration

When assessing Kerberos, useful information can include:

* Domain name
* Domain Controllers
* Kerberos service availability
* SPNs
* Service accounts
* Encryption configuration
* Delegation settings
* Authentication-related attributes

A conceptual workflow is:

```text id="z8n3k6"
Identify Domain
      ↓
Identify Domain Controller
      ↓
Confirm Kerberos
      ↓
Enumerate SPNs
      ↓
Identify Service Accounts
      ↓
Identify Roasting Opportunities
      ↓
Identify Delegation
      ↓
Map Relationships
      ↓
Evaluate Potential Attack Paths
```

The specific commands and tools will be introduced later.

## Assessment Questions

When examining Kerberos, ask:

```text id="k4r6s8"
Which Domain Controllers provide Kerberos?
          ↓
Which accounts have SPNs?
          ↓
Which services do those SPNs represent?
          ↓
Are any accounts configured without pre-authentication?
          ↓
Are delegation configurations present?
          ↓
Which principals are trusted for delegation?
          ↓
What privileges do those principals have?
          ↓
Can any relationship create a privilege-escalation path?
```

## What to Remember

1. Kerberos is the primary authentication protocol used by Active Directory.
2. The KDC is commonly provided by a Domain Controller.
3. The KDC logically contains the Authentication Service and Ticket Granting Service.
4. A TGT is used to obtain service tickets.
5. Service tickets authenticate users to specific services.
6. SPNs identify service instances for Kerberos authentication.
7. SPNs are important when identifying service accounts and investigating Kerberoasting.
8. Accounts without Kerberos pre-authentication can be relevant to AS-REP Roasting.
9. Delegation allows services to perform authentication-related actions on behalf of users.
10. Unconstrained, constrained, and resource-based constrained delegation have different security models.
11. Kerberos depends on correct domain infrastructure, including DNS and time synchronization.
12. Kerberos-related security issues usually involve weak credentials, unsafe configuration, excessive privileges, or compromised authentication material.

## Security Assessment Perspective

Do not think of Kerberos simply as:

```text id="7n8w4m"
"the protocol that authenticates users."
```

For an AD security assessment, think of it as:

```text id="p5c9x2"
Identity
   ↓
Authentication
   ↓
Tickets
   ↓
Services
   ↓
Delegation
   ↓
Trust Relationships
   ↓
Potential Attack Paths
```

Understanding this chain provides the foundation for the SPN, Kerberoasting, AS-REP Roasting, delegation, and attack-path workflows later in the repository.
