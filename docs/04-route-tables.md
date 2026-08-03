# AWS Route Tables – Public and Private Subnet Routing

## 📌 What is a Route Table?

A **route table** is a set of rules, called routes, that determines where network traffic from a subnet is directed.

Each route contains two important components:

```text
Destination → Target
```

For example:

```text
31.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

The **destination** specifies the IP address range the traffic is trying to reach, while the **target** specifies where matching traffic should be sent.

In this project, separate route tables are used for the public and private subnets so that each subnet can have different routing behavior.

---

## 🎯 Why Do We Need Route Tables?

Creating a VPC, subnets, and an Internet Gateway does not by itself determine how traffic should move.

Route tables provide those routing decisions.

At this stage of the project, the architecture contains:

```text
VPC: 31.0.0.0/16
│
├── Public Subnet
│   └── 31.0.1.0/24
│
└── Private Subnet
    └── 31.0.2.0/24
```

We want different routing behavior for these two subnets.

The public subnet requires a route toward the Internet Gateway.

The private subnet should not have a **direct** route to the Internet Gateway.

Therefore, separate route tables are used.

---

# 🗺️ Initial Routing Design

## Public Route Table

The public route table contains:

| Destination | Target | Purpose |
|---|---|---|
| `31.0.0.0/16` | `local` | Communication within the VPC |
| `0.0.0.0/0` | Internet Gateway | Internet-bound IPv4 traffic |

It is associated with:

```text
Public Subnet
31.0.1.0/24
```

---

## Private Route Table

At this stage of the project, the private route table initially contains:

| Destination | Target | Purpose |
|---|---|---|
| `31.0.0.0/16` | `local` | Communication within the VPC |

It is associated with:

```text
Private Subnet
31.0.2.0/24
```

There is no:

```text
0.0.0.0/0 → Internet Gateway
```

route in the private route table.

Therefore, the private subnet does not have a direct route to the Internet Gateway.

> **Architecture Update:** Later in the project, a NAT Gateway is introduced to provide outbound internet connectivity to resources in the private subnet. The private route table is then updated with `0.0.0.0/0 → NAT Gateway`. The private subnet still does not receive a direct route to the Internet Gateway. See [NAT Gateway Configuration](05-nat-gateway.md) for the updated configuration.

---

# 🏠 Understanding the Local Route

When a VPC is created, AWS automatically adds a **local route** to its route tables for communication within the VPC.

For this project:

```text
Destination: 31.0.0.0/16
Target:      local
```

This allows resources using IP addresses within the VPC CIDR to route traffic to each other, subject to other networking and security controls.

For example:

```text
Public Subnet
31.0.1.0/24
       │
       │
       ▼
31.0.0.0/16 → local
       │
       ▼
Private Subnet
31.0.2.0/24
```

Because both subnet CIDR blocks belong to the same VPC CIDR, the local route provides the routing path between them.

Security Groups, Network ACLs, and resource-level configuration can still determine whether particular traffic is permitted.

---

# 🌎 Understanding the Default Route

The public route table also contains:

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

`0.0.0.0/0` represents all IPv4 destinations.

This route acts as the default path for IPv4 traffic that needs to leave the VPC toward the Internet.

Therefore:

```text
Public Subnet
      │
      ▼
Public Route Table
      │
      ▼
0.0.0.0/0
      │
      ▼
Internet Gateway
      │
      ▼
Internet
```

Later in the project, the private route table also receives a default route, but its target is different:

```text
Public Route Table
0.0.0.0/0 → Internet Gateway

Private Route Table
0.0.0.0/0 → NAT Gateway
```

This distinction allows the public and private subnets to have different Internet connectivity models.

---

# 🧠 How Does AWS Choose a Route?

A route table can contain multiple routes.

AWS uses the **most specific matching route** for the destination IP address.

This is commonly known as **longest prefix match**.

Consider the public route table:

```text
Destination       Target

31.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

Suppose traffic is being sent to:

```text
31.0.2.10
```

This destination matches both:

```text
31.0.0.0/16
```

and:

```text
0.0.0.0/0
```

However, `/16` is more specific than `/0`.

Therefore, AWS selects:

```text
31.0.0.0/16 → local
```

and the traffic remains within the VPC.

If the destination is instead:

```text
8.8.8.8
```

it does not match the VPC's `31.0.0.0/16` route.

The default route matches:

```text
0.0.0.0/0 → Internet Gateway
```

so traffic from the public subnet is directed toward the Internet Gateway.

After NAT Gateway is introduced later in the project, traffic from the private subnet to the same external destination instead matches:

```text
0.0.0.0/0 → NAT Gateway
```

because the private subnet uses a different route table.

---

# 🛠️ Route Table Implementation

## Step 1 – Create the Public Route Table

From the AWS Management Console:

```text
AWS Management Console
        ↓
VPC
        ↓
Route Tables
        ↓
Create route table
```

A custom route table was created for the public subnet.

It was associated with:

```text
VPC: aws-course-mu1-vpc
```

### Public Route Table Creation

![Public Route Table Creation](../screenshots/08-public-route-table-creation.png)

---

## Step 2 – Add the Internet Route

The public route table needs a route for Internet-bound traffic.

The following route was added:

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

The route table therefore contains:

```text
31.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

### Public Route Table Routes

![Public Route Table Internet Route](../screenshots/09-public-route-table-internet-route.png)

This creates the routing path from the public subnet toward the Internet Gateway.

---

## Step 3 – Associate the Public Subnet

Creating a route table does not automatically make a subnet use it.

The public route table was explicitly associated with:

```text
Public Subnet
31.0.1.0/24
```

### Public Subnet Association

![Public Subnet Route Table Association](../screenshots/10-public-subnet-route-table-association.png)

The public subnet now uses the routing rules defined in the public route table.

---

# 🔒 Private Route Table

## Step 4 – Create the Private Route Table

A separate route table was created for the private subnet.

### Private Route Table Creation

![Private Route Table Creation](../screenshots/11-private-route-table-creation.png)

At this stage, the private route table retains only the VPC local route:

```text
31.0.0.0/16 → local
```

and does not contain:

```text
0.0.0.0/0 → Internet Gateway
```

Therefore, the private subnet has no direct Internet Gateway route.

Later, when the NAT Gateway is introduced, this same private route table is extended with:

```text
0.0.0.0/0 → NAT Gateway
```

That later configuration is documented separately in:

**[05 – NAT Gateway Configuration →](05-nat-gateway.md)**

---

# 🔄 Comparing the Initial Public and Private Routing

At this stage of the project, the routing design can be summarized as:

```text
                         Internet
                             │
                             │
                      Internet Gateway
                             │
                             │
                ┌────────────┴────────────┐
                │                         │
          Public Route               No direct IGW
             Table                       route
                │                         │
                ▼                         ▼
        Public Subnet               Private Subnet
        31.0.1.0/24                 31.0.2.0/24
```

More specifically:

```text
PUBLIC

31.0.1.0/24
     │
     ▼
Public Route Table
     │
     ├── 31.0.0.0/16 → local
     │
     └── 0.0.0.0/0 → IGW
                       │
                       ▼
                    Internet
```

versus the initial private routing:

```text
PRIVATE

31.0.2.0/24
     │
     ▼
Private Route Table
     │
     └── 31.0.0.0/16 → local

No direct IGW route
```

---

# 🔄 Routing After the NAT Gateway Extension

Later in the project, the private subnet's routing is extended without giving the private subnet a direct Internet Gateway route.

The updated design becomes:

```text
PUBLIC SUBNET

31.0.1.0/24
     │
     ▼
Public Route Table
     │
     ├── 31.0.0.0/16 → local
     │
     └── 0.0.0.0/0 → Internet Gateway
```

and:

```text
PRIVATE SUBNET

31.0.2.0/24
     │
     ▼
Private Route Table
     │
     ├── 31.0.0.0/16 → local
     │
     └── 0.0.0.0/0 → NAT Gateway
```

This allows resources in the private subnet to initiate outbound Internet connections while remaining without a direct route to the Internet Gateway.

The NAT Gateway implementation and validation are covered in the next documentation section.

---

# 📸 Initial VPC Resource Map

After configuring the original route tables and subnet associations, the AWS VPC Resource Map showed the relationship between the networking components at that stage.

![Initial AWS VPC Resource Map](../screenshots/12-final-vpc-resource-map.png)

The architecture at this point contained:

```text
VPC
31.0.0.0/16
│
├── Public Subnet
│   └── Public Route Table
│       ├── Local Route
│       └── Internet Gateway Route
│
└── Private Subnet
    └── Private Route Table
        └── Local Route
```

> This screenshot represents the architecture **before the NAT Gateway and EC2 instances were added**. A newer VPC Resource Map is included later in the project after the architecture is extended.

---

# ⚠️ Important: Route to IGW vs Actual Internet Access

A public subnet route such as:

```text
0.0.0.0/0 → Internet Gateway
```

provides a routing path toward the Internet Gateway.

However, an EC2 instance placed inside that subnet would still require appropriate configuration for direct Internet communication, including:

- A public IPv4 address or Elastic IP
- Appropriate Security Group rules
- Appropriate Network ACL rules
- A functioning Internet Gateway attached to the VPC

Therefore:

```text
IGW route ≠ automatic Internet access for every resource
```

The route is one important part of the complete networking configuration.

Similarly, adding:

```text
0.0.0.0/0 → NAT Gateway
```

to the private route table provides an outbound routing path for the private subnet, but the complete traffic path still depends on the NAT Gateway and the surrounding VPC networking configuration.

---

# 🧠 Key Concepts Learned

Through the route table configuration, this project demonstrates:

- How AWS route tables control network traffic
- How the VPC local route works
- How `0.0.0.0/0` acts as a default IPv4 route
- How an Internet Gateway is used as a route target
- How route tables are associated with subnets
- How public and private subnet routing differs
- How longest prefix matching determines which route AWS selects
- Why an Internet Gateway attachment alone does not make a subnet public
- How different subnets can use different default route targets
- How the private route table can later be extended with a NAT Gateway without giving the private subnet a direct IGW route

---

# ✅ Result

The initial routing architecture implemented in this stage was:

```text
AWS VPC: 31.0.0.0/16
│
├── Public Subnet: 31.0.1.0/24
│      │
│      └── Public Route Table
│             ├── 31.0.0.0/16 → local
│             └── 0.0.0.0/0 → Internet Gateway
│
└── Private Subnet: 31.0.2.0/24
       │
       └── Private Route Table
              └── 31.0.0.0/16 → local
```

The architecture is subsequently extended to:

```text
AWS VPC: 31.0.0.0/16
│
├── Public Subnet: 31.0.1.0/24
│      │
│      └── Public Route Table
│             ├── 31.0.0.0/16 → local
│             └── 0.0.0.0/0 → Internet Gateway
│
└── Private Subnet: 31.0.2.0/24
       │
       └── Private Route Table
              ├── 31.0.0.0/16 → local
              └── 0.0.0.0/0 → NAT Gateway
```

This preserves the private subnet's separation from a direct Internet Gateway route while allowing outbound Internet connectivity through the NAT Gateway.

---

## 🏁 Route Table Configuration Complete

The following routing concepts have now been configured and documented:

- ✅ Public Route Table
- ✅ Private Route Table
- ✅ VPC Local Route
- ✅ Public Default Route
- ✅ Internet Gateway Route
- ✅ Subnet Associations
- ✅ Longest Prefix Matching
- ✅ Public vs Private Routing
- ✅ NAT Gateway Route Extension

The next stage introduces the NAT Gateway used by the private subnet.

---

[← Previous: Internet Gateway](03-internet-gateway.md) | [Next: NAT Gateway →](05-nat-gateway.md)

[Back to Project README](../README.md)