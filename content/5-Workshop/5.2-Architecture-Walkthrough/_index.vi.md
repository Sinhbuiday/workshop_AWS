---
title: "Architecture Walkthrough"
weight: 52
chapter: false
pre: " <b> 5.2. </b> "
---

![Architecture](/images/architecture.png)
<p align="center"><em>Hình: Batch-based Clickstream Analytics Platform.</em></p>

---

## 5.2.1 Miền User & Frontend

### (1) User Browser – Người dùng

**Nhiệm vụ**

- Truy cập website: duyệt catalogue, xem trang chi tiết sản phẩm, quản lý giỏ hàng, hoàn tất thanh toán, v.v.
- Tạo tài khoản hoặc đăng nhập vào hệ thống.
- Phát sinh **clickstream events** như:
  - `page_view`, `product_view`, `add_to_cart`, `purchase`, …

**Vai trò trong pipeline**

- Là **nguồn phát sinh mọi tín hiệu hành vi** mà hệ thống analytics xử lý.
- Lớp JavaScript nhúng trong frontend gom các events này và chuyển chúng về backend thông qua API (6).

---

### (2) Amazon CloudFront (CDN / Edge)

**Nhiệm vụ**

- Hoạt động như **lớp CDN** đảm nhận phân phối nội dung website:
  - Cache HTML/CSS/JS/ảnh để giảm latency.
  - Giảm tải áp lực từ origin (Amplify, S3 assets).
- Đóng vai trò **điểm vào thống nhất** cho toàn bộ traffic web:
  - Định tuyến và chuyển tiếp request đến đúng origin:
    - Nội dung web động → Amplify.
    - Ảnh / media tĩnh → S3 (3).

**Tại sao quan trọng**

- Website thương mại điện tử đòi hỏi **tốc độ tải trang cao**.
- CloudFront cải thiện trải nghiệm người dùng ở nhiều vùng địa lý bằng cách phục vụ nội dung từ các edge location gần nhất.

---

### (3) Amazon S3 – Media Assets

**Nhiệm vụ**

- Lưu trữ **ảnh sản phẩm** và các **assets tĩnh** khác của website.
- Đóng vai trò **origin** mà CloudFront (2) lấy ảnh và file tĩnh để phục vụ.

**Lưu ý**

- Bucket này (ví dụ `clickstream-s3-sbw`) được **tách biệt** khỏi RAW clickstream bucket (8) nhằm:
  - Duy trì ranh giới phân quyền rõ ràng.
  - Đơn giản hóa việc quản lý và dọn dẹp tài nguyên.

---

### (4) Amazon Amplify – Front-end Hosting

**Nhiệm vụ**

- Build và deploy ứng dụng **Next.js** (ví dụ `ClickSteam.NextJS`).
- Host frontend (SSR/ISR, API routes).
- Quản lý toàn bộ pipeline deploy: build, lưu artifact và ánh xạ domain.

**Liên kết với dịch vụ khác**

- Kết nối với **Cognito (5)** để xử lý luồng xác thực người dùng.
- Đẩy **clickstream events** từ frontend tới **API Gateway (6)**.
- Giao tiếp với **EC2 OLTP (20)** qua Prisma để đọc ghi dữ liệu giao dịch.

---

### (5) Amazon Cognito – Authentication / Authorization

**Nhiệm vụ**

- Quản lý **user identity**:
  - Luồng đăng ký / đăng nhập / đặt lại mật khẩu.
  - Duy trì user pool và cấp phát JWT tokens (ID token, access token).
- Cung cấp token cho frontend để gọi các API backend với đúng quyền hạn.

**Vai trò trong clickstream**

- Cho phép hệ thống xác định được:
  - `user_login_state` (logged-in / guest).
  - `identity_source` (Cognito, social login, v.v.).
- Thông tin này được gắn vào events và lưu trong S3/DWH để phục vụ phân tích hành vi người dùng.

---

## 5.2.2 Miền Ingestion & Data Lake

### (6) Amazon API Gateway – Clickstream HTTP API

**Nhiệm vụ**

- Cung cấp endpoint HTTP public cho frontend:
  - `POST /clickstream`.
- Thực hiện validate request cơ bản:
  - Kiểm tra HTTP method/path, throttling.
  - Tùy chọn: tích hợp Cognito authorizer.

**Luồng xử lý**

- Nhận JSON payload từ browser.
- Chuyển tiếp mỗi payload đến **Lambda Clickstream Ingest (7)**.

---

### (7) AWS Lambda – Clickstream Ingest

**Nhiệm vụ chính**

- Nhận events được API Gateway (6) chuyển tiếp sang.
- Đóng vai trò **ingestion layer**:

  - Validate schema và loại event.
  - Thực hiện enrichment tối thiểu:
    - Sinh `event_id` duy nhất.
    - Chuẩn hóa `event_timestamp`.
    - Gắn `client_id`, `session_id`, `user_login_state`, `identity_source` khi cần.

- Ghi events thô (`raw JSON`) vào **S3 Clickstream Raw bucket (8)** theo time-based prefix.

- Xuất logs ra **CloudWatch (17)** để theo dõi và debug.

---

### (8) Amazon S3 – Clickstream Raw Data

**Nhiệm vụ**

- Lưu toàn bộ **dữ liệu clickstream thô** do Lambda Ingest (7) nạp vào.
- Đóng vai trò một **"mini data lake"** cho downstream batch ETL:

  - Dữ liệu được tổ chức theo **prefix thời gian**:
    - `events/YYYY/MM/DD/`.
  - ETL (11) đọc dữ liệu theo lô (theo giờ / theo ngày, v.v.).

**Vai trò kiến trúc**

- Là **source of truth** cho toàn bộ dữ liệu clickstream:
  - Data Warehouse có thể được rebuild hoặc reprocess lại từ bucket này bất kỳ lúc nào.

---

### (9) Amazon EventBridge – ETL Trigger / Cron Job

**Nhiệm vụ**

- Thực thi lịch batch schedule, ví dụ: `rate(1 hour)`.
- Mỗi khi trigger được kích hoạt:
  - Gọi **Lambda ETL (11)**.

**Tại sao tách riêng**

- "Đồng hồ" ETL được đặt hoàn toàn trong EventBridge:
  - Dễ dàng điều chỉnh cadence (1h, 30m, 5m, v.v.).
  - Có thể disable/enable mà không cần chạm vào code Lambda.
  - Tách "thời điểm chạy" ra khỏi logic nghiệp vụ ETL.

---

## 5.2.3 Miền VPC & Private Analytics

### (19) VPC + Subnets + Internet Gateway

**Amazon VPC**

- Cô lập mạng và kiểm soát routing cùng security cho toàn bộ nền tảng.

**Public subnet – OLTP**

- Chứa **EC2 OLTP (20)**.
- Có route mặc định: `0.0.0.0/0 → Internet Gateway`.
- Cho phép:
  - Amplify và traffic internet truy cập PostgreSQL OLTP, được kiểm soát bởi security group.

**Private subnet – Analytics**

- Chứa:
  - **EC2 Shiny + DWH (12)**.
  - **Lambda ETL (11)**.
- Không được gán public IP.
- Lưu lượng ra ngoài chỉ đi qua **VPC Endpoints (10, 15)**.

**Internet Gateway**

- Cung cấp kết nối internet cho các tài nguyên trong public subnet.
- Private subnet không có route tới IGW (trừ khi được cấu hình rõ ràng).

---

### (10) Gateway Endpoint – S3 Gateway Endpoint

**Nhiệm vụ**

- Cho phép các tài nguyên trong VPC (Lambda ETL, EC2 private) truy cập S3 (8) **mà không cần đi qua NAT hoặc internet công khai**.
- Traffic đến S3 đi hoàn toàn trong **backbone mạng riêng của AWS**, bảo mật hơn và tiết kiệm hơn so với NAT Gateway.

**Vai trò**

- Là thành phần then chốt giúp:
  - Duy trì quyết định thiết kế "**không dùng NAT Gateway**".
  - Cho phép các thành phần ETL và DW đọc ghi S3 bình thường.

---

### (11) AWS Lambda – ETL

**Nhiệm vụ**

- Được **EventBridge (9)** trigger theo lịch.
- Thực thi trong **private subnet** của VPC.
- Chạy các thao tác ETL theo lô:

  - Đọc raw events từ **S3 (8)** qua Gateway Endpoint (10).
  - Transform, làm sạch và flatten dữ liệu theo schema của Data Warehouse (ví dụ 15 fields đã được xác nhận).
  - Nạp kết quả vào **PostgreSQL DWH trên EC2 private (12)**:
    - Insert / upsert, tùy chọn phân vùng theo ngày hoặc giờ.

- Ghi logs và metrics ra **CloudWatch (17)**; khi gặp lỗi có thể bắn thông báo qua **SNS (18)**.

---

### (12) Amazon EC2 – Private (Shiny + DWH)

**Nhiệm vụ**

- Chạy **Data Warehouse (PostgreSQL)** và **Shiny Server**.
- Tiếp nhận dữ liệu đã transform từ Lambda ETL (11) và lưu vào các bảng DWH.
- Cung cấp dữ liệu cho các dashboard phân tích Shiny:
  - Shiny chạy truy vấn trực tiếp trên Postgres instance cục bộ/private.

**Đặc điểm**

- Chạy trong **private subnet**.
- Không có địa chỉ IP public.
- Truy cập quản trị được thực hiện qua **SSM Session Manager (14 + 15)**.

---

### (13) R Shiny Server (trên EC2 Private)

**Nhiệm vụ**

- Host các dashboard phân tích bao gồm:

  - KPI tổng quan.
  - Trực quan hóa conversion funnel.
  - Bảng xếp hạng hiệu quả sản phẩm.
  - Phân tích hành vi người dùng và session.

- Kết nối trực tiếp tới **PostgreSQL DWH** trên cùng EC2 instance.

**Truy cập**

- Lắng nghe trên port `3838`.
- Thường được truy cập qua:

  - VPN nội bộ, hoặc
  - **SSM port forwarding** (14).

---

### (14) AWS Systems Manager – Session Manager

**Nhiệm vụ**

- Cấp quyền truy cập cho admin vào EC2 private **không cần SSH và không cần IP public**.
- Hỗ trợ:

  - Mở phiên terminal tương tác trên EC2 (run command).
  - **Port forwarding** (ví dụ: forward `localhost:3838` → Shiny chạy trên EC2).
  - Ghi nhật ký phiên truy cập để audit.

**Ví dụ**

- Bạn có thể truy cập Shiny từ máy local tại:  
  `http://localhost:3838/sbw_dashboard`  
  sau khi thiết lập thành công phiên SSM port forwarding.

---

### (15) Interface Endpoint – SSM VPC Endpoints

**Nhiệm vụ**

- Tạo đường kết nối **private** từ EC2/Lambda trong VPC tới các service endpoint của SSM/Session Manager:

  - `ssm`, `ssmmessages`, `ec2messages`, …

- Đảm bảo:

  - EC2 private vẫn sử dụng được SSM.
  - Traffic không bao giờ đi qua internet hoặc NAT gateway.

---

## 5.2.4 Cross-cutting Services

### (16) Amazon IAM

**Nhiệm vụ**

- Quản trị **roles và policies** trên toàn bộ hệ thống:

  - Quyền cho API Gateway và Lambda.
  - Lambda Ingest: quyền ghi vào S3 (8).
  - Lambda ETL: quyền đọc S3 (8) và kết nối tới DB (12).
  - EC2 instance role cho SSM và CloudWatch agent (nếu được triển khai).

**Nguyên tắc**

- **Least privilege** ở mọi thời điểm.
- Quyền được phân tách rõ theo chức năng và theo từng service.

---

### (17) Amazon CloudWatch

**Nhiệm vụ**

- Thu thập **logs và metrics** từ:

  - Lambda Ingest (7).
  - Lambda ETL (11).
  - EventBridge rule (9).
  - EC2, nếu CloudWatch agent được cài đặt.

- Định nghĩa **alarms** cho: lỗi ETL, lỗi ingest, số lần retry, thời gian thực thi, v.v.

**Vai trò**

- Là **trung tâm quan sát chính** để chẩn đoán vấn đề trong pipeline.

---

### (18) Amazon SNS

**Nhiệm vụ**

- Đóng vai trò **kênh thông báo** cho các sự cố vận hành:

  - ETL job thất bại.
  - Ingest function thất bại.
  - CloudWatch alarm được kích hoạt.

- Gửi thông báo qua email / SMS / webhook tùy theo cấu hình subscription.

---

### (20) Amazon EC2 – Public (OLTP)

**Nhiệm vụ**

- Chạy **PostgreSQL OLTP** phục vụ ứng dụng web thương mại điện tử:

  - Giao dịch đơn hàng.
  - Dữ liệu catalog sản phẩm và tồn kho.
  - Thông tin tài khoản người dùng.

**Liên hệ với DWH**

- **Tách biệt** khỏi Data Warehouse (12):

  - OLTP được tối ưu cho **các thao tác đọc/ghi giao dịch**.
  - DWH được tối ưu cho **truy vấn phân tích và nạp dữ liệu theo lô**.