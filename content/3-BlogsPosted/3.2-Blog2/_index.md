---
title: "Blog 2"
date: "2026-07-09"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# How Alight Solutions achieved 55% cost savings with Amazon OpenSearch Service
<span class="meta-info">by Mark Larson, Andrew Kummerow, and Tim Razik | on 09 JUL 2026 | in</span> [Advanced (300)](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-redshift-analytics/), [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/) | [Permalink](https://aws.amazon.com/blogs/big-data/how-alight-solutions-achieved-55-cost-savings-with-amazon-opensearch-service/)

[Alight Solutions](https://www.alight.com/) is a leading cloud-based human capital technology and services provider. Alight’s technology stack generates over 1 billion log records per day across their containerized microservices architecture, with peaks reaching 100,000 records per second. 

Previously, Alight relied on a self-managed Elastic Stack deployment. As logging volumes grew, the operational burden of maintaining this infrastructure consumed their entire operational budget, leaving no capacity for innovation. This post shares how Alight migrated to [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/) and achieved a 55% cost reduction.

---

## Challenges with Self-Managed Elasticsearch

Alight’s self-managed Elastic Stack presented compounding challenges:
* **High Maintenance Overhead:** Routine tasks like security vulnerability patching and Elastic upgrades consumed thousands of engineering hours (equivalent to 1 FTE).
* **Unreliable Ingestion:** Logstash using TCP-socket shipping experienced log loss at high volumes.
* **P1 Incidents:** Backpressure from Logstash caused two P1 incidents where the logging subsystem directly impacted microservice tasks.
* **End of Support:** Elasticsearch 7.x approaching end of support created an urgent need for migration.

---

## Evaluating Alternatives

Alight evaluated several solutions, including New Relic, Dynatrace, and Amazon CloudWatch. They selected **Amazon OpenSearch Service** based on five key factors:
1. **Cost:** Significantly cheaper than competing solutions.
2. **Minimal Change Management:** As a fork of Elasticsearch 7.10, engineers were already familiar with the query syntax and dashboards.
3. **Compliance:** Using a native AWS service avoided extensive regulatory audits.
4. **Cloud-Native Strategy:** Aligned with Alight’s overriding strategy.
5. **Data Privacy:** Keeping data within their AWS landing zone avoided egress concerns.

---

## Solution Architecture

Alight partnered with AWS to design a cloud-native log aggregation architecture:

![Cross-account architecture diagram](/images/3-Blogs/Blog-2/image-1.jpeg)
<span class="meta-info">*Figure 1 — Cross-account log ingestion architecture*</span>

* **Cross-Account Design:** ECS and EC2 workloads in application accounts send logs to the OpenSearch domain located in a Shared Services account.
* **Ingestion Paths:** 
  * **ECS Applications:** Fluent Bit / FireLens sidecars ship logs over HTTPS to Amazon OpenSearch Ingestion (OSIS).
  * **EC2 Applications:** Fluent Bit RPM packages read log files and ship to OSIS.
* **Persistent Buffering:** Amazon EFS provides persistent filesystem buffering for Fluent Bit to prevent log loss during spikes.
* **SSO Authentication:** User access is governed via AWS IAM Identity Center and SCIM synchronization.

---

## Migration Process
The migration was completed over seven months (February to August 2025). The team used **Terraform** to automate the deployment of OSIS pipelines and OpenSearch domains. Onboarding new applications was reduced from 80-120 hours to just 4-8 hours. The team executed parallel runs for validation before fully shutting down the old Elasticsearch clusters.

---

## Results and Benefits

The migration delivered substantial improvements across cost, operations, and scale:

### Cost & Resource Efficiency
* **Cost Reduction:** Achieved a 55% monthly infrastructure and licensing cost reduction (projected to reach 65% once decommissioning is complete).
* **Zero Licensing Cost:** Eliminated the Elastic Platinum subscription.

### Operational Improvements

| Metric | Self-Managed Elasticsearch | Managed OpenSearch Service |
| :--- | :--- | :--- |
| **Engineering Overhead** | ~2,000 hours/year | Near zero (AWS managed) |
| **Patching & Maintenance** | Manual (including holiday work) | Fully managed by AWS |
| **Application Onboarding** | 80–120 hours (manual) | 4–8 hours (Terraform) |
| **Log-induced P1 Incidents** | 2 in past 2 years | Zero since migration |

---

## Lessons Learned
* **Separate Concerns for Durability:** Do not place a 100% delivery guarantee on logging infrastructure. Use Amazon SQS for critical data that cannot tolerate loss.
* **Infrastructure as Code (IaC):** Standardizing with Terraform modules drastically reduces time-to-market.
* **Plan Around Peak Periods:** Pausing the production rollout during peak enrollment periods was critical to avoiding business disruptions.

For more details, see the original post on [AWS Big Data Blog: How Alight Solutions achieved 55% cost savings with Amazon OpenSearch Service](https://aws.amazon.com/vi/blogs/big-data/how-alight-solutions-achieved-55-cost-savings-with-amazon-opensearch-service/).