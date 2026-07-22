---
title: "Proposal"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Team Super Beast Warrior(SBW)

# Batch-based Clickstream Analytics Platform

### 1. Executive Summary

The goal of this project is to build and deploy a **Batch-based Clickstream Analytics Platform** tailored for an e-commerce website specializing in computers and accessories. The website frontend embeds a lightweight JavaScript SDK that captures user activity events (clicks, page views, searches) and forwards them to a backend API. The entire platform runs on **AWS Cloud Services**.

Raw interaction data — including browsing events, search queries, and page visits — flows from the website into **Amazon S3**, where it is stored as unstructured log files. **Amazon EventBridge** fires on an hourly schedule to trigger **AWS Lambda** functions, which process and shape the data before it is loaded into a **data warehouse running on Amazon EC2**.

Processed data surfaces through **R Shiny dashboards**, giving store operators an at-a-glance view of customer behavior patterns, product popularity trends, and overall site engagement.

The architecture is centered on **batch analytics**, end-to-end **ETL pipelines**, and **business intelligence**, while maintaining strong **security**, **scalability**, and **cost efficiency** through the use of **AWS managed services**.

### 2. Problem Statement

#### What's the Problem?

E-commerce sites continuously generate massive volumes of **clickstream data** — product views, cart interactions, search activity — that carry rich business value.

Despite this potential, **small and medium-sized** businesses typically lack the tools and technical know-how to collect, process, and extract value from this data in a structured way.

Without a proper system, these businesses struggle to:

- Understand how customers navigate and make purchasing decisions
- Identify which products are driving the most revenue
- Tune marketing campaigns and improve site performance
- Make informed, evidence-based decisions on pricing and inventory planning

#### The Solution

This project proposes an **AWS-powered batch clickstream analytics** system that automatically gathers user interaction data on an hourly basis, passes it through serverless processing functions, and persists structured outputs into a central data warehouse hosted on **Amazon EC2**.

The results are rendered as interactive **R Shiny dashboards**, giving business owners clear, actionable views of customer behavior and areas for improvement.

#### Benefits and Return on Investment

- **Data-driven decision making**: Surface customer preferences, trending products, and seasonal purchasing patterns.
- **Scalable and modular design**: The architecture can be extended to accommodate growing traffic or additional data sources without redesigning from scratch.
- **Cost-efficient batch processing**: By running on a scheduled cadence, continuous compute costs are minimized compared to real-time streaming alternatives.
- **Business insight enablement**: Store owners gain analytical tools to refine sales strategies and grow revenue through evidence-based planning.

### 3. Solution Architecture

![Architecture](/images/2-Proposal/AWS_Architecture_2.jpg)

#### AWS Services Used

- **Amazon Cognito**: Manages authentication and access control for both administrators and website shoppers, ensuring only authorized users interact with the platform.
- **Amazon S3**: Serves as the central storage layer — hosting the website's static frontend assets and storing raw clickstream log files from user sessions. It also acts as a staging area for batch files awaiting processing and transfer to the data warehouse.
- **Amazon CloudFront**: Delivers static site content worldwide with minimal latency, improving page load speed and caching assets geographically closer to end users.
- **Amazon API Gateway**: Functions as the front door for incoming API requests from the website, providing a secure channel for submitting clickstream and browsing data into the AWS ecosystem.
- **AWS Lambda**: Runs serverless code to preprocess and structure incoming clickstream data written to S3. It also executes scheduled transformation jobs triggered by EventBridge that prepare data for ingestion into the data warehouse.
- **Amazon EventBridge**: Coordinates and schedules batch workflows — invoking Lambda functions at regular hourly intervals to move and transform clickstream data from S3 to the EC2 data warehouse.
- **Amazon EC2 (Data Warehouse)**: Hosts the data warehouse environment, running PostgreSQL or a compatible relational database system for batch analytics, historical trend analysis, and business reporting. Both EC2 instances reside in VPC private subnets for network isolation.
- **R Shiny (on EC2)**: Powers interactive analytics dashboards that display batch-processed results, helping the business team explore customer journeys, top products, and sales opportunities.
- **AWS IAM**: Enforces access control and permission boundaries across all AWS components, ensuring each service and user has only the privileges they require.
- **Amazon CloudWatch**: Monitors metrics, aggregates logs, and tracks scheduled job outcomes from Lambda and EC2, providing a unified view of system health and performance.
- **Amazon SNS**: Dispatches notifications and alerts when batch jobs complete successfully, fail, or encounter unexpected errors, keeping the operations team informed in real time.

### 4. Technical Implementation

#### End-to-end data flow

1. Auth (Cognito).

- The browser authenticates against Amazon Cognito (via Hosted UI or JS SDK).
- The resulting ID token (JWT) is held in memory; the SDK attaches `Authorization: Bearer <JWT>` to subsequent API requests.

2. Static web (CloudFront + S3).

- The SPA and its assets are hosted in S3; CloudFront sits in front with OAC, gzip/brotli compression, HTTP/2, and WAF managed rules.
- On load, the page initializes a small analytics SDK that collects behavioral events and sends them to API Gateway.

3. Event ingest (API Gateway).

- `POST /v1/events` (HTTP API). CORS is scoped to the site's origin; a JWT authorizer validates the Cognito token (or an API key is used for anonymous pre-login flows).
- Validated requests are proxied to Lambda.

4. Security & Ops.

- IAM least-privilege roles are applied to every component.
- CloudWatch captures logs, metrics, and alarms covering API 5xx rates, Lambda errors, throttling, and Shiny health.
- SNS delivers notifications on triggered alarms and growing DLQ depth.

5. Processing & storage (Lambda → S3 batch buffer → EventBridge → Lambda(ETL) → PostgreSQL on EC2 (data warehouse) → Shiny).

- The Ingest Lambda validates and enriches incoming events, then writes NDJSON objects to S3 partitioned by date and hour.
- EventBridge (cron) fires the ETL Lambda on a fixed schedule (e.g., every 60 minutes).
- The ETL Lambda reads new S3 partitions since the last watermark, deduplicates, transforms, and upserts rows into PostgreSQL on EC2 via VPC networking.
- The R Shiny Server on EC2 queries curated database tables and renders dashboards for admin users.

#### Data Contracts & Governance

**Event JSON (ingest)**

```yaml
{
  "event_id": "uuid-v4",
  "ts": "2025-10-18T12:34:56.789Z",
  "event_type": "view|click|search|add_to_cart|checkout|purchase",
  "session_id": "uuid-v4",
  "user_id": "cognito-sub-or-null",
  "anonymous_id": "stable-anon-id",
  "page_url": "https://site/p/123",
  "referrer": "https://google.com",
  "device": { "type": "mobile|desktop|tablet" },
  "geo": { "country": "VN", "city": null },
  "ecom":
    {
      "product_id": "sku-123",
      "category": "Shoes",
      "currency": "USD",
      "price": 79.99,
      "qty": 1,
    },
  "props": { "search_query": "running shoes" },
}
```

- **PII**: Names, email addresses, and phone numbers must never be included; any optional identifier is hashed inside Lambda before storage.
- **Behaviour**: `anonymous_id` is generated once per device; `session_id` is maintained in memory and rotates after 30 minutes of inactivity. Events are dispatched via `navigator.sendBeacon` with a `fetch` retry fallback; an optional IndexedDB buffer queues events when the device is offline.

**S3 raw layout & retention**

- Bucket: `s3://clickstream-raw/`
- Object format: NDJSON, optionally GZIP.
- Partitioning: `year=YYYY/month=MM/day=DD/hour=HH/ → events-<uuid>.ndjson.gz`
- Optional manifest per batch: processed watermark, object list, record counts, hash.
- Lifecycle: raw → (30 days Standard/IA) → (365+ days Glacier/Flex).
- Idempotency: a compact staging table in PostgreSQL (or a small S3 key-value manifest) tracks the last successfully processed object/batch and prevents duplicate loads.

#### Frontend SDK (Static site on S3 + CloudFront)

**Instrumentation**

- A minimal JS snippet is loaded site-wide (defer).
- Generates `anonymous_id` once and stores `session_id` in `localStorage`; the session resets after 30 minutes of user inactivity.
- Dispatches events via `navigator.sendBeacon`; falls back to `fetch` with retry and jitter.

**Auth context**

- If the user signs in via **Cognito**, `user_id = idToken.sub` is attached to enable logged-in conversion funnels.

**Offline durability**

- Optional Service Worker queue: when the device is offline, events are buffered in IndexedDB and flushed once connectivity is restored.

#### Ingestion API (API Gateway → Lambda)

**API Gateway (HTTP API)**

- Route: `POST /v1/events`.
- JWT authorizer (Cognito user pool). For anonymous pre-login events, an API key usage-plan with strict rate limits is used.
- WAF: AWS Managed Core + Bot Control; non-site origins are blocked via strict CORS.

**Lambda (Node.js or Python)**

- Validates payloads against JSON Schema (ajv/pydantic).
- Idempotency: recent `event_id` values are cached in memory (short TTL) plus batch-level deduplication occurs during ETL.
- Enrichment: derives date/hour partition keys, parses User-Agent, infers country from `CloudFront-Viewer-Country` when present.
- Persist: PutObject to S3 path `.../year=YYYY/month=MM/day=DD/hour=HH/....`
- Failure path: publishes to SQS DLQ; SNS alarm fires if DLQ depth exceeds 0.

#### Batch Buffer (S3)

- Purpose: low-cost, durable staging area for batch analytics workloads.
- Write pattern: small per-request objects or micro-batches (1–5 MB each) with GZIP. An optional compactor merges these into ≥64 MB files for more efficient downstream reads.
- Read pattern: ETL Lambda scans only partitions/objects created after the previous watermark.
- Schema-on-read: ETL enforces the schema at read time and handles late-arriving data by reprocessing a small sliding window (e.g., the last 2 hours) to correct session boundaries.

#### EC2 "data warehouse" node

Purpose: run ETL and host the curated analytical store that Shiny queries. Two choices:

- Postgres on EC2 (recommended if team prefers SQL/window functions)
  - Instance: t3.small/t4g.small; gp3 50–100GB.
  - Schema: `fact_events`, `fact_sessions`, `dim_date`, `dim_product`.
  - Security: within VPC private subnet; access via ALB/SSM Session Manager; automated daily snapshots to S3
- ETL (Lambda, batch via EventBridge cron):
  - Trigger: rate(5 minutes) / cron(...) depending on cost & freshness.
  - Steps: list new S3 objects → read → validate/dedupe → transform (flatten nested JSON, cast types, add ingest_date, session_window_start/end) → upsert into Postgres using COPY to temp tables + merge, hoặc batched INSERT ... ON CONFLICT.
  - Networking: Lambda attached to VPC private subnets to reach EC2 Postgres security group.

#### R Shiny Server on EC2 (admin analytics)

**Server**

- EC2 (t3.small/t4g.small) with: R 4.4+, Shiny Server (open-source), Nginx reverse proxy, TLS via ACM/ALB or Let's Encrypt.
- IAM instance profile (no static keys). Security group allows HTTPS from office/VPN or Cognito-gated admin site.

**App (packages)**

- `shiny`, `shinydashboard/bslib`, `plotly`, `DT`, `dplyr`, `DBI` + `RPostgres` or `duckdb`, `lubridate`.
- If querying DynamoDB directly for small cards, use `paws.dynamodb` (optional).

**Dashboards**

- **Traffic & Engagement**: DAU/MAU, sessions, avg pages, bounce proxy.
- **Funnels**: view→add_to_cart→checkout→purchase with stage conversion & drop-off.
- **Product Performance**: views, CTR, ATC rate, revenue by product/category.
- **Acquisition**: referrer, campaign, device, country.
- **Reliability**: Lambda error rate, DLQ depth, ETL lag, data freshness.

**Caching**

- Query results cached in-process (reactive values) or materialized by ETL; cache keys by date range and filters

#### Security baseline

**IAM**

- Ingest Lambda: s3:PutObject to raw bucket (scoped to prefix), s3:ListBucket on needed prefixes.
- ETL Lambda: s3:GetObject/ListBucket on raw prefixes; permission to fetch secrets from SSM Parameter Store; no broad S3 access.
- EC2 roles: read/write only to its own DB/volumes; optional read to S3 for backups.
- Shiny EC2: no write to S3 raw; read-only to Postgres as needed.

**Network**

- Place EC2 in private subnets; public access through ALB (HTTPS 443).
- Lambda for ETL joins the VPC to reach Postgres; SG rules least-priv (Postgres port from ETL SG only).
- No wide 0.0.0.0/0 to DB ports.

**Data**

- Encrypt EBS (KMS), S3 server-side encryption, RDS/PG TLS, secrets in SSM Parameter Store.
- No PII in events; retention: raw S3 90–365 days (lifecycle), curated Postgres per business policy

#### Observability & alerting

**CloudWatch metrics/alarms**

- API Gateway 5xx/latency, Lambda (ingest) errors/throttles, S3 PutObject failures, EventBridge schedule success rate, ETL duration/lag, DLQ depth, Shiny health check

**SNS topics**: on-call email/SMS/Slack webhook.

**Structured logs**: JSON logs from Lambda & ETL (request_id, event_type, status, ms, error_code).

**Watermark tracking**: custom metric "DW Freshness (minutes since last successful upsert)".

#### Cost Controls (stay near Free/low tier)

- Use HTTP API (cheaper), minimal Lambda memory (256–512MB), compress requests.
- Batch over realtime: S3 as buffer eliminates DynamoDB write/read costs.
- S3 lifecycle: Standard → Standard-IA/Intelligent-Tiering → Glacier for older raw; enable GZIP to cut storage/transfer.
- Tune ETL cadence (e.g., 15–60 min) and process only new objects; compact small files into bigger chunks to reduce read I/O.
- Single small EC2 for Shiny + DW at start; scale vertically or split later.
- AWS Budgets with SNS alerts (actual & forecast).

#### Deliverables

- Analytics SDK (TypeScript) with sessionization + beacon + optional offline queue.
- API/Lambda (ingest) with validation, enrichment, idempotency hints, DLQ.
- S3 raw bucket spec (prefixing/partitioning, compression, lifecycle) + optional compactor.
- ETL Lambda (batch) + EventBridge cron + watermarking + upsert strategy to PostgreSQL.
- PostgreSQL schema (fact_events, fact_sessions, dims) + indexes + vacuum/maintenance plan.
- R Shiny dashboard app (5 modules) + Nginx/ALB TLS setup.
- Runbook: alarms, on-call, backups, disaster recovery, freshness SLO, cost guardrails

### 5. Timeline & Milestones

### Project Timeline

#### Month 1 – Learning & Preparation

Explore a broad range of AWS services spanning compute, storage, analytics, and security.
Build foundational knowledge of cloud architecture patterns, data pipeline design, and serverless paradigms.
Hold regular team syncs to align on project objectives and divide responsibilities among members.

#### Month 2 – Architecture Design & Prototyping

Draft the end-to-end system architecture and map out the data flow between each component.
Provision initial AWS resources — including S3, Lambda, API Gateway, EventBridge, and EC2.
Evaluate open-source visualization and reporting tools for dashboard development.
Run sample code exercises to verify data ingestion and pipeline behavior end-to-end.

#### Month 3 – Implementation & Testing

Build out the complete architecture according to the finalized design.
Connect all AWS services and validate overall system reliability and correctness.
Execute functional and performance tests across all pipeline stages.
Wrap up documentation and prepare presentation materials for the project showcase.

### 6. Budget Estimation

You can find the budget estimation on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01).
Or you can download the [Budget Estimation File](../attachments/budget_estimation.pdf).

### Infrastructure Costs

- AWS Services

  - **Amazon Cognito(User Pools)**: 0.10 USD/monthly(1 Number of monthly active users (MAU), 1 Number of monthly active users (MAU) who sign in through SAML or OIDC federation)

  - **Amazon S3**

    - 3 Standard:0.17 USD/monthly(6 GB, 1,000 PUT requests, 1,000 GET requests, 6 GB Data returned, 6 GB Data scanned)
    - Data Transfer: 0.00 USD/monthly(Outbound: 6 TB, Inbound: 6 TB)

  - **Amazon CloudFront(United States)**: 0.64 USD/monthly(6 GB Data transfer out to internet, 6 GB Data transfer out to origin, 10,000 requests Number of requests (HTTPS))

  - **Amazon API Gateway(HTTP APIs)**: 0.01 USD/monthly(10,000 requests for HTTP API requests units)

  - **Amazon Lambda(Service settings)**: 0.00 USD/monthly(1,000,000 requests, 512 MB)

  - **Amazon CloudWatch(APIs)**: 0.03 USD/monthly(100 metrics GetMetricData, 1,000 metrics GetMetricWidgetImage, 1,000 requests API)

  - **Amazon SNS(Service settings)**: 0.02 USD/monthly(1,000,000 requests, 100,000 calls HTTP/HTTPS Notifications, 1,000 calls EMAIL/EMAIL-JSON Notifications, 100,000,000 notifications QS Notifications, 100,000,000 deliveries Amazon Web Services Lambda, 100,000 notifications Amazon Kinesis Data Firehose)

  - **Amazon EC2(EC2 specifications)**: 1.68 USD/monthly(1 instances, 730 Compute Savings Plans)

  - **Amazon EventBridge**: 0.00 USD/monthly(1,000,000 events(Number of AWS management events) EventBridge Event Bus - Ingestion)

Total: 2.65 USD/month, 31.8 USD/12 months

### 7. Risk Assessment

| **Risk**                                                                                                                   | **Likelihood** | **Impact** | **Mitigation Strategy**                                                                                                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------- | -------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| High costs exceeding the estimated budget                                                                                  | Medium         | High       | Continuously track and evaluate all potential AWS expenses. Avoid high-cost services where possible, and substitute with leaner, cost-effective alternatives that deliver equivalent functionality. |
| Potential issues during data transfer or service integration between AWS components                                        | Medium         | Medium     | Validate each integration step before going live. Begin testing early, rely on AWS managed services, and monitor performance continuously through Amazon CloudWatch.                              |
| Data collection or processing risks (e.g., excessive user interactions, network instability, missing or duplicated events) | High           | Medium     | Enforce data validation, buffering strategies, and schema rules to preserve consistency. Use structured logging and automated alarms to detect and address ingestion anomalies early.              |
| Low or no user adoption of the analytics dashboard                                                                         | Low            | High       | Run internal training sessions and use existing communication channels to promote the tool. Drive adoption by demonstrating concrete business value and highlighting actionable insights.    |

### 8. Expected Outcomes

#### Understanding Customer Behavior and Journey

The platform records every step of the customer journey — the pages they visit, the products they browse, how much time they spend on each page, and where they ultimately leave the site.

By analyzing session duration, bounce rate, and navigation paths, businesses gain a comprehensive picture of user engagement and the overall experience quality.

This data foundation enables targeted improvements to the website's interface, page structure, and overall customer satisfaction.

#### Identifying Popular Products and Consumer Trends

Leveraging clickstream data processed through the AWS pipeline, the system surfaces the products with the highest view and purchase rates.

Items that attract little attention are also tracked, allowing businesses to reconsider product listings, revisit pricing or imagery, and make smarter inventory decisions.

The system also helps uncover shopping trends across different time periods, geographic regions, and device types — supporting timely, data-informed business strategies.

#### Optimizing Marketing and Sales Strategies

Customer behavioral data is transformed into business insights and surfaced through R Shiny dashboards.

Armed with these analytical outputs, businesses can:

- Precisely define target customer segments for marketing initiatives
- Personalize advertising and promotional campaigns for specific product categories or audience groups
- Measure the effectiveness of marketing efforts through concrete engagement and conversion metrics

Ultimately, marketing and sales strategies become more precise and evidence-driven, leading to better decision-making and stronger business outcomes.


