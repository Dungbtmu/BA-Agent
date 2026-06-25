# Customer Model — CDP VNPost/TCT

**Dự án:** CDP cho VNPost/TCT  
**Nguồn tham chiếu:** `.claude/input/CDP/CDP.md` (mục 6.2, 6.3, 6.4, 6.5, 6.8, 6.9, 7.5)  
**Ngày tạo:** 2026-06-25  
**Mục đích:** Định nghĩa cấu trúc dữ liệu "một khách hàng" trong CDP — làm nền tảng cho Identity Resolution, Customer 360, Segmentation

---

## 1. Tổng quan mô hình

Customer Model của CDP VNPost được xây theo nguyên tắc **"một Customer ID, nhiều vai trò, nhiều nguồn"**:

- Một khách hàng → **một Unified Customer ID (UID)** duy nhất do CDP quản lý
- Một UID có thể có **nhiều vai trò** trong giao dịch bưu chính: người gửi, người nhận, chủ shop, KHL
- Một UID tổng hợp dữ liệu từ **nhiều hệ thống nguồn** khác nhau (CAS, CRM, MyVNPost, PayPost...)
- Hai loại khách hàng **khác nhau về cấu trúc**: Cá nhân (Individual) và Doanh nghiệp (Business)

---

## 2. Hai loại Customer

### 2.1. Khách hàng cá nhân (Individual Customer)

Là cá nhân sử dụng dịch vụ VNPost — người gửi qua app/bưu cục, người nhận hàng, chủ shop nhỏ (SME).

**Nguồn dữ liệu chính:** CAS, MyVNPost, PostID, CRM/Care Đơn, PNS/DingDong

### 2.2. Khách hàng doanh nghiệp (Business Customer / KHL)

Là doanh nghiệp, tổ chức, hoặc chủ shop TMĐT có sản lượng gửi lớn, ký hợp đồng với VNPost.

**Nguồn dữ liệu chính:** Portal KHL, CRM, CAS, MPITS

**Quy tắc phân biệt — theo thứ tự ưu tiên:**

```
1. Có CCCD → Individual Customer
   (MST nếu có = CCCD — đây là MST cá nhân, không phải MST doanh nghiệp)

2. Không có CCCD, có MST → Business Customer
   (MST doanh nghiệp — 10 hoặc 13 số, khác cấu trúc CCCD)

3. Không có cả hai → Individual Customer (default)
   (người nhận không có tài khoản, receiver profile)
```

> **Lưu ý:** Cá nhân có thể có MST trùng với CCCD — đây là MST cá nhân do cơ quan thuế cấp, không phải MST doanh nghiệp. Không dùng MST đơn lẻ để phân loại Business nếu đã có CCCD.

---

## 3. Cấu trúc Customer Model đầy đủ

### BLOCK A — Định danh & Hồ sơ cơ bản (Core Identity)

> Nhóm dữ liệu dùng để **nhận diện** và **hợp nhất** hồ sơ.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `unified_customer_id` | String | CDP (tự sinh) | UID duy nhất, không đổi sau khi tạo |
| `customer_type` | Enum | CDP (suy luận) | `INDIVIDUAL` / `BUSINESS` |
| `customer_status` | Enum | CDP | `ACTIVE` / `INACTIVE` / `BLOCKED` / `MERGED` |
| `full_name` | String | CRM, CAS, MyVNPost | Tên đã chuẩn hóa, có dấu tiếng Việt |
| `phone` | String | CRM, CAS, MyVNPost, Portal KHL | Chuẩn hóa về 0xxxxxxxxx (10 số) |
| `email` | String | CRM, Website, MyVNPost, Portal KHL | Lowercase, đã kiểm tra định dạng |
| `cccd` | String (encrypted) | CRM, PostID | Mã hóa bắt buộc; masking khi hiển thị (012\*\*\*\*\*\*\*\*) |
| `post_id` | String | PostID | Điểm neo định danh chính của hệ sinh thái VNPost |
| `user_id_app` | String | MyVNPost App | Liên kết hành vi số |
| `crm_id` | String | CRM/Care Đơn | Dùng để đồng bộ ngược CRM |
| `managing_post_office` | String | CAS, CRM | Bưu cục quản lý khách hàng |
| `customer_group` | Enum | CRM, CDP | `B2C` / `B2B` / `KHL` / `SME` / `ECOMMERCE` |
| `created_at` | DateTime | CDP | Thời điểm tạo hồ sơ trong CDP |
| `last_updated_at` | DateTime | CDP | Lần cập nhật gần nhất |
| `data_completeness_score` | Float | CDP (tính tự động) | % trường quan trọng đã có dữ liệu (0–100) |

---

### BLOCK B — Thông tin Doanh nghiệp (Business Profile)

> Chỉ áp dụng khi `customer_type = BUSINESS`. Null nếu là Individual.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `company_name` | String | Portal KHL, CRM | Tên doanh nghiệp đã chuẩn hóa |
| `tax_id` | String | Portal KHL, CRM | MST 10 hoặc 13 số; khóa merge mạnh nhất cho B2B |
| `khl_code` | String | Portal KHL | Mã khách hàng lớn — khóa merge thứ 2 cho B2B |
| `legal_representative` | String | Portal KHL, CRM | Người đại diện pháp lý |
| `contract_status` | Enum | Portal KHL, CRM | `ACTIVE` / `EXPIRED` / `PENDING` / `TERMINATED` |
| `contract_start_date` | Date | Portal KHL | Ngày bắt đầu hợp đồng |
| `contract_end_date` | Date | Portal KHL | Ngày kết thúc hợp đồng |
| `business_segment` | String | CRM, CDP | Ngành nghề / phân loại hợp đồng |
| `contacts` | Array[Contact] | Portal KHL, CRM | Danh sách người liên hệ trong doanh nghiệp — xem mô hình phụ bên dưới |

**Mô hình phụ — Contact (người liên hệ trong doanh nghiệp):**

| Trường | Kiểu | Ghi chú |
|---|---|---|
| `contact_name` | String | Họ tên người liên hệ |
| `contact_phone` | String | SĐT liên hệ (không phải SĐT doanh nghiệp) |
| `contact_email` | String | Email cá nhân người liên hệ |
| `contact_role` | String | Chức vụ / vai trò trong doanh nghiệp |
| `is_primary` | Boolean | Người liên hệ chính |

> **Lý do tách Contact riêng:** Một doanh nghiệp/KHL có thể có nhiều nhân sự dùng SĐT/email khác nhau. Không được gộp tất cả contact thành một cá nhân trong Identity Resolution (mục 6.9 case 9).

---

### BLOCK C — Địa chỉ (Address)

> Một khách hàng có thể có nhiều địa chỉ với các loại khác nhau.

> **Lưu ý hành chính:** Từ 1/7/2025, Việt Nam bỏ cấp huyện/quận — cấu trúc chính thức hiện tại chỉ còn **Xã/Phường → Tỉnh/Thành phố**. Tuy nhiên CDP vẫn giữ trường `district` để tương thích với dữ liệu lịch sử và hệ thống tuyến phát đang vận hành theo địa bàn cũ. Trường `administrative_version` ghi nhận địa chỉ thuộc cấu trúc nào.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `address_id` | String | CDP (tự sinh) | ID địa chỉ nội bộ |
| `address_type` | Enum | CDP | `CONTACT` / `SENDER` / `RECEIVER` / `BUSINESS` |
| `is_primary` | Boolean | CDP | Địa chỉ chính (dùng mặc định) |
| `is_frequently_used` | Boolean | CDP (tính từ lịch sử) | Địa chỉ thường dùng (gửi ≥ 3 lần trong 6 tháng) |
| `raw_address` | String | CAS, MyVNPost | Địa chỉ gốc từ hệ thống nguồn — không chỉnh sửa |
| `province` | String | VPostCode, Vmap | Tỉnh/Thành phố — cấp hành chính chính thức hiện tại |
| `district` | String | VPostCode, Vmap | Quận/Huyện — **legacy, giữ lại để tương thích**; null với địa chỉ mới nhập sau 1/7/2025 |
| `ward` | String | VPostCode, Vmap | Xã/Phường — cấp trực tiếp dưới Tỉnh trong cấu trúc mới |
| `street_detail` | String | CAS, MyVNPost | Số nhà, tên đường |
| `administrative_version` | Enum | CDP | `NEW` (Xã→Tỉnh, từ 1/7/2025) / `LEGACY` (Xã→Huyện→Tỉnh, trước 1/7/2025) |
| `vpostcode` | String | VPostCode | Mã địa chỉ số VNPost — dùng làm chuẩn cho tuyến phát |
| `service_zone` | String | VPostCode | Vùng phục vụ / bưu cục phụ trách |
| `standardization_status` | Enum | CDP | `STANDARDIZED` / `PARTIAL` / `FAILED` |
| `source_system` | String | CDP | Nguồn gốc địa chỉ |
| `created_at` | DateTime | CDP | |

---

### BLOCK D — Định danh & Alias (Identity Graph)

> Lưu toàn bộ ID từ các hệ thống nguồn để truy vết và đồng bộ ngược.

| Trường | Kiểu | Nguồn | Mức tin cậy | Ghi chú |
|---|---|---|---|---|
| `alias_id` | String | CDP (tự sinh) | — | ID của bản ghi alias |
| `id_type` | Enum | CDP | — | `PHONE` / `EMAIL` / `POST_ID` / `CRM_ID` / `KHL_CODE` / `TAX_ID` / `CCCD` / `USER_APP_ID` / `COOKIE_ID` / `DEVICE_ID` / `SHIPMENT_CODE` / `PAYMENT_ID` |
| `id_value` | String | Hệ thống nguồn | — | Giá trị định danh gốc |
| `source_system` | String | CDP | — | CAS / CRM / MyVNPost / Portal KHL / PostID... |
| `confidence_level` | Enum | CDP | — | `HIGH` / `MEDIUM` / `LOW` |
| `is_shared` | Boolean | CDP | — | `true` nếu ID này dùng chung nhiều người (hotline, email kế toán...) |
| `is_primary` | Boolean | CDP | — | ID chính đang dùng của loại này |
| `status` | Enum | CDP | — | `ACTIVE` / `INACTIVE` / `MERGED_INTO` |
| `merged_into_uid` | String | CDP | — | Nếu status = MERGED_INTO, trỏ về UID master |
| `verified_at` | DateTime | CDP | — | Thời điểm xác thực |

**Thứ tự ưu tiên khi merge (từ cao xuống thấp):**

```
1. CCCD (rất cao) — khóa mạnh nhất cho Individual Customer
2. Tax ID / MST (rất cao) — khóa mạnh nhất cho Business Customer
   (Lưu ý: nếu hồ sơ đã có CCCD thì MST = MST cá nhân, không dùng để merge B2B)
3. PostID (cao) — neo định danh VNPost
4. SĐT + Email (cao khi có cả 2)
5. CRM ID / KHL Code (cao)
6. User App ID đã xác thực (cao)
7. SĐT đơn lẻ (cao, nhưng kiểm tra shared)
8. Email đơn lẻ (cao, nhưng kiểm tra shared)
9. Device ID (trung bình — chỉ kết hợp)
10. Cookie ID (thấp — chỉ liên kết hành vi ẩn danh)
```

---

### BLOCK E — Vai trò trong Giao dịch (Transaction Role)

> Quan trọng đặc thù VNPost: một người có thể là người gửi ở giao dịch này, người nhận ở giao dịch khác.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `roles` | Array[Enum] | CDP (suy luận từ giao dịch) | `SENDER` / `RECEIVER` / `SHOP_OWNER` / `KHL` / `SME` |
| `primary_role` | Enum | CDP | Vai trò chính dựa theo lịch sử (role xuất hiện nhiều nhất) |
| `sender_transaction_count` | Integer | CDP (tổng hợp) | Số lần là người gửi |
| `receiver_transaction_count` | Integer | CDP (tổng hợp) | Số lần là người nhận |

> **Lý do quan trọng:** Tránh gộp nhầm người gửi và người nhận chỉ vì trùng SĐT — phải kiểm tra role trước khi merge (mục 6.8.2 và 6.9 case 8).

---

### BLOCK F — Lịch sử Giao dịch (Transaction Summary)

> Lưu **tóm tắt tổng hợp** phục vụ phân tích và hiển thị C360. Chi tiết từng đơn lưu ở bảng Transaction riêng.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `total_orders` | Integer | CAS, MPITS | Tổng số đơn hàng |
| `total_revenue` | Decimal | CAS, MPITS | Tổng doanh thu cước phí |
| `first_transaction_date` | Date | CAS | Ngày giao dịch đầu tiên |
| `last_transaction_date` | Date | CAS, MPITS | Ngày giao dịch gần nhất |
| `avg_monthly_orders` | Float | CDP (tính) | Trung bình số đơn / tháng |
| `most_used_service` | String | CDP (tính) | Loại dịch vụ dùng nhiều nhất |
| `frequent_sender_routes` | Array[String] | CDP (tính) | Tuyến gửi thường xuyên (top 3) |
| `success_delivery_rate` | Float | PNS/DingDong, BCCP | Tỷ lệ phát thành công (0–1) |
| `return_rate` | Float | PNS/DingDong | Tỷ lệ hoàn hàng (0–1) |

---

### BLOCK G — COD & Tài chính (COD Summary)

> Tổng hợp COD toàn bộ — không phải chi tiết từng đơn.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `total_cod_amount` | Decimal | PayPost | Tổng COD phát sinh |
| `total_cod_collected` | Decimal | PayPost | Tổng COD đã thu |
| `total_cod_pending` | Decimal | PayPost | Tổng COD chưa thu |
| `cod_reconciliation_status` | Enum | PayPost | `MATCHED` / `DISCREPANCY` / `PENDING` |
| `cod_risk_history` | String | CDP | Ghi chú lịch sử rủi ro COD (nếu có) |
| `payment_account` | String (masked) | PayPost | Tài khoản nhận tiền COD — masked khi hiển thị |

---

### BLOCK H — Hành vi Số (Digital Behavior)

> Các sự kiện hành vi từ app/web — phân tích engagement.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `last_app_login` | DateTime | MyVNPost | Lần đăng nhập app gần nhất |
| `last_web_visit` | DateTime | Website/CMS | Lần truy cập web gần nhất |
| `app_install_date` | Date | MyVNPost | Ngày cài app lần đầu |
| `device_os` | String | MyVNPost, SDK | iOS / Android |
| `total_app_sessions_30d` | Integer | CDP (tính) | Số phiên đăng nhập app trong 30 ngày |
| `behavior_events` | Array[Event] | MyVNPost, SDK, Website | Danh sách sự kiện — xem mô hình Event bên dưới |

**Mô hình phụ — Event (sự kiện hành vi):**

| Trường | Kiểu | Ghi chú |
|---|---|---|
| `event_id` | String | ID sự kiện |
| `event_type` | Enum | `APP_LOGIN` / `WEB_VISIT` / `ORDER_CREATED` / `PRICE_INQUIRY` / `TRACKING` / `FIND_POST_OFFICE` / `BANNER_CLICK` / `CAMPAIGN_RESPONSE` / `SMS_RECEIVED` / `EMAIL_OPENED` / `ZALO_INTERACTED` / `COUNTER_VISIT` / `CALL_CENTER_CONTACT` |
| `event_time` | DateTime | Thời điểm xảy ra |
| `channel` | Enum | `APP` / `WEB` / `COUNTER` / `CALL_CENTER` / `SMS` / `EMAIL` / `ZALO` / `PUSH` |
| `source_system` | String | Hệ thống ghi nhận |
| `metadata` | JSON | Thông tin bổ sung tùy loại event (campaign_id, search_keyword...) |

---

### BLOCK I — Chăm sóc Khách hàng (CSKH Summary)

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `total_complaints` | Integer | CRM, Care Đơn | Tổng số khiếu nại |
| `open_complaints` | Integer | CRM | Khiếu nại đang xử lý |
| `last_complaint_date` | Date | CRM | Ngày khiếu nại gần nhất |
| `avg_resolution_time_hours` | Float | CRM | Thời gian xử lý trung bình (giờ) |
| `sla_compliance_rate` | Float | CRM | Tỷ lệ xử lý đúng hạn (0–1) |
| `satisfaction_score` | Float | CRM | Điểm hài lòng trung bình (0–5) |
| `complaint_history` | Array[Complaint] | CRM, Care Đơn | Chi tiết từng khiếu nại |

---

### BLOCK J — Loyalty

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `loyalty_tier` | Enum | Loyalty, CRM | `STANDARD` / `SILVER` / `GOLD` / `PLATINUM` — null nếu chưa có chương trình |
| `loyalty_points` | Integer | Loyalty | Điểm tích lũy hiện tại |
| `points_expiry_date` | Date | Loyalty | Ngày hết hạn điểm |
| `tier_achieved_date` | Date | Loyalty | Ngày đạt hạng hiện tại |
| `benefits_history` | Array | Loyalty | Lịch sử sử dụng quyền lợi |

> **Lưu ý MVP:** Dữ liệu Loyalty có thể chưa sẵn sàng ở giai đoạn 1. Hiển thị "Chưa có dữ liệu" thay vì bỏ trống.

---

### BLOCK K — Phân khúc & Điểm số (Scores & Segments)

> Kết quả tính toán từ CDP Analytics — cập nhật định kỳ hoặc theo sự kiện.

| Trường | Kiểu | Nguồn | Scale | Ghi chú |
|---|---|---|---|---|
| `current_segments` | Array[String] | CDP Analytics | — | Danh sách segment đang thuộc (có thể thuộc nhiều segment) |
| `rfm_recency` | Integer | CDP Analytics | 1–5 | 5 = giao dịch rất gần đây |
| `rfm_frequency` | Integer | CDP Analytics | 1–5 | 5 = giao dịch rất thường xuyên |
| `rfm_monetary` | Integer | CDP Analytics | 1–5 | 5 = giá trị rất cao |
| `rfm_group` | String | CDP Analytics | — | VD: "Champions", "At Risk", "Lost" |
| `clv_score` | Decimal | CDP Analytics | VND | Giá trị vòng đời ước tính |
| `churn_score` | Integer | CDP Analytics | 0–100 | Nguy cơ rời bỏ — 100 = rất cao |
| `engagement_score` | Integer | CDP Analytics | 0–100 | Mức độ tương tác số |
| `cod_risk_score` | Integer | CDP Analytics | 0–100 | Rủi ro COD — 100 = rất rủi ro |
| `fraud_score` | Integer | CDP Analytics | 0–100 | Nguy cơ gian lận — 100 = rất cao |
| `scores_updated_at` | DateTime | CDP Analytics | — | Lần tính điểm gần nhất |

**Quy tắc hiển thị theo vai trò:**

| Điểm số | CSKH | Marketing | Kinh doanh | Vận hành / COD | Admin |
|---|---|---|---|---|---|
| RFM | Xem | Xem | Xem | Không | Xem |
| CLV | Xem (tổng quát) | Xem | Xem | Không | Xem |
| Churn Score | Xem | Xem | Xem | Không | Xem |
| Engagement Score | Xem | Xem | Không | Không | Xem |
| COD Risk Score | Không | Không | Xem | Xem | Xem |
| Fraud Score | Không | Không | Xem | Xem | Xem |

---

### BLOCK L — Consent & Quyền riêng tư

> Bắt buộc kiểm tra trước mọi activation. Tuân thủ Luật Bảo vệ Dữ liệu Cá nhân số 91/2025/QH15.

| Trường | Kiểu | Nguồn | Ghi chú |
|---|---|---|---|
| `consent_sms` | Enum | CDP, CRM, App | `OPT_IN` / `OPT_OUT` / `UNKNOWN` |
| `consent_email` | Enum | CDP, CRM, Website | `OPT_IN` / `OPT_OUT` / `UNKNOWN` |
| `consent_zalo` | Enum | CDP, Zalo OA | `OPT_IN` / `OPT_OUT` / `UNKNOWN` |
| `consent_push` | Enum | CDP, MyVNPost App | `OPT_IN` / `OPT_OUT` / `UNKNOWN` |
| `consent_data_analysis` | Enum | CDP | Đồng ý cho phép phân tích dữ liệu |
| `consent_source` | String | CDP | Nguồn ghi nhận consent (App / Web / Quầy / CRM...) |
| `consent_updated_at` | DateTime | CDP | Lần cập nhật consent gần nhất |
| `data_deletion_requested` | Boolean | CDP | Khách hàng đã yêu cầu xóa dữ liệu |
| `data_deletion_date` | Date | CDP | Ngày xử lý yêu cầu xóa |
| `consent_history` | Array[ConsentLog] | CDP | Lịch sử thay đổi consent — audit trail |

---

### BLOCK M — Metadata & Audit

| Trường | Kiểu | Ghi chú |
|---|---|---|
| `source_systems` | Array[String] | Danh sách hệ thống đã đóng góp dữ liệu |
| `merge_history` | Array[MergeLog] | Lịch sử merge/unmerge |
| `merged_from_uids` | Array[String] | Các UID đã được merge vào UID này |
| `data_steward_notes` | String | Ghi chú của Data Steward |
| `last_reviewed_by` | String | Người review hồ sơ lần cuối |
| `profile_version` | Integer | Version hồ sơ (tăng mỗi lần cập nhật) |
| `created_by_system` | String | Hệ thống tạo hồ sơ ban đầu |

---

## 4. Sơ đồ quan hệ tổng thể

```
                    ┌─────────────────────────────┐
                    │     CUSTOMER (UID)           │
                    │  Block A: Core Identity      │
                    │  Block E: Transaction Roles  │
                    │  Block M: Metadata & Audit   │
                    └──────────┬──────────────────┘
                               │ 1
           ┌───────────────────┼────────────────────┐
           │                   │                    │
           ▼ 0..1              ▼ 0..n              ▼ 0..n
  ┌─────────────────┐  ┌──────────────┐   ┌──────────────────┐
  │  BUSINESS       │  │  ADDRESS     │   │  ALIAS ID        │
  │  PROFILE        │  │  Block C     │   │  Block D         │
  │  Block B        │  └──────────────┘   └──────────────────┘
  │  + Contacts     │
  └─────────────────┘
           │
           ▼ 0..n
    ┌──────────────┐

  ┌───────────────────────────────────────────────────────────┐
  │  DỮ LIỆU TỔNG HỢP (tính toán từ lịch sử, cập nhật định kỳ)│
  ├───────────────┬───────────────┬───────────────────────────┤
  │ TRANSACTION   │ COD SUMMARY   │ CSKH SUMMARY              │
  │ SUMMARY       │ Block G       │ Block I                   │
  │ Block F       │               │                           │
  ├───────────────┼───────────────┼───────────────────────────┤
  │ DIGITAL       │ LOYALTY       │ SCORES &                  │
  │ BEHAVIOR      │ Block J       │ SEGMENTS                  │
  │ Block H       │               │ Block K                   │
  └───────────────┴───────────────┴───────────────────────────┘

  ┌──────────────────────────────────────────┐
  │  CONSENT & QUYỀN RIÊNG TƯ — Block L     │
  │  (kiểm tra trước MỌI activation)         │
  └──────────────────────────────────────────┘
```

---

## 5. Quy tắc nguồn dữ liệu ưu tiên (Master Data Rules)

Khi cùng một trường có giá trị khác nhau từ nhiều nguồn:

| Trường | Nguồn ưu tiên cao nhất | Lý do |
|---|---|---|
| Họ tên | CRM → PostID → CAS | CRM có dữ liệu đã được xác thực |
| Số điện thoại | PostID → CRM → CAS → MyVNPost | PostID là nguồn xác thực định danh |
| Email | PostID → CRM → MyVNPost | PostID có xác thực email |
| Địa chỉ | VPostCode/Vmap → CAS → MyVNPost | Địa chỉ đã chuẩn hóa ưu tiên hơn raw |
| Trạng thái phát | PNS/DingDong → BCCP → MPITS | Nguồn cập nhật trực tiếp kết quả phát |
| COD | PayPost → PNS/DingDong | PayPost là nguồn tài chính chính thức |
| Trạng thái consent | Bản ghi mới nhất có bằng chứng ghi nhận | Không ưu tiên theo hệ thống mà theo thời điểm |
| Điểm số / Segment | CDP Analytics | CDP là nguồn duy nhất tính và phân phối |
| Loyalty | Hệ thống Loyalty → CRM | Hệ thống Loyalty là master |

---

## 6. Quy tắc Masking theo vai trò

| Trường | Vai trò được xem đầy đủ | Vai trò bị masking | Định dạng masking |
|---|---|---|---|
| Số điện thoại | Admin, IT Security | CSKH cơ bản, Marketing | 0912\*\*\*444 |
| Email | Admin, IT Security | CSKH cơ bản | nguy\*\*\*\*\*@gmail.com |
| CCCD | Admin, IT Security, Compliance | Tất cả vai trò khác | 012\*\*\*\*\*\*\*\* |
| Tài khoản COD/thanh toán | Admin, Tài chính/COD | Tất cả vai trò khác | \*\*\*\*1234 |
| COD Risk Score | Admin, Kinh doanh, Vận hành | CSKH, Marketing | Ẩn hoàn toàn |
| Fraud Score | Admin, Kinh doanh, Vận hành | CSKH, Marketing | Ẩn hoàn toàn |

---

## 7. Trường hợp đặc biệt cần xử lý

| Tình huống | Xử lý |
|---|---|
| Người nhận không có tài khoản | Tạo **Receiver Profile** tạm — lưu tên, SĐT, địa chỉ nhận; không activation nếu thiếu consent |
| SĐT dùng chung (hotline, gia đình) | Đánh dấu `is_shared = true`; không dùng làm khóa merge tự động |
| Khách hàng đổi SĐT | Giữ SĐT cũ dưới dạng alias `INACTIVE`; cập nhật SĐT mới là primary |
| Doanh nghiệp có nhiều người liên hệ | Quản lý theo mô hình Organization → Contact; không gộp thành 1 cá nhân |
| Hồ sơ bị merge nhầm | Unmerge: tách thành 2 UID riêng, trả lại alias ID gốc, lưu audit log |
| Khách hàng yêu cầu xóa dữ liệu | Bật `data_deletion_requested = true`; xử lý theo Luật 91/2025/QH15; lưu `data_deletion_date` |

---

## 9. Conflict Resolution & Data Quality Rules

> Áp dụng khi CDP nhận dữ liệu từ nhiều hệ thống nguồn về cùng một khách hàng. Mục tiêu: chọn đúng giá trị, không mất dữ liệu, có thể rollback.

---

### Nhóm A — Fuzzy Matching (So khớp mờ)

Dùng khi so sánh các trường dạng text (tên, địa chỉ) để xác định "hai bản ghi có phải cùng một người không".

**Bước 1 — Chuẩn hóa trước khi so (bắt buộc với mọi trường text):**

```
1. Bỏ dấu tiếng Việt → chuyển về ASCII (Nguyễn → Nguyen)
2. Lowercase toàn bộ
3. Normalize Unicode về NFC (xử lý ký tự tổ hợp)
4. Trim + bỏ khoảng trắng thừa giữa các từ
5. Bỏ ký tự đặc biệt (dấu chấm, phẩy, gạch ngang, ngoặc đơn)
```

Sau bước này nhiều case tự khớp 100% mà không cần fuzzy: `Nguyễn Văn A` và `nguyen van a` đều thành `nguyen van a`.

**Bước 2 — Thuật toán đo độ giống nhau:**

Dùng **Jaro-Winkler** — ưu tiên khớp phần đầu chuỗi, phù hợp với tên người Việt (họ đứng đầu, thường giống nhau).

**Bước 3 — Ngưỡng quyết định:**

| Ngưỡng | Hành động |
|---|---|
| **100%** | Tự động merge |
| **85–99%** | Tự động merge + ghi log để audit (sai chính tả nhỏ, viết tắt tên đệm) |
| **70–84%** | Đưa vào hàng đợi review thủ công — Data Steward xác nhận |
| **< 70%** | Không merge — coi là hai người khác nhau |

**Bước 4 — Confidence Score tổng hợp:**

Tên đơn lẻ không đủ để merge. Phải kết hợp nhiều trường:

```
Confidence Score =
  Tên (30%)
  + Số điện thoại (40%)
  + Ngày sinh (20%)
  + Tỉnh/Thành phố (10%)
```

| Confidence Score tổng hợp | Hành động |
|---|---|
| ≥ 90% | Tự động merge |
| 70–89% | Merge + ghi log |
| 50–69% | Hàng đợi review thủ công |
| < 50% | Không merge |

> **Ví dụ:** Tên giống 90% + SĐT khác hoàn toàn → Confidence Score thấp → KHÔNG merge dù tên gần giống (có thể là hai người trùng tên).

**Các case hay gặp với dữ liệu VNPost:**

| Tình huống | Ví dụ | Xử lý |
|---|---|---|
| Có dấu vs không dấu | `Nguyễn Văn A` vs `Nguyen Van A` | Khớp 100% sau chuẩn hóa |
| Viết hoa/thường | `NGUYEN VAN A` vs `nguyen van a` | Khớp 100% sau chuẩn hóa |
| Viết tắt tên đệm | `Nguyễn V. An` vs `Nguyễn Văn An` | Fuzzy ~88% → merge + log |
| Thừa khoảng trắng | `Nguyễn  Văn  A` vs `Nguyễn Văn A` | Khớp 100% sau chuẩn hóa |
| Sai chính tả 1 ký tự | `Nguyên Văn An` vs `Nguyễn Văn An` | Fuzzy ~92% → merge + log |
| Đảo thứ tự họ tên | `Văn An Nguyễn` vs `Nguyễn Văn An` | Fuzzy thấp → review thủ công |
| Tên DN viết tắt | `Cty TNHH ABC` vs `Công ty TNHH ABC` | Dùng MST làm khóa chính, không dùng tên để merge DN |

---

### Nhóm B — Xung đột giá trị cùng mức ưu tiên (Tie-breaking)

Khi hai nguồn có cùng mức ưu tiên (theo Mục 5) nhưng giá trị khác nhau:

| Quy tắc | Chi tiết |
|---|---|
| **Tie-break theo thời gian** | Lấy giá trị có `updated_at` mới hơn |
| **Không xóa giá trị cũ** | Giá trị thua vẫn lưu trong Identity Graph (Block D) dưới dạng alias `INACTIVE` |
| **Ghi log conflict** | Lưu vào `merge_history` (Block M): trường nào conflict, giá trị nào thắng, lý do |

---

### Nhóm C — Giá trị thay đổi đột ngột (Anomaly Detection)

Khi giá trị mới từ hệ thống nguồn khác hoàn toàn so với giá trị đang có — dù nguồn có ưu tiên cao hơn:

| Trường | Ngưỡng cảnh báo | Hành động |
|---|---|---|
| Họ tên | Fuzzy < 60% so với tên hiện tại | Không ghi đè — đưa vào hàng đợi review |
| Số điện thoại | Đổi hoàn toàn sang SĐT chưa từng xuất hiện | Ghi đè có điều kiện: chỉ nếu có OTP xác thực |
| CCCD | Bất kỳ thay đổi nào | Không tự động ghi đè — bắt buộc review thủ công |
| Địa chỉ tỉnh/thành | Thay đổi sang tỉnh khác trong < 24 giờ | Ghi nhận cả hai, không ghi đè địa chỉ cũ |
| `customer_type` | Thay đổi từ INDIVIDUAL → BUSINESS hoặc ngược lại | Bắt buộc Data Steward xác nhận |

> **Lý do:** Thay đổi đột ngột có thể là dữ liệu sai từ hệ thống nguồn, hoặc dấu hiệu gian lận chiếm tài khoản.

---

### Nhóm D — Xử lý Null và trường rỗng

| Quy tắc | Chi tiết |
|---|---|
| **Null ≠ Rỗng** | `null` = chưa có dữ liệu; `""` = dữ liệu được nhập nhưng trống — xử lý khác nhau |
| **Null không ghi đè** | Nếu nguồn mới gửi `null` cho một trường đang có giá trị → giữ nguyên giá trị cũ, không ghi `null` |
| **Rỗng có thể ghi đè** | Nếu nguồn ưu tiên cao hơn gửi `""` → ghi đè + log (có thể khách hàng đã xóa thông tin) |
| **Trường bắt buộc** | `full_name`, `phone`, `customer_type` không được để null sau khi tạo hồ sơ — validate trước khi ghi |

---

### Nhóm E — Rollback & Unmerge

Khi phát hiện merge sai (hai người khác nhau bị gộp vào một UID):

```
Bước 1: Tạo UID mới cho hồ sơ bị merge nhầm
Bước 2: Phân tách alias ID — trả về đúng UID gốc của từng ID
Bước 3: Phân tách dữ liệu lịch sử (giao dịch, COD, khiếu nại) — gán về đúng UID
Bước 4: Đánh dấu UID cũ là MERGED → cập nhật status = ACTIVE cho cả hai UID mới
Bước 5: Ghi audit log đầy đủ vào Block M: ai unmerge, lúc nào, lý do
```

> **Nguyên tắc bất biến:** Không xóa UID cũ — chỉ đổi status thành `MERGED`. Toàn bộ lịch sử phải truy vết được.

---

### Nhóm F — Tần suất đồng bộ và độ trễ chấp nhận được

| Nhóm trường | Chế độ đồng bộ | Độ trễ tối đa chấp nhận |
|---|---|---|
| Trạng thái phát đơn (PNS/DingDong) | Real-time / near real-time | < 5 phút |
| Consent (mọi kênh) | Real-time | < 1 phút — vi phạm luật nếu trễ hơn |
| COD / thanh toán (PayPost) | Near real-time | < 15 phút |
| Thông tin định danh (CRM, CAS) | Batch hàng ngày | < 24 giờ |
| Điểm số / Segment (CDP Analytics) | Batch định kỳ | < 24 giờ (có thể tăng lên theo năng lực hệ thống) |
| Loyalty (hệ thống Loyalty) | Batch hàng ngày | < 24 giờ |
| Hành vi số (app/web events) | Near real-time | < 5 phút |

> **Lưu ý:** `scores_updated_at` trong Block K cho phép hệ thống hiển thị cảnh báo khi điểm số đã cũ hơn ngưỡng cho phép.

---

## 8. Open Questions cần chốt

- [ ] **OQ-01:** `customer_group` có bao nhiêu giá trị? Ngoài B2C / B2B / KHL / SME / ECOMMERCE còn loại nào khác không?
- [ ] **OQ-02:** Loyalty tier có 4 hạng như trong model chưa, hay VNPost đang dùng hạng khác?
- [ ] **OQ-03:** Scale điểm số (0–100) đã được stakeholder đồng ý chưa, hay cần điều chỉnh?
- [ ] **OQ-04:** Receiver Profile (người nhận không có tài khoản) có được đưa vào Customer List không, hay chỉ lưu trong Identity Graph?
- [ ] **OQ-05:** `managing_post_office` — VNPost có rule gán bưu cục quản lý theo địa bàn không, hay do nhân viên gán thủ công?
