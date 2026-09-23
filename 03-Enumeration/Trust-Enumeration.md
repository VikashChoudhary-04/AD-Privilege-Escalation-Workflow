# Trust Enumeration

## Purpose

Trust enumeration is the process of identifying and analyzing trust relationships between Active Directory domains and forests.

In a multi-domain or multi-forest environment, authentication and access may extend beyond the user's current domain.

The objective is to determine:

* Which domains exist
* Which forests exist
* Which trust relationships are configured
* Trust direction
* Trust type
* Transitivity
* Cross-domain authentication relationships
* External vs forest relationships
* Which principals may cross administrative boundaries
* Potential cross-domain privilege paths

The goal is to understand **where authentication and authorization boundaries connect**.

---

## Enumeration Workflow

Use the following workflow:

```text id="7c4m2x"
Identify Current Domain
        ↓
Identify Forest
        ↓
Identify Known Domains
        ↓
Enumerate Trusts
        ↓
Identify Trust Direction
        ↓
Identify Trust Type
        ↓
Determine Transitivity
        ↓
Identify Trusted / Trusting Domains
        ↓
Review Cross-Domain Principals
        ↓
Map Authentication Relationships
        ↓
Identify Potential Cross-Domain Paths
        ↓
Record Findings
        ↓
Move to Delegation Enumeration
```

---

## 1. Understand AD Trusts

A trust establishes a relationship that can allow security principals in one domain to be recognized or authenticated across another domain.

Conceptually:

```text id="6m9p2k"
Domain A
   │
   │ Trust
   ▼
Domain B
```

A trust does not automatically mean:

```text id="q8x4v1"
Domain A → Full Administrative Access → Domain B
```

The actual security relationship depends on:

* Direction
* Type
* Transitivity
* Authentication configuration
* SID filtering
* Resource permissions
* Group memberships
* Delegation

---

## 2. Identify the Current Domain

Start with the domain already identified during domain enumeration.

Record:

```text id="n3v7c5"
Current Domain:
Domain SID:
Forest:
Domain Controllers:
```

This becomes the reference point for analyzing trust relationships.

---

## 3. Identify Known Domains

Determine whether the environment contains additional domains.

Example:

```text id="4q8m2p"
Forest
│
├── corp.example.com
├── dev.example.com
└── europe.example.com
```

Record:

```text id="f7c3x9"
Domain Name
Domain SID
Forest
Relationship to Current Domain
```

The existence of another domain does not by itself establish that the current user can access it.

---

## 4. Enumerate Trust Relationships

Windows provides tools for viewing trust information.

Example:

```cmd id="x5m2k8"
nltest /domain_trusts
```

PowerShell:

```powershell id="q8v4n1"
Get-ADTrust -Filter *
```

These can provide information such as:

```text id="2c7p5m"
Source Domain
Target Domain
Trust Direction
Trust Type
Transitivity
```

Other LDAP and graph-based tools can also collect trust relationships.

---

## 5. Understand Trust Direction

Trust direction is important.

Consider:

```text id="6w9m3x"
Domain A ─────► Domain B
```

The arrow represents the direction of the trust relationship.

Do not interpret this simply as "A trusts B" without understanding what that means in the specific AD trust model.

Record the exact direction returned by the directory.

Example:

```text id="r4c8p2"
Source:
corp.example.com

Target:
dev.example.com

Direction:
[Recorded Value]
```

---

## 6. Understand Trust Types

Common trust relationships include:

```text id="8n3q6v"
Parent-Child
Tree-Root
External
Forest
Shortcut
```

The exact relationships depend on the AD architecture.

### Parent-Child Trust

Child domains within the same forest have automatic transitive trust relationships.

Example:

```text id="z5m2x8"
example.com
     │
     └── corp.example.com
```

### Tree-Root Trust

Connects domain trees within the same forest.

### External Trust

Can connect domains across forest boundaries or legacy environments.

### Forest Trust

Connects two forests.

### Shortcut Trust

Can provide a more direct trust path between domains within a forest.

Each type should be documented rather than treated as equivalent.

---

## 7. Understand Trust Transitivity

A trust may be:

```text id="q6v9m3"
Transitive
Non-Transitive
```

A transitive relationship can extend beyond the directly connected domains depending on the trust architecture.

Example:

```text id="7x4p8k"
Domain A
   │
   ▼
Domain B
   │
   ▼
Domain C
```

With appropriate transitive relationships, authentication relationships may extend across the chain.

Do not assume that every domain in a forest is equally reachable from every account.

Validate the actual relationship.

---

## 8. Understand Intra-Forest Trusts

Domains within the same AD forest normally have inherent trust relationships.

Example:

```text id="k2m8v4"
Forest
│
├── corp.example.com
│
├── dev.example.com
│
└── europe.example.com
```

These relationships allow authentication and directory operations to work across the forest architecture.

However:

```text id="v7q3m1"
Trust
  ≠
Administrative Privilege
```

Resource permissions and security boundaries still determine effective access.

---

## 9. Identify External Trusts

External trusts can connect domains outside the current forest.

Example:

```text id="f4x8n2"
Forest A
   │
   │ External Trust
   ▼
Forest B / External Domain
```

Record:

```text id="2k6m9p"
Source Domain
Target Domain
Trust Type
Direction
Transitivity
Authentication Configuration
```

External relationships deserve careful analysis because they may cross organizational or administrative boundaries.

---

## 10. Identify Forest Trusts

Forest trusts can connect separate AD forests.

Example:

```text id="w8m3q5"
Forest A
    │
    │ Forest Trust
    ▼
Forest B
```

A forest trust does not automatically grant all users unrestricted access to the other forest.

Access still depends on:

* Trust configuration
* Authentication
* Resource permissions
* Group memberships
* SID filtering
* Selective authentication where configured

---

## 11. Identify Selective Authentication

Some trust configurations can restrict which resources trusted principals may authenticate to.

Conceptually:

```text id="c5m8x2"
Trusted Domain
      ↓
Selective Authentication
      ↓
Specific Resources
```

This means:

```text id="a2v7k4"
Trust Exists
      ≠
All Resources Accessible
```

Where possible, determine whether selective authentication or equivalent restrictions are configured.

---

## 12. Understand SID Filtering

SID filtering can affect how security identifiers from trusted domains are handled across trust boundaries.

Conceptually:

```text id="m7q4p9"
Trusted Principal
       ↓
Trust Boundary
       ↓
SID Validation / Filtering
       ↓
Effective Identity
```

Its configuration can influence the security properties of certain trust relationships.

Record whether relevant SID filtering settings are present where accessible.

Do not infer the setting solely from the existence of a trust.

---

## 13. Identify Cross-Domain Group Membership

Trust relationships become particularly interesting when combined with group membership.

Example:

```text id="r8c2v6"
Domain A User
      ↓
Domain B Group
      ↓
Resource in Domain B
```

Record:

```text id="5n7x3m"
Source Domain
Source Principal
Target Domain
Target Group
Resource
Effective Permission
```

This helps determine whether the trust is actually relevant to the current security context.

---

## 14. Identify Cross-Domain Administrative Relationships

Look for relationships such as:

```text id="9m4p7x"
Domain A Group
      ↓
Administrative Group
      ↓
Domain B Computer
```

or:

```text id="v6q2k8"
Domain A User
      ↓
Domain B Group
      ↓
Resource Access
```

These relationships may create cross-domain attack paths.

The relationship should be validated rather than inferred from trust alone.

---

## 15. Identify Cross-Domain Authentication

Determine whether principals from one domain can authenticate to resources in another domain.

Conceptually:

```text id="w3x8m5"
Domain A User
      ↓
Kerberos / Authentication
      ↓
Domain B Resource
```

Record:

```text id="1q7c4v"
Source Principal
Source Domain
Target Resource
Target Domain
Authentication Method
Observed / Configured Access
```

Authentication capability does not necessarily mean authorization to the resource.

Keep these concepts separate:

```text id="2v9m6x"
Authentication
     ≠
Authorization
```

---

## 16. Identify Trust Paths

In environments with multiple domains, build a trust graph.

Example:

```text id="k4m8p2"
CORP
 │
 ├── Trust ──► DEV
 │
 └── Trust ──► EUROPE
                │
                └── Trust ──► PARTNER
```

Then annotate each relationship with:

```text id="x6q3v9"
Direction
Type
Transitivity
Authentication Restrictions
```

This provides a much clearer picture than a simple list of trusts.

---

## 17. Correlate Trusts with Domain Enumeration

Use the information collected earlier:

```text id="m2v7c5"
Domain
 ↓
Domain Controller
 ↓
Forest
 ↓
Trust
 ↓
Other Domain
```

For each trusted domain, determine:

* Domain Controllers
* DNS information
* Domain SID
* Forest membership
* Available directory services
* Relevant groups
* Relevant computer objects

Only enumerate additional environments when they are within the authorized scope.

---

## 18. Correlate Trusts with User and Group Enumeration

Trust relationships become more useful when combined with identity data.

Example:

```text id="7n4p8c"
User
 ↓
Group
 ↓
Trust
 ↓
Other Domain
 ↓
Group
 ↓
Resource
```

This can reveal potential cross-domain privilege relationships.

For example:

```text id="q5m2x7"
CORP\alice
   ↓
CORP\Developers
   ↓
Trust
   ↓
DEV\App-Admins
   ↓
DEV\APP01
```

The actual permissions must be verified.

---

## 19. Identify High-Value Cross-Domain Relationships

Prioritize relationships involving:

```text id="8x3v6m"
Domain Administrators
Enterprise Administrators
Privileged Groups
Administrative Workstations
Domain Controllers
Management Systems
Backup Systems
Certificate Infrastructure
```

A trust touching a high-value resource does not automatically imply a vulnerability.

The important question is:

```text id="m7q4c2"
Can the current principal actually traverse this relationship
and obtain meaningful access?
```

---

## 20. Useful Enumeration Tools

### Windows

```text id="p8m4x2"
nltest
netdom
```

Example:

```cmd id="4q7v9m"
nltest /domain_trusts
```

### PowerShell

```text id="c6x2n8"
Get-ADTrust
Get-ADDomain
Get-ADForest
```

Examples:

```powershell id="y3m7p5"
Get-ADTrust -Filter *
```

```powershell id="f8q4k2"
Get-ADForest
```

### LDAP

```text id="m5v8x1"
ldapsearch
```

### BloodHound-Compatible Collection

```text id="z7c3q9"
BloodHound-compatible collectors
```

Graph collection can make relationships between:

```text id="8m2v6p"
Domains
Trusts
Users
Groups
Computers
```

easier to visualize.

---

## 21. Build a Trust Inventory

Create a structured inventory.

Example:

```text id="q2x7m4"
Source Domain:
Target Domain:
Forest:
Trust Type:
Trust Direction:
Transitive:
Selective Authentication:
SID Filtering:
Authentication Notes:
Cross-Domain Groups:
Relevant Resources:
Potential Security Impact:
Notes:
```

Example:

```text id="c8m3v6"
Source Domain:
corp.example.com

Target Domain:
dev.example.com

Trust Type:
Forest / Domain relationship

Direction:
[Observed Value]

Transitive:
[Observed Value]

Notes:
- Cross-domain administrative relationships require further analysis
```

---

## 22. Build a Trust Graph

Represent relationships visually.

```text id="6v4q8m"
           FOREST A
              │
        ┌─────┴─────┐
        ▼           ▼
      CORP         DEV
        │           │
        │           └── APP01
        │
        └── DC01
```

Then add identity relationships:

```text id="n5x2p7"
CORP\User
    ↓
CORP\Group
    ↓
Trust
    ↓
DEV\Group
    ↓
DEV\Resource
```

This provides the foundation for cross-domain attack-path analysis.

---

## 23. Separate Trust from Access

One of the most important principles is:

```text id="u8c5m3"
Trust
  ↓
Authentication Relationship
```

does not necessarily mean:

```text id="x4q7p9"
Trust
  ↓
Administrative Access
```

Effective access may additionally require:

* Group membership
* Resource permissions
* ACLs
* Authentication configuration
* Delegation
* GPO permissions
* Local administrative rights

Always validate the complete path.

---

## 24. Identify Potential Cross-Domain Attack Paths

During enumeration, document relationships such as:

```text id="k6m2v8"
User
 ↓
Group
 ↓
Trust
 ↓
Target Domain
 ↓
Target Group
 ↓
Resource
```

or:

```text id="p4x9c7"
Computer
 ↓
Trust
 ↓
Other Domain
 ↓
Privileged Relationship
```

These are hypotheses for later validation.

Do not modify trust configuration during enumeration.

---

## 25. Assessment Mindset

Ask:

```text id="r7m3q5"
Which domains exist?
        ↓
Which forests exist?
        ↓
How are they connected?
        ↓
What type of trust exists?
        ↓
Which direction does it operate?
        ↓
Is it transitive?
        ↓
Are authentication restrictions present?
        ↓
Which users/groups cross the boundary?
        ↓
Which resources are reachable?
        ↓
Does the relationship create a meaningful privilege path?
```

The goal is to transform:

```text id="a2v8m4"
Trust Relationship
```

into:

```text id="c7q3x9"
Source Identity
      ↓
Trust
      ↓
Target Identity / Domain
      ↓
Resource
      ↓
Effective Access
```

---

## Trust Enumeration Checklist

### Domain Discovery

* [ ] Current domain identified
* [ ] Forest identified
* [ ] Other domains identified
* [ ] Domain SIDs recorded

### Trust Discovery

* [ ] Trusts enumerated
* [ ] Trust direction recorded
* [ ] Trust type recorded
* [ ] Transitivity recorded
* [ ] External trusts identified
* [ ] Forest trusts identified
* [ ] Parent-child relationships identified

### Security Configuration

* [ ] Selective authentication reviewed where accessible
* [ ] SID filtering reviewed where applicable
* [ ] Authentication restrictions documented
* [ ] Trust scope documented

### Cross-Domain Relationships

* [ ] Cross-domain group membership identified
* [ ] Cross-domain administrative relationships identified
* [ ] Cross-domain resources identified
* [ ] Authentication relationships documented
* [ ] Potential cross-domain attack paths recorded

### Documentation

* [ ] Trust inventory created
* [ ] Trust graph created
* [ ] Source and target domains recorded
* [ ] Evidence sources recorded
* [ ] Trust separated from actual authorization
* [ ] Follow-up relationships documented

---

## Transition to Delegation Enumeration

Trust enumeration establishes **where domain and forest boundaries connect**.

The final enumeration stage is **Delegation Enumeration**, which examines how Kerberos delegation configurations allow services and computers to act on behalf of users.

Move to:

```text id="w3m7x2"
03-Enumeration/Delegation-Enumeration.md
```

The next stage will connect:

```text
Computer / Account
        ↓
Delegation Configuration
        ↓
Kerberos
        ↓
Service / Resource
        ↓
Potential Privilege Relationship
```
