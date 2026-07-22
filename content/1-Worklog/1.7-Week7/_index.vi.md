---
title: "Worklog Tuần 7"
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

- Các dịch vụ cơ sở dữ liệu trên AWS.
- Amazon RDS & Amazon Aurora.
- Redshift - Elasticache.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                              | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                |
| --- | -------------------------------------- | ------------ | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Khái niệm cơ sở dữ liệu              | 15/06/2026   | 16/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 2   | - Amazon RDS & Amazon Aurora           | 18/06/2026   | 18/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon ElastiCache & Amazon RedShift | 20/06/2026   | 21/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Kết quả đạt được tuần 7:

- Khái niệm cơ sở dữ liệu:

  - **Cơ sở dữ liệu(Database)** là nơi lưu trữ thông tin để xử lý và phân tích dữ liệu từ nhiều nguồn khác nhau, hỗ trợ người dùng trong việc tra cứu thông tin một cách nhanh chóng và hiệu quả khi cần thiết, đặc biệt là trong các hệ thống phức tạp hoặc khi người dùng cần dịch khac nhau.
  - **Phiên(Session)** là khoảng thời gian từ thời gian bắt đầu kết nối vào hệ thống CSDL (time start) đến thời gian kết thúc (time end) là khoảng thời gian bản ngắt kết nối.
  - **Khóa chính(Primary Key)** là một cột (hoặc kết hợp các cột) được chỉ định để xác định duy nhất mỗi bản ghi trong bảng.
  - **Khóa ngoại(Foreign Key)** là một cột (hoặc nhóm cột) trong bảng có thể quan hệ với cột trong một bảng khác, giúp tạo ra mối tham chiếu giữa các bảng và thiết lập liên kết giữa chúng.
  - **Chỉ mục(Index)** cơ sở dữ liệu là một cấu trúc dữ liệu được xây dựng để cải thiện tốc độ của các hoạt động truy xuất dữ liệu (read) trên bảng cơ sở dữ liệu, tuy nhiên sẽ làm tăng chi phí ghi thêm (write) và không gian lưu trữ (storage) để duy trì cấu trúc dữ liệu chỉ mục. Các chỉ mục được sử dụng để định vị dữ liệu một cách nhanh chóng hơn khi truy cập bảng cơ sở dữ liệu. Chỉ mục có thể được tạo ra trên bảng cơ sở dữ liệu theo một cách khác nhau.
  - **Phân vùng(Partitions)** là quá trình chia nhỏ dữ liệu trong các bảng lớn được chia ra để lưu trữ dữ liệu trên nhiều phân vùng khác nhau. Bằng cách này, các truy vấn có thể truy cập một phân vùng dữ liệu thay vì toàn bộ dữ liệu, giúp tăng tốc độ truy vấn và giảm chi phí khi xử lý các truy vấn lớn trên cơ sở dữ liệu.
  - **Kế hoạch thực thi truy vấn - Kế hoạch truy vấn(Execution Plan - Query Plan)** là một chương trình hoặc các bước được sử dụng để thực thi truy vấn trong hệ quản trị cơ sở dữ liệu quan hệ SQL. Khi một truy vấn được gửi đến cơ sở dữ liệu, trình tối ưu hóa sẽ chọn phương án tốt nhất để thực thi truy vấn dựa trên các điều kiện và các yếu tố khác nhau, từ đó tối ưu hóa truy vấn và tối ưu hóa hiệu quả truy vấn.
  - **Database Log** là một phần quan trọng trong thiết kế giải pháp cơ sở dữ liệu giúp bảo vệ tính toàn vẹn dữ liệu và phục hồi dữ liệu trong trường hợp xảy ra sự cố. Tất cả các thay đổi dữ liệu được ghi lại trong các bản ghi (log file) được liên kết với cơ sở dữ liệu.
  - **Bộ đệm(Buffers)** là một vùng lưu trữ tạm thời trong bộ nhớ máy tính. Nó cho phép lưu trữ dữ liệu tạm thời khi di chuyển dữ liệu từ nơi này sang nơi khác. Bộ đệm cơ sở dữ liệu lưu trữ các bản sao của các khối dữ liệu. Thông thường bộ đệm được sử dụng để tăng tốc độ truy xuất (đọc dữ liệu từ buffer) hoặc giảm thời gian ghi dữ liệu vào buffer, sau đó buffer sẽ đồng bộ hóa với cơ sở dữ liệu.
  - **NOSQL Cơ sở** dữ liệu NoSQL (hay còn gọi là "Không chỉ SQL") thường không phải dạng bảng như cơ sở dữ liệu quan hệ. Cơ sở dữ liệu NoSQL có nhiều loại dựa trên mô hình dữ liệu của chúng.
  - **Các loại chính là tài liệu (document)**, khóa-giá trị (key-value), wide-column và graph - biểu đồ. Chúng cung cấp các lược đồ linh hoạt và mô hình quy mô được điều chỉnh phù hợp với lượng dữ liệu lớn và tải lượng người dùng cao.
  - **OLTP - Online Transaction Processing** - Hệ thống OLTP nhằm bảo vệ và duy trì giao dịch trong cơ sở dữ liệu. Mỗi giao dịch liên quan đến các bản ghi cơ sở dữ liệu sẽ được tạo thành từ nhiều trường hợp khác nhau.
  - **Vì độ bảo mật** hoạt động nhanh và trong thời gian thực, hệ thống OLTP thường được sử dụng để xử lý các giao dịch trong cơ sở dữ liệu OLTP thường xuyên. Nếu một giao dịch không thành công, logic hệ thống sẽ đảm bảo tính toàn vẹn dữ liệu.
  - **OLAP - Online Analytical Processing** - OLAP áp dụng các truy vấn phục vụ cho việc phân tích dữ liệu lịch sử, thường được sử dụng để tổng hợp dữ liệu OLTP theo các nguồn khác, chẳng hạn như các báo cáo chi tiết, phân tích và các dự đoán theo thời gian. Trong OLAP, trong một khoảng thời gian, các truy vấn phục vụ này thường liên quan đến một hệ thống lớn hơn.
  - **Vì độ bảo mật** hiệu suất trong việc phân tích dữ liệu lịch sử, cơ sở dữ liệu (data warehouse) OLAP cung cấp các khả năng phân tích nâng cao để sử dụng các công cụ báo cáo hoặc các công cụ phân tích khác nhau để xử lý dữ liệu trong kho dữ liệu và đưa ra các dự đoán chính xác hơn.

- Amazon RDS & Amazon Aurora:

  - Amazon RDS: Là cơ sở dữ liệu được quản lý trên AWS, cung cấp chi phí thấp và khả năng mở rộng RDBMS, không thể truy cập vào các cơ sở dữ liệu cũ. Bao gồm **Aurora, MySQL, Postgres SQL, MSSQL, Oracle, Maria**.
  - Amazon RDS cung cấp các tính năng:
    1. Tự động sao lưu. (các log của database – max 35 ngày)
    2. Tạo bản sao chi đọc (Read Replica) phục vụ cho các Read workload, (reporting).
    3. Read Replica có thể được gắn ra khỏi chuỗi thanht toán Primary node.
    4. Chạy với cơ chế tự động failover, Primary/Standby, hay còn gọi là cơ chế Multi-AZ.
    5. RDS thường được sử dụng cho các ứng dụng OLTP.
    6. RDS cung cấp tính năng mã hóa dữ liệu at rest và in transit.
    7. RDS cung cấp bảo vệ bổ tĩnh mạng tường lửa cùng với EC2 (Security Group và NACL).
    8. Thay đổi quy mô (Thay đổi instance size).
    9. Tự động tăng dung lượng lưu trữ (Storage Auto scaling).
  - Amazon Aurora: Amazon Aurora là một công cụ cơ sở dữ liệu quan hệ được tối ưu hóa để hỗ trợ lưu trữ dưới dạng đám mây, với các bản ghi song song trên cơ sở dữ liệu. Nó được xây dựng dựa trên các hệ thống RDBMS với hai lựa chọn là MySQL và PostgreSQL. Aurora cung cấp một RDBMS dựa trên Amazon RDS với nhiều cải tiến nâng cao hơn so với RDS.
  - Ngoại ra Amazon Aurora cung cấp các tính năng:
    1. **Back Track** – Phục hồi lại DB về thời điểm trước đó.
    2. **Clone** – Tạo bản sao.
    3. **Global Database** – 1 Master và Multi Read nằm ở các Region khác nhau.
    4. **Multi Master** – Multi Master database.

- Amazon ElastiCache & Amazon RedShift:

  - **Amazon RedShift**: Là dịch vụ Data warehouse được quản lý bởi AWS. Lợi thế Postgre SQL nhưng được tối ưu cho OLAP.
  - Redshift sử dụng kiến trúc Massively-Parallel Processing (MPP) Database, dữ liệu được chia (partition) và lưu trữ tại các Compute node (bao gồm cả compute/storage). Leader node nhận vai trò điều phối và tổng hợp truy vấn.
  - Redshift lưu trữ dữ liệu dưới dạng columnar storage, phù hợp cho ứng dụng OLAP.
  - Redshift sử dụng SQL với các driver **JDBC, ODBC** thông dụng.
  - Hỗ trợ nâng tính năng để tối ưu hóa chi phí (Transient cluster, Redshift spectrum).
  - **Amazon ElastiCache**: Là một dịch vụ được quản lý bởi AWS giúp tạo ra các cluster caching engines. Hiện ElastiCache hỗ trợ 2 engine là Redis và Memcached.
  - Amazon ElastiCache sẽ trách nhiệm phát hiện và thay thế các node bị failed.
  - Amazon ElastiCache thường được đặt trực tiếp trên CSDL để cache dữ liệu, giám sát cho phép CSDL.
  - Với các workload với ứng dụng mới, ưu tiên sử dụng Redis. (tính năng đa dạng, hiệu năng tốt)
  - Sử dụng ElastiCache yêu cầu phải viết lại quy trình caching logic trên ứng dụng.
