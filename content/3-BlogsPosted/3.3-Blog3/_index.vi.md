---
title: "Blog 3"
date: "2023-07-17"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Thu thập dữ liệu clickstream bằng các dịch vụ serverless của AWS
<span class="meta-info">Tác giả: Pritam Bedse và Jared Warren | ngày: 17 tháng 07, 2023 | trong</span> [Advanced (300)](https://aws.amazon.com/blogs/industries/category/cross-industry/), [AWS for Industries](https://aws.amazon.com/blogs/industries/) | [Permalink](https://aws.amazon.com/blogs/industries/capture-clickstream-data-using-aws-serverless-services/)

Dữ liệu clickstream (luồng nhấp chuột) đề cập đến tập hợp các tương tác kỹ thuật số diễn ra giữa người dùng và trang web hoặc ứng dụng di động. Việc thu thập và tạo ra thông tin chi tiết hữu ích từ dữ liệu người dùng trong thời gian thực có thể là một thách thức lớn.

Bài viết này trình bày chi tiết cách tận dụng các dịch vụ không máy chủ (serverless) của AWS để xây dựng một kiến trúc có khả năng mở rộng linh hoạt, giúp thu thập, xử lý, trực quan hóa và tải dữ liệu clickstream vào các nền tảng phân tích một cách liền mạch.

---

## Kiến trúc giải pháp

Giải pháp sử dụng Amazon API Gateway, AWS Lambda và Amazon Kinesis Data Streams để tiếp nhận và xử lý dữ liệu clickstream. Sau đó, Amazon Kinesis Data Firehose được dùng để lưu trữ dữ liệu thô vào Amazon S3, tiếp theo là Amazon Athena và Amazon QuickSight để phân tích và trực quan hóa các tương tác của người dùng.

![Sơ đồ kiến trúc Clickstream](/images/3-Blogs/Blog-3/image-1.png)
*Hình 1 — Sơ đồ luồng dữ liệu Clickstream Serverless*

1. **Tiếp nhận (Ingestion):** Ứng dụng khách (ví dụ: cổng thông tin web) gửi các payload clickstream (dữ liệu nhấp chuột, lượt xem trang, thời gian ở lại trang, lượt gửi form) đến **Amazon API Gateway**.
2. **Chuẩn hóa (Standardization):** API Gateway truyền bản ghi dữ liệu tới **AWS Lambda** để tiến hành làm sạch và chuẩn hóa dữ liệu.
3. **Luồng dữ liệu (Streaming):** Lambda gửi các bản ghi đã chuẩn hóa đến **Amazon Kinesis Data Streams** để xử lý truyền phát thời gian thực không đồng bộ ở quy mô lớn.
4. **Bộ đệm & Lưu trữ (Buffering & Delivery):** Kinesis Data Streams chuyển dữ liệu sang **Amazon Kinesis Data Firehose**. Dịch vụ này gom dữ liệu theo chu kỳ và tải lên một **Amazon S3** bucket làm data lake.
5. **Phân tích (Analytics):** **Amazon Athena** chạy các truy vấn SQL trực tiếp trên dữ liệu thô dạng CSV lưu trong S3.
6. **Trực quan hóa (Visualization):** **Amazon QuickSight** kết nối với Athena để hiển thị các dashboard tương tác (BI) hiện đại cho doanh nghiệp.

---

## Tại sao chọn mô hình Serverless cho Clickstream?

* **Khả năng mở rộng tự động:** Lưu lượng clickstream thay đổi liên tục theo hành vi người dùng. Kiến trúc serverless tự động co giãn tài nguyên để đáp ứng lưu lượng truy cập tăng vọt (như trong đợt đăng ký mua sắm, ngày hội giảm giá) mà không cần cấu hình thủ công.
* **Tối ưu chi phí (Pay-as-you-go):** Không mất chi phí duy trì, quản lý máy chủ. Doanh nghiệp chỉ chi trả cho thời gian chạy thực tế và dung lượng dữ liệu truyền qua hệ thống, tránh việc lãng phí tài nguyên dự phòng.
* **Thông tin thời gian thực:** Luồng xử lý qua Kinesis và Lambda giúp dữ liệu sẵn sàng phân tích gần như lập tức, hỗ trợ doanh nghiệp đánh giá ngay hiệu năng của các tính năng hoặc chiến dịch marketing vừa ra mắt.

---

## Kiểm thử giải pháp

Để kiểm thử kiến trúc, một hàm Lambda giả lập sẽ tạo và gửi ngẫu nhiên các tương tác của người dùng e-commerce. Payload bao gồm các trường thông tin cơ bản:
* `customerid` / `deviceid`: Định danh người dùng và thiết bị để theo dõi hành vi.
* `productid` / `productcategory` / `productsubcategory`: Danh mục sản phẩm người dùng đang xem.
* `activitytype`: Hành động cụ thể (ví dụ: click xem sản phẩm, thêm vào giỏ hàng, thanh toán).

Dữ liệu giả lập sau đó được Kinesis Data Firehose lưu đệm trong 60 giây trước khi ghi vào Amazon S3 dưới dạng tệp CSV. Sau đó, lập tức có thể sử dụng câu lệnh SQL chuẩn trên **Amazon Athena** để truy vấn trực tiếp bảng `clickstream_data` này.

---

## Kết luận
Tận dụng các dịch vụ serverless của AWS mang lại một giải pháp mạnh mẽ, bảo mật và có khả năng mở rộng cao để thu thập dữ liệu clickstream. Kiến trúc này giúp doanh nghiệp nhanh chóng đưa ra các quyết định dựa trên dữ liệu và cải thiện trải nghiệm khách hàng, đồng thời loại bỏ hoàn toàn gánh nặng vận hành hạ tầng máy chủ phức tạp.

Chi tiết bài viết tham khảo gốc tại [AWS for Industries Blog: Capture clickstream data using AWS serverless services](https://aws.amazon.com/blogs/industries/capture-clickstream-data-using-aws-serverless-services/).