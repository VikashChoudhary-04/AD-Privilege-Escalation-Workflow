# Active Directory Certificate Services (AD CS)

## Purpose

Active Directory Certificate Services (AD CS) provides certificate-based identity and authentication capabilities within Windows domains.

When certificate templates, enrollment permissions, CA configuration, or certificate mappings are insecure, AD CS can create privilege escalation paths.

This file focuses on **enumerating, analyzing, validating, and documenting AD CS security weaknesses** during an authorized security assessment.

The core workflow is:

```text id="6t5x8m"
Identify AD CS
      ↓
Enumerate CAs
      ↓
Enumerate Certificate Templates
      ↓
Analyze Template Permissions
      ↓
Analyze Enrollment / Issuance Settings
      ↓
Identify Identity-Mapping Conditions
      ↓
Map Requester to Resulting Identity
      ↓
Assess Security Impact
      ↓
Validate Authorized Path
      ↓
Document Evidence
```

---

## Position in the Workflow

AD CS is part of:

```text id="m6x9g3"
05-Privilege-Escalation-Paths
```

The previous files covered:

* Group membership abuse
* ACL abuse
* GPO abuse
* Delegation abuse
* RBCD

AD CS adds certificate-based attack paths to the workflow.

The central question is:

> Can a principal obtain or influence a certificate in a way that results in authentication as a more privileged identity?

---

## AD CS Fundamentals

AD CS is Microsoft's public key infrastructure implementation for Windows environments.

It can provide certificates for:

* Users
* Computers
* Services
* Applications
* Authentication
* Encryption
* Signing

A simplified architecture is:

```text id="o4w3x7"
Active Directory
      ↓
Certificate Authority
      ↓
Certificate Templates
      ↓
Certificate Enrollment
      ↓
Issued Certificate
      ↓
Authentication / Application Use
```

---

## Main AD CS Components

Important components include:

### Certification Authority (CA)

Issues and manages certificates.

### Certificate Template

Defines how certificates are configured and who can request them.

### Enrollment

Process through which an eligible principal requests a certificate.

### Certificate Subject / SAN

Identifies the entity represented by the certificate.

### Certificate Mapping

Determines how a certificate corresponds to an AD identity during authentication.

---

## AD CS Attack-Path Model

A simplified privilege escalation path is:

```text id="h7s2v9"
Low-Privilege Principal
      ↓
Enrollment Permission
      ↓
Misconfigured Certificate Template
      ↓
Certificate Issuance
      ↓
Certificate Represents Privileged Identity
      ↓
Certificate-Based Authentication
      ↓
Higher Privilege
```

The complete path must be verified.

---

# 1. Identify AD CS

First determine whether AD CS exists in the environment.

Useful PowerShell:

```powershell id="x9h8y6"
Get-Service -Name CertSvc
```

On a CA server, inspect:

```powershell id="8f4c1s"
Get-Service CertSvc
```

Also identify domain computers that may host CA services.

Useful AD enumeration:

```powershell id="g6p3y8"
Get-ADComputer -Filter * -Properties OperatingSystem,DNSHostName |
    Where-Object {
        $_.OperatingSystem -match "Server"
    }
```

This only identifies candidate systems; the presence of a Windows Server does not prove that AD CS is installed.

---

# 2. Enumerate Certification Authorities

Identify:

* CA name
* CA hostname
* CA type
* CA status
* Domain
* Certificate validity
* Published templates

AD CS enumeration tools can help discover enterprise CAs.

A commonly used assessment tool is:

```text
certutil
```

For example:

```powershell id="6n2w4r"
certutil -config - -ping
```

This can help identify reachable enterprise CA configurations.

---

# 3. Enumerate Certificate Templates

Certificate templates define how certificates are issued.

Important properties include:

* Template name
* Template version
* Validity period
* Renewal period
* Enrollment permissions
* Auto-enrollment
* EKUs
* Subject/SAN configuration
* Approval requirements
* Manager approval
* Authorized signatures
* Object permissions

The assessment should identify templates that can produce authentication-capable certificates.

---

# 4. Template Permissions

Certificate templates have their own security descriptors.

Review who can:

* Read
* Enroll
* Auto-enroll
* Modify
* Write permissions
* Change ownership
* Modify template configuration

Important principals may include:

* Domain Users
* Authenticated Users
* Domain Computers
* Custom groups
* Service accounts
* Individual users

The key relationship is:

```text id="r9e1h7"
Principal
   ↓
Template Permission
   ↓
Certificate Enrollment
```

---

# 5. Enrollment Permission

A principal must generally have appropriate enrollment rights to request a certificate.

Identify whether enrollment is available to:

* Everyone
* Authenticated Users
* Domain Users
* Domain Computers
* Specific groups
* Specific service accounts

A broadly enrollable template is not automatically vulnerable.

The template's remaining security properties must also be analyzed.

---

# 6. Authentication-Capable Certificates

Not every certificate can be used for authentication.

Review Extended Key Usage (EKU).

Relevant EKUs can include:

* Client Authentication
* Smart Card Logon
* PKINIT-related authentication use
* Server Authentication, depending on context

Determine whether the certificate can actually authenticate to the intended service.

---

# 7. Subject and SAN Configuration

Certificate templates can control how the certificate subject and Subject Alternative Name (SAN) are populated.

Important questions include:

* Who controls the subject?
* Who controls the SAN?
* Can the requester specify identity information?
* Is the identity derived automatically from the requesting account?
* Are additional approval controls required?

A simplified risky relationship may be:

```text id="4g3j9p"
Requester
    ↓
Controls Certificate Identity
    ↓
Authentication-Capable Certificate
    ↓
Privileged AD Identity
```

This must be validated against the actual template configuration.

---

# 8. Manager Approval

Some certificate templates require additional approval before issuance.

Check whether:

* Manager approval is enabled
* Authorized signatures are required
* Enrollment agents are required
* Automatic issuance is enabled

These controls can prevent a requester from immediately obtaining a certificate.

---

# 9. Authorized Signatures

Some templates require one or more authorized signatures.

Determine:

* Number of required signatures
* Required certificate policies
* Enrollment agent requirements
* Who can provide approval

A template that appears interesting may therefore be non-exploitable without additional authorization.

---

# 10. Certificate Template Security

When reviewing a template, collect:

```text id="1x8q2r"
Template Name
      ↓
Enrollment Permissions
      ↓
Template ACL
      ↓
EKUs
      ↓
Subject/SAN Rules
      ↓
Approval Requirements
      ↓
Issuance Requirements
      ↓
Certificate Validity
```

This provides the minimum context required to assess risk.

---

# 11. Template Modification Rights

A certificate template may itself be an AD object.

Determine who can modify:

* Template permissions
* Template attributes
* Enrollment settings
* Security descriptors

Potentially relevant permissions include:

* GenericAll
* GenericWrite
* WriteDacl
* WriteOwner
* WriteProperty

A template-management path can sometimes be more significant than simple enrollment.

---

# 12. CA Permissions

The CA itself has permissions and configuration settings.

Review:

* CA administrators
* CA officers
* Certificate managers
* Enrollment permissions
* CA ACLs
* Issuance requirements
* Published templates

Do not assume that control over one certificate template means control over the entire CA.

---

# 13. Certificate Mapping

Certificate authentication depends on how the certificate is mapped to an AD identity.

Relevant mechanisms can include:

* SAN-based identity
* UPN
* DNS name
* Object SID
* Other certificate mapping mechanisms
* Stronger certificate-binding mechanisms

Mapping behavior can vary by Windows version and domain configuration.

Therefore, validate the actual authentication behavior rather than relying solely on template configuration.

---

# 14. Strong Certificate Binding

Modern Windows environments include stronger certificate-to-account binding protections.

When assessing certificate authentication, determine:

* Domain functional configuration
* Domain controller certificate-mapping behavior
* Certificate issuance properties
* Certificate mapping method
* Whether strong binding is enforced
* Relevant Windows security updates and configuration

A template that historically produced an attack path may behave differently in a hardened environment.

---

# 15. Certificate Templates and Privileged Accounts

Identify whether a template can issue authentication-capable certificates that could represent:

* Domain administrators
* Enterprise administrators
* Server administrators
* Service accounts
* Other privileged identities

The important question is:

```text id="2o3g5z"
Who Can Enroll?
        ↓
What Identity Can Be Represented?
        ↓
What Authentication Is Possible?
        ↓
What Privilege Does That Identity Have?
```

---

# 16. Common AD CS Misconfiguration Categories

AD CS weaknesses commonly involve combinations of:

* Excessive enrollment permissions
* Weak certificate template ACLs
* User-controlled subject/SAN fields
* Authentication-capable EKUs
* Missing approval controls
* Excessive template modification rights
* Weak CA permissions
* Insecure enrollment-agent configurations
* Insecure certificate mapping

Do not classify a template solely because it matches a known configuration pattern.

Validate the complete path.

---

# 17. BloodHound-Compatible AD CS Analysis

Modern BloodHound-compatible tooling may collect AD CS-related relationships depending on the collector and environment.

These relationships can help identify:

* Certificate authorities
* Certificate templates
* Enrollment permissions
* Template control
* Principals
* Potential authentication paths

Use graph analysis as a discovery mechanism:

```text id="x7w4d2"
Principal
      ↓
Template
      ↓
Enrollment
      ↓
Certificate
      ↓
Identity
      ↓
Privilege
```

Then manually verify each relationship.

---

# 18. Certipy

**Certipy** is widely used for AD CS security assessment.

It can assist with:

* CA enumeration
* Certificate-template enumeration
* Finding potentially dangerous configurations
* Certificate requests
* Authentication testing
* AD CS attack-path analysis

Example enumeration:

```bash
certipy find -u user@example.local -p 'PASSWORD' -dc-ip 10.10.10.10
```

Use credentials and targets only within the authorized assessment environment.

Avoid storing real credentials in public repositories or documentation.

---

# 19. Certificate Request Validation

When a certificate request is authorized for testing, verify:

* Requesting account
* Certificate template
* CA
* Requested identity
* EKU
* Subject/SAN
* Approval requirements
* Issuance result
* Certificate validity

The objective is to establish whether the issued certificate actually represents the intended identity.

---

# 20. Certificate-Based Authentication

A certificate should not be considered proof of privilege by itself.

The complete path is:

```text id="k8m4t2"
Certificate
      ↓
Certificate Mapping
      ↓
AD Identity
      ↓
Authentication
      ↓
Authorization
      ↓
Effective Privilege
```

The final authorization step is critical.

---

# 21. AD CS and ACL Abuse

AD CS frequently overlaps with ACL abuse.

For example:

```text id="e7w9x2"
User
 ↓
Template Control
 ↓
Modify Template
 ↓
Authentication-Capable Configuration
 ↓
Certificate Enrollment
 ↓
Privileged Identity
```

Therefore, correlate certificate-template permissions with Section 05 ACL analysis.

---

# 22. AD CS and Credential Findings

AD CS can also correlate with credential findings.

For example:

```text id="j4c8p1"
Compromised Account
 ↓
Certificate Enrollment
 ↓
Persistent Certificate Credential
 ↓
Authentication
```

Certificate-based access should be treated separately from password-based credentials when documenting persistence or access.

---

# 23. AD CS and Privilege Escalation

A complete escalation path should contain:

```text id="v6y3h8"
Initial Principal
      ↓
Enrollment / Template Permission
      ↓
Certificate Configuration
      ↓
Certificate Issuance
      ↓
Identity Mapping
      ↓
Authentication
      ↓
Higher Privilege
```

If any link is missing, classify the result accordingly rather than assuming successful escalation.

---

# 24. Validate the Attack Path

Use this validation sequence:

```text id="q4h6z8"
AD CS Identified
      ↓
CA Identified
      ↓
Template Identified
      ↓
Enrollment Permission Confirmed
      ↓
Template Configuration Confirmed
      ↓
Identity Control Confirmed
      ↓
Certificate Issued
      ↓
Certificate Mapping Confirmed
      ↓
Authentication Confirmed
      ↓
Effective Privilege Confirmed
```

This provides a reproducible evidence chain.

---

# 25. Evidence Collection

Record:

| Field          | Description                                  |
| -------------- | -------------------------------------------- |
| CA             | Certification Authority                      |
| Template       | Certificate template                         |
| Principal      | Requesting identity                          |
| Enrollment     | Enrollment permission                        |
| EKU            | Authentication/use properties                |
| Subject/SAN    | Certificate identity fields                  |
| Approval       | Approval requirements                        |
| Mapping        | Certificate-to-account mapping               |
| Authentication | Result of authentication validation          |
| Privilege      | Resulting access                             |
| Evidence       | Commands, output, screenshots, or graph data |

Do not store private keys, passwords, or authentication tokens in a public repository.

---

# 26. Common False Positives

### Enrollment Without Authentication

A principal may enroll but the resulting certificate may not support the intended authentication scenario.

### Approval Required

Manager or authorized approval may prevent unauthorized issuance.

### Strong Certificate Binding

Modern certificate-mapping protections may prevent identity impersonation.

### No Privileged Identity

The certificate may map only to a low-privileged account.

### Restricted Template

Security filtering may limit who can actually enroll.

### Disabled Account

The represented account may not be usable.

### Incorrect Identity Mapping

The certificate may not map to the expected AD identity.

---

# 27. Common Mistakes

Avoid:

* Treating every AD CS deployment as vulnerable
* Treating every certificate template as dangerous
* Ignoring enrollment permissions
* Ignoring template ACLs
* Ignoring EKUs
* Ignoring subject/SAN controls
* Ignoring manager approval
* Ignoring authorized signatures
* Ignoring certificate mapping
* Ignoring modern strong-binding protections
* Assuming certificate issuance automatically means Domain Admin
* Publishing private keys or credentials
* Modifying production certificate infrastructure without authorization

---

# 28. Assessment Checklist

### AD CS Discovery

* [ ] Identify CAs
* [ ] Identify CA hosts
* [ ] Enumerate certificate templates
* [ ] Identify published templates
* [ ] Identify enrollment endpoints

### Template Analysis

* [ ] Identify enrollment permissions
* [ ] Review template ACL
* [ ] Review EKUs
* [ ] Review subject/SAN configuration
* [ ] Review manager approval
* [ ] Review authorized signatures
* [ ] Review validity and renewal

### CA Analysis

* [ ] Review CA permissions
* [ ] Identify CA administrators
* [ ] Identify certificate managers
* [ ] Review issuance requirements
* [ ] Review published templates

### Identity Analysis

* [ ] Identify certificate mapping
* [ ] Check strong certificate binding
* [ ] Identify represented account
* [ ] Determine account privilege
* [ ] Check account status

### Validation

* [ ] Confirm enrollment
* [ ] Confirm certificate configuration
* [ ] Confirm issuance
* [ ] Confirm identity mapping
* [ ] Confirm authentication
* [ ] Confirm effective privilege
* [ ] Collect evidence

---

## Transition

After AD CS analysis, continue to:

**`05-Privilege-Escalation-Paths/Trust-Abuse.md`**

The next file covers **Active Directory trust relationships** and how misconfigured or excessive trust relationships can create cross-domain or cross-forest privilege paths.
