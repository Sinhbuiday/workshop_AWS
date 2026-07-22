---
title: "Tóm tắt & Dọn dẹp"
weight: 56
chapter: false
pre: " <b> 5.6. </b> "
---

## 5.6.1 Tóm tắt

Kết thúc bài Lab, chúng ta đã xây dựng thành công một nền tảng **Clickstream Analytics Platform** vận hành hoàn chỉnh:

1. **Lớp User-Facing**
   - Ứng dụng Next.js (`ClickSteam.NextJS`) được triển khai thông qua Amplify và CloudFront  
   - Quản lý định danh và xác thực người dùng bằng Cognito  
   - Cơ sở dữ liệu PostgreSQL OLTP (`clickstream_web`) đặt trên `SBW_EC2_WebDB` (nằm trong public subnet)  

2. **Lớp Ingestion & Raw Data**
   - Endpoint API Gateway HTTP API: `clickstream-http-api` (cung cấp route `POST /clickstream`)  
   - Hàm xử lý Lambda Ingest: `clickstream-lambda-ingest`  
   - S3 Raw bucket lưu trữ gốc: `clickstream-s3-ingest/events/YYYY/MM/DD/event-<uuid>.json`  

3. **Lớp Analytics Private**
   - Mạng VPC chứa cả public và private subnets (`SBW_Project_VPC`)  
   - Định tuyến an toàn bằng S3 Gateway Endpoint và các SSM Interface Endpoints  
   - Data Warehouse chạy trên EC2: `SBW_EC2_ShinyDWH`, quản trị DB `clickstream_dw`  
   - Hàm ETL Lambda hoạt động trong VPC: `SBW_Lamda_ETL`, được tự động kích hoạt bởi `SBW_ETL_HOURLY_RULE`  
   - Các R Shiny dashboards (`sbw_dashboard`) phục vụ phân tích, được bảo mật và chỉ có thể truy cập bằng SSM port forwarding  

Tựu trung, kiến trúc này phác họa phương pháp thiết kế một **batch-based analytics platform** đề cao tính bảo mật, tối ưu hóa chi phí, tận dụng sức mạnh của các dịch vụ serverless kết hợp với hai máy chủ EC2.

---

## 5.6.2 Nội dung chính 

- **Separation of concerns (Phân tách mối quan tâm)**:
  - Khối lượng công việc OLTP và Analytics được phân chia rõ ràng trên hai máy chủ EC2, tách biệt hoàn toàn về miền logic và yêu cầu hiệu năng.  

- **Security**:
  - DW và Shiny Server được bảo vệ an toàn trong private subnet, không lộ diện trước internet.  
  - Truy cập SSH rủi ro được thay thế hoàn toàn bằng SSM Session Manager an toàn hơn.  
  - S3 Gateway Endpoint đảm bảo mọi lưu lượng tới S3 luôn nằm gọn trong mạng nội bộ của AWS.  

- **Tối ưu chi phí**:
  - Hệ thống loại bỏ hoàn toàn chi phí đắt đỏ của NAT Gateway.  
  - Tiến trình ETL được xử lý bởi các dịch vụ serverless linh hoạt và tiết kiệm (Lambda + EventBridge).  
  - S3 cung cấp giải pháp lưu trữ dữ liệu thô với độ bền cao và chi phí thấp.  

- **Dễ mở rộng**:
  - Dù đang áp dụng mô hình batch-based, nền tảng này hoàn toàn có thể được nâng cấp để xử lý luồng sự kiện real-time, áp dụng phân tích ML nâng cao, hoặc dịch chuyển sang các hệ thống DW quy mô lớn hơn.

---

## 5.6.3 Dọn dẹp Resource

1. **Amplify & CloudFront**
   - Xóa bỏ ứng dụng Amplify (`ClickSteam.NextJS`).  
   - Thao tác này sẽ tự động dọn dẹp CloudFront distribution liên kết.

2. **API Gateway & Lambda**
   - Loại bỏ API Gateway `clickstream-http-api`.  
   - Tiến hành xóa các hàm Lambda sau:
     - `clickstream-lambda-ingest`  
     - `SBW_Lamda_ETL`  

3. **EventBridge**
   - Xóa bỏ rule đặt lịch `SBW_ETL_HOURLY_RULE`.  

4. **S3 Buckets**
   - Xóa sạch dữ liệu (empty) bên trong, sau đó tiến hành xóa bucket:
     - `clickstream-s3-ingest` (nơi chứa RAW clickstream)  
     - `clickstream-s3-sbw` (nơi chứa assets), với điều kiện bucket này không còn phục vụ cho dự án nào khác  

5. **EC2 Instances**
   - Dừng lại hoặc terminate vĩnh viễn:
     - `SBW_EC2_WebDB`  
     - `SBW_EC2_ShinyDWH`  
   - Trả lại (release) các Elastic IP nếu chúng từng được gắn vào các instance này.

6. **VPC & Networking**
   - Xóa bỏ mọi VPC endpoints (bao gồm S3 Gateway và các SSM Interface Endpoints).  
   - Tiếp tục xóa các route tables, subnets, và Internet Gateway.  
   - Sau khi VPC đã trống rỗng, thực hiện xóa `SBW_Project_VPC`.
