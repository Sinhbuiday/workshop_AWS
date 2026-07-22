---
title: "Worklog Tuần 4"
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Tìm hiểu về máy tính ảo trên AWS .
- Có các hệ điều hành nào và cái nào dùng phổ biến nhất trên AWS.
- EC2 được điều hành như thế nào và tại sao lại dùng nhiều.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                             | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                |
| --- | --------------------------------------------------------------------- | ------------ | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance type AWS.           | 25/05/2026   | 26/05/2026      |
| 2   | - Amazon Elastic Compute Cloud ( EC2 ) - AMI / Backup / Key Pair EC2. | 26/05/2026   | 26/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon Elastic Compute Cloud ( EC2 ) - Elastic block store.         | 27/05/2026  | 27/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance store               | 27/05/2026  | 27/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Amazon Elastic Compute Cloud ( EC2 ) - User data                    | 28/05/2026   | 28/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 6   | - Amazon Elastic Compute Cloud ( EC2 ) - Meta data                    | 28/05/2026   | 29/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 7   | - Amazon Elastic Compute Cloud ( EC2 ) - EC2 auto scaling             | 30/05/2026    | 30/05/2026       | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 8   | - EC2 Autoscaling - EFS/FSx - Lightsail - MGN                         | 31/05/2026    | 31/05/2026       | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Kết quả đạt được tuần 4:

- Tìm hiểu hệ sinh thái Máy chủ ảo AWS
    - AWS cung cấp giải pháp máy chủ và lưu trữ ảo hóa đa dạng bao gồm: Amazon EC2, Amazon Lightsail, Amazon EFS/FSx và AWS Application Migration Service (MGN).

  - Chi tiết về Amazon EC2
    - Bản chất: Dịch vụ máy chủ điện toán linh hoạt, khởi tạo nhanh hơn nhiều so với máy chủ vật lý truyền thống nhưng vẫn đáp ứng đầy đủ các tác vụ như lưu trữ ứng dụng, vận hành ứng dụng/website và cơ sở dữ liệu.

    - Thành phần của một Instance Type: Cấu hình chuẩn bao gồm Vi xử lý (CPU/GPU), Bộ nhớ RAM, Kết nối Mạng và Ổ đĩa Lưu trữ.

    - Quản lý Hình ảnh & Bảo mật (AMI & Backup):

      - AMI (Amazon Machine Image): Mẫu đóng gói chứa hệ điều hành root, quyền truy cập và sơ đồ gắn đĩa EBS; dùng để nhân bản một hoặc nhiều máy chủ cùng lúc (cung cấp bởi AWS, AWS Marketplace hoặc do người dùng tự tạo).

      - Sao lưu & Bảo mật: Dùng Snapshot để lưu trữ trạng thái máy chủ và Key Pair (khóa công khai/bảo mật) để mã hóa thông tin đăng nhập.

    - Phương thức Lưu trữ:

      - Amazon EBS (Elastic Block Store):

        - Ổ đĩa dạng khối nối qua mạng riêng biệt, hoạt động độc lập với EC2.

        - Hỗ trợ chuẩn SSD và HDD, tự động nhân bản dữ liệu qua 3 node trong cùng một AZ để đạt độ sẵn sàng 99.999%.

        - Mặc định 1 đĩa EBS gắn cho 1 EC2, nhưng kiến trúc Nitro Hypervisor cho phép 1 đĩa EBS kết nối đồng thời nhiều EC2 (Multi-Attach).

        - Bản sao lưu Snapshot đầu tiên là dạng Full-backup, các bản tiếp theo lưu dưới dạng tăng dần (Incremental) tại Amazon S3.

      - Instance Store:

        - Ổ đĩa NVMe trực tiếp trên phần cứng vật lý chứa EC2, mang lại tốc độ IOPS siêu cao.

        - Đặc tính dữ liệu: Không mất khi Restart nhưng sẽ mất sạch nếu Stop máy chủ hoặc gặp sự cố phần cứng. Không hỗ trợ tự sao lưu nên thường được đồng bộ sang EBS để an toàn.

    - Nâng cao & Tự động hóa:

      - User Data: Đoạn mã script (Bash trên Linux hoặc PowerShell trên Windows) tự động thực thi duy nhất một lần khi khởi tạo EC2 từ AMI.

      - EC2 Metadata: Tập hợp dữ liệu định danh của chính Instance đó (Private/Public IP, Hostname, Security Group,...).

      - EC2 Auto Scaling: Tự động điều chỉnh số lượng máy chủ dựa trên tải thực tế, hỗ trợ phân bổ đa vùng khả dụng (AZ), kết nối với Load Balancer và áp dụng linh hoạt các mô hình chi phí.

  - Mô hình Chi phí & Dịch vụ liên quan
    - 4 Hình thức Thanh toán EC2:

      - On-Demand: Trả tiền theo thời gian dùng thực tế (phút/giây), tối ưu cho tác vụ ngắn hạn (ví dụ vài giờ/ngày).

      - Reserved Instances: Cam kết sử dụng 1–3 năm cho một họ máy chủ cố định để nhận ưu đãi giảm giá.

      - Savings Plans: Cam kết mức dùng 1–3 năm để nhận chiết khấu, linh hoạt hơn về loại máy chủ.

      - Spot Instances: Tận dụng tài nguyên dư thừa của AWS với giá rẻ, nhưng có thể bị thu hồi sau 2 phút thông báo.

    - Amazon Lightsail:

      - Giải pháp máy chủ gói gọn, chi phí cố định (từ $3.5/tháng), đi kèm băng thông giá rẻ.

      - Phù hợp cho môi trường Dev/Test hoặc ứng dụng nhẹ không yêu cầu CPU chạy liên tục (>2 tiếng/ngày).

      - Hoạt động trong VPC riêng biệt (kết nối VPC khác qua Peering) và có hỗ trợ tạo Snapshot.

    - Giải pháp Lưu trữ Tệp chia sẻ (EFS & FSx):

      - Amazon EFS: Hệ thống tệp chuẩn NFSv4 dành riêng cho Linux, cho phép nhiều EC2 truy cập đồng thời, tự động mở rộng dung lượng đến Peatabyte và chỉ tính tiền trên dung lượng thực tế lưu trữ. Hỗ trợ kết nối On-premises qua Direct Connect/VPN.

      - Amazon FSx: Hệ thống tệp chuẩn NTFS qua giao thức SMB hỗ trợ cả Windows & Linux. Hỗ trợ tính năng khử trùng lặp (Deduplication) giúp tiết kiệm 30–50% chi phí.

    - AWS Application Migration Service (MGN):

      - Công cụ dịch chuyển và nhân bản máy chủ vật lý/ảo hóa lên AWS để xây dựng trung tâm dự phòng sự cố (DR).

      - Đồng bộ dữ liệu liên tục về các máy chủ trung gian (Staging) có cấu hình tối thiểu trên AWS.

      - Khi chuyển giao (Cutover), hệ thống gốc sẽ dừng lại và hệ thống máy chủ EC2 chính thức trên AWS được khởi chạy ngay lập tức.
