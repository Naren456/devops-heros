# AWS VPC — Virtual Private Cloud

## 1. What is VPC?

**Amazon VPC (Virtual Private Cloud)** is a logically isolated virtual network inside AWS.

It allows us to control how AWS resources communicate with:

- The internet
- Other AWS resources
- Private networks
- Internal applications

For example, EC2 instances and RDS databases can be placed inside a VPC and connected using controlled network routes.

```text
                    AWS
                     │
                     ▼
              ┌───────────────┐
              │      VPC      │
              │ 10.0.0.0/16   │
              │               │
              │ ┌───────────┐ │
              │ │  Subnet   │ │
              │ │           │ │
              │ │   EC2     │ │
              │ └───────────┘ │
              │               │
              │ ┌───────────┐ │
              │ │  Subnet   │ │
              │ │           │ │
              │ │    RDS    │ │
              │ └───────────┘ │
              └───────────────┘
```

---

# 2. Why do we need VPC?

Without network isolation and routing controls, it would be difficult to safely design production applications.

A typical application might contain:

```text
Internet
   │
   ▼
Load Balancer
   │
   ▼
Application Servers
   │
   ▼
Database
```

We don't necessarily want the database directly accessible from the public internet.

A VPC allows us to separate these resources into different network areas.

```text
                 Internet
                    │
                    ▼
             Public Subnet
                    │
              Load Balancer
                    │
                    ▼
            Private Subnet
                    │
               EC2 Servers
                    │
                    ▼
            Private Subnet
                    │
                  RDS
```

---

# 3. VPC CIDR

A VPC needs an IP address range.

This is defined using **CIDR notation**.

Example:

```text
10.0.0.0/16
```

This represents the IP address space available to the VPC.

We can divide this into smaller subnet ranges.

```text
VPC
10.0.0.0/16
│
├── Public Subnet
│   10.0.1.0/24
│
├── Private Subnet
│   10.0.2.0/24
│
└── Database Subnet
    10.0.3.0/24
```

---

# 4. Subnets

A **subnet** is a smaller IP network inside a VPC.

Subnets are associated with a specific Availability Zone.

For example:

```text
VPC: 10.0.0.0/16

        ┌──────────────────────────────┐
        │             VPC              │
        │                              │
        │  AZ-a              AZ-b      │
        │   │                  │       │
        │   ▼                  ▼       │
        │ Public             Public    │
        │ Subnet             Subnet    │
        │   │                  │       │
        │   ▼                  ▼       │
        │  EC2                EC2       │
        └──────────────────────────────┘
```

Using multiple Availability Zones improves availability and fault tolerance.

---

# 5. Public Subnet

A subnet is considered **public** when its routing configuration provides a path to an Internet Gateway.

Example:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

Typical resources:

- Load balancers
- Web servers
- Bastion hosts
- Public-facing services

A resource also needs appropriate IP addressing and security rules to actually be reachable from the internet.

---

# 6. Private Subnet

A private subnet does not have a direct route to an Internet Gateway.

Example:

```text
Internet
   X
   │
   │
Private Subnet
   │
   ├── Application Server
   │
   └── Database
```

Typical resources:

- Application servers
- Databases
- Internal services
- Caches

Private subnets are commonly used to reduce direct exposure of internal resources.

---

# 7. Route Tables

A **route table** determines where network traffic should go.

Example:

```text
Destination       Target

10.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

The first route allows communication within the VPC.

The second route sends other IPv4 traffic toward the Internet Gateway.

Example:

```text
EC2
 │
 ▼
Route Table
 │
 ├── 10.0.0.0/16 → local
 │
 └── 0.0.0.0/0 → Internet Gateway
```

---

# 8. Internet Gateway

An **Internet Gateway (IGW)** provides a path between a VPC and the internet.

Example:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

An Internet Gateway is attached to the VPC.

However, simply attaching an Internet Gateway does **not** automatically make every resource public.

Routing, IP addressing, and security rules must also be configured appropriately.

---

# 9. NAT Gateway

A **NAT Gateway** allows resources in a private subnet to initiate connections to external networks without requiring those resources to be directly reachable from the internet.

Example:

```text
                 Internet
                    │
                    ▼
             Internet Gateway
                    │
                    ▼
             Public Subnet
                    │
               NAT Gateway
                    │
                    ▼
             Private Subnet
                    │
                   EC2
```

For example, a private EC2 instance might need to:

- Download software updates
- Access external APIs
- Download packages

The private instance can use the NAT Gateway for outbound connectivity.

---

# 10. Security Groups

A **Security Group** acts as a stateful virtual firewall for resources such as EC2 instances.

Example:

```text
EC2 Security Group

Inbound:
22   → SSH
80   → HTTP
443  → HTTPS
```

For example:

```text
Internet
   │
   ├── HTTPS :443 ──► EC2
   │
   └── HTTP  :80  ──► EC2
```

Unnecessary ports should not be exposed.

Security Groups are **stateful**, meaning return traffic for an allowed connection is automatically permitted.

---

# 11. Network ACL

A **Network Access Control List (NACL)** provides another layer of network traffic control at the subnet level.

Simplified difference:

```text
Security Group
      ↓
Resource-level firewall

NACL
      ↓
Subnet-level traffic rules
```

NACLs are **stateless**, so inbound and outbound traffic rules need to be considered separately.

---

# 12. Public vs Private Subnet

| Feature | Public Subnet | Private Subnet |
|---|---|---|
| Direct route to Internet Gateway | Yes | No |
| Typical use | Web/load-balancer layer | Application/database layer |
| Direct internet exposure | Possible | Not directly through IGW |
| Common resources | Load Balancer, web server | EC2 app, RDS |
| NAT Gateway | Not required for inbound internet access | Often used for outbound internet access |

A subnet itself doesn't automatically make a resource public or private; the routing and resource configuration determine connectivity.

---

# 13. Example Architecture

A common three-tier architecture looks like this:

```text
                         INTERNET
                             │
                             ▼
                    ┌────────────────┐
                    │ Internet       │
                    │ Gateway        │
                    └───────┬────────┘
                            │
                    ┌───────▼────────┐
                    │ Public Subnet  │
                    │                │
                    │ Load Balancer  │
                    └───────┬────────┘
                            │
                            ▼
                    ┌────────────────┐
                    │ Private Subnet │
                    │                │
                    │ EC2 App Server │
                    └───────┬────────┘
                            │
                            ▼
                    ┌────────────────┐
                    │ Private Subnet │
                    │                │
                    │      RDS       │
                    └────────────────┘
```

This architecture separates:

```text
Internet-facing layer
        ↓
Application layer
        ↓
Database layer
```

---

# 14. Example: VPS inside a VPC

Suppose we want to deploy a Node.js application.

We could create:

```text
VPC
10.0.0.0/16
│
├── Public Subnet
│   10.0.1.0/24
│   │
│   └── EC2
│       Node.js application
│
└── Private Subnet
    10.0.2.0/24
    │
    └── RDS
        PostgreSQL
```

Traffic flow:

```text
User
 │
 ▼
Internet
 │
 ▼
Internet Gateway
 │
 ▼
Public EC2
 │
 ▼
Private RDS
```

The database doesn't need to be directly exposed to the internet.

---

# 15. VPC and AWS Services

VPC is particularly important when working with services such as:

```text
VPC
 │
 ├── EC2
 │
 ├── RDS
 │
 ├── Load Balancer
 │
 ├── ECS
 │
 ├── EKS
 │
 └── ElastiCache
```

These services can participate in your AWS network architecture.

---

# 16. Important VPC Components

The most important components to remember are:

```text
VPC
 │
 ├── CIDR
 │
 ├── Subnets
 │
 ├── Route Tables
 │
 ├── Internet Gateway
 │
 ├── NAT Gateway
 │
 ├── Security Groups
 │
 └── Network ACLs
```

---

# 17. Simple Mental Model

Think of AWS networking like a physical organization:

```text
VPC
 │
 └── Your private campus

Subnet
 │
 └── A building/section

Route Table
 │
 └── Road directions

Internet Gateway
 │
 └── Main connection to the internet

NAT Gateway
 │
 └── Controlled outbound exit

Security Group
 │
 └── Firewall around a server

NACL
 │
 └── Security checkpoint for a subnet

EC2
 │
 └── Your server

RDS
 │
 └── Your database
```

---

# 18. VPC in One Diagram

```text
                              INTERNET
                                  │
                                  ▼
                         Internet Gateway
                                  │
                     ┌────────────┴────────────┐
                     │          VPC            │
                     │       10.0.0.0/16       │
                     │                         │
                     │  ┌──────────────────┐   │
                     │  │  Public Subnet   │   │
                     │  │   10.0.1.0/24    │   │
                     │  │                  │   │
                     │  │  Load Balancer   │   │
                     │  │       │          │   │
                     │  └───────┼──────────┘   │
                     │          │              │
                     │          ▼              │
                     │  ┌──────────────────┐   │
                     │  │ Private Subnet   │   │
                     │  │   10.0.2.0/24    │   │
                     │  │                  │   │
                     │  │      EC2         │   │
                     │  │       │          │   │
                     │  └───────┼──────────┘   │
                     │          │              │
                     │          ▼              │
                     │  ┌──────────────────┐   │
                     │  │ Database Subnet  │   │
                     │  │   10.0.3.0/24    │   │
                     │  │                  │   │
                     │  │       RDS        │   │
                     │  └──────────────────┘   │
                     │                         │
                     └─────────────────────────┘
```

# 19. Key Takeaways

1. **VPC = private network in AWS**
2. **CIDR = IP address range of the network**
3. **Subnet = smaller network inside the VPC**
4. **Route Table = controls where traffic goes**
5. **Internet Gateway = connectivity between VPC and internet**
6. **NAT Gateway = outbound internet access for private resources**
7. **Security Group = stateful resource-level firewall**
8. **NACL = stateless subnet-level traffic control**
9. **Public subnet = subnet with a route to an Internet Gateway**
10. **Private subnet = no direct route to an Internet Gateway**

## One-line definition

> **Amazon VPC allows you to design and control the network environment in which your AWS resources run.**