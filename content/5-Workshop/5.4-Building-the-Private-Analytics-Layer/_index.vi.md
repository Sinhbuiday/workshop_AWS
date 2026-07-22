---
title: "Xây dựng lớp Private Analytics"
weight: 54
chapter: false
pre: " <b> 5.4. </b> "
---

## 5.4.1 Cấu hình VPC, Subnets và Route Tables

- **VPC CIDR**: `10.0.0.0/16`  
- **Public subnet**: `10.0.0.0/20` → `SBW_Project-subnet-public1-ap-southeast-1a`  
  - Chứa `SBW_EC2_WebDB` (EC2 OLTP).  
- **Private subnet**: `10.0.128.0/20` → `SBW_Project-subnet-private1-ap-southeast-1a`  
  - Chứa `SBW_EC2_ShinyDWH` và `SBW_Lamda_ETL`.  

**Public Route Table**

- `10.0.0.0/16 → local`  
- `0.0.0.0/0 → Internet Gateway`  

**Private Route Table**

- `10.0.0.0/16 → local`  
- S3 prefix list → Gateway VPC Endpoint cho S3  
- Không có route `0.0.0.0/0` tới IGW hoặc NAT Gateway  

---

## 5.4.2 VPC Endpoints (S3 & SSM)

![Các VPC Endpoint cho S3 & SSM](/images/aws-vpc-endpoints-s3-ssm.png)

### S3 Gateway VPC Endpoint

- Cho phép **truy cập S3 trong private network** cho:
  - `SBW_Lamda_ETL`   
- Loại bỏ nhu cầu sử dụng NAT Gateway.

### SSM Interface Endpoints

- `com.amazonaws.ap-southeast-1.ssm`  
- `com.amazonaws.ap-southeast-1.ssmmessages`  
- `com.amazonaws.ap-southeast-1.ec2messages`  

Các endpoint này cho phép **Session Manager** quản trị và port-forward tới `SBW_EC2_ShinyDWH` mà không cần địa chỉ IP public hay cổng SSH mở.

---

## 5.4.3 Data Warehouse trên EC2 – `SBW_EC2_ShinyDWH`

Trên EC2 private này:

- PostgreSQL database: `clickstream_dw`  
- Bảng chính: `clickstream_events` với các field:

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

Instance cho phép:

- `SBW_Lamda_ETL` kết nối tới PostgreSQL database `clickstream_dw`.
- Truy cập web Shiny từ local qua SSM port forwarding.

---

## 5.4.4 ETL Lambda – `SBW_Lamda_ETL` (chạy trong Private subnet)

ETL Lambda là nơi xử lý logic batch chính.

**Cấu hình VPC:**

- Subnet: `SBW_Project-subnet-private1-ap-southeast-1a`  
- Security group: `sg_Lambda_ETL`  

**Environment variables:**

- `DWH_HOST`, `DWH_PORT=5432`, `DWH_USER`, `DWH_PASSWORD`, `DWH_DATABASE=clickstream_dw`  
- `RAW_BUCKET=clickstream-s3-ingest`  
- `AWS_REGION=ap-southeast-1`  

**Nhiệm vụ:**

1. Xác định tập hợp file trong `s3://clickstream-s3-ingest/events/YYYY/MM/DD/` cần xử lý trong batch hiện tại.  
2. Với mỗi file JSON:
   - Extract
   - Transform
   - Load

**IAM role:**

- Cấp cho Lambda các quyền cần thiết để truy cập `EC2_ShinyDWH` (ví dụ qua database endpoint trong private subnet).

---

## 5.4.5 Lên lịch bằng EventBridge – `SBW_ETL_HOURLY_RULE`

![EventBridge rule](/images/aws-eventbridge-sbw-etl-hourly-rule.png)

EventBridge duy trì nền tảng hoạt động theo nhịp **batch**:

- **Tên rule**: `SBW_ETL_HOURLY_RULE`  
- **Schedule**: `rate(1 hour)`  
- **Target**: `SBW_Lamda_ETL`  

Mỗi lần rule kích hoạt:

1. ETL Lambda khởi chạy bên trong private subnet.  
2. Đọc các events mới từ S3 qua Gateway Endpoint.  
3. Nạp dữ liệu đã xử lý vào `clickstream_dw`.  

Bạn cũng có thể **trigger ETL Lambda thủ công** (từ Lambda console) cho mục đích backfill hoặc kiểm thử.

---

## 5.4.6 Tóm tắt Security Groups & Connectivity

- `sg_Lambda_ETL`:
  - Outbound tới S3 endpoint và `sg_analytics_ShinyDWH:5432`.  

- `sg_analytics_ShinyDWH`:
  - Inbound `5432/tcp` từ `sg_Lambda_ETL`.  
  - Inbound `3838/tcp` cho Shiny (chỉ truy cập qua SSM port forwarding).  
