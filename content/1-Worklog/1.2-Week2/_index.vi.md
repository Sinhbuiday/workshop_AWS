---
title: "Worklog Tuần 2"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Học và hiểu VPC,VPN là gì
- Cách tạo, kết nối và hoạt động của dịch vụ mạng
- Chạy được 1 server ảo

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                         | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                |
| --- | ------------------------------------------------- | ------------ | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Tìm hiểu khái niệm VPC, VPN tập                 | 11/05/2026   | 12/05/2026      |
| 2   | - Thực hành: Sử dụng dịch vụ của AWS với VPC, VPN | 12/05/2026   | 13/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Tạo **Subnets**, **Internet Gateway**           | 13/05/2026   | 13/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Tạo **Route Table**, **security groups**        | 14/05/2026   | 15/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Tạo **Route Table**, **Security groups**        | 16/05/2026   | 17/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Kết quả đạt được tuần 2:

  - Kiến trúc VPC & Thành phần Mạng cơ bản
    - Tổng quan Amazon VPC: Cung cấp môi trường mạng ảo độc lập giúp triển khai an toàn các tài nguyên AWS (như EC2).

    - Phạm vi: Nằm hoàn toàn trong một Region cụ thể nhưng có thể trải rộng qua nhiều vùng khả dụng (AZ).

    - Định danh IP: Bắt buộc khai báo dải IPv4 CIDR khi khởi tạo; hỗ trợ IPv6 tùy chọn.

    - Giới hạn & Ứng dụng: Tối đa 5 VPC cho mỗi Region trên một tài khoản. Thường dùng để phân tách không gian mạng giữa các môi trường (Dev, Staging, Prod).

  - Phân chia Subnet:

    - Mỗi Subnet thuộc duy nhất một AZ và nhận một dải IP con cắt ra từ CIDR của VPC.

  - 5 địa chỉ IP bị AWS giữ lại trong mỗi Subnet:

    - Network Address (VD: 10.10.1.0)

    - Router Address (VD: 10.10.1.1)

    - DNS Address (VD: 10.10.1.2)

    - Địa chỉ dự phòng tương lai (VD: 10.10.1.3)

    - Broadcast Address (VD: 10.10.1.255)

  - Định tuyến & Giao diện Mạng
    - Route Table: Tập hợp các chỉ dẫn hướng đi cho lưu lượng mạng.

    - Default Route Table: Tạo tự động, không thể xóa, chứa tuyến mặc định giúp các Subnet nội bộ thông suốt với nhau.

    - Custom Route Table: Bảng định tuyến tùy chỉnh do người dùng tạo thêm (vẫn duy trì tuyến liên lạc nội bộ mặc định).

  - ENI (Elastic Network Interface): Card mạng ảo gắn vào EC2, lưu giữ cố định cấu hình mạng (Private IP, Elastic IP, địa chỉ MAC) ngay cả khi thay đổi máy chủ.

  - Kết nối & Kiểm soát An ninh Mạng
    - VPC Endpoint: Kết nối riêng tư tới các dịch vụ AWS mà không cần đi qua Internet public (gồm loại Interface và Gateway).

    - So sánh Tường lửa Bảo mật:

      - Security Group (SG): Tường lửa ảo Stateful hoạt động ở cấp độ ENI/Instance. Chỉ hỗ trợ quy tắc Allow (dựa trên IP nguồn, Port hoặc SG khác).

      - NACL (Network Access Control List): Tường lửa ảo Stateless hoạt động ở cấp độ Subnet. Hỗ trợ cả quy tắc Allow lẫn Deny. NACL mặc định mở toàn bộ lưu lượng ra/vào.

      - VPC Flow Logs: Thu thập dữ liệu giám sát luồng IP truy cập ra/vào VPC (lưu tại CloudWatch hoặc S3, không ghi lại nội dung chi tiết của gói tin).

  - Mở rộng Mạng & Điều hướng Dòng dữ liệu
    - Phương thức liên kết VPC:

    - VPC Peering: Kết nối trực tiếp 1-1 giữa 2 VPC (không chấp nhận dải IP trùng lặp).

      - Transit Gateway: Trạm trung chuyển đóng vai trò Hub kết nối hàng loạt VPC và mạng nội bộ.

      - Giải pháp VPN & Kết nối Lai (Hybrid):

    - Site-to-Site VPN: Đảm bảo liên lạc liên tục giữa hạ tầng On-Premises và VPC.

      - Client-to-Site VPN: Cho phép các thiết bị người dùng cuối truy cập bảo mật vào VPC.

    - AWS Direct Connect: Đường truyền cáp riêng biệt kết nối On-Premises tới AWS với độ trễ siêu thấp (20–30ms).

    - Elastic Load Balancing (ELB): Bộ cân bằng tải tự động hỗ trợ các giao thức HTTP, HTTPS, TCP, SSL với 4 dòng sản phẩm chính: ALB, NLB, CLB và GLB.

  - Thực hành Thiết lập Mạng
    - Tìm kiếm dịch vụ VPC trên Console -> Tiến hành tạo mới và đặt tên.

    - Thiết lập dải địa chỉ IPv4 CIDR: 10.10.0.0/16.

    - Khởi tạo các Subnets tương ứng.

    - Gắn thêm Internet Gateway để mở kết nối Internet.

    - Cấu hình Security Groups để phân quyền truy cập.
