# AWS VPC Networking Fundamentals

A hands-on AWS networking project demonstrating the design, implementation, and testing of a custom Amazon VPC with public and private subnets, an Internet Gateway, custom route tables, a NAT Gateway, EC2 instances deployed across the network, and Security Groups controlling EC2 traffic.

This project was created to develop a practical understanding of AWS networking fundamentals, subnet routing, public and private EC2 connectivity, private subnet outbound Internet access through a NAT Gateway, Security Groups, and automated EC2 initialization using User Data.

---

## 🏗️ Architecture

![AWS VPC Network Architecture](architecture/AWS-VPC-EC2-NAT-Gateway-Architecture.png)

### Architecture Overview

The network is deployed in the **Asia Pacific (Mumbai) `ap-south-1`** AWS Region and currently consists of:

- One custom VPC with CIDR block `31.0.0.0/16`
- One public subnet in `ap-south-1a`
- One private subnet in `ap-south-1b`
- One Internet Gateway attached to the VPC
- One custom public route table
- One custom private route table
- One NAT Gateway for private subnet outbound Internet connectivity
- EC2 instances deployed in the public and private subnets
- Security Group controlling inbound and outbound traffic for the public Apache EC2 instance
- EC2 User Data used to automate Apache web server configuration

The public subnet uses a default route to the Internet Gateway.

The private subnet does **not** have a direct route to the Internet Gateway. Instead, Internet-bound traffic from the private subnet is sent to the NAT Gateway.

The public EC2 instance can be accessed externally using SSH, while the private EC2 instance is accessed through the public EC2 instance using its private IPv4 address.

---

## 🎯 Project Objectives

The objective of this project is to gain hands-on experience with core AWS VPC networking concepts, including:

- Creating a custom Amazon VPC
- Understanding IPv4 CIDR blocks
- Creating public and private subnets
- Deploying subnets across Availability Zones
- Creating and attaching an Internet Gateway
- Creating custom route tables
- Configuring public Internet routing
- Associating route tables with subnets
- Creating and configuring a NAT Gateway
- Providing outbound Internet connectivity to a private subnet
- Launching EC2 instances into specific VPC subnets
- Understanding public and private IPv4 addressing
- Connecting to EC2 instances using SSH
- Accessing a private EC2 instance through a public EC2 instance
- Validating outbound Internet connectivity from a private EC2 instance
- Configuring AWS Security Groups
- Understanding inbound and outbound Security Group rules
- Allowing HTTP traffic on TCP port `80`
- Allowing SSH traffic on TCP port `22`
- Automating EC2 initialization using User Data
- Automatically installing and configuring Apache
- Validating web server deployment using `curl` and a browser

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
| Public Default Route | `0.0.0.0/0 → Internet Gateway` |
| Private Default Route | `0.0.0.0/0 → NAT Gateway` |
| Public EC2 | Public + Private IPv4 |
| Private EC2 | Private IPv4 only |
| Private EC2 Internet Access | NAT Gateway |
| Public EC2 HTTP Access | TCP `80` |
| Public EC2 SSH Access | TCP `22` |

---

## 🧩 AWS Networking Components

### Amazon VPC

The VPC provides a logically isolated virtual network in AWS where networking resources can be created and configured.

**VPC CIDR:**

```text
31.0.0.0/16
```

[View VPC implementation →](docs/01-vpc.md)

---

### Public and Private Subnets

The VPC is divided into two subnets.

#### Public Subnet

```text
CIDR: 31.0.1.0/24
Availability Zone: ap-south-1a
```

The public subnet is associated with a route table containing a default route to the Internet Gateway.

#### Private Subnet

```text
CIDR: 31.0.2.0/24
Availability Zone: ap-south-1b
```

The private subnet does not have a direct route to the Internet Gateway.

Instead, its default route points to the NAT Gateway for outbound Internet connectivity.

[View subnet implementation →](docs/02-subnets.md)

---

### Internet Gateway

An Internet Gateway is created and attached to the VPC to provide a path between the VPC and the Internet.

Attaching an Internet Gateway to a VPC alone does not automatically make a subnet public.

The public subnet must use a route table containing:

```text
0.0.0.0/0 → Internet Gateway
```

[View Internet Gateway implementation →](docs/03-internet-gateway.md)

---

### Route Tables

Two custom route tables are used to provide different routing behavior for the public and private subnets.

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

The private route table was initially created with only the VPC local route.

After introducing the NAT Gateway, it was extended to:

| Destination | Target |
|---|---|
| `31.0.0.0/16` | `local` |
| `0.0.0.0/0` | NAT Gateway |

Associated with:

```text
31.0.2.0/24 — Private Subnet
```

The private subnet therefore has outbound Internet connectivity without receiving a direct route to the Internet Gateway.

[View route table implementation →](docs/04-route-tables.md)

---

### NAT Gateway

A NAT Gateway provides outbound Internet connectivity for resources deployed inside the private subnet.

The private route table contains:

```text
0.0.0.0/0 → NAT Gateway
```

Internet-bound traffic from the private subnet therefore follows:

```text
Private Resource
       ↓
Private Route Table
       ↓
NAT Gateway
       ↓
Internet Gateway
       ↓
Internet
```

This allows a private EC2 instance to initiate connections to the Internet without requiring its own public IPv4 address.

[View NAT Gateway implementation →](docs/05-nat-gateway.md)

---

### EC2 Instances in the VPC

Two EC2 instances were deployed to validate the public and private subnet networking architecture.

#### Public EC2

The public EC2 instance was deployed inside:

```text
Public Subnet
31.0.1.0/24
```

It has:

```text
Private IPv4: 31.0.1.161
Public IPv4:  Assigned by AWS
```

The public IPv4 address allows the instance to be accessed externally using SSH when the required network access configuration is present.

#### Private EC2

The private EC2 instance was deployed inside:

```text
Private Subnet
31.0.2.0/24
```

It has:

```text
Private IPv4: 31.0.2.152
Public IPv4:  None
```

Because the instance has no public IPv4 address, it was accessed through the public EC2 instance using VPC local routing.

[View EC2 implementation →](docs/06-ec2-in-vpc.md)

---

### Security Groups

A Security Group acts as a virtual firewall for the EC2 instance and controls inbound and outbound network traffic.

The Security Group was configured for the public EC2 instance running Apache.

The inbound rules allow:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| SSH | TCP | `22` | `0.0.0.0/0` |
| HTTP | TCP | `80` | `0.0.0.0/0` |

The outbound rule allows all IPv4 traffic:

```text
All traffic → 0.0.0.0/0
```

The HTTP rule allows external clients to reach the Apache web server on port `80`.

The Security Group is a traffic-control layer attached to the EC2 instance and works together with the route table.

[View Security Group implementation →](docs/08-security-groups.md)

---

### EC2 User Data

EC2 User Data was used to automate the initial configuration of a public EC2 instance.

A Bash script was supplied during instance launch to automatically:

- Update the package repository
- Install Apache
- Generate a custom HTML web page
- Display the instance hostname and private IPv4 address
- Restart the Apache service

The deployment was validated by checking the Apache service:

```bash
sudo systemctl status apache2
```

The web server was then tested locally using:

```bash
curl localhost
```

Finally, the instance's public IPv4 address was opened in a web browser to verify that Apache was serving the generated page externally.

[View EC2 User Data implementation →](docs/07-ec2-user-data.md)

---

## 🔄 Traffic Flow

The completed architecture demonstrates several different networking paths.

### Public EC2 → Internet

Internet-bound traffic from the public EC2 instance follows:

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

The private EC2 instance is accessed through the public EC2 instance.

```text
Administrator
      ↓
     SSH
      ↓
Public EC2
31.0.1.161
      ↓
VPC Local Routing
      ↓
Private EC2
31.0.2.152
```

Both addresses belong to:

```text
31.0.0.0/16
```

Therefore, the VPC local route provides the routing path between the two subnet networks.

---

### Private EC2 → Internet

The private EC2 instance does not have a public IPv4 address and does not have a direct Internet Gateway route.

Its outbound Internet traffic follows:

```text
Private EC2
31.0.2.152
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

This allows the private EC2 instance to initiate outbound Internet connections while remaining without a public IPv4 address.

---

### Internet → Public EC2 → Apache

HTTP traffic to the Apache web server follows:

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

The Security Group determines whether the incoming HTTP traffic is allowed to reach the EC2 instance.

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

The test successfully received responses with `0% packet loss`.

![Private EC2 Internet Test](screenshots/24-private-ec2-internet-test.png)

This validates the path:

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

The EC2 instance launched with User Data successfully completed its automated initialization.

Apache was verified as running:

![Apache Service Running](screenshots/28-apache-service-running.png)

The generated page was tested locally using `curl localhost`:

![EC2 User Data Localhost Test](screenshots/29-user-data-localhost-test.png)

The web server was also successfully accessed using the EC2 instance's public IPv4 address from a browser:

![EC2 User Data Browser Test](screenshots/30-user-data-browser-test.png)

This confirms that the User Data script successfully installed Apache and generated the custom web page during instance initialization.

---

### Security Group Configuration

The public Apache EC2 instance was configured with a Security Group controlling its network access.

The Security Group configuration was reviewed:

![Security Group Configuration](screenshots/31-security-group-configuration.png)

HTTP traffic on TCP port `80` was allowed through the inbound rules:

![Security Group Inbound HTTP](screenshots/32-security-group-inbound-http.png)

The outbound rules were also reviewed:

![Security Group Outbound Rules](screenshots/33-security-group-outbound-rules.png)

The resulting HTTP access path is:

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
EC2
     ↓
Apache
```

---

## 📸 Final AWS Resource Map

The AWS VPC Resource Map below shows the updated networking relationships after adding the NAT Gateway and extending the VPC architecture.

![AWS VPC Resource Map](screenshots/25-final-vpc-resource-map.png)

An earlier Resource Map representing the architecture before the NAT Gateway and EC2 extension is preserved in the route table documentation to show the progression of the lab.

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

Each section includes explanations of the networking concept, configuration details, implementation steps, traffic-flow explanations, and AWS Console screenshots.

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
│   └── AWS-VPC-EC2-NAT-Gateway-Architecture.png
│
├── docs/
│   ├── 01-vpc.md
│   ├── 02-subnets.md
│   ├── 03-internet-gateway.md
│   ├── 04-route-tables.md
│   ├── 05-nat-gateway.md
│   ├── 06-ec2-in-vpc.md
│   ├── 07-ec2-user-data.md
│   └── 08-security-groups.md
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
    └── 33-security-group-outbound-rules.png
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

EC2 User Data additionally demonstrates how instance initialization tasks can be automated during launch instead of being performed manually after connecting to the server.

---

## 🧹 Resource Cleanup

AWS resources created for hands-on practice should be removed after completing the lab when they are no longer required.

Resources created throughout this project currently include:

- Custom VPC
- Public subnet
- Private subnet
- Internet Gateway
- Public route table
- Private route table
- NAT Gateway
- NAT Gateway public IP / Elastic IP resources where applicable
- Public EC2 instances
- Private EC2 instance
- Security Groups

NAT Gateway and public IPv4 resources can incur charges, so temporary lab resources should not be left running unnecessarily.

---

## ⚠️ Note

This project is intended for educational and hands-on AWS networking practice.

The architecture focuses on understanding AWS VPC networking concepts and is **not intended to represent a complete production environment**.

The VPC CIDR `31.0.0.0/16` follows the addressing used during the training lab. For real-world private VPC designs, RFC 1918 private address ranges such as `10.0.0.0/8`, `172.16.0.0/12`, or `192.168.0.0/16` would normally be used.

The Security Group configuration used in this lab allows SSH and HTTP from `0.0.0.0/0` for learning and testing purposes. Restricting administrative access to trusted source addresses or using managed access mechanisms is preferable in production environments.

The project will continue to evolve as additional AWS networking concepts are implemented.