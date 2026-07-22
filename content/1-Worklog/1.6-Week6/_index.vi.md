---
title: "Worklog Tuần 6"
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

- Tìm hiểu dịch vụ bảo mật trên AWS.
- Quản lý danh tính và quyền truy cập của Amazon.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                             | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                |
| --- | ----------------------------------------------------- | ------------ | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Amazon Cognito - AWS Organization                   | 8/10/2025   | 9/10/2025      |
| 2   | - AWS Identity Center - Amazon Key Management Service | 10/10/2025   | 11/10/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - AWS Security Hub                                    | 13/10/2025   | 14/10/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Kết quả đạt được tuần 6:

- Mô hình Chia sẻ Trách nhiệm:

  - Mô hình chia sẻ trách nhiệm: Khi sử dụng dịch vụ trên nền tảng đám mây, việc bảo mật chia sẻ trách nhiệm giữa khách hàng và nhà cung cấp dịch vụ.
  - Khách hàng sẽ chịu trách nhiệm cho việc bảo vệ dữ liệu, áp dụng các best practices, sử dụng các dịch vụ để đảm bảo việc bảo mật máy ảo, Hypervisor hoặc máy chủ dữ liệu.
  - Trách nhiệm bảo mật sẽ thay đổi tùy vào tầng dịch vụ:
    - Các dịch vụ cơ bản **(IaaS)**
    - Các dịch vụ quản lý máy chủ **(PaaS)**
    - Các dịch vụ quản lý toàn bộ **AWS**

- Quản lý danh tính và quyền truy cập của Amazon:

  - Tài khoản gốc: Tài khoản này có toàn quyền truy cập vào tất cả các dịch vụ và tài nguyên AWS, và có thể bổ sung thêm quyền vào các tài nguyên.
  - Thông tin tài khoản: **Đã liên kết (khi đăng ký account)**, **Không bị giới hạn quyền**.
  - Best Practices: Tạo và sử dụng **IAM Administrator User**, Khóa thông tin xác thực của **root user (chia sẻ người dùng)**, Đảm bảo renew thông tin domain qua email của root user.
  - IAM là dịch vụ giúp bạn kiểm soát quyền truy cập vào các dịch vụ và tài nguyên trong AWS account của mình. IAM cho phép bạn tạo nhiều tài khoản người dùng (IAM user) với thông tin xác thực (credentials) và quyền hạn khác nhau.
  - IAM Principal (chủ thể IAM) là một thực thể có thể thực hiện các hành động trên tài nguyên trong AWS account của bạn.
    - AWS account and root user
    - IAM users
    - Federated users (sử dụng web identity hoặc SAML federation)
    - IAM roles
    - Assumed-role sessions
    - AWS services
    - Anonymous users (không khuyến nghị)
  - Người dùng IAM (IAM User) không phải là tài khoản AWS riêng biệt, IAM Users có thể được quản lý thông qua Management Console hoặc access key/secret key để thực hiện programmatic access (AWS CLI hoặc AWS SDK).
    - IAM User khi được tạo ra mặc định không có bất cứ quyền nào.
    - IAM User không được dùng để quản lý truy cập vào ứng dụng hay hệ thống khác.
    - Đề cập quyền cho IAM User chủ yếu thông qua việc gắn IAM Policy vào IAM User.
    - Đề quản lý đề dành hơn cho việc gắn với một nhóm nhiều IAM User thông qua IAM Group.
    - IAM Group không thể là thành viên của 1 IAM Group khác.
  - AM Policy: IAM Policy có 2 loại
    - **Identity based Policy** gắn với một **IAM Principal**.
    - **Resource based Policy** gắn với một **AWS Resource**.
  - Cách thực thi quyền của IAM luôn ưu tiên Deny so với Allow, nếu có Deny tương minh (explicit) thì dù có allow ở trên một IAM Policy khác thì vẫn luôn là Deny.
  - IAM Role:
    - Cho phép xác định một tập hợp quyền truy cập vào các tài nguyên (thông qua việc gắn IAM Policy vào IAM Role). IAM Role không có thông tin đăng nhập (credentials) để truy cập vào management console hay AWS CLI/SDK.
    - Khi một IAM User muốn sử dụng IAM Role, IAM User sẽ có thể assume (đảm nhận) IAM Role, ngay sau khi assume role, quyền hạn tại user sẽ được thay bằng quyền đang được cấp cho IAM Role. Ngoài ra, thông tin xác thực bảo mật tạm thời sẽ được cấp cho IAM User hoặc một AWS Service để có thể truy cập tới các dịch vụ trên AWS. Việc assume role sẽ làm việc với **AWS STS - Security Token Service** giúp tạo ra các thông tin chứng thực tạm (tương tự như access key).
    - Để một user có thể sử dụng IAM Role, IAM Role sẽ được gắn với một resource base IAM policy, hay còn gọi là **IAM Role trust policy**, quy định xem ai có thể sử dụng IAM Role.
    - IAM Role thường được dùng trong trường hợp để đảm bảo nguyên tắc cấp quyền thông qua việc cấp quyền cho các AWS account khác truy cập tài nguyên của AWS account hiện tại.
    - Ngoài việc sử dụng cho IAM User, IAM Role còn được sử dụng để cấp quyền truy cập các resource của AWS cho các AWS Service.
    - Trường hợp sử dụng thường thấy là dùng IAM Role cấp quyền cho các ứng dụng chạy bên trong dịch vụ tính toán (EC2).

- Amazon Cognito - AWS Organization:

  - Amazon Cognito: Là dịch vụ quản lý bộ AWS có chức năng xác thực, cấp phép và quản lý người dùng cho các ứng dụng web và di động. Người dùng có thể đăng nhập trực tiếp bằng tên người dùng và mật khẩu hoặc thông qua một bên thứ ba như Facebook, Amazon hoặc Google.
  - Hai thành phần chính của Amazon Cognito là **User Pool** và **Identity Pool**.
    - **User Pool** là nơi lưu trữ thông tin người dùng và cung cấp các tùy chọn đăng ký và đăng nhập cho người dùng ứng dụng.
    - **Identity Pool** cấp cho người dùng quyền truy cập vào các dịch vụ AWS khác.
  - AWS Organization: AWS Organizations giúp quản lý và điều hành tập trung môi trường gồm nhiều AWS account.
    - AWS Organizations có thể tạo các tài khoản AWS mới, phân bổ tài nguyên, sắp xếp các AWS account theo OU **(Organization Unit)**, đồng thời đơn giản việc thanh toán toàn tập trung **(consolidated billing)**.
    - AWS Organization có thể áp dụng chính sách kiểm soát dịch vụ (Service Control Policies) lên các OU hoặc các AWS account, SCP thiết lập giới hạn quyền tại các IAM User hoặc IAM role trong OU hoặc AWS account đó.
    - SCP cung cấp khả năng thiết lập deny-based policy.

- AWS Identity Center (SSO):

  - AWS Identity Center giúp quản lý quyền truy cập tới AWS account và các ứng dụng bên ngoài.
  - Identity source có thể là tên trong AWS Identity Center hoặc được liên kết với Active Directory. (AWS Managed Microsoft AD, On-premises Microsoft AD thông qua Two way trust hoặc AD Connector)
  - Permission Set xác định khả năng truy cập của User và Group tới các tài khoản AWS trong AWS Organization. Các quyền được lưu trữ trong AWS Identity Center và được cung cấp cho tài khoản AWS dưới dạng IAM roles. Bạn có thể gán nhiều quyền cho một User.

- Amazon Key Management Service(KMS):

  - AWS Key Management Service giúp tạo và quản lý các encryption key, phục vụ cho mục đích encrypt/decrypt dữ liệu trên AWS.
  - Encryption key luôn nằm trong AWS KMS, đảm bảo tiêu chuẩn FIPS 140-2.
  - CMK ( Customer Managed Key ) đóng vai trò là tài nguyên chính trong AWS KMS. CMK có thể có kích thước tới 4KB. Tuy nhiên thông thường, chúng ta chỉ sử dụng CMK cho mục đích tạo, mã hóa và giải mã Data Key - loại khóa được dùng bên ngoài AWS KMS để mã hóa dữ liệu.

- AWS Security Hub: Là dịch vụ cho phép chuẩn hóa thực hiện kiểm tra bảo mật dựa trên các tiêu chuẩn và best practices.

  - Security Hub chạy liên tục, kiểm tra các dịch vụ trong tài khoản AWS và kiểm tra bảo mật dựa trên các best practice của AWS và tiêu chuẩn ngành (VD: PCI DSS).
  - Security Hub cung cấp kết quả kiểm tra dưới dạng điểm số và giúp chuẩn hóa xác định các tài khoản và tài nguyên có thể cần được chú ý.
