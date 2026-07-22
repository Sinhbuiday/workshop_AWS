---
title: "Building the Private Analytics Layer"
weight: 54
chapter: false
pre: " <b> 5.4. </b> "
---

## 5.4.1 VPC, Subnets, and Route Tables Configuration

- **VPC CIDR**: `10.0.0.0/16`  
- **Public subnet**: `10.0.0.0/20` → `SBW_Project-subnet-public1-ap-southeast-1a`  
  - Hosts `SBW_EC2_WebDB` (EC2 OLTP).  
- **Private subnet**: `10.0.128.0/20` → `SBW_Project-subnet-private1-ap-southeast-1a`  
  - Hosts `SBW_EC2_ShinyDWH` and `SBW_Lamda_ETL`.  

**Public Route Table**

- `10.0.0.0/16 → local`  
- `0.0.0.0/0 → Internet Gateway`  

**Private Route Table**

- `10.0.0.0/16 → local`  
- S3 prefix list → Gateway VPC Endpoint for S3  
- No `0.0.0.0/0` route to the IGW or a NAT Gateway  

---

## 5.4.2 VPC Endpoints (S3 & SSM)

![VPC Endpoints for S3 & SSM](/images/aws-vpc-endpoints-s3-ssm.png)

### S3 Gateway VPC Endpoint

- Enables **private network access to S3** for:
  - `SBW_Lamda_ETL`   
- Eliminates the need for a NAT Gateway.

### SSM Interface Endpoints

- `com.amazonaws.ap-southeast-1.ssm`  
- `com.amazonaws.ap-southeast-1.ssmmessages`  
- `com.amazonaws.ap-southeast-1.ec2messages`  

These endpoints allow **Session Manager** to administer and port-forward to `SBW_EC2_ShinyDWH` without requiring a public IP address or an open SSH port.

---

## 5.4.3 Data Warehouse on EC2 – `SBW_EC2_ShinyDWH`

On this private EC2 instance:

- PostgreSQL database: `clickstream_dw`  
- Main table: `clickstream_events` with the following fields:

```text
event_id
event_timestamp
event_name
user_id
user_login_state
identity_source
client_id
session_id
is_first_visit
context_product_id
context_product_name
context_product_category
context_product_brand
context_product_price
context_product_discount_price
context_product_url_path
```

The instance allows:

- `SBW_Lamda_ETL` to connect to the PostgreSQL database `clickstream_dw`.  
- Local web access to Shiny via SSM port forwarding.

---

## 5.4.4 ETL Lambda – `SBW_Lamda_ETL` (running in the Private subnet)

The ETL Lambda is where the core batch processing logic runs.

**VPC configuration:**

- Subnet: `SBW_Project-subnet-private1-ap-southeast-1a`  
- Security group: `sg_Lambda_ETL`  

**Environment variables:**

- `DWH_HOST`, `DWH_PORT=5432`, `DWH_USER`, `DWH_PASSWORD`, `DWH_DATABASE=clickstream_dw`  
- `RAW_BUCKET=clickstream-s3-ingest`  
- `AWS_REGION=ap-southeast-1`  

**Responsibilities:**

1. Determine the set of files in `s3://clickstream-s3-ingest/events/YYYY/MM/DD/` for the current batch to process.  
2. For each JSON file:  
   - Extract  
   - Transform  
   - Load  

**IAM role:**

- Grants the Lambda the permissions needed to reach `EC2_ShinyDWH` (e.g., via the database endpoint in the private subnet).

---

## 5.4.5 Scheduling with EventBridge – `SBW_ETL_HOURLY_RULE`

![EventBridge rule](/images/aws-eventbridge-sbw-etl-hourly-rule.png)

EventBridge keeps the platform operating in a **batch** cadence:

- **Rule name**: `SBW_ETL_HOURLY_RULE`  
- **Schedule**: `rate(1 hour)`  
- **Target**: `SBW_Lamda_ETL`  

Each time the rule fires:

1. The ETL Lambda starts up inside the private subnet.  
2. It reads new events from S3 through the Gateway Endpoint.  
3. It loads the processed data into `clickstream_dw`.  

You can also **trigger the ETL Lambda manually** (from the Lambda console) for backfill or testing purposes.

---

## 5.4.6 Summary of Security Groups & Connectivity

- `sg_Lambda_ETL`:
  - Outbound access to the S3 endpoint and to `sg_analytics_ShinyDWH:5432`.  

- `sg_analytics_ShinyDWH`:
  - Inbound `5432/tcp` from `sg_Lambda_ETL`.  
  - Inbound `3838/tcp` for Shiny (accessed only via SSM port forwarding).  
