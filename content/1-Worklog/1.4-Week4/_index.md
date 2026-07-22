---
title: "Week 4 Worklog"
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

- Learn about virtual machines on AWS.
- What operating systems are available and which ones are the most commonly used on AWS.
- How EC2 is managed and why it is widely used.

### Tasks to be carried out this week:

| Day | Task                                                                  | Start Date | Completion Date | Reference Material                                                                            |
| --- | --------------------------------------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance type AWS.           | 25/05/2026 | 26/05/2026      |
| 2   | - Amazon Elastic Compute Cloud ( EC2 ) - AMI / Backup / Key Pair EC2. | 26/05/2026 | 26/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon Elastic Compute Cloud ( EC2 ) - Elastic block store.         | 27/05/2026 | 27/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance store               | 27/05/2026 | 27/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Amazon Elastic Compute Cloud ( EC2 ) - User data                    | 28/05/2026 | 28/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 6   | - Amazon Elastic Compute Cloud ( EC2 ) - Meta data                    | 28/05/2026 | 29/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 7   | - Amazon Elastic Compute Cloud ( EC2 ) - EC2 auto scaling             | 30/05/2026  | 30/05/2026       | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 8   | - EC2 Autoscaling - EFS/FSx - Lightsail - MGN                         | 31/05/2026  | 31/05/2026       | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 4 Achievements:

- Overview of AWS Virtualization Ecosystem
    - AWS delivers compute and storage services tailored for diverse workloads, including Amazon EC2, Amazon Lightsail, Amazon EFS/FSx, and AWS Application Migration Service (MGN).

  - Core Concepts of Amazon EC2
    - Definition: A high-speed, flexible virtual machine infrastructure designed to replace traditional physical servers, supporting web hosting, enterprise applications, and databases.

    - Instance Sizing Factors: Defined by compute power (CPU/GPU), memory capacity (RAM), network throughput, and attached storage.

    - Machine Images & Safeguards (AMI & Key Pairs):

      - AMI (Amazon Machine Image): Pre-configured templates containing root OS, permissions, and block mappings used to launch single or multiple instances (sourced from AWS, Marketplace, or custom-built).

      - Backup & Authentication: Utilizes Snapshots for point-in-time state backups and Key Pairs for secure credential encryption.

  - Storage Architecture:

    - Amazon EBS (Elastic Block Store):

      - Network-attached block storage operating independently from the EC2 lifecycle.

      - Offers HDD and SSD tiers, maintaining 99.999% availability by mirroring data across 3 nodes within an AZ.

      - Standard EBS volumes connect to a single instance, but Nitro-based instances support EBS Multi-Attach to share a volume across multiple hosts.

      - Initial Snapshots are full backups; subsequent ones are stored incrementally in Amazon S3.

    - Instance Store:

      - Directly attached ephemeral NVMe storage providing ultra-low latency and high IOPS.

      - Data Persistence: Persists through reboots, but data is permanently wiped if the instance is stopped or suffers hardware failure. Typically paired with EBS for secondary replication.

    - Provisioning & Automation Controls:

      - User Data: Bootstrapping scripts (Bash for Linux, PowerShell for Windows) executed once during initial instance creation.

      - EC2 Metadata: Instance-specific runtime information accessible from within the server (IP addresses, Hostname, Security Groups).

      - EC2 Auto Scaling: Dynamically expands or contracts capacity based on demand metrics, spanning multiple AZs, integrating with Load Balancers, and utilizing varied pricing models.

  - Pricing Schemes & Supporting Services
    - 4 EC2 Purchasing Options:

      - On-Demand: Pay-as-you-go pricing without long-term commitments; ideal for spiky or temporary workloads.

      - Reserved Instances: Contractual 1-to-3-year commitments offering deep discounts for specific instance families.

      - Savings Plans: Flexible 1-to-3-year commitment plans offering discounted hourly rates across instance types.

      - Spot Instances: Unused EC2 capacity available at massive discounts, subject to termination with a 2-minute warning.

    - Amazon Lightsail:

      - All-in-one VPS solution starting at $3.5/month, featuring bundled data transfers.

      - Tailored for lightweight applications, staging environments, or low CPU-bound workloads (<2 hours/day).

      - Operates in an isolated VPC with VPC Peering and snapshot capabilities.

    - Shared File Systems (EFS & FSx):

      - Amazon EFS: Linux-exclusive NFSv4 shared storage that scales elastically to petabytes, charging solely for consumed capacity. Connects seamlessly to hybrid On-Premises environments via Direct Connect or VPN.

      - Amazon FSx: Fully managed Windows-compatible SMB/NTFS file storage supporting both Linux and Windows. Features built-in data deduplication to slash storage costs by 30–50%.

    - AWS Application Migration Service (MGN):

      - Automated migration tool used to continuously replicate physical or virtual servers into AWS for Disaster Recovery (DR) or cloud adoption.

      - Uses low-cost staging instances in AWS to keep storage continuously synced with minimal overhead.

      - During the cutover phase, source servers shut down as production-ready EC2 instances spin up on AWS.