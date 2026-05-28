# Prompt vẽ Wireframe CVM System v9

> Dùng cho Lovable, Gemini, hoặc bất kỳ AI generation tool nào.
> Copy toàn bộ phần **SYSTEM PROMPT** + chọn 1 trong các **SCREEN PROMPT** bên dưới để vẽ từng màn hình.

---

## SYSTEM PROMPT — Dán vào đầu mỗi lần chat

```
You are a UI wireframe designer. I will describe a screen from a B2B SaaS marketing automation system called CVM (Customer Value Management) — used by telecom marketing operators to create and manage campaign messaging.

Design style:
- B2B SaaS, inspired by Braze / Klaviyo / HubSpot
- Clean, compact, information-dense
- Color palette: white background, #F8FAFC sidebar, #1E293B text, #22C55E (Active), #EAB308 (Pending), #F97316 (Paused), #94A3B8 (Draft), #EF4444 (error/warning)
- Font: Inter or similar sans-serif
- Left sidebar navigation (fixed): Dashboard / Campaign / Template / Trigger / Customer / Blacklist / Report / Settings
- Top header bar: logo left, breadcrumb center, notification bell + avatar right

Render as a high-fidelity wireframe (grayscale or light color), with clear component labels, realistic placeholder text, and interaction states where applicable. Show the full browser window including the left nav sidebar.

Language: Vietnamese UI labels (as specified per screen).
```

---

## SCREEN 1 — Dashboard

```
Draw Screen 1: Dashboard

Layout: Left nav sidebar (fixed) + main content area (scrollable).

Main content has 5 rows:

ROW 1 — 8 KPI Cards in a 4+4 grid:
Card 1: "Active Campaigns" | value: 12 | trend: ↑2 vs hôm qua | sparkline 7 days
Card 2: "Trigger Fired Today" | value: 3,842 | trend: ↑18% | sparkline
Card 3: "Messages Sent Today" | value: 48,320 | sparkline
Card 4: "Delivery Rate" | value: 96.4% | sparkline | badge "● SLA: OK" green
Card 5: "Failed Messages" | value: 1,760 | trend: ⚠ ↑3% | RED border warning | sparkline trending up
Card 6: "Conversion Rate" | value: 8.3% | trend: ↑0.4% vs tuần
Card 7: "Realtime SLA" | value: < 2.1s avg | badge "✅ Target: <3s"
Card 8: "Blacklist Blocked Today" | value: 4,210 | sub-line: "BSS DNC: 3,840 | CVM BL: 370"

ROW 2 — System Health (2 line charts side by side + 2 info boxes below):
- Left chart: "TRIGGER / EVENT VOLUME" — area line chart, 24h, 3 series (Realtime / Near RT / Offline)
- Right chart: "PROCESSING LATENCY" — line chart, p50/p95/p99 lines, red dashed SLA target at 3s
- Bottom left box: "QUEUE & BACKLOG" — shows processing queue 342 events with progress bar 68%, pending blackout 1,240, scheduled 8,900. Button [Xem queue]
- Bottom right box: "TOP SYSTEM ERRORS" — 3 error rows with red/orange icons, error code, count, last seen time. Button [Xem tất cả →]

ROW 3 — Campaign Monitoring (2 tables side by side + timeline below):
- Left table: "TOP ACTIVE CAMPAIGNS" — columns: Campaign name | Sent | Rate | Trend sparkline. 4 rows.
- Right table: "TRIGGER FIRED NHIỀU NHẤT HÔM NAY" — columns: Trigger | Fired | Match. 4 rows.
- Below both: "RECENT TRIGGER EVENTS TIMELINE" — list of realtime events, each row: timestamp | trigger code | masked phone number | result (Matched/Blocked/No match). Toggle button [Live ● pause]

ROW 4 — User Journey Funnel (full width):
Title: "USER JOURNEY FUNNEL · Welcome eSIM Q2/2026 · [Chọn campaign ▾]"
Horizontal funnel bars (decreasing width left to right):
Step 1: SIM Activated — 32,100 — 100%
Step 2: Message Delivered — 29,530 — 92% (drop 8%)
Step 3: Message Opened — 12,200 — 38% (drop 61%)
Step 4: App Installed — 9,400 — 29% (drop 23%)
Step 5: Package Purchased — 5,800 — 18% (drop 20%)
Footer: "Conversion rate 18.1% · vs tuần trước: ↑1.2%" | Button [Xem chi tiết →]

ROW 5 — Trigger Analytics (2 panels side by side + heatmap below):
- Left panel: "TOP TRIGGERS (7 ngày)" — table with Trigger name, Fired count, Match rate % with bar
- Right panel: "TRIGGER ANOMALY DETECTION" — alert card showing LOW_DATA_BALANCE spike +340%, details
- Below: "TRIGGER HEATMAP" — hour × weekday grid (24 cols × 7 rows), color intensity: light=low, dark=high, one cell marked as spike
```

---

## SCREEN 2 — Campaign List

```
Draw Screen 2: Campaign List

Full browser with left nav sidebar. "Campaign" item highlighted in nav.

Header: Page title "Campaign" (left) + button [+ Tạo Campaign] (right, primary blue)

Search bar: full width, placeholder "🔍 Tìm tên campaign, mã hoặc trigger code..."

Filter tag row (multi-select chips):
[Tất cả ×] [Active] [Draft] [Pending] [Paused] [Realtime] [Offline] [Có SMS] [Chưa đủ nội dung]

Data table with columns: TÊN / MÃ CAMPAIGN | TRIGGER | HIỆU LỰC | TRẠNG THÁI | HÀNH ĐỘNG

Row 1: "Welcome eSIM Q2/2026" / "CVM-WELCOME-ESIM-Q2-2026" | chips [SIM_ACT] [NO_APP] +1ⓘ | 01/04 – 30/06/2026 | ● Draft (gray chip) | [Xem] [Sửa]
Row 2: "Nhắc nạp tiền" / "CVM-REMIND-TOPUP-05-2026" | [TOP_UP] | 01/05 – 31/05/2026 | ● Active (green chip) | [Xem] [Dừng]
Row 3: "Hết hạn gói data" / "CVM-DATA-EXPIRE-05-2026" | [PKG_EXP] | 01/05 – 31/05/2026 | ● Paused (orange chip) | [Xem] [Bật]
Row 4: "Chào mừng du lịch" / "CVM-TRAVEL-PROMO-Q2" | [LOC_TRV] | 15/05 – 30/06/2026 | ⏳ Pending (yellow chip, sub-text "Chờ duyệt") | [Xem]

Pagination: < 1 2 3 > | [20/trang ▾]
```

---

## SCREEN 2B — Campaign Detail View

```
Draw Screen 2B: Campaign Detail View (read-only)

Full browser with left nav sidebar.

Page header (top bar):
- Breadcrumb: ← Campaign
- Campaign name: "Welcome eSIM Q2/2026 · CVM-WELCOME-ESIM-Q2-2026"
- Status chip: ⏳ Pending (yellow)
- Button [Đóng] top right

Content area (single scrollable column, clean section dividers):

Section 1 — "Thông tin Campaign":
Label-value pairs in 2-column grid:
Tên campaign: Welcome eSIM Q2/2026
Mã kịch bản: CVM-WELCOME-ESIM-Q2-2026
Mục tiêu: Onboard eSIM, nhắc cài app
Thời gian: 01/04/2026 – 30/06/2026
Người tạo: QTV Marketing
Ngày tạo: 10/05/2026 14:32
Ngày gửi duyệt: 12/05/2026 09:15

Section 2 — "Trigger & Logic":
Chế độ: Advanced · Logic: OR
Triggers listed as chips: ★ P1 SIM_ACTIVATED · Kích hoạt SIM thành công | P2 NO_APP_INSTALL_24H · Chưa cài app sau 24h
Ưu tiên: Chỉ gửi trigger P1
Ước tính: ~6,800 KH × 2 kênh = ~13,600 tin

Section 3 — "Audience":
Phân khúc: tag cards [Gen Z User (18–25)] [Sắp hết data]
Logic: OR
Điều kiện lọc: Loại thiết bị = Android
Reach: ~6,800 KH

Section 4 — "Message Matrix":
Thời gian gửi: Gửi ngay sau khi trigger kích hoạt
Tab bar: [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○]
Active tab "Push" shows 2 message preview cards (read-only):
Card 1: T1 · SIM_ACTIVATED | Title: Chào mừng bạn! | Body: SIM eSIM đã kích hoạt thành công. Khám phá MyViettel.
Card 2: T2 · NO_APP_INSTALL_24H | Title: Cài MyViettel nhận ưu đãi | Body: Bạn vẫn chưa cài app MyViettel. Cài ngay để...

Section 5 — "Channel Strategy":
Đặt lịch theo kênh: Không (tất cả theo thời gian gửi message)

Section 6 — "An toàn":
Blackout: Bật · 22:00 – 08:00 · Hủy luôn
DNC toàn hệ thống: Bật
Blacklist campaign: BL_ESIM_Q2_2026 · 320 KH
Whitelist: Không dùng
Reach cuối cùng: ~6,480 KH (highlighted in green)

All fields are read-only — no input boxes, no edit icons.
```

---

## SCREEN 3 — Campaign Builder

```
Draw Screen 3: Campaign Builder

Full browser with left nav. Two-column layout:

FIXED HEADER (top bar, full width):
- Breadcrumb: ← Campaign
- Title: "Welcome eSIM Q2/2026 · CVM-WELCOME-ESIM-Q2-2026 · ● Draft"
- Buttons right: [Lưu Nháp] (secondary) | [Gửi duyệt → ⚠ 2 issue] (primary, red badge)

LEFT COLUMN (~60%, scrollable):

Section 1 — "1. Thông tin Campaign" [Thu gọn]:
- Input: Tên campaign * → "Welcome eSIM Q2/2026"
- Input: Mã kịch bản → "CVM-WELCOME-ESIM-Q2-2026"
- Input row: Mục tiêu | Date range picker: 01/04/2026 – 30/06/2026
- Label: Người tạo: QTV Marketing (read-only)

Section 2 — "2. Trigger & Logic" [Thu gọn]:
- Radio: ○ Basic ● Advanced
- Trigger chips (vertical list): ★ P1 [SIM_ACTIVATED · Kích hoạt SIM thành công] [≡][✕] | P2 [NO_APP_INSTALL_24H · Chưa cài app sau 24h] [≡][✕]
- Button: [+ Chọn trigger ▾]
- Radio row: Logic ● OR ○ AND
- Info box: "Mỗi trigger có message riêng. KH match trigger nào → nhận message của trigger đó."
- Priority conflict option: ● Chỉ gửi trigger P1 ○ Gửi tất cả
- Estimate box (light blue bg): ~6,800 KH × 2 kênh = ~13,600 tin | Chi phí SMS ~720,000 VND

Section 4 — "4. Message Matrix · Trigger × Kênh" [Thu gọn]:
- Time config box: ● Gửi ngay | ○ Sau __ phút/giờ | ○ Vào lúc __:__
- Progress bar: Push ●● Zalo ●● SMS ●○ Banner ○○ Email ○○ USSD ○○
- Tab bar: [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○] [+ Kênh]
- Active tab PUSH shows 2 message cards. Each card = 2-column inline:
  LEFT (55%): PARAMS chips | Image upload box | Template dropdown | Title input (0/65) | Body textarea (0/240) | [+ Biến thể đối tượng]
  RIGHT (45%): Phone notification mockup (live preview) | SAMPLE PARAMS inputs

Section 6 — "6. An toàn" [Thu gọn]:
GROUP A — Blackout:
  Toggle Bật/Tắt | Time pickers: 22:00 — 08:00 | Dropdown: [Hủy luôn ∨] (options: Hủy luôn / Delay đến đầu khung giờ)
GROUP B — Suppression:
  Checkbox ☑ Check DNC toàn hệ thống
  Blacklist radio: ○ Không dùng ● Dùng tệp riêng: [BL_ESIM_Q2_2026 · 320 KH ▾] [Xem][Thay][Upload mới]
  Whitelist radio: ● Không dùng ○ Chỉ gửi cho whitelist
  Reach cuối: ~6,480 KH = 6,800 → trừ DNC → trừ BL 320
  Note: ⓘ Frequency cap tại System Settings [Đến System Settings →]

RIGHT COLUMN (~40%, sticky):

Top: Campaign Summary mini card:
  ⚠ 2 issue · Reach ~6,480 KH | Chi phí SMS ~720,000 VND

Below: Section 3 — Audience:
  Nguồn: Customer 360 · Team Data · BSS · OCS
  Search dropdown: [🔍 Tìm phân khúc... ▾]
  Segment tag cards: [Gen Z User (18–25) · 18,450 KH ×] [Sắp hết data · 8,920 KH ×]
  Logic: ● OR ○ AND
  Filter row (collapsed): ▸ Điều kiện lọc thêm [Mở rộng]
  Reach: ~6,800 KH [↻ Tính lại]

Below: Section 5 — Channel Strategy:
  Accordion: "Đặt lịch gửi theo kênh (optional) ▸ [Mở rộng]"
```

---

## SCREEN 3 — Message Matrix: Audience Variant

```
Draw the Audience Variant (Biến thể đối tượng) state inside Section 4 Message Matrix, Tab Push, for trigger T1 · SIM_ACTIVATED.

Show a full-width card with:
- Header row: "T1 · SIM_ACTIVATED" | PARAMS chips: {{ten_kh}} {{loai_sim}} {{ngay_kich_hoat}} {{so_dien_thoai}}

Variant 1 card (full width, slightly highlighted as default/fallback):
- Sub-header: "Biến thể 1 · Đối tượng: [Tất cả ▾]" — label "(mặc định — fallback khi không match biến thể nào khác)"
- 2-column layout:
  LEFT: Image upload box (1:1) | Template dropdown [Welcome Push ▾] | Title: "Chào mừng bạn!" | Body: "SIM {{loai_sim}} đã kích hoạt..."
  RIGHT: Push notification mockup | SAMPLE PARAMS inputs
- No [×] delete button (fallback cannot be deleted)

Variant 2 card (below, bordered):
- Sub-header: "Biến thể 2 · Đối tượng: [VIP ▾]" — label "(ưu tiên khi KH thuộc segment này)" | [×] delete button top right
- 2-column layout:
  LEFT: Image upload box | Template [Welcome VIP ▾] | Title: "Ưu đãi đặc biệt dành cho VIP" | Body: "Cảm ơn bạn đã tin dùng..."
  RIGHT: Push notification mockup | SAMPLE PARAMS

Footer of card:
- Button [+ Biến thể đối tượng]
- Info note: ⓘ KH match nhiều biến thể → áp dụng biến thể thứ tự cao nhất; Biến thể 1 luôn là fallback.
```

---

## SCREEN 4 — Template Management

```
Draw Screen 4A: Template List

Full browser with left nav. "Template" highlighted in nav.

Header: "TEMPLATE" (page title) + [+ Tạo Template] button right

Filter row: [🔍 Tìm tên template...] | Kênh: [Tất cả ▾] | Trạng thái: [Tất cả ▾]

Table columns: Tên Template | Kênh hỗ trợ | Dùng (số lần) | Hành động

Rows:
1. Chào mừng SIM | Push · Zalo | 3 lần | [Sửa] [Clone] [Tắt]
2. Nhắc nạp thẻ | SMS · USSD | 5 lần | [Sửa] [Clone] [Tắt]
3. Sắp hết data | Push · SMS | 2 lần | [Sửa] [Clone] [Tắt]
4. Sinh nhật KH | Zalo · Email | 4 lần | [Sửa] [Clone] [Tắt]
5. Cài app nhắc nhở | Push | 1 lần | [Sửa] [Clone] [Bật] (row slightly grayed = Inactive)

Pagination: < 1 2 > [20/trang ▾]

---

Draw Screen 4B: Template Editor (below or separate)

Page header: ← Template | [Tên template *___________] input | toggle ● Active | [Lưu Template] button
Sub-row: [Mô tả (optional)___] input

Progress row: Push ● Zalo ● SMS ○ USSD ○ Banner ○ Email ○

Tab bar: [Push ●] [Zalo OA ●] [SMS ○] [USSD ○] [Banner ○] [Email ○] [+ Kênh]

Active tab Push — 2-column card:
LEFT:
  THAM SỐ ĐỘNG chips: [{{ten_kh}}] [{{loai_sim}}] [{{so_dien_thoai}}] [{{so_du}}] [{{data_con_lai}}] [{{ngay_het_han}}]
  Note: → Click chip để chèn vào nội dung
  Image upload: [Tải lên] [Chọn thư viện]
  Title *: [Chào mừng {{ten_kh}}!]
  Body *: [SIM {{loai_sim}} của bạn đã được kích hoạt thành công.]
RIGHT:
  Phone push notification mockup (rendered)
  SAMPLE PARAMS:
    ten_kh: [Nguyễn Văn A]
    loai_sim: [eSIM]
    data_con_lai: [120 MB]
  [↻ Render lại] button
```

---

## SCREEN 5 — Trigger Management

```
Draw Screen 5: Trigger Management

Full browser with left nav. "Trigger" highlighted.

Header: "TRIGGER MANAGEMENT" + [+ Thêm Trigger] button
Filter: [🔍 Tìm trigger code hoặc tên...] | Trạng thái: [Tất cả ▾]

3 collapsible group sections:

▼ REALTIME (2 trigger) — expanded:
Table: Code | Tên | Source | Trạng thái | Hành động
SIM_ACTIVATED | Kích hoạt SIM | BSS | ● Active (green) | [Sửa][Tắt]
LOC_TRAVEL_PROV | Đến tỉnh du lịch | OCS | ● Active | [Sửa][Tắt]

▼ NEAR REALTIME (1 trigger) — expanded:
NO_APP_INSTALL_24H | Chưa cài app sau 24h | CRM | ● Active | [Sửa][Tắt]

▼ OFFLINE / BATCH (3 trigger) — expanded:
LOW_DATA_BALANCE | Data còn dưới 100MB | OCS | ● Active | [Sửa][Tắt]
USAGE_DROP_40PCT | Usage giảm >40% | OCS | ○ Testing (blue) | [Sửa][Bật]
SIM_NO_TXN_30D | SIM ngừng GD >30 ngày | BSS | ○ Paused (orange) | [Sửa][Bật]

Also show the "Thêm Trigger Mới" modal floating on top:
Modal fields: Trigger code * | Loại * [Realtime ▾] | Tên trigger * | Event source [BSS/OCS/CRM/Other ▾] | Trạng thái toggle | Mô tả | Payload table (2 rows with [source ▾] and [✕])
Buttons: [Hủy] [Lưu]
```

---

## SCREEN 6 — Blacklist Management

```
Draw Screen 6A: Blacklist List

Full browser with left nav. "Blacklist" highlighted.

Header: "BLACKLIST"
Filter row: Campaign: [Tất cả ▾] | Kênh: [Tất cả ▾] | [Lọc]

Table columns: Số điện thoại | Campaign | Kênh | Hành động
0987 xxx 001 | Chào mừng SIM | Zalo OA | [Xóa]
0912 xxx 002 | Hết data | SMS | [Xóa]
0965 xxx 003 | Hết data | Zalo OA | [Xóa]
0976 xxx 004 | Nhắc nạp tiền | Push | [Xóa]

Pagination: < 1 2 3 >
Buttons below: [+ Thêm thủ công] | [📥 Upload danh sách]

Also show the Upload modal (6C) floating on top:
Fields: Campaign * [dropdown] | Kênh * [dropdown] | File CSV (drag-drop zone with format note [📥 Tải file mẫu])
Preview result: ✅ 245 số hợp lệ | ⚠ 3 số sai định dạng (bỏ qua)
Buttons: [Hủy] [Xác nhận Upload]
```

---

## SCREEN 7 — Customer List

```
Draw Screen 7: Customer List

Full browser with left nav. "Customer" highlighted.

Header: "KHÁCH HÀNG"
Search: [🔍 Tìm số điện thoại...]
Filter row: Trạng thái SIM: [Tất cả ▾] | Cài app: [Tất cả ▾] | DNC: [Tất cả ▾]

Table columns: Số điện thoại | Loại SIM | Trạng thái | Cài app | Hành động
0987 xxx 001 | eSIM | ● Active (green) | Có | [Xem 360]
0912 xxx 002 | SIM vật lý | ● Active | Không | [Xem 360]
0965 xxx 003 | eSIM | ○ Inactive (gray) | Có | [Xem 360]
0976 xxx 004 | SIM vật lý | ● Active | Không | [Xem 360]

Pagination: < 1 2 ... > [20/trang ▾]
```

---

## SCREEN 8 — Customer 360

```
Draw Screen 8: Customer 360

Full browser with left nav.

Page header: ← Danh sách khách hàng | Title: "CUSTOMER 360 — 0987 xxx 001"

Two-column layout:

LEFT COLUMN — "THÔNG TIN & SỬ DỤNG":
Profile section:
  Loại SIM: eSIM · Active
  Gói: D50 · hết 18/05/2026
  Ngày KH: 01/04/2026
  Cài app: Có · Đăng nhập: Có

Usage section:
  Data còn: 1.2 GB
  Hôm nay: 320 MB
  Số dư: 45,000 VND
  Nạp tiền: 3 lần
  Sinh nhật: 15/08

RIGHT COLUMN — "TRẠNG THÁI KÊNH & LỊCH SỬ":
Channel status table:
  Kênh | Trạng thái | Cập nhật
  Zalo | ⛔ Blocked (red) | 05/05 09:32
  SMS  | ✅ Active (green) | —
  USSD | ✅ Active | —
  Push | ✅ Active | —
Note: ⓘ Tự cập nhật từ phản hồi Gateway

Throttling box:
  Tin hôm nay: 2/3
  Cooldown: Không · DNC: Không

Message History list (most recent first):
  10/05 | Chào mừng SIM | Zalo · ✓ Delivered
  08/05 | Nạp thẻ lỗi | SMS · ✓ Delivered
  07/05 | Hết data | Zalo · ✕ Blocked → SMS · ✓ Delivered (2-sub-row entry)

Button: [Xem đầy đủ lịch sử →]
```

---

## SCREEN 9 — Report / Analytics

```
Draw Screen 9: Report — Tab 1 Delivery + Tab 2 Engagement

Full browser with left nav. "Report" highlighted.

Page header: "ANALYTICS & REPORT" + [Xuất Excel] button

Filter bar:
  Thời gian: [7 ngày ▾] [Tháng này]
  Campaign: [Tất cả ▾]
  Segment: [Tất cả ▾]
  [So sánh: ON ▾]
  Kênh: [Tất cả ▾]

Tab navigation: [Delivery] [Engagement] [Campaign Comparison] [Segment] [Funnel] [Spam]
Active tab: Delivery

TAB 1 — DELIVERY ANALYTICS:

Summary KPI cards (4 in a row):
  Total Sent: 124,500
  Delivered: 119,940 (96.3%)
  Failed: 4,560 (3.7% ⚠ orange)
  Delivery Rate: 96.3%

Area chart (full width): "SENT vs DELIVERED" — line chart 7 days, 3 series (Sent / Delivered / Failed area)

2 charts side by side:
  Left: "DELIVERY RATE TREND" — line chart % per day, dashed line at 95% target
  Right: "FAILED DELIVERY TREND" — stacked bar chart, 4 failure reasons (Gateway error / User blocked / DNC-Blacklist / Network timeout)

---

TAB 2 — ENGAGEMENT ANALYTICS (show below or as alt state):

KPI cards (4):
  Open Rate: 38.4% (↑2.1%)
  CTR: 12.1% (↑0.8%)
  Conversion Rate: 8.3% (↑0.4%)
  Best Channel: Push 97.7%

Horizontal bar chart: "CHANNEL PERFORMANCE COMPARISON"
  Push    | ████████████ Open 52% CTR 18%
  Zalo OA | ██████████  Open 44% CTR 15%
  SMS     | ████████    Open 22% CTR  8%
  USSD    | ██████      Open 15% CTR  5%
  Banner  | ████        Open  9% CTR  3%
  Email   | ████████    Open 21% CTR  9%
  Legend: ■ Open Rate ░ CTR ▒ Conversion

Line chart below: "ENGAGEMENT TREND" — 3 lines (Open / CTR / Conversion) over 7 days
```

---

## TIP SỬ DỤNG

- **Lovable**: Dán System Prompt trước, sau đó dán từng Screen Prompt vào cuộc trò chuyện. Nếu muốn toàn bộ app cùng lúc, dán System Prompt + tất cả Screen Prompt liên tiếp.
- **Gemini**: Dán System Prompt + 1 Screen Prompt trong 1 lần nhắn. Thêm dòng "Generate as a UI mockup image / React wireframe component" cuối prompt.
- Muốn **interactive prototype**: thêm vào cuối mỗi Screen Prompt: *"Make it interactive with hover states and basic click navigation between screens."*
- Muốn **dark mode**: thêm vào System Prompt: *"Use dark mode: #0F172A background, #1E293B cards, #E2E8F0 text."*
