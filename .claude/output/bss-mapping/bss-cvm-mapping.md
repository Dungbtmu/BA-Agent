# Plan: Mapping DB BSS → CVM System

## Context

Jun yêu cầu đọc toàn bộ tài liệu DB BSS (8 file schema) và phân tích mối quan hệ với dự án CVM (Campaign & Voucher Management). Mục tiêu: hiểu CVM sẽ lấy dữ liệu từ đâu trong BSS, gửi tin qua kênh nào, và segment khách hàng dựa trên field nào — để làm nền tảng cho wireframe + spec tiếp theo.

---

## Tổng quan DB BSS (8 schema)

| Schema | Mục đích | Liên quan CVM |
|---|---|---|
| `auth` | Người dùng, vai trò, phân quyền | QTV/Admin login CVM |
| `crm` | Customers, Subscribers, vòng đời | **Nguồn dữ liệu KH chính** |
| `resource` | MSISDN, SIM, Pattern số | **Nguồn trigger + segment** |
| `notification` | Kênh gửi, template, queue | **Hạ tầng gửi tin** |
| `cdr` | Call Detail Record (ETL log) | Nguồn phân tích hành vi |
| `esim_agency` | Đại lý eSIM, ví, đơn hàng | Ít liên quan trực tiếp |
| `esim_portal` | Portal khách hàng của đại lý | Ít liên quan trực tiếp |
| `travel` | Travel eSIM | Ít liên quan trực tiếp |

---

## Mapping chi tiết: BSS → CVM

### 1. Nguồn dữ liệu Khách hàng (Segment Engine)

CVM cần biết: ai là khách hàng, trạng thái thuê bao, số điện thoại.

| Bảng BSS | Trường quan trọng | Dùng trong CVM |
|---|---|---|
| `crm.customers` | `customer_code`, `full_name`, `contact_phone`, `contact_email`, `status` | Định danh KH, gửi email/push |
| `crm.subscribers` | `msisdn`, `customer_id`, `status`, `activation_date` | Xác định số điện thoại đang active |
| `crm.subscriber_lifecycles` | `lifecycle_number`, `start_date`, `end_date` | Phân tích churn, retention segment |
| `resource.msisdns` | `msisdn`, `customer_id`, `status`, `activation_date`, `expiry_date`, `is_ported` | Trạng thái số, ngày hết hạn gói |
| `resource.number_patterns` | `pattern_code`, `name`, `priority_level` | Phân loại số (Viettel/Mobifone...) |

**→ CVM Segment Engine query từ:** `subscribers JOIN customers JOIN msisdns`

---

### 2. Nguồn Trigger (Trigger Engine)

BSS/OCS push event vào CVM khi có sự kiện. Các bảng BSS sinh ra trigger:

| Sự kiện | Bảng BSS nguồn | Trigger CVM tương ứng |
|---|---|---|
| KH kích hoạt SIM | `resource.sims` (`status` → ACTIVE) | `SIM_ACTIVATED` |
| Số hết hạn gói | `resource.msisdns` (`expiry_date` sắp đến) | `DATA_EXPIRING` |
| KH chuyển mạng vào | `resource.msisdns` (`is_ported=true`, `port_in_date`) | `PORT_IN` |
| KH hủy thuê bao | `crm.subscriber_lifecycles` (`end_date` set) | `SUBSCRIBER_CHURNED` |
| KH đăng ký mới | `crm.subscribers` (record mới tạo) | `NEW_SUBSCRIBER` |
| SIM bị thu hồi | `crm.msisdn_reclaim_history` (record mới) | `SIM_RECLAIMED` |

**Payload trigger** = các trường từ BSS đi kèm event:
- `{{ten_kh}}` ← `crm.customers.full_name`
- `{{so_dien_thoai}}` ← `crm.subscribers.msisdn`
- `{{ngay_kich_hoat}}` ← `resource.sims.activation_date`
- `{{ngay_het_han}}` ← `resource.msisdns.expiry_date`

---

### 3. Kênh gửi tin (Message Dispatcher)

CVM gửi tin qua infrastructure của BSS Notification Service:

| Bảng BSS | Trường quan trọng | Dùng trong CVM |
|---|---|---|
| `notification.notification_channels` | `channel_code` (SMS/EMAIL/PUSH), `retry_count`, `timeout_ms` | Định nghĩa kênh fallback |
| `notification.channel_config_templates` | `template_content`, `template_variables` (JSONB), `channel_id` | Template tin nhắn tái sử dụng |
| `notification.monitor_queues` | `pending_messages`, `failed_messages`, `status` (HEALTHY/DEGRADED) | Dashboard System Health CVM |

**→ CVM Template Engine** render nội dung → gọi BSS Notification API → BSS dispatch qua Gateway

---

### 4. Phân quyền (Auth)

| Bảng BSS | Dùng trong CVM |
|---|---|
| `auth.users` | QTV/Admin đăng nhập CVM |
| `auth.roles` + `auth.role_permissions` | RBAC: QTV chỉ tạo campaign, Admin mới duyệt |
| `auth.user_sessions` | Session management CVM web app |

---

## Sơ đồ quan hệ tổng thể

```
BSS/OCS (nguồn dữ liệu)
│
├── crm.customers ──────────────────────────┐
├── crm.subscribers ─────────────────────── ├──→ CVM Segment Engine
├── resource.msisdns ────────────────────── ┘     (lọc đối tượng nhận tin)
│
├── resource.sims (event: ACTIVE)
├── resource.msisdns (event: expiry_date)
├── crm.subscriber_lifecycles (event: end_date) ──→ CVM Trigger Engine
│                                                    (nhận & match event)
│
├── notification.notification_channels ─────────→ CVM Message Dispatcher
├── notification.channel_config_templates ──────→ CVM Template Engine
└── notification.monitor_queues ────────────────→ CVM Dashboard (System Health)
```

---

## Tham số động (Dynamic Params) — Mapping đầy đủ

| Tham số CVM | Bảng BSS | Trường |
|---|---|---|
| `{{ten_kh}}` | `crm.customers` | `full_name` |
| `{{so_dien_thoai}}` | `crm.subscribers` | `msisdn` |
| `{{email}}` | `crm.customers` | `contact_email` |
| `{{ngay_kich_hoat}}` | `resource.sims` | `activation_date` |
| `{{ngay_het_han}}` | `resource.msisdns` | `expiry_date` |
| `{{loai_sim}}` | `resource.sims` | `sim_type` (PHYSICAL/ESIM) |
| `{{loai_so}}` | `resource.number_patterns` | `name` |
| `{{trang_thai_thue_bao}}` | `crm.subscribers` | `status` |

*Các tham số từ OCS (`{{so_du}}`, `{{data_con_lai}}`, `{{ten_goi_hien_tai}}`) — BSS không có trực tiếp, OCS cần push riêng.*

---

## Cơ chế tích hợp BSS ↔ CVM (đã xác nhận)

**Hybrid — cả 2 cơ chế:**

| Loại | Cơ chế | Dùng cho |
|---|---|---|
| Trigger real-time | **BSS push event** (webhook/event bus) | SIM_ACTIVATED, DATA_EXPIRING, PORT_IN... |
| Dữ liệu KH | **CVM gọi BSS API** | Lấy thông tin KH, trạng thái thuê bao cho Segment Engine |

BSS cần expose các API tối thiểu:
- `GET /customers/{id}` — thông tin KH
- `GET /subscribers?status=ACTIVE` — danh sách thuê bao đang hoạt động
- `GET /subscribers/{msisdn}` — chi tiết 1 thuê bao

---

## Điểm còn cần làm rõ với PO / Dev

1. **Notification Service của BSS và CVM Message Dispatcher có overlap không?**
   - BSS có sẵn `notification_channels` + `channel_config_templates`
   - CVM cần quyết định: dùng chung hay build riêng

4. **`crm.customers` vs `esim_portal.customers` — CVM dùng cái nào?**
   - `crm.customers`: thuê bao viễn thông (core)
   - `esim_portal.customers`: KH mua eSIM qua portal đại lý
   - CVM campaign targeting: có thể cần cả 2

---

## Output dự kiến

Sau khi Jun xác nhận hướng tiếp cận, sẽ tạo:
- Tài liệu **Data Source Mapping** (BSS fields → CVM params) chính thức
- Bổ sung vào **URD/SRS v4**: section tích hợp BSS
- Cập nhật **Wireframe v9**: trigger list + payload hiển thị đúng tên field BSS
