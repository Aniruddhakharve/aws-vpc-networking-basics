# AWS VPC Networking Fundamentals

A hands-on AWS networking project demonstrating the design, implementation, and validation of core Amazon VPC networking concepts, including public and private subnets, Internet Gateway, route tables, NAT Gateway, EC2, Security Groups, EC2 User Data, Application Load Balancer, Target Groups, VPC Peering, AWS Transit Gateway, and Transit Gateway Cross-Region Peering.

---

## 🏗️ Architecture

![AWS VPC, EC2 and Application Load Balancer Architecture](architecture/AWS-VPC-EC2-ALB-Architecture.png)

### Architecture Overview

The primary network is deployed in the **Asia Pacific (Mumbai) `ap-south-1`** AWS Region.

The architecture includes:

- One custom VPC — `31.0.0.0/16`
- One public subnet in `ap-south-1a`
- One private subnet in `ap-south-1b`
- One Internet Gateway
- One public route table
- One private route table
- One NAT Gateway
- Two public EC2 instances
- One private EC2 instance
- Security Groups
- EC2 User Data
- Apache web servers
- One internet-facing Application Load Balancer
- One Target Group

The public subnet uses a default route to the Internet Gateway.

The private subnet uses a default route to the NAT Gateway for outbound Internet connectivity.

The private EC2 instance does not have a public IPv4 address and was accessed through the public EC2 instance using its private IPv4 address.

The Application Load Balancer provides an internet-facing entry point and forwards HTTP traffic to healthy backend EC2 instances registered with the Target Group.

### VPC Peering Extension

A separate VPC Peering lab was implemented between:

- Mumbai — `ap-south-1`
- N. Virginia — `us-east-1`

The Mumbai VPC uses:

```text
31.0.0.0/16
```

The N. Virginia VPC uses:

```text
41.0.0.0/16
```

The two VPCs were connected using a VPC Peering connection.

Private IPv4 connectivity was tested between EC2 instances in both VPCs.

![AWS VPC Peering Architecture](architecture/AWS-VPC-Peering-Architecture.png)

### Transit Gateway Extension

A separate Transit Gateway lab was implemented to connect three VPCs through a centralized AWS Transit Gateway.

The three VPCs are:

```text
AWS Course VPC
31.0.0.0/16

Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
```

Each VPC was connected to the Transit Gateway using a dedicated VPC attachment.

The required remote VPC routes were then configured in the VPC route tables.

![AWS Transit Gateway Architecture](architecture/AWS-VPC-Transit-Gateway-Architecture.png)

### Transit Gateway Cross-Region Peering Extension

A separate cross-region Transit Gateway Peering lab was implemented between two VPCs located in different AWS Regions.

The practical used:

- Mumbai — `ap-south-1`
- N. Virginia — `us-east-1`

The Mumbai VPC uses:

```text
31.0.0.0/16
```

The N. Virginia VPC uses:

```text
41.0.0.0/16
```

Each region contains its own Transit Gateway.

The two regional Transit Gateways are connected using a **Transit Gateway Peering Attachment**.

Mumbai acts as the **requester** and N. Virginia acts as the **accepter**.

![AWS VPC Transit Gateway Cross-Region Peering Architecture](architecture/AWS-VPC-Transit-Gateway-Cross-Region-Peering-Architecture.png)

---

## 🎯 Project Objectives

The objective of this project is to gain hands-on experience with:

- Amazon VPC architecture
- IPv4 CIDR addressing
- Public and private subnet design
- Availability Zones
- Internet Gateways
- AWS route tables
- Local VPC routing
- Default routes
- Longest prefix matching
- Subnet-to-route-table associations
- NAT Gateway configuration
- Private subnet outbound Internet connectivity
- Public and private EC2 deployment
- Public and private IPv4 addressing
- SSH connectivity
- Accessing private resources through a public instance
- Network connectivity testing
- AWS VPC Resource Map
- AWS Security Groups
- Security Group inbound and outbound rules
- HTTP access on TCP port `80`
- SSH access on TCP port `22`
- EC2 User Data
- Automated Apache installation
- Automated server initialization using Bash
- HTTP validation using `curl`
- Application Load Balancers
- Target Groups
- EC2 target registration
- ALB listeners
- ALB security groups
- Target health checks
- Healthy and unhealthy targets
- Load balancing across multiple EC2 instances
- ALB DNS names
- Backend response validation
- VPC Peering
- Cross-region VPC connectivity
- VPC Peering requester and accepter concepts
- VPC Peering connection states
- Cross-region private IPv4 communication
- Route-table configuration for VPC Peering
- Bidirectional VPC Peering connectivity
- VPC Peering limitations
- Non-transitive VPC routing
- AWS Transit Gateway
- Transit Gateway VPC attachments
- Centralized connectivity between multiple VPCs
- Transit Gateway route-table configuration
- Private IPv4 communication through Transit Gateway
- Transit Gateway connectivity validation
- Negative connectivity testing
- Transit Gateway routing dependencies
- Transit Gateway Peering
- Cross-region Transit Gateway connectivity
- Transit Gateway requester and accepter concepts
- Transit Gateway Peering connection states
- Cross-region Transit Gateway route configuration
- Bidirectional private connectivity through Transit Gateway Peering
- Comparing VPC Peering with Transit Gateway architecture

---

## 🌐 Network Configuration

### Primary VPC

| Resource | Configuration |
|---|---|
| AWS Region | `ap-south-1` (Mumbai) |
| VPC | `31.0.0.0/16` |
| Public Subnet | `31.0.1.0/24` |
| Public Subnet AZ | `ap-south-1a` |
| Private Subnet | `31.0.2.0/24` |
| Private Subnet AZ | `ap-south-1b` |
| Public Default Route | `0.0.0.0/0 → Internet Gateway` |
| Private Default Route | `0.0.0.0/0 → NAT Gateway` |
| Public EC2 Instances | Public + Private IPv4 |
| Private EC2 | Private IPv4 only |
| Private EC2 Internet Access | NAT Gateway |
| Public EC2 HTTP Access | TCP `80` |
| Public EC2 SSH Access | TCP `22` |
| ALB Scheme | Internet-facing |
| ALB IP Type | IPv4 |
| ALB Listener | HTTP `80` |
| Target Group Protocol | HTTP |
| Target Group Port | `80` |
| Target Type | EC2 Instances |

### VPC Peering Lab

| Resource | Mumbai | N. Virginia |
|---|---|---|
| AWS Region | `ap-south-1` | `us-east-1` |
| VPC CIDR | `31.0.0.0/16` | `41.0.0.0/16` |
| Public Subnet | `31.0.1.0/24` | `41.0.1.0/24` |
| VPC Peering | Connected | Connected |
| Remote Route | `41.0.0.0/16 → Peering` | `31.0.0.0/16 → Peering` |

### Transit Gateway Lab

| Resource | AWS Course VPC | Demo Course VPC | Redshift VPC |
|---|---|---|---|
| VPC CIDR | `31.0.0.0/16` | `71.0.0.0/16` | `10.0.0.0/16` |
| Transit Gateway Attachment | Attached | Attached | Attached |
| EC2 | Deployed | Deployed | Deployed |
| Remote VPC Routing | Through TGW | Through TGW | Through TGW |

The three VPCs are connected through one centralized Transit Gateway:

```text
AWS Transit Gateway
   /      |      \
  /       |       \
 /        |        \
AWS Course  Demo Course  Redshift
31.0.0.0/16 71.0.0.0/16 10.0.0.0/16
```

### Transit Gateway Cross-Region Peering Lab

| Resource | Mumbai | N. Virginia |
|---|---|---|
| AWS Region | `ap-south-1` | `us-east-1` |
| VPC CIDR | `31.0.0.0/16` | `41.0.0.0/16` |
| Transit Gateway | Mumbai TGW | Virginia TGW |
| TGW Role | Requester | Accepter |
| TGW Peering | Connected | Connected |
| VPC Attachment | Attached | Attached |
| Remote VPC Route | `41.0.0.0/16 → Transit Gateway` | `31.0.0.0/16 → Transit Gateway` |
| EC2 | Deployed | Deployed |

The cross-region architecture is:

```text
Mumbai VPC                         Virginia VPC
31.0.0.0/16                       41.0.0.0/16
     │                                  │
     ▼                                  ▼
Mumbai Transit Gateway  ◄──────►  Virginia Transit Gateway
                              Peering
```

---

## 🧩 AWS Networking Components

### Amazon VPC

The VPC provides a logically isolated virtual network in AWS where networking resources can be created and configured.

**VPC CIDR:** `31.0.0.0/16`

[View VPC implementation →](docs/01-vpc.md)

---

### Public and Private Subnets

The primary VPC is divided into public and private subnets.

#### Public Subnet

```text
CIDR: 31.0.1.0/24
Availability Zone: ap-south-1a
```

The public subnet is associated with a route table containing:

```text
0.0.0.0/0 → Internet Gateway
```

#### Private Subnet

```text
CIDR: 31.0.2.0/24
Availability Zone: ap-south-1b
```

The private subnet does not have a direct route to the Internet Gateway.

Its default route points to the NAT Gateway for outbound Internet connectivity.

[View subnet implementation →](docs/02-subnets.md)

---

### Internet Gateway

An Internet Gateway was created and attached to the VPC to provide a path between the VPC and the Internet.

Attaching an Internet Gateway to a VPC alone does not automatically make a subnet public.

The public subnet requires:

```text
0.0.0.0/0 → Internet Gateway
```

[View Internet Gateway implementation →](docs/03-internet-gateway.md)

---

### Route Tables

Two custom route tables were used.

#### Public Route Table

| Destination | Target |
|---|---|
| `31.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

Associated with:

```text
31.0.1.0/24 — Public Subnet
```

#### Private Route Table

| Destination | Target |
|---|---|
| `31.0.0.0/16` | `local` |
| `0.0.0.0/0` | NAT Gateway |

Associated with:

```text
31.0.2.0/24 — Private Subnet
```

The private subnet therefore has outbound Internet connectivity without having a direct route to the Internet Gateway.

[View route table implementation →](docs/04-route-tables.md)

---

### NAT Gateway

A NAT Gateway provides outbound Internet connectivity for resources deployed inside the private subnet.

The private route table contains:

```text
0.0.0.0/0 → NAT Gateway
```

The traffic flow is:

```text
Private EC2
     ↓
Private Route Table
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

This allows the private EC2 instance to initiate outbound Internet connections without requiring its own public IPv4 address.

[View NAT Gateway implementation →](docs/05-nat-gateway.md)

---

### EC2 Instances in the VPC

EC2 instances were deployed to validate the public and private subnet architecture.

#### Public EC2 Instances

Two EC2 instances were deployed in:

```text
Public Subnet
31.0.1.0/24
```

The public EC2 instances run Apache and are used as backend targets for the Application Load Balancer.

#### Private EC2

The private EC2 instance was deployed in:

```text
Private Subnet
31.0.2.0/24
```

It has no public IPv4 address.

The private EC2 instance was accessed through the public EC2 instance using its private IPv4 address.

[View EC2 implementation →](docs/06-ec2-in-vpc.md)

---

### Security Groups

Security Groups act as virtual firewalls for AWS resources and control inbound and outbound traffic.

The lab used Security Groups for the EC2 instances and Application Load Balancer.

Example inbound rules used during the lab:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | `22` | `0.0.0.0/0` |
| HTTP | TCP | `80` | `0.0.0.0/0` |

The HTTP rule allows external clients to reach Apache on port `80`.

The SSH rule allows administrative access during the lab.

[View Security Group implementation →](docs/08-security-groups.md)

---

### EC2 User Data

EC2 User Data was used to automate the initial configuration of the public EC2 instances.

The Bash script was used to:

- Update the package repository
- Install Apache
- Generate a custom HTML page
- Display the instance hostname and private IPv4 address
- Restart the Apache service

Apache was verified using:

```bash
sudo systemctl status apache2
```

The web server was tested locally using:

```bash
curl localhost
```

The public IPv4 address was then opened in a browser to validate external HTTP access.

[View EC2 User Data implementation →](docs/07-ec2-user-data.md)

---

### Application Load Balancer

An **Application Load Balancer (ALB)** was introduced to provide an internet-facing entry point for the application.

The ALB was configured with:

```text
Scheme   : Internet-facing
IP Type  : IPv4
Protocol : HTTP
Port     : 80
```

The ALB receives HTTP requests and forwards them to targets registered in the Target Group.

Traffic flows as:

```text
Internet User
      ↓
Application Load Balancer
      ↓
HTTP Listener : 80
      ↓
Target Group
      ↓
Healthy EC2 Target
      ↓
Apache
```

---

### Target Group

A Target Group provides a logical grouping of backend resources for the Application Load Balancer.

The Target Group was configured with:

```text
Target Type : Instances
Protocol    : HTTP
Port        : 80
Health Path : /
```

The Target Group initially contained the existing EC2 instances.

A second public EC2 instance was later added to demonstrate load balancing across multiple healthy backend targets.

The final Target Group contained:

```text
Target Group
     |
     +── Public EC2 #1
     |
     +── Public EC2 #2
     |
     └── Private EC2
```

The two public EC2 instances were healthy targets.

The private EC2 instance remained registered but was unhealthy in the lab.

[View Application Load Balancer implementation →](docs/09-application-load-balancer.md)

---

### VPC Peering

VPC Peering was implemented to establish private connectivity between two VPCs located in different AWS Regions.

The VPCs were:

```text
Mumbai
ap-south-1
31.0.0.0/16
```

and:

```text
N. Virginia
us-east-1
41.0.0.0/16
```

The Mumbai VPC acted as the requester and the N. Virginia VPC acted as the accepter.

After the peering request was accepted, the connection became active.

However, the EC2 instances could not communicate immediately because VPC Peering does not automatically modify route tables.

The following routes were therefore added.

#### Mumbai Route Table

```text
Destination: 41.0.0.0/16
Target:      VPC Peering Connection
```

#### N. Virginia Route Table

```text
Destination: 31.0.0.0/16
Target:      VPC Peering Connection
```

After configuring both route tables, the EC2 instances were able to communicate using their private IPv4 addresses.

Connectivity was successfully tested in both directions.

[View VPC Peering implementation →](docs/10-vpc-peering.md)

---

### Transit Gateway

AWS Transit Gateway was implemented to connect three VPCs through a centralized networking hub.

The three VPCs were:

```text
AWS Course VPC
31.0.0.0/16

Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
```

A single Transit Gateway was created and used as the central connectivity point.

```text
AWS Transit Gateway
   /      |      \
  /       |       \
 /        |        \
AWS Course  Demo Course  Redshift
31.0.0.0/16 71.0.0.0/16 10.0.0.0/16
```

Each VPC was connected to the Transit Gateway using a dedicated VPC attachment.

The route tables were then configured with routes for the remote VPC CIDRs.

#### AWS Course VPC

```text
71.0.0.0/16 → Transit Gateway
10.0.0.0/16 → Transit Gateway
```

#### Demo Course VPC

```text
31.0.0.0/16 → Transit Gateway
10.0.0.0/16 → Transit Gateway
```

#### Redshift VPC

```text
31.0.0.0/16 → Transit Gateway
71.0.0.0/16 → Transit Gateway
```

EC2 instances were deployed in all three VPCs and private IPv4 connectivity was tested between them.

A negative test was also performed by removing a required Transit Gateway route. Connectivity was then blocked, demonstrating that creating a Transit Gateway attachment alone does not automatically establish end-to-end VPC routing.

[View Transit Gateway implementation →](docs/11-transit-gateway.md)

---

### Transit Gateway Cross-Region Peering

Transit Gateway Cross-Region Peering was implemented to establish private connectivity between VPCs located in different AWS Regions through separate regional Transit Gateways.

The practical used:

```text
Mumbai
ap-south-1
VPC CIDR: 31.0.0.0/16
```

and:

```text
N. Virginia
us-east-1
VPC CIDR: 41.0.0.0/16
```

A Transit Gateway was created in each region.

```text
Mumbai VPC                         Virginia VPC
31.0.0.0/16                       41.0.0.0/16
     │                                  │
     ▼                                  ▼
Mumbai Transit Gateway  ◄──────►  Virginia Transit Gateway
                              Peering
```

Mumbai was configured as the **requester** and Virginia as the **accepter**.

The Transit Gateway Peering connection was created from the Mumbai region using the Transit Gateway ID of the Virginia region.

The peering request was then accepted in the Virginia region.

After the peering connection became available, a VPC attachment was created in each region.

The Mumbai VPC was attached to the Mumbai Transit Gateway.

The Virginia VPC was attached to the Virginia Transit Gateway.

Remote VPC routes were then configured on both sides.

#### Mumbai Route Tables

```text
41.0.0.0/16 → Transit Gateway
```

#### Virginia Route Tables

```text
31.0.0.0/16 → Transit Gateway
```

EC2 instances were launched in the public subnet of each VPC for connectivity testing.

The EC2 instances successfully communicated using their **private IPv4 addresses**.

[View Transit Gateway Cross-Region Peering implementation →](docs/12-transit-gateway-cross-region-peering.md)

---

## 🔄 Traffic Flow

### Public EC2 → Internet

```text
Public EC2
    ↓
Public Subnet
    ↓
Public Route Table
    ↓
0.0.0.0/0 → Internet Gateway
    ↓
Internet
```

---

### Public EC2 → Private EC2

```text
Administrator
     ↓
    SSH
     ↓
 Public EC2
 31.0.1.x
     ↓
 VPC Local Routing
     ↓
 Private EC2
 31.0.2.x
```

Both addresses belong to:

```text
31.0.0.0/16
```

Therefore, the VPC local route provides connectivity between the subnets.

---

### Private EC2 → Internet

```text
Private EC2
31.0.2.x
     ↓
Private Subnet
31.0.2.0/24
     ↓
Private Route Table
     ↓
0.0.0.0/0 → NAT Gateway
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

The private EC2 instance can therefore initiate outbound Internet connections without a public IPv4 address.

---

### Internet → Public EC2 → Apache

```text
Internet
    ↓
Internet Gateway
    ↓
Public Route Table
    ↓
Public Subnet
    ↓
Security Group
    ↓
TCP : 80
    ↓
Public EC2
    ↓
Apache Web Server
```

---

### Internet → ALB → Target Group → EC2

```text
Internet User
      ↓
Application Load Balancer
      ↓
HTTP Listener : 80
      ↓
Target Group
      ↓
Healthy EC2 Target
      ↓
Apache
```

With two healthy public EC2 instances, the same ALB DNS name can return responses from different backend instances.

---

### Mumbai VPC → VPC Peering → N. Virginia VPC

```text
Mumbai EC2
31.0.1.x
     ↓
Mumbai Route Table
     ↓
41.0.0.0/16 → VPC Peering Connection
     ↓
N. Virginia Route Table
     ↓
N. Virginia EC2
41.0.1.x
```

The reverse path was also tested:

```text
N. Virginia EC2
41.0.1.x
     ↓
N. Virginia Route Table
     ↓
31.0.0.0/16 → VPC Peering Connection
     ↓
Mumbai Route Table
     ↓
Mumbai EC2
31.0.1.x
```

---

### AWS Course VPC → Transit Gateway → Demo Course VPC

```text
AWS Course EC2
31.0.0.0/16
     ↓
AWS Course Route Table
     ↓
71.0.0.0/16 → Transit Gateway
     ↓
AWS Transit Gateway
     ↓
Demo Course Route Table
     ↓
Demo Course EC2
71.0.0.0/16
```

---

### AWS Course VPC → Transit Gateway → Redshift VPC

```text
AWS Course EC2
31.0.0.0/16
     ↓
AWS Course Route Table
     ↓
10.0.0.0/16 → Transit Gateway
     ↓
AWS Transit Gateway
     ↓
Redshift Route Table
     ↓
Redshift EC2
10.0.0.0/16
```

---

### Demo Course VPC → Transit Gateway → Redshift VPC

```text
Demo Course EC2
71.0.0.0/16
     ↓
Demo Course Route Table
     ↓
10.0.0.0/16 → Transit Gateway
     ↓
AWS Transit Gateway
     ↓
Redshift Route Table
     ↓
Redshift EC2
10.0.0.0/16
```

The Transit Gateway therefore provides centralized connectivity between all three VPCs.

---

### Mumbai VPC → Transit Gateway → Virginia VPC

```text
Mumbai EC2
31.0.0.0/16
     ↓
Mumbai VPC Route Table
     ↓
41.0.0.0/16 → Mumbai Transit Gateway
     ↓
Mumbai Transit Gateway
     ↓
Transit Gateway Peering
     ↓
Virginia Transit Gateway
     ↓
Virginia VPC Route Table
     ↓
Virginia EC2
41.0.0.0/16
```

---

### Virginia VPC → Transit Gateway → Mumbai VPC

```text
Virginia EC2
41.0.0.0/16
     ↓
Virginia VPC Route Table
     ↓
31.0.0.0/16 → Virginia Transit Gateway
     ↓
Virginia Transit Gateway
     ↓
Transit Gateway Peering
     ↓
Mumbai Transit Gateway
     ↓
Mumbai VPC Route Table
     ↓
Mumbai EC2
31.0.0.0/16
```

The reverse path is also required for bidirectional communication.

---

## 🔐 Private EC2 Access

The private EC2 instance was not accessed directly from the Internet.

For this hands-on lab, the public EC2 instance was used as an intermediate host.

The access path was:

```text
Local / AWS Environment
        ↓
       SSH
        ↓
     Public EC2
        ↓
       SSH
        ↓
    Private EC2
```

The SSH private key was temporarily transferred to the public EC2 instance for the course lab and was **not committed to this repository**.

---

## 📸 Final AWS Resource Map

The AWS VPC Resource Map shows the networking relationships after adding the NAT Gateway and deploying the EC2 instances.

![AWS VPC Resource Map](screenshots/25-final-vpc-resource-map.png)

The primary Mumbai architecture includes:

```text
VPC: 31.0.0.0/16
│
├── Public Subnet
│   ├── Public EC2 #1
│   ├── Public EC2 #2
│   └── Public Route Table
│       ├── 31.0.0.0/16 → local
│       └── 0.0.0.0/0 → Internet Gateway
│
└── Private Subnet
    ├── Private EC2
    └── Private Route Table
        ├── 31.0.0.0/16 → local
        └── 0.0.0.0/0 → NAT Gateway
```

The Application Load Balancer and Target Group provide the application entry point and backend target layer above this VPC architecture.

The VPC Peering architecture extends the networking concepts into another AWS Region:

```text
Mumbai VPC
31.0.0.0/16
     |
     | VPC Peering
     |
N. Virginia VPC
41.0.0.0/16
```

The Transit Gateway architecture extends the project further by connecting three VPCs through a centralized Transit Gateway:

```text
AWS Transit Gateway
   /      |      \
  /       |       \
 /        |        \
AWS Course  Demo Course  Redshift
31.0.0.0/16 71.0.0.0/16 10.0.0.0/16
```

The Transit Gateway Cross-Region Peering architecture extends the project into two regional Transit Gateways:

```text
Mumbai VPC                         Virginia VPC
31.0.0.0/16                       41.0.0.0/16
     │                                  │
     ▼                                  ▼
Mumbai Transit Gateway  ◄──────►  Virginia Transit Gateway
                              Peering
```

---

## 📚 Detailed Implementation

Detailed step-by-step documentation is available for each part of the project:

| Section | Documentation |
|---|---|
| 01 | [VPC](docs/01-vpc.md) |
| 02 | [Subnets](docs/02-subnets.md) |
| 03 | [Internet Gateway](docs/03-internet-gateway.md) |
| 04 | [Route Tables](docs/04-route-tables.md) |
| 05 | [NAT Gateway](docs/05-nat-gateway.md) |
| 06 | [EC2 in VPC](docs/06-ec2-in-vpc.md) |
| 07 | [EC2 User Data](docs/07-ec2-user-data.md) |
| 08 | [Security Groups](docs/08-security-groups.md) |
| 09 | [Application Load Balancer](docs/09-application-load-balancer.md) |
| 10 | [VPC Peering](docs/10-vpc-peering.md) |
| 11 | [Transit Gateway](docs/11-transit-gateway.md) |
| 12 | [Transit Gateway Cross-Region Peering](docs/12-transit-gateway-cross-region-peering.md) |

---

## 🗂️ Project Structure

```text
aws-vpc-networking-basics/
│
├── architecture/
│   ├── AWS-VPC-EC2-ALB-Architecture.png
│   ├── AWS-VPC-EC2-NAT-Gateway-Architecture.png
│   ├── AWS-VPC-Subnet-Routing-Architecture.png
│   ├── AWS-VPC-Peering-Architecture.png
│   └── AWS-VPC-Transit-Gateway-Architecture.png
│
├── docs/
│   ├── 01-vpc.md
│   ├── 02-subnets.md
│   ├── 03-internet-gateway.md
│   ├── 04-route-tables.md
│   ├── 05-nat-gateway.md
│   ├── 06-ec2-in-vpc.md
│   ├── 07-ec2-user-data.md
│   ├── 08-security-groups.md
│   ├── 09-application-load-balancer.md
│   ├── 10-vpc-peering.md
│   ├── 11-transit-gateway.md
│   └── 12-transit-gateway-cross-region-peering.md
│
├── screenshots/
│   ├── 01-vpc-configuration.png
│   ├── 02-vpc-created.png
│   ├── 03-public-subnet-configuration.png
│   ├── 04-private-subnet-configuration.png
│   ├── 05-subnets-created.png
│   ├── 06-internet-gateway-creation.png
│   ├── 07-internet-gateway-attached.png
│   ├── 08-public-route-table-creation.png
│   ├── 09-public-route-table-internet-route.png
│   ├── 10-public-subnet-route-table-association.png
│   ├── 11-private-route-table-creation.png
│   ├── 12-final-vpc-resource-map.png
│   ├── ...
│   └── 161-virginia-to-mumbai-ping-success.png
│
└── README.md
```

---

## 📈 Skills Demonstrated

This project demonstrates practical experience with:

- AWS VPC networking
- IPv4 CIDR planning
- Public and private subnet architecture
- Internet Gateway
- NAT Gateway
- Route tables and routing
- Security Groups
- EC2
- EC2 User Data
- Apache web server
- Application Load Balancer
- Target Groups
- VPC Peering
- AWS Transit Gateway
- Transit Gateway VPC attachments
- Transit Gateway route tables
- Transit Gateway Peering
- Cross-region networking
- Private IPv4 connectivity
- Network troubleshooting
- Connectivity validation
- Negative testing
- AWS VPC Resource Map
- AWS Management Console

---

## 🧪 Connectivity Validation

The project included multiple connectivity tests.

### VPC Peering

Private connectivity was validated between:

```text
Mumbai VPC
31.0.0.0/16
        ↕
VPC Peering
        ↕
N. Virginia VPC
41.0.0.0/16
```

### Transit Gateway

Connectivity was validated between the three VPCs:

```text
31.0.0.0/16
       ↕
Transit Gateway
       ↕
71.0.0.0/16

31.0.0.0/16
       ↕
Transit Gateway
       ↕
10.0.0.0/16

71.0.0.0/16
       ↕
Transit Gateway
       ↕
10.0.0.0/16
```

A negative test was also performed by removing a required Transit Gateway route and verifying that connectivity was blocked.

### Transit Gateway Cross-Region Peering

Private IPv4 connectivity was successfully validated in both directions:

```text
Mumbai EC2
31.0.x.x
    ↓
Mumbai Transit Gateway
    ↓
Transit Gateway Peering
    ↓
Virginia Transit Gateway
    ↓
Virginia EC2
41.0.x.x
```

and:

```text
Virginia EC2
41.0.x.x
    ↓
Virginia Transit Gateway
    ↓
Transit Gateway Peering
    ↓
Mumbai Transit Gateway
    ↓
Mumbai EC2
31.0.x.x
```

The final test confirmed **bidirectional private IPv4 connectivity** between the Mumbai and N. Virginia EC2 instances.

---

## 🧠 Key Learning Outcomes

One of the key concepts demonstrated by this project is that simply naming a subnet **public** or **private** does not determine its networking behavior.

Routing and resource configuration determine how resources communicate.

Security Groups provide an additional traffic-control layer at the resource level.

In this architecture:

```text
Public Subnet
0.0.0.0/0 → Internet Gateway
      ↓
Security Group
      ↓
TCP 80 → Apache
```

while:

```text
Private Subnet
0.0.0.0/0 → NAT Gateway
```

The private EC2 instance can therefore initiate outbound Internet connections without having a public IPv4 address or a direct Internet Gateway route.

EC2 User Data demonstrates how instance initialization tasks can be automated during launch instead of being performed manually after connecting to the server.

The Application Load Balancer demonstrates how a single internet-facing endpoint can distribute HTTP requests across multiple healthy backend EC2 instances.

VPC Peering demonstrates how two VPCs in different AWS Regions can communicate privately when the peering connection is active and the appropriate routes are configured on both sides.

Transit Gateway extends this concept by providing a centralized connectivity hub through which multiple VPCs can communicate using dedicated VPC attachments and appropriate route-table entries.

The Transit Gateway lab also demonstrates that creating attachments alone does not automatically establish end-to-end connectivity. The required remote VPC CIDR routes must be present in the VPC route tables.

Transit Gateway Cross-Region Peering extends this architecture by connecting two regional Transit Gateways through a Transit Gateway Peering Attachment.

The cross-region lab demonstrates that private communication between VPCs in different AWS Regions requires:

```text
VPC Attachment
       +
Transit Gateway Peering
       +
Remote VPC Routes
       +
Security Group Rules
       =
Private Cross-Region Connectivity
```

The final validation confirmed bidirectional private IPv4 connectivity between the Mumbai and Virginia EC2 instances.

---

## 🧹 Resource Cleanup

AWS resources created for hands-on practice should be removed after completing the lab when they are no longer required.

Resources created throughout this project include:

- Custom VPCs
- Public subnets
- Private subnets
- Internet Gateways
- Public route tables
- Private route tables
- NAT Gateway
- NAT Gateway public IP / Elastic IP resources where applicable
- Public EC2 instances
- Private EC2 instance
- Security Groups
- Target Group
- Application Load Balancer
- VPC Peering connection
- Additional VPC and networking resources used in the N. Virginia VPC Peering lab
- EC2 instances used for cross-region VPC connectivity testing
- Transit Gateway
- Transit Gateway VPC attachments
- Additional VPCs used for Transit Gateway testing
- EC2 instances used for Transit Gateway connectivity testing
- Regional Transit Gateways used for cross-region Transit Gateway Peering
- Transit Gateway Peering Attachment
- VPC attachments used for cross-region Transit Gateway connectivity
- EC2 instances used for cross-region Transit Gateway connectivity testing

NAT Gateway, public IPv4 resources, EC2 instances, Transit Gateway resources, and other running AWS resources can incur charges, so temporary lab resources should not be left running unnecessarily.

---

## ⚠️ Note

This project is intended for educational and hands-on AWS networking practice.

The architecture focuses on understanding AWS VPC networking concepts and is **not intended to represent a complete production environment**.

The VPC CIDR `31.0.0.0/16` follows the addressing used during the training lab. For real-world private VPC designs, RFC 1918 private address ranges such as `10.0.0.0/8`, `172.16.0.0/12`, or `192.168.0.0/16` would normally be used.

The Security Group configuration used in this lab allows SSH and HTTP from `0.0.0.0/0` for learning and testing purposes. Restricting administrative access to trusted source addresses or using managed access mechanisms is preferable in production environments.
