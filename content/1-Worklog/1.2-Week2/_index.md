---
title: "Week 2 Worklog"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

- Study and understand what VPC and VPN are
- How to create, connect, and operate networking services
- Experimentation and results

### Tasks to be carried out this week:

| Day | Task                                          | Start Date | Completion Date | Reference Material                                                                            |
| --- | --------------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Learn the concepts of VPC and VPN           | 11/05/2026 | 12/05/2026      |
| 2   | - Practice: Use AWS services with VPC and VPN | 12/05/2026 | 13/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Create **Subnets**, **Internet Gateway**    | 13/05/2026 | 13/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Create Route **Table**, **security groups** | 14/05/2026 | 15/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Create **Route Table**, **security groups** | 16/05/2026 | 17/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 2 Achievements:

Understanding VPC and VPN 
  - Amazon VPC (Virtual Private Cloud)

    - Amazon VPC lets you provision an isolated virtual network where you can launch AWS  resources like virtual servers, with full control over networking configuration.
    - A single VPC can stretch across multiple Availability Zones (AZs), but it's always    confined to one Region.
    - When setting one up, specifying an IPv4 CIDR block is required, while IPv6 is an optional addition.
    - Each account is capped at 5 VPCs per Region.
    - Its main purpose is to keep different environments isolated from one another (e.gProduction, Dev, Test, Staging).
    -Instances can be organized into subnets — each subnet must live within a single AZ and use a CIDR range that falls within the VPC's overall CIDR block.
  - Every subnet automatically reserves 5 IP addresses for internal use:
    - Network address (e.g., 10.10.1.0)
    - Broadcast address (e.g., 10.10.1.255)
    - Router address (10.10.1.1)
    - DNS address (10.10.1.2)
    - Reserved for future use (10.10.1.3)

  - Route Tables

    - A route table is essentially a rulebook of paths that directs where network traffic goes.
    - The Main/Default Route Table can't be deleted — it contains one built-in route enabling subnets within the VPC to talk to each other. Every subnet needs to be linked to a route table.
    - Custom Route Tables can be created as needed, though the default local route can never be removed.

  - ENI (Elastic Network Interface)
    A virtual network interface card that attaches to EC2 instances.

    - Keeps networking configuration intact even if the underlying instance is swapped out.
    - Comes with a private IP.
    - Can have an Elastic IP.
    - Has its own MAC address.

  - VPC Endpoint
  Enables private, internet-free connectivity to AWS services from within your VPC.

    - Comes in two flavors: Interface Endpoint and Gateway Endpoint.

  - VPC Security Group
  A stateful firewall that governs inbound and outbound traffic for AWS resources.

    - Rules can reference source IPs, ports, or other Security Groups.
    - Only supports "allow" rules, and applies at the ENI level.

  - NACL (Network Access Control List)
  A stateless firewall operating at the subnet level.

    - Rules are based on source IP and port.
    - Applied directly to VPC subnets.
    - The default NACL permits all inbound and outbound traffic by default.

  - VPC Flow Logs
  Captures metadata about IP traffic flowing in and out of the VPC.

    - Logs can be sent to CloudWatch Logs or stored in S3.
    - Does not capture the actual packet payload/content.

  - VPC Peering
  Establishes direct connectivity between two VPCs.

    - Requires a dedicated 1:1 peering connection.
    - Overlapping IP ranges between peered VPCs aren't supported.

  - Transit Gateway
  Acts as a central hub connecting multiple VPCs together.

  - VPN Options

    - Site-to-Site VPN: Ideal for hybrid setups, maintaining a persistent connection between on-premises infrastructure and a VPC.
    - Client-to-Site VPN: Lets individual devices securely connect into VPC resources remotely.
    - AWS Direct Connect: A dedicated private link between on-premises data centers and AWS, typically achieving 20–30ms latency.

  - Elastic Load Balancing (ELB)
  A fully managed load balancing service.

    - Supports HTTP, HTTPS, secure TCP, and SSL protocols.
    - Can be configured as either public-facing or internal/private.
    - Comes in four types: Application Load Balancer (ALB), Network Load Balancer (NLB), Classic Load Balancer (CLB), and Gateway Load Balancer (GLB).

  - Hands-on Practice

  1. Search for VPC in the console → click Create VPC → assign it a name.
  2. Select IPv4 and enter the CIDR block 10.10.0.0/16.
  3. Create the necessary Subnets.
  4. Set up an Internet Gateway.
  5. Configure Security Groups.
