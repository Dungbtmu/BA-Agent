# Wireframe CVM System — v5
> Cập nhật: Trigger Management + Campaign Builder Section 4 — lifecycle stage, priority, campaign linked, segment taxonomy
> Thay đổi so với v4: Screen 5 Trigger Management (cột mới + modal mới), Section 4 Campaign Builder (phân tách segment sẵn + điều kiện bổ sung)

---

## Thay đổi so với v4

| v4 | v5 |
|---|---|
| Trigger Management: 3 cột (Tên, Source, Trạng thái) | Thêm cột: Giai đoạn VĐ, Ưu tiên, Campaign Linked |
| Không có banner quy tắc ưu tiên | Thêm info bar quy tắc ưu tiên |
| Modal trigger: không có giai đoạn và priority | Thêm dropdown Giai đoạn VĐ + input Priority |
| Không có modal Campaign Linked | Thêm modal drill-down campaign |
| Section 4: dropdown segment đơn + condition builder | Phân tách: multi-select segment sẵn (A) + condition builder bổ sung (B) |

---

## Screen 1: Dashboard
*(Giữ nguyên)*

---

## Screen 2: Campaign List

### Layout cập nhật

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Campaign                                             [+ Tạo Campaign]        │
├──────────────────────────────────────────────────────────────────────────────┤
│ [Tìm kiếm tên campaign...]  Trạng thái [Tất cả ▾]  Trigger [Tất cả ▾]       │
├──────────────────────────────────────────────────────────────────────────────┤
│ Campaign Name        │ Trigger codes             │ Logic │ Trạng thái │ Action│
├──────────────────────┼───────────────────────────┼───────┼────────────┼───────┤
│ Welcome eSIM Q2/2026 │ [SIM_ACTIVATED]           │ OR    │ ● Draft    │ [Xem] │
│                      │ [NO_APP_INSTALL_24H]      │       │            │ [Sửa] │
│                      │ [LOW_DATA_BALANCE]        │       │            │       │
├──────────────────────┼───────────────────────────┼───────┼────────────┼───────┤
│ Nhắc nạp tiền        │ [TOP_UP_FAIL]             │ Basic │ ● Active   │ [Xem] │
├──────────────────────┼───────────────────────────┼───────┼────────────┼───────┤
│ Hết hạn gói data     │ [PACKAGE_EXPIRED]         │ Basic │ ● Paused   │ [Xem] │
└──────────────────────┴───────────────────────────┴───────┴────────────┴───────┘
```

**Hành vi:**
- Cột `Trigger codes` hiển thị tối đa 3 mã trigger dạng badge để người dùng scan nhanh campaign đang dùng trigger nào.
- Nếu campaign có nhiều hơn 3 trigger, hiển thị `+N` badge. Hover/click `+N` mở popover danh sách đầy đủ trigger.
- Hover từng trigger code hiển thị tooltip: tên trigger, source, kiểu chạy.
- Click trigger code mở quick view trigger hoặc filter sang màn Trigger Management theo mã đó.
- Cột `Logic` hiển thị `Basic`, `OR`, hoặc `AND` để người dùng biết campaign đang dùng 1 trigger hay advanced multi-trigger.
- Nút `[Xem]` mở Campaign Detail View chỉ đọc, giúp user xem nhanh toàn bộ thông tin khai báo mà không vào chế độ sửa.

---

## Screen 3: Campaign Builder (Smart UX Revision) — UPDATED after demo 11/05/2026

> Mục tiêu cập nhật: giữ khả năng 1 campaign có 1 hoặc nhiều trigger, nhưng làm UI thông minh hơn bằng Basic/Advanced mode, message matrix theo Trigger x Segment x Channel, lịch gửi theo trigger, channel strategy riêng và cột phải sticky để kiểm soát thiếu sót/rủi ro.

### Layout tổng quan

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ HEADER FIXED                                                                 │
├──────────────────────────────────────────────┬───────────────────────────────┤
│ CỘT TRÁI ~68% (scroll)                        │ CỘT PHẢI ~32% (sticky)       │
│                                              │                               │
│ 1. Thông tin Campaign                         │ Campaign Summary             │
│ 2. Trigger & Logic                            │ Validation Checklist         │
│ 3. Audience / Phân khúc                       │ Cost & SMS Risk              │
│ 4. Message Matrix                             │ Approval Status              │
│ 5. Channel Strategy                           │                               │
│ 6. Cấu hình gửi theo Trigger                  │                               │
│ 7. Review & Submit                            │                               │
└──────────────────────────────────────────────┴───────────────────────────────┘
```

---

### Header fixed

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ ← Danh sách Campaign                                                         │
│ Welcome eSIM Q2/2026                                      ● Draft            │
│ CVM-WELCOME-ESIM-Q2-2026                                  [Lưu Nháp] [Gửi duyệt →]
└──────────────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Header luôn sticky để QTV không mất context khi scroll.
- Nút `Gửi duyệt` chỉ enabled khi validation checklist không còn lỗi blocking.
- Campaign chỉ chuyển Active sau khi Admin duyệt.

---

### Section 1: Thông tin Campaign

```
┌────────────────────────────────────────────────────────────────────┐
│ 1. Thông tin Campaign                                              │
├────────────────────────────────────────────────────────────────────┤
│ Tên campaign *                                                     │
│ [Welcome eSIM Q2/2026___________________________________________]  │
│                                                                    │
│ Mã campaign / kịch bản                                             │
│ [CVM-WELCOME-ESIM-Q2-2026_______________________________________]  │
│                                                                    │
│ Mục tiêu campaign                                                  │
│ [Onboard thuê bao eSIM mới, nhắc cài app, nhắc mua thêm data____]  │
│                                                                    │
│ Thời gian hiệu lực                                                 │
│ Từ ngày [01/04/2026]  Đến ngày [30/06/2026]                        │
│                                                                    │
│ Người tạo: QTV Marketing        Trạng thái: Draft                  │
└────────────────────────────────────────────────────────────────────┘
```

---

### Section 2: Trigger & Logic

```
┌────────────────────────────────────────────────────────────────────┐
│ 2. Trigger & Logic                                                 │
├────────────────────────────────────────────────────────────────────┤
│ Chế độ trigger                                                     │
│ ○ Basic — 1 trigger chính                                          │
│ ● Advanced — nhiều trigger + AND/OR rõ ràng                        │
│                                                                    │
│ [+ Thêm trigger]                                                   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ★ T1 · SIM_ACTIVATED                                           │ │
│ │ BSS · Kích hoạt SIM thành công                                 │ │
│ │ Kiểu chạy: Realtime · gửi ngay khi event đến                    │ │
│ │ Payload: [ten_kh] [loai_sim] [ngay_kich_hoat] [so_dien_thoai]  │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │   T2 · NO_APP_INSTALL_24H                                      │ │
│ │ CRM · Chưa cài app sau 24h                                     │ │
│ │ Kiểu chạy: Near realtime · sau 24h                              │ │
│ │ Payload: [app_installed] [last_active]                          │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │   T3 · LOW_DATA_BALANCE                                        │ │
│ │ OCS · Data còn dưới 100MB                                      │ │
│ │ Kiểu chạy: Offline / Batch · 09:00 hằng ngày                    │ │
│ │ Payload: [data_con_lai] [ten_goi] [ngay_het_han]                │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ Logic xử lý khi có nhiều trigger                                   │
│ ● OR — KH match trigger nào thì gửi message của trigger đó         │
│ ○ AND — KH phải thỏa tất cả trigger, gửi 1 message chung           │
│                                                                    │
│ Diễn giải OR                                                       │
│ Nếu KH kích hoạt SIM      → gửi Message Welcome                    │
│ Nếu KH chưa cài app 24h   → gửi Message Chưa cài app               │
│ Nếu KH còn data < 100MB   → gửi Message Sắp hết data               │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Basic mode chỉ cho chọn 1 trigger, ẩn AND/OR để giảm độ rối.
- Advanced mode cho nhiều trigger và bắt buộc chọn AND/OR.
- Mỗi trigger hiển thị rõ source, kiểu chạy, payload và trạng thái mapping message.
- Lịch xử lý ban đầu phụ thuộc trigger, không phụ thuộc campaign-level schedule.

---

### Section 3: Audience / Phân khúc

```
┌────────────────────────────────────────────────────────────────────┐
│ 3. Audience / Phân khúc                                            │
├────────────────────────────────────────────────────────────────────┤
│ Nguồn phân khúc: Customer 360 / Team Data / BSS / OCS              │
│ Cập nhật lần cuối: 12/05/2026 08:00                                │
│                                                                    │
│ [Tìm phân khúc...]                                                 │
│                                                                    │
│ Danh sách phân khúc                                                │
│ ☑ Gen Z User (18–25)                         18,450 KH             │
│ ☐ Thuê bao mới kích hoạt                      32,100 KH             │
│ ☑ Sắp hết data                                 8,920 KH             │
│ ☐ Chưa cài app MyViettel                      14,300 KH             │
│ ☐ ARPU cao                                     5,600 KH             │
│ ☐ Nguy cơ rời mạng                             2,150 KH             │
│                                                                    │
│ Đã chọn: [Gen Z User ×] [Sắp hết data ×]                           │
│                                                                    │
│ Logic phân khúc                                                     │
│ ● Thuộc bất kỳ phân khúc nào (OR)                                  │
│ ○ Thuộc tất cả phân khúc đã chọn (AND)                              │
│                                                                    │
│ Điều kiện lọc bổ sung (optional)                                   │
│ [Loại thiết bị] [=] [Android] [×]                                   │
│ [+ Thêm điều kiện]                                                  │
│                                                                    │
│ Ước tính reach: ~6,800 KH ↻                                        │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- CVM không tạo segment master trong phase này; segment được Team Data/BSS/OCS cung cấp.
- Customer 360 hiển thị một khách hàng thuộc segment nào; Campaign Builder hiển thị toàn bộ segment để QTV chọn.
- Điều kiện lọc bổ sung chỉ là filter trên tập segment đã chọn, không phải rule builder để tạo segment mới.

---

### Section 4: Message Matrix

```
┌────────────────────────────────────────────────────────────────────┐
│ 4. Message Matrix                                                  │
├────────────────────────────────────────────────────────────────────┤
│ Message model: Trigger × Segment × Channel                         │
│ Advanced OR: mỗi trigger có message riêng trên từng kênh            │
│                                                                    │
│ Tiến độ nội dung                                                    │
│ Push: 3/3 ✓   Zalo: 3/3 ✓   SMS: 2/3 ⚠   Email: 0/3                │
│                                                                    │
│ Tab kênh: [Push] [Zalo OA] [SMS] [Banner] [Email] [+ Thêm kênh]    │
│                                                                    │
│ ── Tab Push đang active ─────────────────────────────────────────  │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T1 · SIM_ACTIVATED                                           │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Welcome Push Template ▾]                             │ │
│ │ Title: Chào mừng bạn!                                          │ │
│ │ Body: SIM eSIM đã kích hoạt thành công. Tải MyViettel...       │ │
│ │ Preview: [Push notification preview]                            │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T2 · NO_APP_INSTALL_24H                                      │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Install App Reminder ▾]                              │ │
│ │ Title: Cài MyViettel để nhận ưu đãi                             │ │
│ │ Body: Bạn vẫn chưa cài app MyViettel...                         │ │
│ │ Preview: [Push notification preview]                            │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T3 · LOW_DATA_BALANCE                                        │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Low Data Gen Z Push ▾]                               │ │
│ │ Title: Sắp hết data rồi 😮                                      │ │
│ │ Body: Bạn chỉ còn {{data_con_lai}} MB. Mua thêm gói...         │ │
│ │ [+ Thêm biến thể đối tượng]                                     │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ── Khi chuyển sang tab SMS ──────────────────────────────────────  │
│ [T1] SMS Welcome                  ✓  92/160 ký tự                  │
│ [T2] SMS Chưa cài app             ✓  118/160 ký tự                 │
│ [T3] SMS Sắp hết data             ⚠ Chưa cấu hình                  │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- QTV thao tác theo kênh trước, sau đó cấu hình message theo trigger trong kênh đó.
- Với OR: thuê bao match trigger nào thì gửi message tương ứng của trigger đó trên kênh đang xử lý.
- Với AND: mỗi kênh chỉ còn 1 message chung, nhưng vẫn là message riêng theo từng kênh.
- Cùng một trigger có thể có nhiều audience variant nếu các phân khúc cần thông điệp khác nhau.
- Template Library được hỗ trợ nghiêm túc hơn nhưng không khóa cứng nhập tự do ở phase này; campaign approval sẽ duyệt toàn bộ nội dung campaign.

---

### Section 5: Channel Strategy

```
┌────────────────────────────────────────────────────────────────────┐
│ 5. Channel Strategy                                                │
├────────────────────────────────────────────────────────────────────┤
│ Mục tiêu: ưu tiên kênh rẻ trước, SMS là kênh cuối                  │
│                                                                    │
│ Thứ tự kênh gửi                                                     │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ 1. Push App                                                    │ │
│ │    Gửi theo lịch của trigger                                   │ │
│ │    Chờ phản hồi: [30] phút                                     │ │
│ │    Nếu KH [Clicked ▾] → dừng fallback                          │ │
│ └────────────────────────────────────────────────────────────────┘ │
│          ↓ nếu chưa clicked / fail                                 │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ 2. Zalo OA / ZNS                                               │ │
│ │    Gửi sau Push [30] phút nếu chưa clicked                     │ │
│ │    Nếu [Delivered hoặc Clicked ▾] → dừng fallback              │ │
│ └────────────────────────────────────────────────────────────────┘ │
│          ↓ nếu fail / không phản hồi                               │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ 3. SMS                                                         │ │
│ │    Kênh cuối, chỉ gửi nếu các kênh trước không đạt điều kiện   │ │
│ │    Ước tính SMS: ~1,200 tin                                    │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ☑ Không gửi SMS nếu KH đã click/open ở kênh trước                  │
│ Điều kiện dừng fallback: ☐ Delivered  ☐ Opened  ☑ Clicked          │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Trigger xác định thời điểm bắt đầu xử lý.
- Channel Strategy xác định thứ tự kênh, thời gian chờ và điều kiện dừng fallback.
- Safety rule xác định có được gửi tại thời điểm dự kiến hay phải queue/chặn.

---

### Section 6: Cấu hình gửi theo Trigger

```
┌────────────────────────────────────────────────────────────────────┐
│ 6. Cấu hình gửi theo Trigger                                       │
├────────────────────────────────────────────────────────────────────┤
│ ⓘ Thời điểm gửi phụ thuộc vào trigger; không có lịch gửi chung     │
│                                                                    │
│ T1 · SIM_ACTIVATED                                                 │
│ Kiểu chạy: Realtime                                                │
│ Xử lý: ngay khi nhận event hợp lệ                                  │
│ Blackout: queue đến 08:00 nếu rơi 20:00–08:00                     │
│                                                                    │
│ T2 · NO_APP_INSTALL_24H                                            │
│ Kiểu chạy: Near realtime                                           │
│ Delay: 24h sau SIM_ACTIVATED                                       │
│ Điều kiện: app_installed = false                                   │
│ Blackout: queue đến 08:00 nếu rơi 20:00–08:00                     │
│                                                                    │
│ T3 · LOW_DATA_BALANCE                                              │
│ Kiểu chạy: Offline / Batch                                         │
│ Lịch quét: 09:00 hằng ngày                                         │
│ Điều kiện: data_con_lai <= 100MB                                   │
│ Blackout: không chạy batch trong 20:00–08:00                       │
└────────────────────────────────────────────────────────────────────┘
```

---

### Section 7: Review & Submit

```
┌────────────────────────────────────────────────────────────────────┐
│ 7. Review & Submit                                                 │
├────────────────────────────────────────────────────────────────────┤
│ Checklist trước khi gửi duyệt                                      │
│ ✓ Có ít nhất 1 trigger                                             │
│ ✓ Đã chọn phân khúc                                                │
│ ⚠ SMS còn thiếu message cho T3 LOW_DATA_BALANCE                    │
│ ✓ Có channel strategy                                              │
│ ✓ Có blacklist / DNC check                                         │
│ ✓ Có frequency cap                                                 │
│                                                                    │
│ Ghi chú gửi duyệt                                                  │
│ [Campaign onboard eSIM Q2, ưu tiên Push/Zalo trước SMS__________]  │
│                                                                    │
│ [Lưu Nháp]                                      [Gửi duyệt →]      │
└────────────────────────────────────────────────────────────────────┘
```

---

### Cột phải sticky

```
┌──────────────────────────────┐
│ Campaign Summary             │
├──────────────────────────────┤
│ Trigger                       │
│ Advanced · OR · 3 triggers    │
│                              │
│ Audience                      │
│ 2 segments · ~6,800 KH        │
│                              │
│ Message                       │
│ Push: 3/3 ✓                  │
│ Zalo: 3/3 ✓                  │
│ SMS: 2/3 ⚠                  │
│ Email: 0/3                   │
│                              │
│ Channel Strategy              │
│ Push → Zalo → SMS             │
│ Stop fallback: Clicked        │
│                              │
│ Risk / Cost                   │
│ Estimated SMS: ~1,200         │
│ Estimated cost: ~360,000 VND  │
│                              │
│ Approval                      │
│ Draft                         │
│ Admin duyệt để Active         │
└──────────────────────────────┘
```

**Hành vi cột phải:**
- Sticky khi scroll để QTV luôn thấy campaign thiếu gì.
- Click vào item lỗi, ví dụ `SMS: 2/3`, scroll tới đúng message còn thiếu.
- Cost/SMS risk cập nhật khi thay đổi channel strategy, segment hoặc fallback condition.

---

### User Flow — Tạo Campaign

```
1. Nhập thông tin campaign
   ↓
2. Chọn Basic hoặc Advanced trigger mode
   ↓
3. Chọn trigger, xác định AND/OR nếu Advanced
   ↓
4. Chọn phân khúc từ Customer 360 / Team Data
   ↓
5. Cấu hình Message Matrix theo Trigger × Segment × Channel
   ↓
6. Cấu hình Channel Strategy: Push → Zalo → SMS
   ↓
7. Kiểm tra lịch gửi theo từng trigger
   ↓
8. Xem sticky summary / validation / SMS cost risk
   ↓
9. Lưu nháp hoặc gửi Admin duyệt
```

---

## Screen 4: Template Management
*(Giữ nguyên so với v3)*

---

## Screen 5: Trigger Management — UPDATED v5

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ TRIGGER MANAGEMENT                              [+ Thêm Trigger]    │
├─────────────────────────────────────────────────────────────────────┤
│ Trạng thái: [Tất cả ▾]    Giai đoạn VĐ: [Tất cả ▾]                │
├─────────────────────────────────────────────────────────────────────┤
│ ⚠ INFO BAR (màu vàng nhạt — yellow-50)                              │
│ Quy tắc ưu tiên khi xung đột: T-LOC-2 (Di chuyển GT) >             │
│ T-LOC-1 (Du lịch) > T-JOB > T-AGE                                  │
│ Khi KH nhận 2 trigger cùng lúc, chỉ campaign có priority cao nhất  │
│ được gửi.                                                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Bảng danh sách Trigger

```
┌──────┬────────────────────────────┬──────────────────────┬──────────────────┬───────────┬────────────────┬───────────┬─────────────┐
│ Mã   │ Tên Trigger                │ Event Source         │ Giai đoạn VĐ     │ Ưu tiên   │ Campaign Linked│ Trạng thái│ Hành động   │
├──────┼────────────────────────────┼──────────────────────┼──────────────────┼───────────┼────────────────┼───────────┼─────────────┤
│ T1   │ Kích hoạt SIM thành công   │ BSS SIM_ACTIVATED    │ [G3 Early Use]   │ ★★★★★ P1 │ [Welcome eSIM] │ ● Active  │ [Sửa][Tắt] │
│ T2   │ Chưa cài app sau 24h       │ CRM NO_APP_INSTALL_24H│ [G3 Early Use]  │ ★★★★ P2  │ [Welcome eSIM] │ ● Active  │ [Sửa][Tắt] │
│ T3   │ Data còn dưới 100MB        │ OCS LOW_DATA_BALANCE │ [G3 Early Use]   │ ★★★★ P2  │ [Welcome eSIM] │ ● Active  │ [Sửa][Tắt] │
│ G5   │ T-LOC-1 · Đến tỉnh du lịch │ OCS LOC_TRAVEL_PROV  │ [G4 Tăng trưởng] │ ★★★★★ P2 │ [1 campaign]   │ ● Active  │ [Sửa][Tắt] │
│ D1   │ Usage giảm >40% vs baseline│ OCS USAGE_DROP_40PCT │ [G5 Suy giảm]    │ ★★★ P3   │ [0 campaigns]  │ ○ Testing │ [Sửa][Bật] │
│ W1   │ SIM ngừng GD >30 ngày      │ BSS SIM_NO_TXN_30D   │ [G6 Win-back]    │ ★★★★ P2  │ [1 campaign]   │ ○ Paused  │ [Sửa][Bật] │
└──────┴────────────────────────────┴──────────────────────┴──────────────────┴───────────┴────────────────┴───────────┴─────────────┘
```

**Chú thích badge Giai đoạn VĐ:**
- G3 Early Use → badge xanh dương (blue)
- G4 Tăng trưởng → badge xanh lá (green)
- G5 Suy giảm → badge cam (orange)
- G6 Win-back → badge tím (purple)

**Chú thích cột Ưu tiên:**
- Hiển thị số P1/P2/P3 kèm sao (★): P1 = ★★★★★, P2 = ★★★★, P3 = ★★★
- Số nhỏ hơn = ưu tiên cao hơn (P1 > P2 > P3)

**Hành vi:**
- Filter "Giai đoạn VĐ" → lọc bảng theo lifecycle stage
- Filter "Trạng thái" → giữ nguyên v4
- Click số campaign trong cột "Campaign Linked" (dạng link text) → mở Modal Campaign Linked
- [Tắt/Bật]: confirm dialog trước khi toggle

---

### Modal Thêm/Sửa Trigger — UPDATED v5

```
┌────────────────────────────────────────────────────────────┐
│ Thêm Trigger Mới                                           │
├────────────────────────────────────────────────────────────┤
│ Tên trigger *                                              │
│ [_________________________________________]                │
│                                                            │
│ Event source *                                             │
│ [BSS / OCS / CRM / Other ▾]                                │
│                                                            │
│ Mô tả                                                      │
│ [_________________________________________]                │
│                                                            │
│ Giai đoạn vòng đời *                                       │
│ [Chọn giai đoạn... ▾]                                      │
│   G3 — Early Use (Ngày 0–30)                               │
│   G4 — Tăng trưởng (Tháng 2+)                             │
│   G5 — Suy giảm                                            │
│   G6 — Win-back                                            │
│                                                            │
│ Độ ưu tiên *                                               │
│ [___]  ⓘ Số nguyên dương. Số nhỏ hơn = ưu tiên cao hơn.  │
│   Khi KH nhận nhiều trigger cùng lúc, chỉ campaign của    │
│   trigger có số ưu tiên nhỏ nhất được gửi.                 │
│                                                            │
│ Payload                                                    │
│ ┌──────────────────────────────────────────────────────┐  │
│ │ [Tên tham số *_______] [BSS / OCS / CRM ▾] [✕]   │  │
│ │ [Tên tham số *_______] [BSS / OCS / CRM ▾] [✕]   │  │
│ │ [+ Thêm tham số]                                     │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                            │
│ Trạng thái: ○ Active  ○ Inactive                           │
│                                                            │
│                              [Hủy]        [Lưu]           │
└────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Tất cả field có dấu * là bắt buộc
- Độ ưu tiên: chỉ nhận số nguyên dương; validate inline nếu nhập chữ hoặc số âm
- [Lưu] → đóng modal + cập nhật bảng + toast "Đã lưu trigger ✓"

---

### Modal Campaign Linked — NEW v5

Mở khi click số campaign trong cột "Campaign Linked".

```
┌────────────────────────────────────────────────────┐
│ Campaign đang dùng trigger "Chào mừng kích hoạt SIM"│
│ 2 campaign                                         │
├────────────────────────┬────────────┬──────────────┤
│ Tên campaign           │ Trạng thái │ Hiệu lực     │
├────────────────────────┼────────────┼──────────────┤
│ Chào mừng SIM          │ ● Active   │ 04/05–18/05  │
│ Ưu đãi KH mới          │ ● Active   │ Liên tục     │
└────────────────────────┴────────────┴──────────────┘
│ ⚠ Tắt trigger này sẽ ảnh hưởng 2 campaign Active  │
│                                          [Đóng]    │
└────────────────────────────────────────────────────┘
```

Ví dụ với trigger "T-LOC-1 · Đến tỉnh du lịch" (1 campaign):
```
┌────────────────────────────────────────────────────┐
│ Campaign đang dùng trigger "T-LOC-1 · Đến tỉnh"   │
│ 1 campaign                                         │
├────────────────────────┬────────────┬──────────────┤
│ Tên campaign           │ Trạng thái │ Hiệu lực     │
├────────────────────────┼────────────┼──────────────┤
│ Ưu đãi du lịch nội địa │ ● Active   │ 01/05–31/05  │
└────────────────────────┴────────────┴──────────────┘
│ ⚠ Tắt trigger này sẽ ảnh hưởng 1 campaign Active  │
│                                          [Đóng]    │
└────────────────────────────────────────────────────┘
```

Ví dụ với trigger "D1 — Usage giảm >40%" (0 campaigns):
```
┌────────────────────────────────────────────────────┐
│ Campaign đang dùng trigger "Usage giảm >40%"       │
│ 0 campaign                                         │
├────────────────────────────────────────────────────┤
│ Chưa có campaign nào sử dụng trigger này.          │
└────────────────────────────────────────────────────┘
│                                          [Đóng]    │
└────────────────────────────────────────────────────┘
```

**Hành vi:**
- Click số "N campaigns" dạng link text → mở modal này
- Nếu có ít nhất 1 campaign Active → hiện cảnh báo màu đỏ/vàng phía dưới bảng
- Nếu 0 campaigns Active → không hiện cảnh báo, chỉ hiện thông báo chưa có campaign
- [Đóng] → đóng modal, không có action nào khác

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
| Tắt trigger có Active campaign | Hiện cảnh báo trong modal Campaign Linked trước khi tắt |
| Trigger priority trùng nhau | Hệ thống xử lý theo thứ tự ID, admin cần được cảnh báo khi lưu |
| Chọn 0 segment, 0 điều kiện | Mặc định gửi T-ALL (tất cả KH), hiện thông báo xác nhận |
| Segment OR + điều kiện AND | Ước tính cập nhật realtime khi thay đổi bất kỳ điều kiện nào |
