# Wireframe CVM System — v4
> Cập nhật: Kết hợp layout 2 cột từ giao diện tham khảo + điểm mạnh của v3
> Layout 3 cột → 2 cột. Preview sticky cột phải. Section đánh số. Segment + An toàn vào cột phải.

---

## Thay đổi so với v3

| v3 (3 cột) | v4 (2 cột) |
|---|---|
| 3 cột scroll song song | 2 cột: trái chính + phải phụ+preview |
| Preview chiếm 1 cột riêng | Preview sticky đầu cột phải |
| Segment ở cột trái | Segment chuyển cột phải |
| An toàn gộp vào Cấu hình gửi | An toàn thành vùng riêng ở cột phải |
| Không đánh số section | Section đánh số 1–3 (trái) + 4–5 (phải) |

---

## Screen 1: Dashboard
*(Giữ nguyên)*

---

## Screen 2: Campaign List
*(Giữ nguyên)*

---

## Screen 3: Campaign Builder (2-column layout) — UPDATED

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ HEADER (fixed)                                                      │
├──────────────────────────────────────┬──────────────────────────────┤
│ CỘT TRÁI ~65% (scroll)               │ CỘT PHẢI ~35% (sticky)      │
│                                      │                              │
│  1. Thông tin & Trigger              │  PREVIEW                     │
│  2. Kênh & Tin nhắn                  │  ────────────────────        │
│  3. Cấu hình gửi                     │  4. Phân khúc                │
│                                      │  ────────────────────        │
│                                      │  5. An toàn & Chống Spam     │
└──────────────────────────────────────┴──────────────────────────────┘
```

---

### Header (fixed)

```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Danh sách Campaign                                                │
│ [Tên campaign *___________________]  [Từ ngày__] → [Đến ngày__ 🗓] │
│ ● Draft                              [Lưu Nháp]    [Gửi Duyệt →]  │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Cột trái — Section 1, 2, 3 (scroll)

---

#### Section 1: Thông tin & Trigger

```
┌──────────────────────────────────────────────────────┐
│ 1. Thông tin & Trigger                               │
├──────────────────────────────────────────────────────┤
│ Tên chiến dịch *                                     │
│ [___________________________________]                │
│                                                      │
│ Mã kịch bản (ID)                                     │
│ [___________________________________]                │
│                                                      │
│ Thời gian hiệu lực                                   │
│ Từ ngày [__________]  Đến ngày [__________]          │
│                                                      │
│ ────────────────────────────────────────             │
│                                                      │
│ Chọn Trigger kích hoạt                               │
│ [Chọn trigger... ▾]                                  │
│                                                      │
│  ↓ Khi chọn trigger → vùng payload xuất hiện inline │
│                                                      │
│ ┌──────────────────────────────────────────────────┐ │
│ │ ★ BSS: Kích hoạt SIM thành công      [✕]        │ │
│ │   Tham số:                                       │ │
│ │   [BSS] ten_kh  [BSS] loai_sim                   │ │
│ │   [BSS] ngay_kich_hoat  [BSS] so_dien_thoai      │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ ┌──────────────────────────────────────────────────┐ │
│ │     OCS: Hết dung lượng data         [✕]        │ │
│ │   Tham số:                                       │ │
│ │   [OCS] data_con_lai  [OCS] ten_goi              │ │
│ │   [OCS] ngay_het_han                             │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ [+ Thêm trigger]                                     │
│                                                      │
│ Logic kết hợp:  [OR (Hoặc)]  [AND (Và)]             │
│                                                      │
│ Trigger chính: [BSS: Kích hoạt SIM ▾]               │
│ ⓘ Tham số trong tin nhắn lấy từ trigger này         │
└──────────────────────────────────────────────────────┘
```

**Hành vi:**
- Chọn trigger từ dropdown → card xuất hiện ngay bên dưới với payload
- Badge `★` đánh dấu trigger chính
- Chỉ hiện dropdown "Trigger chính" khi có 2+ trigger
- Logic OR/AND: toggle button, chỉ hiện khi có 2+ trigger
- [✕] xóa trigger → nếu là trigger chính → tự chọn trigger còn lại

---

#### Section 2: Kênh & Tin nhắn

```
┌──────────────────────────────────────────────────────┐
│ 2. Kênh & Tin nhắn                                   │
├──────────────────────────────────────────────────────┤
│ Ngày gửi (T = ngày phát sinh trigger)                │
│ [Ngày T — Gửi ngay ▾]                                │
│                                                      │
│ Khung giờ gửi tin nhắn                               │
│ Từ [08:00]  đến [20:00]                              │
│                                                      │
│ ────────────────────────────────────────             │
│                                                      │
│ Tab kênh: [Zalo OA] [SMS] [USSD] [Banner] [Push]    │
│           [Email]   [+ Thêm kênh]                    │
│                                                      │
│ ── Tab Zalo OA đang active ──                        │
│                                                      │
│ Fallback: (kênh đầu)                                 │
│                                                      │
│ ○ Chọn template có sẵn  [Tìm template... ▾]         │
│ ● Soạn mới                                           │
│                                                      │
│ Hình ảnh (optional):                                 │
│ ┌──────────────────────────────────────────────┐    │
│ │  [🖼 Placeholder]  [Tải ảnh lên]             │    │
│ │  hoặc [Chọn từ thư viện]                     │    │
│ └──────────────────────────────────────────────┘    │
│                                                      │
│ Tham số (BSS: Kích hoạt SIM):                        │
│ [{ten_kh}] [{loai_sim}] [{ngay_kich_hoat}]           │
│ [{so_dien_thoai}]                                    │
│                                                      │
│ Nội dung tin nhắn:                                   │
│ ┌────────────────────────────────────────────────┐   │
│ │                                                │   │
│ │                                                │   │
│ └────────────────────────────────────────────────┘   │
│                                                      │
│ ── Tab SMS ──                                        │
│ Fallback sau: [5] phút                               │
│ (Không hỗ trợ hình ảnh)                              │
│ [Soạn nội dung SMS...]        0 / 160 ký tự          │
│                                                      │
│ ── Tab USSD ──                                       │
│ Fallback sau: [10] phút                              │
│ (Không hỗ trợ hình ảnh)                              │
│ [Soạn nội dung USSD...]                              │
│ ⚠ USSD không hỗ trợ tiếng Việt có dấu               │
│                                                      │
│ ── Tab Banner App/Web ──                             │
│ Fallback sau: [15] phút                              │
│                                                      │
│ Hình ảnh banner (optional):                          │
│ ┌──────────────────────────────────────────────┐    │
│ │  [🖼 Placeholder — 16:9]  [Tải ảnh lên]      │    │
│ │  hoặc [Chọn từ thư viện]                     │    │
│ └──────────────────────────────────────────────┘    │
│ Tiêu đề: [___________________]  0 / 50 ký tự        │
│ Nội dung: [__________________]                       │
│ CTA Button: [________________]                       │
│                                                      │
│ ── Tab Push Notification ──                          │
│ Fallback sau: [5] phút                               │
│                                                      │
│ Hình ảnh thumbnail (optional):                       │
│ ┌──────────────────────────────────────────────┐    │
│ │  [🖼 Placeholder — 1:1]  [Tải ảnh lên]       │    │
│ │  hoặc [Chọn từ thư viện]                     │    │
│ └──────────────────────────────────────────────┘    │
│ Tiêu đề: [___________________]  0 / 65 ký tự        │
│ Nội dung: [__________________]  0 / 240 ký tự       │
│                                                      │
│ ── Tab Email ──                                      │
│ (kênh cuối — không fallback)                         │
│                                                      │
│ Hình ảnh header (optional):                          │
│ ┌──────────────────────────────────────────────┐    │
│ │  [🖼 Placeholder — banner]  [Tải ảnh lên]    │    │
│ │  hoặc [Chọn từ thư viện]                     │    │
│ └──────────────────────────────────────────────┘    │
│ Subject: [_______________________]  0 / 100 ký tự   │
│ Nội dung: [rich text editor]                         │
└──────────────────────────────────────────────────────┘
```

**Hành vi:**
- Tab kênh: click để chuyển kênh, mỗi kênh có nội dung riêng
- [+ Thêm kênh] → dropdown chỉ hiện kênh chưa có tab
- Chip tham số: click → insert vào vị trí con trỏ trong textarea
- Tham số không hợp lệ (không thuộc trigger chính) → highlight đỏ
- Tab đầu tiên = kênh đầu (không có fallback), các tab sau có input fallback delay
- SMS / USSD: không có vùng upload ảnh, hiện label "(Không hỗ trợ hình ảnh)"
- USSD: hiện cảnh báo không hỗ trợ tiếng Việt có dấu
- SMS: đếm ký tự realtime, đỏ khi > 160
- Xóa tab kênh: click [✕] trên tab → confirm nếu đã có nội dung

**Hành vi upload ảnh (áp dụng cho Zalo OA, Banner, Push, Email):**
- [Tải ảnh lên] → mở file picker → upload → hiện thumbnail thay placeholder
- [Chọn từ thư viện] → mở Media Library modal → chọn ảnh đã có → hiện thumbnail
- Khi chưa có ảnh → hiện placeholder grey box có icon 🖼
- Khi đã có ảnh → hiện thumbnail + nút [Xóa ảnh] [Đổi ảnh]
- Tỉ lệ ảnh khuyến nghị theo từng kênh:
  - Zalo OA: tự do
  - Banner App/Web: 16:9
  - Push Notification: 1:1
  - Email header: banner ngang

#### Media Library Modal
```
┌────────────────────────────────────────────────────┐
│ Thư viện ảnh                    [Tải ảnh mới lên]  │
│ [🔍 Tìm kiếm...]                                   │
├────────────────────────────────────────────────────┤
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│ │ 🖼   │ │ 🖼   │ │ 🖼   │ │ 🖼   │               │
│ │img1  │ │img2  │ │img3  │ │img4  │               │
│ └──────┘ └──────┘ └──────┘ └──────┘               │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│ │ 🖼   │ │ 🖼   │ │ 🖼   │ │ 🖼   │               │
│ └──────┘ └──────┘ └──────┘ └──────┘               │
├────────────────────────────────────────────────────┤
│                          [Hủy]  [Chọn ảnh này]    │
└────────────────────────────────────────────────────┘
```

**Hành vi Media Library:**
- Click ảnh → selected state (border xanh + checkmark)
- [Chọn ảnh này] → đóng modal, điền thumbnail vào kênh đang soạn
- [Tải ảnh mới lên] → file picker → upload → thêm vào thư viện + tự động chọn

---

#### Section 3: Cấu hình gửi

```
┌──────────────────────────────────────────────────────┐
│ 3. Cấu hình gửi                                      │
├──────────────────────────────────────────────────────┤
│ Loại gửi:                                            │
│ ○ Realtime                                           │
│   ⚠ Gửi ngay, không áp dụng blackout window         │
│ ● Near Realtime                                      │
│   Gửi sau: [10] phút kể từ trigger                   │
│ ○ Offline                                            │
│   Gửi lúc: [09:00] mỗi ngày                          │
└──────────────────────────────────────────────────────┘
```

---

### Cột phải — Preview + Section 4, 5 (sticky)

---

#### Preview (sticky top)

Preview thay đổi format theo kênh đang chọn:

```
── Zalo OA ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [Zalo OA ▾]           │
│                              │
│ ┌──────────────────────────┐ │
│ │ 🟦 Tên thương hiệu       │ │
│ │ ──────────────────────── │ │
│ │ [🖼 Ảnh đính kèm nếu có] │ │
│ │                          │ │
│ │  Xin chào Nguyễn Văn A,  │ │
│ │  SIM eSIM đã kích hoạt   │ │
│ │  01/04/2026.             │ │
│ └──────────────────────────┘ │
│ chat bubble style            │

── SMS ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [SMS ▾]               │
│                              │
│ ┌──────────────────────────┐ │
│ │ 📱 SMS                   │ │
│ │ ──────────────────────── │ │
│ │ Xin chao Nguyen Van A,   │ │
│ │ SIM cua ban da kich hoat │ │
│ └──────────────────────────┘ │
│ plain text, no image         │
│ ⚠ 68 / 160 ký tự            │

── USSD ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [USSD ▾]              │
│                              │
│ ┌──────────────────────────┐ │
│ │ 📟 USSD                  │ │
│ │ ──────────────────────── │ │
│ │ Chao Nguyen Van A        │ │
│ │ SIM da kich hoat.        │ │
│ │                          │ │
│ │ 1. Xem goi cuoc          │ │
│ │ 0. Thoat                 │ │
│ └──────────────────────────┘ │
│ plain text + menu, no image  │
│ ⚠ Không có dấu tiếng Việt   │

── Banner App/Web ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [Banner ▾]            │
│                              │
│ ┌──────────────────────────┐ │
│ │ [🖼 Ảnh 16:9 nếu có     ]│ │
│ │ [hoặc placeholder grey  ]│ │
│ │ ──────────────────────── │ │
│ │ Tiêu đề: Chào mừng!     │ │
│ │ Nội dung: SIM eSIM của  │ │
│ │ bạn đã kích hoạt.       │ │
│ │ [CTA Button]             │ │
│ └──────────────────────────┘ │

── Push Notification ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [Push ▾]              │
│                              │
│ ┌──────────────────────────┐ │
│ │ [App icon] Tên App  now  │ │
│ │ Tiêu đề: Chào mừng!     │ │
│ │ SIM eSIM của bạn đã...  │ │
│ │              [🖼 1:1 ảnh]│ │
│ └──────────────────────────┘ │
│ Tiêu đề: 18/65              │
│ Nội dung: 32/240            │

── Email ──
┌──────────────────────────────┐
│ XEM TRƯỚC (PREVIEW)          │
│ Kênh: [Email ▾]             │
│                              │
│ ┌──────────────────────────┐ │
│ │ Subject: Chào mừng bạn! │ │
│ │ ──────────────────────── │ │
│ │ [🖼 Header image nếu có] │ │
│ │ [hoặc placeholder grey  ]│ │
│ │                          │ │
│ │ Xin chào Nguyễn Văn A,  │ │
│ │ SIM eSIM của bạn đã kích │ │
│ │ hoạt thành công.         │ │
│ └──────────────────────────┘ │
│ Subject: 22/100              │
```

**Hành vi Preview:**
- Dropdown kênh sync với tab đang active ở cột trái
- Soạn nội dung → preview cập nhật realtime
- Upload ảnh → preview hiện thumbnail ngay
- Khi chưa có ảnh → placeholder grey box có icon 🖼
- Tham số mẫu: user tự điền → preview render lại
- Đếm ký tự hiện dưới preview theo từng kênh:
  - SMS: X / 160
  - Banner tiêu đề: X / 50
  - Push tiêu đề: X / 65, nội dung: X / 240
  - Email subject: X / 100

---

#### Section 4: Phân khúc

```
┌──────────────────────────────┐
│ 4. Phân khúc                 │
├──────────────────────────────┤
│ Chọn Segment động            │
│ [User Gen Z (18-25) ▾]      │
│                              │
│ Hoặc tự định nghĩa:          │
│ [+ Thêm điều kiện]           │
│                              │
│ ┌────────────────────────┐   │
│ │[Loại TB▾][=▾][Android▾]│✕ │
│ │[Ngày dùng▾][≥▾][30]ngày│✕ │
│ └────────────────────────┘   │
│                              │
│ Control Group: [5] %         │
│ ⓘ Giữ lại X% KH không gửi   │
│   để đo Uplift               │
│                              │
│ Ước tính: ~12,400 KH ↻       │
└──────────────────────────────┘
```

---

#### Section 5: An toàn & Chống Spam

```
┌──────────────────────────────┐
│ 5. An toàn & Chống Spam      │
├──────────────────────────────┤
│ Giờ giới nghiêm (Blackout)   │
│ [22:00] - [08:00]            │
│ Xử lý: [Hủy luôn (Discard)▾]│
│                              │
│ Giới hạn nhận tin            │
│ Gửi [1] lần / [30] ngày      │
│                              │
│ ☑ Kiểm tra Blacklist DNC     │
└──────────────────────────────┘
```

---

### User Flow — Tạo Campaign

```
Vào trang → Header: điền tên + ngày
   ↓
Cột trái section 1:
  Chọn trigger → payload hiện inline
  Chọn trigger chính (nếu có 2+)
  Chọn logic OR/AND
   ↓ (song song với section 1)
Cột phải section 4: chọn segment + control group
Cột phải section 5: config blackout + giới hạn
   ↓
Cột trái section 2:
  Chọn tab kênh → soạn/chọn template
  Preview realtime ở cột phải
   ↓
Cột trái section 3: chọn loại gửi
   ↓
[Lưu Nháp] bất cứ lúc nào
[Gửi Duyệt →] → confirm → Pending Review
```

---

## Screen 4: Template Management
*(Giữ nguyên so với v3)*

---

## Screen 5: Trigger Management
*(Giữ nguyên so với v3)*

---

## Screen 6: Blacklist Management (New)

### 6A: Blacklist List

```
┌─────────────────────────────────────────────────────────┐
│ BLACKLIST                                               │
├─────────────────────────────────────────────────────────┤
│ Campaign: [Tất cả ▾]  Kênh: [Tất cả ▾]      [Lọc]    │
├────────────────┬──────────────────┬──────────┬──────────┤
│ Số điện thoại  │ Campaign         │ Kênh     │ Hành động│
├────────────────┼──────────────────┼──────────┼──────────┤
│ 0987 xxx 001   │ Chào mừng SIM    │ Zalo OA  │ [Xóa]   │
│ 0912 xxx 002   │ Hết data         │ SMS      │ [Xóa]   │
│ 0965 xxx 003   │ Hết data         │ Zalo OA  │ [Xóa]   │
└────────────────┴──────────────────┴──────────┴──────────┘
│                        < 1 2 3 >                        │
│                                                         │
│ [+ Thêm thủ công]        [📥 Upload danh sách]         │
└─────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Filter Campaign + Kênh → lọc danh sách realtime
- [Xóa] → confirm dialog → xóa khỏi blacklist ngay
- [+ Thêm thủ công] → mở Modal Thêm
- [📥 Upload danh sách] → mở Modal Upload

---

### 6B: Modal Thêm thủ công

```
┌──────────────────────────────────────┐
│ Thêm vào Blacklist                   │
│                                      │
│ Số điện thoại *                      │
│ [______________________________]     │
│                                      │
│ Campaign *                           │
│ [Chọn campaign... ▾]                │
│                                      │
│ Kênh *                               │
│ [Chọn kênh... ▾]                    │
│ (Zalo OA / SMS / USSD / Banner /    │
│  Push / Email)                       │
│                                      │
│           [Hủy]       [Thêm]        │
└──────────────────────────────────────┘
```

---

### 6C: Modal Upload danh sách

```
┌──────────────────────────────────────┐
│ Upload Blacklist                     │
│                                      │
│ Campaign *                           │
│ [Chọn campaign... ▾]                │
│                                      │
│ Kênh *                               │
│ [Chọn kênh... ▾]                    │
│                                      │
│ File CSV *                           │
│ ┌──────────────────────────────┐    │
│ │  📄 Kéo thả file vào đây    │    │
│ │  hoặc [Chọn file]           │    │
│ │  Format: 1 cột so_dien_thoai│    │
│ │  [📥 Tải file mẫu]          │    │
│ └──────────────────────────────┘    │
│                                      │
│ Kết quả preview (sau khi chọn):     │
│ ✅ 245 số hợp lệ                    │
│ ⚠ 3 số sai định dạng (bỏ qua)      │
│                                      │
│      [Hủy]    [Xác nhận Upload]     │
└──────────────────────────────────────┘
```

**Hành vi:**
- Chọn file → parse ngay, hiện preview kết quả
- Số sai định dạng → bỏ qua, không block upload
- [Xác nhận Upload] → thêm vào blacklist, hiện toast "Đã upload X số"

---

## Screen 7: Customer List
*(Giữ nguyên so với v3)*

---

## Screen 8: Customer 360 (Updated)

```
┌─────────────────────────────────────────────────────────┐
│ ← Danh sách khách hàng                                  │
│ CUSTOMER 360 — 0987 xxx 001                             │
├─────────────────────────────────────────────────────────┤
│ THÔNG TIN KHÁCH HÀNG                                    │
│ Số ĐT: 0987 xxx 001 │ Loại SIM: eSIM │ Trạng thái: Active│
│ Gói: D50 │ Hết hạn: 18/05/2026 │ Ngày KH: 01/04/2026  │
│ Thiết bị: Android │ Cài app: Có │ Đăng nhập app: Có    │
├─────────────────────────────────────────────────────────┤
│ THỐNG KÊ SỬ DỤNG                                        │
│ Data còn lại: 1.2 GB  │ Lưu lượng hôm nay: 320 MB      │
│ Số dư: 45,000 VNĐ    │ Số lần nạp: 3 lần               │
│ Gia hạn gói: 2 lần   │ Cuộc gọi thất bại: 0 lần        │
│ Ngày sinh nhật: 15/08 │ Nghề nghiệp: Giáo viên          │
├─────────────────────────────────────────────────────────┤
│ TRẠNG THÁI KÊNH (Sync-back)                             │
│ ┌──────────────────────────────────────────────────┐   │
│ │ Kênh   │ Trạng thái          │ Cập nhật          │   │
│ ├────────┼─────────────────────┼───────────────────┤   │
│ │Zalo OA │ ⛔ Blocked          │ 05/05/2026 09:32  │   │
│ │SMS     │ ✅ Active            │ —                 │   │
│ │USSD    │ ✅ Active            │ —                 │   │
│ │Push    │ ✅ Active            │ —                 │   │
│ └──────────────────────────────────────────────────┘   │
│ ⓘ Trạng thái tự động cập nhật từ phản hồi Gateway      │
├─────────────────────────────────────────────────────────┤
│ THROTTLING STATUS                                        │
│ Tin nhắn hôm nay: 2/3 │ Cooldown: Không │ BSS DNC: Không│
├─────────────────────────────────────────────────────────┤
│ LỊCH SỬ NHẬN TIN NHẮN                                   │
│ Thời gian   │ Campaign      │ Kênh │ Trạng thái         │
│ 10/05 09:32 │ Chào mừng SIM│ Zalo │ ✓ Delivered        │
│ 08/05 14:10 │ Nạp thẻ lỗi  │ SMS  │ ✓ Delivered        │
│ 07/05 08:45 │ Hết data     │ Zalo │ ✕ Blocked→Fallback │
│ 07/05 08:45 │ Hết data     │ SMS  │ ✓ Delivered        │
└─────────────────────────────────────────────────────────┘
```

---

## Screen 9: Report
*(Giữ nguyên so với v3)*

---

## Edge Cases

| Tình huống | Hành vi |
|---|---|
| Chỉ 1 trigger | Tự động là trigger chính, ẩn dropdown chọn + logic OR/AND |
| Đổi trigger chính | Chip tham số cập nhật, tham số cũ không hợp lệ highlight đỏ |
| Xóa trigger chính | Tự chọn trigger còn lại làm trigger chính |
| Soạn xong mới đổi trigger chính | Giữ nguyên nội dung, chỉ highlight tham số không hợp lệ |
| Thêm kênh đã có tab | Không cho thêm, kênh đó không hiện trong dropdown |
| SMS vượt 160 ký tự | Đếm đỏ + cảnh báo, không block submit |
| Gửi duyệt khi còn trống | Validate: highlight các field bắt buộc còn trống |
