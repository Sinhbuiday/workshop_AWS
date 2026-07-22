---
title: "Bản đề xuất"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Team Super Beast Warrior(SBW)

# Batch-based Clickstream Analytics Platform

### 1. Tóm tắt

Mục tiêu của dự án là **xây dựng và vận hành một Batch-based Clickstream Analytics Platform** phục vụ website thương mại điện tử chuyên về **máy tính và phụ kiện**. Giao diện frontend được tích hợp một **JavaScript SDK** gọn nhẹ có nhiệm vụ thu thập các sự kiện người dùng như **clicks**, **views**, **searches** và đẩy chúng lên **backend API**. Toàn bộ nền tảng được xây dựng trên **AWS Cloud Services**.

Dữ liệu tương tác thô — bao gồm sự kiện duyệt web, truy vấn tìm kiếm và lượt truy cập trang — từ website được đẩy vào **Amazon S3** dưới dạng file log. **Amazon EventBridge** chạy theo lịch mỗi giờ để kích hoạt **AWS Lambda**, thực hiện xử lý và chuyển đổi dữ liệu trước khi tải vào **data warehouse** trên **Amazon EC2**.

Dữ liệu sau xử lý được hiển thị qua **R Shiny dashboards**, cung cấp cho người vận hành cửa hàng cái nhìn tổng quan về hành vi khách hàng, xu hướng sản phẩm và mức độ tương tác trên website.

Kiến trúc tập trung vào **batch analytics**, **ETL pipeline** đầu cuối và **business intelligence**, đồng thời đảm bảo **bảo mật (security)**, **khả năng mở rộng (scalability)** và **hiệu quả chi phí (cost efficiency)** thông qua **AWS managed services**.

### 2. Vấn đề đặt ra

#### Vấn đề hiện tại là gì?

Các website thương mại điện tử không ngừng tạo ra lượng lớn **clickstream data** — bao gồm product views, cart interactions, và search activity — chứa đựng giá trị kinh doanh tiềm năng rất cao.

Tuy vậy, các doanh nghiệp **nhỏ và vừa** thường thiếu cả công cụ lẫn năng lực kỹ thuật để thu thập, xử lý và khai thác dữ liệu này một cách có hệ thống.

Hệ quả là, họ gặp khó khăn trong việc:

- Nắm bắt cách khách hàng điều hướng và ra quyết định mua hàng
- Xác định sản phẩm nào đang tạo ra doanh thu cao nhất
- Tinh chỉnh marketing campaigns và cải thiện hiệu suất website
- Đưa ra các quyết định dựa trên bằng chứng liên quan đến tồn kho và định giá

#### Giải pháp

Dự án này đề xuất một **hệ thống batch clickstream analytics chạy trên AWS**, tự động thu thập dữ liệu tương tác của người dùng theo chu kỳ mỗi giờ, đưa qua các serverless processing functions và lưu kết quả có cấu trúc vào data warehouse tập trung trên **Amazon EC2**.

Kết quả phân tích được trình bày qua **R Shiny dashboards** tương tác, giúp chủ doanh nghiệp nhìn thấy rõ hành vi khách hàng và các điểm cần cải thiện.

#### Lợi ích và hoàn vốn đầu tư

- **Data-driven decision making**: Khám phá sở thích khách hàng, sản phẩm đang hot và xu hướng mua sắm theo mùa.
- **Scalable and modular design**: Kiến trúc có thể mở rộng để đáp ứng lưu lượng tăng hoặc tích hợp thêm nguồn dữ liệu mà không cần thiết kế lại từ đầu.
- **Cost-efficient batch processing**: Chạy theo lịch định kỳ giúp giảm thiểu chi phí tính toán liên tục so với các giải pháp streaming thời gian thực.
- **Business insight enablement**: Chủ cửa hàng có được công cụ phân tích để tinh chỉnh chiến lược bán hàng và tăng doanh thu dựa trên dữ liệu thực.

### 3. Kiến trúc giải pháp

![Architecture](/images/2-Proposal/AWS_Architecture_2.jpg)

#### Dịch vụ AWS sử dụng

- **Amazon Cognito**: Quản lý xác thực và kiểm soát quyền truy cập cho quản trị viên và khách hàng của website, đảm bảo chỉ người dùng được cấp phép mới tương tác được với nền tảng.
- **Amazon S3**: Đóng vai trò lớp lưu trữ trung tâm — chứa các file static frontend và file log clickstream thô từ phiên người dùng. Đồng thời là khu vực staging cho batch files chờ xử lý và chuyển vào data warehouse.
- **Amazon CloudFront**: Phân phối nội dung website tĩnh trên toàn thế giới với độ trễ thấp, tăng tốc độ tải trang và đưa tài nguyên cache gần hơn với người dùng cuối.
- **Amazon API Gateway**: Là cổng tiếp nhận chính cho các API request từ website, cung cấp kênh bảo mật để đẩy dữ liệu clickstream và browsing vào hệ sinh thái AWS.
- **AWS Lambda**: Thực thi code serverless để tiền xử lý và chuẩn hóa clickstream data ghi vào S3. Đồng thời chạy các transformation job theo lịch do EventBridge kích hoạt, chuẩn bị dữ liệu trước khi nạp vào data warehouse.
- **Amazon EventBridge**: Điều phối và lên lịch cho batch workflows — gọi Lambda functions mỗi giờ để di chuyển và chuyển đổi clickstream data từ S3 sang EC2 data warehouse.
- **Amazon EC2 (Data Warehouse)**: Chạy môi trường data warehouse với PostgreSQL hoặc hệ quản trị cơ sở dữ liệu tương thích, phục vụ batch analytics, phân tích xu hướng lịch sử và báo cáo kinh doanh. Các EC2 instance được đặt trong VPC private subnet để cô lập mạng.
- **R Shiny (on EC2)**: Cung cấp nền tảng dashboard tương tác hiển thị kết quả phân tích batch, giúp đội ngũ kinh doanh khám phá hành trình khách hàng, sản phẩm nổi bật và cơ hội doanh thu.
- **AWS IAM**: Thiết lập ranh giới phân quyền và kiểm soát truy cập trên toàn bộ thành phần AWS, đảm bảo mỗi service và người dùng chỉ có đúng quyền cần thiết.
- **Amazon CloudWatch**: Theo dõi metrics, tổng hợp logs và ghi nhận kết quả scheduled job từ Lambda và EC2, cung cấp cái nhìn thống nhất về sức khỏe và hiệu suất hệ thống.
- **Amazon SNS**: Phát thông báo và cảnh báo khi batch jobs hoàn thành, thất bại hoặc gặp lỗi bất ngờ, giữ cho đội vận hành luôn được cập nhật kịp thời.

### 4. Triển khai

#### End-to-end data flow

1. Auth (Cognito)

- Trình duyệt xác thực với Amazon Cognito (qua Hosted UI hoặc JS SDK).
- ID token (JWT) thu được được giữ trong bộ nhớ; SDK tự động gắn `Authorization: Bearer <JWT>` vào các API request tiếp theo.

2. Static web (CloudFront + S3)

- SPA và các assets được lưu trên S3; CloudFront đứng trước với OAC, nén gzip/brotli, HTTP/2 và WAF managed rules.
- Khi tải trang, một analytics SDK nhỏ được khởi tạo, thu thập sự kiện hành vi và gửi lên API Gateway.

3. Event ingest (API Gateway)

- `POST /v1/events` (HTTP API). CORS giới hạn theo origin của site; JWT authorizer xác thực Cognito token (hoặc dùng API key cho luồng anonymous trước khi đăng nhập).
- Các request hợp lệ được proxy sang Lambda.

4. Security & Ops

- IAM least-privilege role được áp dụng cho mọi thành phần.
- CloudWatch ghi log, metrics và alarm cho API 5xx, Lambda errors, throttling và Shiny health.
- SNS gửi thông báo khi alarm kích hoạt hoặc DLQ tăng đột biến.

5. Processing & storage (Lambda → S3 batch buffer → EventBridge → Lambda (ETL) → PostgreSQL trên EC2 (data warehouse) → Shiny)

- Ingest Lambda xác thực và enrich events đầu vào, sau đó ghi NDJSON objects vào S3 theo phân vùng date và hour.
- EventBridge (cron) kích hoạt ETL Lambda theo lịch cố định (ví dụ: mỗi 60 phút).
- ETL Lambda đọc các S3 partition mới kể từ watermark lần trước, loại bỏ trùng lặp, transform và upsert rows vào PostgreSQL trên EC2 qua mạng VPC.
- R Shiny Server trên EC2 truy vấn các bảng dữ liệu được tổng hợp sẵn và render dashboards cho admin.

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

- **PII**: Tuyệt đối không gửi name/email/phone; mọi optional identifier nếu có sẽ được hash bên trong Lambda trước khi lưu trữ.
- **Behaviour**: `anonymous_id` được tạo một lần duy nhất trên mỗi thiết bị; `session_id` được duy trì trong bộ nhớ và tự động đặt lại sau 30 phút không hoạt động. Events được gửi qua `navigator.sendBeacon` với fallback là `fetch` retry; một offline buffer tùy chọn qua IndexedDB giữ events khi thiết bị mất mạng.

**S3 raw layout & retention**

- **Bucket**: `s3://clickstream-raw/`
- **Object format**: NDJSON, tùy chọn GZIP.
- **Partitioning**: `year=YYYY/month=MM/day=DD/hour=HH/ → events-<uuid>.ndjson.gz`
- **Optional manifest per batch**: bao gồm processed watermark, object list, record counts, và hash.
- **Lifecycle**: raw → (30 days Standard/IA) → (365+ days Glacier/Flex).
- **Idempotency**: một compact staging table trong PostgreSQL (hoặc S3 key-value manifest nhỏ) theo dõi object/batch cuối cùng đã xử lý thành công, ngăn chặn việc nạp trùng dữ liệu.

#### Frontend SDK (Static site on S3 + CloudFront)

**Instrumentation**

- Một JS snippet tối giản được load trên toàn site (defer).
- Tạo `anonymous_id` một lần và lưu `session_id` trong `localStorage`; session reset sau 30 phút không có hoạt động.
- Gửi events qua `navigator.sendBeacon`; fallback sang `fetch` với retry và jitter.

**Auth context**

- Nếu người dùng đăng nhập qua **Cognito**, `user_id = idToken.sub` được đính kèm để theo dõi conversion funnel cho người dùng đã xác thực.

**Offline durability**

- Service Worker queue tùy chọn: khi mất kết nối, events được buffer vào IndexedDB và flush ngay khi có mạng trở lại.

#### Ingestion API (API Gateway → Lambda)

**API Gateway (HTTP API)**

- Route: `POST /v1/events`.
- JWT authorizer (Cognito user pool). Với anonymous pre-login events, sử dụng API key usage-plan với rate limit chặt chẽ.
- WAF: AWS Managed Core + Bot Control; chặn non-site origins bằng strict CORS.

**Lambda (Node.js or Python)**

- Validate payload theo JSON Schema (ajv/pydantic).
- Idempotency: cache `event_id` gần đây trong bộ nhớ (TTL ngắn) cộng thêm deduplication ở cấp batch trong ETL.
- Enrichment: suy ra partition key date/hour, phân tích User-Agent, lấy country từ `CloudFront-Viewer-Country` khi có.
- Persist: PutObject vào đường dẫn S3 `.../year=YYYY/month=MM/day=DD/hour=HH/....`
- Failure path: publish lên SQS DLQ; SNS alarm kích hoạt nếu DLQ depth vượt ngưỡng 0.

#### Batch Buffer (S3)

- Mục đích: vùng staging bền vững, chi phí thấp phục vụ batch analytics.
- Write pattern: object nhỏ cho mỗi request hoặc micro-batches (1–5 MB) với GZIP. Compactor tùy chọn gộp các file nhỏ thành file ≥64 MB để đọc hiệu quả hơn ở downstream.
- Read pattern: ETL Lambda chỉ scan các partition/object được tạo sau watermark lần trước.
- Schema-on-read: ETL áp dụng schema tại thời điểm đọc và xử lý dữ liệu đến muộn bằng cách reprocess một sliding window nhỏ (ví dụ: 2 giờ gần nhất) để hiệu chỉnh ranh giới session.

#### EC2 "data warehouse" node

Mục đích: chạy ETL và lưu trữ analytical store được Shiny truy vấn. Hai lựa chọn:

- Postgres trên EC2 (khuyến nghị nếu nhóm quen SQL/window functions)

  - Instance: t3.small/t4g.small; gp3 50–100GB.
  - Schema: `fact_events`, `fact_sessions`, `dim_date`, `dim_product`.
  - Security: trong private subnet của VPC; truy cập qua ALB/SSM Session Manager; snapshot tự động hàng ngày lên S3.

- ETL (Lambda, batch qua EventBridge cron):
  - Trigger: rate(5 minutes) / cron(...) tùy theo cost & freshness.
  - Các bước: liệt kê S3 objects mới → đọc → validate/dedupe → transform (flatten JSON, cast types, thêm ingest_date, session_window_start/end) → upsert vào Postgres bằng COPY tới bảng tạm + merge, hoặc batched INSERT ... ON CONFLICT.
  - Networking: Lambda kết nối vào private subnets của VPC để truy cập Postgres security group trên EC2.

#### R Shiny Server on EC2 (admin analytics)

**Server**

- EC2 (t3.small/t4g.small) với: R 4.4+, Shiny Server (open-source), Nginx reverse proxy, TLS qua ACM/ALB hoặc Let's Encrypt.
- IAM instance profile (không dùng static keys). Security group chỉ cho phép HTTPS từ office/VPN hoặc Cognito-gated admin site.

**App (packages)**

- Các package R: `shiny`, `shinydashboard/bslib`, `plotly`, `DT`, `dplyr`, `DBI` + `RPostgres` hoặc `duckdb`, `lubridate`.
- Nếu truy vấn DynamoDB trực tiếp cho small cards, có thể dùng `paws.dynamodb` (tùy chọn).

**Dashboards**

- **Traffic & Engagement**: DAU/MAU, sessions, avg pages, bounce proxy.
- **Funnels**: view → add_to_cart → checkout → purchase với tỷ lệ chuyển đổi từng stage và drop-off.
- **Product Performance**: views, CTR, ATC rate, revenue theo product/category.
- **Acquisition**: referrer, campaign, device, country.
- **Reliability**: Lambda error rate, DLQ depth, ETL lag, data freshness.

**Caching**

- Kết quả truy vấn được cache trong process (reactive values) hoặc materialized bởi ETL; cache keys dựa trên date range và filters.

#### Security baseline

**IAM**

- Ingest Lambda: quyền s3:PutObject tới raw bucket (giới hạn theo prefix), s3:ListBucket trên các prefix cần thiết.
- ETL Lambda: quyền s3:GetObject/ListBucket trên raw prefixes; được phép lấy secrets từ SSM Parameter Store; không có quyền S3 rộng.
- EC2 roles: chỉ đọc/ghi vào DB/volumes của chính nó; có thể đọc từ S3 để backup.
- Shiny EC2: không ghi vào S3 raw; chỉ read-only tới Postgres khi cần.

**Network**

- Đặt EC2 trong private subnets; truy cập công khai qua ALB (HTTPS 443).
- Lambda thực hiện ETL được join vào VPC để kết nối tới Postgres; Security Group áp dụng least-privilege (chỉ mở Postgres port từ ETL SG).
- Không mở 0.0.0.0/0 tới các DB ports.

**Data**

- Mã hóa: EBS bằng KMS, S3 server-side encryption, RDS/PG TLS, secrets lưu trong SSM Parameter Store.
- Không có PII trong events; retention: raw S3 90–365 ngày (lifecycle), curated Postgres theo chính sách kinh doanh.

#### Observability & alerting

**CloudWatch metrics/alarms**

- API Gateway 5xx/latency, Lambda (ingest) errors/throttles, S3 PutObject failures, EventBridge schedule success rate, ETL duration/lag, DLQ depth, Shiny health check.

**SNS topics**: gửi thông báo on-call qua email/SMS/Slack webhook.
**Structured logs**: JSON logs từ Lambda & ETL (bao gồm request_id, event_type, status, ms, error_code).
**Watermark tracking**: custom metric "DW Freshness (minutes since last successful upsert)".

#### Cost Controls (stay near Free/low tier)

- Ưu tiên HTTP API (rẻ hơn), giới hạn Lambda memory ở mức tối thiểu (256–512MB), nén các request.
- Batch thay vì realtime: dùng S3 làm buffer để loại bỏ chi phí đọc/ghi DynamoDB.
- S3 lifecycle: Standard → Standard-IA/Intelligent-Tiering → Glacier cho raw data cũ; bật GZIP để giảm chi phí lưu trữ và truyền tải.
- Điều chỉnh cadence ETL (15–60 phút) và chỉ xử lý các object mới; gộp file nhỏ thành file lớn để giảm read I/O.
- Bắt đầu với single small EC2 chạy cả Shiny lẫn DW; scale vertically hoặc tách ra khi cần thiết.
- Cấu hình AWS Budgets kèm SNS alerts cho chi phí thực tế và dự báo.

#### Deliverables

- Analytics SDK (TypeScript): hỗ trợ sessionization, beacon, và optional offline queue.
- API/Lambda (ingest): xử lý validation, enrichment, idempotency hints, và DLQ.
- S3 raw bucket spec: prefixing/partitioning, compression, lifecycle, kèm optional compactor.
- ETL Lambda (batch): kết hợp với EventBridge cron, watermarking, và upsert strategy vào PostgreSQL.
- PostgreSQL schema: fact_events, fact_sessions, các dims, cùng indexes và vacuum/maintenance plan.
- R Shiny dashboard app: gồm 5 modules, triển khai với Nginx/ALB TLS setup.
- Runbook: bao gồm alarms, on-call, backups, disaster recovery, freshness SLO, và cost guardrails.

### 5. Kế hoạch triển khai

### Dự án theo tiến độ

#### Tháng 1 – Học tập & Chuẩn bị

Tìm hiểu đa dạng các dịch vụ AWS bao gồm compute, storage, analytics và security.
Xây dựng kiến thức nền tảng về cloud architecture, thiết kế data pipeline và mô hình serverless.
Tổ chức các buổi sync định kỳ để thống nhất mục tiêu dự án và phân công công việc trong nhóm.

#### Tháng 2 – Thiết kế kiến trúc & Prototyping

Phác thảo kiến trúc hệ thống tổng thể và xác định luồng dữ liệu giữa các thành phần.
Khởi tạo các tài nguyên AWS ban đầu: S3, Lambda, API Gateway, EventBridge và EC2.
Đánh giá các công cụ mã nguồn mở phục vụ visualization và reporting cho dashboard.
Thực thi code mẫu để kiểm chứng hoạt động của data ingestion và processing pipeline đầu cuối.

#### Tháng 3 – Triển khai & Kiểm thử

Xây dựng toàn bộ kiến trúc theo bản thiết kế đã phê duyệt.
Kết nối tất cả dịch vụ AWS và xác nhận độ tin cậy, tính chính xác của hệ thống.
Thực hiện kiểm thử chức năng và hiệu năng trên toàn bộ các bước pipeline.
Hoàn thiện tài liệu và chuẩn bị tài liệu thuyết trình cho buổi nghiệm thu dự án.

### 6. Ước tính chi phí

Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).

### Chi phí hạ tầng

- AWS Services

  - **Amazon Cognito (User Pools)**: 0.10 USD/tháng(1 monthly active user (MAU), 1 MAU đăng nhập qua SAML hoặc OIDC federation)

  - **Amazon S3**

    - S3 Standard: 0.17 USD/tháng (6 GB, 1,000 PUT requests, 1,000 GET requests, 6 GB Data returned, 6 GB Data scanned)
    - Data Transfer: 0.00 USD/tháng (Outbound 6 TB, Inbound 6 TB)

  - **Amazon CloudFront (United States)**: 0.64 USD/tháng(6 GB Data transfer out to internet, 6 GB Data transfer out to origin, 10,000 HTTPS requests)

  - **Amazon API Gateway (HTTP APIs)**: 0.01 USD/tháng(10,000 HTTP API requests units)

  - **Amazon Lambda (Service settings)**: 0.00 USD/tháng(1,000,000 requests, 512 MB memory)

  - **Amazon CloudWatch (APIs)**: 0.03 USD/tháng(100 metrics GetMetricData, 1,000 metrics GetMetricWidgetImage, 1,000 API requests)

  - **Amazon SNS (Service settings)**: 0.02 USD/tháng(1,000,000 requests, 100,000 HTTP/HTTPS Notifications, 1,000 EMAIL/EMAIL-JSON Notifications, 100,000,000 QS Notifications, 100,000,000 Lambda deliveries, 100,000 Kinesis Data Firehose notifications)

  - **Amazon EC2 (EC2 specifications)**: 1.68 USD/tháng(1 instance, 730 Compute Savings Plans)

  - **Amazon EventBridge**: 0.00 USD/tháng(1,000,000 events (AWS management events - EventBridge Event Bus Ingestion))

Tổng cộng: 2.65 USD/tháng, 31.8 USD/12 tháng

### 7. Đánh giá rủi ro

| Risk                                                                                                                             | Likelihood | Impact | Mitigation Strategy                                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Chi phí cao vượt quá ngân sách ước tính                                                                                          | Medium     | High   | Liên tục theo dõi và đánh giá toàn bộ chi phí AWS tiềm năng. Hạn chế dùng các dịch vụ tốn kém và thay thế bằng giải pháp tinh gọn, tiết kiệm hơn mà vẫn đáp ứng chức năng tương đương. |
| Các vấn đề tiềm ẩn trong việc truyền dữ liệu hoặc tích hợp dịch vụ giữa các thành phần AWS                                       | Medium     | Medium | Xác thực từng bước tích hợp trước khi đưa vào vận hành. Bắt đầu kiểm thử sớm, ưu tiên dùng AWS managed services và giám sát hiệu suất liên tục qua Amazon CloudWatch.                                         |
| Rủi ro trong thu thập hoặc xử lý dữ liệu (ví dụ: tương tác người dùng quá mức, mạng không ổn định, thiếu hoặc trùng lặp sự kiện) | High       | Medium | Áp dụng data validation, chiến lược buffering và schema enforcement để đảm bảo tính nhất quán. Dùng structured logging và automated alarm để phát hiện và xử lý sớm các bất thường khi ingest.                                        |
| Người dùng ít hoặc không sử dụng analytics dashboard                                                                             | Low        | High   | Tổ chức các buổi đào tạo nội bộ và tận dụng kênh truyền thông nội bộ để giới thiệu công cụ. Thúc đẩy việc sử dụng bằng cách trình bày giá trị kinh doanh cụ thể và các actionable insights mà hệ thống mang lại.       |

### 8. Kết quả kỳ vọng

#### Hiểu Hành Vi và Hành Trình Khách Hàng

Nền tảng ghi lại từng bước trong hành trình khách hàng — các trang họ ghé thăm, sản phẩm họ xem, thời gian dừng lại trên từng trang, và điểm họ rời bỏ website.

Qua việc phân tích session duration, bounce rate và navigation paths, doanh nghiệp có được bức tranh toàn diện về mức độ gắn kết của người dùng và chất lượng trải nghiệm tổng thể.

Nền tảng dữ liệu này cho phép thực hiện các cải tiến có chủ đích về giao diện website, cấu trúc trang và mức độ hài lòng của khách hàng.

#### Xác Định Sản Phẩm Phổ Biến và Xu Hướng Người Tiêu Dùng

Dựa trên clickstream data được xử lý qua pipeline AWS, hệ thống làm nổi bật các sản phẩm có tỷ lệ xem và mua cao nhất.

Các mặt hàng ít được quan tâm cũng được theo dõi, giúp doanh nghiệp xem xét lại product listings, điều chỉnh giá hay hình ảnh sản phẩm, và lên kế hoạch tồn kho thông minh hơn.

Ngoài ra hệ thống còn giúp phát hiện xu hướng mua sắm theo khoảng thời gian, khu vực địa lý hay loại thiết bị — hỗ trợ các quyết định kinh doanh kịp thời, dựa trên dữ liệu thực.

#### Tối Ưu Chiến Lược Marketing và Bán Hàng

Dữ liệu hành vi khách hàng được chuyển đổi thành business insights và trình bày qua R Shiny dashboards.

Với kết quả phân tích này, doanh nghiệp có thể:

- Xác định chính xác target customer segments cho các chiến dịch marketing
- Cá nhân hóa quảng cáo và khuyến mãi theo từng nhóm sản phẩm hoặc đối tượng khách hàng cụ thể
- Đo lường hiệu quả các sáng kiến marketing bằng các chỉ số tương tác và chuyển đổi cụ thể

Kết quả là, marketing và sales strategies trở nên chính xác và dựa trên bằng chứng hơn, hỗ trợ ra quyết định tốt hơn và cải thiện hiệu suất kinh doanh bền vững.


