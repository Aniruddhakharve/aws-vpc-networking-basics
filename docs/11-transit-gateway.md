# Section 11: Transit Gateway

## Overview

AWS Transit Gateway is a network transit hub that can be used to connect multiple VPCs and other networks through a centralized networking service.

In the previous section, I used VPC Peering to connect two VPCs. VPC Peering works well when only a small number of VPCs need to communicate, but managing many individual peering connections can become difficult as the environment grows.

In this section, I created and configured an AWS Transit Gateway in the Mumbai (`ap-south-1`) region and used it to connect multiple VPCs.

The practical implementation contains three VPCs:

- AWS Course VPC — `31.0.0.0/16`
- Demo Course VPC — `71.0.0.0/16`
- Redshift VPC — `10.0.0.0/16`

The Transit Gateway provides centralized connectivity between these VPCs.

The final connectivity was tested using EC2 instances in all three VPCs. Communication was successfully verified in both directions between all three VPCs using their private IP addresses.

---

# 1. What is AWS Transit Gateway?

AWS Transit Gateway is a managed network transit hub that allows multiple VPCs and other networks to connect through a centralized gateway.

Instead of creating individual connections between every VPC, each VPC can be attached to the Transit Gateway.

The architecture becomes:

```text
VPC-A
   |
   |
   v
Transit Gateway
   ^
   |
   |
VPC-B

VPC-C
   |
   |
   +----> Transit Gateway
```

The Transit Gateway acts as a central networking hub.

For this practical implementation:

```text
AWS Course VPC
31.0.0.0/16
        |
        |
        v
   Transit Gateway
        ^
        |
        |
Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
        |
        |
        +----> Transit Gateway
```

Once the VPCs are attached and the appropriate routes are configured, resources in the connected VPCs can communicate using private IP addresses.

---

# 2. Why Do We Need Transit Gateway?

Suppose an organization has only two VPCs:

```text
VPC-A <--------> VPC-B
```

A VPC Peering connection can be sufficient.

However, imagine an environment containing:

```text
VPC-A
VPC-B
VPC-C
VPC-D
VPC-E
VPC-F
```

With VPC Peering, many individual connections would have to be created and maintained.

The networking architecture can quickly become difficult to manage.

Transit Gateway provides a centralized alternative:

```text
                 VPC-A
                   |
                   |
VPC-B -------- Transit Gateway -------- VPC-C
                   |
                   |
                 VPC-D
                   |
                   |
                 VPC-E
```

Each VPC only needs an attachment to the Transit Gateway.

This makes the overall network architecture easier to manage and scale.

---

# 3. What Problem Does Transit Gateway Solve?

The main problem Transit Gateway solves is **complex connectivity between multiple networks**.

Without a centralized networking hub:

```text
VPC-A -------- VPC-B
  | \            / |
  |  \          /  |
  |   \        /   |
  |    \      /    |
  |     \    /     |
  |      \  /      |
  VPC-C -------- VPC-D
```

As the number of VPCs increases, the number of connections and routing relationships can become difficult to manage.

Transit Gateway provides a centralized architecture:

```text
                 VPC-A
                   |
                   |
                 VPC-B
                   |
                   v
            +-------------+
            |   Transit   |
            |   Gateway   |
            +-------------+
              ^    ^    ^
              |    |    |
            VPC-C VPC-D VPC-E
```

In this lab, I used three VPCs to demonstrate this centralized connectivity model.

---

# 4. VPCs Used in This Lab

The practical implementation used the following VPCs:

| VPC | CIDR | Purpose |
|---|---|---|
| AWS Course VPC | `31.0.0.0/16` | Primary VPC |
| Demo Course VPC | `71.0.0.0/16` | Additional VPC |
| Redshift VPC | `10.0.0.0/16` | Additional VPC |

The Transit Gateway was used as the central networking hub.

```text
                 AWS Course VPC
                  31.0.0.0/16
                       |
                       |
                       v
                +-------------+
                |   Transit   |
                |   Gateway   |
                +-------------+
                   ^        ^
                   |        |
                   |        |
          Demo Course VPC   Redshift VPC
             71.0.0.0/16    10.0.0.0/16
```

---

# 5. Architecture

![AWS VPC Transit Gateway Architecture](../architecture/AWS-VPC-Transit-Gateway-Architecture.png)

The architecture contains:

- Three VPCs.
- Public and private subnets.
- Internet Gateways.
- Route tables.
- Transit Gateway.
- Transit Gateway VPC attachments.
- EC2 instances used for connectivity testing.

The Transit Gateway provides the centralized path between the three VPCs.

---

# 6. Create Demo Course VPC

The Demo Course VPC was created with the CIDR:

```text
71.0.0.0/16
```

![Demo Course VPC Configuration](../screenshots/68-vpc-71-configuration.png)

The VPC was successfully created.

![Demo Course VPC Created](../screenshots/69-vpc-71-created.png)

---

# 7. Create Redshift VPC

The Redshift VPC was created with the CIDR:

```text
10.0.0.0/16
```

![Redshift VPC Configuration](../screenshots/70-vpc-10-configuration.png)

The VPC was successfully created.

![Redshift VPC Created](../screenshots/71-vpc-10-created.png)

---

# 8. Create Demo Course Public Subnet

A public subnet was created inside the Demo Course VPC.

![Demo Course Public Subnet Configuration](../screenshots/72-demo-public-subnet-configuration.png)

The public subnet was successfully created.

![Demo Course Public Subnet Created](../screenshots/73-demo-public-subnet-created.png)

---

# 9. Create Demo Course Private Subnet

A private subnet was created inside the Demo Course VPC.

![Demo Course Private Subnet Configuration](../screenshots/74-demo-private-subnet-configuration.png)

The private subnet was successfully created.

![Demo Course Private Subnet Created](../screenshots/75-demo-private-subnet-created.png)

---

# 10. Create Demo Course Internet Gateway

An Internet Gateway was created for the Demo Course VPC.

![Demo Course Internet Gateway Creation](../screenshots/76-demo-internet-gateway-creation.png)

The Internet Gateway was attached to the Demo Course VPC.

![Demo Course Internet Gateway Attached](../screenshots/77-demo-internet-gateway-attached.png)

---

# 11. Configure Demo Course Public Route Table

A public route table was created for the Demo Course public subnet.

![Demo Course Public Route Table Creation](../screenshots/78-demo-public-route-table-creation.png)

An Internet Gateway route was added to provide Internet connectivity.

![Demo Course Public Route Table Internet Route](../screenshots/79-demo-public-route-table-internet-route.png)

The public subnet was associated with the public route table.

![Demo Course Public Subnet Route Table Association](../screenshots/80-demo-public-subnet-route-table-association.png)

---

# 12. Configure Demo Course Private Route Table

A private route table was created for the Demo Course private subnet.

![Demo Course Private Route Table Creation](../screenshots/81-demo-private-route-table-creation.png)

The private subnet was associated with the private route table.

![Demo Course Private Subnet Route Table Association](../screenshots/82-demo-private-subnet-route-table-association.png)

---

# 13. Create Redshift Public Subnet

A public subnet was created inside the Redshift VPC.

![Redshift Public Subnet Configuration](../screenshots/83-redshift-public-subnet-configuration.png)

The public subnet was successfully created.

![Redshift Public Subnet Created](../screenshots/84-redshift-public-subnet-created.png)

---

# 14. Create Redshift Private Subnet

A private subnet was created inside the Redshift VPC.

![Redshift Private Subnet Configuration](../screenshots/85-redshift-private-subnet-configuration.png)

The private subnet was successfully created.

![Redshift Private Subnet Created](../screenshots/86-redshift-private-subnet-created.png)

---

# 15. Create Redshift Internet Gateway

An Internet Gateway was created for the Redshift VPC.

![Redshift Internet Gateway Creation](../screenshots/87-redshift-internet-gateway-creation.png)

The Internet Gateway was attached to the Redshift VPC.

![Redshift Internet Gateway Attached](../screenshots/88-redshift-internet-gateway-attached.png)

---

# 16. Configure Redshift Public Route Table

A public route table was created for the Redshift public subnet.

![Redshift Public Route Table Creation](../screenshots/89-redshift-public-route-table-creation.png)

An Internet Gateway route was added to the public route table.

![Redshift Public Route Table Internet Route](../screenshots/90-redshift-public-route-table-internet-route.png)

The public subnet was associated with the public route table.

![Redshift Public Subnet Route Table Association](../screenshots/91-redshift-public-subnet-route-table-association.png)

---

# 17. Configure Redshift Private Route Table

A private route table was created for the Redshift private subnet.

![Redshift Private Route Table Creation](../screenshots/92-redshift-private-route-table-creation.png)

The private subnet was associated with the private route table.

![Redshift Private Subnet Route Table Association](../screenshots/93-redshift-private-subnet-route-table-association.png)

---

# 18. Verify the Three-VPC Environment

At this stage, the environment contained three separate VPCs:

```text
AWS Course VPC
31.0.0.0/16

Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
```

The three-VPC environment was verified using the AWS resource map.

![Three VPC Resource Map](../screenshots/94-three-vpc-resource-map.png)

At this point, the VPCs existed independently.

The next step was to create the Transit Gateway and use it to connect these VPCs.

---

# 19. Create Transit Gateway

A Transit Gateway was created to act as the central networking hub.

![Transit Gateway Configuration](../screenshots/95-transit-gateway-configuration.png)

The Transit Gateway was successfully created.

![Transit Gateway Created](../screenshots/96-transit-gateway-created.png)

The Transit Gateway now acts as the central point through which the VPCs can communicate.

```text
                  AWS Course VPC
                  31.0.0.0/16
                       |
                       |
                       v
                +-------------+
                |   Transit   |
                |   Gateway   |
                +-------------+
                       ^
                       |
                 +-----+-----+
                 |           |
                 |           |
          Demo Course VPC   Redshift VPC
           71.0.0.0/16     10.0.0.0/16
```

---

# 20. Attach AWS Course VPC to Transit Gateway

The AWS Course VPC was attached to the Transit Gateway.

The attachment was configured using:

```text
Attachment Type: VPC
VPC: AWS Course VPC
```

The required subnets were selected for the attachment.

![AWS Course Transit Gateway Attachment Configuration](../screenshots/97-aws-course-transit-gateway-attachment-configuration.png)

The attachment became available.

![AWS Course Transit Gateway Attachment Available](../screenshots/98-aws-course-transit-gateway-attachment-available.png)

---

# 21. Attach Demo Course VPC to Transit Gateway

The Demo Course VPC was attached to the same Transit Gateway.

![Demo Course Transit Gateway Attachment Configuration](../screenshots/99-demo-course-transit-gateway-attachment-configuration.png)

The attachment became available.

![Demo Course Transit Gateway Attachment Available](../screenshots/100-demo-course-transit-gateway-attachment-available.png)

---

# 22. Attach Redshift VPC to Transit Gateway

The Redshift VPC was also attached to the Transit Gateway.

![Redshift Transit Gateway Attachment Configuration](../screenshots/101-redshift-transit-gateway-attachment-configuration.png)

The attachment became available.

![Redshift Transit Gateway Attachment Available](../screenshots/102-redshift-transit-gateway-attachment-available.png)

At this point, all three VPCs were attached to the same Transit Gateway.

![Transit Gateway All Attachments](../screenshots/103-transit-gateway-all-attachments.png)

The centralized architecture was now:

```text
                         Transit Gateway
                              |
             +----------------+----------------+
             |                |                |
             |                |                |
             v                v                v
       AWS Course VPC   Demo Course VPC   Redshift VPC
        31.0.0.0/16     71.0.0.0/16       10.0.0.0/16
```

---

# 23. Configure Transit Gateway Routes

Creating VPC attachments alone does not automatically provide the required routing between all VPCs.

The Transit Gateway route table must contain routes for the destination VPC CIDRs.

The required destinations in this lab are:

```text
31.0.0.0/16
71.0.0.0/16
10.0.0.0/16
```

The Transit Gateway route table was configured to associate the VPC CIDRs with their corresponding VPC attachments.

---

# 24. Configure AWS Course Public Route Table

The AWS Course public route table was updated to send traffic destined for the Demo Course VPC through the Transit Gateway.

Destination:

```text
71.0.0.0/16
```

Target:

```text
Transit Gateway
```

The route was also configured for the Redshift VPC.

Destination:

```text
10.0.0.0/16
```

Target:

```text
Transit Gateway
```

![AWS Course Public Route Table Transit Gateway Routes](../screenshots/104-aws-course-public-route-table-transit-gateway-routes.png)

---

# 25. Configure AWS Course Private Route Table

The AWS Course private route table was updated with routes toward the other VPCs through the Transit Gateway.

Routes:

```text
71.0.0.0/16 -> Transit Gateway
10.0.0.0/16 -> Transit Gateway
```

![AWS Course Private Route Table Transit Gateway Routes](../screenshots/105-aws-course-private-route-table-transit-gateway-routes.png)

---

# 26. Configure Demo Course Public Route Table

The Demo Course public route table was updated to send traffic toward the AWS Course and Redshift VPCs through the Transit Gateway.

Routes:

```text
31.0.0.0/16 -> Transit Gateway
10.0.0.0/16 -> Transit Gateway
```

![Demo Course Public Route Table Transit Gateway Routes](../screenshots/106-demo-public-route-table-transit-gateway-routes.png)

---

# 27. Configure Demo Course Private Route Table

The Demo Course private route table was updated with the required Transit Gateway routes.

Routes:

```text
31.0.0.0/16 -> Transit Gateway
10.0.0.0/16 -> Transit Gateway
```

![Demo Course Private Route Table Transit Gateway Routes](../screenshots/107-demo-private-route-table-transit-gateway-routes.png)

---

# 28. Configure Redshift Public Route Table

The Redshift public route table was updated to send traffic toward the AWS Course and Demo Course VPCs through the Transit Gateway.

Routes:

```text
31.0.0.0/16 -> Transit Gateway
71.0.0.0/16 -> Transit Gateway
```

![Redshift Public Route Table Transit Gateway Routes](../screenshots/108-redshift-public-route-table-transit-gateway-routes.png)

---

# 29. Configure Redshift Private Route Table

The Redshift private route table was updated with the required Transit Gateway routes.

Routes:

```text
31.0.0.0/16 -> Transit Gateway
71.0.0.0/16 -> Transit Gateway
```

![Redshift Private Route Table Transit Gateway Routes](../screenshots/109-redshift-private-route-table-transit-gateway-routes.png)

---

# 30. Final Transit Gateway Routing Configuration

After configuring the VPC route tables and Transit Gateway routes, the complete routing configuration was verified.

![Final Transit Gateway Routing Configuration](../screenshots/110-final-transit-gateway-routing-configuration.png)

The final routing architecture was:

```text
                         Transit Gateway
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
     31.0.0.0/16         71.0.0.0/16         10.0.0.0/16
     AWS Course           Demo Course           Redshift
        VPC                   VPC                  VPC
```

---

# 31. Launch EC2 in AWS Course VPC

An EC2 instance was launched in the AWS Course VPC.

The instance was placed in the public subnet so that it could be accessed using SSH.

![AWS Course EC2 Network Configuration](../screenshots/111-aws-course-ec2-network-configuration.png)

The instance was successfully launched and reached the running state.

![AWS Course EC2 Running](../screenshots/112-aws-course-ec2-running.png)

---

# 32. Launch EC2 in Demo Course VPC

An EC2 instance was launched in the Demo Course VPC.

![Demo Course EC2 Network Configuration](../screenshots/113-demo-course-ec2-network-configuration.png)

The instance was successfully launched and reached the running state.

![Demo Course EC2 Running](../screenshots/114-demo-course-ec2-running.png)

---

# 33. Launch EC2 in Redshift VPC

An EC2 instance was launched in the Redshift VPC.

![Redshift EC2 Network Configuration](../screenshots/115-redshift-ec2-network-configuration.png)

The instance was successfully launched and reached the running state.

![Redshift EC2 Running](../screenshots/116-redshift-ec2-running.png)

---

# 34. Verify EC2 Private IP Addresses

The private IP details of the three EC2 instances were verified before testing connectivity.

![Three VPC EC2 Private IP Details](../screenshots/117-three-vpc-ec2-private-ip-details.png)

The three instances were located in separate VPCs:

```text
AWS Course VPC
31.0.0.0/16

Demo Course VPC
71.0.0.0/16

Redshift VPC
10.0.0.0/16
```

Each EC2 instance had a private IP address within its respective VPC CIDR.

---

# 35. Test AWS Course to Demo Course Connectivity

From the AWS Course EC2 instance, connectivity to the Demo Course EC2 instance was tested using its private IP address.

The ping was successful.

![AWS Course to Demo Ping Success](../screenshots/118-aws-course-to-demo-ping-success.png)

This confirmed that traffic could travel:

```text
AWS Course VPC
      |
      v
Transit Gateway
      |
      v
Demo Course VPC
```

---

# 36. Test AWS Course to Redshift Connectivity

Connectivity from the AWS Course EC2 instance to the Redshift EC2 instance was tested.

The ping was successful.

![AWS Course to Redshift Ping Success](../screenshots/119-aws-course-to-redshift-ping-success.png)

The traffic path was:

```text
AWS Course VPC
      |
      v
Transit Gateway
      |
      v
Redshift VPC
```

---

# 37. Test Demo Course to AWS Course Connectivity

The reverse direction was tested from the Demo Course EC2 instance to the AWS Course EC2 instance.

The ping was successful.

![Demo to AWS Course Ping Success](../screenshots/120-demo-to-aws-course-ping-success.png)

This confirmed bidirectional communication between the two VPCs.

---

# 38. Test Demo Course to Redshift Connectivity

The Demo Course EC2 instance was then used to test connectivity with the Redshift EC2 instance.

The ping was successful.

![Demo to Redshift Ping Success](../screenshots/121-demo-to-redshift-ping-success.png)

The traffic path was:

```text
Demo Course VPC
      |
      v
Transit Gateway
      |
      v
Redshift VPC
```

---

# 39. Test Redshift to AWS Course Connectivity

Connectivity from the Redshift EC2 instance to the AWS Course EC2 instance was tested.

The ping was successful.

![Redshift to AWS Course Ping Success](../screenshots/122-redshift-to-aws-course-ping-success.png)

This confirmed that communication was also working from the Redshift VPC toward the AWS Course VPC.

---

# 40. Test Redshift to Demo Course Connectivity

Finally, connectivity from the Redshift EC2 instance to the Demo Course EC2 instance was tested.

The ping was successful.

![Redshift to Demo Ping Success](../screenshots/123-redshift-to-demo-ping-success.png)

This completed the bidirectional connectivity testing between all three VPCs.

---

# 41. Negative Test – Remove a Transit Gateway Route

To verify that the routing configuration was actually responsible for the connectivity, one of the Transit Gateway routes was temporarily removed.

![Transit Gateway Route Removed Negative Test](../screenshots/124-transit-gateway-route-removed-negative-test.png)

After removing the required route, connectivity was tested again.

The communication was blocked.

![Transit Gateway Connectivity Blocked](../screenshots/125-transit-gateway-connectivity-blocked.png)

This negative test demonstrated that the Transit Gateway route table is an important part of the communication path.

The VPC attachment alone is not enough.

The required route must exist for traffic to reach the destination VPC.

---

# 42. Restore Final Transit Gateway Connectivity

The required Transit Gateway route was restored.

Connectivity between the VPCs was tested again.

![Final Transit Gateway Connectivity](../screenshots/126-final-transit-gateway-connectivity.png)

The connectivity was successfully restored.

The final architecture therefore provides:

```text
AWS Course VPC
31.0.0.0/16
        |
        |
        v
   Transit Gateway
      /       \
     /         \
    v           v
Demo Course   Redshift
71.0.0.0/16   10.0.0.0/16
```

---

# 43. Key Concepts Learned

## Centralized Connectivity

Transit Gateway provides a centralized networking hub for connecting multiple VPCs.

Instead of maintaining separate peering relationships between every VPC, VPCs can attach to the same Transit Gateway.

---

## Transit Gateway Attachments

A VPC must be attached to the Transit Gateway before it can use the Transit Gateway for connectivity.

In this lab, three VPC attachments were created:

```text
AWS Course VPC
       |
       v
Transit Gateway

Demo Course VPC
       |
       v
Transit Gateway

Redshift VPC
       |
       v
Transit Gateway
```

---

## Transit Gateway Route Table

The Transit Gateway has its own route table.

This route table determines where traffic should be forwarded.

The destination CIDRs used in this lab were:

```text
31.0.0.0/16
71.0.0.0/16
10.0.0.0/16
```

---

## VPC Route Tables

The VPC route tables also need routes pointing to the Transit Gateway.

For example, the AWS Course VPC requires routes toward:

```text
71.0.0.0/16 -> Transit Gateway
10.0.0.0/16 -> Transit Gateway
```

Similarly, the other VPCs require routes toward the destination VPCs.

Therefore, successful communication requires routing to be correctly configured on both sides of the Transit Gateway path.

---

## Private IP Connectivity

The connectivity tests were performed between EC2 instances using their private IP addresses.

The traffic remained within the AWS networking environment through the Transit Gateway rather than requiring communication through public IP addresses.

---

# 44. Transit Gateway vs VPC Peering

VPC Peering and Transit Gateway solve related but different networking problems.

### VPC Peering

VPC Peering provides a direct connection between two VPCs:

```text
VPC-A <------> VPC-B
```

For multiple VPCs, additional peering connections may be required.

### Transit Gateway

Transit Gateway provides a centralized networking hub:

```text
             VPC-A
               |
               v
          Transit Gateway
            /        \
           v          v
        VPC-B       VPC-C
```

This makes Transit Gateway particularly useful when an environment contains many VPCs and requires centralized connectivity.

---

# 45. Important Routing Lesson

One of the most important lessons from this lab is that simply attaching a VPC to a Transit Gateway does not automatically guarantee connectivity.

The complete path requires:

```text
Source EC2
    |
    v
VPC Route Table
    |
    v
Transit Gateway
    |
    v
Transit Gateway Route Table
    |
    v
Destination VPC Attachment
    |
    v
Destination VPC Route Table
    |
    v
Destination EC2
```

If one required route is missing, communication can fail.

The negative testing performed in this lab demonstrated this behavior.

---

# 46. Final Architecture

The final Transit Gateway architecture consists of three VPCs connected through one centralized Transit Gateway.

```text
                         +----------------------+
                         |   Transit Gateway    |
                         +----------------------+
                            /        |        \
                           /         |         \
                          /          |          \
                         v           v           v
                AWS Course VPC  Demo Course VPC  Redshift VPC
                 31.0.0.0/16    71.0.0.0/16     10.0.0.0/16
                      |               |                |
                      v               v                v
                   EC2             EC2              EC2
```

The Transit Gateway provides the centralized connectivity path between the three VPCs.

---

# 47. Final Result

The Transit Gateway configuration was successfully completed.

### VPCs

```text
AWS Course VPC
31.0.0.0/16
```

```text
Demo Course VPC
71.0.0.0/16
```

```text
Redshift VPC
10.0.0.0/16
```

### Transit Gateway

```text
AWS Course VPC
       |
       v
Transit Gateway
       ^
       |
       +------ Demo Course VPC
       |
       +------ Redshift VPC
```

### Connectivity Tests

```text
AWS Course -> Demo Course
SUCCESS
```

```text
AWS Course -> Redshift
SUCCESS
```

```text
Demo Course -> AWS Course
SUCCESS
```

```text
Demo Course -> Redshift
SUCCESS
```

```text
Redshift -> AWS Course
SUCCESS
```

```text
Redshift -> Demo Course
SUCCESS
```

The negative test also confirmed that removing a required Transit Gateway route blocked communication.

After restoring the route, connectivity was successfully restored.

---

# 48. What This Lab Demonstrated

This hands-on implementation demonstrated how to use **AWS Transit Gateway to provide centralized private connectivity between multiple VPCs**.

The complete implementation covered:

1. Creating multiple VPCs.
2. Creating public and private subnets.
3. Configuring Internet Gateways.
4. Configuring route tables.
5. Creating a Transit Gateway.
6. Creating VPC Transit Gateway attachments.
7. Configuring Transit Gateway routes.
8. Updating VPC route tables.
9. Launching EC2 instances.
10. Testing private IP connectivity between VPCs.
11. Performing a negative routing test.
12. Restoring the route and verifying connectivity.

The key networking model demonstrated in this lab is:

```text
Multiple VPCs
      |
      v
Transit Gateway
      |
      v
Centralized Connectivity
      |
      v
Private VPC-to-VPC Communication
```

---

## ➡️ Next Section

[← Previous: VPC Peering](10-vpc-peering.md) | [Next: Transit Gateway Cross-Region Peering →](12-transit-gateway-cross-region-peering.md)

[Back to Project README](../README.md)
