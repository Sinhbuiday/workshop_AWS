---
title: "Workshop"
weight: 5
chapter: false
pre: "<b>5. </b>"
---

# Batch-Based Clickstream Analytics Platform

![Architecture](/images/architecture.png)

<p align="center"><em>Figure: Architecture Batch-base Clickstream Analytics Platform.</em></p>

#### Tổng quan

Workshop này hướng dẫn từng bước triển khai một **nền tảng phân tích Clickstream theo kiểu batch (Batch-Based Clickstream Analytics Platform)** dành cho website thương mại điện tử bán sản phẩm máy tính.

Hệ thống thu thập các sự kiện clickstream phát sinh từ frontend, lưu trữ JSON thô trong **Amazon S3**, thực thi ETL định kỳ (**AWS Lambda + EventBridge**), rồi tải dữ liệu đã chuyển đổi vào **PostgreSQL Data Warehouse trên EC2** đặt trong private subnet.

Dashboard phân tích được vận hành bởi **R Shiny**, triển khai trên cùng EC2 với Data Warehouse, và truy cập an toàn qua **AWS Systems Manager Session Manager**.

Nền tảng được thiết kế theo các nguyên tắc:

- Ranh giới rõ ràng giữa workload **OLTP và Analytics**
- Backend analytics hoàn toàn chạy trong private (**không có truy cập public vào DW**)
- Các thành phần serverless của AWS để tối đa hóa hiệu quả chi phí và khả năng mở rộng
- Không SSH trực tiếp: toàn bộ quản trị thực hiện qua **SSM Session Manager** vào EC2 chạy DW / Shiny
- Shiny app có thể truy cập từ máy local tại **localhost:3838**

#### Các thành phần kiến trúc chính

**Miền Frontend & OLTP**

- Ứng dụng Next.js: **`ClickSteam.NextJS`** được host bằng **AWS Amplify Hosting**
- **Amazon CloudFront** được tích hợp trong **Amplify** giúp tăng tốc độ truyền tải file tĩnh
- **Amazon Cognito** User Pool để xác thực người dùng
- PostgreSQL OLTP chạy trên EC2: **`SBW_EC2_WebDB`** (public subnet)
  - Database: `clickstream_web` (schema `public`)
  - Port: `5432`

**Miền Ingestion & Data Lake**

- **Amazon API Gateway (HTTP API)**: `clickstream-http-api`
  - Route: `POST /clickstream`
- **Lambda Ingest**: `clickstream-lambda-ingest`
  - Validate payload, enrich metadata, ghi file JSON vào S3
- **S3 Raw Clickstream Bucket**: `clickstream-s3-ingest`
  - Prefix: `events/YYYY/MM/DD/`
  - Mẫu tên file: `event-<uuid>.json`
  - `RAW_BUCKET = clickstream-s3-ingest`

**Miền Analytics & Data Warehouse**

- **EC2 private cho DWH + Shiny**: `SBW_EC2_ShinyDWH` (private subnet `10.0.128.0/20`)

  - Database DW: `clickstream_dw`
  - Bảng chính: `clickstream_events` với các field:
    - `event_id, event_timestamp, event_name`
    - `user_id, user_login_state, identity_source, client_id, session_id, is_first_visit`
    - `context_product_id, context_product_name, context_product_category, context_product_brand`
    - `context_product_price, context_product_discount_price, context_product_url_path`
  - R Shiny Server chạy trên port `3838`, web path `/sbw_dashboard`

- **Lambda ETL**: `SBW_Lamda_ETL` (chạy trong VPC)

  - Đọc JSON thô từ `clickstream-s3-ingest`
  - Transform thành các dòng dữ liệu dạng SQL-ready
  - Insert vào bảng `clickstream_dw.public.clickstream_events`

- **EventBridge Rule**: `SBW_ETL_HOURLY_RULE`

  - Lịch chạy: `rate(1 hour)`

- **VPC & Networking**

  - VPC CIDR: `10.0.0.0/16`
  - Public subnet: `10.0.0.0/20` → `SBW_Project-subnet-public1-ap-southeast-1a` (EC2 OLTP)
  - Private subnet: `10.0.128.0/20` → `SBW_Project-subnet-private1-ap-southeast-1a` (DW, Shiny, ETL Lambda)
  - **S3 Gateway VPC Endpoint** để truy cập S3 trong private network
  - **SSM Interface Endpoints** (SSM, SSMMessages, EC2Messages) cho Session Manager

- **Truy cập quản trị (SSM)**
  - Port forwarding:
    - `localPort = 3838`
    - `portNumber = 3838`
  - Shiny URL khi truy cập từ local: `http://localhost:3838/sbw_dashboard`

#### Bản đồ nội dung

1. **[5.1. Mục tiêu & Phạm vi](5.1-objectives--scope/)**
2. **[5.2. Giới thiệu kiến trúc](5.2-architecture-walkthrough/)**
3. **[5.3. Triển khai hệ thống thu thập Clickstream](5.3-implementing-clickstream-ingestion/)**
4. **[5.4. Xây dựng lớp phân tích dữ liệu private](5.4-building-private-analytics-layer/)**
5. **[5.5. Trực quan hóa phân tích với Shiny Dashboards](5.5-visualizing-analytics-with-shiny-dashboards/)**
6. **[5.6. Tổng kết & Dọn dẹp](5.6-summary-cleanup/)**
