# Application Load Balancer and Target Groups

## Overview

In the previous sections of this project, the VPC, subnets, route tables, Internet Gateway, NAT Gateway, security groups, and EC2 instances were configured.

In this section, the architecture is extended by introducing an **Application Load Balancer (ALB)** and a **Target Group**.

The purpose of the Application Load Balancer is to accept incoming HTTP traffic from users and distribute that traffic across healthy EC2 instances registered in a Target Group.

The architecture is extended from:

```text
    Internet
        |
        v
    Public EC2
```

to:

```text
    Internet
        |
        v
    Application Load Balancer
        |
        v
    Target Group
        |
        +-------------------+
        |                   |
        v                   v
    Public EC2 #1       Public EC2 #2
```

The project also demonstrates what happens when a private EC2 instance is registered in the same Target Group.

The private EC2 instance becomes unhealthy because the Application Load Balancer cannot successfully reach its HTTP service through the required network path.

---

# 1. What is an Application Load Balancer?

An **Application Load Balancer (ALB)** is an AWS load balancing service that operates at the **application layer (Layer 7)**.

It is designed to receive application traffic such as HTTP and HTTPS requests and distribute those requests across multiple backend targets.

In this project, the Application Load Balancer acts as the **central entry point** for the application.

Instead of users directly accessing individual EC2 instances, they access the ALB using its DNS name.

The ALB then forwards the request to a healthy backend EC2 instance registered in a Target Group.

The basic architecture is:

```text
                    Internet
                       |
                       | HTTP
                       v
             +----------------------+
             | Application Load     |
             | Balancer             |
             +----------+-----------+
                        |
                        v
                +---------------+
                | Target Group  |
                +-------+-------+
                        |
              +---------+---------+
              |                   |
              v                   v
        Public EC2 #1       Public EC2 #2
```

The ALB therefore provides a single endpoint through which users can access an application running on multiple backend servers.

---

# 2. Why Do We Need an Application Load Balancer?

Without a load balancer, users would need to access individual EC2 instances directly.

For example:

```text
User
 |
 +----> EC2 #1
 |
 +----> EC2 #2
```

This creates several problems.

### 1. Users need to know individual server addresses

Users would have to access the public IP address or DNS name of a specific EC2 instance.

### 2. Traffic is not centrally distributed

There is no single component responsible for distributing incoming requests between multiple backend servers.

### 3. Backend availability becomes important

If a particular EC2 instance becomes unavailable, users directly accessing that instance cannot reach the application.

### 4. Adding more servers becomes difficult

When additional EC2 instances are created, users should not have to manually discover and use every new server address.

An Application Load Balancer solves these problems by providing a **single entry point** and distributing traffic across healthy backend targets.

The architecture becomes:

```text
                    User
                     |
                     v
             Application Load
                Balancer
                     |
                     v
                Target Group
                     |
             +-------+-------+
             |               |
             v               v
          EC2 #1           EC2 #2
```

The user only needs to know the ALB DNS name.

---

# 3. How Does an Application Load Balancer Work?

The Application Load Balancer works together with several AWS components.

The main components used in this project are:

- Application Load Balancer
- Listener
- Target Group
- EC2 instances
- Security Groups
- Subnets
- Route Tables
- Health Checks

The traffic flow is:

```text
Client
  |
  | HTTP request
  v
Application Load Balancer
  |
  | Listener : HTTP 80
  v
Target Group
  |
  +----> Healthy EC2 #1
  |
  +----> Healthy EC2 #2
```

### Listener

The listener waits for incoming traffic on a configured protocol and port.

In this project:

```text
Protocol : HTTP
Port     : 80
```

### Target Group

The Target Group contains the backend resources that can receive traffic from the ALB.

In this project, the targets are EC2 instances.

### Health Check

The ALB continuously checks the health of registered targets.

Only healthy targets are considered available for normal traffic routing.

Therefore:

```text
ALB
 |
 +----> Healthy Target    → Can receive traffic
 |
 +----> Unhealthy Target  → Not selected for normal traffic
```

This allows the load balancer to route traffic toward available backend servers.

---

# 4. What Problem Are We Solving?

The objective of this section is to extend the existing AWS VPC architecture from direct EC2 access to a **centralized application entry point using an Application Load Balancer**.

Initially, the application can be accessed directly:

```text
Internet
   |
   v
Public EC2
```

The problem is that the application is directly dependent on an individual EC2 instance.

We want to create an architecture where:

```text
Internet
   |
   v
Application Load Balancer
   |
   v
Target Group
   |
   +----> Public EC2 #1
   |
   +----> Public EC2 #2
```

The ALB should:

- Receive HTTP traffic from users.
- Forward requests to backend EC2 instances.
- Perform health checks.
- Route traffic to healthy targets.
- Provide a single DNS endpoint for the application.
- Allow multiple EC2 instances to serve the same application.

This project also intentionally registers a private EC2 instance in the Target Group so that the health status of public and private targets can be observed.

The final implementation demonstrates:

```text
Internet
    |
    v
Application Load Balancer
    |
    v
Target Group
    |
    +----> Public EC2 #1 → Healthy
    |
    +----> Public EC2 #2 → Healthy
    |
    └----> Private EC2   → Unhealthy
```

---

## 🎯 Objective

The objectives of this section were:

- Understand the purpose of an Application Load Balancer.
- Understand what a Target Group is.
- Create a Target Group for EC2 instances.
- Register EC2 instances as targets.
- Configure HTTP traffic on port `80`.
- Configure health checks for the registered targets.
- Create an internet-facing Application Load Balancer.
- Configure the ALB inside the existing custom VPC.
- Configure the ALB listener to forward traffic to the Target Group.
- Verify the ALB using its DNS name.
- Add a second public EC2 instance.
- Demonstrate traffic reaching different healthy EC2 instances through the same ALB.
- Observe the health status of public and private EC2 instances.

---

# Part 1 – Understanding the Target Group

## What is a Target Group?

A Target Group is a logical grouping of resources that can receive traffic from a load balancer.

In this lab, the targets are **EC2 instances**.

The Target Group was configured to use:

```text
    Target Type     : Instances
    Protocol        : HTTP
    Port            : 80
    IP Address Type : IPv4
    VPC             : aws-course-mu1-vpc
    Health Check    : HTTP
    Health Path     : /
```

The Target Group provides the collection of backend resources that the Application Load Balancer can route requests to.

The traffic flow is:

```text
    Client
       |
       v
    Application Load Balancer
       |
       v
    Target Group
       |
       +----------------------+
       |                      |
       v                      v
    EC2 Instance #1       EC2 Instance #2
```

---

## Step 1 – Create the Target Group

A new Target Group was created from the EC2 console.

The target type was set to:

```text
Instances
```

because the backend resources used in this lab are EC2 instances.

The communication protocol was configured as:

```text
HTTP
```

and the target port was configured as:

```text
80
```

The existing custom VPC was selected:

```text
aws-course-mu1-vpc
31.0.0.0/16
```

The health check was configured using HTTP with the default path:

```text
/
```

### Target Group Configuration

![Target Group Configuration](../screenshots/34-target-group-configuration.png)

The Target Group is therefore configured to send HTTP traffic to the registered EC2 instances on port `80`.

---

# Part 2 – Registering EC2 Instances

## Step 2 – Register the Initial Targets

The Target Group was configured with the existing EC2 instances.

At this stage, both the public and private EC2 instances were selected as targets.

```text
    Public EC2
        |
        +---- Port 80

    Private EC2
        |
        +---- Port 80
```

Both instances were registered with the Target Group.

### Registered Targets

![Target Group Registered Targets](../screenshots/35-target-group-registered-targets.png)

The Target Group now contains two registered EC2 instances.

The important point is that a Target Group can contain multiple backend resources.

---

## Step 3 – Initial Target Group Status

After registering the instances, the Target Group displayed the registered targets.

Before the Application Load Balancer was associated with the Target Group, the targets were not yet being actively used by a load balancer.

### Initial Target Group Status

![Target Group Health Status](../screenshots/36-target-group-health-status.png)

The Target Group is now ready to be associated with an Application Load Balancer.

---

# Part 3 – Creating the Application Load Balancer

## What is an Application Load Balancer?

An **Application Load Balancer** operates at the application layer and is designed to distribute HTTP and HTTPS traffic across multiple targets.

In this project, the ALB acts as the internet-facing entry point.

The architecture becomes:

```text
    Internet User
          |
          | HTTP
          v
    Application Load Balancer
          |
          v
    Target Group
          |
          +-------------------+
          |                   |
          v                   v
    Public EC2           Private EC2
```

The ALB receives the external request and forwards it to a registered target in the Target Group according to its listener and load-balancing configuration.

---

## Step 4 – Configure the Application Load Balancer

An **Application Load Balancer** was selected from the EC2 Load Balancers section.

The basic configuration was:

```text
    Name   : aws-course-mu-alb
    Scheme : Internet-facing
    IP     : IPv4
```

The internet-facing scheme allows the load balancer to receive traffic from users over the internet.

### Application Load Balancer Configuration

![Application Load Balancer Configuration](../screenshots/37-application-load-balancer-configuration.png)

The ALB was configured as an IPv4 internet-facing Application Load Balancer.

---

## Step 5 – Configure ALB Network Mapping

The existing custom VPC was selected:

```text
VPC
```

```text
    aws-course-mu1-vpc
```

CIDR:

```text
    31.0.0.0/16
```

The ALB was configured across the available Availability Zones and their associated subnets.

The public subnet contains the Internet Gateway route and is suitable for internet-facing ALB connectivity.

The private subnet does not have an Internet Gateway route.

### ALB Network Configuration

![ALB Network Configuration](../screenshots/38-alb-network-configuration.png)

AWS displayed a reachability warning for the private subnet because its route table does not provide a direct Internet Gateway path.

This is consistent with the private subnet design used throughout this project.

---

# Part 4 – ALB Security Group

## Step 6 – Allow HTTP Traffic

The Application Load Balancer requires a security group that allows incoming HTTP traffic.

The security group was configured to allow:

```text
    Protocol : TCP
    Port     : 80
    Source   : 0.0.0.0/0
```

This allows HTTP requests from IPv4 clients on the internet to reach the ALB.

### ALB Security Group HTTP Rule

![ALB Security Group HTTP Rule](../screenshots/39-alb-security-group-http-rule.png)

The HTTP port `80` rule is required because the Application Load Balancer is serving HTTP traffic.

---

# Part 5 – Configure the ALB Listener

## Step 7 – HTTP Listener and Target Group

The Application Load Balancer was configured with an HTTP listener:

```text
    Protocol : HTTP
    Port     : 80
```

The default routing action was configured as:

```text
    Forward to target groups
```

The previously created Target Group was selected:

```text
    aws-course-mu1-target-group
```

The routing weight was:

```text
    Weight : 1
```

which represents:

```text
    100%
```

of the traffic assigned to this Target Group.

### ALB Listener and Target Group

![ALB Listener Target Group](../screenshots/40-alb-listener-target-group.png)

The listener therefore follows this path:

```text
    HTTP : 80
        |
        v
    Target Group
        |
        v
    Registered Targets
```

---

# Part 6 – Application Load Balancer Created

## Step 8 – Verify the ALB

After the configuration was completed, the Application Load Balancer was created successfully.

The ALB reached the:

```text
    Active
```

state.

The load balancer was configured as:

```text
    Type   : Application
    Scheme : Internet-facing
    IP     : IPv4
    VPC    : aws-course-mu1-vpc
```

### ALB DNS Details

AWS automatically provided a DNS name for the Application Load Balancer.

![ALB DNS Details](../screenshots/41-alb-dns-details.png)

The DNS name is used by clients to access the Application Load Balancer.

Instead of directly accessing an EC2 public IP address, the user can access the application through the ALB DNS name.

---

# Part 7 – Test the Application Load Balancer

## Step 9 – Access the Application Through the ALB

The ALB DNS name was entered into a web browser.

The Application Load Balancer successfully returned the Apache web page running on an EC2 instance.

The page displays the backend instance information generated by the EC2 User Data script.

### ALB Application Test

![ALB Application Test](../screenshots/42-alb-application-test.png)

The response confirms that the request reached an EC2 backend through the Application Load Balancer.

The architecture is now:

```text
    Browser
       |
       | HTTP
       v
    ALB
       |
       v
    Target Group
       |
       v
    Public EC2
       |
       v
    Apache
```

---

# Part 8 – Adding a Second Public EC2 Instance

## Why Add Another Public EC2 Instance?

At this point, only one public EC2 instance was available.

To better demonstrate the load-balancing behavior, a second public EC2 instance was created.

The second instance was configured with the same basic Apache User Data script used earlier.

The purpose was to have two healthy public backend servers:

```text
    Public EC2 #1
    31.0.1.181

    Public EC2 #2
    31.0.1.141
```

This makes it possible to observe different backend responses when accessing the same ALB DNS name.

---

## Step 10 – Configure the Second Public EC2

The second EC2 instance was launched inside the existing public subnet.

The custom VPC and public subnet were selected.

The instance was also configured with HTTP access and the Apache User Data script.

### Second Public EC2 Configuration

![Second Public EC2 Configuration](../screenshots/43-second-public-ec2-configuration.png)

The User Data script automatically installs Apache and generates the custom server information page.

---

## Step 11 – Verify the Second Public EC2

After launching the instance, it successfully entered the:

```text
    Running
```

state.

The instance also passed its status checks.

### Second Public EC2 Running

![Second Public EC2 Running](../screenshots/44-second-public-ec2-running.png)

The second public EC2 instance is now available to serve HTTP traffic.

---

## Step 12 – Test the Second Public EC2 Directly

Before registering the new instance with the Target Group, its Apache server was tested directly using its public IPv4 address.

The server successfully returned its custom page.

### Second Public EC2 Application Test

![Second Public EC2 Application Test](../screenshots/45-second-public-ec2-application-test.png)

The second server is therefore independently working before being added to the load-balancing configuration.

---

# Part 9 – Add the Second Public EC2 to the Target Group

## Step 13 – Register the New Public EC2

The second public EC2 instance was registered with the existing Target Group.

The Target Group now contains three EC2 instances:

```text
    Target Group
         |
         +── Public EC2 #1
         |
         +── Public EC2 #2
         |
         └── Private EC2
```

All three targets are configured to receive traffic on:

```text
    HTTP : 80
```

### Target Group with Two Public EC2 Instances

![Target Group Two Public EC2](../screenshots/46-target-group-two-public-ec2.png)

The Target Group now provides multiple backend resources for the Application Load Balancer.

---

# Part 10 – Target Health Status

## Step 14 – Verify Target Health

The Application Load Balancer performs health checks against registered targets.

The final Target Group status showed:

```text
    Public EC2 #1 → Healthy
    Public EC2 #2 → Healthy
    Private EC2   → Unhealthy
```

### Final Target Health Status

![Target Group Final Health Status](../screenshots/47-target-group-final-health-status.png)

This is an important result of the lab.

The two public EC2 instances are reachable by the internet-facing ALB and successfully respond to the HTTP health check.

The private EC2 instance is registered in the Target Group, but it is unhealthy because the ALB does not have a working network path to the target under the routing and security configuration used in this lab.

---

# Part 11 – Demonstrating Load Balancing

## Step 15 – Test the ALB Again

With two healthy public EC2 instances registered in the Target Group, the same ALB DNS name can now be used to access both backend servers.

The ALB does not require the user to know the individual EC2 public IP addresses.

The user only accesses:

```text
    Application Load Balancer DNS
```

The ALB receives the request and routes it to a healthy target.

---

## Backend Response 1

The first request through the ALB returned the server information for:

```text
    Hostname: ip-31-0-1-141
    IP Address: 31.0.1.141
```

### ALB Backend Response 1

![ALB Backend Response 1](../screenshots/48-alb-backend-response-1.png)

This confirms that the ALB successfully routed the request to the EC2 instance with private IP:

```text
    31.0.1.141
```

---

## Backend Response 2

A subsequent request through the same ALB DNS name returned the server information for:

```text
    Hostname: ip-31-0-1-181
    IP Address: 31.0.1.181
```

### ALB Backend Response 2

![ALB Backend Response 2](../screenshots/49-alb-backend-response-2.png)

The backend response is now coming from a different healthy public EC2 instance.

This demonstrates that multiple healthy targets can serve requests through the same Application Load Balancer.

---

# 🔄 Complete Architecture

The final architecture demonstrated in this section is:

```text
                             Internet
                                |
                                |
                                v
                      +----------------------+
                      | Application Load     |
                      | Balancer              |
                      | Internet-facing       |
                      | HTTP : 80             |
                      +----------+-----------+
                                |
                                v
                      +----------------------+
                      | Target Group          |
                      | HTTP : 80             |
                      +----------+-----------+
                                |
                     +-------------+-------------+
                     |             |             |
                     v             v             v
               Public EC2 #1  Public EC2 #2  Private EC2
                 Healthy        Healthy       Unhealthy
                 :80             :80            :80
                     |             |
                     +------+------+
                            |
                          Apache
```

The two healthy public EC2 instances are available to receive traffic from the Application Load Balancer.

The private EC2 instance remains registered but unhealthy.

---

# 🔀 How the Traffic Flow Works

When a user accesses the ALB DNS name:

```text
    User
     |
     | HTTP request
     v
    Application Load Balancer
     |
     | Listener : HTTP 80
     v
    Target Group
     |
     +----> Healthy Public EC2 #1
     |
     └----> Healthy Public EC2 #2
```

The ALB selects a healthy registered target for the request.

The private EC2 instance is not selected while it remains unhealthy.

Therefore, the user does not need to directly access the EC2 instances.

The ALB provides a single entry point to the application.

---

# 🧪 Why the Private EC2 is Unhealthy

The private EC2 instance was intentionally kept in the Target Group to demonstrate the effect of network accessibility on ALB health checks.

The private EC2 instance:

```text
    Subnet        : Private
    Public IPv4   : None
    HTTP Service  : Apache
```

The public EC2 instances are reachable by the internet-facing ALB through the configured network path.

The private subnet in this lab does not have a direct Internet Gateway route, and the configured networking does not provide a working path for the ALB health check to the private target.

Therefore, the ALB cannot successfully complete its HTTP health check against the private target in this architecture.

This results in:

```text
    Public EC2 #1 → Healthy
    Public EC2 #2 → Healthy
    Private EC2   → Unhealthy
```

This demonstrates an important relationship between:

- Load balancers
- Target groups
- Security groups
- Subnets
- Route tables
- Internet connectivity
- Target health checks

---

# ⚖️ Target Group vs Application Load Balancer

These two components have different responsibilities.

| Component | Responsibility |
|---|---|
| Application Load Balancer | Receives client traffic and routes requests |
| Listener | Listens for traffic on a configured protocol and port |
| Target Group | Logically groups backend targets |
| Health Check | Determines whether a registered target is healthy |
| EC2 Instance | Processes the actual application request |

The overall relationship is:

```text
    Client
      |
      v
    ALB
      |
      | Listener
      v
    Target Group
      |
      +----> Healthy Target
      |
      +----> Healthy Target
      |
      └----> Unhealthy Target
```

---

# 🧠 Key Concepts Learned

Through this section, I gained hands-on experience with:

- Application Load Balancers
- Target Groups
- EC2 target registration
- Internet-facing load balancers
- ALB network mapping
- ALB security groups
- HTTP listeners
- Listener forwarding rules
- Target group routing
- Target health checks
- Healthy and unhealthy targets
- Load balancing across multiple EC2 instances
- ALB DNS names
- Testing backend responses
- Understanding public and private subnet behavior with an ALB
- Understanding the relationship between security groups, routing, and load balancer health checks

---

# ✅ Final Result

The Application Load Balancer was successfully created and connected to the custom VPC.

The final implementation contains:

```text
    VPC
     |
     +-- Public Subnet
     |      |
     |      +-- Public EC2 #1
     |      |
     |      +-- Public EC2 #2
     |
     +-- Private Subnet
            |
            +-- Private EC2
```

The ALB and Target Group provide the application traffic path:

```text
    Internet
        |
        v
    Application Load Balancer
        |
        v
    Target Group
        |
        +----> Public EC2 #1  [Healthy]
        |
        +----> Public EC2 #2  [Healthy]
        |
        └----> Private EC2    [Unhealthy]
```

Testing the same ALB DNS name produced responses from both public EC2 instances, confirming that the ALB is successfully routing traffic across the healthy backend targets.

The project has now progressed from manually accessing individual EC2 instances to using an **Application Load Balancer as a centralized entry point for the application**.

---

## ➡️ Next Step

Continue to the next section of the AWS VPC networking project.

---

[← Previous: Security Groups](08-security-groups.md) | [Back to Project README](../README.md)
