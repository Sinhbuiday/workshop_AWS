---
title: "Implementing Clickstream Ingestion"
weight: 53
chapter: false
pre: " <b> 5.3. </b> "
---

## 5.3.1 Ingestion Flow Overview

High-level data flow:

1. A user interacts with the Next.js frontend (`ClickSteam.NextJS`) hosted on Amplify.  
2. The frontend JavaScript bundles clickstream metadata (page/user/session/product context) into a JSON payload.  
3. The browser sends a `POST` request to `clickstream-http-api` at the route `POST /clickstream`.  
4. API Gateway routes the request to `clickstream-lambda-ingest`.  
5. Lambda Ingest appends `_ingest` metadata and writes one JSON file per event to:
   - `s3://clickstream-s3-ingest/events/YYYY/MM/DD/HH/event-<uuid>.json` (UTC hour partition)  

The design remains **stateless** and **append-only**, making it well-suited for downstream batch ETL and idempotent inserts.

---

## 5.3.2 S3 Bucket Design

Two buckets are involved:

1. **Asset Bucket** — `clickstream-s3-sbw`  
   - Stores website assets: product images, static files.  
   - Not used for clickstream event storage.

2. **RAW Clickstream Bucket** — `clickstream-s3-ingest`  
   - Stores only raw clickstream JSON events.  
   - Partitioned by UTC hour: `events/YYYY/MM/DD/HH/`  
   - File naming convention: `event-<uuid>.json`  

Hour-based partitioning makes batch ETL more efficient (e.g., process the previous hour or a specific day/hour prefix).



---

## 5.3.3 Lambda Ingest Design — `clickstream-lambda-ingest`

![Lambda Ingest](/images/aws-lambda-clickstream-ingest-config.png)

### Responsibilities

The `clickstream-lambda-ingest` function:

- Parses the incoming JSON payload received from API Gateway.  
- Appends `_ingest` metadata only: `receivedAt`, `sourceIp`, `userAgent`, `method`, `path`, `requestId`, `apiId`, `stage`, `traceId`.  
- Writes the event body **exactly as sent by the client** (no server-side filling of user/session/product fields) to S3.

### IAM Permissions

The execution role must grant:

- `s3:PutObject` on: `arn:aws:s3:::clickstream-s3-ingest/events/*`  
- CloudWatch Logs APIs to emit function logs.

No read permissions are needed for this function.

---

## 5.3.4 API Gateway HTTP API — `clickstream-http-api`

The HTTP API exposes a public HTTPS endpoint for event ingestion:

- Route:
  - `POST /clickstream` → Lambda `clickstream-lambda-ingest`  

![Route POST /clickstream](/images/aws-apigw-clickstream-routes.png)

Recommended settings:

- Enable **CORS** so the Amplify frontend can call the endpoint from its own domain.  
- Enable **access logs** to a CloudWatch log group for debugging.  
- (Optional) Attach an **API key** or **Cognito authorizer** to restrict who can submit events.

---

## 5.3.5 Frontend Clickstream Publisher (Logic)

### Identity & idempotency
- Generates a unique `eventId` per event (UUID) to guarantee idempotent inserts.
- Persists `clientId` in `localStorage` (sticky across browser sessions).
- Maintains `sessionId` in `sessionStorage` (30-minute idle timeout) and tracks `isFirstVisit`.

### User, page, and click metadata
- User/auth context (when available): `userId`, `userLoginState`, optional `identity_source`.
- Page/click context: `pageUrl`, `referrer`, element metadata for clicks (tag/id/role/text/dataset).

### Product context
- Sent as `product.{id,name,category,brand,price,discountPrice,urlPath}`; ETL maps these to the DW `context_product_*` columns.

### Event coverage
- Auto-tracked: `page_view`, global `click`.
- Custom/product events: `home_view`, `category_view`, `product_view`, `add_to_cart_click`, `remove_from_cart_click`, `wishlist_toggle`, `share_click`, `login_open`, `login_success`, `logout`, `checkout_start`, `checkout_complete`.

### Domain event wiring 
- `home_view`: fires on home page load (tracker component).
- `category_view`: fires on category listing render (slug/params).
- `product_view`: fires on product detail render (includes product context).
- `add_to_cart_click` / `remove_from_cart_click`: cart add/remove action handlers.
- `wishlist_toggle`: wishlist button handler.
- `share_click`: share button handler.
- `login_open` / `login_success` / `logout`: authentication flows.
- `checkout_start` / `checkout_complete`: checkout entry and order success flows.

### Components & wiring
- `lib/clickstreamClient.ts`: identity/session management, base event builders, console logging, fire-and-forget `fetch` to `NEXT_PUBLIC_CLICKSTREAM_ENDPOINT` (required env var).
- `lib/clickstreamEvents.ts`: domain helpers that wrap `trackCustom` and build product/cart/order context objects.
- `contexts/ClickstreamProvider.tsx` + `app/layout.tsx`: wires the global provider, auto-fires `page_view`, and registers the global click listener.
- Tracker components: `HomeTracker.tsx`, `CategoryTracker.tsx`, `ProductViewTracker.tsx` suppress the auto page_view and emit domain-specific events.
- Instrumented UI: `AddToCartButton.tsx`, `FavoriteButton.tsx`, `app/(client)/cart/page.tsx` emit add/remove cart, wishlist, and checkout events; buttons are marked with `global-clickstream-ignore-click` to prevent duplicate global click events.

### Runtime behavior
- Client-side only (no SSR side effects).
- Logs every event to the console; if the endpoint is missing, runs in dry-run mode and warns once.
- Network errors are non-blocking — the UI continues to function normally.

## 5.3.6 Field projection (frontend -> S3 -> DW)

| Field / Block          | Frontend payload                              | Raw S3 (after Ingest)                                           | DW (PostgreSQL)                               | Notes                                   |
| ---                    | ---                                           | ---                                                             | ---                                           | ---                                     |
| event_id               | `eventId` generated on client                 | same as payload                                                 | `event_id` (maps from eventId; ETL fallback UUID) | Primary key, ON CONFLICT DO NOTHING     |
| event_timestamp        | -                                             | - (S3 LastModified exists)                                      | `_ingest.receivedAt` > payload > LastModified | Derived by ETL                          |
| event_name             | `eventName`                                   | `eventName`                                                     | `event_name`                                  |                                           |
| page_url               | `pageUrl`                                     | `pageUrl`                                                       | -                                             | Not stored in DW                        |
| referrer               | `referrer`                                    | `referrer`                                                      | -                                             | Not stored in DW                        |
| user_id                | `userId`                                      | `userId`                                                        | `user_id`                                     |                                           |
| user_login_state       | `userLoginState`                              | `userLoginState`                                                | `user_login_state`                            |                                           |
| identity_source        | optional                                      | optional                                                        | `identity_source`                             | Needs frontend/auth to populate         |
| client_id              | `clientId`                                    | `clientId`                                                      | `client_id`                                   |                                           |
| session_id             | `sessionId`                                   | `sessionId`                                                     | `session_id`                                  |                                           |
| is_first_visit         | `isFirstVisit`                                | `isFirstVisit`                                                  | `is_first_visit`                              |                                           |
| product id             | `product.id`                                  | `product.id`                                                    | `context_product_id`                          |                                           |
| product name           | `product.name`                                | `product.name`                                                  | `context_product_name`                        |                                           |
| product category       | `product.category`                            | `product.category`                                              | `context_product_category`                    |                                           |
| product brand          | `product.brandName` / `brand?.name` / `brand?.title` / `brand` | `product.brand` or `product.brandName`                          | `context_product_brand`                       | Frontend maps brand from DB fields      |
| product price          | `product.price`                               | `product.price`                                                 | `context_product_price`                       | BIGINT                                   |
| product discount price | `product.discountPrice`                       | `product.discountPrice`                                         | `context_product_discount_price`              | BIGINT                                   |
| product url path       | `product.urlPath`                             | `product.urlPath`                                               | `context_product_url_path`                    |                                           |
| element metadata       | `element.{tag,id,role,text,dataset}`          | `element...`                                                    | -                                             | Not stored (needs schema change)        |
| ingest metadata        | -                                             | `_ingest.{receivedAt,sourceIp,userAgent,method,path,requestId,apiId,stage,traceId}` | -                                             | Not stored; available during ETL        |

Call pattern (one event per request):

```ts
await fetch("https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/clickstream", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(eventPayload),
  keepalive: true,
});
```

ETL maps `eventId -> event_id` (PK with ON CONFLICT DO NOTHING), derives `event_timestamp` from `_ingest.receivedAt` > payload > S3 LastModified, and inserts records into `clickstream_dw.public.clickstream_events`.

---

## 5.3.6 Testing & Validation

To verify that ingestion is working correctly:

1. Use the Amplify app UI:
   - Navigate through a few product pages  
   - Add items to the shopping cart  
2. Inspect the S3 bucket `clickstream-s3-ingest`:
   - Navigate to `events/YYYY/MM/DD/HH/`  
   - Confirm that new `event-<uuid>.json` files have appeared.  
3. Open and inspect one JSON file:
   - Verify that `_ingest` metadata and product context fields are present.  
4. Review the logs:
   - API Gateway access logs  
   - Lambda function logs  
