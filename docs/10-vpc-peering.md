# Section 8: VPC Peering

## Overview

AWS VPC Peering allows two Virtual Private Clouds (VPCs) to communicate with each other using private IP addresses.

In this section, I configured VPC Peering between two VPCs located in different AWS Regions:

- Mumbai (`ap-south-1`)
- N. Virginia (`us-east-1`)

The Mumbai VPC uses the CIDR:

    31.0.0.0/16

The N. Virginia VPC uses the CIDR:

    41.0.0.0/16

After establishing the VPC Peering connection, I launched an EC2 instance in each VPC and tested communication between them using their private IPv4 addresses.

Initially, the ping failed because the required routes were not present in the route tables.

After adding the appropriate VPC Peering routes to both VPCs, communication between the two EC2 instances was successful.

---

## Architecture

![AWS VPC Peering Architecture](../architecture/AWS-VPC-Peering-Architecture.png)

### Environment

| Component | Mumbai | N. Virginia |
|---|---|---|
| Region | `ap-south-1` | `us-east-1` |
| VPC CIDR | `31.0.0.0/16` | `41.0.0.0/16` |
| Public Subnet | `31.0.1.0/24` | `41.0.1.0/24` |
| Private Subnet | `31.0.2.0/24` | `41.0.2.0/24` |
| EC2 Instance | `mumbai-instance` | `virginia-instance` |
| EC2 Private IP | `31.0.1.139` | `41.0.1.223` |

---

# 1. N. Virginia VPC Configuration

The Mumbai VPC and its networking resources were already available from the previous sections of this project.

For the VPC Peering lab, I created a new VPC in the N. Virginia (`us-east-1`) region.

The VPC was configured with the CIDR:

    41.0.0.0/16

![N. Virginia VPC Configuration](../screenshots/50-virginia-vpc-configuration.png)

---

# 2. N. Virginia Public Subnet

A public subnet was created inside the N. Virginia VPC.

Configuration:

    VPC CIDR:       41.0.0.0/16
    Subnet CIDR:    41.0.1.0/24
    Availability Zone: us-east-1a

![N. Virginia Public Subnet Configuration](../screenshots/51-virginia-public-subnet-configuration.png)

---

# 3. N. Virginia Private Subnet

A private subnet was also created in the N. Virginia VPC.

Configuration:

    VPC CIDR:       41.0.0.0/16
    Subnet CIDR:    41.0.2.0/24
    Availability Zone: us-east-1b

![N. Virginia Private Subnet Configuration](../screenshots/52-virginia-private-subnet-configuration.png)

---

# 4. Internet Gateway

An Internet Gateway was created and attached to the N. Virginia VPC.

This allows resources in the public subnet to communicate with the Internet when the appropriate route is configured.

![N. Virginia Internet Gateway](../screenshots/53-virginia-internet-gateway.png)

---

# 5. N. Virginia Route Table

The N. Virginia public route table was configured for the public subnet.

The route table contains the local VPC route and the Internet Gateway route.

The local route allows communication inside the VPC, while the default route allows Internet-bound traffic through the Internet Gateway.

![N. Virginia Route Table Configuration](../screenshots/54-virginia-route-table-configuration.png)

---

# 6. Create VPC Peering Connection

The VPC Peering connection was created from the Mumbai region.

The Mumbai VPC was selected as the Requester VPC:

    Requester Region: Mumbai (ap-south-1)
    Requester VPC CIDR: 31.0.0.0/16

The N. Virginia VPC was selected as the Accepter VPC:

    Accepter Region: N. Virginia (us-east-1)
    Accepter VPC CIDR: 41.0.0.0/16

The peering connection was named:

    mu1-vgr1-peering-connection

![VPC Peering Request Configuration](../screenshots/55-vpc-peering-request-configuration.png)

The peering connection was created as a request from the Mumbai VPC to the N. Virginia VPC.

---

# 7. VPC Peering Pending Acceptance

After creating the peering connection, it appeared in the N. Virginia region with the status:

    Pending acceptance

At this stage, the peering connection had been requested but had not yet been accepted by the accepter VPC.

![VPC Peering Pending Acceptance](../screenshots/56-vpc-peering-pending-acceptance.png)

---

# 8. Accept VPC Peering Request

The peering request was opened in the N. Virginia region.

The Accept request action was selected to approve the VPC Peering connection.

![VPC Peering Acceptance](../screenshots/57-vpc-peering-acceptance.png)

---

# 9. VPC Peering Connection Active

After accepting the request, the VPC Peering connection changed to:

    Active

The peering connection was now established between:

    Mumbai VPC
    31.0.0.0/16

and:

    N. Virginia VPC
    41.0.0.0/16

![VPC Peering Active](../screenshots/58-vpc-peering-active.png)

The peering connection details confirmed:

    Requester Region: Mumbai (ap-south-1)
    Requester CIDR:   31.0.0.0/16

    Accepter Region:  N. Virginia (us-east-1)
    Accepter CIDR:    41.0.0.0/16

    Status: Active

---

# 10. Launch Mumbai EC2 Instance

An EC2 instance was launched in the existing Mumbai VPC.

The instance was placed in the Mumbai public subnet:

    VPC CIDR:    31.0.0.0/16
    Subnet CIDR: 31.0.1.0/24

A public IPv4 address was enabled so that the instance could be accessed through SSH.

The security group allowed:

- SSH (`TCP 22`)
- ICMP IPv4

![Mumbai EC2 Network Configuration](../screenshots/59-mumbai-ec2-network-configuration.png)

---

# 11. Launch N. Virginia EC2 Instance

A second EC2 instance was launched in the N. Virginia VPC.

The instance was placed in the N. Virginia public subnet:

    VPC CIDR:    41.0.0.0/16
    Subnet CIDR: 41.0.1.0/24

A public IPv4 address was enabled.

The security group allowed:

- SSH (`TCP 22`)
- ICMP IPv4

![N. Virginia EC2 Network Configuration](../screenshots/60-virginia-ec2-network-configuration.png)

---

# 12. Mumbai EC2 Instance Running

The Mumbai EC2 instance was successfully launched and reached the running state.

The instance received the private IPv4 address:

    31.0.1.139

![Mumbai EC2 Running](../screenshots/61-mumbai-ec2-running.png)

---

# 13. N. Virginia EC2 Instance Running

The N. Virginia EC2 instance was successfully launched and reached the running state.

The instance received the private IPv4 address:

    41.0.1.223

![N. Virginia EC2 Running](../screenshots/62-virginia-ec2-running.png)

---

# 14. Test Connectivity Before Updating Routes

At this point, the VPC Peering connection was already Active.

However, the route tables did not yet contain routes pointing to the remote VPC through the VPC Peering connection.

From the Mumbai EC2 instance, I attempted to ping the private IP address of the N. Virginia EC2 instance:

    ping 41.0.1.223

The ping failed and resulted in packet loss.

![Mumbai to N. Virginia Ping Failed](../screenshots/63-mumbai-to-virginia-ping-failed.png)

This demonstrated an important point:

> Creating and activating a VPC Peering connection alone does not automatically add routes to the VPC route tables.

The route tables must explicitly contain routes for the remote VPC CIDR.

---

# 15. Add VPC Peering Route in Mumbai

The Mumbai public route table was updated to route traffic destined for the N. Virginia VPC through the VPC Peering connection.

The destination was:

    41.0.0.0/16

The target was:

    VPC Peering Connection

![Mumbai Route Table Peering Route](../screenshots/64-mumbai-route-table-peering-route.png)

The resulting routing logic is:

    Mumbai EC2
        |
        v
    Mumbai Route Table
        |
        | 41.0.0.0/16
        v
    VPC Peering Connection
        |
        v
    N. Virginia VPC

---

# 16. Add VPC Peering Route in N. Virginia

The N. Virginia public route table was then updated to route traffic destined for the Mumbai VPC through the same VPC Peering connection.

The destination was:

    31.0.0.0/16

The target was:

    VPC Peering Connection

![N. Virginia Route Table Peering Route](../screenshots/65-virginia-route-table-peering-route.png)

The resulting routing logic is:

    N. Virginia EC2
        |
        v
    N. Virginia Route Table
        |
        | 31.0.0.0/16
        v
    VPC Peering Connection
        |
        v
    Mumbai VPC

---

# 17. Test Mumbai to N. Virginia Connectivity

After updating the route tables, I returned to the Mumbai EC2 instance.

The N. Virginia EC2 private IP was:

    41.0.1.223

I ran:

    ping 41.0.1.223

The ping was successful.

The Mumbai EC2 instance was able to communicate with the N. Virginia EC2 instance using its private IP address.

![Mumbai to N. Virginia Ping Success](../screenshots/66-mumbai-to-virginia-ping-success.png)

Example successful response:

    64 bytes from 41.0.1.223: icmp_seq=1 ttl=64
    64 bytes from 41.0.1.223: icmp_seq=2 ttl=64
    64 bytes from 41.0.1.223: icmp_seq=3 ttl=64

This confirmed that traffic from the Mumbai VPC could reach the N. Virginia VPC through the VPC Peering connection.

---

# 18. Test N. Virginia to Mumbai Connectivity

The reverse direction was also tested.

From the N. Virginia EC2 instance, I ran:

    ping 31.0.1.139

The ping was successful.

![N. Virginia to Mumbai Ping Success](../screenshots/67-virginia-to-mumbai-ping-success.png)

Example successful response:

    64 bytes from 31.0.1.139: icmp_seq=1 ttl=64
    64 bytes from 31.0.1.139: icmp_seq=2 ttl=64
    64 bytes from 31.0.1.139: icmp_seq=3 ttl=64

This confirmed that communication was working in both directions.

---

# 19. Final VPC Peering Architecture

The final architecture consists of two VPCs in different AWS Regions connected through a VPC Peering connection.

    AWS
     |
     +-----------------------+
     |                       |
     |     VPC Peering       |
     |                       |
     +-----------------------+
              |
       +------+------+
       |             |
    Mumbai       N. Virginia
      VPC             VPC
    31.0.0.0/16    41.0.0.0/16
       |               |
       |               |
    Public          Public
    Subnet          Subnet
    31.0.1.0/24    41.0.1.0/24
       |               |
       |               |
    Mumbai EC2      Virginia EC2
    31.0.1.139     41.0.1.223
       |               |
       +------ Ping ---+

The final traffic flow is:

    Mumbai EC2
    31.0.1.139
          |
          | Destination: 41.0.0.0/16
          v
    Mumbai Route Table
          |
          v
    VPC Peering Connection
          |
          v
    N. Virginia Route Table
          |
          v
    Virginia EC2
    41.0.1.223

And the reverse path:

    Virginia EC2
    41.0.1.223
          |
          | Destination: 31.0.0.0/16
          v
    N. Virginia Route Table
          |
          v
    VPC Peering Connection
          |
          v
    Mumbai Route Table
          |
          v
    Mumbai EC2
    31.0.1.139

---

# 20. Key Concepts Learned

## VPC Peering

VPC Peering creates a private network connection between two VPCs.

The VPCs can be in:

- The same AWS Region
- Different AWS Regions
- The same AWS account
- Different AWS accounts

In this lab, the VPCs were in different AWS Regions within the same AWS account.

---

## Requester and Accepter

When creating a VPC Peering connection, one VPC acts as the Requester and the other acts as the Accepter.

In this lab:

    Requester:
    Mumbai VPC
    ap-south-1

    Accepter:
    N. Virginia VPC
    us-east-1

The accepter must accept the peering request before the connection becomes active.

---

## Route Tables Are Required

VPC Peering does not automatically modify the route tables.

Routes must be configured manually on both sides.

For Mumbai:

    Destination: 41.0.0.0/16
    Target:      VPC Peering Connection

For N. Virginia:

    Destination: 31.0.0.0/16
    Target:      VPC Peering Connection

Without these routes, the VPCs remain unable to communicate even if the peering connection itself is active.

---

## Private IP Communication

The connectivity test was performed using the EC2 instances' private IP addresses:

    Mumbai:
    31.0.1.139

    N. Virginia:
    41.0.1.223

The public IP addresses were not used for the VPC-to-VPC connectivity test.

---

# 21. VPC Peering Limitation

VPC Peering uses a one-to-one relationship between VPCs.

For example, if there are three VPCs:

    VPC-A
    VPC-B
    VPC-C

Creating:

    VPC-A <-> VPC-B

does not automatically allow:

    VPC-A <-> VPC-C

A separate peering connection would be required.

Similarly, VPC Peering does not provide transitive routing.

For example:

    VPC-A <-> VPC-B
    VPC-B <-> VPC-C

does not automatically mean:

    VPC-A <-> VPC-C

This limitation becomes important when designing architectures with many VPCs. For larger environments, AWS Transit Gateway can be considered for centralized connectivity.

---

# 22. Final Result

The VPC Peering configuration was successfully completed between the Mumbai and N. Virginia VPCs.

### Final Configuration

    Mumbai VPC
    Region:          ap-south-1
    CIDR:            31.0.0.0/16
    EC2 Private IP:  31.0.1.139
             |
             |
             | VPC Peering
             |
             v
    N. Virginia VPC
    Region:          us-east-1
    CIDR:            41.0.0.0/16
    EC2 Private IP:  41.0.1.223

### Connectivity Verification

    Mumbai -> N. Virginia
    PING 41.0.1.223
    SUCCESS

    N. Virginia -> Mumbai
    PING 31.0.1.139
    SUCCESS

The lab demonstrated that two VPCs in different AWS Regions can communicate privately through VPC Peering when the peering connection is active and the appropriate routes are configured in both VPC route tables.