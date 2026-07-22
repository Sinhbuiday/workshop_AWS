---
title: "Summary & Clean up"
weight: 56
chapter: false
pre: " <b> 5.6. </b> "
---

## 5.6.1 Summary

With the lab complete, we have successfully deployed a functional **Clickstream Analytics Platform**:

1. **User-Facing Layer**
   - The Next.js application (`ClickSteam.NextJS`) hosted via Amplify and CloudFront  
   - Identity management and authentication powered by Cognito  
   - A PostgreSQL OLTP database (`clickstream_web`) running on `SBW_EC2_WebDB` (located in a public subnet)  

2. **Ingestion & Raw Data Layer**
   - The API Gateway HTTP API: `clickstream-http-api` (exposing `POST /clickstream`)  
   - The Lambda Ingest function: `clickstream-lambda-ingest`  
   - The S3 Raw storage bucket: `clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json`  

3. **Private Analytics Layer**
   - A custom VPC configured with both public and private subnets (`SBW_Project_VPC`)  
   - Secure routing via an S3 Gateway Endpoint and SSM Interface Endpoints  
   - A Data Warehouse on EC2: `SBW_EC2_ShinyDWH`, managing the `clickstream_dw` database  
   - A VPC-bound ETL Lambda: `SBW_Lamda_ETL`, invoked on schedule by `SBW_ETL_HOURLY_RULE`  
   - Analytics visualizations via R Shiny dashboards (`sbw_dashboard`), securely accessed through SSM port forwarding  

In summary, this architecture highlights best practices for designing a **batch-based analytics platform** that prioritizes security and cost optimization, utilizing a mix of serverless services and targeted EC2 compute.

---

## 5.6.2 Key Takeaways

- **Separation of concerns**:
  - The OLTP and Analytics workloads are strictly separated across two different EC2 instances, isolating their logical domains and performance requirements.  

- **Security**:
  - The Data Warehouse and Shiny Server operate in a private subnet, shielded from direct internet access.  
  - SSH access is entirely replaced by the more secure SSM Session Manager.  
  - The S3 Gateway Endpoint ensures that S3 traffic never leaves the AWS private backbone.  

- **Cost optimization**:
  - The architecture completely avoids the cost of a NAT Gateway.  
  - The ETL pipeline is driven by cost-effective serverless components (Lambda + EventBridge).  
  - Amazon S3 provides highly durable, low-cost storage for the raw event data.  

- **Easy to extend**:
  - While currently engineered for batch processing, the foundation can be adapted for real-time streaming, advanced machine learning analytics, or migrated to enterprise-grade DW solutions.

---

## 5.6.3 Resource Clean-up

1. **Amplify & CloudFront**
   - Delete the Amplify application (`ClickSteam.NextJS`).  
   - This process automatically tears down the associated CloudFront distribution.

2. **API Gateway & Lambda**
   - Remove the `clickstream-http-api` API Gateway.  
   - Delete the following Lambda functions:
     - `clickstream-lambda-ingest`  
     - `SBW_Lamda_ETL`  

3. **EventBridge**
   - Delete the scheduling rule named `SBW_ETL_HOURLY_RULE`.  

4. **S3 Buckets**
   - Empty the contents, then delete the following buckets:
     - `clickstream-s3-ingest` (the RAW clickstream store)  
     - `clickstream-s3-sbw` (the assets bucket), provided it is not shared with other projects  

5. **EC2 Instances**
   - Stop or permanently terminate:
     - `SBW_EC2_WebDB`  
     - `SBW_EC2_ShinyDWH`  
   - Release any Elastic IPs that were attached to these instances.

6. **VPC & Networking**
   - Remove all VPC endpoints (S3 Gateway, and all SSM Interface Endpoints).  
   - Delete the route tables, subnets, and finally the Internet Gateway.  
   - Once empty, delete the `SBW_Project_VPC` itself.
