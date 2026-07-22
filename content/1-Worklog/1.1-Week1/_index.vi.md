---
title: "Worklog Tuần 1"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu cần làm tuần 1:

- Lên văn phòng cùng team và hiểu nội quy của văn phòng.
- Hiểu cách viết workshop.
- Tìm hiểu AWS tạo tài khoản.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                  | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                 |
| --- | ---------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ---------------------------------------------------------------------------------------------- |
| 1   | - Hiểu rõ nội quy của văn phòng và thực hiện đúng cho những buổi sau                                       | 04/05/2026   | 05/05/2026      |
| 2   | - Viết workshop cách tạo sườn                                                                              | 05/05/2026   | 06/05/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3>  |
| 3   | - Tạo tài khoản AWS và làm nhiệm vụ nhận 200 credit free <br> - Thực hành sơ bộ với các nhiệm vụ nhận 200$ | 06/05/2026   | 07/05/2026      |                                                                                                |
| 4   | - Học và hiểu về cloud                                                                                     | 07/05/2026   | 08/05/2026      | <https://www.youtube.com/watch?v=HxYZAK1coOI&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=4>  |
| 5   | - Thực hiện tạo IAM user để admin user và admin group                                                      | 09/05/2026   | 10/05/2026      | <https://www.youtube.com/watch?v=b9pK1oG534Q&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=13> |

### Kết quả đạt được tuần 1:

 - Điện toán đám mây là gì?

Là hình thức cung cấp các tài nguyên công nghệ thông tin thông qua Internet, thay vì phải đầu tư hạ tầng vật lý.
Áp dụng mô hình trả phí theo mức sử dụng (pay-as-you-go) — dùng bao nhiêu trả bấy nhiêu.
Giúp tối ưu chi phí vì không phải trả cho tài nguyên nhàn rỗi, đồng thời vẫn đảm bảo tốc độ xử lý cao.
Có khả năng mở rộng hoặc thu hẹp tài nguyên linh hoạt tùy theo nhu cầu thực tế.
Được triển khai trên quy mô toàn cầu, phù hợp sử dụng ở bất kỳ đâu trên thế giới.

  - Cách tiếp cận và học Cloud Computing:

Trao đổi, học hỏi kinh nghiệm từ những người đã có chuyên môn đi trước.
Thực hành trực tiếp trên các dịch vụ của AWS để hiểu bản chất.
Tự học thêm qua các nền tảng đào tạo trực tuyến.

  - Khởi tạo tài khoản AWS và làm quen với công cụ:

Đã tạo tài khoản thành công và nhận được 200 USD sau khi hoàn thành các nhiệm vụ kích hoạt.
Hiểu rõ sự khác biệt giữa Root User và IAM User:
Root User là tài khoản gốc, có toàn quyền quản lý hệ thống.
IAM User là tài khoản con do Root User tạo ra và được cấp quyền hạn cụ thể.
Đã thiết lập xác thực đa yếu tố (MFA) để tăng cường bảo mật cho tài khoản.

  - Các bước tạo User trong IAM:

Vào AWS Console, tìm kiếm dịch vụ IAM (Identity and Access Management).
Trên Dashboard, chú ý hai mục chính: Users và User Groups.
Chọn Users → Create user.
Nhập tên người dùng, tick chọn mục cho phép truy cập AWS Management Console (tùy chọn), sau đó chọn hình thức tạo mật khẩu cho IAM User.
Nhấn Next rồi Create user để hoàn tất.

  - Các bước tạo User Group trong IAM:

Vào AWS Console, tìm IAM.
Chọn User Groups → Create user group, đặt tên nhóm và tạo.
Mở nhóm vừa tạo, chọn Add user, tick chọn user cần thêm rồi xác nhận.

