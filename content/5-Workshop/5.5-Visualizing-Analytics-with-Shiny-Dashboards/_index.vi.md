---
title: "Trực quan hóa phân tích với các bảng điều khiển Shiny"
weight: 55
chapter: false
pre: " <b> 5.5. </b> "
---

## 5.5.1 Thông tin môi trường

- OS: **Ubuntu 22.04 (Jammy)** – Chạy trên EC2 instance trong private subnet  
- PostgreSQL: **v18** (được cài đặt qua repo `apt.postgresql.org`)  
- Shiny Server: File binary `.deb` từ RStudio (Posit)  
- Service User: `shiny`  
- Application Path: `/srv/shiny-server/sbw_dashboard/app.R`

---

## 5.5.2 Cài các package hệ thống (system libs)

Lưu ý: Cần kích hoạt NAT Gateway trước khi tiến hành tải các package hệ thống.
Kết nối vào EC2 thông qua **SSM Session Manager** (hoặc dùng SSH tạm thời, nếu được cấu hình), rồi thực thi:

```bash
# 1) Cập nhật danh sách package
sudo apt-get update

# 2) Cài đặt R (nếu hệ thống chưa có)
sudo apt-get install -y r-base

# 3) Cài đặt Postgres client & development headers (cần thiết cho RPostgres)
#    Sử dụng postgresql-server-dev-18 nếu DB của bạn là PG 18
#    (nếu dùng version khác, thay đổi 18 -> 14, 15, v.v. cho phù hợp)
sudo apt-get install -y postgresql-client-18 postgresql-server-dev-18

# 4) Cài đặt libpq và libssl (bắt buộc để compile RPostgres)
sudo apt-get install -y libpq-dev libssl-dev

# 5) (Trường hợp Shiny Server chưa được cài đặt)
#    Bất kể phương pháp cài đặt là gì, hãy lưu ý các đường dẫn sau:
#    - shiny-server service: /etc/systemd/system/shiny-server.service
#    - thư mục chứa app: /srv/shiny-server/
#    - user thực thi: shiny
```

Kiểm tra để chắc chắn `libpq` và các dev headers đã hiện diện:

```bash
dpkg -l | grep -E 'libpq-dev|postgresql-server-dev' || echo "MISSING_LIBS"
ls -l /usr/include/postgresql/libpq-fe.h || echo "NO_LIBPQ_HEADER"
```

Nếu **không hiển thị lỗi nào** → Quá trình cài đặt đã thành công.

---

## 5.5.3 Cấu hình thư mục R libraries cho user `shiny`

Để Shiny Server có thể nạp các R package, chúng ta sẽ cài đặt chúng dưới quyền của user `shiny` tại thư mục:

- `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`

Chạy các lệnh sau:

```bash
sudo -u shiny R --vanilla <<'EOF'
# Tạo thư mục library cho user shiny nếu nó chưa tồn tại
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)

# Chèn R_LIBS_USER lên vị trí đầu tiên của .libPaths()
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

q("no")
EOF
```

Bạn sẽ thấy output `LIBPATHS` với dòng đầu tiên trỏ tới `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`.

---

## 5.5.4 Cài các R package cần thiết

Dashboard yêu cầu các package sau:

- `shiny`
- `DBI`
- `RPostgres`
- `dplyr`
- `ggplot2`
- `lubridate`
- `pool`

Cài đặt tất cả dưới quyền user `shiny`:

```bash
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)

q("no")
EOF
```

💡 **Xử lý lỗi liên quan tới `libpq-fe.h` hoặc `libpq`:**

1. Xác minh lại rằng các gói `libpq-dev`, `postgresql-server-dev-XX`, và `libssl-dev` đã được cài đặt đầy đủ.  
2. Chạy lại lệnh `install.packages("RPostgres", ...)` sau khi đã bổ sung các thư viện thiếu.  

Xác nhận các package có thể được nạp thành công:

```bash
sudo -u shiny R --vanilla <<'EOF'
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

library(shiny)
library(DBI)
library(RPostgres)
library(dplyr)
library(ggplot2)
library(lubridate)
library(pool)

cat("All packages loaded OK
")
q("no")
EOF
```

Nếu **không phát sinh error** → môi trường R đã sẵn sàng.

---

## 5.5.5 Triển khai Shiny app

### 5.5.5.1 Tạo thư mục app và copy code

```bash
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
```

Tạo file ứng dụng (hoặc ghi đè nếu đã có):

```bash
sudo nano /srv/shiny-server/sbw_dashboard/app.R
# DÁN TOÀN BỘ CODE CỦA app.R (sử dụng phiên bản hoàn chỉnh của bạn)
# Nhấn Ctrl+O, Enter, rồi Ctrl+X để lưu và thoát
```

Kiểm tra lại phân quyền để đảm bảo tính chính xác:

```bash
sudo chown shiny:shiny /srv/shiny-server/sbw_dashboard/app.R
sudo chmod 644 /srv/shiny-server/sbw_dashboard/app.R
```

### 5.5.5.2 Restart Shiny Server

```bash
sudo systemctl restart shiny-server
sudo systemctl status shiny-server
```

---

## 5.5.6 Kiểm tra app từ EC2 (local)

Thông qua session SSM trên EC2 (giao diện terminal):

```bash
# Kiểm tra khả năng truy cập trang welcome của Shiny
curl -m 5  -sS -o /dev/null -w "WELCOME HTTP %{http_code}
"   http://127.0.0.1:3838/

# Kiểm tra ứng dụng SBW dashboard
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```

Nếu kết quả trả về là `DASHBOARD HTTP 200` → app đang hoạt động ổn định.

Nếu nhận được lỗi `500`:

```bash
LATEST=$(ls -1t /var/log/shiny-server/sbw_dashboard-shiny-*.log | head -n 1)
echo "LATEST=$LATEST"
sudo tail -n 100 "$LATEST"
```

Kiểm tra error log để tìm nguyên nhân và khắc phục.

---

## 5.5.7 Truy cập dashboard từ máy local

Do EC2 instance nằm trong **private subnet**, việc truy cập phải được thực hiện qua **SSM port forwarding**:

```bash
# Ví dụ sử dụng AWS CLI v2 trên máy tính local của bạn:
aws ssm start-session   --target <INSTANCE_ID_PRIVATE>   --document-name AWS-StartPortForwardingSessionToRemoteHost   --parameters '{"host":["127.0.0.1"],"portNumber":["3838"],"localPortNumber":["3838"]}'
```

Khi session đã mở, hãy khởi động trình duyệt trên máy local và truy cập vào:

```text
http://127.0.0.1:3838/sbw_dashboard/
```

Giao diện dashboard sẽ xuất hiện với các thành phần như:

- **Các KPI cards** (tổng hợp events, users, sessions, v.v.)  
- Các biểu đồ về **events over time**, **event mix**, **events by login state**  
- Tab **Products & Raw sample** (hỗ trợ phân trang, hiển thị dữ liệu mới nhất, và tự động làm mới mỗi 10s – phụ thuộc vào logic code app)

---

## 5.5.8 Tóm tắt nhanh các lệnh quan trọng

```bash
# Cài đặt các thư viện hệ thống cần thiết
sudo apt-get update
sudo apt-get install -y r-base postgresql-client-18 postgresql-server-dev-18 libpq-dev libssl-dev

# Cài đặt R packages dưới quyền user shiny
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)
q("no")
EOF

# Triển khai mã nguồn app
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo nano /srv/shiny-server/sbw_dashboard/app.R   # Dán mã nguồn tại đây
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
sudo systemctl restart shiny-server

# Xác nhận dashboard hoạt động
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```
