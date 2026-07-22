---
title: "Mục tiêu & Phạm vi"
weight: 51
chapter: false
pre: " <b> 5.1. </b> "
---

### Bối cảnh kinh doanh

Hệ thống mục tiêu là một website thương mại điện tử chuyên bán laptop, màn hình và phụ kiện máy tính.  
Doanh nghiệp mong muốn:

- Nắm rõ **cách người dùng tương tác** với website:
  - Họ điều hướng tới những trang nào
  - Họ xem và duyệt qua sản phẩm nào
  - Các sự kiện thêm vào giỏ hàng và thanh toán
- Đo lường **conversion funnel** xuyên suốt hành trình: product view → add to cart → checkout
- Xác định:
  - Sản phẩm hiệu quả nhất (bán chạy / được xem nhiều)
  - Khung giờ lưu lượng cao điểm (theo giờ và theo ngày)
- Tận dụng những insights này để xây dựng chiến lược marketing hiệu quả nhằm:
  - Tăng doanh thu
  - Tối ưu chi tiêu và giảm chi phí không cần thiết

Để thực hiện điều đó, nền tảng:

- Thu thập tập dữ liệu hành vi người dùng thông qua clickstream events, sau đó tổng hợp thống kê và báo cáo phân tích.

---

### Mục tiêu học tập

#### Hiểu kiến trúc

- Có khả năng trình bày kiến trúc đầu cuối của một **batch-based clickstream analytics platform** sử dụng:
  - Amplify, CloudFront, Cognito, EC2 OLTP ở **miền user-facing**
  - API Gateway, Lambda Ingest, S3 Raw bucket ở **miền ingestion & data lake**
  - ETL Lambda, PostgreSQL DW trên EC2, R Shiny ở **miền analytics & DW**
- Có khả năng lý giải vì sao OLTP và Analytics được tách biệt:
  - **Logic**: schema cơ sở dữ liệu khác nhau, loại workload khác nhau
  - **Physical**: public subnet vs. private subnet, hai EC2 instance riêng biệt

#### Kỹ năng thực hành

- Gửi clickstream events từ frontend qua API Gateway → Lambda Ingest → S3 Raw (`clickstream-s3-ingest`).
- Thiết lập **Gateway VPC Endpoint cho S3** và cập nhật private route table để các thành phần private có thể truy cập S3.
- Cấu hình và kiểm tra **ETL Lambda** (`SBW_Lamda_ETL`) có khả năng:
  - Đọc file JSON thô từ `s3://clickstream-s3-ingest/events/YYYY/MM/DD/`
  - Chuyển đổi events thành các dòng dữ liệu cho bảng `clickstream_dw.public.clickstream_events`
- Kết nối vào DW (`SBW_EC2_ShinyDWH`) và thực thi các câu truy vấn SQL mẫu:
  - Đếm số lượng event
  - Top sản phẩm
  - Phân tích funnel cơ bản
- Truy cập **R Shiny dashboards** qua SSM port forwarding và đọc hiểu:
  - Biểu đồ funnel
  - Biểu đồ mức độ tương tác sản phẩm
  - Biểu đồ time-series hoạt động theo thời gian

#### Nhận thức về bảo mật và chi phí

- Hiểu lý do bảo vệ dữ liệu hành vi người dùng bằng cách đặt `EC2_ShinyDWH` và `Lambda_ETL` trong **private subnet**:
  - Chỉ `Lambda_ETL` được phép truy cập S3 qua **Gateway VPC Endpoint**
  - Chỉ quản trị viên có thể vào `EC2_ShinyDWH` thông qua SSM bằng **Interface VPC Endpoints**
- Nhận biết các control bảo mật quan trọng trong hệ thống:
  - Phân tách public subnet và private subnet
  - Ranh giới security group giữa `sg_oltp_webDB`, `sg_Lambda_ETL`, `sg_analytics_ShinyDWH`
  - IAM permissions tối thiểu (least-privilege) riêng cho từng Lambda function
  - Quản trị không cần SSH qua AWS Systems Manager Session Manager (không cần bastion host, không mở cổng SSH).

---

### Phạm vi Workshop

Workshop tập trung vào ba nhóm kỹ năng chính:

1. **Triển khai lớp ingestion clickstream**
   - Thu thập tương tác phía browser trong frontend Next.js
   - Đẩy JSON events lên API Gateway (`clickstream-http-api`)
   - Lưu trữ bền vững các events thô theo thời gian trong `clickstream-s3-ingest`

2. **Xây dựng lớp analytics private**
   - Khởi tạo VPC, subnets, route tables và VPC endpoints
   - Chạy `SBW_Lamda_ETL` bên trong VPC
   - Kết nối Lambda ETL tới EC2 Data Warehouse private (`SBW_EC2_ShinyDWH`)

3. **Trực quan hóa analytics với Shiny dashboards**
   - Truy vấn `clickstream_dw` trực tiếp từ R Shiny
   - Hiển thị funnel, chỉ số hiệu quả sản phẩm và xu hướng theo thời gian
   - Truy cập Shiny thông qua **SSM Session Manager port forwarding**

---

### Ngoài phạm vi

Để workshop duy trì trọng tâm và khả thi, chúng ta **không** đi sâu vào:

- Giải pháp streaming thời gian thực (Kinesis, Kafka, MSK, …)
- Các dịch vụ DW được quản lý (Amazon Redshift / Redshift Serverless)
- Recommendation engine, phân khúc người dùng, hay phát hiện bất thường bằng ML
- CI/CD production, blue/green deployment, hay kiến trúc multi-account
- Tối ưu SQL nâng cao hay chiến lược thiết kế index chi tiết

Đây là những hướng mở rộng tự nhiên sau khi nền tảng batch-based clickstream trong workshop này đã đi vào vận hành.
