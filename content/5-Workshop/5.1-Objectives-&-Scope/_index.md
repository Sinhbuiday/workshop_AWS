---
title: "Objectives & Scope"
weight: 51
chapter: false
pre: " <b> 5.1. </b> "
---

### Business Context

The target system is an e-commerce website that sells laptops, monitors, and computer accessories.  
The business wants to:

- Gain insight into **how users engage** with the website:
  - Which pages they navigate to
  - Which products they browse and view
  - Add-to-cart and checkout interactions
- Measure the **conversion funnel** across the journey: product view → add to cart → checkout
- Pinpoint:
  - High-performing products (bestsellers / most-viewed)
  - Peak traffic windows (by hour and by day)
- Leverage these insights to craft effective marketing strategies that:
  - Grow revenue
  - Optimize spending and reduce unnecessary costs

To accomplish this, the platform:

- Captures a dataset of user behavioral signals through clickstream events, then produces statistics and analytical reports.

---

### Learning Objectives

#### Understand the Architecture

- Be able to walk through the end-to-end architecture of a **batch-based clickstream analytics platform** that uses:
  - Amplify, CloudFront, Cognito, EC2 OLTP in the **user-facing domain**
  - API Gateway, Lambda Ingest, S3 Raw bucket in the **ingestion & data lake domain**
  - ETL Lambda, PostgreSQL DW on EC2, R Shiny in the **analytics & DW domain**
- Be able to articulate why OLTP and Analytics are kept separate:
  - **Logical**: different database schemas, different types of workloads
  - **Physical**: public subnet vs. private subnet, two distinct EC2 instances

#### Hands-on Skills

- Send clickstream events from the frontend through API Gateway → Lambda Ingest → S3 Raw (`clickstream-s3-ingest`).
- Set up a **Gateway VPC Endpoint for S3** and update the private route table so private components can reach S3.
- Configure and validate an **ETL Lambda** (`SBW_Lamda_ETL`) capable of:
  - Reading raw JSON files from `s3://clickstream-s3-ingest/events/YYYY/MM/DD/`
  - Transforming events into rows for the table `clickstream_dw.public.clickstream_events`
- Connect to the DW (`SBW_EC2_ShinyDWH`) and execute sample SQL queries:
  - Event count
  - Top products
  - Basic funnel analysis
- Access **R Shiny dashboards** through SSM port forwarding and interpret:
  - Funnel charts
  - Product engagement visualizations
  - Time-series activity charts

#### Security & Cost Awareness

- Understand why user behavioral data is protected by placing `EC2_ShinyDWH` and `Lambda_ETL` inside **private subnets**:
  - Only `Lambda_ETL` is allowed to reach S3 via a **Gateway VPC Endpoint**
  - Only administrators can access `EC2_ShinyDWH` via SSM using **Interface VPC Endpoints**
- Recognize the principal security controls in place:
  - Separation of public and private subnets
  - Security group boundaries between `sg_oltp_webDB`, `sg_Lambda_ETL`, `sg_analytics_ShinyDWH`
  - Least-privilege IAM permissions scoped per Lambda function
  - Zero-SSH administration via AWS Systems Manager Session Manager (no bastion host, no open SSH port).

---

### Workshop Scope

The workshop concentrates on three core capability areas:

1. **Implementing the clickstream ingestion layer**

   - Capture browser-side interactions within the Next.js frontend
   - Push JSON events to API Gateway (`clickstream-http-api`)
   - Persistently store raw events over time in `clickstream-s3-ingest`

2. **Building the private analytics layer**

   - Provision the VPC, subnets, route tables, and VPC endpoints
   - Run `SBW_Lamda_ETL` inside the VPC
   - Wire the ETL Lambda to the private EC2 Data Warehouse (`SBW_EC2_ShinyDWH`)

3. **Visualizing analytics with Shiny dashboards**
   - Query `clickstream_dw` directly from R Shiny
   - Render funnels, product performance metrics, and time-based trends
   - Access Shiny through **SSM Session Manager port forwarding**

---

### Out of Scope

To keep the workshop focused and achievable, we **do not** cover in depth:

- Real-time streaming solutions (Kinesis, Kafka, MSK, …)
- Managed DW services (Amazon Redshift / Redshift Serverless)
- Recommendation engines, user segmentation, or ML-based anomaly detection
- Production-grade CI/CD pipelines, blue/green deployments, or multi-account architectures
- Advanced SQL performance tuning or detailed index strategy

These represent natural next steps once the batch-based clickstream foundation established in this workshop is operational.
