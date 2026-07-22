---
title: "Blog 2"
date: "2026-07-09"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Cách Alight Solutions tiết kiệm 55% chi phí với Amazon OpenSearch Service
<span class="meta-info">Tác giả: Mark Larson, Andrew Kummerow, và Tim Razik | ngày: 09 tháng 07, 2026 | trong</span> [Advanced (300)](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-redshift-analytics/), [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/) | [Permalink](https://aws.amazon.com/blogs/big-data/how-alight-solutions-achieved-55-cost-savings-with-amazon-opensearch-service/)

[Alight Solutions](https://www.alight.com/) là nhà cung cấp dịch vụ và công nghệ đám mây hàng đầu về quản lý nhân sự. Hệ thống công nghệ của Alight tạo ra hơn 1 tỷ bản ghi nhật ký (logs) mỗi ngày trên toàn bộ kiến trúc microservices container hóa, với thời điểm đỉnh cao đạt 100.000 bản ghi mỗi giây.

Trước đây, Alight sử dụng hệ thống Elastic Stack tự quản lý (self-managed). Khi khối lượng logs tăng trưởng mạnh, gánh nặng vận hành hệ thống này đã ngốn hết ngân sách của đội ngũ kỹ thuật, khiến họ không còn thời gian để tập trung đổi mới công nghệ. Bài viết này chia sẻ cách Alight di chuyển sang [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/) và đạt mức giảm 55% chi phí.

---

## Những thách thức với hệ thống Elasticsearch tự quản lý

Hệ thống Elastic Stack tự quản lý của Alight mang lại nhiều khó khăn cả về kỹ thuật lẫn vận hành:
* **Chi phí bảo trì vận hành lớn:** Các công việc như vá lỗ hổng bảo mật và nâng cấp phiên bản Elasticsearch ngốn mất khoảng 2.000 giờ kỹ sư mỗi năm (tương đương 1 nhân sự toàn thời gian).
* **Độ tin cậy thu thập logs thấp:** Logstash sử dụng giao thức TCP-socket thường bị mất logs khi lưu lượng tăng cao.
* **Sự cố nghiêm trọng (P1):** Hiện tượng nghẽn cổ chai (backpressure) từ Logstash đã gây ra 2 sự cố P1 nghiêm trọng trong vòng 2 năm, ảnh hưởng trực tiếp đến các tác vụ microservices của hệ thống chính.
* **Hết hạn hỗ trợ:** Phiên bản Elasticsearch 7.x sắp hết hạn hỗ trợ tạo áp lực lớn phải di chuyển trước mùa đăng ký thường niên của khách hàng.

---

## Đánh giá các giải pháp thay thế

Alight đã đánh giá một số giải pháp như New Relic, Dynatrace và Amazon CloudWatch. Cuối cùng, họ quyết định lựa chọn **Amazon OpenSearch Service** vì 5 lý do chính:
1. **Chi phí:** Tiết kiệm hơn đáng kể so với các giải pháp đối thủ.
2. **Thay đổi tối thiểu:** Là phiên bản phân nhánh (fork) từ Elasticsearch 7.10, các kỹ sư đã quen thuộc với cú pháp truy vấn và giao diện dashboard.
3. **Tuân thủ quy định:** Sử dụng dịch vụ gốc của AWS giúp cắt giảm hàng trăm giờ làm việc để hoàn tất các thủ tục kiểm định, tuân thủ nhà cung cấp.
4. **Chiến lược Cloud-native:** Hoàn toàn đồng nhất với chiến lược dịch chuyển sang các dịch vụ đám mây gốc của doanh nghiệp.
5. **Bảo mật và riêng tư:** Dữ liệu hoàn toàn nằm trong vùng AWS Landing Zone của công ty, giải quyết triệt để lo ngại về rò rỉ dữ liệu.

---

## Kiến trúc giải pháp

Alight hợp tác với AWS thiết kế kiến trúc gom logs dạng đám mây gốc:

![Sơ đồ kiến trúc tài khoản chéo](/images/3-Blogs/Blog-2/image-1.jpeg)
*Hình 1 — Sơ đồ kiến trúc thu thập logs tài khoản chéo (cross-account)*

* **Mô hình tài khoản chéo:** Các ứng dụng ECS và EC2 chạy ở các tài khoản ứng dụng (application accounts) gửi logs trực tiếp về OpenSearch domain đặt tại tài khoản dịch vụ chung (shared services account).
* **Luồng thu thập logs:**
  * **Ứng dụng ECS:** Container phụ Fluent Bit / FireLens bắt luồng logs (stdout/stderr) và đẩy qua HTTPS đến Amazon OpenSearch Ingestion (OSIS).
  * **Ứng dụng EC2:** Fluent Bit chạy trực tiếp trên máy ảo đọc các file logs và đẩy về OSIS.
* **Bộ đệm bền vững (Persistent Buffering):** Amazon EFS được sử dụng làm bộ đệm tệp tin cho Fluent Bit để chống mất logs khi hệ thống bị quá tải hoặc mất kết nối tạm thời.
* **Đăng nhập một lần (SSO):** Quyền truy cập OpenSearch Dashboards của người dùng được quản lý tập trung qua AWS IAM Identity Center tích hợp đồng bộ SCIM.

---

## Quy trình di chuyển
Quá trình chuyển đổi diễn ra trong vòng 7 tháng (từ tháng 2 đến tháng 8 năm 2025). Đội ngũ sử dụng **Terraform** để tự động hóa hạ tầng OSIS pipelines và OpenSearch domains. Việc tích hợp (onboarding) ứng dụng mới giảm từ 80-120 giờ xuống chỉ còn 4-8 giờ. Hệ thống mới được chạy song song để xác thực dữ liệu trước khi tắt hoàn toàn cụm Elasticsearch cũ.

---

## Kết quả đạt được

Quá trình chuyển đổi mang lại kết quả vượt bậc trên cả 3 phương diện: chi phí, vận hành và hiệu năng:

### Tối ưu chi phí & Bản quyền
* **Tiết kiệm chi phí:** Cắt giảm 55% chi phí hạ tầng và bản quyền hàng tháng (dự kiến đạt 65% sau khi tắt hoàn toàn các cụm cũ).
* **Không mất phí bản quyền:** Loại bỏ hoàn toàn chi phí bản quyền cố định của Elastic Platinum.

### Cải thiện vận hành

| Chỉ số | Elasticsearch Tự quản lý | Managed OpenSearch Service |
| :--- | :--- | :--- |
| **Công sức kỹ sư quản lý cụm** | ~2.000 giờ/năm | Gần như bằng 0 (AWS quản lý) |
| **Vá lỗi bảo mật & Bảo trì** | Thủ công (phải làm cả ngày nghỉ) | AWS tự động xử lý |
| **Tích hợp ứng dụng mới** | 80–120 giờ (thủ công) | 4–8 giờ (Terraform) |
| **Sự cố P1 do lỗi logs** | 2 sự cố trong 2 năm | 0 sự cố từ khi di chuyển |

---

## Bài học kinh nghiệm
* **Tách biệt luồng dữ liệu quan trọng:** Không nên đặt kỳ vọng 100% không mất logs vào hệ thống logging. Nên sử dụng luồng sự kiện riêng như Amazon SQS đối với dữ liệu nghiệp vụ quan trọng không thể mất.
* **Tự động hóa hạ tầng (IaC):** Chuẩn hóa bằng Terraform giúp giảm tối đa thời gian triển khai ứng dụng.
* **Lên kế hoạch quanh các kỳ kinh doanh cao điểm:** Tạm dừng triển khai trong các đợt đăng ký lớn của khách hàng là quyết định đúng đắn để tránh rủi ro hệ thống.

Để biết thêm chi tiết, bạn có thể tham khảo bài viết gốc tại [AWS Big Data Blog: How Alight Solutions achieved 55% cost savings with Amazon OpenSearch Service](https://aws.amazon.com/vi/blogs/big-data/how-alight-solutions-achieved-55-cost-savings-with-amazon-opensearch-service/).