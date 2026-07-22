---
title: "Architecture Walkthrough"
weight: 52
chapter: false
pre: " <b> 5.2. </b> "
---

![Architecture](/images/architecture.png)
<p align="center"><em>Figure: Architecture Batch-base Clickstream Analytics Platform.</em></p>

---

## 5.2.1 User & Frontend Domain

### (1) User Browser – End User

**Responsibilities**

- Visits the website: browses the product catalogue, views product detail pages, manages the cart, completes checkout, etc.
- Creates or signs into an account.
- Triggers **clickstream events** such as:
  - `page_view`, `product_view`, `add_to_cart`, `purchase`, …

**Role in the pipeline**

- Is the **originating source of all behavioral signals** that the analytics system processes.
- The JavaScript layer embedded in the frontend gathers these events and dispatches them to the backend through the API (6).

---

### (2) Amazon CloudFront (CDN / Edge)

**Responsibilities**

- Operates as the **CDN layer** responsible for delivering website content:
  - Caches HTML/CSS/JS/images to cut down on latency.
  - Offloads traffic pressure from the origin (Amplify, S3 assets).
- Acts as a **unified entry point** for all web traffic:
  - Routes and forwards requests to the appropriate origin:
    - Dynamic web content → Amplify.
    - Static images/media → S3 (3).

**Why it matters**

- An e-commerce site demands **fast page load times**.  
- CloudFront improves the user experience across geographic regions by serving content from edge locations.

---

### (3) Amazon S3 – Media Assets

**Responsibilities**

- Stores **product images** and other **static website assets**.
- Functions as the **origin** from which CloudFront (2) serves images and other static files.

**Notes**

- This bucket (e.g., `clickstream-s3-sbw`) is kept **separate** from the RAW clickstream bucket (8) in order to:
  - Maintain clear permission boundaries.
  - Simplify management and eventual cleanup.

---

### (4) Amazon Amplify – Front-end Hosting

**Responsibilities**

- Builds and deploys the **Next.js** application (e.g., `ClickSteam.NextJS`).
- Hosts the frontend (SSR/ISR, API routes).
- Manages the full deployment pipeline: build, artifact storage, and domain mapping.

**Integration with other services**

- Connects to **Cognito (5)** to handle user authentication flows.
- Pushes **clickstream events** from the frontend to **API Gateway (6)**.
- Talks to **EC2 OLTP (20)** via Prisma to read and write transactional data.

---

### (5) Amazon Cognito – Authentication / Authorization

**Responsibilities**

- Manages **user identity**:
  - Sign up / sign in / password reset flows.
  - Maintains the user pool and issues JWT tokens (ID token, access token).
- Provides the frontend with tokens it can use to call backend APIs with the correct permissions.

**Role in clickstream**

- Enables the system to determine:
  - `user_login_state` (logged-in / guest).
  - `identity_source` (Cognito, social login, etc.).
- This context is embedded in events and stored in S3/DWH to support user behavior analysis.

---

## 5.2.2 Ingestion & Data Lake Domain

### (6) Amazon API Gateway – Clickstream HTTP API

**Responsibilities**

- Exposes a public HTTP endpoint for the frontend:
  - `POST /clickstream`.
- Applies basic request validation:
  - HTTP method/path checking, throttling.
  - Optional: Cognito authorizer integration.

**Processing flow**

- Receives JSON payloads from the browser.
- Forwards each payload to **Lambda Clickstream Ingest (7)**.

---

### (7) AWS Lambda – Clickstream Ingest

**Main responsibilities**

- Receives events forwarded by API Gateway (6).
- Serves as the **ingestion layer**:

  - Validates the schema and event type.
  - Performs lightweight enrichment:
    - Generates a unique `event_id`.
    - Normalizes the `event_timestamp`.
    - Attaches `client_id`, `session_id`, `user_login_state`, `identity_source` as needed.

- Writes raw events (`raw JSON`) to the **S3 Clickstream Raw bucket (8)** using time-based prefixes.

- Outputs logs to **CloudWatch (17)** for observability and debugging.

---

### (8) Amazon S3 – Clickstream Raw Data

**Responsibilities**

- Stores all **raw clickstream data** deposited by Lambda Ingest (7).
- Acts as a **"mini data lake"** for downstream batch ETL:

  - Data is organized into **time-based folder prefixes**:
    - `events/YYYY/MM/DD/`.
  - ETL (11) reads data in batches (hourly, daily, etc.).

**Architectural role**

- Serves as the **source of truth** for all clickstream data:
  - The Data Warehouse can be rebuilt or reprocessed from this bucket at any time.

---

### (9) Amazon EventBridge – ETL Trigger / Cron Job

**Responsibilities**

- Executes batch schedules, for example: `rate(1 hour)`.
- On each scheduled trigger:
  - Invokes **Lambda ETL (11)**.

**Why it is isolated**

- The ETL "clock" lives entirely in EventBridge:
  - Cadence is easy to adjust (1h, 30m, 5m, etc.).
  - Can be disabled or re-enabled without touching the Lambda code.
  - Cleanly separates "when to run" from the ETL business logic.

---

## 5.2.3 VPC & Private Analytics Domain

### (19) VPC + Subnets + Internet Gateway

**Amazon VPC**

- Provides network isolation and controls routing and security for the entire platform.

**Public subnet – OLTP**

- Houses **EC2 OLTP (20)**.
- Has a default route: `0.0.0.0/0 → Internet Gateway`.
- Allows:
  - Amplify and internet traffic to reach PostgreSQL OLTP, governed by security groups.

**Private subnet – Analytics**

- Houses:
  - **EC2 Shiny + DWH (12)**.
  - **Lambda ETL (11)**.
- No public IP is assigned.
- Egress is only possible through **VPC Endpoints (10, 15)**.

**Internet Gateway**

- Provides internet connectivity for resources in the public subnet.
- The private subnet has no route to the IGW (unless explicitly configured otherwise).

---

### (10) Gateway Endpoint – S3 Gateway Endpoint

**Responsibilities**

- Allows VPC-resident resources (Lambda ETL, private EC2) to access S3 (8) **without routing through NAT or the public Internet**.
- Traffic to S3 stays within the **AWS private backbone network**, which is more secure and cost-effective than a NAT Gateway.

**Role**

- A foundational piece that enables:
  - Maintaining the "**no NAT Gateway**" design decision.
  - Allowing ETL and DW components to read from and write to S3.

---

### (11) AWS Lambda – ETL

**Responsibilities**

- Triggered by **EventBridge (9)** on a schedule.
- Executes inside the **private subnet** of the VPC.
- Runs batch ETL operations:

  - Reads raw events from **S3 (8)** via the Gateway Endpoint (10).
  - Transforms, cleans, and flattens data to match the Data Warehouse schema (e.g., the 15 confirmed fields).
  - Loads the result into **PostgreSQL DWH on private EC2 (12)**:
    - Insert / upsert, optionally partitioned by day or hour.

- Writes logs and metrics to **CloudWatch (17)**; on errors, it can fire notifications via **SNS (18)**.

---

### (12) Amazon EC2 – Private (Shiny + DWH)

**Responsibilities**

- Hosts the **Data Warehouse (PostgreSQL)** and the **Shiny Server**.
- Accepts clean, transformed data from Lambda ETL (11) and persists it in DWH tables.
- Supplies data to Shiny analytics dashboards:
  - Shiny runs queries against the local private Postgres instance.

**Characteristics**

- Runs inside a **private subnet**.
- Has no public IP address.
- Administrative access is managed through **SSM Session Manager (14 + 15)**.

---

### (13) R Shiny Server (on Private EC2)

**Responsibilities**

- Hosts analytics dashboards covering:

  - Top-level KPIs.
  - Conversion funnel visualization.
  - Product performance rankings.
  - User behavior and session analysis.

- Queries **PostgreSQL DWH** running on the same EC2 instance.

**Access**

- Listens on port `3838`.
- Typically accessed through:

  - An internal VPN, or
  - **SSM port forwarding** (14).

---

### (14) AWS Systems Manager – Session Manager

**Responsibilities**

- Grants admins access to private EC2 instances **without SSH and without public IP addresses**.
- Supports:

  - Opening an interactive terminal session on EC2 (run command).
  - **Port forwarding** (e.g., forward `localhost:3838` → Shiny running on EC2).
  - Improved session audit logging.

**Example**

- You can reach Shiny from your local machine at:  
  `http://localhost:3838/sbw_dashboard`  
  once the SSM port forwarding session is established.

---

### (15) Interface Endpoint – SSM VPC Endpoints

**Responsibilities**

- Establishes **private** connectivity between EC2/Lambda inside the VPC and the SSM/Session Manager service endpoints:

  - `ssm`, `ssmmessages`, `ec2messages`, …

- Ensures that:

  - Private EC2 instances can still use SSM.
  - Traffic never traverses the internet or a NAT gateway.

---

## 5.2.4 Cross-cutting Services

### (16) Amazon IAM

**Responsibilities**

- Governs **roles and policies** across the entire system:

  - Permissions for API Gateway and Lambda.
  - Lambda Ingest: permission to write to S3 (8).
  - Lambda ETL: permission to read S3 (8) and connect to the DB (12).
  - EC2 instance role for SSM and CloudWatch agent (if deployed).

**Principles**

- **Least privilege** at all times.
- Permissions are clearly scoped by function and by service.

---

### (17) Amazon CloudWatch

**Responsibilities**

- Collects **logs and metrics** from:

  - Lambda Ingest (7).
  - Lambda ETL (11).
  - EventBridge rule (9).
  - EC2, if the CloudWatch agent is installed.

- Defines **alarms** for: ETL failures, ingest errors, retry counts, execution duration, etc.

**Role**

- The **primary observability hub** for diagnosing pipeline problems.

---

### (18) Amazon SNS

**Responsibilities**

- Acts as the **notification channel** for operational incidents:

  - ETL job failure.
  - Ingest function failure.
  - CloudWatch alarm activation.

- Delivers notifications via email / SMS / webhook based on subscription configuration.

---

### (20) Amazon EC2 – Public (OLTP)

**Responsibilities**

- Runs **PostgreSQL OLTP** to serve the e-commerce web application:

  - Order transactions.
  - Product catalog and inventory data.
  - User account information.

**Relation to DWH**

- **Kept separate** from the Data Warehouse (12):

  - OLTP is optimized for **transactional read/write operations**.
  - DWH is optimized for **analytics queries and batch data loads**.
