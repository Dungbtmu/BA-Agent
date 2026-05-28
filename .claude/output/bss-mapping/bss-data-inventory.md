# BSS Data Inventory — Những gì BSS đang cung cấp

> Tài liệu này tổng hợp toàn bộ dữ liệu hiện có trong DB BSS (8 schema), phân tích mức độ liên quan đến CVM và xác định các khoảng trống cần bổ sung từ nguồn khác.
>
> Cập nhật: 2026-05-14

---

## 1. Tổng quan 8 Schema BSS

| Schema | Mục đích | Liên quan CVM |
|---|---|---|
| `crm` | Khách hàng, thuê bao, vòng đời | 🔴 Cao — nguồn dữ liệu KH chính |
| `resource` | SIM, MSISDN, pattern số | 🔴 Cao — nguồn trigger & phân loại số |
| `notification` | Kênh gửi tin, template, queue | 🔴 Cao — hạ tầng gửi thông báo |
| `auth` | Người dùng, phân quyền, session | 🟡 Trung bình — QTV/Admin login CVM |
| `cdr` | Bản ghi cước (CDR) — lịch sử | 🟡 Trung bình — phân tích hành vi, không dùng làm trigger realtime |
| `sales` | Đơn hàng, coupons, campaigns | 🟡 Trung bình — lịch sử mua hàng, voucher |
| `esim_portal` | Portal bán eSIM cho KH đại lý | 🟢 Thấp — tệp KH khác |
| `esim_agency` | Quản lý đại lý eSIM, ví | 🟢 Thấp — không liên quan |
| `travel` | eSIM du lịch quốc tế | 🟢 Thấp — không liên quan |
| `cms` | Nội dung website, bài viết | 🟢 Thấp — không liên quan |

---

## 2. Chi tiết từng schema liên quan CVM

### 2.1 Schema `crm` — Dữ liệu Khách hàng & Thuê bao

**Bảng `customers`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `customer_code` | Mã khách hàng | Định danh |
| `full_name` | Họ tên | `{{ten_kh}}` |
| `id_number` | CMND/CCCD | Xác thực |
| `contact_phone` | SĐT liên hệ | Gửi SMS dự phòng |
| `contact_email` | Email | `{{email}}` — kênh Email |
| `address` | Địa chỉ | — |
| `status` | ACTIVE / INACTIVE / BLOCKED | Lọc segment |

**Bảng `subscribers`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `msisdn` | Số thuê bao | `{{so_dien_thoai}}` — kênh USSD/SMS |
| `customer_id` | FK → customers | Join lấy thông tin KH |
| `sim_id` | FK → sims | Join lấy thông tin SIM |
| `status` | Trạng thái thuê bao | `{{trang_thai_thue_bao}}`, lọc segment |
| `activation_date` | Ngày kích hoạt | Tham chiếu NGAY_0 |

**Bảng `subscriber_lifecycles`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `lifecycle_number` | Số lần kích hoạt | Phát hiện tái kích hoạt |
| `start_date` | Ngày bắt đầu | Phân tích retention |
| `end_date` | Ngày kết thúc | Phát hiện churn |

**Bảng `customer_complaints`** — lịch sử khiếu nại → segment KH có vấn đề dịch vụ

**Bảng `msisdn_reclaim_history`** — lịch sử thu hồi số → trigger `SIM_RECLAIMED`

⚠️ **Không có trong `crm`:**
- Tuổi / ngày sinh
- Giới tính
- Nghề nghiệp
- Điểm trung thành / hạng KH

---

### 2.2 Schema `resource` — SIM & Số điện thoại

**Bảng `sims`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `iccid` | Mã ICCID | Định danh SIM |
| `sim_type` | PHYSICAL / ESIM | `{{loai_sim}}` |
| `activation_date` | Ngày kích hoạt SIM | `{{ngay_kich_hoat}}` — mốc NGAY_0 |
| `deactivation_date` | Ngày vô hiệu hóa | Trigger churn |
| `is_esim` | Có phải eSIM không | Phân nhóm |
| `status` | Trạng thái lifecycle | Lọc segment |
| `customer_id` | FK → customers | Join |

**Bảng `msisdns`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `msisdn` | Số điện thoại | Khóa chính segment |
| `expiry_date` | Ngày hết hạn gói | `{{ngay_het_han}}` — trigger DATA_EXPIRING |
| `is_ported` | Đã chuyển mạng vào | Trigger PORT_IN |
| `port_in_date` | Ngày chuyển mạng vào | Payload trigger PORT_IN |
| `status` | Trạng thái | Lọc segment |

**Bảng `number_patterns`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `name` | Tên pattern | `{{loai_so}}` — VD: "Số Viettel", "Số đẹp" |
| `pattern_code` | Mã pattern | Phân loại segment theo loại số |

⚠️ **Không có trong `resource`:**
- Dữ liệu sử dụng (data MB, phút thoại, SMS) — thuộc OCS
- Số dư tài khoản — thuộc OCS
- Tên gói hiện tại — thuộc OCS

---

### 2.3 Schema `notification` — Hạ tầng Gửi tin

**Bảng `notification_channels`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `channel_code` | EMAIL / SMS / PUSH | Định nghĩa kênh |
| `retry_count` | Số lần retry | Cấu hình fallback |
| `timeout_ms` | Timeout | Cấu hình pipeline |
| `is_active` | Bật/tắt kênh | Kill switch per kênh |

⚠️ **Chưa thấy** kênh USSD và Zalo OA trong schema — cần xác nhận với Dev

**Bảng `channel_config_templates`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `template_content` | Nội dung template | CVM Template Engine render |
| `template_variables` | JSONB — danh sách biến | Tham số động |
| `channel_id` | FK → channels | Template per kênh |
| `priority` | Độ ưu tiên | Thứ tự xử lý |

**Bảng `monitor_queues`**

| Trường | Mô tả | Dùng trong CVM |
|---|---|---|
| `pending_messages` | Tin chờ xử lý | System Health Dashboard |
| `failed_messages` | Tin lỗi | Anomaly Detection |
| `status` | HEALTHY / DEGRADED / FAILED | Cảnh báo hệ thống |
| `average_processing_time_ms` | Thời gian xử lý trung bình | Latency chart |

---

### 2.4 Schema `auth` — Phân quyền

| Bảng | Dùng trong CVM |
|---|---|
| `users` | Tài khoản QTV/Admin đăng nhập CVM |
| `roles` + `role_permissions` | RBAC: QTV tạo campaign, Admin duyệt |
| `user_sessions` | Session management CVM web app |

---

### 2.5 Schema `cdr` — Dữ liệu Cước (lịch sử)

- Lưu bản ghi cuộc gọi, SMS, data đã xử lý từ MNO
- **Không phải realtime** — dữ liệu batch, xử lý sau
- Dùng cho: phân tích hành vi, tính toán mức nền để phát hiện tăng đột biến (trigger G3S2.E13)
- ⚠️ Không dùng làm trigger NearRealtime

---

### 2.6 Schema `sales` — Đơn hàng & Voucher

**Phát hiện quan trọng:** BSS đã có sẵn nghiệp vụ voucher (coupons) và campaign sales

**Bảng `coupons`**

| Trường | Mô tả |
|---|---|
| `code` | Mã giảm giá |
| `discount_type` | PERCENTAGE / FIXED_AMOUNT |
| `discount_value` | Giá trị giảm |
| `usage_limit` | Giới hạn sử dụng |
| `start_date` / `end_date` | Hiệu lực |
| `status` | ACTIVE / INACTIVE / EXPIRED / USED_UP |

**Bảng `campaigns`** (Sales campaigns — khác với CVM campaigns)

| Trường | Mô tả |
|---|---|
| `name` | Tên chiến dịch |
| `start_date` / `end_date` | Thời gian |
| `status` | Trạng thái |

**Bảng `orders`** — lịch sử mua SIM → phân tích hành vi mua hàng KH

---

## 3. Bảng tổng hợp: BSS có — BSS không có

| Thông tin cần | BSS có? | Nguồn | Tham số CVM |
|---|---|---|---|
| Họ tên khách hàng | ✅ | `crm.customers.full_name` | `{{ten_kh}}` |
| Số điện thoại thuê bao | ✅ | `crm.subscribers.msisdn` | `{{so_dien_thoai}}` |
| Email khách hàng | ✅ | `crm.customers.contact_email` | `{{email}}` |
| Ngày kích hoạt SIM | ✅ | `resource.sims.activation_date` | `{{ngay_kich_hoat}}` |
| Ngày hết hạn gói | ✅ | `resource.msisdns.expiry_date` | `{{ngay_het_han}}` |
| Loại SIM | ✅ | `resource.sims.sim_type` | `{{loai_sim}}` |
| Loại số (pattern) | ✅ | `resource.number_patterns.name` | `{{loai_so}}` |
| Trạng thái thuê bao | ✅ | `crm.subscribers.status` | `{{trang_thai_thue_bao}}` |
| Đã chuyển mạng vào | ✅ | `resource.msisdns.is_ported` | Trigger PORT_IN |
| Lịch sử mua hàng | ✅ | `sales.orders` | Segment hành vi mua |
| Mã giảm giá / voucher | ✅ | `sales.coupons` | Gắn vào campaign |
| Lịch sử khiếu nại | ✅ | `crm.customer_complaints` | Segment chất lượng dịch vụ |
| **Tuổi / ngày sinh** | ❌ | Không có trong BSS | `{{tuoi}}`, phân khúc tuổi |
| **Giới tính** | ❌ | Không có trong BSS | Trigger ngày lễ 8/3, 20/10 |
| **Nghề nghiệp** | ❌ | Không có trong BSS | Phân khúc nghề nghiệp |
| **Số dư tài khoản** | ❌ | Thuộc OCS | `{{so_du}}` |
| **Data còn lại / đã dùng** | ❌ | Thuộc OCS | `{{data_con_lai}}`, `{{data_da_dung}}` |
| **Tên gói hiện tại** | ❌ | Thuộc OCS | `{{ten_goi}}` |
| **Số lần nạp tiền** | ❌ | Thuộc OCS | `{{so_lan_nap}}` |
| **Số tiền nạp** | ❌ | Thuộc OCS | `{{so_tien_nap}}` |
| **Data ngày đã dùng** | ❌ | Thuộc OCS | `{{data_da_dung_ngay}}` |
| **Hạn mức data ngày** | ❌ | Thuộc OCS | `{{data_quota_ngay}}` |
| **Điểm trung thành** | ❌ | Chưa có schema nào | `{{diem_trung_thanh}}` |
| **Hạng khách hàng** | ❌ | Chưa có schema nào | `{{hang_khach_hang}}` |
| **App install log** | ❌ | Thuộc SuperApp | Trigger E02, E04 |
| **App open log** | ❌ | Thuộc SuperApp | Trigger E03, E04 |
| **Device type / OS** | ❌ | Thuộc SuperApp | `{{loai_thiet_bi}}` |
| **Màn hình đổi gói** | ❌ | Thuộc SuperApp | Trigger E09 |

---

## 4. Khoảng trống — Cần bổ sung từ nguồn ngoài BSS

### Từ OCS (Online Charging System)
Tất cả dữ liệu sử dụng realtime và tài chính:
- Số dư tài khoản, lịch sử nạp tiền
- Data đã dùng / còn lại (ngày + chu kỳ)
- Tên gói, ngày hết hạn chu kỳ gói
- Lý do cuộc gọi thất bại (call_fail_reason)

### Từ SuperApp
Hành vi sử dụng ứng dụng:
- Lịch sử cài đặt / mở app (app_install_log, app_open_log)
- Hành vi trong app (xem màn hình đổi gói, hoàn thành nhiệm vụ)
- Device info (loại thiết bị, OS, Firebase token)

### Chưa rõ nguồn — Cần clarify với PO/Tech
- **Nhân khẩu học** (tuổi, giới tính, nghề nghiệp): KH có tự khai khi đăng ký không? Có hệ thống profile riêng không?
- **Điểm trung thành / hạng KH**: BSS chưa có module loyalty — đây là tính năng mới hay dùng hệ thống ngoài?

---

## 5. Open Questions liên quan đến DB BSS

| # | Câu hỏi | Mức độ |
|---|---|---|
| Q1 | Nhân khẩu học (tuổi, giới tính, nghề nghiệp) lấy từ đâu? BSS có schema nào chưa được cung cấp không? | 🔴 Critical |
| Q2 | USSD và Zalo OA có trong `notification.notification_channels` không? Nếu chưa, ai quản lý 2 kênh này? | 🔴 Critical |
| Q3 | Module loyalty (điểm trung thành, hạng KH) có trong BSS roadmap không, hay CVM tự quản lý? | 🟡 Important |
| Q4 | `sales.campaigns` và CVM campaigns có bị nhầm lẫn không? Cần phân biệt rõ 2 loại campaign này | 🟡 Important |
| Q5 | `crm.customers` và `esim_portal.customers` có sync không? CVM dùng bảng nào làm nguồn chính? | 🟡 Important |
