# Domains, Forests and Trees

Active Directory can be organized into multiple levels of logical structure.

The three concepts that are especially important for understanding larger AD environments are:

* Domains
* Trees
* Forests

These structures define how Active Directory objects are organized and how different domains can relate to one another.

From a security-assessment perspective, understanding this hierarchy is important because privilege relationships and trust relationships can extend beyond a single domain.

## Domain

An Active Directory domain is a logical administrative and security boundary containing objects such as:

* Users
* Groups
* Computers
* Servers
* Organizational Units
* Other directory objects

A domain commonly has a DNS name such as:

```text
corp.example.com
```

A simplified domain might look like:

```text
corp.example.com
│
├── Users
├── Groups
├── Computers
├── Servers
└── Organizational Units
```

The domain provides a common identity and authentication context for its members.

## Domain Naming

Active Directory domains use DNS-based names.

For example:

```text
corp.example.com
```

The domain name can provide useful information during enumeration.

An assessment may identify:

```text
Domain:
corp.example.com

Domain Controller:
DC01.corp.example.com

Domain IP:
192.0.2.10
```

The exact naming scheme depends on the organization's environment.

## Domain Controllers and Domains

A domain is serviced by one or more Domain Controllers.

For example:

```text
corp.example.com
       │
       ├── DC01
       └── DC02
```

Multiple Domain Controllers can provide:

* Availability
* Redundancy
* Load distribution
* Replication

The Domain Controllers collectively provide the directory services for the domain.

## Child Domains

An Active Directory domain can have child domains.

For example:

```text
example.com
    │
    └── corp.example.com
```

The child domain is a separate domain while maintaining a hierarchical DNS namespace.

A larger structure could look like:

```text
example.com
    │
    ├── corp.example.com
    │
    └── research.example.com
```

Each child is its own domain with its own directory objects and Domain Controllers.

## Tree

An Active Directory **tree** is a collection of one or more domains that share a contiguous DNS namespace.

For example:

```text
example.com
    │
    ├── corp.example.com
    │
    └── research.example.com
```

These domains form a tree because their DNS namespaces are related hierarchically.

Another example:

```text
example.com
    │
    └── europe.example.com
             │
             └── uk.europe.example.com
```

The domains belong to the same DNS namespace hierarchy.

## Forest

An Active Directory **forest** is the highest-level logical container in the AD hierarchy.

A forest can contain:

* One or more domains
* One or more domain trees
* Domain Controllers
* Users
* Groups
* Computers
* Other directory objects

A simplified forest might look like:

```text
                    Forest
                      │
          ┌───────────┴───────────┐
          │                       │
       Tree A                  Tree B
          │                       │
     example.com             example.net
          │
     ┌────┴────┐
     │         │
   corp      research
```

The exact topology varies between environments.

## Forest Root Domain

The first domain created in a forest is commonly referred to as the **forest root domain**.

For example:

```text
Forest
  │
  └── example.com
       │
       ├── corp.example.com
       └── research.example.com
```

The forest root domain has a special role in the overall forest structure.

## Domain vs Tree vs Forest

A useful distinction is:

| Concept | Meaning                                                                        |
| ------- | ------------------------------------------------------------------------------ |
| Domain  | A logical AD security and administrative boundary containing directory objects |
| Tree    | A collection of domains sharing a contiguous DNS namespace                     |
| Forest  | A collection of one or more domain trees forming an AD forest                  |

A simple mental model is:

```text
Forest
  │
  ├── Tree
  │    ├── Domain
  │    └── Domain
  │
  └── Tree
       ├── Domain
       └── Domain
```

## Trust Relationships

Domains can have relationships that allow authentication and access across domain boundaries.

These relationships are commonly referred to as **trusts**.

A simplified example:

```text
Domain A
   │
   │ Trust
   ▼
Domain B
```

Trusts are important because they can affect how identities from one domain can access resources in another domain.

However:

> A trust does not automatically mean that users in one domain have administrative privileges in another domain.

The actual security impact depends on the trust configuration, permissions, group memberships, and other relationships.

## Transitive Trust

Some Active Directory trust relationships can be transitive.

Conceptually:

```text
Domain A
   │
   │ Trust
   ▼
Domain B
   │
   │ Trust
   ▼
Domain C
```

Depending on the trust configuration, relationships can extend through multiple domains.

This is one reason why domain boundaries should not automatically be treated as complete isolation boundaries during an assessment.

## Parent and Child Domains

Consider:

```text
example.com
    │
    └── corp.example.com
```

The domains have a parent-child relationship in the DNS namespace.

This relationship is also relevant when analyzing authentication and trust relationships.

A security assessment should therefore identify:

* Parent domains
* Child domains
* Other domains in the forest
* Trust relationships
* Administrative relationships between domains

## Security Boundaries

It is important to distinguish between different types of boundaries.

### Domain Boundary

A domain provides an administrative and security boundary for its directory objects.

### Forest Boundary

The forest represents a broader AD security boundary containing domains and trees.

### Network Boundary

Network segmentation determines which systems can communicate with each other.

These boundaries do not necessarily correspond exactly.

For example:

```text
Forest Boundary
┌──────────────────────────────┐
│                              │
│   Domain A    Domain B       │
│      │           │           │
│      └── Trust ──┘           │
│                              │
└──────────────────────────────┘

        Network Boundary
────────────────────────────────
```

Therefore, an AD assessment should examine both logical relationships and actual network connectivity.

## Why This Matters During Enumeration

When an assessor discovers a domain, the next questions should include:

```text
What domain am I in?
        ↓
Is there a parent domain?
        ↓
Are there child domains?
        ↓
Are there additional domains?
        ↓
What forest contains them?
        ↓
What trusts exist?
        ↓
What resources or identities are accessible across those relationships?
```

This prevents the assessment from stopping at the first discovered domain.

## Example Environment

Consider:

```text
                    Forest
              example.com
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
     corp.example.com    research.example.com
          │                   │
     ┌────┴────┐              │
     │         │              │
   Users     Servers        Users
     │                        │
     └──────── Trust ─────────┘
```

An assessment of `corp.example.com` should not necessarily assume that it is the entire AD environment.

The existence of another domain may introduce additional identities, resources, and relationships that need to be understood.

## Trust Enumeration

Trust enumeration is the process of identifying relationships between domains.

Information of interest can include:

* Trusted domains
* Trust direction
* Trust type
* Transitivity
* Associated security relationships

The exact commands and tools used for trust enumeration belong in the later **Enumeration** section of this repository.

The foundation-level objective here is simply to understand why these relationships matter.

## Cross-Domain Access

Cross-domain access can occur when permissions or trust relationships allow an identity from one domain to interact with resources in another.

For example:

```text
User A
  │
  │ Domain A
  ▼
Trust Relationship
  │
  ▼
Domain B
  │
  ▼
Resource B
```

The existence of the trust alone does not establish that the user can access the resource.

The relevant permissions and authentication relationships must be evaluated.

## Forest-Level Privileges

Some administrative privileges operate at a forest-wide level rather than being limited to a single domain.

This is important when assessing environments containing multiple domains.

For example:

```text
Forest
   │
   ├── Domain A
   │
   ├── Domain B
   │
   └── Domain C
```

A security relationship that provides forest-level administrative control can have broader consequences than one limited to a single domain.

This is why forest-level relationships should be considered during attack-path analysis.

## Assessment Mindset

When encountering a new AD environment, think beyond the first domain discovered.

Use the following mental model:

```text
Current Domain
      ↓
Domain Hierarchy
      ↓
Forest
      ↓
Other Domains
      ↓
Trust Relationships
      ↓
Cross-Domain Permissions
      ↓
Potential Attack Paths
```

The goal is to understand the complete identity and trust structure before drawing conclusions about privilege.

## What to Remember

1. A domain is a logical AD administrative and security boundary.
2. A tree is a collection of domains sharing a contiguous DNS namespace.
3. A forest is a collection of one or more domain trees.
4. A forest can contain multiple domains.
5. A domain can have multiple Domain Controllers.
6. Domains can have trust relationships with other domains.
7. Trust does not automatically grant administrative access.
8. Trust direction, transitivity, permissions, and authentication relationships matter.
9. Cross-domain relationships can create additional attack paths.
10. Forest-level relationships can have consequences beyond a single domain.

## Security Assessment Perspective

The key question is not simply:

```text
"What domain am I in?"
```

It is:

```text
"What is the complete AD structure,
and what security relationships connect its components?"
```

Understanding domains, trees, forests, and trusts provides the foundation for the trust enumeration and attack-path analysis performed later in this workflow.
