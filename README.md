# VPC-Traffic-Flow-and-Security
Hands-on AWS VPC project covering traffic flow, subnet segmentation, route tables, security groups, and network ACLs — with multi-region deployment and EC2 Global View for centralized visibility.




# 🌐 AWS VPC Networking & Security Project

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)
![VPC](https://img.shields.io/badge/Service-VPC-blue?style=for-the-badge)
![Region](https://img.shields.io/badge/Deployment-Multi--Region-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [VPC Architecture](#-vpc-architecture)
- [Route Tables](#-route-tables)
- [Security Groups](#-security-groups)
- [Network ACLs (NACLs)](#-network-acls-nacls)
  - [Default NACLs](#default-nacls)
  - [Custom NACLs](#custom-nacls)
- [Multi-Region Deployment](#-multi-region-deployment)
- [EC2 Global View](#-ec2-global-view)
- [Key Learnings & Reflections](#-key-learnings--reflections)
- [Technologies Used](#-technologies-used)

---

## 🚀 Project Overview

This project demonstrates a comprehensive implementation of **AWS Virtual Private Cloud (VPC) networking and security** concepts — designed to reflect real-world, production-grade cloud infrastructure practices. The focus was on building secure, scalable, and well-segmented network architectures using native AWS services.

Key objectives accomplished:

- **Designed and deployed custom VPCs** with public and private subnet segmentation
- **Configured route tables and internet gateways** to control traffic flow
- **Implemented layered security** using both Security Groups (stateful) and Network ACLs (stateless)
- **Extended the architecture across multiple AWS regions** to demonstrate global deployment strategies
- **Leveraged EC2 Global View** for centralized cross-region resource visibility

This project reflects a strong foundational understanding of AWS networking and is directly applicable to cloud architecture, DevOps, and cloud security roles.

---

## 🗺️ Architecture Diagram

┌───────────────────────────────────────────────────────────┐
│                        AWS Cloud                          │
│                                                           │
│   ┌─────────────────────────────────────────────────┐    │
│   │              Custom VPC (10.0.0.0/16)           │    │
│   │                                                  │    │
│   │  ┌──────────────────┐  ┌──────────────────┐     │    │
│   │  │  Public Subnet   │  │  Private Subnet  │     │    │
│   │  │  (10.0.1.0/24)   │  │  (10.0.2.0/24)  │     │    │
│   │  │                  │  │                  │     │    │
│   │  │  ┌────────────┐  │  │  ┌────────────┐  │     │    │
│   │  │  │  EC2 Web   │  │  │  │  EC2 App   │  │     │    │
│   │  │  │  Server    │  │  │  │  Server    │  │     │    │
│   │  │  └────────────┘  │  │  └────────────┘  │     │    │
│   │  │       ▲          │  │        ▲          │     │    │
│   │  │  [SG + NACL]     │  │   [SG + NACL]    │     │    │
│   │  └──────────────────┘  └──────────────────┘     │    │
│   │          │                                       │    │
│   │   ┌──────▼──────┐                               │    │
│   │   │  Internet   │                               │    │
│   │   │  Gateway    │                               │    │
│   │   └─────────────┘                               │    │
│   └─────────────────────────────────────────────────┘    │
│                                                           │
│              Multi-Region: us-east-1 | us-east-2      │
└───────────────────────────────────────────────────────────┘


## 🏗️ VPC Architecture

A **Virtual Private Cloud (VPC)** is a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network you define.

### Custom VPC Configuration

| Parameter | Value |
|---|---|
| CIDR Block | `10.0.0.0/16` |
| Public Subnet | `10.0.1.0/24` |
| Private Subnet | `10.0.2.0/24` |
| Internet Gateway | Attached to VPC |
| DNS Hostnames | Enabled |
| DNS Resolution | Enabled |

### Key Design Decisions

- **Public Subnet** — Hosts internet-facing resources (e.g., web servers, load balancers). Associated with a route table that directs `0.0.0.0/0` traffic to the **Internet Gateway (IGW)**.
- **Private Subnet** — Hosts backend resources (e.g., application servers, databases). Isolated from direct inbound and outbound internet access to minimize the attack surface.
- **Subnet Segmentation** — Separating workloads into public/private subnets enforces a **defense-in-depth** network model, ensuring that backend services are never directly reachable from the public internet.

> **Why this matters:** Using a /16 CIDR block provides over 65,000 IP addresses across the VPC, giving ample room for subnet expansion, additional availability zones, and future services — a critical consideration for production environments.

---

## 🛣️ Route Tables

Route tables contain a set of rules (**routes**) that determine where network traffic from your subnet or gateway is directed.

### Public Subnet Route Table

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Internal VPC communication |
| `0.0.0.0/0` | Internet Gateway (igw-056af54033c256a | Outbound internet access |

### Private Subnet Route Table

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Internal VPC communication |

### Route Table Concepts

- **Local Route** — Automatically present in every route table; enables communication between all resources within the VPC CIDR range without leaving the AWS network.
- **Internet Gateway Route** — Added explicitly to route tables associated with public subnets; gives resources in those subnets a path to the internet.
- **Route Priority** — AWS uses the most specific route match. A `/32` host route takes priority over a `/0` default route.

> **Design note:** Each subnet should be associated with exactly one route table. By explicitly creating separate route tables for public and private subnets, you maintain clear and auditable traffic separation.

---

## 🔒 Security Groups

Security Groups act as **virtual stateful firewalls** at the EC2 instance level. They control inbound and outbound traffic based on rules you define.

### How Security Groups Work

- **Stateful** — If you allow inbound traffic on a port, the return traffic is automatically allowed, regardless of outbound rules.
- **Allow-only** — Security Groups can only have **allow** rules; there are no explicit deny rules. Traffic not matched by any rule is implicitly denied.
- **Instance-level** — Applied directly to ENIs (Elastic Network Interfaces) of EC2 instances.

### Web Server Security Group (Public Subnet)

| Direction | Protocol | Port | Source/Destination | Purpose |
|---|---|---|---|---|
| Inbound | TCP | 80 | `0.0.0.0/0` | HTTP web traffic |
| Outbound | All | All | `0.0.0.0/0` | All outbound allowed |

### App Server Security Group (Private Subnet)

| Direction | Protocol | Port | Source/Destination | Purpose |
|---|---|---|---|---|
| Outbound | All | All | `0.0.0.0/0` | All outbound allowed |

### Security Group Chaining

A powerful pattern used in this project is **Security Group referencing** — the app server's inbound rule references the **web server's Security Group ID** (rather than a CIDR block) as its source. This means only instances associated with the web server SG can communicate with the app server, regardless of IP changes. This is a production best practice for service-to-service communication.

---

## 🚧 Network ACLs (NACLs)

Network ACLs (Access Control Lists) are **stateless, subnet-level firewalls**. They provide an additional layer of security on top of Security Groups.

### NACL vs. Security Group — Key Differences

| Feature | Security Group | Network ACL |
|---|---|---|
| Operates at | Instance level (ENI) | Subnet level |
| State | Stateful | Stateless |
| Rule types | Allow only | Allow and Deny |
| Rule evaluation | All rules evaluated | Rules evaluated in order (lowest number first) |
| Default behavior | Deny all inbound; Allow all outbound | Allow all (default NACL) |
| Return traffic | Automatically allowed | Must be explicitly allowed |

---

### Default NACLs

When you create a VPC, AWS automatically creates a **default NACL** associated with all subnets that don't have an explicit NACL assignment.

**Default NACL Behavior:**

| Rule # | Type | Protocol | Port | Source | Action |
|---|---|---|---|---|---|
| 100 | All Traffic | All | All | `0.0.0.0/0` | ✅ ALLOW |
| `*` | All Traffic | All | All | `0.0.0.0/0` | ❌ DENY |

- The default NACL **allows all inbound and outbound traffic**, making it permissive by design.
- This is intentional — it ensures that subnets are not accidentally isolated when first created.
- **Production note:** The default NACL should never remain in use without modification, as it provides no meaningful traffic filtering.

---

### Custom NACLs

A **custom NACL** starts with all traffic denied by default (unlike the default NACL) and requires explicit allow rules for both inbound and outbound traffic.

**Custom NACL — Public Subnet (Inbound Rules):**

| Rule # | Protocol | Port | Source | Action | Purpose |
|---|---|---|---|---|---|
| 100 | TCP | 80 | `0.0.0.0/0` | ✅ ALLOW | HTTP |
| `*` | All | All | `0.0.0.0/0` | ❌ DENY | Catch-all deny |

**Custom NACL — Public Subnet (Outbound Rules):**

| Rule # | Protocol | Port | Destination | Action | Purpose |
|---|---|---|---|---|---|
| 100 | TCP | 80 | `0.0.0.0/0` | ✅ ALLOW | HTTP responses |
| `*` | All | All | `0.0.0.0/0` | ❌ DENY | Catch-all deny |

### Ephemeral Ports — A Critical Detail

Because NACLs are **stateless**, return traffic must be explicitly allowed. Clients initiate connections using well-known ports (80, 443) but receive responses on **ephemeral (temporary) ports** in the range `1024–65535`. Forgetting to allow this range is one of the most common NACL misconfiguration errors.

> **Layered security:** The combination of Security Groups (instance-level, stateful) and NACLs (subnet-level, stateless) provides defense-in-depth. Even if a Security Group is misconfigured, a properly configured NACL acts as a backstop — and vice versa.

---

## 🌍 Multi-Region Deployment

This project extended the VPC architecture across **multiple AWS regions** to explore global deployment, redundancy, and region-specific configurations.

### Regions Used

| Region | Code | Purpose |
|---|---|---|
| US East (N. Virginia) | `us-east-1` | Primary region |
| EU East (Ohio) | `us-east-2` | Secondary / DR region |

### Multi-Region Architecture Highlights

- **Independent VPCs per region** — Each region has its own VPC with identical architecture (CIDR blocks, subnet structure, security configuration), following infrastructure-as-code principles.
- **No VPC Peering (intentional)** — This exercise focused on understanding region-isolated deployments rather than cross-region connectivity, emphasizing that regions are fully independent failure domains.
- **Region-specific resource naming** — Resources were named with region identifiers (e.g., `vpc-web-us-east-1`, `vpc-web-us-east-2`) to maintain clarity across the AWS console.
- **Availability Zone awareness** — Subnets were distributed across multiple AZs within each region to ensure high availability within that region.

### Why Multi-Region Matters

| Benefit | Description |
|---|---|
| Disaster Recovery | If one region experiences an outage, traffic can be rerouted |
| Latency Reduction | Users in Europe connect to the EU region for lower latency |
| Data Residency | Some regulations require data to stay within geographic boundaries |
| Global Scale | Foundation for Route 53 latency-based or geolocation routing |

---

## 🖥️ EC2 Global View

**EC2 Global View** is an AWS Console feature that provides a unified, single-pane view of EC2 resources across all AWS regions simultaneously — eliminating the need to switch regions manually.

### What EC2 Global View Provides

- **Cross-region EC2 instance inventory** — View all running, stopped, and terminated instances across every region in one table.
- **VPC and subnet overview** — See all VPCs and subnets across regions without switching context.
- **Security Group visibility** — Review Security Group counts and associations globally.
- **Quick region filtering** — Filter resources by specific regions, instance types, or states.

### How It Was Used in This Project

EC2 Global View was used to:

1. **Verify deployments** — Confirm that EC2 instances and VPCs were correctly provisioned in both `us-east-1` and `us-east-2` without toggling between region dropdowns.
2. **Audit resource consistency** — Ensure that both regions maintained identical instance configurations and that no orphaned resources were left running (avoiding unexpected costs).
3. **Cross-region troubleshooting** — Quickly identify discrepancies in instance states between regions during testing.

> **Operational insight:** EC2 Global View is invaluable for multi-region projects and reflects a real-world operational practice used by cloud engineers managing global infrastructure. It reinforces the importance of centralized visibility in large-scale deployments.

---

## 💡 Key Learnings & Reflections

This project deepened practical AWS networking knowledge across several dimensions:

### Technical Takeaways

**1. VPCs are the foundation of AWS security**
Every AWS networking security control ultimately anchors back to VPC design. Getting the CIDR blocks, subnet segmentation, and routing right at the start is far easier than refactoring later.

**2. Stateful vs. Stateless is a critical distinction**
Security Groups and NACLs are often confused because both filter traffic — but their stateful vs. stateless behavior has major operational implications. Forgetting ephemeral ports in NACL outbound rules silently breaks connectivity in ways that are difficult to debug without this knowledge.

**3. Least privilege should be applied at every layer**
The principle of least privilege applies not just to IAM but to every security control: Security Groups should allow only required ports from known sources, NACLs should explicitly deny everything not needed, and private subnets should never have a route to the IGW.

**4. The default is not always safe**
The default NACL allows all traffic — a counterintuitive but deliberate AWS design choice. Understanding what defaults do (and do not) provide is essential for building genuinely secure architectures.

**5. Multi-region thinking changes architecture decisions**
Designing for multi-region from the start forces better practices: consistent naming conventions, region-agnostic CIDR planning, and resource tagging — all of which pay dividends as infrastructure scales.

**6. Global View is a multiplier for operational efficiency**
Managing resources across regions without a global view creates blind spots. EC2 Global View demonstrated how centralized visibility is not just a convenience but a security control — you cannot protect what you cannot see.

### Areas for Future Exploration

- [ ] Implement **VPC Peering** or **AWS Transit Gateway** for cross-region connectivity
- [ ] Deploy a **Bastion Host** architecture for secure private subnet access
- [ ] Add **VPC Flow Logs** for network traffic auditing and anomaly detection
- [ ] Integrate **AWS WAF** with a public-facing Application Load Balancer
- [ ] Automate infrastructure with **Terraform** or **AWS CloudFormation**
- [ ] Explore **AWS PrivateLink** for private connectivity to AWS services without IGW exposure

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| AWS VPC | Core network isolation and architecture |
| AWS EC2 | Compute instances for testing connectivity |
| Internet Gateway | Public subnet internet access |
| Security Groups | Stateful instance-level firewall |
| Network ACLs | Stateless subnet-level firewall |
| Route Tables | Traffic routing within and out of VPC |
| EC2 Global View | Cross-region resource visibility |
| AWS Console | Resource provisioning and management |

---

## 📌 Project Status
✅ VPC architecture designed and deployed  
✅ Public/private subnet segmentation implemented  
✅ Route tables configured for both subnets  
✅ Security Groups applied with least-privilege rules  
✅ Custom NACLs deployed with ephemeral port handling  
✅ Multi-region deployment completed (us-east-1, us-east-2)  
✅ EC2 Global View validated across regions  

---

*Built as part of an AWS cloud networking and security deep-dive. Designed to reflect production-grade practices and portfolio-level documentation standards.*



