# Wireframe CVM System — v2 (Single Page Campaign Builder)
> Cập nhật: Thiết kế lại Campaign Builder từ stepper 7 bước → single page layout
> Logic nghiệp vụ giữ nguyên, UX thay đổi hoàn toàn

---

## Thay đổi so với v1

| v1 (Stepper) | v2 (Single Page) |
|---|---|
| 7 bước rời nhau | 1 màn hình, 3 vùng song song |
| Phải nhớ đã config bước trước | Toàn bộ hiển thị cùng lúc |
| Không thấy preview khi soạn | Preview realtime cột phải |
| Phải next/back để chỉnh | Chỉnh trực tiếp tại chỗ |
| Dễ bỏ sót kênh nào đó | Kênh + nội dung liền nhau |

---

## Screen 1: Dashboard
*(Giữ nguyên so với v1)*

---

## Screen 2: Campaign List
*(Giữ nguyên so với v1)*

---

## Screen 3: Campaign Builder (Single Page) — UPDATED

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ HEADER (fixed, không scroll)                                        │
├─────────────────────┬───────────────────────┬───────────────────────┤
│ CỘT TRÁI (30%)      │ CỘT GIỮA (40%)        │ CỘT PHẢI (30%)       │
│ scroll độc lập      │ scroll độc lập         │ sticky               │
│                     │                       │                       │
│ TRIGGER             │ KÊNH & TIN NHẮN       │ PREVIEW               │
│ SEGMENT             │ CẤU HÌNH GỬI          │                       │
└─────────────────────┴───────────────────────┴───────────────────────┘
```

---

### Header (fixed)

```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Danh sách Campaign                                                │
│ [Tên campaign *___________________]  [Từ ngày__] → [Đến ngày__ 🗓] │
│ ● Draft                              [Lưu Draft]    [Submit →]     │
└─────────────────────────────────────────────────────────────────────┘
```

- Tên campaign: edit inline, required
- Date range picker: hiệu lực campaign
- Status badge: tự cập nhật (Draft / Pending / Active...)
- [Lưu Draft]: lưu bất cứ lúc nào, không cần điền đủ
- [Submit →]: validate đủ thông tin → confirm dialog → Pending Review

---

### Cột trái — Trigger & Segment (scroll độc lập)

```
┌─────────────────────┐
│ TRIGGER             │
│                     │
│ Logic: [OR ▾]       │
│  OR  = 1 trigger    │
│  AND = tất cả       │
│                     │
│ ┌─────────────────┐ │
│ │ SIM_ACTIVATE  ✕ │ │
│ │ TOP_UP_FAIL   ✕ │ │
│ └─────────────────┘ │
│                     │
│ [+ Thêm trigger]    │
│   ↓ dropdown +      │
│     search          │
│                     │
│ ─────────────────── │
│                     │
│ SEGMENT             │
│                     │
│ ┌─────────────────┐ │
│ │Loại TB=Android ✕│ │
│ │Ngày dùng ≥ 30  ✕│ │
│ │Data ≤ 100 MB   ✕│ │
│ └─────────────────┘ │
│                     │
│ [+ Thêm điều kiện]  │
│   ↓ inline row mới  │
│                     │
│ ─────────────────── │
│ Ước tính:           │
│ ~12,400 KH ↻        │
└─────────────────────┘
```

**Hành vi:**
- [+ Thêm trigger] → dropdown có search, click chọn → thêm tag ngay
- Tag trigger/segment: click ✕ → xóa ngay
- [+ Thêm điều kiện] → append dòng mới inline
- Ước tính tự cập nhật khi thay đổi segment

---

### Cột giữa — Kênh & Tin nhắn + Cấu hình gửi (scroll độc lập)

```
┌───────────────────────┐
│ KÊNH & TIN NHẮN       │
│                       │
│ ┌───────────────────┐ │
│ │⠿ 1  USSD     [✕] │ │  ← kéo thả đổi thứ tự
│ │ Fallback: (kênh   │ │
│ │           đầu)    │ │
│ │ ──────────────── │ │  ← click để expand
│ │ ○ Chọn template   │ │
│ │   [Tìm template▾] │ │
│ │ ● Soạn mới        │ │
│ │ ┌─────────────┐   │ │
│ │ │             │   │ │  ← textarea
│ │ └─────────────┘   │ │
│ │ Tham số:          │ │
│ │ [{{ten_kh}}]      │ │  ← click để insert vào textarea
│ │ [{{so_du}}  ]     │ │
│ └───────────────────┘ │
│                       │
│ ┌───────────────────┐ │
│ │⠿ 2 Zalo OA   [✕] │ │
│ │ Fallback: 5 phút  │ │
│ │ ──────────────── │ │
│ │ ● Chọn template   │ │
│ │ [Chào mừng SIM▾]  │ │
│ └───────────────────┘ │
│                       │
│ ┌───────────────────┐ │
│ │⠿ 3 SMS       [✕] │ │
│ │ Fallback: 10 phút │ │
│ │ ──────────────── │ │
│ │ ○ Chọn template   │ │
│ │ ● Soạn mới        │ │
│ │ ┌─────────────┐   │ │
│ │ │             │   │ │
│ │ └─────────────┘   │ │
│ │ ⚠ 0/160 ký tự    │ │  ← đếm ký tự SMS
│ └───────────────────┘ │
│                       │
│ [+ Thêm kênh]         │
│                       │
│ ─────────────────────  │
│ CẤU HÌNH GỬI          │
│                       │
│ Loại gửi:             │
│ ○ Realtime            │
│   ⚠ Gửi ngay, không   │
│     áp dụng blackout  │
│ ● Near Realtime       │
│   Gửi sau: [10] phút  │
│ ○ Offline             │
│   Gửi lúc: [09:00]    │
│                       │
│ Cooldown: [30] phút   │
│ Khung giờ:            │
│ [08:00] → [22:00]     │
└───────────────────────┘
```

**Hành vi kênh:**
- [⠿] kéo thả đổi thứ tự → fallback tự cập nhật
- Click vào kênh → expand/collapse nội dung
- [✕] xóa kênh → confirm nếu đã có nội dung
- [+ Thêm kênh] → dropdown chỉ hiện kênh chưa chọn
- Chọn template → nội dung điền sẵn, vẫn có thể override
- Click tham số → insert vào vị trí con trỏ trong textarea
- SMS: đếm ký tự realtime, đổi màu đỏ khi > 160

---

### Cột phải — Preview (sticky, không scroll)

```
┌───────────────────────┐
│ PREVIEW               │
│ Kênh: [Zalo OA ▾]    │
│                       │
│ ┌───────────────────┐ │
│ │                   │ │
│ │  Xin chào         │ │
│ │  Nguyễn Văn A,    │ │
│ │  data còn lại     │ │
│ │  1.2 GB.          │ │
│ │  Nạp thêm ngay!   │ │
│ │                   │ │
│ └───────────────────┘ │
│                       │
│ Tham số mẫu:          │
│ ten_kh =              │
│ [Nguyễn Văn A    ]    │
│ so_du =               │
│ [45,000 VNĐ      ]    │
│ data_con_lai =        │
│ [1.2 GB          ]    │
│                       │
│ ⚠ SMS: 142/160 ký tự  │
└───────────────────────┘
```

**Hành vi preview:**
- Dropdown chọn kênh → preview đổi format tương ứng
- Soạn nội dung cột giữa → preview cập nhật realtime
- Tham số mẫu: user tự điền → preview render lại
- SMS: hiện đếm ký tự + cảnh báo khi gần giới hạn

---

## Screen 4: Template Management (New)

```
┌─────────────────────────────────────────────────────────┐
│ TEMPLATE                              [+ Tạo Template]  │
├─────────────────────────────────────────────────────────┤
│ [🔍 Tìm template...]   Trạng thái: [Tất cả ▾]          │
├──────────────────┬──────────────┬────────┬──────────────┤
│ Tên Template     │ Kênh hỗ trợ  │ Dùng   │ Hành động    │
├──────────────────┼──────────────┼────────┼──────────────┤
│ Chào mừng SIM    │ Zalo, SMS    │ 3 lần  │ [Sửa][Clone] │
│ Nạp thẻ hướng dẫn│ SMS, USSD    │ 5 lần  │ [Sửa][Clone] │
│ Sinh nhật KH     │ Zalo, Email  │ 2 lần  │ [Sửa][Clone] │
└──────────────────┴──────────────┴────────┴──────────────┘
```

**Màn hình Tạo / Sửa Template:**

```
┌─────────────────────────────────────────────────────────────────┐
│ ← Template                                                      │
│ [Tên template *_________________]  Trạng thái: ● Active        │
│ [Mô tả___________________________]             [Lưu Template]  │
├─────────────────────────────────┬───────────────────────────────┤
│ SOẠN NỘI DUNG                   │ PREVIEW                       │
│                                 │                               │
│ Tab: [USSD][Zalo OA][SMS]       │ Kênh: [USSD ▾]               │
│      [Banner][Push][Email]      │                               │
│                                 │ ┌─────────────────────────┐  │
│ ── Tab USSD đang active ──      │ │                         │  │
│                                 │ │  Xin chào               │  │
│ ┌─────────────────────────────┐ │ │  Nguyễn Văn A           │  │
│ │                             │ │ │  Data còn: 1.2 GB       │  │
│ │                             │ │ │                         │  │
│ └─────────────────────────────┘ │ └─────────────────────────┘  │
│                                 │                               │
│ Tham số động:                   │ Tham số mẫu:                  │
│ [{{ten_kh}}  ] [{{so_du}}    ]  │ ten_kh = [Nguyễn Văn A  ]    │
│ [{{data_con_lai}}] [{{ten_goi}}]│ so_du  = [45,000 VNĐ    ]    │
│ → click để insert vào nội dung  │                               │
└─────────────────────────────────┴───────────────────────────────┘
```

---

## Screen 5: Trigger Management
*(Giữ nguyên so với v1)*

---

## Screen 6: Customer List → Customer 360
*(Giữ nguyên so với v1)*

---

## Screen 7: Report
*(Giữ nguyên so với v1)*

---

## User Flow tổng (Updated)

```
Login
   ↓
Dashboard
   ├─→ Campaign List
   │      ├─→ [+ Tạo] → Campaign Builder (single page)
   │      │              ├── Cột trái: Trigger + Segment
   │      │              ├── Cột giữa: Kênh + Nội dung + Config
   │      │              └── Cột phải: Preview realtime
   │      └─→ [Sửa]   → Campaign Builder (prefilled)
   │
   ├─→ Template Management
   │      ├─→ [+ Tạo] → Template Editor (2 cột: soạn + preview)
   │      └─→ [Sửa]   → Template Editor (prefilled)
   │
   ├─→ Trigger Management (Admin)
   ├─→ Customer List → Customer 360
   └─→ Report
```
