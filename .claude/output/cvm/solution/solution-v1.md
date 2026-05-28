# Solution: CVM System (Customer Value Management)
> Version: v1
> Cập nhật: Bổ sung Template & Tin nhắn quản lý trong CVM

---

## Overview

**CVM (Customer Value Management System)** — Hệ thống quản lý campaign marketing tự động cho viễn thông ảo. CVM đóng vai trò **não điều phối**: nhận trigger từ hệ thống ngoài, quyết định gửi gì, gửi cho ai, gửi qua kênh nào, khi nào.

Bổ sung mới: CVM tự quản lý toàn bộ **Template tin nhắn** — bao gồm soạn thảo, lưu trữ, tái sử dụng và preview trước khi gửi.

---

## Kiến trúc tổng quan (BA level)

```
Hệ thống ngoài (BSS/OCS)
        ↓ push trigger (event) + cung cấp tham số động
      CVM
   ┌──────────────────────────────────────────────────┐
   │  1. Trigger Engine                               │
   │  2. Segment Engine                               │
   │  3. Campaign Engine                              │
   │  4. Template Engine  ← MỚI                      │
   │     - Quản lý template theo kênh                │
   │     - Inject tham số động                       │
   │     - Preview nội dung                          │
   │  5. Throttling & Dedup Engine                    │
   │  6. Message Dispatcher                           │
   └──────────────────────────────────────────────────┘
        ↓ gọi API/Command kèm nội dung đã render
   SMS Gateway / Zalo API / Firebase / USSD Gateway
```

---

## Thiết kế Template Engine

### Cấu trúc 1 Template

```
Template
├── Tên template        (VD: "Chúc mừng sinh nhật")
├── Mô tả               (ghi chú nội bộ)
├── Trạng thái          (Active / Inactive)
├── Nội dung theo kênh:
│   ├── USSD            → text ngắn, menu số
│   ├── Zalo OA         → text dài, có thể kèm hình/button
│   ├── SMS             → text ngắn, giới hạn ký tự
│   ├── Banner App/Web  → tiêu đề + nội dung + CTA
│   ├── Push Notification → tiêu đề + body ngắn
│   └── Email           → subject + body HTML/text
└── Tham số động        → danh sách tham số dùng trong nội dung
```

### Tham số động

Nguồn từ **BSS** (định danh):
- `{{ten_khach_hang}}`
- `{{so_dien_thoai}}`
- `{{loai_sim}}`
- `{{ngay_kich_hoat}}`
- `{{ngay_sinh_nhat}}`

Nguồn từ **OCS** (dữ liệu sử dụng):
- `{{so_du}}`
- `{{data_con_lai}}`
- `{{ten_goi_hien_tai}}`
- `{{ngay_het_han_goi}}`
- `{{so_lan_nap}}`

---

## User Flow (Bổ sung)

### Flow 1: Quản lý Template (màn hình riêng)

```
QTV vào Template Management
   ↓
Xem danh sách template (tên, kênh, trạng thái, lần dùng)
   ↓
[+ Tạo template mới]
   ↓
Nhập tên + mô tả
   ↓
Soạn nội dung theo từng kênh (tab: USSD / Zalo / SMS / ...)
  - Toolbar soạn thảo cơ bản
  - Insert tham số động từ danh sách gợi ý
  - Preview realtime bên phải
   ↓
Lưu → trạng thái Active
   ↓
Template có thể được dùng trong nhiều campaign
```

### Flow 2: Gắn Template vào Campaign (Step mới trong stepper)

```
Stepper tạo campaign cũ:
[1. Thông tin] → [2. Trigger] → [3. Segment]
→ [4. Kênh gửi] → [5. Cấu hình gửi] → [6. Review]

Stepper mới (bổ sung Step 5):
[1. Thông tin] → [2. Trigger] → [3. Segment]
→ [4. Kênh gửi] → [5. Tin nhắn] → [6. Cấu hình gửi] → [7. Review]
```

**Step 5 — Tin nhắn:**

```
Với mỗi kênh đã chọn ở Step 4:

Tab: USSD | Zalo OA | SMS | Banner | Push | Email

Mỗi tab:
  Option A: Chọn template có sẵn
    → Dropdown tìm kiếm template
    → Preview nội dung sau khi chọn
    → Có thể override nội dung cho campaign này

  Option B: Soạn mới (không lưu lại thành template)
    → Soạn trực tiếp trong editor
    → Insert tham số động
    → Preview realtime

Preview panel:
  → Hiển thị nội dung đã render với dữ liệu mẫu
  → Hiển thị đúng format của từng kênh
    (VD: SMS hiện giới hạn ký tự, USSD hiện dạng menu)
```

### Flow 3: Trigger kích hoạt → Render & Gửi

```
Trigger nhận từ BSS/OCS
   ↓
Campaign Engine match trigger → campaign
   ↓
Segment Engine lọc khách hàng
   ↓
Throttling check
   ↓
Template Engine:
  - Load template của campaign theo kênh
  - Lấy tham số động từ BSS (định danh KH)
  - Lấy tham số động từ OCS (dữ liệu KH)
  - Render nội dung hoàn chỉnh
   ↓
Message Dispatcher gửi nội dung đã render → Gateway
   ↓
Ghi log (nội dung đã gửi, tham số đã dùng)
```

---

## Màn hình chính (Cập nhật)

| Màn hình | Role | Mục đích |
|---|---|---|
| Dashboard | QTV, Admin | Tổng quan hệ thống |
| Campaign List | QTV, Admin | Xem, tìm kiếm campaign |
| Campaign Create/Edit | QTV | Tạo campaign (7 bước) |
| **Template Management** | QTV, Admin | Tạo, sửa, quản lý template tái sử dụng |
| Trigger Management | Admin | Thêm, sửa, bật/tắt trigger |
| Customer List | QTV, Admin | Danh sách khách hàng từ BSS |
| Customer 360 | QTV, Admin | Chi tiết 1 khách hàng |
| Report | QTV, Admin | Hiệu quả campaign |
| Throttling Config | Admin | Cấu hình global cap, blackout |

---

## Risks & Dependencies (Bổ sung)

| Risk | Mức độ | Đề xuất |
|---|---|---|
| Tham số động không có giá trị lúc render (KH thiếu data) | HIGH | Định nghĩa giá trị fallback cho từng tham số (VD: {{ten_kh}} fallback = "Quý khách") |
| Template bị sửa sau khi campaign đang chạy | MEDIUM | Lưu snapshot nội dung template tại thời điểm campaign Active |
| Nội dung SMS vượt giới hạn ký tự sau khi inject tham số | MEDIUM | Validate độ dài sau render, cảnh báo khi soạn |
| USSD chỉ hỗ trợ text thuần, không nhận HTML | LOW | Validate format theo kênh khi soạn |

---

## Assumptions

- QTV soạn nội dung cho tất cả kênh đã chọn ở Step 4 — không được bỏ trống kênh nào
- Preview dùng dữ liệu mẫu (mock), không dùng dữ liệu khách hàng thật
- Template được lưu version — khi edit tạo version mới, campaign cũ giữ nguyên version cũ
- Giới hạn ký tự SMS: 160 ký tự / tin (hiển thị đếm realtime khi soạn)
