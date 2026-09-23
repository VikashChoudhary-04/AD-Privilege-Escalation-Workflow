# Network Enumeration

Network enumeration is the process of identifying hosts, network ranges, services, and infrastructure that are relevant to an Active Directory environment.

In an AD assessment, network enumeration provides the context needed to understand:

* Where the current host sits
* Which networks are reachable
* Which systems are present
* Which systems may provide AD services
* Which services are exposed
* Where Domain Controllers and other important systems may exist

The objective is not to scan everything indiscriminately.

The objective is to build an accurate picture of the network that is relevant to the authorized assessment.

## Objective

The network enumeration stage should establish:

```text
Current Host
     ↓
Network Configuration
     ↓
Reachable Networks
     ↓
Live Hosts
     ↓
Exposed Services
     ↓
Potential AD Infrastructure
```

Important targets can include:

* Domain Controllers
* DNS servers
* File servers
* Application servers
* Workstations
* Management systems
* Other domain-joined hosts

## Network Enumeration Workflow

```text
01. Identify Local Network Configuration
        ↓
02. Identify DNS Configuration
        ↓
03. Identify Routing
        ↓
04. Identify Reachable Networks
        ↓
05. Discover Relevant Hosts
        ↓
06. Identify Open Services
        ↓
07. Identify AD-Related Services
        ↓
08. Identify Domain Controllers
        ↓
09. Record Network Relationships
```

---

# 1. Identify Local Network Configuration

Start with the current host.

Collect:

* IP address
* Network interface
* Subnet mask / prefix
* Default gateway
* DNS servers
* IPv4/IPv6 configuration

Conceptually:

```text
Host
 │
 ├── IP Address
 ├── Subnet
 ├── Gateway
 └── DNS
```

This establishes the network from which further enumeration will occur.

## Why It Matters

Suppose the current system has:

```text
IP:
10.10.20.15

Network:
10.10.20.0/24

Gateway:
10.10.20.1
```

The local subnet may contain additional systems worth investigating.

However, the presence of a local subnet does not mean every address should automatically be scanned.

Use the authorized scope as the boundary.

---

# 2. Identify DNS Configuration

DNS is especially important in Active Directory.

Determine:

* DNS server addresses
* DNS search suffix
* Domain DNS name
* Relevant internal DNS infrastructure

Conceptually:

```text
Host
 │
 ▼
DNS
 │
 ├── Domain Resolution
 ├── Host Resolution
 └── AD Service Discovery
```

A domain-joined Windows host commonly uses internal DNS infrastructure associated with the AD environment.

## DNS Search Suffix

The DNS suffix can provide an early indication of the domain.

For example:

```text
corp.example.com
```

This can help establish:

* Domain name
* Internal naming convention
* Host naming patterns

---

# 3. Identify Routing

Review the local routing table.

The objective is to determine:

* Default route
* Directly connected networks
* Additional routes
* Potentially reachable internal networks

Conceptually:

```text
Current Host
     │
     ▼
Routing Table
     │
 ┌───┼────┐
 ▼   ▼    ▼
Net A Net B Net C
```

This can reveal network segmentation that may not be obvious from the local IP address.

## Assessment Principle

Do not assume:

```text
Local Subnet = Entire Environment
```

An AD environment can span multiple subnets and network segments.

---

# 4. Identify Reachable Networks

Once the local network configuration is understood, identify networks that are within the authorized assessment scope and reachable from the current host.

Potentially relevant networks may contain:

* Domain Controllers
* Server networks
* Workstation networks
* Management networks
* Application infrastructure

A simplified example:

```text
Current Network
10.10.20.0/24
       │
       ├── Workstations
       │
       └── Gateway
              │
              ├── Server Network
              │
              └── Infrastructure Network
```

Network segmentation may prevent direct access to some networks.

That itself is useful information.

---

# 5. Discover Relevant Hosts

Host discovery attempts to identify systems that are reachable within the authorized scope.

Potential targets include:

```text
Domain Controllers
File Servers
Application Servers
Workstations
Database Servers
Management Systems
```

The discovery method should account for:

* Network architecture
* Firewall rules
* ICMP filtering
* Host-based firewalls
* Scope restrictions

A host that does not respond to one discovery method should not automatically be considered offline.

---

# 6. Identify Open Services

Once relevant hosts are identified, determine which network services are exposed.

Common services encountered in AD environments include:

| Port | Common Service                     |
| ---: | ---------------------------------- |
|   53 | DNS                                |
|   88 | Kerberos                           |
|  135 | RPC                                |
|  139 | NetBIOS Session Service            |
|  389 | LDAP                               |
|  445 | SMB                                |
|  464 | Kerberos password-related services |
|  636 | LDAPS                              |
| 3268 | Global Catalog LDAP                |
| 3269 | Global Catalog over TLS            |

Other ports may be present depending on the environment.

The presence of a port does not by itself establish the identity or security significance of the service.

Service identification should be validated.

---

# 7. Recognize AD-Related Services

Certain services are strong indicators of Active Directory infrastructure.

A simplified example:

```text
Host
 │
 ├── 53    DNS
 ├── 88    Kerberos
 ├── 389   LDAP
 ├── 445   SMB
 ├── 3268  Global Catalog
 └── Other AD Services
```

A host exposing several of these services may be a Domain Controller.

However:

> **Service presence alone should not be treated as definitive proof of a system's role.**

Combine service information with DNS, LDAP, SMB, hostname, and domain information.

---

# 8. Identify Domain Controllers

One of the primary objectives of AD network enumeration is identifying Domain Controllers.

Useful indicators include:

* Kerberos
* LDAP
* SMB
* DNS
* Global Catalog
* Domain-related DNS records
* Hostname patterns
* Directory information

Conceptually:

```text
Network
  │
  ▼
Potential Host
  │
  ├── DNS
  ├── Kerberos
  ├── LDAP
  ├── SMB
  └── Global Catalog
        │
        ▼
Potential Domain Controller
```

The result should be validated using multiple indicators where possible.

---

# 9. DNS Service Discovery

Active Directory relies heavily on DNS.

DNS can provide information about:

* Domain Controllers
* Domain names
* Hostnames
* Service locations
* Internal infrastructure

Service discovery records can help clients locate AD services.

Conceptually:

```text
DNS
 │
 ├── Domain
 ├── Domain Controllers
 ├── LDAP
 ├── Kerberos
 └── Other Services
```

This makes DNS an important source of AD infrastructure information.

---

# 10. Hostname and Naming Patterns

Hostnames can provide useful contextual information.

Examples might indicate:

```text
DC01
DC02
FILE01
SQL01
APP01
WS01
```

These names can suggest possible system roles.

However:

> **Hostname conventions are indicators, not proof.**

Always validate the actual system role using available evidence.

---

# 11. Network Segmentation

Network segmentation can separate different parts of an AD environment.

For example:

```text
                 Internal Network
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
 Workstation Net   Server Net    Management Net
        │              │              │
        ▼              ▼              ▼
      Users         Servers       Admin Systems
```

Segmentation can affect:

* Host discovery
* Service access
* Authentication
* SMB connectivity
* LDAP connectivity
* Domain Controller reachability

A system that is visible from one subnet may not be reachable from another.

---

# 12. Firewall Awareness

Firewalls can affect enumeration results.

For example:

```text
Scanner
   │
   │ Request
   ▼
Firewall
   │
   ├── Allowed
   │
   └── Blocked
```

A blocked connection does not necessarily mean the service is absent.

Therefore, distinguish between:

```text
Service Not Detected
```

and:

```text
Service Confirmed Inaccessible
```

when possible.

---

# 13. Network Services and Security Context

A discovered service should be interpreted in context.

For example:

```text
Port 445
   ↓
SMB
   ↓
Which Host?
   ↓
Which Shares?
   ↓
Which Authentication?
   ↓
Which Permissions?
```

Similarly:

```text
Port 389
   ↓
LDAP
   ↓
Which Domain?
   ↓
Which Objects?
   ↓
Which Authentication?
   ↓
What Information Is Available?
```

The port number is only the beginning of the investigation.

---

# 14. Network Enumeration and AD Relationships

Network enumeration becomes more useful when combined with directory information.

For example:

```text
Network Discovery
       ↓
DC01 identified
       ↓
LDAP available
       ↓
Domain identified
       ↓
Users identified
       ↓
Computers identified
       ↓
Network relationships expanded
```

This creates an iterative process between network and directory enumeration.

---

# 15. Example Network Picture

A simplified environment may look like:

```text
                         Domain
                    corp.example.com
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
            DC01                      DC02
       10.10.10.10                10.10.10.11
              │                         │
              └────────────┬────────────┘
                           │
                     Server Network
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          FILE01         SQL01          APP01
             │
             │
       Workstation Network
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
      WS01  WS02  WS03
```

This is the type of network relationship the enumeration stage attempts to establish.

---

# 16. Tools

Different tools can support different parts of network enumeration.

Common categories include:

### Host Discovery

Used to identify reachable systems.

Examples:

* Nmap
* RustScan
* Other authorized network discovery tools

### Service Enumeration

Used to identify exposed services and versions.

Examples:

* Nmap
* Netcat
* Service-specific enumeration tools

### DNS Enumeration

Used to understand DNS records and service discovery.

Examples:

* nslookup
* dig
* DNS enumeration tools

### AD-Specific Enumeration

Once AD services are identified, tools and protocols such as:

* LDAP clients
* SMB enumeration tools
* BloodHound-compatible collectors

can provide additional context.

The exact command workflow is covered in the relevant later sections rather than being duplicated here.

---

# 17. Avoid Blind Scanning

A common mistake is:

```text
Run Huge Scan
      ↓
Generate Massive Output
      ↓
Search for "interesting" ports
```

A better approach is:

```text
Define Scope
     ↓
Understand Local Network
     ↓
Identify Relevant Networks
     ↓
Discover Hosts
     ↓
Identify Services
     ↓
Validate AD Infrastructure
     ↓
Expand Enumeration
```

This reduces unnecessary traffic and makes the results easier to interpret.

---

# Network Enumeration Checklist

```text
[ ] Local IP identified
[ ] Network/subnet identified
[ ] Default gateway identified
[ ] DNS servers identified
[ ] DNS suffix/domain identified
[ ] Routing table reviewed
[ ] Authorized reachable networks identified
[ ] Relevant hosts discovered
[ ] Services identified
[ ] AD-related services identified
[ ] Potential Domain Controllers identified
[ ] Domain Controllers validated
[ ] DNS infrastructure identified
[ ] Network segmentation considered
[ ] Firewall restrictions considered
[ ] Findings documented
```

---

# Transition to Domain Enumeration

Network enumeration should provide the infrastructure context required for the next step.

The workflow becomes:

```text
Network
   ↓
Hosts
   ↓
Services
   ↓
Potential Domain Controllers
   ↓
Domain
   ↓
Directory Enumeration
```

Once the domain infrastructure is identified, move to:

```text
Domain Enumeration
```

The next stage will focus on the **Active Directory structure itself**, including domains, Domain Controllers, domain configuration, and directory-level information.

## Final Principle

Network enumeration is not about finding the most ports.

It is about answering:

```text
"What systems and services form the
Active Directory environment,
and how can I reach them from my current position?"
```

That understanding provides the network foundation for the rest of the AD enumeration workflow.
