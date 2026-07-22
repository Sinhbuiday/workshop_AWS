---
title: "Triển khai thu nhận Clickstream"
weight: 53
chapter: false
pre: " <b> 5.3. </b> "
---

## 5.3.1 Tổng quan luồng thu nhận

Luồng dữ liệu mức cao:

1. Người dùng tương tác với frontend Next.js (`ClickSteam.NextJS`) được host trên Amplify.  
2. JavaScript phía frontend đóng gói metadata clickstream (page/user/session/product context) thành payload JSON.  
3. Trình duyệt gửi request `POST` tới `clickstream-http-api` tại route `POST /clickstream`.  
4. API Gateway định tuyến request tới `clickstream-lambda-ingest`.  
5. Lambda Ingest bổ sung metadata `_ingest` và ghi mỗi sự kiện thành một file JSON vào:
   - `s3://clickstream-s3-ingest/events/YYYY/MM/DD/HH/event-<uuid>.json` (phân vùng theo giờ UTC)  

Thiết kế duy trì tính **stateless** và **append-only**, phù hợp cho ETL theo lô phía downstream và các thao tác chèn idempotent.

---

## 5.3.2 Thiết kế S3 Bucket

Hai bucket liên quan:

1. **Bucket Asset** — `clickstream-s3-sbw`  
   - Lưu trữ tài nguyên website: ảnh sản phẩm, file tĩnh.  
   - Không dùng để lưu clickstream events.

2. **Bucket Clickstream RAW** — `clickstream-s3-ingest`  
   - Chỉ lưu raw clickstream JSON events.  
   - Phân vùng theo giờ UTC: `events/YYYY/MM/DD/HH/`  
   - Quy tắc đặt tên file: `event-<uuid>.json`  

Phân vùng theo giờ giúp batch ETL hiệu quả hơn (ví dụ: chỉ xử lý giờ trước hoặc một prefix ngày/giờ cụ thể).



---

## 5.3.3 Thiết kế Lambda Ingest — `clickstream-lambda-ingest`

![Lambda Ingest](/images/aws-lambda-clickstream-ingest-config.png)

### Nhiệm vụ

Hàm `clickstream-lambda-ingest`:

- Phân tích payload JSON nhận từ API Gateway.  
- Chỉ bổ sung metadata `_ingest`: `receivedAt`, `sourceIp`, `userAgent`, `method`, `path`, `requestId`, `apiId`, `stage`, `traceId`.  
- Ghi phần thân sự kiện **đúng như client gửi lên** (không tự điền user/session/product phía server) vào S3.

### Quyền IAM

Execution role phải cấp:

- `s3:PutObject` trên: `arn:aws:s3:::clickstream-s3-ingest/events/*`  
- Các API CloudWatch Logs để xuất logs của hàm.

Hàm này không cần quyền đọc.

---

## 5.3.4 API Gateway HTTP API — `clickstream-http-api`

HTTP API cung cấp endpoint HTTPS public cho việc thu nhận sự kiện:

- Route:
  - `POST /clickstream` → Lambda `clickstream-lambda-ingest`  

![Route POST /clickstream](/images/aws-apigw-clickstream-routes.png)

Cấu hình được khuyến nghị:

- Bật **CORS** để frontend Amplify có thể gọi endpoint từ domain của nó.  
- Bật **access log** tới CloudWatch log group phục vụ debug.  
- (Tùy chọn) Gắn **API key** hoặc **Cognito authorizer** để kiểm soát ai được phép gửi events.

---

## 5.3.5 Frontend Clickstream Publisher (Logic)

### Danh tính & tính idempotent
- Sinh `eventId` duy nhất cho mỗi sự kiện (UUID) để đảm bảo chèn idempotent.
- Lưu `clientId` trong `localStorage` (giữ cố định qua các phiên trình duyệt).
- Duy trì `sessionId` trong `sessionStorage` (timeout 30 phút không hoạt động) và theo dõi `isFirstVisit`.

### Metadata người dùng, trang và click
- Thông tin user/auth (khi có): `userId`, `userLoginState`, tùy chọn `identity_source`.
- Thông tin trang/click: `pageUrl`, `referrer`, metadata phần tử HTML khi click (tag/id/role/text/dataset).

### Ngữ cảnh sản phẩm
- Gửi dưới dạng `product.{id,name,category,brand,price,discountPrice,urlPath}`; ETL ánh xạ sang các cột DW `context_product_*`.

### Phạm vi sự kiện
- Tự động theo dõi: `page_view`, `click` toàn cục.
- Sự kiện tùy chỉnh/sản phẩm: `home_view`, `category_view`, `product_view`, `add_to_cart_click`, `remove_from_cart_click`, `wishlist_toggle`, `share_click`, `login_open`, `login_success`, `logout`, `checkout_start`, `checkout_complete`.

### Mapping sự kiện domain 
- `home_view`: kích hoạt khi trang chủ được tải (tracker component).
- `category_view`: kích hoạt khi danh sách danh mục được render (slug/params).
- `product_view`: kích hoạt khi chi tiết sản phẩm được render (bao gồm product context).
- `add_to_cart_click` / `remove_from_cart_click`: handler thêm/xóa sản phẩm khỏi giỏ.
- `wishlist_toggle`: handler nút wishlist.
- `share_click`: handler nút chia sẻ.
- `login_open` / `login_success` / `logout`: các luồng xác thực.
- `checkout_start` / `checkout_complete`: bước bắt đầu checkout và hoàn tất đơn hàng.

### Component & wiring
- `lib/clickstreamClient.ts`: quản lý danh tính/phiên, builder event cơ sở, ghi console log, fire-and-forget `fetch` tới `NEXT_PUBLIC_CLICKSTREAM_ENDPOINT` (biến môi trường bắt buộc).
- `lib/clickstreamEvents.ts`: các domain helper bọc `trackCustom` và xây dựng object ngữ cảnh sản phẩm/giỏ hàng/đơn hàng.
- `contexts/ClickstreamProvider.tsx` + `app/layout.tsx`: kết nối global provider, tự động bắn `page_view`, đăng ký global click listener.
- Tracker components: `HomeTracker.tsx`, `CategoryTracker.tsx`, `ProductViewTracker.tsx` bỏ qua auto page_view và phát sự kiện domain riêng.
- UI đã gắn instrument: `AddToCartButton.tsx`, `FavoriteButton.tsx`, `app/(client)/cart/page.tsx` phát sự kiện thêm/xóa giỏ, wishlist và checkout; các nút được đánh dấu với `global-clickstream-ignore-click` để tránh trùng với global click event.

### Hành vi runtime
- Chỉ chạy phía client (không có side effect SSR).
- Log mọi sự kiện ra console; nếu thiếu endpoint, chuyển sang chế độ dry-run và cảnh báo một lần.
- Lỗi mạng không chặn UI — giao diện người dùng tiếp tục hoạt động bình thường.

## 5.3.6 Chiếu trường (frontend -> S3 -> DW)

| Trường / Khối          | Payload frontend                              | Raw S3 (sau Ingest)                                             | DW (PostgreSQL)                               | Ghi chú                                  |
| ---                    | ---                                           | ---                                                             | ---                                           | ---                                      |
| event_id               | `eventId` sinh ở client                       | giữ nguyên payload                                              | `event_id` (ánh xạ từ eventId; ETL fallback UUID) | Khóa chính, ON CONFLICT DO NOTHING      |
| event_timestamp        | -                                             | - (có LastModified của S3)                                      | `_ingest.receivedAt` > payload > LastModified | ETL suy ra                               |
| event_name             | `eventName`                                   | `eventName`                                                     | `event_name`                                  |                                           |
| page_url               | `pageUrl`                                     | `pageUrl`                                                       | -                                             | Không lưu trong DW                       |
| referrer               | `referrer`                                    | `referrer`                                                      | -                                             | Không lưu trong DW                       |
| user_id                | `userId`                                      | `userId`                                                        | `user_id`                                     |                                           |
| user_login_state       | `userLoginState`                              | `userLoginState`                                                | `user_login_state`                            |                                           |
| identity_source        | tùy chọn                                      | tùy chọn                                                        | `identity_source`                             | Cần frontend/auth cung cấp               |
| client_id              | `clientId`                                    | `clientId`                                                      | `client_id`                                   |                                           |
| session_id             | `sessionId`                                   | `sessionId`                                                     | `session_id`                                  |                                           |
| is_first_visit         | `isFirstVisit`                                | `isFirstVisit`                                                  | `is_first_visit`                              |                                           |
| product id             | `product.id`                                  | `product.id`                                                    | `context_product_id`                          |                                           |
| product name           | `product.name`                                | `product.name`                                                  | `context_product_name`                        |                                           |
| product category       | `product.category`                            | `product.category`                                              | `context_product_category`                    |                                           |
| product brand          | `product.brandName` / `brand?.name` / `brand?.title` / `brand` | `product.brand` hoặc `product.brandName`                        | `context_product_brand`                       | Frontend ánh xạ brand từ các trường DB  |
| product price          | `product.price`                               | `product.price`                                                 | `context_product_price`                       | BIGINT                                   |
| product discount price | `product.discountPrice`                       | `product.discountPrice`                                         | `context_product_discount_price`              | BIGINT                                   |
| product url path       | `product.urlPath`                             | `product.urlPath`                                               | `context_product_url_path`                    |                                           |
| element metadata       | `element.{tag,id,role,text,dataset}`          | `element...`                                                    | -                                             | Không lưu (cần thay đổi schema)         |
| ingest metadata        | -                                             | `_ingest.{receivedAt,sourceIp,userAgent,method,path,requestId,apiId,stage,traceId}` | -                                             | Không lưu; chỉ có khi ETL               |

Mẫu gọi (một event mỗi request):

```ts
await fetch("https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/clickstream", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(eventPayload),
  keepalive: true,
});
```

ETL ánh xạ `eventId -> event_id` (PK với ON CONFLICT DO NOTHING), suy ra `event_timestamp` từ `_ingest.receivedAt` > payload > S3 LastModified, và chèn records vào `clickstream_dw.public.clickstream_events`.

---

## 5.3.6 Kiểm thử & Xác nhận

Để xác minh rằng ingestion đang hoạt động đúng:

1. Dùng UI của app Amplify:
   - Duyệt qua một vài trang sản phẩm  
   - Thêm sản phẩm vào giỏ hàng  
2. Kiểm tra S3 bucket `clickstream-s3-ingest`:
   - Điều hướng tới `events/YYYY/MM/DD/HH/`  
   - Xác nhận các file `event-<uuid>.json` mới đã xuất hiện.  
3. Mở và kiểm tra một file JSON:
   - Xác nhận metadata `_ingest` và các trường product context có mặt đầy đủ.  
4. Xem lại logs:
   - API Gateway access logs  
   - Lambda function logs  
