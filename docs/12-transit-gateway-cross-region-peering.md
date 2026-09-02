# AWS Transit Gateway & Cross-Region Peering

## 1. What is AWS Transit Gateway?

AWS Transit Gateway (TGW) is a network transit hub that allows multiple VPCs and other network connections to communicate through a centralized networking architecture.

Instead of creating separate network connections between every VPC, Transit Gateway provides a **hub-and-spoke architecture** where VPCs can connect to a central Transit Gateway.

For example:

```text
             VPC 1
               |
               |
             TGW
            /   \
           /     \
        VPC 2   VPC 3
```

Each VPC can attach to the Transit Gateway, and routing can then be configured to control communication between the connected networks.

---

## 2. Why Do We Need Transit Gateway?

Without Transit Gateway, connecting multiple VPCs can become difficult to manage.

For example, if we have several VPCs that need to communicate with each other, we could create multiple VPC peering connections.

This can result in a complicated mesh architecture:

```text
VPC 1 -------- VPC 2
 | \            / |
 |  \          /  |
 |   \        /   |
 |    \      /    |
 |     \    /     |
VPC 3 -------- VPC 4
```

As the number of VPCs increases, the number of required connections and routing configurations also increases.

Transit Gateway solves this problem by providing a centralized hub:

```text
              VPC 1
                |
                |
              TGW
             / | \
            /  |  \
         VPC 2 VPC 3 VPC 4
```

### Problems Transit Gateway Helps Solve

- Reduces the complexity of VPC-to-VPC connectivity
- Provides a centralized networking hub
- Makes routing easier to manage
- Supports communication between multiple VPCs
- Can connect VPCs across AWS accounts
- Can connect VPCs across AWS regions using Transit Gateway peering
- Can also integrate with other network connections such as VPN and Direct Connect

---

# 3. What is Transit Gateway Peering?

Transit Gateway Peering allows two Transit Gateways in different AWS regions to communicate with each other.

For example:

```text
Mumbai Region
ap-south-1
31.0.0.0/16
       |
       |
   Mumbai TGW
       |
       |
       |  Transit Gateway Peering
       |
       |
 Virginia TGW
       |
       |
Virginia Region
us-east-1
41.0.0.0/16
```

This allows VPCs attached to the Transit Gateway in one region to communicate with VPCs attached to the Transit Gateway in another region.

In this practical:

```text
Mumbai TGW  <--------->  Virginia TGW
Requester                 Accepter
```

Mumbai acts as the **requester**, while Virginia acts as the **accepter**.

---

# 4. Problem We Are Solving

The objective of this practical is to establish private network connectivity between two VPCs located in different AWS regions.

### Mumbai

```text
Region: ap-south-1
VPC CIDR: 31.0.0.0/16
Role: Transit Gateway Peering Requester
```

### Virginia

```text
Region: us-east-1
VPC CIDR: 41.0.0.0/16
Role: Transit Gateway Peering Accepter
```

The final connectivity should look like:

```text
Mumbai EC2
Private IP
    |
    v
Mumbai VPC
31.0.0.0/16
    |
    v
Mumbai Transit Gateway
    |
    v
Transit Gateway Peering
    |
    v
Virginia Transit Gateway
    |
    v
Virginia VPC
41.0.0.0/16
    |
    v
Virginia EC2
Private IP
```

The reverse direction must also work.

---

# 5. Architecture

![AWS Transit Gateway Cross-Region Peering Architecture](../architecture/AWS-VPC-Transit-Gateway-Cross-Region-Peering-Architecture.png)

## Architecture Summary

The final architecture contains:

| Component | Mumbai | Virginia |
|---|---|---|
| AWS Region | `ap-south-1` | `us-east-1` |
| VPC CIDR | `31.0.0.0/16` | `41.0.0.0/16` |
| Transit Gateway | Mumbai TGW | Virginia TGW |
| TGW Role | Requester | Accepter |
| VPC Attachment | Mumbai VPC → Mumbai TGW | Virginia VPC → Virginia TGW |
| Public Subnet | Present | Present |
| Private Subnet | Present | Present |
| EC2 | Public subnet | Public subnet |
| Remote Route | `41.0.0.0/16` → TGW | `31.0.0.0/16` → TGW |

---

# 6. Important CIDR Ranges

The two VPCs must have **non-overlapping CIDR ranges**.

### Mumbai VPC

```text
31.0.0.0/16
```

### Virginia VPC

```text
41.0.0.0/16
```

Because these ranges do not overlap, AWS can route traffic between them.

---

# 7. Lab Environment

## Mumbai Region

```text
Region: ap-south-1
VPC: aws-course-mu1-vpc
CIDR: 31.0.0.0/16
```

The Mumbai VPC contains:

```text
31.0.0.0/16
│
├── Public Subnet
│
└── Private Subnet
```

The public subnet is associated with a route table containing an Internet Gateway route.

The private subnet does not use an Internet Gateway or NAT Gateway in this practical.

---

## Virginia Region

```text
Region: us-east-1
VPC CIDR: 41.0.0.0/16
```

The Virginia VPC contains:

```text
41.0.0.0/16
│
├── Public Subnet
│
└── Private Subnet
```

The public subnet is associated with a route table containing an Internet Gateway route.

The private subnet does not use an Internet Gateway or NAT Gateway in this practical.

---

# 8. Transit Gateway Cross-Region Peering Flow

The implementation follows this sequence:

```text
1. Create Transit Gateway in Mumbai
        |
        v
2. Create Transit Gateway in Virginia
        |
        v
3. Create Peering Attachment in Mumbai
        |
        v
4. Enter Virginia TGW ID
        |
        v
5. Request sent to Virginia
        |
        v
6. Accept Peering Request in Virginia
        |
        v
7. Peering becomes Available
        |
        v
8. Attach Mumbai VPC to Mumbai TGW
        |
        v
9. Attach Virginia VPC to Virginia TGW
        |
        v
10. Update VPC Route Tables
        |
        v
11. Update Transit Gateway Route Tables
        |
        v
12. Launch EC2 instances
        |
        v
13. Test private-IP connectivity
```

---

# 9. Step 1 — Create Transit Gateway in Mumbai

Switch to the Mumbai region:

```text
ap-south-1
```

Navigate to:

```text
VPC
→ Transit Gateways
→ Create transit gateway
```

Create the Mumbai Transit Gateway.

The Mumbai Transit Gateway will act as the **requester side** of the cross-region peering connection.

### Screenshot

```text
screenshots/141-mumbai-transit-gateway-configuration.png
```

This screenshot documents the Transit Gateway configuration in Mumbai.

### Result

The Mumbai Transit Gateway is created and becomes available.

### Screenshot

```text
screenshots/142-mumbai-transit-gateway-created.png
```

---

# 10. Step 2 — Create Transit Gateway in Virginia

Switch to the Virginia region:

```text
us-east-1
```

Create another Transit Gateway.

This Transit Gateway will act as the **accepter side**.

The Virginia Transit Gateway ID will be required while creating the peering attachment from Mumbai.

### Screenshot

```text
screenshots/139-virginia-transit-gateway-configuration.png
```

### Result

The Virginia Transit Gateway becomes available.

### Screenshot

```text
screenshots/140-virginia-transit-gateway-created.png
```

---

# 11. Step 3 — Create Transit Gateway Peering Attachment in Mumbai

Switch back to Mumbai.

Navigate to:

```text
VPC
→ Transit Gateway Attachments
→ Create Transit Gateway attachment
```

Select:

```text
Attachment type:
Peering Connection
```

The Mumbai Transit Gateway is the local Transit Gateway.

For the accepter Transit Gateway, enter the **Transit Gateway ID of the Virginia Transit Gateway**.

Select:

```text
Peer Region:
US East (N. Virginia)
```

This creates the peering request from Mumbai to Virginia.

### Screenshot

```text
screenshots/143-mumbai-transit-gateway-peering-configuration.png
```

---

# 12. Step 4 — Verify Peering Request

After creating the peering connection in Mumbai, the request is sent to the Virginia Transit Gateway.

The attachment initially enters a pending acceptance state.

### Screenshot

```text
screenshots/144-mumbai-transit-gateway-peering-pending-acceptance.png
```

This demonstrates that Mumbai has successfully sent the peering request to Virginia.

---

# 13. Step 5 — Accept the Peering Request in Virginia

Switch to:

```text
US East (N. Virginia)
```

Navigate to:

```text
VPC
→ Transit Gateway Attachments
```

The peering request from Mumbai should appear as pending acceptance.

Select the attachment and choose:

```text
Actions
→ Accept Transit Gateway attachment
```

### Screenshot

```text
screenshots/145-virginia-transit-gateway-peering-request.png
```

This shows the incoming peering request on the Virginia side.

After accepting the request:

### Screenshot

```text
screenshots/146-virginia-transit-gateway-peering-accepted.png
```

---

# 14. Step 6 — Verify Transit Gateway Peering

After AWS finishes processing the request, the Transit Gateway peering connection should become:

```text
Available
```

### Screenshot

```text
screenshots/147-transit-gateway-peering-available.png
```

At this point:

```text
Mumbai TGW
     |
     | Peering
     |
Virginia TGW
```

The two regional Transit Gateways are now connected.

However, VPC connectivity is not complete yet.

The VPCs must still be attached to their respective Transit Gateways.

---

# 15. Step 7 — Attach Mumbai VPC to Mumbai Transit Gateway

Switch to Mumbai.

Navigate to:

```text
VPC
→ Transit Gateway Attachments
→ Create Transit Gateway attachment
```

Select:

```text
Attachment type:
VPC
```

Select the Mumbai Transit Gateway.

Select the Mumbai VPC:

```text
aws-course-mu1-vpc
```

Then select the required subnets.

In this practical, both the public and private subnets are selected.

Create the attachment.

### Screenshot

```text
screenshots/148-mumbai-vpc-transit-gateway-attachment-configuration.png
```

After the attachment becomes available:

### Screenshot

```text
screenshots/149-mumbai-vpc-transit-gateway-attachment-available.png
```

The architecture is now:

```text
Mumbai VPC
    |
    | VPC Attachment
    |
Mumbai TGW
```

---

# 16. Step 8 — Attach Virginia VPC to Virginia Transit Gateway

Switch to Virginia.

Navigate to:

```text
VPC
→ Transit Gateway Attachments
→ Create Transit Gateway attachment
```

Select:

```text
Attachment type:
VPC
```

Select the Virginia Transit Gateway.

Select the Virginia VPC.

Select both public and private subnets.

Create the attachment.

### Screenshot

```text
screenshots/150-virginia-vpc-transit-gateway-attachment-configuration.png
```

After the attachment becomes available:

### Screenshot

```text
screenshots/151-virginia-vpc-transit-gateway-attachment-available.png
```

The architecture is now:

```text
Mumbai VPC
    |
Mumbai TGW
    |
    | Peering
    |
Virginia TGW
    |
Virginia VPC
```

---

# 17. Step 9 — Configure Mumbai VPC Route Table

The Mumbai VPC needs a route telling it how to reach the Virginia VPC.

Virginia VPC CIDR:

```text
41.0.0.0/16
```

Go to the Mumbai public route table.

Add:

```text
Destination:
41.0.0.0/16

Target:
Transit Gateway
```

Select the Mumbai Transit Gateway.

### Screenshot

```text
screenshots/152-mumbai-public-route-table-transit-gateway-route.png
```

---

# 18. Step 10 — Configure Mumbai Private Route Table

The same remote CIDR must also be routed through the Transit Gateway from the Mumbai private subnet.

Add:

```text
Destination:
41.0.0.0/16

Target:
Transit Gateway
```

### Screenshot

```text
screenshots/153-mumbai-private-route-table-transit-gateway-route.png
```

---

# 19. Step 11 — Configure Virginia VPC Route Table

Now configure the Virginia side.

The Mumbai VPC CIDR is:

```text
31.0.0.0/16
```

Add the route to the Virginia public route table:

```text
Destination:
31.0.0.0/16

Target:
Transit Gateway
```

### Screenshot

```text
screenshots/154-virginia-public-route-table-transit-gateway-route.png
```

---

# 20. Step 12 — Configure Virginia Private Route Table

Add the same route to the Virginia private route table:

```text
Destination:
31.0.0.0/16

Target:
Transit Gateway
```

### Screenshot

```text
screenshots/155-virginia-private-route-table-transit-gateway-route.png
```

---

# 21. Why Do We Need Routes on Both Sides?

Network communication requires a route in both directions.

For Mumbai to reach Virginia:

```text
31.0.0.0/16
      |
      | destination 41.0.0.0/16
      v
Mumbai TGW
      |
      v
TGW Peering
      |
      v
Virginia TGW
      |
      v
41.0.0.0/16
```

For Virginia to reach Mumbai:

```text
41.0.0.0/16
      |
      | destination 31.0.0.0/16
      v
Virginia TGW
      |
      v
TGW Peering
      |
      v
Mumbai TGW
      |
      v
31.0.0.0/16
```

Therefore, the routing must be configured in both regions.

---

# 22. Step 13 — Transit Gateway Route Tables

Transit Gateway route tables are also involved in forwarding traffic between the attached VPCs and the peering connection.

The Mumbai Transit Gateway route table contains the Mumbai VPC route as a propagated route.

Example:

```text
31.0.0.0/16 → Mumbai VPC Attachment
```

The remote Virginia network is configured through the Transit Gateway peering attachment.

Example:

```text
41.0.0.0/16 → Peering Attachment
```

Similarly, the Virginia Transit Gateway route table contains:

```text
41.0.0.0/16 → Virginia VPC Attachment
```

and:

```text
31.0.0.0/16 → Peering Attachment
```

This creates the complete forwarding path between the two regions.

---

# 23. Mumbai Transit Gateway Route Table

The Mumbai Transit Gateway route table shows the Mumbai VPC route.

Example:

```text
Destination: 31.0.0.0/16
Resource: Mumbai VPC
Route Type: Propagated
Route State: Active
```

The remote Virginia network can then be reached through the Transit Gateway peering attachment.

---

# 24. Virginia Transit Gateway Route Table

The Virginia Transit Gateway route table contains the Virginia VPC route.

Example:

```text
Destination: 41.0.0.0/16
Resource: Virginia VPC
Route Type: Propagated
Route State: Active
```

The Mumbai network is reachable through the Transit Gateway peering attachment.

---

# 25. Step 14 — Launch EC2 in Mumbai

Launch an EC2 instance in the Mumbai VPC.

The instance is placed inside the **public subnet**.

The purpose of using the public subnet is to allow SSH access from the local machine for testing.

The EC2 security group allows:

```text
SSH - TCP 22
ICMP
```

The EC2 instance receives a private IP address from the Mumbai VPC:

```text
31.0.0.0/16
```

### Screenshot

```text
screenshots/156-mumbai-ec2-network-configuration.png
```

### Running Instance

```text
screenshots/157-mumbai-ec2-running.png
```

---

# 26. Step 15 — Launch EC2 in Virginia

Launch another EC2 instance in the Virginia VPC.

Place it in the Virginia public subnet.

The security group allows:

```text
SSH - TCP 22
ICMP
```

The EC2 instance receives a private IP address from the Virginia VPC:

```text
41.0.0.0/16
```

### Screenshot

```text
screenshots/158-virginia-ec2-network-configuration.png
```

### Running Instance

```text
screenshots/159-virginia-ec2-running.png
```

---

# 27. Understanding Public and Private IP Connectivity

The EC2 instances are deployed in public subnets.

The public subnets have:

```text
0.0.0.0/0 → Internet Gateway
```

This allows Internet connectivity and SSH access using the public IP.

However, the cross-region communication that we are testing should use the **private IP addresses**.

The Transit Gateway provides private network connectivity between:

```text
31.0.0.0/16
```

and:

```text
41.0.0.0/16
```

The Internet Gateway is not responsible for this private cross-region path.

The traffic should travel through:

```text
Mumbai VPC
→ Mumbai TGW
→ TGW Peering
→ Virginia TGW
→ Virginia VPC
```

---

# 28. Step 16 — Test Mumbai to Virginia Connectivity

SSH into the Mumbai EC2 instance.

From the Mumbai EC2 instance, ping the **private IP address** of the Virginia EC2 instance.

Example:

```bash
ping <Virginia-EC2-private-IP>
```

The ping should succeed.

### Screenshot

```text
screenshots/160-mumbai-to-virginia-ping-success.png
```

This proves that traffic can travel:

```text
Mumbai EC2
    ↓
Mumbai VPC
    ↓
Mumbai TGW
    ↓
TGW Peering
    ↓
Virginia TGW
    ↓
Virginia VPC
    ↓
Virginia EC2
```

---

# 29. Step 17 — Test Virginia to Mumbai Connectivity

Now SSH into the Virginia EC2 instance.

From the Virginia EC2 instance, ping the **private IP address** of the Mumbai EC2 instance.

Example:

```bash
ping <Mumbai-EC2-private-IP>
```

The ping should also succeed.

### Screenshot

```text
screenshots/161-virginia-to-mumbai-ping-success.png
```

This proves that connectivity is working in the reverse direction.

---

# 30. Final Connectivity

The final architecture provides bidirectional private connectivity:

```text
             AWS Cloud
                 |
        ┌────────┴────────┐
        |                 |
     Mumbai            Virginia
   ap-south-1           us-east-1
        |                 |
31.0.0.0/16          41.0.0.0/16
        |                 |
   Mumbai VPC        Virginia VPC
        |                 |
    Mumbai TGW       Virginia TGW
        |                 |
        └───── Peering ───┘
                 |
          Private Network
          Connectivity
```

Traffic from Mumbai to Virginia:

```text
Mumbai EC2
    ↓
31.0.0.0/16
    ↓
Mumbai VPC Route Table
    ↓
Mumbai Transit Gateway
    ↓
Transit Gateway Peering
    ↓
Virginia Transit Gateway
    ↓
Virginia VPC
    ↓
Virginia EC2
```

Traffic from Virginia to Mumbai:

```text
Virginia EC2
    ↓
41.0.0.0/16
    ↓
Virginia VPC Route Table
    ↓
Virginia Transit Gateway
    ↓
Transit Gateway Peering
    ↓
Mumbai Transit Gateway
    ↓
Mumbai VPC
    ↓
Mumbai EC2
```

---

# 31. Important Difference: Internet Gateway vs Transit Gateway

An Internet Gateway provides connectivity between a VPC and the public Internet.

```text
EC2
 |
VPC
 |
Internet Gateway
 |
Internet
```

A Transit Gateway provides private connectivity between connected networks.

```text
EC2
 |
VPC
 |
Transit Gateway
 |
Transit Gateway Peering
 |
Transit Gateway
 |
VPC
 |
EC2
```

Therefore, for this cross-region private communication:

```text
Internet Gateway
        ❌
        |
        X
        |
Private Cross-Region Traffic

Transit Gateway
        ✅
        |
        v
Transit Gateway Peering
        |
        v
Remote VPC
```

---

# 32. Why Did We Use Public Subnets for the EC2 Instances?

The EC2 instances were launched in public subnets mainly so that we could access them through SSH from the local machine.

The public subnet has a route:

```text
0.0.0.0/0 → Internet Gateway
```

This provides Internet connectivity.

However, the private communication between the two EC2 instances uses their private IP addresses and the Transit Gateway path.

The Internet Gateway is therefore used for management access such as SSH, while Transit Gateway is responsible for the cross-region private network path.

---

# 33. Why Does the Private Subnet Not Need an Internet Gateway?

A private subnet does not have a direct route to an Internet Gateway.

For example:

```text
Private Subnet
     |
     | Local / TGW routes
     |
Transit Gateway
```

The private subnet can still communicate with another VPC through the Transit Gateway as long as the appropriate routes and security rules are configured.

A NAT Gateway would only be required if private instances needed outbound Internet access.

For this practical, Internet access from the private subnet was not required, so no NAT Gateway was used.

---

# 34. Routing Summary

## Mumbai VPC Route Table

```text
31.0.0.0/16 → local
41.0.0.0/16 → Transit Gateway
0.0.0.0/0   → Internet Gateway
```

## Virginia VPC Route Table

```text
41.0.0.0/16 → local
31.0.0.0/16 → Transit Gateway
0.0.0.0/0   → Internet Gateway
```

The important cross-region routes are:

```text
Mumbai:
41.0.0.0/16 → Transit Gateway

Virginia:
31.0.0.0/16 → Transit Gateway
```

---

# 35. Transit Gateway Peering vs VPC Peering

Both technologies can provide connectivity between networks, but they solve different networking requirements.

## VPC Peering

```text
VPC A
  |
  | Peering
  |
VPC B
```

It directly connects two VPCs.

For many VPCs, this can become difficult to manage.

## Transit Gateway Peering

```text
VPC A
  |
 TGW A
  |
  | Peering
  |
 TGW B
  |
VPC B
```

Transit Gateway provides a centralized architecture and is particularly useful when many VPCs and networks need to communicate.

---

# 36. What We Implemented

In this practical, the following infrastructure was created:

### Mumbai

```text
VPC
31.0.0.0/16

Public Subnet
Private Subnet

Transit Gateway

VPC → Transit Gateway Attachment
```

### Virginia

```text
VPC
41.0.0.0/16

Public Subnet
Private Subnet

Transit Gateway

VPC → Transit Gateway Attachment
```

### Cross-Region

```text
Mumbai TGW
     |
     | Transit Gateway Peering
     |
Virginia TGW
```

### Routing

```text
Mumbai:
41.0.0.0/16 → Transit Gateway

Virginia:
31.0.0.0/16 → Transit Gateway
```

### Testing

```text
Mumbai EC2
     |
     | Private IP
     v
Virginia EC2

Virginia EC2
     |
     | Private IP
     v
Mumbai EC2
```

Both tests were successful.

---

# 37. Screenshots

The implementation was documented using the following screenshots:

```text
127-virginia-vpc-configuration.png
128-virginia-vpc-created.png
129-virginia-public-subnet-configuration.png
130-virginia-private-subnet-configuration.png
131-virginia-subnets-created.png
132-virginia-internet-gateway-creation.png
133-virginia-internet-gateway-attached.png
134-virginia-public-route-table-creation.png
135-virginia-public-route-table-internet-route.png
136-virginia-public-subnet-route-table-association.png
137-virginia-private-route-table-creation.png
138-virginia-private-subnet-route-table-association.png
139-virginia-transit-gateway-configuration.png
140-virginia-transit-gateway-created.png
141-mumbai-transit-gateway-configuration.png
142-mumbai-transit-gateway-created.png
143-mumbai-transit-gateway-peering-configuration.png
144-mumbai-transit-gateway-peering-pending-acceptance.png
145-virginia-transit-gateway-peering-request.png
146-virginia-transit-gateway-peering-accepted.png
147-transit-gateway-peering-available.png
148-mumbai-vpc-transit-gateway-attachment-configuration.png
149-mumbai-vpc-transit-gateway-attachment-available.png
150-virginia-vpc-transit-gateway-attachment-configuration.png
151-virginia-vpc-transit-gateway-attachment-available.png
152-mumbai-public-route-table-transit-gateway-route.png
153-mumbai-private-route-table-transit-gateway-route.png
154-virginia-public-route-table-transit-gateway-route.png
155-virginia-private-route-table-transit-gateway-route.png
156-mumbai-ec2-network-configuration.png
157-mumbai-ec2-running.png
158-virginia-ec2-network-configuration.png
159-virginia-ec2-running.png
160-mumbai-to-virginia-ping-success.png
161-virginia-to-mumbai-ping-success.png
```

---

# 38. Key Takeaways

### 1. Transit Gateway is a centralized networking hub

It simplifies communication between multiple VPCs and networks.

### 2. Transit Gateway Peering connects regional Transit Gateways

In this practical:

```text
Mumbai TGW ↔ Virginia TGW
```

### 3. VPCs must be attached to their regional Transit Gateway

```text
Mumbai VPC → Mumbai TGW
Virginia VPC → Virginia TGW
```

### 4. CIDRs must not overlap

```text
Mumbai:
31.0.0.0/16

Virginia:
41.0.0.0/16
```

### 5. Routing is required

Mumbai needs:

```text
41.0.0.0/16 → Transit Gateway
```

Virginia needs:

```text
31.0.0.0/16 → Transit Gateway
```

### 6. Private connectivity does not require an Internet Gateway

The cross-region private traffic follows the Transit Gateway and peering path.

### 7. Connectivity was successfully validated

The EC2 instances successfully communicated using their **private IP addresses** in both directions.

---

# 39. Final Result

Successfully implemented **AWS Transit Gateway Cross-Region Peering** between:

```text
Mumbai
ap-south-1
31.0.0.0/16
```

and:

```text
US East (N. Virginia)
us-east-1
41.0.0.0/16
```

Final architecture:

```text
                 ┌───────────────────────┐
                 │      AWS Cloud        │
                 │                       │
                 │  Mumbai      Virginia │
                 │    |            |     │
                 │   VPC          VPC    │
                 │    |            |     │
                 │   TGW ←─Peering─→ TGW │
                 │    |            |     │
                 │   EC2          EC2     │
                 │                       │
                 └───────────────────────┘
```

Private connectivity was successfully verified:

```text
Mumbai EC2
    ↓
Virginia EC2
    ✓ Ping successful
```

and:

```text
Virginia EC2
    ↓
Mumbai EC2
    ✓ Ping successful
```

This demonstrates successful **private cross-region VPC connectivity using AWS Transit Gateway Peering**.