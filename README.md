# AWS VPC Networking Fundamentals

A hands-on AWS networking project demonstrating the design, implementation, and validation of core Amazon VPC networking concepts, including public and private subnets, Internet Gateway, route tables, NAT Gateway, EC2, Security Groups, EC2 User Data, Application Load Balancer, Target Groups, VPC Peering, and AWS Transit Gateway.

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
- VPC Peering vs. Transit Gateway architecture

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

> **Security Note:** Copying private SSH keys onto an intermediate instance is not the preferred approach for production environments. More secure access methods can include SSH agent forwarding or AWS Systems Manager Session Manager.

---

## 🧪 Connectivity Validation

### Public EC2 Access

Successful SSH connectivity to the public EC2 instance was verified.

![Public EC2 SSH Connection](screenshots/20-public-ec2-ssh-connection.png)

---

### Private EC2 Access

The private EC2 instance was successfully accessed from the public EC2 instance using its private IPv4 address.

![Private EC2 SSH via Public EC2](screenshots/23-private-ec2-ssh-via-public-ec2.png)

---

### Private EC2 Internet Connectivity

Outbound Internet connectivity from the private EC2 instance was tested using:

```bash
ping 8.8.8.8
```

The test successfully received responses.

![Private EC2 Internet Test](screenshots/24-private-ec2-internet-test.png)

This validates:

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

---

### EC2 User Data and Apache Validation

Apache was verified as running:

![Apache Service Running](screenshots/28-apache-service-running.png)

The generated page was tested locally using `curl localhost`:

![EC2 User Data Localhost Test](screenshots/29-user-data-localhost-test.png)

The web server was also accessed using the EC2 instance's public IPv4 address:

![EC2 User Data Browser Test](screenshots/30-user-data-browser-test.png)

---

### Security Group Validation

The Security Group configuration was reviewed:

![Security Group Configuration](screenshots/31-security-group-configuration.png)

HTTP traffic on TCP port `80` was allowed:

![Security Group Inbound HTTP](screenshots/32-security-group-inbound-http.png)

Outbound rules were also reviewed:

![Security Group Outbound Rules](screenshots/33-security-group-outbound-rules.png)

---

## ⚖️ Application Load Balancer Validation

### Target Group Configuration

The Target Group was configured for EC2 instances using HTTP on port `80` with an HTTP health check on `/`.

![Target Group Configuration](screenshots/34-target-group-configuration.png)

---

### Initial Target Registration

The EC2 instances were registered in the Target Group.

![Target Group Registered Targets](screenshots/35-target-group-registered-targets.png)

The Target Group health status was then reviewed:

![Target Group Health Status](screenshots/36-target-group-health-status.png)

---

### Application Load Balancer Configuration

An internet-facing IPv4 Application Load Balancer was created in the existing VPC.

![Application Load Balancer Configuration](screenshots/37-application-load-balancer-configuration.png)

The ALB network configuration was completed using the VPC subnets.

![ALB Network Configuration](screenshots/38-alb-network-configuration.png)

---

### ALB Security Group

The ALB Security Group was configured to allow HTTP traffic on TCP port `80`.

![ALB Security Group HTTP Rule](screenshots/39-alb-security-group-http-rule.png)

---

### ALB Listener

The Application Load Balancer was configured with an HTTP listener on port `80`.

The listener forwards traffic to the Target Group.

![ALB Listener Target Group](screenshots/40-alb-listener-target-group.png)

---

### ALB DNS

The Application Load Balancer provided a DNS name for accessing the application.

![ALB DNS Details](screenshots/41-alb-dns-details.png)

---

### First ALB Application Test

The ALB DNS name was opened in a browser.

The request successfully reached an EC2 backend and returned the Apache page.

![ALB Application Test](screenshots/42-alb-application-test.png)

---

### Second Public EC2 Instance

A second public EC2 instance was created to demonstrate load balancing across multiple healthy backend servers.

![Second Public EC2 Configuration](screenshots/43-second-public-ec2-configuration.png)

The instance successfully entered the Running state:

![Second Public EC2 Running](screenshots/44-second-public-ec2-running.png)

The second public EC2 instance was tested directly:

![Second Public EC2 Application Test](screenshots/45-second-public-ec2-application-test.png)

---

### Final Target Group

The second public EC2 instance was registered with the existing Target Group.

```text
Target Group
     |
     +── Public EC2 #1
     |
     +── Public EC2 #2
     |
     └── Private EC2
```

![Target Group Two Public EC2](screenshots/46-target-group-two-public-ec2.png)

---

### Target Health

The final Target Group health status showed the health state of the registered targets.

The two public EC2 instances were healthy targets.

The private EC2 instance remained registered but was unhealthy in the lab.

![Target Group Final Health Status](screenshots/47-target-group-final-health-status.png)

---

### ALB Backend Response 1

A request sent to the ALB DNS name returned server information from one of the healthy public EC2 instances.

![ALB Backend Response 1](screenshots/48-alb-backend-response-1.png)

---

### ALB Backend Response 2

A subsequent request through the same ALB DNS name returned server information from the other healthy public EC2 instance.

![ALB Backend Response 2](screenshots/49-alb-backend-response-2.png)

These tests demonstrate that the same ALB DNS name can serve responses from different healthy backend EC2 instances.

---

## 🔗 VPC Peering Validation

### VPC Peering Request

The VPC Peering request was created from the Mumbai VPC toward the N. Virginia VPC.

![VPC Peering Request Configuration](screenshots/55-vpc-peering-request-configuration.png)

---

### Pending Acceptance

The peering request initially appeared in the N. Virginia region with a pending acceptance status.

![VPC Peering Pending Acceptance](screenshots/56-vpc-peering-pending-acceptance.png)

---

### Peering Request Accepted

The peering request was accepted from the N. Virginia VPC.

![VPC Peering Acceptance](screenshots/57-vpc-peering-acceptance.png)

---

### Peering Connection Active

After acceptance, the VPC Peering connection became active.

![VPC Peering Active](screenshots/58-vpc-peering-active.png)

---

### EC2 Instances in Both Regions

An EC2 instance was launched in the Mumbai VPC:

![Mumbai EC2 Network Configuration](screenshots/59-mumbai-ec2-network-configuration.png)

An EC2 instance was also launched in the N. Virginia VPC:

![N. Virginia EC2 Network Configuration](screenshots/60-virginia-ec2-network-configuration.png)

Both instances successfully reached the Running state:

![Mumbai EC2 Running](screenshots/61-mumbai-ec2-running.png)

![N. Virginia EC2 Running](screenshots/62-virginia-ec2-running.png)

---

### Connectivity Test Before Routes

Before adding the VPC Peering routes, the Mumbai EC2 instance attempted to ping the private IP address of the N. Virginia EC2 instance.

The ping failed because the route tables did not yet contain routes toward the remote VPC CIDR.

![Mumbai to N. Virginia Ping Failed](screenshots/63-mumbai-to-virginia-ping-failed.png)

This demonstrated that an active VPC Peering connection alone does not automatically provide routing between the VPCs.

---

### Mumbai Route Table Peering Route

The Mumbai route table was updated with:

```text
Destination: 41.0.0.0/16
Target:      VPC Peering Connection
```

![Mumbai Route Table Peering Route](screenshots/64-mumbai-route-table-peering-route.png)

---

### N. Virginia Route Table Peering Route

The N. Virginia route table was updated with:

```text
Destination: 31.0.0.0/16
Target:      VPC Peering Connection
```

![N. Virginia Route Table Peering Route](screenshots/65-virginia-route-table-peering-route.png)

---

### Mumbai → N. Virginia Connectivity

After configuring the routes, the Mumbai EC2 instance successfully pinged the private IP of the N. Virginia EC2 instance.

![Mumbai to N. Virginia Ping Success](screenshots/66-mumbai-to-virginia-ping-success.png)

This confirmed that traffic from the Mumbai VPC could reach the N. Virginia VPC through the VPC Peering connection.

---

### N. Virginia → Mumbai Connectivity

The reverse direction was also tested.

The N. Virginia EC2 instance successfully pinged the private IP of the Mumbai EC2 instance.

![N. Virginia to Mumbai Ping Success](screenshots/67-virginia-to-mumbai-ping-success.png)

This confirmed that communication was working in both directions.

---

## 🚇 Transit Gateway Validation

### Three-VPC Architecture

The Transit Gateway lab uses three independent VPCs:

```text
AWS Course VPC
31.0.0.0/16

Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
```

Each VPC contains its own networking infrastructure, including subnets, route tables, and Internet Gateway resources.

![Three VPC Resource Map](screenshots/94-three-vpc-resource-map.png)

---

### Transit Gateway Configuration

A centralized AWS Transit Gateway was created for the three-VPC architecture.

![Transit Gateway Configuration](screenshots/95-transit-gateway-configuration.png)

The Transit Gateway was successfully created and became available:

![Transit Gateway Created](screenshots/96-transit-gateway-created.png)

---

### Transit Gateway Attachments

Each VPC was connected to the Transit Gateway using a dedicated VPC attachment.

#### AWS Course VPC Attachment

![AWS Course Transit Gateway Attachment Configuration](screenshots/97-aws-course-transit-gateway-attachment-configuration.png)

The AWS Course VPC attachment became available:

![AWS Course Transit Gateway Attachment Available](screenshots/98-aws-course-transit-gateway-attachment-available.png)

#### Demo Course VPC Attachment

![Demo Course Transit Gateway Attachment Configuration](screenshots/99-demo-course-transit-gateway-attachment-configuration.png)

The Demo Course VPC attachment became available:

![Demo Course Transit Gateway Attachment Available](screenshots/100-demo-course-transit-gateway-attachment-available.png)

#### Redshift VPC Attachment

![Redshift Transit Gateway Attachment Configuration](screenshots/101-redshift-transit-gateway-attachment-configuration.png)

The Redshift VPC attachment became available:

![Redshift Transit Gateway Attachment Available](screenshots/102-redshift-transit-gateway-attachment-available.png)

The final Transit Gateway attachment configuration showed all three VPCs connected:

![Transit Gateway All Attachments](screenshots/103-transit-gateway-all-attachments.png)

---

### Transit Gateway Route Configuration

The route tables of each VPC were updated to send traffic destined for remote VPC CIDRs through the Transit Gateway.

#### AWS Course VPC

The AWS Course public route table was configured with:

```text
71.0.0.0/16 → Transit Gateway
10.0.0.0/16 → Transit Gateway
```

![AWS Course Public Route Table Transit Gateway Routes](screenshots/104-aws-course-public-route-table-transit-gateway-routes.png)

The private route table was configured similarly:

![AWS Course Private Route Table Transit Gateway Routes](screenshots/105-aws-course-private-route-table-transit-gateway-routes.png)

#### Demo Course VPC

The Demo Course VPC public route table was configured with:

```text
31.0.0.0/16 → Transit Gateway
10.0.0.0/16 → Transit Gateway
```

![Demo Public Route Table Transit Gateway Routes](screenshots/106-demo-public-route-table-transit-gateway-routes.png)

The private route table was also updated:

![Demo Private Route Table Transit Gateway Routes](screenshots/107-demo-private-route-table-transit-gateway-routes.png)

#### Redshift VPC

The Redshift public route table was configured with:

```text
31.0.0.0/16 → Transit Gateway
71.0.0.0/16 → Transit Gateway
```

![Redshift Public Route Table Transit Gateway Routes](screenshots/108-redshift-public-route-table-transit-gateway-routes.png)

The private route table was also updated:

![Redshift Private Route Table Transit Gateway Routes](screenshots/109-redshift-private-route-table-transit-gateway-routes.png)

The final Transit Gateway routing configuration was verified:

![Final Transit Gateway Routing Configuration](screenshots/110-final-transit-gateway-routing-configuration.png)

---

### EC2 Instances in the Three VPCs

An EC2 instance was launched in each VPC for connectivity testing.

#### AWS Course EC2

![AWS Course EC2 Network Configuration](screenshots/111-aws-course-ec2-network-configuration.png)

![AWS Course EC2 Running](screenshots/112-aws-course-ec2-running.png)

#### Demo Course EC2

![Demo Course EC2 Network Configuration](screenshots/113-demo-course-ec2-network-configuration.png)

![Demo Course EC2 Running](screenshots/114-demo-course-ec2-running.png)

#### Redshift EC2

![Redshift EC2 Network Configuration](screenshots/115-redshift-ec2-network-configuration.png)

![Redshift EC2 Running](screenshots/116-redshift-ec2-running.png)

The private IPv4 addresses of the three EC2 instances were verified:

![Three VPC EC2 Private IP Details](screenshots/117-three-vpc-ec2-private-ip-details.png)

---

### Transit Gateway Connectivity Tests

The Transit Gateway connectivity was tested using the **private IPv4 addresses** of the EC2 instances.

#### AWS Course → Demo Course

![AWS Course to Demo Ping Success](screenshots/118-aws-course-to-demo-ping-success.png)

#### AWS Course → Redshift

![AWS Course to Redshift Ping Success](screenshots/119-aws-course-to-redshift-ping-success.png)

#### Demo Course → AWS Course

![Demo to AWS Course Ping Success](screenshots/120-demo-to-aws-course-ping-success.png)

#### Demo Course → Redshift

![Demo to Redshift Ping Success](screenshots/121-demo-to-redshift-ping-success.png)

#### Redshift → AWS Course

![Redshift to AWS Course Ping Success](screenshots/122-redshift-to-aws-course-ping-success.png)

#### Redshift → Demo Course

![Redshift to Demo Ping Success](screenshots/123-redshift-to-demo-ping-success.png)

These tests demonstrated bidirectional private connectivity between all three VPCs through the centralized Transit Gateway.

---

### Transit Gateway Negative Test

To demonstrate the importance of route-table configuration, the Transit Gateway routes were removed from the AWS Course VPC route table.

![Transit Gateway Route Removed Negative Test](screenshots/124-transit-gateway-route-removed-negative-test.png)

After removing the required route, the AWS Course EC2 instance could no longer reach the remote EC2 instance.

![Transit Gateway Connectivity Blocked](screenshots/125-transit-gateway-connectivity-blocked.png)

This demonstrates:

```text
Transit Gateway Attachment
        ≠
Automatic VPC Routing
```

The VPC route table must contain the appropriate remote CIDR route pointing to the Transit Gateway.

After restoring the required route, Transit Gateway connectivity was successfully restored:

![Final Transit Gateway Connectivity](screenshots/126-final-transit-gateway-connectivity.png)

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

---

## 📚 Detailed Implementation

Detailed step-by-step documentation is available for each part of the project:

1. [VPC Creation and Configuration](docs/01-vpc.md)
2. [Public and Private Subnet Configuration](docs/02-subnets.md)
3. [Internet Gateway Configuration](docs/03-internet-gateway.md)
4. [Route Table Configuration](docs/04-route-tables.md)
5. [NAT Gateway Configuration](docs/05-nat-gateway.md)
6. [EC2 Instances in Public and Private Subnets](docs/06-ec2-in-vpc.md)
7. [EC2 User Data and Apache Automation](docs/07-ec2-user-data.md)
8. [Security Groups](docs/08-security-groups.md)
9. [Application Load Balancer and Target Groups](docs/09-application-load-balancer.md)
10. [VPC Peering](docs/10-vpc-peering.md)
11. [Transit Gateway and Transit Gateway Attachments](docs/11-transit-gateway.md)

Each section includes explanations of the networking concept, configuration details, implementation steps, traffic-flow explanations, validation, and AWS Console screenshots.

---

## 📁 Repository Structure

```text
aws-vpc-networking-basics/
│
├── README.md
├── LICENSE
│
├── architecture/
│   ├── AWS-VPC-Subnet-Routing-Architecture.png
│   ├── AWS-VPC-EC2-NAT-Gateway-Architecture.png
│   ├── AWS-VPC-EC2-ALB-Architecture.png
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
│   └── 11-transit-gateway.md
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
    ├── 12-final-vpc-resource-map.png
    ├── 13-nat-gateway-configuration.png
    ├── 14-nat-gateway-available.png
    ├── 15-private-route-table-nat-route-configuration.png
    ├── 16-private-route-table-nat-route.png
    ├── 17-public-ec2-network-configuration.png
    ├── 18-public-ec2-running.png
    ├── 19-public-ec2-network-details.png
    ├── 20-public-ec2-ssh-connection.png
    ├── 21-private-ec2-network-configuration.png
    ├── 22-private-ec2-running.png
    ├── 23-private-ec2-ssh-via-public-ec2.png
    ├── 24-private-ec2-internet-test.png
    ├── 25-final-vpc-resource-map.png
    ├── 26-ec2-user-data-configuration.png
    ├── 27-user-data-ec2-running.png
    ├── 28-apache-service-running.png
    ├── 29-user-data-localhost-test.png
    ├── 30-user-data-browser-test.png
    ├── 31-security-group-configuration.png
    ├── 32-security-group-inbound-http.png
    ├── 33-security-group-outbound-rules.png
    ├── 34-target-group-configuration.png
    ├── 35-target-group-registered-targets.png
    ├── 36-target-group-health-status.png
    ├── 37-application-load-balancer-configuration.png
    ├── 38-alb-network-configuration.png
    ├── 39-alb-security-group-http-rule.png
    ├── 40-alb-listener-target-group.png
    ├── 41-alb-dns-details.png
    ├── 42-alb-application-test.png
    ├── 43-second-public-ec2-configuration.png
    ├── 44-second-public-ec2-running.png
    ├── 45-second-public-ec2-application-test.png
    ├── 46-target-group-two-public-ec2.png
    ├── 47-target-group-final-health-status.png
    ├── 48-alb-backend-response-1.png
    ├── 49-alb-backend-response-2.png
    ├── 50-virginia-vpc-configuration.png
    ├── 51-virginia-public-subnet-configuration.png
    ├── 52-virginia-private-subnet-configuration.png
    ├── 53-virginia-internet-gateway.png
    ├── 54-virginia-route-table-configuration.png
    ├── 55-vpc-peering-request-configuration.png
    ├── 56-vpc-peering-pending-acceptance.png
    ├── 57-vpc-peering-acceptance.png
    ├── 58-vpc-peering-active.png
    ├── 59-mumbai-ec2-network-configuration.png
    ├── 60-virginia-ec2-network-configuration.png
    ├── 61-mumbai-ec2-running.png
    ├── 62-virginia-ec2-running.png
    ├── 63-mumbai-to-virginia-ping-failed.png
    ├── 64-mumbai-route-table-peering-route.png
    ├── 65-virginia-route-table-peering-route.png
    ├── 66-mumbai-to-virginia-ping-success.png
    ├── 67-virginia-to-mumbai-ping-success.png
    ├── 68-vpc-71-configuration.png
    ├── 69-vpc-71-created.png
    ├── 70-vpc-10-configuration.png
    ├── 71-vpc-10-created.png
    ├── 72-demo-public-subnet-configuration.png
    ├── 73-demo-public-subnet-created.png
    ├── 74-demo-private-subnet-configuration.png
    ├── 75-demo-private-subnet-created.png
    ├── 76-demo-internet-gateway-creation.png
    ├── 77-demo-internet-gateway-attached.png
    ├── 78-demo-public-route-table-creation.png
    ├── 79-demo-public-route-table-internet-route.png
    ├── 80-demo-public-subnet-route-table-association.png
    ├── 81-demo-private-route-table-creation.png
    ├── 82-demo-private-subnet-route-table-association.png
    ├── 83-redshift-public-subnet-configuration.png
    ├── 84-redshift-public-subnet-created.png
    ├── 85-redshift-private-subnet-configuration.png
    ├── 86-redshift-private-subnet-created.png
    ├── 87-redshift-internet-gateway-creation.png
    ├── 88-redshift-internet-gateway-attached.png
    ├── 89-redshift-public-route-table-creation.png
    ├── 90-redshift-public-route-table-internet-route.png
    ├── 91-redshift-public-subnet-route-table-association.png
    ├── 92-redshift-private-route-table-creation.png
    ├── 93-redshift-private-subnet-route-table-association.png
    ├── 94-three-vpc-resource-map.png
    ├── 95-transit-gateway-configuration.png
    ├── 96-transit-gateway-created.png
    ├── 97-aws-course-transit-gateway-attachment-configuration.png
    ├── 98-aws-course-transit-gateway-attachment-available.png
    ├── 99-demo-course-transit-gateway-attachment-configuration.png
    ├── 100-demo-course-transit-gateway-attachment-available.png
    ├── 101-redshift-transit-gateway-attachment-configuration.png
    ├── 102-redshift-transit-gateway-attachment-available.png
    ├── 103-transit-gateway-all-attachments.png
    ├── 104-aws-course-public-route-table-transit-gateway-routes.png
    ├── 105-aws-course-private-route-table-transit-gateway-routes.png
    ├── 106-demo-public-route-table-transit-gateway-routes.png
    ├── 107-demo-private-route-table-transit-gateway-routes.png
    ├── 108-redshift-public-route-table-transit-gateway-routes.png
    ├── 109-redshift-private-route-table-transit-gateway-routes.png
    ├── 110-final-transit-gateway-routing-configuration.png
    ├── 111-aws-course-ec2-network-configuration.png
    ├── 112-aws-course-ec2-running.png
    ├── 113-demo-course-ec2-network-configuration.png
    ├── 114-demo-course-ec2-running.png
    ├── 115-redshift-ec2-network-configuration.png
    ├── 116-redshift-ec2-running.png
    ├── 117-three-vpc-ec2-private-ip-details.png
    ├── 118-aws-course-to-demo-ping-success.png
    ├── 119-aws-course-to-redshift-ping-success.png
    ├── 120-demo-to-aws-course-ping-success.png
    ├── 121-demo-to-redshift-ping-success.png
    ├── 122-redshift-to-aws-course-ping-success.png
    ├── 123-redshift-to-demo-ping-success.png
    ├── 124-transit-gateway-route-removed-negative-test.png
    ├── 125-transit-gateway-connectivity-blocked.png
    └── 126-final-transit-gateway-connectivity.png
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
- Longest prefix matching
- Subnet-to-route-table associations
- Public vs. private subnet routing
- NAT Gateway configuration
- Private subnet outbound Internet connectivity
- Public and private EC2 deployment
- Public vs. private IPv4 addressing
- SSH connectivity
- Accessing private resources through a public instance
- Testing network connectivity
- AWS VPC Resource Map
- AWS Security Groups
- Security Group inbound rules
- Security Group outbound rules
- Stateful Security Groups
- HTTP access on TCP port `80`
- SSH access on TCP port `22`
- Security Groups vs. route tables
- EC2 User Data and instance bootstrapping
- Automated Apache installation
- Automated server initialization using Bash
- HTTP service validation using `curl`
- Application Load Balancers
- Target Groups
- EC2 target registration
- ALB listeners
- ALB security groups
- Target health checks
- Healthy and unhealthy targets
- Internet-facing load balancers
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
- Comparing VPC Peering with Transit Gateway architecture

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

NAT Gateway, public IPv4 resources, EC2 instances, Transit Gateway resources, and other running AWS resources can incur charges, so temporary lab resources should not be left running unnecessarily.

---

## ⚠️ Note

This project is intended for educational and hands-on AWS networking practice.

The architecture focuses on understanding AWS VPC networking concepts and is **not intended to represent a complete production environment**.

The VPC CIDR `31.0.0.0/16` follows the addressing used during the training lab. For real-world private VPC designs, RFC 1918 private address ranges such as `10.0.0.0/8`, `172.16.0.0/12`, or `192.168.0.0/16` would normally be used.

The Security Group configuration used in this lab allows SSH and HTTP from `0.0.0.0/0` for learning and testing purposes. Restricting administrative access to trusted source addresses or using managed access mechanisms is preferable in production environments.

The Application Load Balancer and Target Group configuration in this project is intended for learning and demonstration purposes.

The VPC Peering configuration is also intended for learning and demonstration purposes. VPC Peering is a one-to-one connection and does not provide transitive routing between multiple VPCs.

The Transit Gateway configuration is also intended for learning and demonstration purposes. Transit Gateway provides centralized connectivity between multiple attached VPCs, but the appropriate route-table configuration is still required for traffic to reach the intended destination.

The project will continue to evolve as additional AWS networking concepts are implemented.