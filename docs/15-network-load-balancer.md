# 🌐 Network Load Balancer (NLB)

## 📌 Overview

A **Network Load Balancer (NLB)** is an AWS load balancer designed to handle network-level traffic.

In this lab, we created an **Internet-facing Network Load Balancer** that listens on **TCP port 80** and forwards traffic to two EC2 instances registered in a target group.

### Lab flow

```text
                          Internet
                             │
                             │ TCP :80
                             ▼
                   ┌─────────────────────┐
                   │ Network Load        │
                   │ Balancer (NLB)      │
                   │      NLB-Lab        │
                   └──────────┬──────────┘
                              │
                              │ TCP :80
                              ▼
                   ┌─────────────────────┐
                   │  NLB-Target-Group   │
                   └──────────┬──────────┘
                              │
                       ┌──────┴──────┐
                       │             │
                       ▼             ▼
                ┌─────────────┐ ┌─────────────┐
                │ NLB-Server-1│ │ NLB-Server-2│
                │ Apache :80  │ │ Apache :80  │
                └─────────────┘ └─────────────┘
```

The two EC2 instances return different HTML responses so that we can identify which backend receives the request.

---

# 1. 🎯 What is a Network Load Balancer?

A **Network Load Balancer (NLB)** operates at the network/transport layer and is designed to handle network traffic using protocols such as:

- TCP
- UDP
- TLS

In this lab, we use:

```text
TCP :80
```

The NLB receives a client request and forwards it to a healthy target registered in the target group.

The lecture describes NLB as working at the TCP layer and highlights its use for high-traffic applications requiring low latency.

---

# 2. 🤔 Why Do We Need a Network Load Balancer?

Suppose an application is running on two EC2 instances:

```text
EC2-1
EC2-2
```

Instead of allowing users to connect directly to individual EC2 instances, we can place a Network Load Balancer in front of them:

```text
Client
   │
   ▼
 NLB
   │
   ├── EC2-1
   │
   └── EC2-2
```

The NLB provides a single endpoint through which clients can access the backend servers.

The lecture specifically presents NLB as useful for high-traffic workloads where low latency is important.

---

# 3. 🆚 Application Load Balancer vs Network Load Balancer

Understanding the difference between ALB and NLB is important.

| Feature | Application Load Balancer | Network Load Balancer |
|---|---|---|
| Layer | Layer 7 | Layer 4 |
| Main protocols | HTTP / HTTPS | TCP / UDP / TLS |
| Path-based routing | Yes | No path-based routing in this lab |
| Example | `/api`, `/users` | TCP connection to port |
| Main focus | Application-level routing | Network-level traffic |
| Target Group | Supported | Supported |
| Example use | Web applications and APIs | High-throughput / low-latency network traffic |

### ALB example

An ALB can route requests based on paths:

```text
https://example.com/foo
https://example.com/bar
```

Different paths can be routed to different applications or target groups.

### NLB example

With the NLB used in this lab, the client connects to the NLB endpoint:

```text
NLB DNS
   │
   ▼
Target Group
   │
   ├── EC2-1
   └── EC2-2
```

The lecture specifically contrasts ALB path-based routing with NLB's network-level forwarding model.

---

# 4. 🏗️ Lab Architecture

### AWS Network Load Balancer Architecture

![AWS Network Load Balancer Architecture](../architecture/AWS-VPC-Network-Load-Balancer-Architecture.png)

---

# 5. 🧪 Lab Environment

The lab uses the existing AWS networking environment from the previous sections.

| Component | Configuration |
|---|---|
| VPC | `31.0.0.0/16` |
| Public Subnet | `31.0.1.0/24` |
| Private Subnet | `31.0.2.0/24` |
| EC2 Server 1 | `NLB-Server-1` |
| EC2 Server 2 | `NLB-Server-2` |
| Backend Application | Apache |
| Backend Port | `80` |
| Target Group | `NLB-Target-Group` |
| Target Group Protocol | TCP |
| Health Check Protocol | HTTP |
| Health Check Path | `/` |
| Load Balancer | `NLB-Lab` |
| Load Balancer Type | Network Load Balancer |
| Scheme | Internet-facing |
| Listener | TCP :80 |
| Backend Subnet | Public subnet |
| Security Group | `my-nlb-sg` |

---

# 6. 🚀 Hands-On Implementation

## Step 1 — Verify the Existing VPC

We used the existing VPC:

```text
VPC CIDR: 31.0.0.0/16
```

The VPC already had the networking components required for this lab:

- VPC
- Public subnet
- Private subnet
- Internet Gateway
- Route table

### 📸 Screenshot

![NLB VPC](../screenshots/205-nlb-vpc.png)

**Screenshot:** `205-nlb-vpc.png`

---

# 7. 🖥️ Create NLB Server 1

We launched the first EC2 instance:

```text
NLB-Server-1
```

The instance was placed in the public subnet.

The server was configured to run Apache on port `80`.

---

## Server 1 User Data

```bash
#!/bin/bash

apt update -y
apt install apache2 -y

echo "<h1>Hello from NLB Server 1</h1>" > /var/www/html/index.html

systemctl enable apache2
systemctl restart apache2
```

This gives Server 1 a unique response so that we can identify it later when testing the NLB.

### 📸 Screenshot — Server 1 Configuration

![NLB Server 1 Configuration](../screenshots/206-nlb-server-1-configuration.png)

**Screenshot:** `206-nlb-server-1-configuration.png`

---

### 📸 Screenshot — Server 1 User Data

![NLB Server 1 User Data](../screenshots/207-nlb-server-1-user-data.png)

**Screenshot:** `207-nlb-server-1-user-data.png`

---

### 📸 Screenshot — Server 1 Running

![NLB Server 1 Running](../screenshots/208-nlb-server-1-running.png)

**Screenshot:** `208-nlb-server-1-running.png`

---

# 8. 🧪 Test Server 1 Directly

Before creating the NLB, we verified that the backend server itself was working.

The Apache page returned:

```text
Hello from NLB Server 1
```

This confirmed that:

```text
Internet → EC2 Server 1 → Apache :80
```

was working independently.

### 📸 Screenshot

![NLB Server 1 Apache Test](../screenshots/209-nlb-server-1-apache-test.png)

**Screenshot:** `209-nlb-server-1-apache-test.png`

---

# 9. 🖥️ Create NLB Server 2

We created the second EC2 instance:

```text
NLB-Server-2
```

It was placed in the same public subnet and configured with Apache.

The second server uses a different HTML response.

## Server 2 User Data

```bash
#!/bin/bash

apt update -y
apt install apache2 -y

echo "<h1>Hello from NLB Server 2</h1>" > /var/www/html/index.html

systemctl enable apache2
systemctl restart apache2
```

The different response helps us identify which backend server receives traffic from the NLB.

### 📸 Screenshot — Server 2 Configuration

![NLB Server 2 Configuration](../screenshots/210-nlb-server-2-configuration.png)

**Screenshot:** `210-nlb-server-2-configuration.png`

---

### 📸 Screenshot — Server 2 User Data

![NLB Server 2 User Data](../screenshots/211-nlb-server-2-user-data.png)

**Screenshot:** `211-nlb-server-2-user-data.png`

---

### 📸 Screenshot — Server 2 Running

![NLB Server 2 Running](../screenshots/212-nlb-server-2-running.png)

**Screenshot:** `212-nlb-server-2-running.png`

---

# 10. 🧪 Test Server 2 Directly

We also tested the second backend directly.

Expected response:

```text
Hello from NLB Server 2
```

This confirmed that both backend servers were independently working before introducing the load balancer.

### 📸 Screenshot

![NLB Server 2 Apache Test](../screenshots/213-nlb-server-2-apache-test.png)

**Screenshot:** `213-nlb-server-2-apache-test.png`

---

# 11. 🎯 Create the Target Group

Go to:

```text
AWS Console
→ EC2
→ Target Groups
→ Create target group
```

---

## Target Group Configuration

### Target Type

```text
Instances
```

### Target Group Name

```text
NLB-Target-Group
```

### Protocol

```text
TCP
```

### Port

```text
80
```

### IP Address Type

```text
IPv4
```

### VPC

```text
31.0.0.0/16
```

### Health Check

```text
Protocol: HTTP
Path: /
```

The target group uses TCP for the target traffic while the health check uses HTTP `/` to verify that the Apache application is responding.

### 📸 Screenshot

![NLB Target Group Configuration](../screenshots/214-nlb-target-group-configuration.png)

**Screenshot:** `214-nlb-target-group-configuration.png`

---

# 12. ➕ Register the EC2 Instances

The following EC2 instances were registered:

```text
NLB-Server-1
NLB-Server-2
```

Both targets use:

```text
Port: 80
```

### 📸 Screenshot

![NLB Target Group Targets](../screenshots/215-nlb-target-group-targets.png)

**Screenshot:** `215-nlb-target-group-targets.png`

---

# 13. 📋 Target Group Created

After creating the target group, both EC2 instances were registered.

At this stage, the targets initially appeared as:

```text
Unused
```

This is expected because the target group had not yet been associated with a load balancer.

AWS displayed:

```text
Target group is not configured to receive traffic from the load balancer.
```

This was an important practical observation during the lab.

### 📸 Screenshot

![NLB Target Group Created](../screenshots/216-nlb-target-group-created.png)

**Screenshot:** `216-nlb-target-group-created.png`

---

# 14. 🌐 Create the Network Load Balancer

Go to:

```text
AWS Console
→ EC2
→ Load Balancers
→ Create Load Balancer
→ Network Load Balancer
```

---

## NLB Configuration

### Name

```text
NLB-Lab
```

### Scheme

```text
Internet-facing
```

### IP Address Type

```text
IPv4
```

### VPC

```text
31.0.0.0/16
```

---

# 15. 🗺️ Network Mapping

For this lab, the Internet-facing NLB was placed in the public subnet:

```text
Availability Zone:
ap-south-1a

Subnet:
public-subnet

CIDR:
31.0.1.0/24
```

The private subnet was not selected because its route table did not have the required Internet Gateway route for this Internet-facing NLB configuration.

### 📸 Screenshot

![NLB Network Mapping](../screenshots/218-nlb-network-mapping.png)

**Screenshot:** `218-nlb-network-mapping.png`

---

# 16. 🔐 NLB Security Group

The NLB uses:

```text
my-nlb-sg
```

The security group allows the required inbound traffic for the NLB listener.

For this lab, TCP port `80` is allowed so that the NLB can receive HTTP client connections.

---

# 17. 🎧 Configure the NLB Listener

The listener was configured as:

```text
Protocol: TCP
Port: 80
```

The listener forwards traffic to:

```text
NLB-Target-Group
```

Therefore:

```text
Client
   │
   │ TCP :80
   ▼
NLB-Lab
   │
   │ Forward
   ▼
NLB-Target-Group
   │
   ├── NLB-Server-1
   │
   └── NLB-Server-2
```

### 📸 Screenshot

![NLB Listener and Routing](../screenshots/219-nlb-listener-and-routing.png)

**Screenshot:** `219-nlb-listener-and-routing.png`

---

# 18. ❤️ Target Health After NLB Association

Before the NLB was associated, the targets were:

```text
NLB-Server-1 → Unused
NLB-Server-2 → Unused
```

After creating and associating the NLB, health checks started.

Both targets eventually became:

```text
NLB-Server-1 → Healthy
NLB-Server-2 → Healthy
```

This confirms that:

- The target group is associated with the NLB.
- The health checks are reaching the backend servers.
- Apache is responding successfully on port 80.
- Both EC2 instances are available as healthy targets.

### 📸 Screenshot

![NLB Targets Healthy](../screenshots/217-nlb-targets-healthy.png)

**Screenshot:** `217-nlb-targets-healthy.png`

---

# 19. 🟢 Verify NLB Status

After creation, the NLB became active.

The NLB provides a DNS name that can be used by clients to access the load-balanced application.

### NLB

```text
NLB-Lab
```

### Scheme

```text
Internet-facing
```

### Listener

```text
TCP :80
```

### 📸 Screenshot

![NLB Active](../screenshots/221-nlb-active.png)

**Screenshot:** `221-nlb-active.png`

---

# 20. 🧪 Test the NLB DNS Name

Copy the DNS name provided by the NLB.

Open it in a browser:

```text
http://<NLB-DNS-NAME>
```

The request is processed through:

```text
Browser
   │
   ▼
NLB DNS
   │
   ▼
NLB-Lab
   │
   ▼
NLB-Target-Group
   │
   ├── NLB-Server-1
   │
   └── NLB-Server-2
```

The first response successfully reached one of the backend servers.

### 📸 Screenshot

![NLB First Request](../screenshots/222-nlb-first-request.png)

**Screenshot:** `222-nlb-first-request.png`

---

# 21. 🔄 Verify Traffic Can Reach Another Backend

The two servers were deliberately configured with different responses:

```text
Server 1:
Hello from NLB Server 1

Server 2:
Hello from NLB Server 2
```

By making additional requests to the NLB endpoint, we could identify traffic reaching the other backend server.

This demonstrates the relationship:

```text
NLB
 │
 ├── Target 1
 │
 └── Target 2
```

### 📸 Screenshot

![NLB Second Backend Response](../screenshots/223-nlb-second-backend-response.png)

**Screenshot:** `223-nlb-second-backend-response.png`

---

# 22. 🔍 How the Complete Request Flow Works

The complete request path is:

```text
                          Internet
                             │
                             │ TCP :80
                             ▼
                     ┌───────────────┐
                     │    NLB-Lab    │
                     │ Internet      │
                     │ facing        │
                     └───────┬───────┘
                             │
                             │ TCP :80
                             ▼
                   ┌─────────────────────┐
                   │  NLB-Target-Group   │
                   │       TCP :80       │
                   └──────────┬──────────┘
                              │
                       ┌──────┴──────┐
                       │             │
                       ▼             ▼
                NLB-Server-1      NLB-Server-2
                   :80                :80
                       │             │
                       ▼             ▼
                    Apache          Apache
```

The NLB receives the client connection and forwards it to a target in the configured target group.

The target group contains the two EC2 instances.

---

# 23. ❤️ Health Check Flow

The target group uses:

```text
Health Check Protocol: HTTP
Health Check Path: /
```

The health-check flow is:

```text
NLB
 │
 │ HTTP /
 ▼
NLB-Server-1:80
 │
 └── Apache responds → Healthy ✅


NLB
 │
 │ HTTP /
 ▼
NLB-Server-2:80
 │
 └── Apache responds → Healthy ✅
```

This is why both targets eventually changed from:

```text
Unused
```

to:

```text
Healthy
```

after the NLB was associated.

---

# 24. 🧠 Important Practical Observation — Unused vs Healthy

One of the important things observed during this lab was that a newly created target group does **not** immediately show the targets as healthy.

Before the NLB was associated:

```text
Target 1 → Unused
Target 2 → Unused
```

AWS indicated that the target group was not configured to receive traffic from a load balancer.

After creating the NLB and forwarding traffic to the target group:

```text
Target 1 → Healthy
Target 2 → Healthy
```

### Key lesson

```text
Registered target
       ≠
Healthy target
```

A target becomes useful to the load balancer after the target group is associated with the load balancer and the health checks succeed.

---

# 25. 🔑 Key Concepts Learned

## Network Load Balancer

An NLB provides network-level load balancing and supports protocols such as TCP, UDP and TLS.

---

## Target Group

A target group contains the backend resources that receive traffic from the load balancer.

In this lab:

```text
NLB-Target-Group
```

contains:

```text
NLB-Server-1
NLB-Server-2
```

---

## NLB Listener

The listener defines how the NLB accepts incoming connections.

Our listener:

```text
TCP :80
```

---

## Health Check

The target group uses:

```text
HTTP /
```

to verify that the backend Apache servers are responding.

---

## Internet-Facing NLB

The NLB is configured as:

```text
Internet-facing
```

so clients from the Internet can access the NLB endpoint.

---

## NLB DNS Name

The NLB provides a DNS endpoint that clients can use instead of directly accessing individual EC2 instances.

---

# 26. 🆚 ALB vs NLB — Quick Revision

```text
ALB
│
├── Layer 7
├── HTTP / HTTPS
├── Path-based routing
└── Application-level routing


NLB
│
├── Layer 4
├── TCP / UDP / TLS
├── Network-level traffic
└── Designed for high-throughput / low-latency network traffic
```

---

# 27. 🧪 Hands-On Validation Checklist

| Test | Result |
|---|---|
| VPC available | ✅ |
| Public subnet available | ✅ |
| NLB-Server-1 launched | ✅ |
| NLB-Server-2 launched | ✅ |
| Apache installed on Server 1 | ✅ |
| Apache installed on Server 2 | ✅ |
| Server 1 direct test | ✅ |
| Server 2 direct test | ✅ |
| Target group created | ✅ |
| TCP :80 configured | ✅ |
| HTTP `/` health check configured | ✅ |
| Both EC2 instances registered | ✅ |
| NLB created | ✅ |
| NLB listener TCP :80 | ✅ |
| Target group associated with NLB | ✅ |
| Server 1 healthy | ✅ |
| Server 2 healthy | ✅ |
| NLB DNS tested | ✅ |
| Backend response verified | ✅ |
| Other backend response verified | ✅ |

---

# 28. 💡 Important Practical Lessons

### 1. A target group can contain registered instances without immediately being healthy

The initial state in this lab was:

```text
Unused
```

because no load balancer was associated yet.

---

### 2. Target registration and target health are different things

Registering an EC2 instance only adds it to the target group.

The health check determines whether the target is healthy.

---

### 3. NLB listener protocol and health-check protocol can be different

Our configuration was:

```text
Traffic:

TCP :80


Health Check:

HTTP /
```

---

### 4. Test backend servers before introducing the load balancer

Testing the EC2 instances individually helped confirm that Apache was working before troubleshooting the NLB.

This makes troubleshooting much easier.

---

### 5. Give backend servers identifiable responses during learning

We used:

```text
Hello from NLB Server 1
```

and:

```text
Hello from NLB Server 2
```

This makes it easier to understand which backend received the request.

---

# 29. 🗣️ How to Explain NLB in an Interview

### Question:

**What is a Network Load Balancer?**

### Spoken Answer:

> A Network Load Balancer is an AWS load balancer that operates at the network or transport layer. It supports protocols such as TCP, UDP and TLS. In my hands-on lab, I created an Internet-facing NLB with a TCP port 80 listener and forwarded the traffic to a target group containing two EC2 instances running Apache. I also configured HTTP health checks to make sure both backend instances were healthy.

---

### Question:

**What is the difference between ALB and NLB?**

### Spoken Answer:

> An ALB works at Layer 7 and is designed for HTTP and HTTPS traffic with application-level features such as path-based routing. An NLB works at the network or transport layer and supports protocols such as TCP, UDP and TLS. In my lab, I used an NLB with a TCP port 80 listener to distribute traffic between two EC2 instances.

---

### Question:

**What is a target group?**

### Spoken Answer:

> A target group contains the backend resources that receive traffic from a load balancer. In my NLB lab, I created a target group called NLB-Target-Group and registered two EC2 instances running Apache on port 80.

---

### Question:

**Why were the targets initially shown as Unused?**

### Spoken Answer:

> The EC2 instances were registered in the target group, but the target group was not yet associated with a load balancer. After I created the NLB and configured its TCP port 80 listener to forward traffic to the target group, health checks started and both targets became healthy.

---

### Question:

**How did you verify that the NLB was working?**

### Spoken Answer:

> I first verified both EC2 instances individually by accessing their Apache pages. Then I created the target group and NLB. After both targets became healthy, I accessed the NLB DNS name from my browser and verified that the request reached the backend servers. I used different responses on the two servers so I could identify the backend receiving the traffic.

---

# 30. ⚡ Quick Revision Cheat Sheet

```text
NLB
│
├── Network-level load balancer
├── Layer 4
├── TCP / UDP / TLS
├── Target Group
├── Listener
├── Health Checks
└── NLB DNS
```

### Our Lab

```text
VPC
31.0.0.0/16

Public Subnet
31.0.1.0/24

EC2-1
NLB-Server-1
Apache :80

EC2-2
NLB-Server-2
Apache :80

Target Group
NLB-Target-Group
TCP :80
Health Check → HTTP /

NLB
NLB-Lab
Internet-facing
TCP :80
```

### Final Flow

```text
Client
  ↓
NLB DNS
  ↓
NLB-Lab
  ↓
NLB-Target-Group
  ↓
Healthy EC2 Target
  ↓
Apache :80
```

---

# 31. 🧹 Cleanup

AWS resources created during this lab can incur charges depending on the resource and account.

After completing the lab, clean up resources that are no longer required.

Suggested cleanup order:

```text
1. Delete Network Load Balancer
2. Delete Target Group
3. Terminate NLB-Server-1
4. Terminate NLB-Server-2
5. Delete unused Security Group if it is no longer required
```

### ⚠️ Important

Do **not** delete shared networking resources such as:

- Existing VPC
- Existing subnets
- Internet Gateway
- Shared route tables

unless they are no longer required by other labs.

---

# 32. 🎯 Final Result

We successfully implemented a Network Load Balancer using AWS.

The final architecture contains:

```text
                          Internet
                             │
                             ▼
                         NLB-Lab
                      Internet-facing
                         TCP :80
                             │
                             ▼
                   NLB-Target-Group
                         TCP :80
                             │
                    ┌────────┴────────┐
                    │                 │
                    ▼                 ▼
             NLB-Server-1       NLB-Server-2
                Apache              Apache
                 :80                 :80
                    │                 │
                    └────────┬────────┘
                             │
                       Public Subnet
                        31.0.1.0/24
```

Both EC2 instances became healthy targets, and the NLB DNS endpoint successfully reached the backend application.

The complete hands-on flow was:

```text
EC2 Instances
      ↓
Target Group
      ↓
Network Load Balancer
      ↓
TCP Listener
      ↓
Health Checks
      ↓
Healthy Targets
      ↓
NLB DNS
      ↓
Backend Application
```

---

# 33. 📸 Screenshot Index

| Screenshot | File | Purpose |
|---:|---|---|
| 205 | `205-nlb-vpc.png` | Verify VPC |
| 206 | `206-nlb-server-1-configuration.png` | Server 1 configuration |
| 207 | `207-nlb-server-1-user-data.png` | Server 1 Apache User Data |
| 208 | `208-nlb-server-1-running.png` | Server 1 running |
| 209 | `209-nlb-server-1-apache-test.png` | Server 1 direct Apache test |
| 210 | `210-nlb-server-2-configuration.png` | Server 2 configuration |
| 211 | `211-nlb-server-2-user-data.png` | Server 2 Apache User Data |
| 212 | `212-nlb-server-2-running.png` | Server 2 running |
| 213 | `213-nlb-server-2-apache-test.png` | Server 2 direct Apache test |
| 214 | `214-nlb-target-group-configuration.png` | Target group configuration |
| 215 | `215-nlb-target-group-targets.png` | Registered EC2 targets |
| 216 | `216-nlb-target-group-created.png` | Initial Unused target state |
| 217 | `217-nlb-targets-healthy.png` | Both targets healthy |
| 218 | `218-nlb-network-mapping.png` | NLB network mapping |
| 219 | `219-nlb-listener-and-routing.png` | TCP listener and target group |
| 220 | `220-nlb-provisioning.png` | NLB provisioning state |
| 221 | `221-nlb-active.png` | NLB active state |
| 222 | `222-nlb-first-request.png` | First request through NLB |
| 223 | `223-nlb-second-backend-response.png` | Backend response through NLB |

---

# 34. 📚 Previous / Next

[← Previous: VPC Flow Logs](./14-vpc-flow-logs.md)

[↑ Back to Project README](../README.md)

---

## 🏁 Conclusion

In this lab, we built a complete AWS Network Load Balancer setup from scratch.

We created two EC2 backend servers, installed Apache, created a TCP target group, registered both instances, configured an Internet-facing NLB, created a TCP port 80 listener, verified target health, and tested the NLB using its DNS endpoint.

The most important practical flow to remember is:

```text
EC2
 ↓
Target Group
 ↓
NLB
 ↓
Listener
 ↓
NLB DNS
 ↓
Healthy Backend
```

This hands-on demonstrates the fundamental architecture and workflow of an AWS Network Load Balancer.