# AWS VPC Networking Fundamentals

A hands-on AWS networking project demonstrating the design and configuration of a custom Amazon VPC with public and private subnets, an Internet Gateway, custom route tables, and subnet associations.

This project was created to develop a practical understanding of AWS networking fundamentals and how routing controls communication between resources inside a VPC and the Internet.

---

## 🏗️ Architecture

![AWS VPC Network Architecture](architecture/aws-vpc-network-architecture.png)

### Architecture Overview

The network is deployed in the **Asia Pacific (Mumbai) `ap-south-1`** AWS Region and consists of:

- One custom VPC with CIDR block `31.0.0.0/16`
- One public subnet in `ap-south-1a`
- One private subnet in `ap-south-1b`
- One Internet Gateway attached to the VPC
- One custom public route table
- One custom private route table
- A default Internet route for the public subnet

The public subnet has a route to the Internet Gateway, while the private subnet does not have a direct route to the Internet.

---

## 🎯 Project Objectives

The objective of this project is to gain hands-on experience with core AWS VPC networking concepts, including:

- Creating a custom Amazon VPC
- Understanding IPv4 CIDR blocks
- Creating public and private subnets
- Deploying subnets across Availability Zones
- Creating and attaching an Internet Gateway
- Creating custom route tables
- Configuring Internet routing
- Associating route tables with subnets
- Understanding the difference between public and private subnet routing

---

## 🌐 Network Configuration

| Resource | Configuration |
|---|---|
| AWS Region | `ap-south-1` (Mumbai) |
| VPC | `31.0.0.0/16` |
| Public Subnet | `31.0.1.0/24` |
| Public Subnet AZ | `ap-south-1a` |
| Private Subnet | `31.0.2.0/24` |
| Private Subnet AZ | `ap-south-1b` |
| Internet Route | `0.0.0.0/0 → Internet Gateway` |

---

## 🧩 AWS Networking Components

### Amazon VPC

The VPC provides a logically isolated virtual network in AWS where networking resources can be created and configured.

**VPC CIDR:** `31.0.0.0/16`

[View VPC implementation →](docs/01-vpc.md)

### Public and Private Subnets

The VPC is divided into two subnets:

**Public Subnet**

```text
CIDR: 31.0.1.0/24
Availability Zone: ap-south-1a
```

**Private Subnet**

```text
CIDR: 31.0.2.0/24
Availability Zone: ap-south-1b
```

The public subnet is associated with a route table containing a route to the Internet Gateway.

The private subnet does not have a direct Internet Gateway route.

[View subnet implementation →](docs/02-subnets.md)

### Internet Gateway

An Internet Gateway is created and attached to the VPC to provide a path between the VPC and the Internet.

Attaching an Internet Gateway to a VPC alone does not automatically make a subnet public. The subnet must use a route table containing an appropriate route to the Internet Gateway.

[View Internet Gateway implementation →](docs/03-internet-gateway.md)

### Route Tables

Two custom route tables are used in the architecture.

#### Public Route Table

| Destination | Target |
|---|---|
| `31.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

Associated with:

`31.0.1.0/24` — Public Subnet

#### Private Route Table

| Destination | Target |
|---|---|
| `31.0.0.0/16` | `local` |

Associated with:

`31.0.2.0/24` — Private Subnet

The private route table does not contain a default route to the Internet Gateway.

[View route table implementation →](docs/04-route-tables.md)

---

## 🔄 Traffic Flow

### Public Subnet

Internet-bound traffic from resources in the public subnet follows:

```text
Public Subnet
     ↓
Public Route Table
     ↓
0.0.0.0/0
     ↓
Internet Gateway
     ↓
Internet
```

### Private Subnet

The private subnet currently contains only the VPC local route:

```text
Private Subnet
     ↓
Private Route Table
     ↓
31.0.0.0/16 → local
```

Therefore, it does not have a direct route to the Internet.

---

## 📸 Final AWS Resource Map

The AWS VPC Resource Map below shows the relationship between the VPC, subnets, route tables, and Internet Gateway after completing the configuration.

![AWS VPC Resource Map](screenshots/12-final-vpc-resource-map.png)

---

## 📚 Detailed Implementation

Detailed step-by-step documentation is available for each part of the project:

1. [VPC Creation and Configuration](docs/01-vpc.md)
2. [Public and Private Subnet Configuration](docs/02-subnets.md)
3. [Internet Gateway Configuration](docs/03-internet-gateway.md)
4. [Route Table Configuration](docs/04-route-tables.md)

Each section includes explanations of the networking concept, configuration details, implementation steps, and AWS Console screenshots.

---

## 📁 Repository Structure

```text
aws-vpc-networking-basics/
│
├── README.md
├── LICENSE
│
├── architecture/
│   └── aws-vpc-network-architecture.png
│
├── docs/
│   ├── 01-vpc.md
│   ├── 02-subnets.md
│   ├── 03-internet-gateway.md
│   └── 04-route-tables.md
│
└── screenshots/
    ├── 01-vpc-configuration.png
    ├── 02-vpc-created.png
    ├── 03-public-subnet-configuration.png
    ├── 04-private-subnet-configuration.png
    ├── 05-subnets-created.png
    ├── 06-internet-gateway-creation.png
    ├── 07-internet-gateway-attached.png
    ├── 08-public-route-table-creation.png
    ├── 09-public-route-table-internet-route.png
    ├── 10-public-subnet-route-table-association.png
    ├── 11-private-route-table-creation.png
    └── 12-final-vpc-resource-map.png
```

---

## 🧠 Key Concepts Learned

Through this project, I gained hands-on experience with:

- Amazon VPC architecture
- IPv4 CIDR addressing
- Public and private subnet design
- Availability Zones
- Internet Gateways
- AWS route tables
- Local VPC routing
- Default routes (`0.0.0.0/0`)
- Subnet-to-route-table associations
- Public vs. private subnet routing
- AWS VPC Resource Map

One of the key concepts demonstrated by this project is that simply naming a subnet "public" does not make it public. Internet connectivity depends on the subnet's routing configuration, along with the networking configuration of resources deployed inside it.

---

## 🧹 Resource Cleanup

AWS resources created for hands-on practice should be removed after completing the lab when they are no longer required.

Resources created in this project include:

- Custom VPC
- Public subnet
- Private subnet
- Internet Gateway
- Public route table
- Private route table

---

## ⚠️ Note

This project is intended for educational and hands-on AWS networking practice. The architecture focuses on fundamental VPC networking concepts and is not intended to represent a complete production environment.