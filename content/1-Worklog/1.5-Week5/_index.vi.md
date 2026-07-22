---
title: "Worklog Tuần 5"
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

- Amazon Simple Storage Service ( S3 ) - Access Point - Storage Class.
- S3 Static Website & CORS - Control Access - Object Key & Performance - Glacier.
- Snow Family - Storage Gateway - Backup.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                             | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                |
| --- | ----------------------------------------------------- | ------------ | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Làm quen Amazon Simple Storage Service (S3)         | 1/06/2026   | 2/06/2026      |
| 2   | - Amazon Simple Storage Service (S3) - Access Point   | 2/06/2026   | 3/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon Simple Storage Service (S3) - Storage Class  | 3/06/2026   | 4/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - Tìm hiểu S3 Static Website & CORS                   | 4/06/2026   | 5/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Control Access - Object Key & Performance - Glacier | 5/06/2026   | 6/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 6   | - Snow Family - Storage Gateway - Backup              | 6/06/2026   | 7/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Kết quả đạt được tuần 5:

- Tìm hiểu Amazon Simple Storage Service (S3):

  - Amazon S3 là nơi lưu trữ dữ liệu tĩnh, có nghĩa là những dữ liệu đã được tạo ra và không thay đổi sau đó.
  - S3 phù hợp với các loại dữ liệu chỉ ghi một lần và đọc nhiều lần (WORM - Write Once Read Many).
  - Amazon S3 không giới hạn tổng khối lượng dữ liệu lưu trữ.
  - Mỗi đối tượng không được lớn hơn **5 TB**.
  - Theo mặc định, dữ liệu trong Amazon S3 được nhân bản trên 3 AZ trong 1 Region.
  - Amazon S3 có khả năng kích hoạt sự kiện (**trigger event**) cho phép bạn kích hoạt các hành động khi một sự kiện xảy ra, như tải lên hoặc xóa một đối tượng khỏi một vùng lưu trữ cụ thể.
  - Amazon S3 được thiết kế để đạt độ bền (**durability**) 99.99999999% và độ sẵn sàng (**high availability**) 99.99%.
  - Amazon S3 hỗ trợ **multipart upload** để upload các đối tượng lớn lên bucket.
  - Chương trình tạo các S3 bucket để có thể lưu trữ các đối tượng trong Amazon S3.
    - s3.amazonaws.com => s3.amazonaws.com/capture.mp4

- Amazon Simple Storage Service (S3) - Access Point: Amazon S3 Access Point là tính năng cho phép tạo các điểm kết nối (hostname unique) dành cho ứng dụng, người dùng hoặc nhóm.

  - Chương trình có thể cấu hình phân quyền khi tạo mỗi access point.

- Amazon Simple Storage Service (S3) - Storage Class:

  - Amazon S3 chia vùng lưu trữ ra nhiều lớp lưu trữ (storage class) giúp chúng ta tối ưu hóa chi phí.
  - Các cấp lưu trữ (Storage Class của S3):
    - **S3 Standard**: Dữ liệu được truy cập thường xuyên.
    - **S3 Standard IA**: Dữ liệu ít được truy cập thường xuyên.
    - **S3 Intelligent Tiering**: Tự động di chuyển các đối tượng giữa các cấp lưu trữ theo số ngày đổi.
    - **S3 One Zone IA**: Dữ liệu có thể tái tạo, dữ liệu trong vùng, ít được truy cập thường xuyên nhưng cần truy cập nhanh.
    - **Amazon Glacier / Deep Archive**: Lưu trữ dữ liệu ít truy cập.
  - Chức năng cấu hình thời gian lưu trữ dữ liệu (Object Life Cycle Management):
    - Di chuyển dữ liệu trong Amazon S3. Bảng chứa dữ liệu trong các trường hợp sau đây, chuyển dữ liệu trong S3 bucket khi dữ liệu lưu trữ theo thời gian (ngày) đủ điều kiện.

- Amazon Simple Storage Service (S3) - Static Website & CORS:

  - Amazon S3 có tính năng cho phép host các static website (html, media, ...), phù hợp cho **Single Page Application**. (ứng dụng web hoạt trang web tĩnh tác vụ người dùng bằng cách tạo lại trang web hiện tại với dữ liệu mới từ máy chủ web sử dụng javascript hoặc các framework của nó như AngularJS, ReactJS, thay vì phương pháp mặc định cua trình duyệt web tải toàn bộ trang mới).
  - Amazon S3 hỗ trợ **CORS**. **CORS** là một cơ chế cho phép nguồn tài nguyên khác nhau (fonts, Javascript, v.v...) của một trang web có thể được truy vấn từ domain khác với domain của trang đó. CORS là viết tắt của **Cross-origin resource sharing**.

- Amazon Simple Storage Service (S3) - Control Access:

  - Amazon S3 có 2 cơ chế kiểm soát quyền truy cập tới bucket:
    - **S3 Access Control List (ACL)** là một cơ chế kiểm soát truy cập cơ bản. Tuy nhiên, nếu bạn đã sử dụng S3 ACL và thấy không cần thay đổi, ACL S3 được gắn bucket và object của S3. Không xác định tại khoảnh khắc nào AWS sẽ dùng cơ chế này để cấp quyền truy cập thông qua loại quyền truy cập.
    - **S3 Bucket Policy và IAM policy** xác định quyền cấp đối tượng bằng cách cung cấp các điều kiện áp dụng cho các đối tượng trong phần Resource trong policy này. Cấu hình này áp dụng cho các đối tượng trong bucket. Việc này cũng cấp quyền điều chỉnh các điều kiện trong bucket (trái ngược với ACL S3) giúp bạn điều chỉnh quyền truy cập.

- Amazon Simple Storage Service (S3) - Object Key & Performance:
  - Mối object trong S3 được ngang hàng, không phân cấp (hierarchy) và được gán 1 object key. Ví dụ: **/image/sample.jpg, sample.jpg**.
  - Sâu bên trong S3 chia ra các Partitions, Partitions sẽ được chia ra tùy theo kích thước request tăng cao dựa trên số lượng S3 object keys lớn. (lắm chấm để điều khiển object trong partition).
  - S3 lưu trữ key map (key map cung cấp chia ra nguồn partition dựa trên hash bộ prefix – tiền tố của object key).
  - Để tối ưu S3 performance có thể dùng random prefix **(/fscd/img/sample.jpg thay vì /img/sample.jpg)**.
  - Mục tiêu của việc làm này là khiến S3 lưu trữ các object trên nhiều partitions nhất có thể vi performance của S3 dựa trên số lượng partitions.
- Amazon Simple Storage Service (S3) - Glacier:

  - Amazon S3 Glacier là lựa chọn lưu trữ với chi phí thấp, phù hợp với dữ liệu không cần truy suất thường xuyên, lưu trữ lâu dài. Nếu bạn cần truy cập nhanh chóng hoặc thường xuyên, hãy chọn Amazon S3.
  - Khi lưu trữ dữ liệu trong Amazon S3 Glacier, bạn không thể truy xuất dữ liệu trực tiếp mà phải đưa (retrieve) dữ liệu về từ một S3 Bucket.
  - Có ba tùy chọn để truy xuất dữ liệu với thời gian truy cập khác nhau:
    - **Truy xuất Nhanh (Expedited)**: Hoàn tất trong vòng 1 - 5 phút.
    - **Truy xuất Tiêu chuẩn (Standard)**: Hoàn tất trong vòng 3 - 5 giờ.
    - **Truy xuất Hàng loạt (Bulk)**: Hoàn tất trong vòng 5 - 12 giờ.

- Snow Family:
  - Snowball:
    - Dịch vụ hỗ trợ migrate dữ liệu từ môi trường on-premise tới AWS ở quy mô lên đến PetaByte (PB). Mỗi Snowball có thể chứa tối đa 80 TeraByte (TB).
    - Snowball sẽ được ship tới các AWS region mà chúng ta lựa chọn để lưu trữ dữ liệu vào trong S3 hoặc Glacier.
    - Chương trình sử dụng Snowball Client tại máy local để thu thập dữ liệu, rồi chuyển dữ liệu qua transfer.
  - Snowball Edge:
    - Dịch vụ hỗ trợ migrate dữ liệu từ môi trường on-premise tới AWS ở quy mô lên đến PetaByte (PB). Mỗi Snowball Edge có thể chứa tối đa 100 TeraByte (TB).
    - Snowball Edge sẽ được ship tới các AWS region mà chúng ta lựa chọn để lưu trữ dữ liệu vào trong Glacier hoặc S3.
    - Chương trình sử dụng Snowball Client tại máy local để thu thập dữ liệu, rồi chuyển dữ liệu qua transfer.
    - Snowball Edge là thiết bị được sử dụng để sao lưu dữ liệu từ các tài nguyên trong toàn bộ hệ thống khi import vào thiết bị.
  - Snowmobile:
    - Dịch vụ hỗ trợ migrate dữ liệu từ môi trường on-premise tới AWS ở quy mô lên đến 100 PByte. Mỗi Snowmobile có thể chứa tối đa 100 PB.
    - Snowmobile sẽ được vận chuyển tới AWS region mà chúng ta lựa chọn để lưu trữ dữ liệu vào trong dịch vụ, với lựa chọn bao gồm S3 hoặc Glacier.
- Storage Gateway:
  - AWS Storage Gateway là giải pháp lưu trữ Hybrid, kết hợp dung lượng lưu trữ trên AWS với dung lượng lưu trữ tại chỗ (on-premise).
  - Chương trình được thiết kế để dùng quy mô qua gia tăng hợp lý với các dịch vụ lưu trữ trên cloud để giúp lưu trữ các dữ liệu lớn trong thời gian yêu cầu lưu trữ lâu.
  - AWS Storage Gateway hỗ trợ ba phương thức lưu trữ chính: tập tin, ổ đĩa và băng từ:
    - **Cổng kết nối tập tin (File Gateway)** cho phép bạn lưu trữ và truy xuất đối tượng trong Amazon S3 bằng cách sử dụng giao thức tệp NFS hoặc SMB. Dữ liệu được ghi thường qua cổng kết nối tập tin này sẽ được truy cập trực tiếp trong S3.
    - **Cổng kết nối ổ đĩa (Volume Gateway)** cung cấp lưu trữ bằng cách sử dụng giao thức iSCSI. Dữ liệu trong ổ đĩa được lưu trữ trong Amazon S3. Để truy cập ổ đĩa iSCSI trong AWS, bạn có thể tạo EBS snapshot (tùy chọn bản AWS Backup) để tạo ra thành EBS Volumes.
    - **Cổng kết nối băng từ (Tape Gateway)** cung cấp cách sử dụng sao lưu dữ liệu bằng cách sử dụng giao thức VTL (iSCSI), các tape drive ảo và tape ảo. Dữ liệu tape ảo được lưu trữ trong Amazon S3 hoặc Glacier.
- Backup:
  - AWS Backup là một dịch vụ quản lý các tác vụ sao lưu. Chúng ta có thể cấu hình và lập lịch (backup schedule), thời gian lưu trữ (backup retention) và giám sát hoạt động sao lưu cho các tài nguyên trên AWS bao gồm:
    - Amazon EBS
    - Amazon EC2
    - Cơ sở dữ liệu Amazon RDS
    - Cơ sở dữ liệu DynamoDB
    - Amazon EFS
    - AWS Storage Gateway volumes
