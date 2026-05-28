# Wireframe CVM System — v9
> Cập nhật từ v8: thêm Campaign Detail View (Screen 2B); bổ sung card soạn nội dung đầy đủ cho Zalo OA, Banner, Email trong Section 4; làm rõ behavior Audience Variant trong AND mode; đồng bộ 2 options Blackout handling.

---

## Thay đổi so với v8

| Thành phần | v8 | v9 |
|---|---|---|
| Campaign Detail View | Thiếu — chỉ mention trong hành vi [Xem] | Thêm Screen 2B đầy đủ (layout, fields, hành vi per trạng thái) |
| Message Matrix — Zalo OA | Chỉ có bảng tóm tắt preview format | Card soạn nội dung đầy đủ 2 cột (Soạn \| Preview) cho từng trigger |
| Message Matrix — Banner | Chỉ có bảng tóm tắt preview format | Card soạn nội dung đầy đủ: image bắt buộc 16:9, Title, Body, CTA Label, CTA URL |
| Message Matrix — Email | Chỉ có bảng tóm tắt preview format | Card soạn nội dung đầy đủ: header image, Subject, Body (plain text) |
| Audience Variant — AND mode | Không rõ button `[+ Biến thể đối tượng]` xử lý thế nào | Button bị ẩn trong AND mode; confirm dialog khi chuyển OR → AND |
| Section An toàn — Blackout | Hành vi liệt kê 3 options (không khớp mockup) | Đồng bộ đúng 2 options: "Hủy luôn" / "Delay đến đầu khung giờ" |

---

## Thay đổi so với v7

| Thành phần | v7 | v8 |
|---|---|---|
| Dashboard | Bảng thống kê phẳng | Card metrics với sparkline mini + shortcut action + cảnh báo contextual |
| Campaign List | Bảng dài, cột rườm rà | Status chip màu sắc, progress bar reach, filter tag nhanh |
| Campaign Builder — cột phải | Smart Panel liệt kê toàn bộ summary | Bỏ Smart Panel — bố cục 2 cột: trái scroll (S1/S2/S4/S6), phải sticky (S3/S5 + Campaign Summary mini) |
| Section 2 — Trigger | 3 trigger mẫu, card to, payload box | 2 trigger mẫu, card compact 2 dòng, payload chip |
| Section 3 — Audience | Checkbox list hiển thị sẵn | Dropdown chọn phân khúc, chọn mới hiện ra; điều kiện lọc chỉ chọn giá trị cố định |
| Section 4 — Message Matrix | Không có cấu hình thời gian gửi | Thêm khối Thời gian gửi trong mỗi message card |
| Section 5 — Channel Strategy | Chưa rõ đặt lịch push | Bổ sung đặt lịch gửi theo kênh |
| Section 6 — An toàn | Frequency Cap ở campaign level | Bỏ Frequency Cap (chuyển lên System Settings); giữ Blackout + DNC/BL/WL |
| Section 7 — Lịch gửi | Section riêng read-only | Bỏ hoàn toàn |
| Trigger Management | Bảng đơn, không nhóm | Group collapsible theo Realtime / Near Realtime / Offline |
| Template Management | Placeholder | Viết đầy đủ |
| Customer List | Placeholder | Viết đầy đủ |
| Report | Placeholder | Viết đầy đủ |

---

## Screen 1: Dashboard

### Mục tiêu
Operational monitoring — theo dõi realtime sức khỏe hệ thống, hiệu suất campaign và các bất thường cần xử lý ngay.

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ [≡] CVM                          LIVE ● 13/05/2026 09:45   🔔  👤       │
├────────────────┬────────────────────────────────────────────────────────┤
│ NAV LEFT       │ MAIN CONTENT (scroll dọc)                              │
│ (fixed)        │                                                        │
│ Dashboard      │ ROW 1 — TOP KPI CARDS (8 card)                        │
│ Campaign       │ ROW 2 — SYSTEM HEALTH (2 chart + queue + errors)       │
│ Template       │ ROW 3 — CAMPAIGN MONITORING (2 bảng + timeline)        │
│ Trigger        │ ROW 4 — USER JOURNEY FUNNEL                            │
│ Customer       │ ROW 5 — TRIGGER ANALYTICS (top + heatmap + anomaly)    │
│ Blacklist      │                                                        │
│ Report         │                                                        │
│ Settings       │                                                        │
└────────────────┴────────────────────────────────────────────────────────┘
```

---

### ROW 1 — Top KPI Cards (8 card, 4+4)

```
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Active Campaigns │ │ Trigger Fired    │ │ Messages Sent    │ │ Delivery Rate    │
│                  │ │ Today            │ │ Today            │ │                  │
│   12             │ │   3,842          │ │   48,320         │ │   96.4%          │
│ ↑2 vs hôm qua   │ │ ↑18% vs hôm qua  │ │ ▁▃▅▇▆▅▇          │ │ ▇▇▇▆▇▇▆          │
│ ▁▃▄▅▆▅▇          │ │ ▂▄▄▅▆▇▆          │ │                  │ │ ● SLA: OK        │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘

┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Failed Messages  │ │ Conversion Rate  │ │ Realtime SLA     │ │ Blacklist Blocked│
│                  │ │                  │ │                  │ │ Today            │
│   1,760          │ │   8.3%           │ │   < 2.1s avg     │ │   4,210          │
│ ⚠ ↑3% vs hôm qua│ │ ↑0.4% vs tuần   │ │ ✅ Target: <3s   │ │ ↑ BSS DNC: 3,840 │
│ ▂▃▂▄▅▆▇ ↑ trend │ │ ▄▅▄▅▆▅▆          │ │ ▂▂▂▃▂▂▃          │ │ CVM BL: 370      │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘
```

**Quy tắc hiển thị card:**
- Tất cả card cập nhật mỗi 60 giây.
- Card có trend xấu (Failed tăng, SLA vượt ngưỡng) → viền đỏ + icon ⚠.
- Sparkline 7 ngày gần nhất; hover → tooltip "Ngày X: giá trị Y".
- Click card → drill-down đến màn hình chi tiết tương ứng.

---

### ROW 2 — System Health

```
┌─────────────────────────────────────┬─────────────────────────────────────┐
│ TRIGGER / EVENT VOLUME              │ PROCESSING LATENCY                  │
│ Line chart — 24h gần nhất           │ Line chart — 24h gần nhất           │
│                                     │                                     │
│  4K ┤           ╭──╮                │  3s ┤                               │
│  3K ┤      ╭────╯  ╰──╮            │  2s ┤  ╭──╮          ╭──╮           │
│  2K ┤  ╭───╯           ╰──         │  1s ┤──╯  ╰──────────╯  ╰───────    │
│  1K ┤──╯                           │  0s ┤                               │
│     └──────────────────────────── h│     └────────────────────────────── h│
│     00  04  08  12  16  20  24     │     00  04  08  12  16  20  24      │
│ ● Realtime  ● Near RT  ● Offline   │ ● p50  ● p95  ● p99                 │
│                                     │ ─── SLA target (3s)                 │
└─────────────────────────────────────┴─────────────────────────────────────┘

┌─────────────────────────────────────┬─────────────────────────────────────┐
│ QUEUE & BACKLOG                     │ TOP SYSTEM ERRORS                   │
│                                     │                                     │
│ Processing queue:  342 events       │ ⛔ GATEWAY_ZALO_TIMEOUT  · 128 lần  │
│ ████████████░░░░   68% capacity     │    Zalo OA API timeout > 5s         │
│                                     │    Lần gần nhất: 09:32             │
│ Pending (blackout):  1,240          │                                     │
│ Scheduled (future):  8,900          │ ⚠ SMS_SEGMENT_EXCEED   · 45 lần    │
│                                     │    Tin nhắn vượt 2 segment          │
│ Oldest pending:  02:15 ago          │                                     │
│ ⓘ Blackout queue flush lúc 08:00   │ ⚠ DEDUP_COLLISION       · 12 lần    │
│                                     │    Event ID trùng bị bỏ qua         │
│                        [Xem queue]  │                      [Xem tất cả →] │
└─────────────────────────────────────┴─────────────────────────────────────┘
```

---

### ROW 3 — Campaign Monitoring

```
┌──────────────────────────────────────┬──────────────────────────────────────┐
│ TOP ACTIVE CAMPAIGNS                 │ TRIGGER FIRED NHIỀU NHẤT HÔM NAY     │
├──────────────┬──────┬──────┬─────────┼──────────────────────┬──────┬────────┤
│ Campaign     │ Sent │ Rate │ Trend   │ Trigger              │ Fired│ Match  │
├──────────────┼──────┼──────┼─────────┼──────────────────────┼──────┼────────┤
│ Nhắc nạp tiền│18,200│96.5% │ ▂▃▄▅▆▇  │ LOW_DATA_BALANCE     │ 1,842│  68%   │
│ Hết hạn data │12,400│95.4% │ ▅▄▅▆▅▆  │ SIM_ACTIVATED        │  940 │  91%   │
│ LOW_DATA batch│9,800│94.2% │ ▆▇▇▆▇▇  │ NO_APP_INSTALL_24H   │  620 │  44%   │
│ Welcome eSIM  │8,100│93.1% │ ▃▄▃▅▄▅  │ TOP_UP_FAIL          │  440 │  82%   │
│              [Xem tất cả →]          │                     [Xem tất cả →]   │
└──────────────────────────────────────┴──────────────────────────────────────┘

RECENT TRIGGER EVENTS TIMELINE
┌─────────────────────────────────────────────────────────────────────────┐
│ 09:44:52  LOW_DATA_BALANCE   ·  0987 xxx 001  →  Matched · Push sent    │
│ 09:44:38  SIM_ACTIVATED      ·  0912 xxx 002  →  Matched · Zalo sent    │
│ 09:44:21  NO_APP_INSTALL_24H ·  0965 xxx 003  →  No match               │
│ 09:44:05  TOP_UP_FAIL        ·  0976 xxx 004  →  Blacklist blocked       │
│ 09:43:58  LOW_DATA_BALANCE   ·  0988 xxx 005  →  Matched · DNC blocked  │
│                                                        [Live ● pause]   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### ROW 4 — User Journey Funnel

```
┌─────────────────────────────────────────────────────────────────────────┐
│ USER JOURNEY FUNNEL  ·  Welcome eSIM Q2/2026  ·  [Chọn campaign ▾]     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SIM Activated          ████████████████████████████  32,100  100%      │
│  ↓ drop 8%                                                              │
│  Message Delivered      ███████████████████████████   29,530   92%      │
│  ↓ drop 61%                                                             │
│  Message Opened         ███████████████             12,200   38%        │
│  ↓ drop 23%                                                             │
│  App Installed          ████████████                 9,400   29%        │
│  ↓ drop 20%                                                             │
│  Package Purchased      ████████                     5,800   18%        │
│                                                                         │
│  Conversion rate (SIM → Purchase): 18.1%  ·  vs tuần trước: ↑1.2%      │
│                                                    [Xem chi tiết →]     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### ROW 5 — Trigger Analytics

```
┌──────────────────────────────────┬──────────────────────────────────────┐
│ TOP TRIGGERS (7 ngày)            │ TRIGGER ANOMALY DETECTION            │
├──────────────┬──────┬────────────┼──────────────────────────────────────┤
│ Trigger      │ Fired│ Match rate │ ⚠ LOW_DATA_BALANCE                   │
├──────────────┼──────┼────────────┤    Volume tăng đột biến +340%        │
│ LOW_DATA     │12,400│  68%  ████ │    So sánh: avg 3,600/ngày           │
│ SIM_ACTIVATED│ 6,580│  91%  ████ │    Hôm nay: 12,400 · 09:00–09:45    │
│ NO_APP_24H   │ 4,340│  44%  ██░  │    → Có thể do batch job OCS re-run  │
│ TOP_UP_FAIL  │ 3,080│  82%  ████ │                     [Xem chi tiết →] │
│ LOC_TRAVEL   │   920│  77%  ███░ │                                      │
│              [Xem tất cả →]      │ ✅ Các trigger còn lại: bình thường  │
└──────────────────────────────────┴──────────────────────────────────────┘

TRIGGER HEATMAP — Hoạt động theo giờ trong ngày (7 ngày gần nhất)
┌─────────────────────────────────────────────────────────────────────────┐
│       00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23
│ Mon   ░  ░  ░  ░  ░  ░  ░  ▒  █  █  █  ▓  ▓  █  █  █  ▓  ▓  █  ▒  ░  ░  ░  ░
│ Tue   ░  ░  ░  ░  ░  ░  ░  ▒  █  █  ▓  ▓  █  █  █  █  ▓  █  █  ▒  ░  ░  ░  ░
│ Wed   ░  ░  ░  ░  ░  ░  ▒  ▒  █  █  █  █  ▓  █  █  ▓  █  █  ▓  ▒  ░  ░  ░  ░
│ Thu   ░  ░  ░  ░  ░  ░  ░  ▒  █  ██ █  ▓  ▓  █  █  █  █  ▓  █  ▒  ░  ░  ░  ░
│ Fri   ░  ░  ░  ░  ░  ░  ░  ▒  █  █  █  █  █  █  █  █  ▓  █  █  ▓  ▒  ░  ░  ░
│ Sat   ░  ░  ░  ░  ░  ░  ░  ░  ▒  ▓  █  ▓  █  █  ▓  ▓  ▓  ▒  ▒  ░  ░  ░  ░  ░
│ Sun   ░  ░  ░  ░  ░  ░  ░  ░  ▒  ▓  ▓  ▓  █  ▓  ▓  ▓  ▒  ▒  ░  ░  ░  ░  ░  ░
│
│ ░ thấp   ▒ trung bình   ▓ cao   █ rất cao   ██ đột biến
└─────────────────────────────────────────────────────────────────────────┘
```

### Hành vi Dashboard
- Toàn bộ card và chart cập nhật mỗi 60 giây; timeline trigger events cập nhật realtime.
- Card KPI vượt ngưỡng xấu → viền đỏ + toast cảnh báo.
- `[Live ● pause]` trên timeline: toggle dừng/tiếp tục stream để đọc dữ liệu.
- Heatmap: hover ô → tooltip "Thứ X · Giờ Y: Z triggers".
- Anomaly detection: so sánh volume hiện tại với baseline 7 ngày trước, cảnh báo nếu lệch >200%.
- Click bất kỳ campaign/trigger trong bảng → điều hướng đến màn chi tiết tương ứng.

---

## Screen 2: Campaign List

### Mục tiêu
Scan nhanh trạng thái các campaign, lọc linh hoạt, tạo mới hoặc vào sửa với ít thao tác nhất.

### Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Campaign                                           [+ Tạo Campaign]     │
├─────────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm tên campaign, mã hoặc trigger code...]                          │
│ FILTER:  [Tất cả ×]  [Active]  [Draft]  [Pending]  [Paused]             │
│          [Realtime]  [Offline]  [Có SMS]  [Chưa đủ nội dung]            │
├──────────────────────────────────┬────────────┬────────────┬────────────┬──────────────┤
│ TÊN / MÃ CAMPAIGN                │ TRIGGER     │ HIỆU LỰC  │ TRẠNG THÁI │ HÀNH ĐỘNG    │
├──────────────────────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Welcome eSIM Q2/2026             │ [SIM_ACT]  │ 01/04 –    │ ● Draft    │ [Xem] [Sửa] │
│ CVM-WELCOME-ESIM-Q2-2026         │ [NO_APP]   │ 30/06/2026 │            │              │
│                                  │ +1 ⓘ       │            │            │              │
├──────────────────────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Nhắc nạp tiền                    │ [TOP_UP]   │ 01/05 –    │ ● Active   │ [Xem][Dừng] │
│ CVM-REMIND-TOPUP-05-2026         │            │ 31/05/2026 │            │              │
├──────────────────────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Hết hạn gói data                 │ [PKG_EXP]  │ 01/05 –    │ ● Paused   │ [Xem] [Bật] │
│ CVM-DATA-EXPIRE-05-2026          │            │ 31/05/2026 │            │              │
├──────────────────────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Chào mừng du lịch                │ [LOC_TRV]  │ 15/05 –    │ ⏳ Pending │ [Xem]        │
│ CVM-TRAVEL-PROMO-Q2              │            │ 30/06/2026 │ Chờ duyệt  │              │
└──────────────────────────────────┴────────────┴────────────┴────────────┴──────────────┘
                                                              < 1 2 3 >  [20/trang ▾]
```

**Status chip màu sắc:**

| Trạng thái | Màu | Hành động |
|---|---|---|
| Active | Xanh lá `#22C55E` | [Xem] [Dừng] |
| Draft | Xám `#94A3B8` | [Xem] [Sửa] |
| Pending | Vàng `#EAB308` | [Xem] |
| Paused | Cam `#F97316` | [Xem] [Bật] |

**Cột Trigger:** Hiển thị tối đa 2 chip `[CODE]`; nếu campaign có nhiều hơn, hiện `+N ⓘ` — hover/click `ⓘ` mở popover danh sách đầy đủ với tên trigger + source + kiểu chạy.

### Hành vi
- Filter tag: multi-select; click lại để bỏ.
- Tìm kiếm: realtime theo tên campaign, mã campaign hoặc trigger code.
- `[Dừng]` trên campaign Active = Kill Switch tức thì — confirm dialog trước khi thực thi; message đang trong queue sẽ bị hủy.
- `[Bật]` trên campaign Paused: reactivate không cần duyệt lại nếu không có thay đổi nội dung.
- `[Xem]` mở Campaign Detail View chỉ đọc — hiển thị toàn bộ thông tin khai báo.

---

## Screen 2B: Campaign Detail View

### Mục tiêu
Xem toàn bộ thông tin chi tiết một campaign ở chế độ chỉ đọc — dành cho QTV xem lại campaign đang Pending/Active/Paused/Ended, hoặc mở từ nút `[Xem]` trong Campaign List.

### Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Campaign                                                           │
│ Welcome eSIM Q2/2026  ·  CVM-WELCOME-ESIM-Q2-2026     ⏳ Pending    │
│                                                          [Đóng]     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ ── 1. Thông tin Campaign ────────────────────────────────────────── │
│ Tên campaign:     Welcome eSIM Q2/2026                              │
│ Mã kịch bản:      CVM-WELCOME-ESIM-Q2-2026                         │
│ Mục tiêu:         Onboard eSIM, nhắc cài app                       │
│ Thời gian:        01/04/2026 – 30/06/2026                           │
│ Người tạo:        QTV Marketing                                     │
│ Ngày tạo:         10/05/2026 14:32                                  │
│ Ngày gửi duyệt:   12/05/2026 09:15                                  │
│                                                                     │
│ ── 2. Trigger & Logic ───────────────────────────────────────────── │
│ Chế độ:  Advanced  ·  Logic: OR                                     │
│ Trigger:  ★ P1  SIM_ACTIVATED · Kích hoạt SIM thành công           │
│           P2  NO_APP_INSTALL_24H · Chưa cài app sau 24h             │
│ Ưu tiên khi match nhiều trigger: Chỉ gửi trigger P1                 │
│ Ước tính tin:  ~6,800 KH × 2 kênh = ~13,600 tin                    │
│                                                                     │
│ ── 3. Audience / Phân khúc ──────────────────────────────────────── │
│ Phân khúc:  Gen Z User (18–25)  ·  Sắp hết data                    │
│ Logic:  Bất kỳ phân khúc nào (OR)                                   │
│ Điều kiện lọc:  Loại thiết bị = Android                             │
│ Reach ước tính:  ~6,800 KH                                          │
│                                                                     │
│ ── 4. Message Matrix ────────────────────────────────────────────── │
│ Thời gian gửi:  Gửi ngay sau khi trigger kích hoạt                  │
│                                                                     │
│ [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○]  │
│                                                                     │
│ ┌──────────────────────────────────────────────────────────────┐   │
│ │ T1 · SIM_ACTIVATED  ·  Push                                  │   │
│ │ Title: Chào mừng bạn!                                        │   │
│ │ Body:  SIM eSIM đã kích hoạt thành công. Khám phá MyViettel. │   │
│ └──────────────────────────────────────────────────────────────┘   │
│ ┌──────────────────────────────────────────────────────────────┐   │
│ │ T2 · NO_APP_INSTALL_24H  ·  Push                             │   │
│ │ Title: Cài MyViettel nhận ưu đãi                             │   │
│ │ Body:  Bạn vẫn chưa cài app MyViettel. Cài ngay để...        │   │
│ └──────────────────────────────────────────────────────────────┘   │
│ (Các tab kênh còn lại hiển thị tương tự — click tab để xem)        │
│                                                                     │
│ ── 5. Channel Strategy ──────────────────────────────────────────── │
│ Đặt lịch theo kênh:  Không (tất cả theo thời gian gửi message)     │
│                                                                     │
│ ── 6. An toàn ───────────────────────────────────────────────────── │
│ Blackout:          Bật  ·  22:00 – 08:00  ·  Hủy luôn              │
│ DNC toàn hệ thống: Bật                                              │
│ Blacklist campaign: BL_ESIM_Q2_2026  ·  320 KH                     │
│ Whitelist:         Không dùng                                       │
│ Reach cuối cùng:   ~6,480 KH                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Toàn bộ nội dung chỉ đọc — không có ô nhập hay nút chỉnh sửa.
- Tiêu đề header hiển thị đúng trạng thái campaign với màu status chip tương ứng.
- Tab kênh trong Message Matrix: click để xem nội dung từng kênh; mặc định hiển thị tab đầu tiên đã có nội dung.
- `[Đóng]` → quay về màn hình trước (Campaign List hoặc Dashboard).
- Với campaign Draft: nút `[Sửa]` hiển thị thay `[Đóng]` → điều hướng sang Campaign Builder.
- Với campaign Active: nút `[Dừng]` hiển thị thêm cạnh `[Đóng]`.
- Với campaign Paused: nút `[Bật]` hiển thị thêm cạnh `[Đóng]`.

---

## Screen 3: Campaign Builder

### Mục tiêu
Tạo hoặc sửa campaign end-to-end. Bỏ Smart Panel và Section 7 — bố cục 2 cột: trái scroll (S1/S2/S4/S6), phải sticky (S3/S5 + Campaign Summary mini), header fixed luôn hiện trạng thái và nút action.

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ HEADER FIXED                                                            │
├────────────────────────────────────────┬────────────────────────────────┤
│ LEFT (~60%, scroll)                    │ RIGHT (~40%, sticky)           │
│────────────────────────────────────────│────────────────────────────────│
│ S1. Thông tin Campaign                 │ [CAMPAIGN SUMMARY MINI]        │
│ S2. Trigger & Logic                    │   ⚠ 2 issue · Reach ~6,480 KH │
│ S4. Message Matrix                     │   Chi phí SMS ~720,000 VND     │
│   ┌──────────────┬──────────┐          │────────────────────────────────│
│   │ Soạn message│ Preview  │          │ S3. Audience / Phân khúc       │
│   │ (trái ~55%) │ live     │          │   Gen Z · Sắp hết data         │
│   │             │(phải 45%)│          │   Reach: ~6,800 KH             │
│   └──────────────┴──────────┘          │────────────────────────────────│
│ S6. An toàn                            │ S5. Channel Strategy           │
│                                        │   Đặt lịch per kênh           │
└────────────────────────────────────────┴────────────────────────────────┘
```

**Quy tắc cột phải (sticky):**
- **Campaign Summary mini**: hiện issue count + reach cuối + cost SMS — cập nhật realtime khi thay đổi audience, channel hoặc suppression.
- **S3 Audience**: hiển thị các segment đã chọn + reach; có thể thêm/bỏ segment trực tiếp từ cột phải mà không cần cuộn sang cột trái.
- **S5 Channel Strategy**: accordion đặt lịch per kênh — mở/đóng từng kênh độc lập.
- Cột phải **sticky toàn trang** — không cuộn theo nội dung cột trái.

---

### Header Fixed

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ← Campaign                                                              │
│ Welcome eSIM Q2/2026  ·  CVM-WELCOME-ESIM-Q2-2026        ● Draft        │
│                                 [Lưu Nháp]  [Gửi duyệt →  ⚠ 2 issue]  │
└─────────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- `[Gửi duyệt →]` disabled khi còn issue blocking; badge đỏ hiện số issue.
- Hover nút disabled → tooltip liệt kê ngắn các issue đang block.
- `[Lưu Nháp]` luôn enabled.

---

### Section 1: Thông tin Campaign

```
┌────────────────────────────────────────────────────────────────────┐
│ 1. Thông tin Campaign                                    [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ Tên campaign *                                                     │
│ [Welcome eSIM Q2/2026___________________________________________]  │
│                                                                    │
│ Mã kịch bản                                                        │
│ [CVM-WELCOME-ESIM-Q2-2026_______________________________________]  │
│                                                                    │
│ Mục tiêu                            Thời gian hiệu lực            │
│ [Onboard eSIM, nhắc cài app____]    01/04/2026  –  30/06/2026     │
│                                                                    │
│ Người tạo: QTV Marketing                                           │
└────────────────────────────────────────────────────────────────────┘
```

---

### Section 2: Trigger & Logic

```
┌────────────────────────────────────────────────────────────────────┐
│ 2. Trigger & Logic                                       [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ Chế độ:  ○ Basic (1 trigger)   ● Advanced (nhiều trigger + logic)  │
│                                                                    │
│ ── Trạng thái ban đầu (chưa chọn trigger) ─────────────────────── │
│                                                                    │
│  [+ Chọn trigger ▾]                                                │
│                                                                    │
│  ⓘ Chưa có trigger. Nhấn "+ Chọn trigger" để bắt đầu.             │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│ ── Sau khi đã chọn trigger (Advanced mode, ví dụ) ──────────────── │
│                                                                    │
│  [+ Chọn trigger ▾]                                                │
│                                                                    │
│  ★ P1  [SIM_ACTIVATED · Kích hoạt SIM thành công]  [≡] [✕]        │
│     P2  [NO_APP_INSTALL_24H · Chưa cài app sau 24h]  [≡] [✕]      │
│                                                                    │
│  Hover chip → tooltip: Source · Kiểu chạy · Payload               │
│  Kéo [≡] để đổi priority; ★ luôn đánh dấu P1                      │
│                                                                    │
│ Logic:  ● OR   ○ AND                                               │
│                                                                    │
│ ── Khi chọn OR ──────────────────────────────────────────────────  │
│ Mỗi trigger có message riêng. KH match trigger nào → nhận message  │
│ của trigger đó.                                                    │
│ Nếu KH match nhiều trigger cùng lúc:                               │
│ ● Chỉ gửi trigger ưu tiên cao nhất (★ P1)                          │
│ ○ Gửi tất cả trigger đã match                                      │
│                                                                    │
│ ── Khi chọn AND ─────────────────────────────────────────────────  │
│ KH phải thỏa đồng thời tất cả trigger mới được gửi tin.            │
│ Tất cả trigger dùng chung 1 message cho mỗi kênh.                  │
│                                                                    │
│ ── Estimate (hiện sau khi đã chọn trigger + audience) ───────────  │
│ Ước tính tin nhắn cần gửi:  ~6,800 KH × 2 kênh = ~13,600 tin      │
│ Trong đó SMS: ~1,200 KH × 2 segment = ~2,400 SMS                  │
│ Chi phí SMS ước tính: ~720,000 VND                                 │
│ ⓘ Tính dựa trên audience hiện tại và kênh đã cấu hình             │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Mặc định Section 2 hiển thị trạng thái trống với nút `[+ Chọn trigger ▾]` — không pre-fill trigger.
- Dropdown chọn trigger: tìm kiếm theo code hoặc tên; hiện danh sách trigger Active; chọn → xuất hiện dạng chip dọc.
- Chip trigger hiển thị `[CODE · Tên trigger ngắn]`; hover → tooltip đầy đủ: Source / Kiểu chạy / danh sách payload.
- Basic mode: chỉ cho chọn 1 trigger; ẩn Logic OR/AND và khối xung đột.
- Advanced mode: cho chọn nhiều trigger; hiện Logic + diễn giải tương ứng.
- Kéo handle `[≡]` để reorder priority; badge `★` luôn đánh dấu P1.
- Xóa `[✕]`: nếu xóa P1, trigger tiếp theo tự động lên P1.
- Khối Estimate chỉ hiện sau khi đã có trigger VÀ audience; cập nhật realtime khi thay đổi audience hoặc kênh ở Message Matrix.

---

### Section 3: Audience / Phân khúc

```
┌────────────────────────────────────────────────────────────────────┐
│ 3. Audience / Phân khúc                                  [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ Nguồn: Customer 360 · Team Data · BSS · OCS                        │
│ Cập nhật: 13/05/2026 08:00                                         │
│                                                                    │
│ Chọn phân khúc                                                     │
│ [🔍 Tìm phân khúc... ▾]                                            │
│                                                                    │
│ Đã chọn:                                                           │
│ ┌─────────────────────────────┐  ┌─────────────────────────────┐  │
│ │ Gen Z User (18–25)          │  │ Sắp hết data                │  │
│ │ 18,450 KH                   │  │ 8,920 KH                    │  │
│ │                         [×] │  │                         [×] │  │
│ └─────────────────────────────┘  └─────────────────────────────┘  │
│                                                                    │
│ Logic: ● Bất kỳ phân khúc nào (OR)   ○ Tất cả phân khúc (AND)    │
│                                                                    │
│ Điều kiện lọc thêm (optional)  ▸ [Mở rộng]                        │
│                                                                    │
│ ── Khi mở rộng ────────────────────────────────────────────────── │
│ [Loại thiết bị ▾]  [= ▾]  [Android ▾]  [×]                        │
│ [+ Thêm điều kiện]                                                 │
│ ⓘ Giá trị được cấu hình sẵn — chỉ chọn, không nhập tự do          │
│ ────────────────────────────────────────────────────────────────── │
│                                                                    │
│ Ước tính reach: ~6,800 KH  [↻ Tính lại]                           │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Dropdown "Tìm phân khúc": gõ để lọc; chọn → hiện dạng tag card bên dưới, không hiện checkbox list sẵn.
- Xóa `[×]` trên tag card → bỏ phân khúc đó; reach cập nhật lại.
- Điều kiện lọc: tất cả field đều là dropdown chọn giá trị cố định — không có ô nhập tự do.
- Ước tính reach cập nhật realtime khi thay đổi phân khúc hoặc điều kiện lọc.
- CVM không tạo segment master — segment do Team Data/BSS/OCS cung cấp.

---

### Section 4: Message Matrix

```
┌────────────────────────────────────────────────────────────────────┐
│ 4. Message Matrix  ·  Trigger × Kênh                     [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ ⏱ THỜI GIAN GỬI (áp dụng cho toàn campaign)                       │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ● Gửi ngay sau khi trigger kích hoạt                           │ │
│ │ ○ Sau  [__] [phút / giờ / ngày ▾]  kể từ T                    │ │
│ │ ○ Vào lúc  [__:__]  ngày  T + [__]                             │ │
│ │                                                                │ │
│ │ ⓘ T = thời điểm trigger kích hoạt cho từng KH cụ thể          │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ TIẾN ĐỘ NỘI DUNG:                                                  │
│ Push ●●  Zalo ●●  SMS ●○  Banner ○○  Email ○○  USSD ○○             │
│ (● = hoàn chỉnh  ○ = chưa cấu hình)                                │
│                                                                    │
│ Tab kênh:  [Push ●●]  [Zalo OA ●●]  [SMS ●○]  [Banner ○○]         │
│            [Email ○○]  [USSD ○○]  [+ Kênh]                        │
│                                                                    │
│ ── Tab Push (đang active) ──────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Push notification mockup]       │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (click chip → chèn vào nội dung)        │ │ 🔔 App icon   Title here   │   │
│ │                                         │ │    Body text rendered...   │   │
│ │ Image (1:1, optional)                   │ │ [thumbnail nếu có ảnh]     │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │ SAMPLE PARAMS                    │
│ │ │     1:1 khuyến nghị              │   │ ten_kh    [Nguyễn Văn A  ]       │
│ │ └──────────────────────────────────┘   │ loai_sim  [eSIM          ]       │
│ │ (Nếu đã upload: thumbnail + [Xóa][Đổi])│ ngay_kt   [01/04/2026    ]       │
│ │                                         │ so_dt     [0987 xxx 001  ]       │
│ │ Template: [Welcome Push ▾]              │                                  │
│ │                                         │ (nhập sample → render realtime)  │
│ │ Title *                    0/65         │                                  │
│ │ [Chào mừng bạn!__________________]     │                                  │
│ │                                         │                                  │
│ │ Body *                     0/240        │                                  │
│ │ [SIM {{loai_sim}} đã kích hoạt...]      │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Push notification mockup]       │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ 🔔 App icon   Title here   │   │
│ │ Image (1:1, optional)                   │ │    Body text rendered...   │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │ SAMPLE PARAMS                    │
│ │ │     1:1 khuyến nghị              │   │ app_installed  [false     ]       │
│ │ └──────────────────────────────────┘   │ last_active    [10/04/2026]       │
│ │ (Nếu đã upload: thumbnail + [Xóa][Đổi])│                                  │
│ │                                         │ (nhập sample → render realtime)  │
│ │ Template: [Install App ▾]               │                                  │
│ │                                         │                                  │
│ │ Title *                    0/65         │                                  │
│ │ [Cài MyViettel nhận ưu đãi________]    │                                  │
│ │                                         │                                  │
│ │ Body *                     0/240        │                                  │
│ │ [Bạn vẫn chưa cài app MyViettel...]    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab SMS ─────────────────────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim] [so_dt]     │ [SMS bubble mockup]              │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ Viettel                    │   │
│ │ (Không hỗ trợ hình ảnh)                 │ │ Chào Nguyễn Văn A! SIM     │   │
│ │                                         │ │ eSIM đã được kích hoạt...  │   │
│ │ Body *                   92/160         │ └────────────────────────────┘   │
│ │ [Nội dung SMS Welcome...]               │ 92/160 ký tự · 1 SMS segment    │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ⚠  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [SMS bubble mockup]              │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ (Chưa có nội dung)         │   │
│ │ (Không hỗ trợ hình ảnh)                 │ └────────────────────────────┘   │
│ │                                         │                                  │
│ │ Body *                    0/160         │                                  │
│ │ [Chưa cấu hình nội dung___________]    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab USSD ────────────────────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim] [so_dt]     │ [USSD terminal mockup]           │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ Chao Nguyen Van A          │   │
│ │ (Không hỗ trợ hình ảnh)                 │ │ SIM eSIM kich hoat thanh   │   │
│ │                                         │ │ cong. Bam 1 de...          │   │
│ │ Body *                    0/182         │ └────────────────────────────┘   │
│ │ [Chỉ plain text, không ký tự đặc biệt] │ 0/182 ký tự · plain text        │
│ │ ⚠ Chỉ plain text, không hỗ trợ ảnh    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [USSD terminal mockup]           │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ (Chưa có nội dung)         │   │
│ │ (Không hỗ trợ hình ảnh)                 │ └────────────────────────────┘   │
│ │                                         │                                  │
│ │ Body *                    0/182         │                                  │
│ │ [Chỉ plain text, không ký tự đặc biệt] │                                  │
│ │ ⚠ Chỉ plain text, không hỗ trợ ảnh    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab Zalo OA ─────────────────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Zalo OA chat bubble mockup]     │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (click chip → chèn vào nội dung)        │ │ 🟦 Viettel Telecom         │   │
│ │                                         │ │ [image nếu có]             │   │
│ │ Image (tự do tỉ lệ, optional)           │ │ Nội dung tin nhắn render   │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │ SAMPLE PARAMS                    │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │ (Nếu đã upload: thumbnail + [Xóa][Đổi])│ loai_sim  [eSIM          ]       │
│ │                                         │ (nhập sample → render realtime)  │
│ │ Template: [Zalo Welcome ▾]              │                                  │
│ │                                         │                                  │
│ │ Nội dung *                 0/1000       │                                  │
│ │ [Chào mừng {{ten_kh}}!______________]   │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Zalo OA chat bubble mockup]     │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ 🟦 Viettel Telecom         │   │
│ │ Image (tự do tỉ lệ, optional)           │ │ (Chưa có nội dung)         │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │                                  │
│ │ └──────────────────────────────────┘   │                                  │
│ │                                         │                                  │
│ │ Nội dung *                 0/1000       │                                  │
│ │ [Chưa cấu hình nội dung___________]    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab Banner ──────────────────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Banner card mockup]             │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (click chip → chèn vào nội dung)        │ │ [Image 16:9 full width]    │   │
│ │                                         │ │ Title text                 │   │
│ │ Image **bắt buộc** (16:9)               │ │ Body text                  │   │
│ │ ┌──────────────────────────────────┐   │ │ [CTA Button]               │   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │ └────────────────────────────┘   │
│ │ │     [Chọn thư viện]              │   │                                  │
│ │ │     Bắt buộc · tỉ lệ 16:9       │   │ SAMPLE PARAMS                    │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │ ⚠ Chưa upload image → block lưu       │ loai_sim  [eSIM          ]       │
│ │                                         │ (nhập sample → render realtime)  │
│ │ Template: [Banner Welcome ▾]            │                                  │
│ │                                         │                                  │
│ │ Title *                    0/65         │                                  │
│ │ [Chào mừng {{ten_kh}}!__________]      │                                  │
│ │                                         │                                  │
│ │ Body *                     0/120        │                                  │
│ │ [SIM {{loai_sim}} đã kích hoạt...]      │                                  │
│ │                                         │                                  │
│ │ CTA Label *                             │                                  │
│ │ [Khám phá ngay_________________]        │                                  │
│ │                                         │                                  │
│ │ CTA URL *                               │                                  │
│ │ [https://___________________________]   │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Banner card mockup]             │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ (Chưa có image)            │   │
│ │ Image **bắt buộc** (16:9)               │ │ (Chưa có nội dung)         │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     Bắt buộc · tỉ lệ 16:9       │   │                                  │
│ │ └──────────────────────────────────┘   │                                  │
│ │                                         │                                  │
│ │ Title *                    0/65         │                                  │
│ │ [_________________________________]     │                                  │
│ │                                         │                                  │
│ │ Body *                     0/120        │                                  │
│ │ [_________________________________]     │                                  │
│ │                                         │                                  │
│ │ CTA Label *                             │                                  │
│ │ [_________________________________]     │                                  │
│ │                                         │                                  │
│ │ CTA URL *                               │                                  │
│ │ [https://___________________________]   │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab Email ───────────────────────────────────────────────────   │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Email client mockup]            │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (click chip → chèn vào nội dung)        │ │ From: Viettel Telecom      │   │
│ │                                         │ │ Subject: [subject render]  │   │
│ │ Header Image (banner ngang, optional)   │ │ [header image nếu có]      │   │
│ │ ┌──────────────────────────────────┐   │ │ Body render plain text     │   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │ └────────────────────────────┘   │
│ │ │     [Chọn thư viện]              │   │                                  │
│ │ │     Banner ngang, optional       │   │ SAMPLE PARAMS                    │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │                                         │ loai_sim  [eSIM          ]       │
│ │ Template: [Email Welcome ▾]             │ (nhập sample → render realtime)  │
│ │                                         │                                  │
│ │ Subject *                  0/100        │                                  │
│ │ [Chào mừng {{ten_kh}} đến Viettel]     │                                  │
│ │                                         │                                  │
│ │ Body *  (plain text)                    │                                  │
│ │ [Xin chào {{ten_kh}},                   │                                  │
│ │  SIM {{loai_sim}} của bạn đã được       │                                  │
│ │  kích hoạt thành công ngày              │                                  │
│ │  {{ngay_kich_hoat}}...]                 │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                 │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Email client mockup]            │
│ │ (click chip → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ (Chưa có nội dung)         │   │
│ │ Header Image (banner ngang, optional)   │ └────────────────────────────┘   │
│ │ ┌──────────────────────────────────┐   │                                  │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ └──────────────────────────────────┘   │                                  │
│ │                                         │                                  │
│ │ Subject *                  0/100        │                                  │
│ │ [_________________________________]     │                                  │
│ │                                         │                                  │
│ │ Body *  (plain text)                    │                                  │
│ │ [_________________________________]     │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
└────────────────────────────────────────────────────────────────────┘
```

**Định nghĩa thời gian gửi:**

| Tùy chọn | Ý nghĩa | Dùng khi |
|---|---|---|
| Gửi ngay | Ngay khi trigger hit và KH pass filter | Realtime, phản hồi tức thì |
| Sau X phút/giờ/ngày kể từ T | T = lúc trigger kích hoạt cho KH đó | Delay có chủ ý |
| Vào lúc HH:MM ngày T+N | T+0 = ngày trigger hit; T+1 = hôm sau | Gửi vào khung giờ cố định |

**Image field per kênh** (trong cột Soạn, trước Title):

| Kênh | Image field |
|---|---|
| Push | Upload optional · ratio 1:1 · preview thumbnail trong mockup bên phải |
| Zalo OA | Upload optional · ratio tự do · hiện phía trên chat bubble trong preview |
| Banner | Upload **bắt buộc** · ratio 16:9 · hiện full width trong preview |
| Email | Upload optional · ratio banner ngang · hiện header image trong preview |
| SMS | Label "(Không hỗ trợ hình ảnh)" — không có upload |
| USSD | Label "(Không hỗ trợ hình ảnh)" — không có upload |

**Preview per kênh** (cột XEM TRƯỚC, render realtime):

| Kênh | Preview mockup |
|---|---|
| Push | OS notification: app icon trái + thumbnail 1:1 phải + Title + Body |
| Zalo OA | Chat bubble: brand header "Viettel Telecom" + image phía trên (nếu có) + nội dung |
| SMS | Plain SMS bubble: chỉ text, không ảnh, hiện counter X/160 |
| USSD | Monospace terminal: plain text, menu style |
| Banner | Card 16:9: image full width + Title + Body + CTA button |
| Email | Email client: subject bar + header image + body |

**Hành vi:**
- Khối **Thời gian gửi** nằm trên cùng Section 4, cấu hình 1 lần, áp dụng cho toàn bộ trigger và kênh trong campaign.
- **PARAMS chips** hiển thị ngay trong mỗi message card, liệt kê đúng payload của trigger đó; click chip → chèn `{{tham_so}}` vào vị trí con trỏ trong textarea đang active.
- AND mode: mỗi tab kênh chỉ có 1 card duy nhất; PARAMS hiển thị hợp nhất payload của tất cả trigger; button `[+ Biến thể đối tượng]` bị ẩn hoàn toàn.
- OR mode: mỗi tab kênh có N card tương ứng N trigger; mỗi card hiển thị đúng PARAMS của trigger đó; button `[+ Biến thể đối tượng]` hiển thị trong mỗi card.
- Completion badge trên tab: `●●` = 2/2 (xanh), `●○` = 1/2 (cam), `○○` = 0/2 (xám).
- `[+ Biến thể đối tượng]` → thêm card mới cho trigger đó với segment khác nhau (xem mock-up bên dưới).
- Mỗi card trigger luôn hiển thị **2 cột inline** (Soạn | XEM TRƯỚC) — không cần click Preview để mở panel.
- SMS: cho nhập vượt 160 ký tự; counter đỏ + badge "X SMS segment"; không block lưu.
- USSD: plain text 182 ký tự; cảnh báo nếu có ký tự đặc biệt.
- Tab `[+ Kênh]`: chỉ liệt kê kênh chưa có tab.

**Biến thể đối tượng — Audience Variant (khi có nhiều message khác nhau cho cùng 1 trigger):**

```
── Tab Push (T1 · SIM_ACTIVATED — đang có 2 biến thể) ───────────────────────────────

┌────────────────────────────────────────────────────────────────────────────────────┐
│ T1 · SIM_ACTIVATED                                                                 │
│ PARAMS:  {{ten_kh}}  {{loai_sim}}  {{ngay_kich_hoat}}  {{so_dien_thoai}}           │
├────────────────────────────────────────────────────────────────────────────────────┤
│ Biến thể 1 · Đối tượng: [Tất cả ▾]  (mặc định — fallback khi không match biến thể nào khác)
├─────────────────────────────────────────┬──────────────────────────────────────────┤
│ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                         │
├─────────────────────────────────────────┼──────────────────────────────────────────┤
│ Image (1:1, optional)                   │ ┌────────────────────────────┐           │
│ ┌──────────────────────────────────┐   │ │ 🔔 App icon   Title here   │           │
│ │  🖼  Kéo thả hoặc [Tải lên]     │   │ │    Body text rendered...   │           │
│ └──────────────────────────────────┘   │ └────────────────────────────┘           │
│                                         │                                          │
│ Template: [Welcome Push ▾]              │ SAMPLE PARAMS                            │
│ Title *  [Chào mừng bạn!__________]    │ ten_kh    [Nguyễn Văn A  ]               │
│ Body *   [SIM {{loai_sim}} đã kích...]  │ loai_sim  [eSIM          ]               │
│                                         │ (nhập sample → render realtime)          │
├─────────────────────────────────────────┴──────────────────────────────────────────┤
│ Biến thể 2 · Đối tượng: [VIP ▾]  (ưu tiên khi KH thuộc segment này)          [×] │
├─────────────────────────────────────────┬──────────────────────────────────────────┤
│ SOẠN NỘI DUNG                           │ XEM TRƯỚC (live)                         │
├─────────────────────────────────────────┼──────────────────────────────────────────┤
│ Image (1:1, optional)                   │ ┌────────────────────────────┐           │
│ ┌──────────────────────────────────┐   │ │ 🔔 App icon   Title here   │           │
│ │  🖼  Kéo thả hoặc [Tải lên]     │   │ │    Body text rendered...   │           │
│ └──────────────────────────────────┘   │ └────────────────────────────┘           │
│                                         │                                          │
│ Template: [Welcome VIP ▾]               │ SAMPLE PARAMS                            │
│ Title *  [Ưu đãi đặc biệt dành cho VIP]│ ten_kh    [Nguyễn Văn A  ]               │
│ Body *   [Cảm ơn bạn đã tin dùng...]   │ loai_sim  [eSIM          ]               │
│                                         │ (nhập sample → render realtime)          │
├─────────────────────────────────────────┴──────────────────────────────────────────┤
│ [+ Biến thể đối tượng]                                                             │
│ ⓘ KH match nhiều biến thể → áp dụng biến thể có thứ tự cao nhất (trái → phải);   │
│   Biến thể 1 luôn là fallback mặc định, không thể xóa.                            │
└────────────────────────────────────────────────────────────────────────────────────┘
```

**Hành vi Audience Variant:**
- Click `[+ Biến thể đối tượng]` → thêm card mới bên phải; dropdown "Đối tượng" mặc định trống, bắt buộc chọn trước khi lưu.
- Dropdown "Đối tượng" liệt kê các phân khúc đã chọn ở S3; chọn "Tất cả" chỉ khả dụng cho Biến thể 1 (fallback).
- Thứ tự ưu tiên: trái → phải; KH match segment của biến thể nào (thứ tự cao hơn) → nhận message của biến thể đó; không match bất kỳ biến thể có segment cụ thể nào → nhận Biến thể 1 (fallback).
- Biến thể 1 không có `[×]`; từ Biến thể 2 trở đi có `[×]` → xóa với confirm dialog.
- PARAMS dùng chung từ trigger, hiển thị ở header card T1; không lặp lại trong từng biến thể.
- Mỗi biến thể có nội dung message độc lập (template, title, body riêng).
- Completion badge tab kênh tính là hoàn chỉnh khi Biến thể 1 (fallback) đã có nội dung.

**Cấu trúc card soạn nội dung (luôn hiển thị 2 cột — không cần click Preview):**

```
┌──────────────────────────────────────────────┬──────────────────────────┐
│ CẤU HÌNH — T1 · SIM_ACTIVATED · Push         │ XEM TRƯỚC                │
├──────────────────────────────────────────────┼──────────────────────────┤
│                                              │ [Phone mockup]           │
│ Image (1:1, optional)                        │ ┌─────────────────┐      │
│ [Tải lên] [Chọn thư viện]                    │ │ App icon  Title │      │
│                                              │ │ Body text...    │      │
│ Template: [Welcome Push ▾]                   │ └─────────────────┘      │
│                                              │                          │
│ Title *                                      │ SAMPLE PARAMS            │
│ [Chào mừng bạn!____________________]         │ ten_kh  [Nguyễn Văn A]   │
│                                              │ loai_sim  [eSIM]         │
│ Body *                                       │ ngay_kich_hoat [01/04]   │
│ [SIM {{loai_sim}} đã kích hoạt thành...]     │ so_dien_thoai [0987xxx]  │
│                                              │                          │
│ PARAMS (T1 · SIM_ACTIVATED):                 │ [↻ Render lại]           │
│ [{{ten_kh}}] [{{loai_sim}}]                  │                          │
│ [{{ngay_kich_hoat}}] [{{so_dien_thoai}}]     │                          │
└──────────────────────────────────────────────┴──────────────────────────┘
```

---

### Section 5: Channel Strategy

> **Lưu ý thiết kế:** Logic pipeline kênh (thứ tự fallback, thời gian chờ, điều kiện dừng) được định nghĩa nội bộ, không hiển thị trên giao diện. Xem chi tiết tại [`deferred/channel-pipeline-logic.md`](deferred/channel-pipeline-logic.md).

```
┌────────────────────────────────────────────────────────────────────┐
│ 5. Channel Strategy                                      [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Đặt lịch gửi theo kênh (optional)  ▸ [Mở rộng]                   │
│                                                                    │
│  ── Khi mở rộng ─────────────────────────────────────────────────  │
│  Đặt lịch cụ thể cho từng kênh (độc lập với thời gian gửi message)│
│                                                                    │
│  Push:    ● Không đặt lịch riêng (theo message)                    │
│           ○ Gửi vào [__:__]  [Hằng ngày ▾]                        │
│                                                                    │
│  Zalo OA: ● Không đặt lịch riêng (theo message)                    │
│           ○ Gửi vào [__:__]  [Hằng ngày ▾]                        │
│                                                                    │
│  SMS:     ● Không đặt lịch riêng (theo message)                    │
│           ○ Gửi vào [__:__]  [Hằng ngày ▾]                        │
│  ──────────────────────────────────────────────────────────────── │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- "Đặt lịch theo kênh" là lịch override riêng per channel, độc lập với thời gian gửi đã cấu hình ở Section 4. Nếu bật, kênh đó chỉ gửi vào khung giờ được chọn dù trigger kích hoạt bất kỳ lúc nào.
- Danh sách kênh hiển thị đúng các kênh đã cấu hình ở Section 4 (Message Matrix); không hiện kênh chưa có nội dung.

---

### Section 6: An toàn

```
┌────────────────────────────────────────────────────────────────────┐
│ 6. An toàn                                               [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ GROUP A — BLACKOUT (Giờ giới nghiêm)                               │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ● Bật   ○ Tắt                                                  │ │
│ │                                                                │ │
│ │ Giờ giới nghiêm (Blackout)                                     │ │
│ │ ┌──────────────┐       ┌──────────────┐                        │ │
│ │ │  22:00   ⊙  │  —    │  08:00   ⊙  │                        │ │
│ │ └──────────────┘       └──────────────┘                        │ │
│ │                                                                │ │
│ │ ┌────────────────────────────────────────────────┐            │ │
│ │ │  Hủy luôn                                  ∨  │            │ │
│ │ └────────────────────────────────────────────────┘            │ │
│ │ (Hủy luôn / Delay đến đầu khung giờ)                          │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ GROUP B — SUPPRESSION (DNC / Blacklist / Whitelist)                │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ☑ Check DNC toàn hệ thống (luôn bật mặc định)                 │ │
│ │   Bỏ tick → confirm dialog cảnh báo rủi ro gửi KH đã từ chối  │ │
│ │                                                                │ │
│ │ Blacklist campaign                                             │ │
│ │ ○ Không dùng                                                  │ │
│ │ ● Dùng tệp riêng: [BL_ESIM_Q2_2026 · 320 KH ▾]                │ │
│ │                   [Xem] [Thay] [Upload mới]                    │ │
│ │   Sau blacklist campaign: loại 320 KH                          │ │
│ │                                                                │ │
│ │ Whitelist                                                      │ │
│ │ ● Không dùng                                                  │ │
│ │ ○ Chỉ gửi cho whitelist:  (chọn tệp ẩn đến khi bật)          │ │
│ │                                                                │ │
│ │ Reach cuối cùng: ~6,480 KH                                     │ │
│ │ = 6,800 → trừ DNC → trừ BL 320 → chưa dùng WL                │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ⓘ Giới hạn tần suất nhận tin (frequency cap) được cấu hình        │
│    tại System Settings → áp dụng cho toàn bộ campaign             │
│                                    [Đến System Settings →]        │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Dropdown Blackout xử lý: "Hủy luôn" (xóa message khỏi queue) / "Delay đến đầu khung giờ" (giữ trong queue, gửi lúc 08:00).
- Bật blacklist campaign nhưng chưa chọn tệp → blocking validation; nút `[Gửi duyệt]` disabled.
- Bật whitelist nhưng chưa chọn tệp → blocking validation.
- Công thức reach: `Audience → trừ DNC → trừ Campaign BL → giao WL (nếu bật)`.
- Modal chọn/xem tệp blacklist và whitelist giống nhau: tìm kiếm tệp + preview 50 dòng + số hợp lệ/trùng/sai định dạng + nút Áp dụng.

**Modal chọn tệp (Blacklist hoặc Whitelist):**

```
┌──────────────────────────────────────────────────────────────┐
│ Chọn tệp blacklist campaign                                  │
├──────────────────────────────────────────────────────────────┤
│ [Tìm tên tệp...]                                             │
│                                                              │
│ ○ BL_TEST_INTERNAL        40 KH     10/05/2026 15:00         │
│ ● BL_ESIM_Q2_2026        320 KH     12/05/2026 09:00         │
│ ○ BL_COMPLAINT_USERS     180 KH     11/05/2026 10:30         │
│                                                              │
│ Preview tệp đang chọn                                        │
│ Hợp lệ: 320  ·  Trùng: 8  ·  Sai định dạng: 2               │
│ 0987 xxx 090                                                 │
│ 0912 xxx 091  ...                                            │
│                                                              │
│                        [Hủy]  [Áp dụng]                     │
└──────────────────────────────────────────────────────────────┘
```

---

### User Flow — Tạo Campaign

```
1. Nhập thông tin campaign (tên, mã, mục tiêu, thời gian)
   ↓
2. Chọn Basic/Advanced trigger mode → thêm trigger → kéo-thả priority
   ↓
3. Chọn phân khúc từ dropdown → thêm điều kiện lọc nếu cần
   ↓
4. Cấu hình Message Matrix: tab kênh → soạn/chọn template → đặt thời gian gửi
   ↓
5. Thiết lập Channel Strategy: sắp xếp pipeline → điều kiện dừng → lịch kênh nếu cần
   ↓
6. Cấu hình An toàn: blackout / DNC / BL campaign / WL
   ↓
7. Lưu nháp hoặc Gửi duyệt (resolve hết issue blocking trước)
```

---

## Screen 4: Template Management

### Mục tiêu
Quản lý thư viện template tin nhắn tái sử dụng. QTV soạn sẵn nội dung theo kênh, có tham số động, để chọn nhanh khi tạo campaign.

### 4A: Danh sách Template

```
┌─────────────────────────────────────────────────────────────────────┐
│ TEMPLATE                                          [+ Tạo Template]  │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm tên template...]    Kênh: [Tất cả ▾]    Trạng thái: [Tất cả ▾]│
├──────────────────────┬───────────────┬────────┬─────────────────────┤
│ Tên Template         │ Kênh hỗ trợ   │ Dùng   │ Hành động           │
├──────────────────────┼───────────────┼────────┼─────────────────────┤
│ Chào mừng SIM        │ Push · Zalo   │ 3 lần  │ [Sửa] [Clone] [Tắt]│
│ Nhắc nạp thẻ         │ SMS · USSD    │ 5 lần  │ [Sửa] [Clone] [Tắt]│
│ Sắp hết data         │ Push · SMS    │ 2 lần  │ [Sửa] [Clone] [Tắt]│
│ Sinh nhật KH         │ Zalo · Email  │ 4 lần  │ [Sửa] [Clone] [Tắt]│
│ Cài app nhắc nhở     │ Push          │ 1 lần  │ [Sửa] [Clone] [Bật]│
└──────────────────────┴───────────────┴────────┴─────────────────────┘
                                                    < 1 2 >  [20/trang ▾]
```

**Hành vi:**
- Filter Kênh → lọc template có nội dung kênh đó.
- `[Clone]` → tạo bản sao với tên "Copy of [Tên]", mở ngay editor.
- `[Tắt/Bật]` → toggle trạng thái Active/Inactive; template Inactive không hiện trong dropdown khi tạo campaign.
- "Dùng" = số campaign đang dùng template này; click → popover danh sách campaign liên quan.

### 4B: Màn hình Tạo / Sửa Template

```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Template                                                          │
│ [Tên template *_______________________]    ● Active   [Lưu Template]│
│ [Mô tả (optional)___________________]                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ TIẾN ĐỘ NỘI DUNG:                                                   │
│ Push ●  Zalo ●  SMS ○  USSD ○  Banner ○  Email ○                    │
│ (● = có nội dung  ○ = chưa cấu hình)                                │
│                                                                     │
│ Tab kênh:  [Push ●]  [Zalo OA ●]  [SMS ○]  [USSD ○]                │
│            [Banner ○]  [Email ○]  [+ Kênh]                          │
│                                                                     │
│ ── Tab Push (đang active) ─────────────────────────────────────── │
│                                                                     │
│ ┌────────────────────────────────────────┬────────────────────────┐ │
│ │ SOẠN NỘI DUNG                          │ XEM TRƯỚC              │ │
│ ├────────────────────────────────────────┼────────────────────────┤ │
│ │ THAM SỐ ĐỘNG:                          │ [Phone mockup]         │ │
│ │ [{{ten_kh}}] [{{loai_sim}}]            │ ┌──────────────────┐   │ │
│ │ [{{so_dien_thoai}}] [{{so_du}}]        │ │ App icon  Title  │   │ │
│ │ [{{data_con_lai}}] [{{ngay_het_han}}]  │ │ Body text render │   │ │
│ │ → Click chip để chèn vào nội dung      │ └──────────────────┘   │ │
│ │                                        │                        │ │
│ │ Image (1:1, optional)                  │ SAMPLE PARAMS          │ │
│ │ [Tải lên] [Chọn thư viện]              │ ten_kh [Nguyễn Văn A]  │ │
│ │                                        │ loai_sim [eSIM]        │ │
│ │ Title *                                │ data_con_lai [120 MB]  │ │
│ │ [Chào mừng {{ten_kh}}!__________]      │                        │ │
│ │                                        │ [↻ Render lại]         │ │
│ │ Body *                                 │                        │ │
│ │ [SIM {{loai_sim}} của bạn đã được      │                        │ │
│ │  kích hoạt thành công._____________]   │                        │ │
│ └────────────────────────────────────────┴────────────────────────┘ │
│                                                                     │
│ ── Tab SMS ────────────────────────────────────────────────────── │
│                                                                     │
│ ┌────────────────────────────────────────┬────────────────────────┐ │
│ │ SOẠN NỘI DUNG                          │ XEM TRƯỚC              │ │
│ ├────────────────────────────────────────┼────────────────────────┤ │
│ │ THAM SỐ ĐỘNG:                          │ [SMS mockup]           │ │
│ │ [{{ten_kh}}] [{{so_du}}]               │ ┌──────────────────┐   │ │
│ │ [{{data_con_lai}}] [{{ngay_het_han}}]  │ │ Nội dung SMS     │   │ │
│ │ → Click chip để chèn vào nội dung      │ │ đã render        │   │ │
│ │                                        │ └──────────────────┘   │ │
│ │ Nội dung * (160 ký tự / SMS)           │                        │ │
│ │ [____________________________]  0/160  │ SAMPLE PARAMS          │ │
│ │                                        │ ten_kh [Nguyễn Văn A]  │ │
│ │ ⚠ Chỉ plain text, không hỗ trợ ảnh    │ so_du [50,000đ]        │ │
│ │                                        │                        │ │
│ │                                        │ [↻ Render lại]         │ │
│ └────────────────────────────────────────┴────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Tab kênh: mỗi tab có card soạn nội dung độc lập (cấu trúc Soạn | Preview giống Message Matrix); `[+ Kênh]` thêm tab mới.
- THAM SỐ ĐỘNG nằm đầu mỗi card; click chip → chèn `{{tham_so}}` vào vị trí con trỏ trong textarea đang active.
- Preview render realtime; SAMPLE PARAMS nhập tay để kiểm tra nội dung hiển thị đúng.
- Completion badge trên tab: `●` = có nội dung, `○` = chưa cấu hình; TIẾN ĐỘ NỘI DUNG tổng hợp tất cả tab.
- Lưu Template: validate tên bắt buộc; cảnh báo nếu tab kênh nào đang trống (không block lưu).
- Khi user chuyển từ OR sang AND mode: hiển thị confirm dialog "Chuyển sang AND mode sẽ xóa toàn bộ Biến thể đối tượng đã tạo. Tiếp tục?" — xác nhận thì xóa variant, button `[+ Biến thể đối tượng]` bị ẩn.

---

## Screen 5: Trigger Management

### Mục tiêu
Quản lý toàn bộ trigger master. Nhóm theo loại để dễ scan; Admin thêm/sửa/bật/tắt.

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ TRIGGER MANAGEMENT                              [+ Thêm Trigger]    │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm trigger code hoặc tên...]    Trạng thái: [Tất cả ▾]        │
└─────────────────────────────────────────────────────────────────────┘
```

### Bảng nhóm theo loại

```
▼ REALTIME  (2 trigger)
┌──────────────────────┬───────────────────────┬──────────┬───────────┬─────────────┐
│ Code                 │ Tên                   │ Source   │ Trạng thái│ Hành động   │
├──────────────────────┼───────────────────────┼──────────┼───────────┼─────────────┤
│ SIM_ACTIVATED        │ Kích hoạt SIM         │ BSS      │ ● Active  │ [Sửa][Tắt] │
│ LOC_TRAVEL_PROV      │ Đến tỉnh du lịch      │ OCS      │ ● Active  │ [Sửa][Tắt] │
└──────────────────────┴───────────────────────┴──────────┴───────────┴─────────────┘

▼ NEAR REALTIME  (1 trigger)
┌──────────────────────┬───────────────────────┬──────────┬───────────┬─────────────┐
│ NO_APP_INSTALL_24H   │ Chưa cài app sau 24h  │ CRM      │ ● Active  │ [Sửa][Tắt] │
└──────────────────────┴───────────────────────┴──────────┴───────────┴─────────────┘

▼ OFFLINE / BATCH  (3 trigger)
┌──────────────────────┬───────────────────────┬──────────┬───────────┬─────────────┐
│ LOW_DATA_BALANCE     │ Data còn dưới 100MB   │ OCS      │ ● Active  │ [Sửa][Tắt] │
│ USAGE_DROP_40PCT     │ Usage giảm >40%       │ OCS      │ ○ Testing │ [Sửa][Bật] │
│ SIM_NO_TXN_30D       │ SIM ngừng GD >30 ngày │ BSS      │ ○ Paused  │ [Sửa][Bật] │
└──────────────────────┴───────────────────────┴──────────┴───────────┴─────────────┘
```

**Hành vi:**
- Click header group `▼ REALTIME` → collapse/expand toàn bộ trigger trong nhóm.
- Filter "Trạng thái" áp dụng cross tất cả group.
- Tìm kiếm realtime, highlight kết quả khớp; expand group chứa kết quả tự động.
- `[Tắt/Bật]`: confirm dialog. Nếu trigger đang dùng trong campaign Active → cảnh báo thêm danh sách campaign bị ảnh hưởng.

### Modal Thêm / Sửa Trigger

```
┌────────────────────────────────────────────────────────────┐
│ Thêm Trigger Mới                                    [✕]    │
├────────────────────────────────────────────────────────────┤
│ Trigger code *              Loại *                         │
│ [SIM_ACTIVATED___________]  [Realtime ▾]                   │
│                                                            │
│ Tên trigger *                                              │
│ [________________________________________]                 │
│                                                            │
│ Event source *              Trạng thái                     │
│ [BSS / OCS / CRM / Other ▾] ● Active  ○ Inactive           │
│                                                            │
│ Mô tả                                                      │
│ [________________________________________]                 │
│                                                            │
│ Payload                              [+ Thêm tham số]      │
│ ┌──────────────────────────────────────────────────────┐  │
│ │ [ten_kh________]  [BSS ▾]  [✕]                       │  │
│ │ [loai_sim______]  [BSS ▾]  [✕]                       │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                            │
│                              [Hủy]        [Lưu]           │
└────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Trigger code: chỉ in hoa, số, gạch dưới; không trùng với code đã có.
- Loại: Realtime / Near Realtime / Offline.
- Priority xung đột không cấu hình ở đây — chỉ cấu hình ở campaign level.
- `[Lưu]` → đóng modal + toast "Đã lưu trigger ✓" + cập nhật bảng theo group đúng loại.

---

## Screen 6: Blacklist Management

### Mục tiêu
Quản lý danh sách số điện thoại bị chặn gửi tin theo campaign và kênh cụ thể (CVM Blacklist — riêng biệt với BSS DNC).

### 6A: Danh sách Blacklist

```
┌─────────────────────────────────────────────────────────────────────┐
│ BLACKLIST                                                           │
├─────────────────────────────────────────────────────────────────────┤
│ Campaign: [Tất cả ▾]   Kênh: [Tất cả ▾]              [Lọc]         │
├──────────────────┬──────────────────────┬──────────┬────────────────┤
│ Số điện thoại    │ Campaign             │ Kênh     │ Hành động      │
├──────────────────┼──────────────────────┼──────────┼────────────────┤
│ 0987 xxx 001     │ Chào mừng SIM        │ Zalo OA  │ [Xóa]          │
│ 0912 xxx 002     │ Hết data             │ SMS      │ [Xóa]          │
│ 0965 xxx 003     │ Hết data             │ Zalo OA  │ [Xóa]          │
│ 0976 xxx 004     │ Nhắc nạp tiền        │ Push     │ [Xóa]          │
└──────────────────┴──────────────────────┴──────────┴────────────────┘
│                              < 1 2 3 >                              │
│                                                                     │
│ [+ Thêm thủ công]                    [📥 Upload danh sách]          │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Filter Campaign + Kênh → lọc realtime.
- `[Xóa]` → confirm dialog "Xóa số này khỏi blacklist của campaign [X] kênh [Y]?" → xóa ngay.
- `[+ Thêm thủ công]` → mở Modal Thêm.
- `[📥 Upload danh sách]` → mở Modal Upload.

### 6B: Modal Thêm thủ công

```
┌──────────────────────────────────────┐
│ Thêm vào Blacklist                [✕]│
├──────────────────────────────────────┤
│ Số điện thoại *                      │
│ [______________________________]     │
│                                      │
│ Campaign *                           │
│ [Chọn campaign... ▾]                 │
│                                      │
│ Kênh *                               │
│ [Chọn kênh... ▾]                     │
│ Zalo OA / SMS / USSD / Push /        │
│ Banner / Email                       │
│                                      │
│            [Hủy]       [Thêm]        │
└──────────────────────────────────────┘
```

### 6C: Modal Upload danh sách

```
┌──────────────────────────────────────┐
│ Upload Blacklist                  [✕]│
├──────────────────────────────────────┤
│ Campaign *                           │
│ [Chọn campaign... ▾]                 │
│                                      │
│ Kênh *                               │
│ [Chọn kênh... ▾]                     │
│                                      │
│ File CSV *                           │
│ ┌──────────────────────────────┐     │
│ │  📄 Kéo thả file vào đây    │     │
│ │  hoặc [Chọn file]           │     │
│ │  Format: 1 cột so_dien_thoai│     │
│ │  [📥 Tải file mẫu]          │     │
│ └──────────────────────────────┘     │
│                                      │
│ Kết quả preview (sau khi chọn file): │
│ ✅ 245 số hợp lệ                     │
│ ⚠ 3 số sai định dạng (bỏ qua)       │
│                                      │
│      [Hủy]    [Xác nhận Upload]      │
└──────────────────────────────────────┘
```

**Hành vi:**
- Chọn file → parse ngay, hiển thị preview kết quả.
- Số sai định dạng: bỏ qua, không block upload.
- `[Xác nhận Upload]` → thêm vào blacklist + toast "Đã upload 245 số vào blacklist".

---

## Screen 7: Customer List

### Mục tiêu
Tra cứu nhanh danh sách khách hàng, tìm kiếm theo số điện thoại, vào Customer 360 để xem chi tiết.

### Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│ KHÁCH HÀNG                                                          │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm số điện thoại...]                                           │
│                                                                     │
│ Trạng thái SIM: [Tất cả ▾]   Cài app: [Tất cả ▾]   DNC: [Tất cả ▾]│
├──────────────────┬────────────┬────────────┬──────────┬─────────────┤
│ Số điện thoại    │ Loại SIM   │ Trạng thái │ Cài app  │ Hành động   │
├──────────────────┼────────────┼────────────┼──────────┼─────────────┤
│ 0987 xxx 001     │ eSIM       │ ● Active   │ Có       │ [Xem 360]   │
│ 0912 xxx 002     │ SIM vật lý │ ● Active   │ Không    │ [Xem 360]   │
│ 0965 xxx 003     │ eSIM       │ ○ Inactive │ Có       │ [Xem 360]   │
│ 0976 xxx 004     │ SIM vật lý │ ● Active   │ Không    │ [Xem 360]   │
└──────────────────┴────────────┴────────────┴──────────┴─────────────┘
                                                    < 1 2 ... >  [20/trang ▾]
```

**Hành vi:**
- Tìm kiếm theo số điện thoại (chính xác hoặc một phần).
- Filter Trạng thái SIM: Active / Inactive / Suspended.
- Filter Cài app: Có / Không.
- Filter DNC: Có DNC / Không có DNC.
- `[Xem 360]` → điều hướng sang Customer 360 của số đó.
- Dữ liệu từ BSS/OCS, không chỉnh sửa trực tiếp trong CVM.

---

## Screen 8: Customer 360

### Mục tiêu
Xem toàn bộ thông tin 1 khách hàng: profile, trạng thái kênh (sync-back từ Gateway), lịch sử nhận tin, throttling status.

### Layout 2 cột

```
┌─────────────────────────────────────────────────────────────────────┐
│ ← Danh sách khách hàng                                              │
│ CUSTOMER 360 — 0987 xxx 001                                         │
├────────────────────────────────┬────────────────────────────────────┤
│ THÔNG TIN & SỬ DỤNG            │ TRẠNG THÁI KÊNH & LỊCH SỬ         │
├────────────────────────────────┼────────────────────────────────────┤
│ Loại SIM: eSIM  ·  Active      │ TRẠNG THÁI KÊNH (Sync-back)        │
│ Gói: D50  ·  hết 18/05/2026   │ ┌──────┬────────────┬───────────┐  │
│ Ngày KH: 01/04/2026           │ │ Kênh │ Trạng thái │ Cập nhật  │  │
│ Cài app: Có  ·  Đăng nhập: Có │ ├──────┼────────────┼───────────┤  │
│                                │ │Zalo  │ ⛔ Blocked │ 05/05 09:32│ │
│ SỬ DỤNG                        │ │SMS   │ ✅ Active  │ —         │  │
│ Data còn: 1.2 GB               │ │USSD  │ ✅ Active  │ —         │  │
│ Hôm nay: 320 MB                │ │Push  │ ✅ Active  │ —         │  │
│ Số dư: 45,000 VND              │ └──────┴────────────┴───────────┘  │
│ Nạp tiền: 3 lần                │ ⓘ Tự cập nhật từ phản hồi Gateway  │
│ Sinh nhật: 15/08               │                                    │
│                                │ THROTTLING                         │
│                                │ Tin hôm nay: 2/3                   │
│                                │ Cooldown: Không  ·  DNC: Không     │
│                                │                                    │
│                                │ LỊCH SỬ NHẬN TIN                   │
│                                │ 10/05  Chào mừng SIM               │
│                                │        Zalo  ·  ✓ Delivered        │
│                                │                                    │
│                                │ 08/05  Nạp thẻ lỗi                 │
│                                │        SMS  ·  ✓ Delivered         │
│                                │                                    │
│                                │ 07/05  Hết data                    │
│                                │        Zalo  ·  ✕ Blocked          │
│                                │        → SMS  ·  ✓ Delivered       │
│                                │                                    │
│                                │ [Xem đầy đủ lịch sử →]             │
└────────────────────────────────┴────────────────────────────────────┘
```

**Hành vi:**
- Trạng thái kênh tự cập nhật từ Gateway; không chỉnh sửa thủ công.
- Dòng "Blocked → Fallback" hiển thị 2 sub-dòng: kênh bị block và kênh fallback thành công.
- `[Xem đầy đủ lịch sử →]` mở drawer/trang với phân trang và filter theo kênh và campaign.

---

## Screen 9: Report / Analytics

### Mục tiêu
Phân tích hiệu quả campaign theo thời gian — dành cho QTV và cấp quản lý xem insight, so sánh campaign, phân tích segment và phát hiện rủi ro spam/fatigue.

### Layout tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ANALYTICS & REPORT                                       [Xuất Excel]   │
├─────────────────────────────────────────────────────────────────────────┤
│ FILTER BAR                                                              │
│ Thời gian: [7 ngày ▾]  Campaign: [Tất cả ▾]  Segment: [Tất cả ▾]       │
│            [Tháng này]  [So sánh: ON ▾]       Kênh: [Tất cả ▾]         │
├─────────────────────────────────────────────────────────────────────────┤
│ TAB NAVIGATION                                                          │
│ [Delivery] [Engagement] [Campaign Comparison] [Segment] [Funnel] [Spam]│
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 1: Delivery Analytics

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SUMMARY KPIs (theo filter đang chọn)                                    │
│ ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│ │ Total Sent │  │ Delivered  │  │ Failed     │  │ Delivery   │          │
│ │  124,500   │  │  119,940   │  │  4,560     │  │ Rate       │          │
│ │            │  │  96.3%     │  │  3.7% ⚠   │  │  96.3%     │          │
│ └────────────┘  └────────────┘  └────────────┘  └────────────┘          │
├─────────────────────────────────────────────────────────────────────────┤
│ SENT vs DELIVERED — Area chart theo thời gian                           │
│                                                                         │
│  50K ┤                                    ╭──────╮                      │
│  40K ┤               ╭──────────╮        ╭╯      ╰─────╮                │
│  30K ┤          ╭────╯  Sent    ╰────────╯  Delivered  ╰─────           │
│  20K ┤    ╭─────╯                                                       │
│  10K ┤────╯                                                             │
│      └──────────────────────────────────────────────────────── ngày     │
│        07  08  09  10  11  12  13                                       │
│  ● Sent (line trên)   ● Delivered (area xanh)   ● Failed (area đỏ)      │
├─────────────────────────────────────────────────────────────────────────┤
│ DELIVERY RATE TREND          │ FAILED DELIVERY TREND                    │
│ Line chart — % theo ngày     │ Stacked bar — phân loại lý do thất bại  │
│                              │                                          │
│  98% ┤      ╭───╮            │ 2K ┤  ■■ ■■■ ■■ ■■■ ■■■ ■■ ■■■          │
│  96% ┤──────╯   ╰────────── │ 1K ┤  ░░ ░░░ ░░ ░░░ ░░░ ░░ ░░░          │
│  94% ┤                      │  0 └──────────────────────────── ngày    │
│      └──────────── ngày      │ ■ Gateway error  ░ User blocked          │
│ ─── target 95%               │ ▒ DNC/Blacklist  ▓ Network timeout       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 2: Engagement Analytics

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ENGAGEMENT KPIs                                                         │
│ ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│ │ Open Rate  │  │ CTR        │  │ Conversion │  │ Best       │          │
│ │  38.4%     │  │  12.1%     │  │ Rate       │  │ Channel    │          │
│ │ ↑2.1% vs   │  │ ↑0.8%      │  │  8.3%      │  │ Push 97.7% │          │
│ │ tuần trước │  │            │  │ ↑0.4%      │  │            │          │
│ └────────────┘  └────────────┘  └────────────┘  └────────────┘          │
├─────────────────────────────────────────────────────────────────────────┤
│ CHANNEL PERFORMANCE COMPARISON — Bar chart                              │
│                                                                         │
│ Push    ████████████████████████████████████  Open 52%  CTR 18%        │
│ Zalo OA ████████████████████████████████      Open 44%  CTR 15%        │
│ SMS     ████████████████████                  Open 22%  CTR  8%        │
│ USSD    █████████████                         Open 15%  CTR  5%        │
│ Banner  ████████                              Open  9%  CTR  3%        │
│ Email   ██████████████████                    Open 21%  CTR  9%        │
│                          ■ Open Rate  ░ CTR  ▒ Conversion               │
├─────────────────────────────────────────────────────────────────────────┤
│ ENGAGEMENT TREND — Line chart theo ngày                                 │
│ ● Open Rate  ● CTR  ● Conversion Rate  (7 ngày, có thể chọn kênh)      │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 3: Campaign Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SO SÁNH CAMPAIGN — chọn tối đa 5 campaign để so sánh                   │
│ [+ Thêm campaign để so sánh ▾]                                          │
├─────────────────────────────────────────────────────────────────────────┤
│ BAR CHART — Delivery Rate / Conversion Rate theo campaign               │
│                                                                         │
│ Nhắc nạp tiền    ████████████████████████████████  96.5% / 9.1%         │
│ Hết hạn data     ███████████████████████████████   95.4% / 7.8%         │
│ Welcome eSIM     █████████████████████████████     93.1% / 8.3%         │
│ LOW_DATA batch   ████████████████████████████████  94.2% / 6.2%         │
│                                    ■ Delivery  ░ Conversion             │
├─────────────────────────────────────────────────────────────────────────┤
│ BẢNG SO SÁNH CHI TIẾT                                                   │
│ ┌─────────────────┬────────┬──────────┬──────┬──────────┬──────────┐    │
│ │ Campaign        │ Sent   │ Delivered│ Open │ Convert  │ Cost SMS │    │
│ ├─────────────────┼────────┼──────────┼──────┼──────────┼──────────┤    │
│ │ Nhắc nạp tiền   │ 32,100 │ 30,980   │38.4% │  9.1%    │ 480K VND │    │
│ │ Hết hạn data    │ 28,400 │ 27,100   │35.2% │  7.8%    │ 360K VND │    │
│ │ Welcome eSIM    │ 20,600 │ 19,400   │42.1% │  8.3%    │ 240K VND │    │
│ │ LOW_DATA batch  │ 24,800 │ 23,900   │29.6% │  6.2%    │ 720K VND │    │
│ └─────────────────┴────────┴──────────┴──────┴──────────┴──────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 4: Segment Analytics

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SEGMENT PERFORMANCE — Delivery & Conversion theo phân khúc              │
│                                                                         │
│ ┌──────────────────┬────────┬──────────┬──────┬──────────┐              │
│ │ Segment          │ Reach  │ Delivered│ Open │ Conversion│              │
│ ├──────────────────┼────────┼──────────┼──────┼──────────┤              │
│ │ Gen Z (18–25)    │ 18,450 │ 17,800   │52.3% │  11.2%   │              │
│ │ Sắp hết data     │  8,920 │  8,600   │44.8% │   8.9%   │              │
│ │ ARPU cao         │  5,600 │  5,480   │38.1% │  14.6%   │              │
│ │ Nguy cơ rời mạng │  2,150 │  2,050   │22.4% │   3.1%   │              │
│ └──────────────────┴────────┴──────────┴──────┴──────────┘              │
├─────────────────────────────────────────────────────────────────────────┤
│ SO SÁNH SEGMENT — Bar chart                                             │
│                                                                         │
│ Gen Z        ████████████████████████████  52.3% open  11.2% convert   │
│ ARPU cao     ██████████████████████        38.1% open  14.6% convert   │
│ Sắp hết data ██████████████████████        44.8% open   8.9% convert   │
│ Nguy cơ rời  ██████████                    22.4% open   3.1% convert   │
│                                    ■ Open Rate  ░ Conversion Rate       │
├─────────────────────────────────────────────────────────────────────────┤
│ DEVICE BREAKDOWN                                                        │
│ Android: 62%  ████████████████████████████████                          │
│ iOS:     28%  ██████████████                                            │
│ Khác:    10%  █████                                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 5: Funnel Analytics

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CONVERSION FUNNEL  ·  [Chọn campaign ▾]  ·  [Chọn kênh ▾]              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│ Audience đủ điều kiện    ████████████████████████████  32,100  100%     │
│                          ↓ Drop-off  7.8%  (−2,510 DNC/BL)             │
│ Messages Sent            █████████████████████████     29,590   92.2%   │
│                          ↓ Drop-off  7.7%  (−2,280 undelivered)        │
│ Delivered                ███████████████████████       27,310   85.1%   │
│                          ↓ Drop-off 47.3%  (−12,930 not opened)        │
│ Opened / Clicked         ████████████                  14,380   44.8%   │
│                          ↓ Drop-off 35.6%  (−5,120 no action)          │
│ App Installed            ████████                       9,260   28.8%   │
│                          ↓ Drop-off 37.3%  (−3,450 no purchase)        │
│ Package Purchased        █████                          5,810   18.1%   │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│ DROP-OFF ANALYSIS                                                       │
│ Điểm drop lớn nhất: Delivered → Opened (−47.3%)                        │
│ → Gợi ý: Thử A/B test tiêu đề Push để tăng open rate                  │
│ → So sánh kênh: Push open 52% vs SMS open 22% vs Zalo 44%             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 6: Spam & Fatigue Report

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SPAM & FATIGUE INDICATORS                                               │
├─────────────────────────────────────────────────────────────────────────┤
│ OPT-OUT & BLACKLIST TREND — Line chart                                  │
│                                                                         │
│  500 ┤                              ╭───╮                               │
│  400 ┤              ╭──╮            │   │ ← Spike cần điều tra          │
│  300 ┤    ╭─────────╯  ╰────────────╯   ╰──                             │
│  200 ┤────╯  Opt-out                                                    │
│  100 ┤───────────────────────────────── Blacklist mới                   │
│      └──────────────────────────────────────────────── ngày             │
│        07  08  09  10  11  12  13                                       │
├─────────────────────────────────────────────────────────────────────────┤
│ FREQUENCY METRICS              │ SPAM RISK INDICATOR                    │
│                                │                                        │
│ Avg tin/user/ngày:   1.8       │ ● LOW RISK  (score 24/100)             │
│ Avg tin/user/tuần:   4.2       │ ████░░░░░░░░░░░░░░░░░░░░               │
│ Max tin/user/ngày:   5         │                                        │
│ User nhận > 3 tin:  8.4%       │ Cảnh báo khi score > 60               │
│                                │ Yếu tố: opt-out rate, BL trend,        │
│ ┌──────────────────────────┐   │ avg msg/user, failed delivery          │
│ │ Histogram: phân phối số  │   │                                        │
│ │ tin/user trong kỳ        │   │ [Xem chi tiết chỉ số →]               │
│ │  ████░░░░   1 tin: 42%   │   │                                        │
│ │  ████░░░    2 tin: 31%   │   │                                        │
│ │  ███░░      3 tin: 18%   │   │                                        │
│ │  █          4+ tin:  9%  │   │                                        │
│ └──────────────────────────┘   │                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Hành vi Report
- Filter bar áp dụng đồng thời cho tất cả tab và chart.
- Filter thời gian: Hôm nay / 7 ngày / 30 ngày / Tháng này / Tùy chỉnh (date range picker).
- `[So sánh: ON]` → hiện thêm đường/cột kỳ trước trên tất cả chart để so sánh.
- `[Xuất Excel]` → xuất toàn bộ data theo filter và tab hiện tại ra file .xlsx.
- Click vào segment/campaign trong bảng → drill-down xem campaign/segment đó trên tất cả tab.
- Drop-off analysis: tự động highlight điểm drop lớn nhất và gợi ý hành động.

---

## Edge Cases

| Tình huống | Hành vi |
|---|---|
| Chỉ 1 trigger | Tự động là P1; ẩn Logic OR/AND và khối xung đột |
| Đổi trigger P1 | Payload chips cập nhật; tham số không còn hợp lệ highlight đỏ trong message |
| Xóa trigger P1 | Trigger còn lại tự thành P1 |
| Soạn message xong rồi đổi trigger | Giữ nội dung, chỉ highlight chip không hợp lệ |
| Thêm kênh đã có tab | Không cho thêm; kênh đó disabled trong dropdown |
| SMS vượt 160 ký tự | Counter đỏ + badge "X segment"; không block lưu; cost tính `ceil(ký tự / 160)` |
| SMS 300/160 ký tự | Hiển thị "2 SMS segment"; cost = reach SMS × 2 |
| USSD chưa cấu hình | Badge `○` trên tab; header badge đỏ tăng issue count |
| Gửi duyệt còn issue | Nút disabled + tooltip liệt kê issue; click issue → scroll đến đúng section |
| Tắt trigger đang dùng trong Active campaign | Confirm dialog + danh sách campaign bị ảnh hưởng |
| Trigger priority trùng nhau | Cảnh báo inline; yêu cầu kéo-thả sắp xếp lại trước khi gửi duyệt |
| KH thỏa nhiều trigger cùng lúc (OR mode) | Chỉ gửi message của trigger P1 |
| Chọn 0 phân khúc | Mặc định gửi T-ALL (tất cả KH); hiển thị confirm trước khi lưu |
| Bật blacklist nhưng chưa chọn tệp | Blocking validation; nút Gửi duyệt disabled |
| Bật whitelist nhưng chưa chọn tệp | Blocking validation |
| KH thuộc audience nhưng ngoài whitelist | Loại khỏi reach; không gửi |
| Whitelist có số sai định dạng | Bỏ qua; hiển thị số lượng bị bỏ qua trong preview modal |
| Xóa kênh ở Message Matrix | Confirm → gỡ kênh khỏi Channel Strategy pipeline |
| Không có kênh Push trong Message Matrix | Khối "Giới hạn Push" ở An toàn disabled với note |
| Kill Switch (Dừng campaign đang Active) | Confirm dialog cảnh báo message đang queue sẽ bị hủy |
| Thời gian gửi "Vào lúc HH:MM ngày T+0" mà đã qua giờ | Hệ thống xử lý như blackout: queue sang ngày T+1 |

---

## User Flow Tổng — CVM System

```
Dashboard
   ↓
Campaign List  →  [+ Tạo Campaign]
                       ↓
                 Campaign Builder
                 S1. Thông tin
                 S2. Trigger & Logic
                 S3. Audience
                 S4. Message Matrix (+ Thời gian gửi)
                 S5. Channel Strategy (+ Lịch kênh)
                 S6. An toàn
                       ↓
                 Gửi duyệt → Pending
                       ↓
                 Admin duyệt → Active
                       ↓
                 Trigger kích hoạt → Gửi tin
                       ↓
Customer 360  ← Sync-back trạng thái kênh
Report        ← Thống kê Delivered / Failed / Cost
```
