# AWS Transit Gateway and Transit Gateway Attachments

## Overview

AWS Transit Gateway provides a centralized way to connect multiple VPCs through a single networking hub.

In the previous section, VPC Peering was used to establish connectivity between two VPCs. While VPC Peering works well for a small number of VPCs, managing many individual peering connections can become difficult.

Transit Gateway provides a hub-and-spoke architecture where multiple VPCs can connect to a single Transit Gateway through Transit Gateway attachments.

In this project, three VPCs were connected using a single Transit Gateway:

**AWS Course VPC** → `31.0.0.0/16`

**Demo Course VPC** → `71.0.0.0/16`

**Redshift VPC** → `10.0.0.0/16`

Each VPC contains public and private subnets with their own route tables.

The Transit Gateway attachments provide the connection between the VPCs and the Transit Gateway, while routes in the VPC route tables determine where traffic destined for another VPC should be forwarded.

---

## Architecture

The final architecture contains three VPCs connected through a single Transit Gateway.

![AWS Transit Gateway Architecture](../architecture/AWS-VPC-Transit-Gateway-Architecture.png)

The architecture follows a hub-and-spoke model:

- AWS Course VPC: `31.0.0.0/16`
- Demo Course VPC: `71.0.0.0/16`
- Redshift VPC: `10.0.0.0/16`
- One Transit Gateway
- Three Transit Gateway attachments

### AWS Resource Map

The following screenshot shows the three VPC environments and their networking resources before establishing Transit Gateway connectivity.

![Three VPC Resource Map](../screenshots/94-three-vpc-resource-map.png)

---

# 1. Creating the Additional VPCs

The existing VPC with CIDR `31.0.0.0/16` from the previous sections was reused.

Two additional VPCs were created for the Transit Gateway demonstration.

The VPC CIDR ranges were:

| VPC | CIDR |
|---|---|
| AWS Course VPC | `31.0.0.0/16` |
| Demo Course VPC | `71.0.0.0/16` |
| Redshift VPC | `10.0.0.0/16` |

The CIDR ranges are non-overlapping, allowing routing between the VPCs.

---

## VPC `71.0.0.0/16`

The first additional VPC was created with the CIDR:

`71.0.0.0/16`

### VPC Configuration

![VPC 71 Configuration](../screenshots/68-vpc-71-configuration.png)

The VPC was created using the VPC-only option with the CIDR range `71.0.0.0/16`.

### VPC Created

![VPC 71 Created](../screenshots/69-vpc-71-created.png)

The VPC successfully entered the available state.

---

## VPC `10.0.0.0/16`

The second additional VPC was created with:

`10.0.0.0/16`

### VPC Configuration

![VPC 10 Configuration](../screenshots/70-vpc-10-configuration.png)

### VPC Created

![VPC 10 Created](../screenshots/71-vpc-10-created.png)

The VPC successfully entered the available state.

---

# 2. Creating Subnets for the `71.0.0.0/16` VPC

The `71.0.0.0/16` VPC was configured with separate public and private subnets.

The subnet ranges were:

- Public Subnet: `71.0.1.0/24`
- Private Subnet: `71.0.2.0/24`

## Public Subnet

![Demo Public Subnet Configuration](../screenshots/72-demo-public-subnet-configuration.png)

The public subnet was created using:

- VPC CIDR: `71.0.0.0/16`
- Subnet CIDR: `71.0.1.0/24`

### Public Subnet Created

![Demo Public Subnet Created](../screenshots/73-demo-public-subnet-created.png)

The public subnet was successfully created.

## Private Subnet

![Demo Private Subnet Configuration](../screenshots/74-demo-private-subnet-configuration.png)

The private subnet was created using:

- VPC CIDR: `71.0.0.0/16`
- Subnet CIDR: `71.0.2.0/24`

### Private Subnet Created

![Demo Private Subnet Created](../screenshots/75-demo-private-subnet-created.png)

The private subnet was successfully created without automatic public IPv4 assignment.

---

# 3. Internet Gateway for the `71.0.0.0/16` VPC

An Internet Gateway was created for the new VPC.

### Internet Gateway Creation

![Demo Internet Gateway Creation](../screenshots/76-demo-internet-gateway-creation.png)

### Internet Gateway Attached

![Demo Internet Gateway Attached](../screenshots/77-demo-internet-gateway-attached.png)

The Internet Gateway was attached to the `71.0.0.0/16` VPC.

---

# 4. Route Tables for the `71.0.0.0/16` VPC

Separate public and private route tables were created.

## Public Route Table

![Demo Public Route Table Creation](../screenshots/78-demo-public-route-table-creation.png)

The public route table was configured with the Internet Gateway route:

`0.0.0.0/0 → Internet Gateway`

![Demo Public Route Table Internet Route](../screenshots/79-demo-public-route-table-internet-route.png)

The public subnet was then associated with the public route table.

![Demo Public Subnet Route Table Association](../screenshots/80-demo-public-subnet-route-table-association.png)

## Private Route Table

A separate private route table was created for the private subnet.

![Demo Private Route Table Creation](../screenshots/81-demo-private-route-table-creation.png)

The private subnet was associated with the private route table.

![Demo Private Subnet Route Table Association](../screenshots/82-demo-private-subnet-route-table-association.png)

The private route table did not receive an Internet Gateway route.

---

# 5. Creating Subnets for the `10.0.0.0/16` VPC

The second new VPC was configured with:

- Public Subnet: `10.0.1.0/24`
- Private Subnet: `10.0.2.0/24`

## Public Subnet

![Redshift Public Subnet Configuration](../screenshots/83-redshift-public-subnet-configuration.png)

The public subnet was created inside the `10.0.0.0/16` VPC.

![Redshift Public Subnet Created](../screenshots/84-redshift-public-subnet-created.png)

## Private Subnet

![Redshift Private Subnet Configuration](../screenshots/85-redshift-private-subnet-configuration.png)

The private subnet was configured with:

`10.0.2.0/24`

![Redshift Private Subnet Created](../screenshots/86-redshift-private-subnet-created.png)

---

# 6. Internet Gateway for the `10.0.0.0/16` VPC

An Internet Gateway was created and attached to the VPC.

### Internet Gateway Creation

![Redshift Internet Gateway Creation](../screenshots/87-redshift-internet-gateway-creation.png)

### Internet Gateway Attached

![Redshift Internet Gateway Attached](../screenshots/88-redshift-internet-gateway-attached.png)

---

# 7. Route Tables for the `10.0.0.0/16` VPC

A public and private route table were created.

## Public Route Table

![Redshift Public Route Table Creation](../screenshots/89-redshift-public-route-table-creation.png)

The public route table was configured with:

`0.0.0.0/0 → Internet Gateway`

![Redshift Public Route Table Internet Route](../screenshots/90-redshift-public-route-table-internet-route.png)

The public subnet was associated with the public route table.

![Redshift Public Subnet Route Table Association](../screenshots/91-redshift-public-subnet-route-table-association.png)

## Private Route Table

The private route table was created separately.

![Redshift Private Route Table Creation](../screenshots/92-redshift-private-route-table-creation.png)

The private subnet was associated with the private route table.

![Redshift Private Subnet Route Table Association](../screenshots/93-redshift-private-subnet-route-table-association.png)

---

# 8. Three-VPC Infrastructure

At this stage, the project contained three VPCs:

- `31.0.0.0/16` — AWS Course VPC
- `71.0.0.0/16` — Demo Course VPC
- `10.0.0.0/16` — Redshift VPC

Each VPC had its own networking resources.

![Three VPC Resource Map](../screenshots/94-three-vpc-resource-map.png)

The three VPCs were now ready to be connected through a Transit Gateway.

---

# 9. Creating the Transit Gateway

AWS Transit Gateway was created to provide a centralized networking hub for the three VPCs.

The Transit Gateway acts as the central point through which the VPCs communicate.

### Transit Gateway Configuration

![Transit Gateway Configuration](../screenshots/95-transit-gateway-configuration.png)

The Transit Gateway was configured within the AWS account.

No cross-account sharing was required for this lab.

### Transit Gateway Created

![Transit Gateway Created](../screenshots/96-transit-gateway-created.png)

The Transit Gateway successfully became available.

---

# 10. Transit Gateway Attachments

Creating the Transit Gateway alone does not connect the VPCs.

Each VPC must be attached to the Transit Gateway.

Three Transit Gateway attachments were created:

- `31.0.0.0/16` → Transit Gateway
- `71.0.0.0/16` → Transit Gateway
- `10.0.0.0/16` → Transit Gateway

---

## AWS Course VPC Attachment

The existing `31.0.0.0/16` VPC was attached to the Transit Gateway.

![AWS Course Transit Gateway Attachment Configuration](../screenshots/97-aws-course-transit-gateway-attachment-configuration.png)

After provisioning, the attachment became available.

![AWS Course Transit Gateway Attachment Available](../screenshots/98-aws-course-transit-gateway-attachment-available.png)

---

## Demo Course VPC Attachment

The `71.0.0.0/16` VPC was attached to the same Transit Gateway.

![Demo Course Transit Gateway Attachment Configuration](../screenshots/99-demo-course-transit-gateway-attachment-configuration.png)

The attachment successfully became available.

![Demo Course Transit Gateway Attachment Available](../screenshots/100-demo-course-transit-gateway-attachment-available.png)

---

## Redshift VPC Attachment

The `10.0.0.0/16` VPC was also attached to the Transit Gateway.

![Redshift Transit Gateway Attachment Configuration](../screenshots/101-redshift-transit-gateway-attachment-configuration.png)

The attachment successfully became available.

![Redshift Transit Gateway Attachment Available](../screenshots/102-redshift-transit-gateway-attachment-available.png)

---

# 11. Final Transit Gateway Attachments

All three VPCs were now attached to the same Transit Gateway.

![All Transit Gateway Attachments](../screenshots/103-transit-gateway-all-attachments.png)

All three attachments were available.

The final attachment structure was:

- AWS Course VPC → Transit Gateway
- Demo Course VPC → Transit Gateway
- Redshift VPC → Transit Gateway

---

# 12. Updating VPC Route Tables

Although the VPCs were attached to the Transit Gateway, connectivity between the VPCs was not yet established.

The VPC route tables needed routes that directed traffic for the other VPC CIDR ranges toward the Transit Gateway.

The routing model was:

### AWS Course VPC

- `71.0.0.0/16 → Transit Gateway`
- `10.0.0.0/16 → Transit Gateway`

### Demo Course VPC

- `31.0.0.0/16 → Transit Gateway`
- `10.0.0.0/16 → Transit Gateway`

### Redshift VPC

- `31.0.0.0/16 → Transit Gateway`
- `71.0.0.0/16 → Transit Gateway`

Both public and private route tables were updated for the lab.

---

## AWS Course VPC Public Route Table

The public route table for the `31.0.0.0/16` VPC was updated with routes for the other VPCs.

![AWS Course Public Transit Gateway Routes](../screenshots/104-aws-course-public-route-table-transit-gateway-routes.png)

Routes included:

- `71.0.0.0/16 → Transit Gateway`
- `10.0.0.0/16 → Transit Gateway`

## AWS Course VPC Private Route Table

![AWS Course Private Transit Gateway Routes](../screenshots/105-aws-course-private-route-table-transit-gateway-routes.png)

The same destination VPC CIDRs were configured through the Transit Gateway.

---

## Demo Course VPC Public Route Table

The `71.0.0.0/16` VPC public route table was updated with:

- `31.0.0.0/16 → Transit Gateway`
- `10.0.0.0/16 → Transit Gateway`

![Demo Public Transit Gateway Routes](../screenshots/106-demo-public-route-table-transit-gateway-routes.png)

## Demo Course VPC Private Route Table

![Demo Private Transit Gateway Routes](../screenshots/107-demo-private-route-table-transit-gateway-routes.png)

The private route table was also updated with the routes toward the other two VPCs.

---

## Redshift VPC Public Route Table

The `10.0.0.0/16` VPC public route table was updated with:

- `31.0.0.0/16 → Transit Gateway`
- `71.0.0.0/16 → Transit Gateway`

![Redshift Public Transit Gateway Routes](../screenshots/108-redshift-public-route-table-transit-gateway-routes.png)

## Redshift VPC Private Route Table

![Redshift Private Transit Gateway Routes](../screenshots/109-redshift-private-route-table-transit-gateway-routes.png)

The private route table was also updated with routes toward the other VPCs.

---

# 13. Final Transit Gateway Routing Configuration

After updating all route tables, the routing configuration was verified.

![Final Transit Gateway Routing Configuration](../screenshots/110-final-transit-gateway-routing-configuration.png)

The route tables now provided paths between all three VPC CIDR ranges through the Transit Gateway.

---

# 14. EC2 Instances for Connectivity Testing

To validate the Transit Gateway connectivity, one EC2 instance was launched inside each VPC.

The instances were placed in public subnets so they could be accessed using SSH.

However, the actual VPC-to-VPC connectivity test was performed using the private IPv4 addresses of the EC2 instances.

This ensures that the test demonstrates private communication between the VPCs through the Transit Gateway rather than communication through public IP addresses.

---

## EC2 in AWS Course VPC

![AWS Course EC2 Network Configuration](../screenshots/111-aws-course-ec2-network-configuration.png)

The EC2 instance was launched inside the `31.0.0.0/16` VPC.

### EC2 Running

![AWS Course EC2 Running](../screenshots/112-aws-course-ec2-running.png)

---

## EC2 in Demo Course VPC

![Demo Course EC2 Network Configuration](../screenshots/113-demo-course-ec2-network-configuration.png)

The EC2 instance was launched inside the `71.0.0.0/16` VPC.

### EC2 Running

![Demo Course EC2 Running](../screenshots/114-demo-course-ec2-running.png)

---

## EC2 in Redshift VPC

![Redshift EC2 Network Configuration](../screenshots/115-redshift-ec2-network-configuration.png)

The EC2 instance was launched inside the `10.0.0.0/16` VPC.

### EC2 Running

![Redshift EC2 Running](../screenshots/116-redshift-ec2-running.png)

---

# 15. EC2 Private IP Addresses

The three EC2 instances received private IPv4 addresses from their respective VPC CIDR ranges.

![Three VPC EC2 Private IP Details](../screenshots/117-three-vpc-ec2-private-ip-details.png)

These private IP addresses were used for the connectivity tests.

The instances belonged to different VPC CIDR ranges:

- AWS Course EC2 → `31.0.0.0/16`
- Demo Course EC2 → `71.0.0.0/16`
- Redshift EC2 → `10.0.0.0/16`

---

# 16. Testing Transit Gateway Connectivity

With the Transit Gateway attachments and routes configured, connectivity between the EC2 instances was tested using their private IPv4 addresses.

The connectivity was tested between all three VPCs.

---

## AWS Course VPC → Demo Course VPC

The EC2 instance in the `31.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `71.0.0.0/16` VPC.

![AWS Course to Demo Ping Success](../screenshots/118-aws-course-to-demo-ping-success.png)

---

## AWS Course VPC → Redshift VPC

The EC2 instance in the `31.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `10.0.0.0/16` VPC.

![AWS Course to Redshift Ping Success](../screenshots/119-aws-course-to-redshift-ping-success.png)

---

## Demo Course VPC → AWS Course VPC

The EC2 instance in the `71.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `31.0.0.0/16` VPC.

![Demo to AWS Course Ping Success](../screenshots/120-demo-to-aws-course-ping-success.png)

---

## Demo Course VPC → Redshift VPC

The EC2 instance in the `71.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `10.0.0.0/16` VPC.

![Demo to Redshift Ping Success](../screenshots/121-demo-to-redshift-ping-success.png)

---

## Redshift VPC → AWS Course VPC

The EC2 instance in the `10.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `31.0.0.0/16` VPC.

![Redshift to AWS Course Ping Success](../screenshots/122-redshift-to-aws-course-ping-success.png)

---

## Redshift VPC → Demo Course VPC

The EC2 instance in the `10.0.0.0/16` VPC successfully pinged the private IP of the EC2 instance in the `71.0.0.0/16` VPC.

![Redshift to Demo Ping Success](../screenshots/123-redshift-to-demo-ping-success.png)

---

# 17. Negative Connectivity Test

To verify that the Transit Gateway routes were responsible for the connectivity, a negative test was performed.

A Transit Gateway route was temporarily removed from the route table.

![Transit Gateway Route Removed](../screenshots/124-transit-gateway-route-removed-negative-test.png)

After removing the required route, the corresponding private-IP connectivity test failed.

![Transit Gateway Connectivity Blocked](../screenshots/125-transit-gateway-connectivity-blocked.png)

This demonstrates that creating a Transit Gateway attachment alone is not enough.

The VPC route tables must contain the correct destination CIDR routes pointing to the Transit Gateway.

---

# 18. Restoring Connectivity

After the required Transit Gateway route was restored, the private-IP connectivity test succeeded again.

![Final Transit Gateway Connectivity](../screenshots/126-final-transit-gateway-connectivity.png)

This confirmed that the complete networking path was functioning correctly.

---

# 19. Complete Transit Gateway Traffic Flow

The final traffic flow can be summarized as:

**Source EC2**

↓

**VPC Route Table**

↓

**Destination VPC CIDR**

↓

**Transit Gateway**

↓

**Transit Gateway Attachment**

↓

**Destination VPC**

↓

**Destination EC2**

For example:

`31.0.0.0/16 → 71.0.0.0/16`

The route table sends traffic destined for `71.0.0.0/16` to the Transit Gateway, which forwards the traffic through the appropriate VPC attachment.

The same process applies between the other VPCs.

---

# VPC Peering vs Transit Gateway

The previous section demonstrated VPC Peering between two VPCs.

With VPC Peering, two VPCs require a dedicated peering connection:

**VPC A ↔ VPC B**

If another VPC is introduced, additional peering connections are required.

With Transit Gateway, multiple VPCs can connect to a single centralized Transit Gateway:

**VPC A → Transit Gateway**

**VPC B → Transit Gateway**

**VPC C → Transit Gateway**

This provides a centralized hub-and-spoke networking architecture.

Transit Gateway becomes particularly useful when the number of connected VPCs grows.

---

# Key Concepts Learned

Through this hands-on project, I gained practical experience with:

- AWS Transit Gateway
- Transit Gateway creation
- Transit Gateway attachments
- Connecting multiple VPCs through a centralized networking hub
- Creating public and private subnets
- Internet Gateway configuration
- Public and private route tables
- Adding Transit Gateway routes
- Routing traffic between multiple VPC CIDR ranges
- Launching EC2 instances in different VPCs
- Testing private IPv4 connectivity between VPCs
- Understanding Transit Gateway attachment behavior
- Validating connectivity using ICMP
- Performing a negative connectivity test
- Understanding the difference between VPC Peering and Transit Gateway

---

# Final Result

The Transit Gateway implementation successfully connected three VPCs:

- AWS Course VPC — `31.0.0.0/16`
- Demo Course VPC — `71.0.0.0/16`
- Redshift VPC — `10.0.0.0/16`

The final implementation consisted of:

1. Three VPCs
2. Public and private subnets
3. Internet Gateways
4. Public and private route tables
5. One Transit Gateway
6. Three Transit Gateway attachments
7. Transit Gateway routes in the VPC route tables
8. Three EC2 instances
9. Private IPv4 connectivity testing
10. Negative connectivity testing

The successful ping tests between all three VPCs confirmed that the Transit Gateway attachments and route-table configuration were working correctly.

The negative test further demonstrated that removing the required Transit Gateway route blocks communication between the VPCs.

---

## Next

The project will continue with additional AWS VPC networking concepts and progressively extend the architecture.

---

[← Previous: VPC Peering](10-vpc-peering.md) | [Back to Project README](../README.md)