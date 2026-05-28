# Wireframe CVM System — v6
> Cập nhật: Campaign Builder — conflict chỉ áp dụng cho OR, thêm Section 5 An toàn & Chống Spam, thêm Section 6 Cấu hình gửi theo Trigger, bỏ Review & Submit
> Thay đổi so với v5: tách lịch gửi trigger khỏi Section 2, bổ sung whitelist targeting, bỏ section review cuối form

---

## Thay đổi so với v4

| v5 | v6 |
|---|---|
| Conflict trigger hiển thị chung trong Advanced mode | Chỉ hiển thị khi Advanced logic = OR |
| Priority trigger trong campaign có mô tả nhưng chưa rõ thao tác | Cho kéo-thả hoặc chọn P1/P2/P3 trên từng trigger trong campaign |
| Lịch xử lý trigger là accordion trong Section 2 | Tách thành Section 6 riêng: Cấu hình gửi theo Trigger |
| Section 6 là Review & Submit | Bỏ Review & Submit khỏi Campaign Builder |
| Chưa thể hiện whitelist targeting trong form campaign | Thêm vùng Whitelist trong Section 5 An toàn & Chống Spam |

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

> Mục tiêu cập nhật: giữ khả năng 1 campaign có 1 hoặc nhiều trigger, nhưng làm UI thông minh hơn bằng Basic/Advanced mode, message matrix theo Trigger x Segment x Channel, an toàn chống spam, lịch gửi theo trigger và cột phải sticky để kiểm soát thiếu sót/rủi ro.

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
│ 5. An toàn & Chống Spam                       │                               │
│ 6. Cấu hình gửi theo Trigger                  │                               │
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
│                                                                    │
│ Xử lý xung đột trigger — chỉ hiển thị khi Logic = OR                │
│ ● Chỉ gửi trigger ưu tiên cao nhất                                 │
│ ○ Gửi tất cả trigger đã match (không khuyến nghị)                  │
│                                                                    │
│ Ưu tiên trigger trong campaign                                      │
│ [≡] T3 · LOW_DATA_BALANCE      [P1 ▾]  Ưu tiên cao nhất            │
│ [≡] T2 · NO_APP_INSTALL_24H    [P2 ▾]                              │
│ [≡] T1 · SIM_ACTIVATED         [P3 ▾]                              │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Basic mode chỉ cho chọn 1 trigger, ẩn AND/OR để giảm độ rối.
- Advanced mode cho nhiều trigger và bắt buộc chọn AND/OR.
- Mỗi trigger hiển thị rõ source, kiểu chạy, payload và trạng thái mapping message.
- Lịch xử lý ban đầu phụ thuộc trigger, không phụ thuộc campaign-level schedule.
- Khối `Xử lý xung đột trigger` chỉ xuất hiện khi Advanced logic = `OR`, vì OR cho phép KH match nhiều trigger độc lập trong cùng campaign.
- Khi logic = `AND`, hệ thống ẩn toàn bộ khối conflict/priority trong campaign vì campaign chỉ gửi khi KH thỏa đồng thời tất cả trigger và dùng 1 message chung theo từng kênh.
- QTV có thể điều chỉnh priority của từng trigger trong campaign bằng kéo-thả hoặc dropdown P1/P2/P3. Priority này chỉ áp dụng trong campaign hiện tại, không thay đổi priority master của trigger.
- `Cấu hình gửi theo Trigger` được tách thành Section 6 riêng để QTV nhìn rõ thời điểm gửi, delay, điều kiện và blackout theo từng trigger.

**State khi chọn Basic — 1 trigger chính:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Chế độ trigger                                                     │
│ ● Basic — 1 trigger chính                                          │
│ ○ Advanced — nhiều trigger + AND/OR rõ ràng                        │
│                                                                    │
│ Trigger chính *                                                    │
│ [BSS · SIM_ACTIVATED · Kích hoạt SIM thành công ▾]                 │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ★ SIM_ACTIVATED                                                │ │
│ │ BSS · Kích hoạt SIM thành công                                 │ │
│ │ Kiểu chạy: Realtime · gửi ngay khi event đến                    │ │
│ │ Payload: [ten_kh] [loai_sim] [ngay_kich_hoat] [so_dien_thoai]  │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ⓘ Basic mode chỉ dùng 1 trigger nên không cần AND/OR.              │
│   Message Matrix sẽ hiển thị 1 message theo từng kênh.             │
└────────────────────────────────────────────────────────────────────┘
```

**State khi Advanced logic = AND:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Logic xử lý khi có nhiều trigger                                   │
│ ○ OR — KH match trigger nào thì gửi message của trigger đó         │
│ ● AND — KH phải thỏa tất cả trigger, gửi 1 message chung           │
│                                                                    │
│ Diễn giải AND                                                      │
│ Chỉ gửi khi KH đồng thời thỏa:                                     │
│ ✓ Đã kích hoạt SIM                                                 │
│ ✓ Sau 24h vẫn chưa cài app                                         │
│ ✓ Data còn lại < 100MB                                             │
│                                                                    │
│ Khi đủ cả 3 điều kiện → dùng 1 message chung theo từng kênh.       │
│                                                                    │
│ Không hiển thị xử lý xung đột trigger trong AND mode.              │
└────────────────────────────────────────────────────────────────────┘
```

---

### Section 3: Audience / Phân khúc

```
┌────────────────────────────────────────────────────────────────────┐
│ 3. Audience / Phân khúc                                            │
├────────────────────────────────────────────────────────────────────┤
│ Nguồn phân khúc: Customer 360 / Team Data / BSS / OCS              │
│ Cập nhật lần cuối: 12/05/2026 08:00                                │
│                                                                    │
│ Phân khúc khách hàng                                               │
│ [Tìm và chọn phân khúc... ▾]                                       │
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
- Dropdown phân khúc chỉ mở danh sách khi user click/search, không hiển thị toàn bộ danh sách cố định bên dưới.
- Khi user tích chọn một phân khúc trong dropdown, hệ thống thêm badge vào vùng `Đã chọn`.
- Khi user bỏ tích hoặc click `×` trên badge, phân khúc bị bỏ khỏi tập target và reach được tính lại.
- Điều kiện lọc bổ sung chỉ là filter trên tập segment đã chọn, không phải rule builder để tạo segment mới.

**Dropdown phân khúc khi mở:**

```
┌────────────────────────────────────────────────────────────────────┐
│ [gen z______________________________________________]              │
├────────────────────────────────────────────────────────────────────┤
│ ☑ Gen Z User (18–25)                         18,450 KH             │
│ ☐ Thuê bao mới kích hoạt                      32,100 KH             │
│ ☑ Sắp hết data                                 8,920 KH             │
│ ☐ Chưa cài app MyViettel                      14,300 KH             │
│ ☐ ARPU cao                                     5,600 KH             │
│ ☐ Nguy cơ rời mạng                             2,150 KH             │
└────────────────────────────────────────────────────────────────────┘
```

**State khi click `+ Thêm điều kiện`:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Điều kiện lọc bổ sung                                              │
│ [Loại thiết bị ▾] [= ▾] [Android ▾] [×]                            │
│ [Data còn lại ▾] [<= ▾] [100 MB____] [×]                           │
│ [Số ngày kích hoạt ▾] [<= ▾] [30 ngày___] [×]                      │
│ [+ Thêm điều kiện]                                                  │
└────────────────────────────────────────────────────────────────────┘
```

Available condition attributes:
- Thiết bị: Loại thiết bị, Hệ điều hành, Handset model
- Thuê bao: Số ngày kích hoạt, Loại SIM, Trạng thái thuê bao
- Dữ liệu sử dụng: Data còn lại, Tên gói, Ngày hết hạn gói
- Ứng dụng: Đã cài app, Lần active gần nhất

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
│ Push: 3/3 ✓  Zalo: 3/3 ✓  SMS: 2/3 ⚠  Banner: 1/3  Email: 0/3     │
│ USSD: 0/3                                                           │
│                                                                    │
│ Tab kênh: [Push] [Zalo OA] [SMS] [Banner] [Email] [USSD] [+ Kênh] │
│                                                                    │
│ ── Tab Push đang active ─────────────────────────────────────────  │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T1 · SIM_ACTIVATED                                           │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Welcome Push Template ▾]                             │ │
│ │ Image: [🖼 Upload] [Chọn từ thư viện]                            │ │
│ │ Title: Chào mừng bạn!                                          │ │
│ │ Body: SIM eSIM đã kích hoạt thành công. Tải MyViettel...       │ │
│ │ Preview: [App icon] Viettel now                                 │ │
│ │          Chào mừng bạn!                                         │ │
│ │          SIM eSIM đã kích hoạt thành công...                    │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T2 · NO_APP_INSTALL_24H                                      │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Install App Reminder ▾]                              │ │
│ │ Image: [🖼 Upload] [Chọn từ thư viện]                            │ │
│ │ Title: Cài MyViettel để nhận ưu đãi                             │ │
│ │ Body: Bạn vẫn chưa cài app MyViettel...                         │ │
│ │ Preview: [Push notification preview]                            │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ✓ T3 · LOW_DATA_BALANCE                                        │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Low Data Gen Z Push ▾]                               │ │
│ │ Image: [🖼 Upload] [Chọn từ thư viện]                            │ │
│ │ Title: Sắp hết data rồi 😮                                      │ │
│ │ Body: Bạn chỉ còn {{data_con_lai}} MB. Mua thêm gói...         │ │
│ │ [+ Thêm biến thể đối tượng]                                     │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ── Khi chuyển sang tab Zalo OA ──────────────────────────────────  │
│ [T1] Template: [Welcome Zalo OA ▾]      ✓                          │
│      Nội dung: Xin chào {{ten_kh}}, eSIM đã kích hoạt...           │
│      Image: [Upload] [Chọn từ thư viện]                            │
│      Preview: [Zalo brand header] + bubble + image                 │
│ [T2] Template: [Install App Zalo ▾]      ✓                         │
│ [T3] Template: [Low Data Zalo ▾]         ✓                         │
│                                                                    │
│ ── Khi chuyển sang tab SMS ──────────────────────────────────────  │
│ [T1] SMS Welcome                  ✓  92/160 ký tự                  │
│ [T2] SMS Chưa cài app             ✓  118/160 ký tự                 │
│ [T3] SMS Sắp hết data             ⚠ 300/160 ký tự = 2 SMS          │
│     Preview: plain text, không ảnh; chi phí tính theo 2 tin SMS    │
│                                                                    │
│ ── Khi chuyển sang tab Banner ─────────────────────────────────── │
│ [T1] Banner Welcome               ✓                                │
│      Image 16:9: [Upload] [Chọn từ thư viện]                       │
│      Title: Kích hoạt eSIM thành công                              │
│      Body: Khám phá ưu đãi dành riêng cho thuê bao mới             │
│      CTA: [Mở MyViettel]                                           │
│      Preview: banner 16:9 + title + body + CTA                     │
│ [T2] Banner Chưa cài app          ⚠ Chưa cấu hình                  │
│ [T3] Banner Sắp hết data          ⚠ Chưa cấu hình                  │
│                                                                    │
│ ── Khi chuyển sang tab Email ──────────────────────────────────── │
│ [T1] Email Welcome                ⚠ Chưa cấu hình                  │
│      Subject: [Chào mừng bạn đến với eSIM____________________]     │
│      Header image: [Upload] [Chọn từ thư viện]                     │
│      Body rich text: [________________________________________]     │
│      Preview: email client style, subject + header image + body    │
│                                                                    │
│ ── Khi chuyển sang tab USSD ───────────────────────────────────── │
│ [T1] USSD Welcome                 ⚠ Chưa cấu hình                  │
│      Nội dung: [______________________________________________]     │
│      Counter: 0/182 ký tự                                           │
│      ⚠ USSD không hỗ trợ ảnh; cần kiểm tra tiếng Việt có dấu       │
│      Preview: màn hình USSD dạng terminal/menu                     │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- QTV thao tác theo kênh trước, sau đó cấu hình message theo trigger trong kênh đó.
- Với OR: thuê bao match trigger nào thì gửi message tương ứng của trigger đó trên kênh đang xử lý.
- Với AND: mỗi kênh chỉ còn 1 message chung, nhưng vẫn là message riêng theo từng kênh.
- Cùng một trigger có thể có nhiều audience variant nếu các phân khúc cần thông điệp khác nhau.
- Template Library được hỗ trợ nghiêm túc hơn nhưng không khóa cứng nhập tự do ở phase này; campaign approval sẽ duyệt toàn bộ nội dung campaign.
- Với các kênh hỗ trợ media như Push, Zalo OA, Banner, Email, mỗi message card có khu vực upload/chọn ảnh riêng.
- Preview nằm trong từng message card và render đúng format của kênh đang chọn.
- SMS/USSD không hiển thị upload ảnh; chỉ hiển thị counter ký tự và cảnh báo format.
- SMS cho phép nhập vượt 160 ký tự; hệ thống tự tính số SMS segment theo công thức `ceil(số ký tự / 160)`.
- Ví dụ `300/160 ký tự = 2 SMS`; cost estimate và SMS risk ở cột phải phải nhân theo số segment, không chỉ theo số thuê bao.
- USSD dùng nội dung plain text/menu, không hỗ trợ ảnh; nếu nội dung có tiếng Việt có dấu hoặc ký tự đặc biệt không được gateway hỗ trợ thì hiển thị warning để QTV chỉnh.
- Banner bắt buộc có ảnh 16:9, title, body và CTA nếu kênh Banner được dùng trong campaign.
- Email bắt buộc có subject và body; header image là optional nhưng preview phải render đúng bố cục email.
- Zalo OA hiển thị brand header, bubble content và ảnh nếu có; nội dung có thể chọn template hoặc nhập trực tiếp trong campaign.

**State khi Advanced logic = AND trong Message Matrix:**

```
┌────────────────────────────────────────────────────────────────────┐
│ 4. Message Matrix                                                  │
├────────────────────────────────────────────────────────────────────┤
│ Advanced AND: tất cả trigger cùng thỏa → 1 message chung / kênh    │
│                                                                    │
│ Tab kênh: [Push] [Zalo OA] [SMS] [Banner] [Email] [USSD]           │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ Message chung · Push                                           │ │
│ │ Điều kiện: T1 + T2 + T3 cùng thỏa                              │ │
│ │ Audience variant: [Gen Z User]                                 │ │
│ │ Template: [Welcome + App + Low Data Push ▾]                    │ │
│ │ Image: [🖼 Upload] [Chọn từ thư viện]                           │ │
│ │ Title: Hoàn tất trải nghiệm eSIM của bạn                       │ │
│ │ Body: SIM đã kích hoạt, hãy cài app và mua thêm data...        │ │
│ │ Preview: [Push notification preview]                            │ │
│ └────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

**State khi click `+ Thêm biến thể đối tượng`:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Thêm biến thể đối tượng cho T3 · LOW_DATA_BALANCE                  │
├────────────────────────────────────────────────────────────────────┤
│ Chọn phân khúc áp dụng                                             │
│ [Khách hàng lớn tuổi ▾]                                            │
│                                                                    │
│ Kênh: Push                                                         │
│ Template: [Low Data Basic Push ▾]                                  │
│ Title: Bạn sắp hết dung lượng data                                 │
│ Body: Dung lượng còn lại {{data_con_lai}} MB. Vui lòng mua thêm... │
│                                                                    │
│ [Hủy]                                      [Thêm biến thể]         │
└────────────────────────────────────────────────────────────────────┘
```

Sau khi thêm, card T3 hiển thị nhiều audience variant:
- [Gen Z User] → Low Data Gen Z Push
- [Khách hàng lớn tuổi] → Low Data Basic Push

**Preview theo từng kênh:**
- Push: app icon, title, body, thumbnail 1:1 nếu có.
- Zalo OA: brand header, bubble content, image nếu có.
- SMS: plain text, không ảnh, counter X/160 và note nếu vượt ký tự.
- Banner: image 16:9, title, body, CTA.
- Email: subject, header image, rich text body.
- USSD: màn hình terminal/menu, plain text, không ảnh, counter ký tự và warning nếu nội dung có dấu/ký tự không hỗ trợ.

**Trường cấu hình theo từng kênh:**

| Kênh | Field bắt buộc | Field optional | Preview |
|---|---|---|---|
| Push | Template hoặc Title + Body | Image 1:1, deep link | OS notification với app icon, title, body, thumbnail |
| Zalo OA | Template hoặc Nội dung tin | Image, CTA/link nếu template hỗ trợ | Brand header + chat bubble + image |
| SMS | Nội dung SMS | Không có | Plain text + counter X/160 + số segment |
| Banner | Image 16:9, Title, Body, CTA | Deep link | Banner card 16:9 + CTA |
| Email | Subject, Body | Header image, CTA | Email client preview |
| USSD | Nội dung USSD/menu | Không có | Terminal/menu preview |

**State khi thêm kênh mới:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Thêm kênh gửi                                                      │
├────────────────────────────────────────────────────────────────────┤
│ Chọn kênh                                                          │
│ ☐ Push App                                                         │
│ ☐ Zalo OA                                                          │
│ ☐ SMS                                                              │
│ ☑ Banner                                                           │
│ ☑ Email                                                            │
│ ☑ USSD                                                             │
│                                                                    │
│ Kênh đã có trong Message Matrix sẽ bị disable trong danh sách.     │
│                                            [Hủy] [Thêm kênh]       │
└────────────────────────────────────────────────────────────────────┘
```

---

### Section 5: An toàn & Chống Spam

```
┌────────────────────────────────────────────────────────────────────┐
│ 5. An toàn & Chống Spam                                            │
├────────────────────────────────────────────────────────────────────┤
│ Giờ giới nghiêm (Blackout)                                         │
│ [22:00 🕘]  -  [08:00 🕘]                                          │
│ [Hủy luôn (Discard) ▾]                                             │
│                                                                    │
│ Giới hạn nhận tin                                                  │
│ Gửi [1] lần / [30] ngày                                            │
│                                                                    │
│ ☑ Check blacklist DNC                                              │
│ ✅ Enabled                                                         │
│                                                                    │
│ Whitelist áp dụng                                                  │
│ ○ Không dùng whitelist                                             │
│ ● Chỉ gửi cho tệp whitelist được chọn                              │
│                                                                    │
│ Tệp whitelist                                                      │
│ [WL_ESIM_Q2_2026 · 1,250 KH · cập nhật 12/05/2026 08:00 ▾]         │
│ [Xem danh sách] [Thay tệp] [Upload tệp mới]                        │
│                                                                    │
│ Sau khi áp whitelist: reach còn ~1,120 KH                          │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Blackout là rule an toàn ở campaign level, áp dụng cho toàn bộ trigger trong campaign.
- Dropdown xử lý blackout gồm: `Queue đến giờ cho phép`, `Hủy luôn (Discard)`, `Gửi ngay nếu kênh cho phép`.
- Frequency cap giới hạn số lần một KH được nhận campaign trong một chu kỳ; nếu vượt cap thì không gửi.
- `Check blacklist DNC` mặc định bật; nếu bỏ chọn phải hiện confirm vì có rủi ro gửi tới KH đã từ chối nhận tin.
- Whitelist là optional. Khi chọn `Chỉ gửi cho tệp whitelist được chọn`, hệ thống chỉ gửi cho phần giao giữa Audience/Phân khúc và whitelist.
- Nếu KH thuộc audience nhưng không nằm trong whitelist, KH bị loại khỏi reach.
- Nếu tệp whitelist rỗng hoặc parse lỗi, hiển thị lỗi và không cho gửi campaign.
- `Xem danh sách` mở modal preview 50 dòng đầu, số KH hợp lệ, số trùng, số sai định dạng.
- `Upload tệp mới` mở modal upload CSV với format 1 cột `so_dien_thoai`; sau upload hệ thống validate và cập nhật reach.

**Modal chọn / xem whitelist:**

```
┌──────────────────────────────────────────────────────────────┐
│ Chọn tệp whitelist                                           │
├──────────────────────────────────────────────────────────────┤
│ [Tìm tên tệp whitelist...]                                   │
│                                                              │
│ ○ WL_TEST_INTERNAL        35 KH     10/05/2026 14:00         │
│ ● WL_ESIM_Q2_2026       1,250 KH    12/05/2026 08:00         │
│ ○ WL_HIGH_ARPU_DATA       860 KH    11/05/2026 09:30         │
│                                                              │
│ Preview tệp đang chọn                                        │
│ Số hợp lệ: 1,250 · Trùng: 12 · Sai định dạng: 3              │
│ 0987 xxx 001                                                 │
│ 0912 xxx 002                                                 │
│ 0965 xxx 003                                                 │
│                                                              │
│                              [Hủy] [Áp dụng whitelist]       │
└──────────────────────────────────────────────────────────────┘
```

---

### Section 6: Cấu hình gửi theo Trigger

```
┌────────────────────────────────────────────────────────────────────┐
│ 6. Cấu hình gửi theo Trigger                                      │
├────────────────────────────────────────────────────────────────────┤
│ ⓘ Thời điểm gửi phụ thuộc vào trigger; không có lịch gửi chung    │
│   cho toàn campaign.                                               │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ T1 · SIM_ACTIVATED                                             │ │
│ │ Kiểu chạy: Realtime                                            │ │
│ │ Xử lý: ngay khi nhận event hợp lệ                              │ │
│ │ Blackout: queue đến 08:00 nếu rơi 20:00–08:00                  │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ T2 · NO_APP_INSTALL_24H                                        │ │
│ │ Kiểu chạy: Near realtime                                       │ │
│ │ Delay: 24h sau SIM_ACTIVATED                                   │ │
│ │ Điều kiện: app_installed = false                               │ │
│ │ Blackout: queue đến 08:00 nếu rơi 20:00–08:00                  │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ T3 · LOW_DATA_BALANCE                                          │ │
│ │ Kiểu chạy: Offline / Batch                                     │ │
│ │ Lịch quét: 09:00 hằng ngày                                     │ │
│ │ Điều kiện: data_con_lai <= 100MB                               │ │
│ │ Blackout: không chạy batch trong 20:00–08:00                   │ │
│ └────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Section này chỉ mô tả và cấu hình thời điểm xử lý theo từng trigger đã chọn ở Section 2.
- Với trigger Realtime, QTV có thể chọn xử lý ngay hoặc queue nếu rơi vào blackout.
- Với trigger Near realtime, QTV thấy rõ delay, trigger phụ thuộc và điều kiện kiểm tra lại trước khi gửi.
- Với trigger Offline / Batch, QTV thấy rõ giờ quét batch và điều kiện lọc.
- Không có campaign-level schedule chung; campaign chỉ có thời gian hiệu lực ở Section 1.

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
│ Safety                        │
│ Blackout 22:00–08:00          │
│ Frequency cap: 1 lần / 30 ngày│
│ DNC: Enabled                  │
│ Whitelist: WL_ESIM_Q2_2026    │
│                              │
│ Risk / Cost                   │
│ Estimated SMS: 2,400 segments │
│ Estimated cost: ~720,000 VND  │
│                              │
│ Approval                      │
│ Draft                         │
│ Admin duyệt để Active         │
└──────────────────────────────┘
```

**Hành vi cột phải:**
- Sticky khi scroll để QTV luôn thấy campaign thiếu gì.
- Click vào item lỗi, ví dụ `SMS: 2/3`, scroll tới đúng message còn thiếu.
- Cost/SMS risk cập nhật khi thay đổi segment, whitelist, trigger schedule, frequency cap hoặc nội dung SMS.

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
6. Cấu hình An toàn & Chống Spam: blackout, frequency cap, DNC, whitelist nếu cần
   ↓
7. Xem Cấu hình gửi theo Trigger
   ↓
8. Lưu nháp hoặc gửi Admin duyệt từ fixed header
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
| Gửi duyệt khi còn trống | Validate từ fixed header: highlight các field bắt buộc còn trống |
| Tắt trigger có Active campaign | Hiện cảnh báo trong modal Campaign Linked trước khi tắt |
| Trigger priority trùng nhau trong OR mode | Hệ thống cảnh báo inline và yêu cầu QTV chọn lại P1/P2/P3 hoặc kéo-thả để phân thứ tự rõ ràng |
| Advanced logic = AND | Ẩn xử lý xung đột trigger; Message Matrix chuyển sang 1 message chung / kênh |
| Chọn 0 segment, 0 điều kiện | Mặc định gửi T-ALL (tất cả KH), hiện thông báo xác nhận |
| Segment OR + điều kiện AND | Ước tính cập nhật realtime khi thay đổi bất kỳ điều kiện nào |
| Bật whitelist nhưng chưa chọn tệp | Blocking validation: yêu cầu chọn tệp whitelist hoặc chuyển về không dùng whitelist |
| KH thuộc audience nhưng không thuộc whitelist | Loại khỏi reach và không gửi |
| Whitelist có số sai định dạng | Bỏ qua số sai định dạng, hiển thị số lượng bị bỏ qua trong preview |
