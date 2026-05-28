# Wireframe CVM System — v7
> Cập nhật từ v5: bổ sung USSD trong Message Matrix, bỏ Review & Submit, thêm An toàn & Chống Spam/whitelist, làm rõ Channel Strategy theo kênh đã thêm, SMS cost theo segment 160 ký tự, và priority trigger trong campaign

---

## Thay đổi so với v5

| v5 | v7 |
|---|---|
| Message Matrix chưa có USSD | Bổ sung tab USSD, field cấu hình và preview tương ứng |
| Section 7 Review & Submit | Bỏ khỏi form; dùng fixed header để Lưu Nháp/Gửi duyệt |
| Thiếu cấu hình whitelist trong Campaign Builder | Thêm khối An toàn & Chống Spam với blacklist DNC và whitelist |
| Channel Strategy là danh sách cố định | Đồng bộ từ kênh đã thêm ở Message Matrix và cho kéo-thả tự do |
| SMS mới cảnh báo ký tự | Cho nhập vượt 160 ký tự và tính chi phí theo segment |
| Chưa có priority trigger trong campaign | Thêm khối xử lý xung đột khi nhiều trigger cùng match |

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
│ 5. Channel Strategy & An toàn                 │                               │
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
│ Xử lý khi KH match nhiều trigger cùng lúc                          │
│ ● Chỉ gửi trigger ưu tiên cao nhất                                 │
│ ○ Gửi tất cả trigger đã match (không khuyến nghị)                  │
│                                                                    │
│ Ưu tiên trigger trong campaign                                      │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ [≡] P1  T3 · LOW_DATA_BALANCE        Data còn dưới 100MB       │ │
│ │ [≡] P2  T2 · NO_APP_INSTALL_24H      Chưa cài app sau 24h      │ │
│ │ [≡] P3  T1 · SIM_ACTIVATED           Kích hoạt SIM thành công  │ │
│ └────────────────────────────────────────────────────────────────┘ │
│ ⓘ Priority này chỉ áp dụng trong campaign, không đổi master trigger│
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Basic mode chỉ cho chọn 1 trigger, ẩn AND/OR để giảm độ rối.
- Advanced mode cho nhiều trigger và bắt buộc chọn AND/OR.
- Mỗi trigger hiển thị rõ source, kiểu chạy, payload và trạng thái mapping message.
- Lịch xử lý ban đầu phụ thuộc trigger, không phụ thuộc campaign-level schedule.
- Khối xử lý xung đột chỉ hiển thị khi Advanced logic = OR. Nếu logic = AND, hệ thống ẩn khối này vì campaign chỉ gửi khi thuê bao thỏa đồng thời tất cả trigger và dùng message chung theo từng kênh.
- Với OR, khi một thuê bao cùng lúc thỏa nhiều trigger trong cùng campaign, mặc định chỉ gửi message của trigger có priority cao nhất để tránh spam.
- QTV có thể kéo-thả thứ tự priority hoặc dùng dropdown P1/P2/P3 trên từng dòng. Nếu hai trigger trùng priority, hiển thị cảnh báo inline và yêu cầu sắp xếp lại trước khi gửi duyệt.
- Priority trong campaign không làm thay đổi priority master của trigger ở Trigger Management.

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
│ Push: 3/3 ✓  Zalo: 3/3 ✓  SMS: 2/3 ⚠  Banner: 1/3 ⚠               │
│ Email: 0/3   USSD: 0/3                                             │
│                                                                    │
│ Tab kênh: [Push] [Zalo OA] [SMS] [Banner] [Email] [USSD] [+ Kênh] │
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
│ [T3] SMS Sắp hết data             ⚠ 300/160 ký tự = 2 SMS segment  │
│     Reach SMS dự kiến: ~1,200 KH                                   │
│     Chi phí: 1,200 × 2 segment = 2,400 SMS                         │
│     Nội dung vẫn được phép lưu; cột phải cập nhật cost theo 2 tin  │
│                                                                    │
│ ── Khi chuyển sang tab USSD ───────────────────────────────────── │
│ [T1] USSD Welcome                 ⚠ Chưa cấu hình                  │
│     Nội dung: [______________________________________________]     │
│     Counter: 0/182 ký tự                                            │
│     Preview: màn hình USSD dạng terminal/menu                      │
│     ⚠ USSD không hỗ trợ ảnh; cần kiểm tra tiếng Việt có dấu        │
│ [T2] USSD Chưa cài app            ⚠ Chưa cấu hình                  │
│ [T3] USSD Sắp hết data            ⚠ Chưa cấu hình                  │
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
- Ví dụ `300/160 ký tự = 2 SMS segment`; cost estimate và SMS risk ở cột phải phải nhân theo số segment, không chỉ theo số thuê bao.
- USSD là kênh plain text/menu, không có upload ảnh; preview dạng terminal/menu và cảnh báo nếu nội dung có dấu hoặc ký tự không được gateway hỗ trợ.
- Tab `+ Kênh` chỉ hiển thị các kênh chưa được thêm. Nếu kênh đã có tab, kênh đó disabled trong danh sách thêm mới.
- Mỗi tab kênh dùng layout 2 cột: bên trái là cấu hình nội dung, bên phải là `XEM TRƯỚC` và `SAMPLE PARAMS`.
- Parameter chips hiển thị theo payload của trigger đang cấu hình. Với AND mode, message chung được dùng cho nhiều trigger nên danh sách parameter là hợp nhất payload của tất cả trigger trong điều kiện AND.
- Click parameter chip sẽ chèn biến vào field đang focus; sample params bên phải cho phép nhập giá trị mẫu để preview render realtime.

**Parameter theo trigger:**

| Trigger | Parameter khả dụng |
|---|---|
| T1 · SIM_ACTIVATED | `{{ten_kh}}`, `{{loai_sim}}`, `{{ngay_kich_hoat}}`, `{{so_dien_thoai}}` |
| T2 · NO_APP_INSTALL_24H | `{{app_installed}}`, `{{last_active}}` |
| T3 · LOW_DATA_BALANCE | `{{data_con_lai}}`, `{{ten_goi}}`, `{{ngay_het_han}}` |
| AND message chung | Toàn bộ parameter của T1 + T2 + T3, hiển thị theo nhóm trigger |

**Layout cấu hình message theo từng kênh:**

```
┌──────────────────────────────────────────────┬───────────────────────────────┐
│ CẤU HÌNH NỘI DUNG                             │ XEM TRƯỚC                     │
├──────────────────────────────────────────────┼───────────────────────────────┤
│ ○ Chọn template có sẵn  ● Soạn mới            │ Preview đúng format kênh      │
│ Media upload nếu kênh hỗ trợ                  │                               │
│ Message card / editor                         │ SAMPLE PARAMS                 │
│ Parameter chips                               │ ten_kh          [Nguyễn Văn A]│
│ Textarea/input theo kênh                      │ loai_sim        [eSIM]        │
│ Counter / warning theo kênh                   │ ngay_kich_hoat  [01/04/2026] │
└──────────────────────────────────────────────┴───────────────────────────────┘
```

**State khi Advanced logic = AND trong Message Matrix:**

```
┌────────────────────────────────────────────────────────────────────┐
│ AND mode — nhiều trigger = 1 tin nhắn chung                        │
├────────────────────────────────────────────────────────────────────┤
│ Tab kênh: [Zalo OA (Kênh đầu)] [SMS ×] [USSD ×] [Push ×]           │
│           [Banner ×] [Email ×] [+ Thêm kênh ▾]                    │
│                                                                    │
│ ── Tab Zalo OA đang active ────────────────────────────────────── │
│ Fallback sau: [5] phút                                             │
│ ○ Chọn template có sẵn  ● Soạn mới                                 │
│ Upload ảnh: [Tải ảnh lên] [Chọn từ thư viện]  Tỉ lệ: Tự do         │
│                                                                    │
│ Message chung cho 3 trigger                                        │
│ Parameter chips theo nhóm:                                         │
│ T1: {{ten_kh}} {{loai_sim}} {{ngay_kich_hoat}} {{so_dien_thoai}}   │
│ T2: {{app_installed}} {{last_active}}                              │
│ T3: {{data_con_lai}} {{ten_goi}} {{ngay_het_han}}                  │
│ Nội dung: [Xin chào {{ten_kh}}, SIM {{loai_sim}} đã kích hoạt...]  │
│                                                                    │
│ Preview Zalo OA: header Viettel Telecom + image + bubble content   │
│ Sample params: ten_kh = Nguyễn Văn A, loai_sim = eSIM, ...         │
└────────────────────────────────────────────────────────────────────┘
```

**Preview trong AND mode theo từng kênh:**

| Kênh | Editor bên trái | Preview bên phải |
|---|---|---|
| Zalo OA | Upload ảnh tỉ lệ tự do, message chung rich/plain text, parameter chips | Brand header `Viettel Telecom`, vùng ảnh, bubble nội dung đã render params |
| SMS | Không hỗ trợ ảnh, textarea plain text, counter `X/160`, segment/cost | SMS card `SMS · Viettel`, nội dung plain text, counter ký tự |
| USSD | Không hỗ trợ ảnh, textarea menu/plain text, warning tiếng Việt có dấu | Terminal đen chữ xanh, hiển thị menu USSD đã render params |
| Push | Upload ảnh 1:1, input title, textarea body, parameter chips | OS notification style: app icon, title, body, thumbnail |
| Banner | Upload ảnh 16:9, input title/body/CTA | Banner card 16:9, title, body, CTA button |
| Email | Header image, subject, rich text body | Email client style: subject, header image, body đã render params |

**Trường cấu hình theo từng kênh:**

| Kênh | Field bắt buộc | Field optional | Preview |
|---|---|---|---|
| Push | Template hoặc Title + Body | Image 1:1, deep link | OS notification với app icon, title, body, thumbnail |
| Zalo OA | Template hoặc nội dung tin | Image, CTA/link nếu template hỗ trợ | Brand header + chat bubble + image |
| SMS | Nội dung SMS | Không có | Plain text + counter + segment/cost |
| Banner | Image 16:9, Title, Body, CTA | Deep link | Banner card 16:9 + CTA |
| Email | Subject, Body | Header image, CTA | Email client preview |
| USSD | Nội dung USSD/menu | Không có | Terminal/menu preview |

---

### Section 5: Channel Strategy & An toàn

```
┌────────────────────────────────────────────────────────────────────┐
│ 5. Channel Strategy & An toàn                                      │
├────────────────────────────────────────────────────────────────────┤
│ 5A. Channel Strategy                                               │
│ Mục tiêu: ưu tiên kênh rẻ trước, SMS là kênh cuối                  │
│ Kênh lấy từ Message Matrix: Push, Zalo OA, SMS, Banner, Email, USSD│
│                                                                    │
│ Thứ tự kênh gửi                                                     │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ [≡] 1. Push App                                                │ │
│ │    Gửi theo lịch của trigger                                   │ │
│ │    Chờ phản hồi: [30] phút                                     │ │
│ │    Nếu KH [Clicked ▾] → dừng fallback                          │ │
│ └────────────────────────────────────────────────────────────────┘ │
│          ↓ nếu chưa clicked / fail                                 │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ [≡] 2. Zalo OA / ZNS                                           │ │
│ │    Gửi sau Push [30] phút nếu chưa clicked                     │ │
│ │    Nếu [Delivered hoặc Clicked ▾] → dừng fallback              │ │
│ └────────────────────────────────────────────────────────────────┘ │
│          ↓ nếu fail / không phản hồi                               │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ [≡] 3. SMS                                                     │ │
│ │    Kênh cuối, chỉ gửi nếu các kênh trước không đạt điều kiện   │ │
│ │    Ước tính SMS: ~1,200 KH × 2 segment = 2,400 SMS             │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ Kênh chưa tham gia fallback: [Banner] [Email] [USSD]               │
│ [+ Thêm vào strategy]                                              │
│                                                                    │
│ ☑ Không gửi SMS nếu KH đã click/open ở kênh trước                  │
│ Điều kiện dừng fallback: ☐ Delivered  ☐ Opened  ☑ Clicked          │
│                                                                    │
│ 5B. An toàn & Chống Spam                                           │
│ Giờ giới nghiêm (Blackout)                                         │
│ [22:00 🕘]  -  [08:00 🕘]                                          │
│ [Hủy luôn (Discard) ▾]                                             │
│                                                                    │
│ Giới hạn nhận tin                                                  │
│ Gửi [1] lần / [30] ngày                                            │
│                                                                    │
│ Giới hạn Push Notification                                         │
│ Tối đa [3] push / ngày                                             │
│ Tối đa [10] push / tuần                                            │
│ Tối đa [30] push / tháng                                           │
│ ☑ Áp dụng theo từng thuê bao                                       │
│                                                                    │
│ Blacklist / Suppression                                            │
│ ☑ Check blacklist DNC toàn hệ thống                                │
│ ✅ Enabled                                                         │
│                                                                    │
│ Blacklist theo chiến dịch                                          │
│ ● Không dùng blacklist riêng                                       │
│ ○ Dùng blacklist riêng cho campaign này                            │
│                                                                    │
│ (Các trường chọn tệp blacklist riêng đang ẩn)                      │
│                                                                    │
│ ── Khi chọn "Dùng blacklist riêng cho campaign này" ───────────── │
│ Tệp blacklist campaign                                             │
│ [BL_ESIM_Q2_2026 · 320 KH · cập nhật 12/05/2026 09:00 ▾]          │
│ [Xem danh sách] [Thay tệp] [Upload tệp mới]                        │
│                                                                    │
│ Sau khi áp blacklist campaign: loại 320 KH                         │
│                                                                    │
│ Whitelist áp dụng                                                  │
│ ● Không dùng whitelist                                             │
│ ○ Chỉ gửi cho tệp whitelist được chọn                              │
│                                                                    │
│ (Các trường chọn tệp whitelist đang ẩn)                            │
│                                                                    │
│ ── Khi chọn "Chỉ gửi cho tệp whitelist được chọn" ─────────────── │
│ Tệp whitelist                                                      │
│ [WL_ESIM_Q2_2026 · 1,250 KH · cập nhật 12/05/2026 08:00 ▾]         │
│ [Xem danh sách] [Thay tệp] [Upload tệp mới]                        │
│                                                                    │
│ Sau khi áp whitelist: reach còn ~1,120 KH                          │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Trigger xác định thời điểm bắt đầu xử lý.
- Channel Strategy xác định thứ tự kênh, thời gian chờ và điều kiện dừng fallback.
- Danh sách kênh trong Channel Strategy phụ thuộc trực tiếp vào các tab kênh đã thêm ở Message Matrix.
- QTV có thể kéo-thả tự do để đổi thứ tự gửi giữa các kênh; SMS mặc định được đặt cuối nhưng vẫn có thể điều chỉnh nếu nghiệp vụ yêu cầu.
- Nếu QTV xóa một kênh ở Message Matrix, hệ thống confirm và gỡ kênh đó khỏi Channel Strategy.
- Nếu QTV thêm kênh mới ở Message Matrix, kênh đó xuất hiện trong vùng `Kênh chưa tham gia fallback` để QTV kéo vào strategy khi cần.
- Safety rule xác định có được gửi tại thời điểm dự kiến hay phải queue/chặn/hủy.
- Blackout là rule an toàn ở campaign level, áp dụng cho toàn bộ trigger trong campaign.
- Dropdown xử lý blackout gồm: `Queue đến giờ cho phép`, `Hủy luôn (Discard)`, `Gửi ngay nếu kênh cho phép`.
- Frequency cap giới hạn số lần một KH được nhận campaign trong một chu kỳ; nếu vượt cap thì không gửi.
- Giới hạn Push Notification kiểm soát số lượng push tối đa mà một thuê bao được nhận theo ngày/tuần/tháng.
- Nếu thuê bao vượt quota Push, hệ thống không gửi Push cho thuê bao đó và xử lý tiếp theo Channel Strategy: fallback sang kênh kế tiếp nếu còn đủ điều kiện.
- Nếu campaign không có kênh Push trong Message Matrix, khối `Giới hạn Push Notification` hiển thị disabled kèm note `Chỉ áp dụng khi campaign có kênh Push`.
- `Check blacklist DNC toàn hệ thống` là global/system-level suppression, mặc định bật; nếu bỏ chọn phải hiện confirm vì có rủi ro gửi tới KH đã từ chối nhận tin.
- `Blacklist theo chiến dịch` là optional, chỉ áp dụng cho campaign hiện tại để loại các thuê bao không muốn gửi trong campaign này.
- Mặc định chọn `Không dùng blacklist riêng`; khi ở state này, ẩn toàn bộ dropdown tệp blacklist, nút `Xem danh sách`, `Thay tệp`, `Upload tệp mới`, preview và dòng `Sau khi áp blacklist campaign`.
- Chỉ khi QTV chọn `Dùng blacklist riêng cho campaign này`, hệ thống mới hiển thị vùng chọn/upload tệp blacklist campaign.
- Nếu bật blacklist riêng nhưng chưa chọn tệp, hệ thống hiển thị blocking validation.
- Tệp blacklist campaign dùng format CSV 1 cột `so_dien_thoai`, có preview số hợp lệ/trùng/sai định dạng giống whitelist.
- Công thức reach cuối cùng: `Audience ban đầu → trừ Global DNC/blacklist hệ thống → trừ Campaign blacklist → giao với Whitelist nếu bật whitelist`.
- Whitelist là optional. Khi chọn `Chỉ gửi cho tệp whitelist được chọn`, hệ thống chỉ gửi cho phần giao giữa Audience/Phân khúc và whitelist.
- Mặc định chọn `Không dùng whitelist`; khi ở state này, ẩn toàn bộ dropdown tệp whitelist, nút `Xem danh sách`, `Thay tệp`, `Upload tệp mới`, preview và dòng `Sau khi áp whitelist`.
- Chỉ khi QTV chọn `Chỉ gửi cho tệp whitelist được chọn`, hệ thống mới hiển thị vùng chọn/upload tệp whitelist.
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

**Modal chọn / xem blacklist campaign:**

```
┌──────────────────────────────────────────────────────────────┐
│ Chọn tệp blacklist campaign                                  │
├──────────────────────────────────────────────────────────────┤
│ [Tìm tên tệp blacklist...]                                   │
│                                                              │
│ ○ BL_TEST_INTERNAL        40 KH     10/05/2026 15:00         │
│ ● BL_ESIM_Q2_2026       320 KH     12/05/2026 09:00          │
│ ○ BL_COMPLAINT_USERS    180 KH     11/05/2026 10:30          │
│                                                              │
│ Preview tệp đang chọn                                        │
│ Số hợp lệ: 320 · Trùng: 8 · Sai định dạng: 2                 │
│ 0987 xxx 090                                                 │
│ 0912 xxx 091                                                 │
│ 0965 xxx 092                                                 │
│                                                              │
│                         [Hủy] [Áp dụng blacklist campaign]   │
└──────────────────────────────────────────────────────────────┘
```

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
│ Banner: 1/3 ⚠               │
│ Email: 0/3                   │
│ USSD: 0/3                    │
│                              │
│ Channel Strategy              │
│ Push → Zalo → SMS             │
│ Stop fallback: Clicked        │
│                              │
│ Safety                        │
│ DNC: Enabled                  │
│ Whitelist: WL_ESIM_Q2_2026    │
│ Reach sau whitelist: ~1,120 KH│
│                              │
│ Risk / Cost                   │
│ Estimated SMS: 2,400 segment  │
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
- Click vào item lỗi, ví dụ `Banner: 1/3` hoặc `USSD: 0/3`, scroll tới đúng tab/kênh còn thiếu.
- Cost/SMS risk cập nhật khi thay đổi channel strategy, segment, whitelist, fallback condition hoặc số ký tự SMS.

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
6. Cấu hình Channel Strategy và An toàn: thứ tự kênh, DNC, whitelist, frequency cap
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

## Screen 5: Trigger Management — UPDATED v7

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ TRIGGER MANAGEMENT                              [+ Thêm Trigger]    │
├─────────────────────────────────────────────────────────────────────┤
│ Trạng thái: [Tất cả ▾]    Loại thông báo: [Tất cả ▾]              │
└─────────────────────────────────────────────────────────────────────┘
```

### Bảng danh sách Trigger

```
┌──────────────────────┬────────────────────────────┬──────────────────────┬──────────────────┬───────────┬─────────────┐
│ Trigger Code         │ Tên Trigger                │ Event Source         │ Loại thông báo   │ Trạng thái│ Hành động   │
├──────────────────────┼────────────────────────────┼──────────────────────┼──────────────────┼───────────┼─────────────┤
│ SIM_ACTIVATED        │ Kích hoạt SIM thành công   │ BSS SIM_ACTIVATED    │ Realtime         │ ● Active  │ [Sửa][Tắt] │
│ NO_APP_INSTALL_24H   │ Chưa cài app sau 24h       │ CRM NO_APP_INSTALL_24H│ Near Realtime   │ ● Active  │ [Sửa][Tắt] │
│ LOW_DATA_BALANCE     │ Data còn dưới 100MB        │ OCS LOW_DATA_BALANCE │ Offline          │ ● Active  │ [Sửa][Tắt] │
│ LOC_TRAVEL_PROV      │ Đến tỉnh du lịch           │ OCS LOC_TRAVEL_PROV  │ Realtime         │ ● Active  │ [Sửa][Tắt] │
│ USAGE_DROP_40PCT     │ Usage giảm >40% baseline   │ OCS USAGE_DROP_40PCT │ Offline          │ ○ Testing │ [Sửa][Bật] │
│ SIM_NO_TXN_30D       │ SIM ngừng GD >30 ngày      │ BSS SIM_NO_TXN_30D   │ Offline          │ ○ Paused  │ [Sửa][Bật] │
└──────────────────────┴────────────────────────────┴──────────────────────┴──────────────────┴───────────┴─────────────┘
```

**Hành vi:**
- Filter "Loại thông báo" → lọc bảng theo Realtime / Near Realtime / Offline
- Filter "Trạng thái" → lọc theo Active / Paused / Testing
- [Tắt/Bật]: confirm dialog trước khi toggle

---

### Modal Thêm/Sửa Trigger — UPDATED v7

```
┌────────────────────────────────────────────────────────────┐
│ Thêm Trigger Mới                                           │
├────────────────────────────────────────────────────────────┤
│ Trigger code *                                             │
│ [SIM_ACTIVATED____________________________]                │
│                                                            │
│ Tên trigger *                                              │
│ [_________________________________________]                │
│                                                            │
│ Event source *                                             │
│ [BSS / OCS / CRM / Other ▾]                                │
│                                                            │
│ Mô tả                                                      │
│ [_________________________________________]                │
│                                                            │
│ Loại thông báo *                                           │
│ [Realtime / Near Realtime / Offline ▾]                     │
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
- `Trigger code` chỉ cho phép chữ in hoa, số và dấu gạch dưới; không được trùng với trigger code đã có.
- `Loại thông báo` gồm 3 giá trị: Realtime, Near Realtime, Offline.
- Không cấu hình `Giai đoạn vòng đời` và `Độ ưu tiên` trong master trigger. Priority khi xung đột được cấu hình ở cấp campaign trong Campaign Builder.
- [Lưu] → đóng modal + cập nhật bảng + toast "Đã lưu trigger ✓"

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
| SMS vượt 160 ký tự | Đếm đỏ + cảnh báo, không block submit; cost tính theo `ceil(số ký tự / 160)` |
| SMS 300/160 ký tự | Hiển thị `2 SMS segment`; cost = reach SMS × 2 segment |
| USSD chưa cấu hình | Hiển thị `USSD: 0/3` ở summary và scroll tới tab USSD khi click |
| Gửi duyệt khi còn trống | Validate từ fixed header: highlight các field bắt buộc còn trống |
| Tắt trigger đang được campaign sử dụng | Hiện confirm dialog cảnh báo ảnh hưởng trước khi tắt |
| Trigger priority trùng nhau trong campaign | Cảnh báo inline và yêu cầu QTV kéo-thả/chọn lại P1/P2/P3 để có thứ tự rõ ràng |
| KH thỏa 3 trigger cùng lúc | Nếu OR mode, chỉ gửi message của trigger có priority cao nhất trong campaign |
| Chọn 0 segment, 0 điều kiện | Mặc định gửi T-ALL (tất cả KH), hiện thông báo xác nhận |
| Segment OR + điều kiện AND | Ước tính cập nhật realtime khi thay đổi bất kỳ điều kiện nào |
| Bật whitelist nhưng chưa chọn tệp | Blocking validation: yêu cầu chọn tệp whitelist hoặc chuyển về không dùng whitelist |
| KH thuộc audience nhưng không thuộc whitelist | Loại khỏi reach và không gửi |
| Whitelist có số sai định dạng | Bỏ qua số sai định dạng, hiển thị số lượng bị bỏ qua trong preview |
