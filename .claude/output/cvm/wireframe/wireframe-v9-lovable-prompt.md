# Lovable Prototype Prompt — CVM System v9

> **Cách dùng:** Dán toàn bộ file này vào Lovable (hoặc chia làm 2 lần: phần FOUNDATION trước, sau đó phần SCREENS + INTERACTIONS).

---

## PART 1 — FOUNDATION

```
Build a fully interactive React prototype for a B2B SaaS internal tool called CVM (Customer Value Management) — a marketing automation platform used by telecom operators to manage customer campaigns across multiple channels (Push, Zalo OA, SMS, USSD, Banner, Email).

Tech stack: React + TypeScript + Tailwind CSS + shadcn/ui. Use React Router for navigation between screens.

---

DESIGN SYSTEM:

Colors:
- Background: #FFFFFF
- Sidebar bg: #F8FAFC
- Border: #E2E8F0
- Text primary: #1E293B
- Text secondary: #64748B
- Brand blue (primary button): #3B82F6
- Success / Active status: #22C55E
- Warning / Pending status: #EAB308
- Paused status: #F97316
- Draft status: #94A3B8
- Error / danger: #EF4444
- Info bg (estimate box): #EFF6FF

Typography: Inter font. Sizes: 12px (label), 13px (table), 14px (body), 16px (section title), 20px (page title).

Spacing: 8px base unit. Cards: rounded-lg, shadow-sm, border border-slate-200.

---

PERSISTENT LAYOUT (appears on all screens):

Left sidebar (fixed, w-56, bg-slate-50, border-r):
- Top: logo "[CVM]" bold
- Nav items (click → navigate):
  • Dashboard → /dashboard
  • Campaign → /campaigns
  • Template → /templates
  • Trigger → /triggers
  • Khách hàng → /customers
  • Blacklist → /blacklist
  • Report → /report
  • Settings → /settings (empty placeholder screen)
- Active item: bg-blue-50 text-blue-700 font-medium, left blue border

Top header bar (fixed, h-14, border-b, bg-white):
- Left: sidebar toggle icon + breadcrumb (auto-updates based on current route)
- Right: notification bell icon (no action) + avatar circle "QTV" (no action)

Main content area: ml-56 mt-14, p-6, overflow-y-auto

---

INITIAL STATE when prototype loads:
- Navigate to /dashboard
- "Dashboard" highlighted in sidebar
```

---

## PART 2 — SCREENS & INTERACTIONS

---

### SCREEN 1 — /dashboard

```
Page: Dashboard

Show a scrollable main content area with 5 rows of content.

ROW 1 — KPI Cards (grid 4 columns, 2 rows = 8 cards total):
Each card: white bg, rounded-lg, border, p-4, shadow-sm.

Card data:
1. Active Campaigns | 12 | "↑2 vs hôm qua" green
2. Trigger Fired Today | 3,842 | "↑18% vs hôm qua" green
3. Messages Sent Today | 48,320 | sparkline visual (use a simple SVG path)
4. Delivery Rate | 96.4% | "● SLA: OK" green badge
5. Failed Messages | 1,760 | "⚠ ↑3% vs hôm qua" — this card has red border + orange warning icon
6. Conversion Rate | 8.3% | "↑0.4% vs tuần" green
7. Realtime SLA | <2.1s avg | "✅ Target: <3s"
8. Blacklist Blocked Today | 4,210 | two sub-lines: "BSS DNC: 3,840" and "CVM BL: 370"

INTERACTION: Click any card → no navigation (tooltip "Drill-down coming soon" on hover is fine).

ROW 2 — System Health (2 charts side by side using recharts or a simple SVG placeholder):
- Left: "TRIGGER / EVENT VOLUME" area chart placeholder (24h, label only)
- Right: "PROCESSING LATENCY" line chart placeholder (p50/p95/p99)
- Below left: "QUEUE & BACKLOG" info card — show: Processing queue 342 (progress bar 68%), Pending blackout 1,240, Scheduled 8,900. Button [Xem queue] → no action, show toast "Tính năng đang phát triển"
- Below right: "TOP SYSTEM ERRORS" — 3 error rows with red ⛔ icon: GATEWAY_ZALO_TIMEOUT (128), SMS_SEGMENT_EXCEED (45), DEDUP_COLLISION (12). Button [Xem tất cả →] → toast

ROW 3 — Campaign Monitoring:
- Left table "TOP ACTIVE CAMPAIGNS": 4 rows (Nhắc nạp tiền, Hết hạn data, LOW_DATA batch, Welcome eSIM). Columns: Campaign | Sent | Rate | Trend.
  INTERACTION: Click any campaign row → navigate to /campaigns
- Right table "TRIGGER FIRED NHIỀU NHẤT": 4 rows (LOW_DATA_BALANCE 1,842, SIM_ACTIVATED 940, NO_APP_INSTALL_24H 620, TOP_UP_FAIL 440).
- Below: "RECENT TRIGGER EVENTS TIMELINE" — list of 5 live event rows. Button [Live ● pause] → toggles to [▶ Resume], list stops "updating".

ROW 4 — User Journey Funnel:
Horizontal funnel with 5 steps, each step is a bar (decreasing width). Show % and absolute number. Dropdown [Chọn campaign ▾] → no action. Button [Xem chi tiết →] → navigate to /report.

ROW 5 — Trigger Analytics:
Left table: TOP TRIGGERS (7 ngày) — 5 rows with match rate mini bar.
Right card: TRIGGER ANOMALY DETECTION — yellow/orange alert card for LOW_DATA_BALANCE spike +340%.
Below: TRIGGER HEATMAP — 7 rows × 24 cols grid, color coded (use bg-slate-100 to bg-slate-800 shades). One cell marked spike with bg-red-200.
```

---

### SCREEN 2 — /campaigns

```
Page: Campaign List

Header: "Campaign" h1 + button [+ Tạo Campaign] → navigate to /campaigns/new

Search input: full width, placeholder "🔍 Tìm tên campaign, mã hoặc trigger code..."
INTERACTION: typing filters the table rows by campaign name (client-side filter on the 4 rows below).

Filter chips (multi-select, toggle on click, active chip = bg-blue-100 text-blue-700):
[Tất cả] [Active] [Draft] [Pending] [Paused] [Realtime] [Offline] [Có SMS] [Chưa đủ nội dung]
INTERACTION: clicking [Active] filters table to show only Active row. Clicking again deselects.

Table columns: TÊN / MÃ CAMPAIGN | TRIGGER | HIỆU LỰC | TRẠNG THÁI | HÀNH ĐỘNG

Row 1: "Welcome eSIM Q2/2026" / "CVM-WELCOME-ESIM-Q2-2026" | [SIM_ACT] [NO_APP] +1ⓘ | 01/04–30/06/2026 | ● Draft (gray chip) | [Xem] [Sửa]
Row 2: "Nhắc nạp tiền" / "CVM-REMIND-TOPUP-05-2026" | [TOP_UP] | 01/05–31/05/2026 | ● Active (green chip) | [Xem] [Dừng]
Row 3: "Hết hạn gói data" / "CVM-DATA-EXPIRE-05-2026" | [PKG_EXP] | 01/05–31/05/2026 | ● Paused (orange chip) | [Xem] [Bật]
Row 4: "Chào mừng du lịch" / "CVM-TRAVEL-PROMO-Q2" | [LOC_TRV] | 15/05–30/06/2026 | ⏳ Pending (yellow chip, sub-text "Chờ duyệt") | [Xem]

Status chip component: colored dot + text, rounded-full, px-2 py-0.5, small font.

INTERACTIONS:
- [Xem] on any row → navigate to /campaigns/:id/detail (show Campaign Detail View, use row 1 data as default)
- [Sửa] on Draft row → navigate to /campaigns/:id/edit (Campaign Builder)
- [+ Tạo Campaign] → navigate to /campaigns/new (Campaign Builder, empty state)
- [Dừng] on Active row → open ConfirmDialog: title "Dừng campaign?", body "Message đang trong queue sẽ bị hủy. Không thể hoàn tác.", buttons [Hủy] [Xác nhận Dừng] (danger red). On confirm → row status changes to Paused, button changes to [Bật], show success toast "Campaign đã dừng".
- [Bật] on Paused row → no confirm needed → status changes to Active, button changes to [Dừng], show toast "Campaign đã kích hoạt lại".
- +1ⓘ trigger chip → hover/click shows Popover with list: "LOC_TRAVEL_PROV · Đến tỉnh du lịch · OCS · Realtime"

Pagination: show < 1 2 3 > decorative (no action needed).
```

---

### SCREEN 2B — /campaigns/:id/detail

```
Page: Campaign Detail View (read-only)

Fixed page header (not the global header — this is a sub-header within main content):
- "← Campaign" link → navigate back to /campaigns
- Campaign name + code: "Welcome eSIM Q2/2026 · CVM-WELCOME-ESIM-Q2-2026"
- Status chip: ⏳ Pending (yellow)
- Button [Đóng] → navigate to /campaigns

Scrollable content below header, organized as labeled sections with dividers:

SECTION 1 — Thông tin Campaign:
Display as 2-col label-value grid (read-only, no inputs):
  Tên campaign: Welcome eSIM Q2/2026
  Mã kịch bản: CVM-WELCOME-ESIM-Q2-2026
  Mục tiêu: Onboard eSIM, nhắc cài app
  Thời gian: 01/04/2026 – 30/06/2026
  Người tạo: QTV Marketing
  Ngày tạo: 10/05/2026 14:32
  Ngày gửi duyệt: 12/05/2026 09:15

SECTION 2 — Trigger & Logic:
  Chế độ: Advanced · Logic: OR
  Triggers: chip "★ P1 SIM_ACTIVATED" + chip "P2 NO_APP_INSTALL_24H"
  Ưu tiên: Chỉ gửi trigger P1
  Ước tính: ~6,800 KH × 2 kênh = ~13,600 tin (shown in blue info box)

SECTION 3 — Audience:
  Phân khúc: tag [Gen Z User (18–25)] [Sắp hết data]
  Logic: OR
  Điều kiện lọc: Loại thiết bị = Android
  Reach: ~6,800 KH

SECTION 4 — Message Matrix:
  Thời gian gửi: Gửi ngay sau khi trigger kích hoạt
  Tab bar: [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○]
  Default tab: Push
  INTERACTION: Click tab → show content for that tab (all tabs switchable)
  Each tab shows read-only message cards. Push tab shows:
    Card: T1 · SIM_ACTIVATED | Title: "Chào mừng bạn!" | Body: "SIM eSIM đã kích hoạt thành công. Khám phá MyViettel."
    Card: T2 · NO_APP_INSTALL_24H | Title: "Cài MyViettel nhận ưu đãi" | Body: "Bạn vẫn chưa cài app MyViettel. Cài ngay để nhận ưu đãi."
  Other tabs show placeholder "Chưa cấu hình nội dung" in empty state card.

SECTION 5 — Channel Strategy:
  Đặt lịch theo kênh: Không (tất cả theo thời gian gửi message)

SECTION 6 — An toàn:
  Blackout: Bật · 22:00 – 08:00 · Hủy luôn
  DNC toàn hệ thống: Bật ✓
  Blacklist campaign: BL_ESIM_Q2_2026 · 320 KH
  Whitelist: Không dùng
  Reach cuối cùng: ~6,480 KH (green bold)

No edit buttons anywhere — entire page is display-only.
```

---

### SCREEN 3 — /campaigns/new AND /campaigns/:id/edit

```
Page: Campaign Builder

FIXED SUB-HEADER (sticky top within main content, bg-white, border-b, z-10):
Left: "← Campaign" → navigate to /campaigns | campaign name "Welcome eSIM Q2/2026" | status badge "● Draft"
Right: [Lưu Nháp] button (secondary/outline) | [Gửi duyệt →] button (primary blue)

STATE: Track a "issues" count in local state, default = 2.
- [Gửi duyệt →] is DISABLED (opacity-50, cursor-not-allowed) when issues > 0. Show red badge "⚠ 2 issue" on the button.
- Hover on disabled button → tooltip "Cần hoàn thiện: SMS T2 chưa có nội dung, USSD T1 chưa có nội dung"
- [Lưu Nháp] always enabled → click shows toast "Đã lưu nháp ✓"

TWO-COLUMN LAYOUT below sub-header:
Left column: width 60%, overflow-y-auto (scrolls independently)
Right column: width 40%, sticky (position sticky, top = sub-header height)

--- LEFT COLUMN ---

SECTION 1 — "1. Thông tin Campaign" (collapsible, default expanded):
[Thu gọn] button top-right → collapses section body.
Fields:
  - "Tên campaign *" text input, value "Welcome eSIM Q2/2026", full width
  - "Mã kịch bản" text input, value "CVM-WELCOME-ESIM-Q2-2026"
  - Row: "Mục tiêu" textarea (short) | "Thời gian hiệu lực" date range display "01/04/2026 – 30/06/2026"
  - "Người tạo: QTV Marketing" plain text (read-only, gray)

SECTION 2 — "2. Trigger & Logic" (collapsible, default expanded):
Radio buttons: ○ Basic ● Advanced (default Advanced selected)

Trigger list (vertical):
  Row 1: star "★" badge + "P1" label + chip [SIM_ACTIVATED · Kích hoạt SIM thành công] + drag handle [≡] + [✕] delete
  Row 2: "P2" label + chip [NO_APP_INSTALL_24H · Chưa cài app sau 24h] + [≡] + [✕]

INTERACTION:
  - [✕] on P2 → removes that trigger chip, re-numbers (only P1 remains)
  - [✕] on P1 (if only P1 left) → shows toast "Phải có ít nhất 1 trigger"
  - [+ Chọn trigger ▾] button → opens Dropdown with options: [LOW_DATA_BALANCE] [TOP_UP_FAIL] [LOC_TRAVEL_PROV]. Click option → adds new trigger chip as next P.

Logic radio: ● OR ○ AND
INTERACTION — switching OR ↔ AND:
  - Click AND → open ConfirmDialog: "Chuyển sang AND mode sẽ xóa toàn bộ Biến thể đối tượng đã tạo. Tiếp tục?" [Hủy] [Xác nhận]. On confirm → switch to AND state.
  - In AND mode: info text changes to "KH phải thỏa đồng thời tất cả trigger mới được gửi tin. Tất cả trigger dùng chung 1 message."
  - In OR mode: info text "Mỗi trigger có message riêng." + priority radio: ● Chỉ gửi trigger P1 ○ Gửi tất cả trigger đã match

Estimate info box (blue bg, only shown when triggers exist):
  "Ước tính tin nhắn: ~6,800 KH × 2 kênh = ~13,600 tin"
  "Chi phí SMS ước tính: ~720,000 VND"
  "ⓘ Tính dựa trên audience và kênh đã cấu hình"

SECTION 4 — "4. Message Matrix · Trigger × Kênh" (collapsible, default expanded):

Time config box (white bg, bordered):
  Radio: ● Gửi ngay sau khi trigger kích hoạt | ○ Sau [__] [phút ▾] kể từ T | ○ Vào lúc [__:__] ngày T+[__]
  INTERACTION: selecting second/third radio shows the input fields inline.

Progress indicator row:
  Push ●● (green) | Zalo ●● (green) | SMS ●○ (orange) | Banner ○○ (gray) | Email ○○ (gray) | USSD ○○ (gray)
  "● = hoàn chỉnh  ○ = chưa cấu hình"

Tab bar: [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○] [+ Kênh]
INTERACTION: click tab → show content for that tab. Default = Push.

=== TAB: PUSH (default active) ===
Show 2 message cards (one per trigger T1 and T2).

Each card structure:
- Card header: completion icon (✓ green or ⚠ orange) + "T1 · SIM_ACTIVATED" or "T2 · NO_APP_INSTALL_24H"
- Two columns inside card:
  LEFT (55%) — compose area:
    PARAMS chips (clickable): [{{ten_kh}}] [{{loai_sim}}] [{{ngay_kich_hoat}}] [{{so_dt}}]
    INTERACTION: click chip → appends "{{chip_name}}" to the Body textarea cursor position
    Image upload zone: dashed border, "🖼 Kéo thả hoặc [Tải lên] / [Chọn thư viện]". Ratio 1:1.
    Template dropdown: [Welcome Push ▾] → on click shows dropdown with 3 options (Chào mừng SIM, Nhắc nạp thẻ, Sắp hết data). Selecting one → fills Title and Body with template text.
    Title input (label "Title *", counter "X/65"): value "Chào mừng bạn!" (T1) or "Cài MyViettel nhận ưu đãi" (T2)
    Body textarea (label "Body *", counter "X/240"): value filled for T1, empty for T2
    INTERACTION: typing in Title/Body → updates counter. Counter goes red when over limit.
    [+ Biến thể đối tượng] button (HIDDEN when Logic = AND mode, VISIBLE in OR mode):
      → click: adds a second variant card below (Audience Variant state — see below)
  RIGHT (45%) — live preview panel:
    Shows phone notification mockup (rectangle resembling phone notif):
      App icon circle + "CVM App" text | Title text (renders from input) | Body text (truncated)
    SAMPLE PARAMS section:
      Input fields: ten_kh [Nguyễn Văn A] | loai_sim [eSIM] | ngay_kich_hoat [01/04/2026] | so_dt [0987xxx001]
      INTERACTION: typing in sample params → re-renders the preview with actual values substituted into {{placeholders}}
    [↻ Render lại] button → re-triggers render

T1 card is complete (green ✓ header). T2 body is empty → orange ⚠ header.

AUDIENCE VARIANT STATE (when [+ Biến thể đối tượng] clicked on T1):
Expands T1 card to show 2 variant sub-cards:
  Variant 1 header: "Biến thể 1 · Đối tượng: [Tất cả ▾]" + italic "(mặc định — fallback)" — no delete button
  Variant 2 header: "Biến thể 2 · Đối tượng: [VIP ▾]" + [×] delete button
    INTERACTION: [×] on Variant 2 → ConfirmDialog "Xóa biến thể này?" [Hủy] [Xóa]. On confirm → Variant 2 removed, T1 returns to single-variant state.
    Dropdown [VIP ▾] → shows segments: [Gen Z User] [Sắp hết data] [VIP] [ARPU cao]
  Below both variants: [+ Biến thể đối tượng] button + info note "ⓘ KH match nhiều biến thể → áp dụng biến thể thứ tự cao nhất"

=== TAB: SMS ===
2 cards same structure. SMS-specific:
  - No image upload, show "(Không hỗ trợ hình ảnh)" note
  - Body counter: X/160. Over 160 → counter red + badge "X SMS segment" appears
  - Preview: SMS bubble mockup (gray bubble, "Viettel" sender, text content)
  T1 has content (92/160). T2 is empty.

=== TAB: USSD ===
2 cards same structure. USSD-specific:
  - No image upload
  - Body counter: X/182. Plain text note "⚠ Chỉ plain text, không ký tự đặc biệt"
  - Preview: monospace terminal-style box
  Both T1 and T2 empty.

=== TAB: ZALO OA ===
2 cards. Zalo-specific:
  - Image upload optional (free ratio)
  - Body counter: X/1000
  - Preview: chat bubble with "🟦 Viettel Telecom" brand header

=== TAB: BANNER ===
2 cards. Banner-specific:
  - Image upload REQUIRED (16:9). Show warning "⚠ Chưa upload image → blocking validation" if missing.
  - Fields: Title (0/65) | Body (0/120) | CTA Label * | CTA URL *
  - Preview: card with 16:9 image placeholder + title + body + CTA button

=== TAB: EMAIL ===
2 cards. Email-specific:
  - Header image optional (banner ratio)
  - Fields: Subject (0/100) | Body plain text
  - Preview: email client frame with From/Subject/body

=== TAB: [+ Kênh] ===
Click → opens small dropdown listing channels NOT yet added. Selecting one → adds new tab.

SECTION 6 — "6. An toàn" (collapsible, default expanded):

GROUP A — BLACKOUT:
  Toggle: [Bật] (active) / [Tắt]
  INTERACTION: toggle off → hides time pickers and dropdown
  When on: show time pickers "22:00 — 08:00" + dropdown [Hủy luôn ∨]
  Dropdown options: "Hủy luôn" / "Delay đến đầu khung giờ". Click → changes selected value.

GROUP B — SUPPRESSION:
  Checkbox ☑ "Check DNC toàn hệ thống (luôn bật mặc định)"
  INTERACTION: unchecking → ConfirmDialog "Tắt kiểm tra DNC có thể gửi tin đến KH đã từ chối. Tiếp tục?" [Hủy] [Tắt DNC]. On cancel → checkbox stays checked.

  Blacklist radio:
    ○ Không dùng
    ● Dùng tệp riêng: shows [BL_ESIM_Q2_2026 · 320 KH ▾] chip + [Xem] [Thay] [Upload mới]
    INTERACTION:
      - Radio "Không dùng" → hides tệp row, reach recalculates (removes BL deduction)
      - [Xem] → opens Modal: "Danh sách BL_ESIM_Q2_2026" with table of 3 masked phone numbers + "Hợp lệ: 320 · Trùng: 8 · Sai định dạng: 2". Button [Đóng].
      - [Upload mới] → opens Upload Modal (see below)
      - Selecting "Dùng tệp riêng" but NOT selecting a file → validation error "Vui lòng chọn tệp blacklist" shown inline + [Gửi duyệt] stays disabled

  Whitelist radio:
    ● Không dùng
    ○ Chỉ gửi cho whitelist: (selecting shows tệp picker, same pattern as BL)

  Reach formula display:
    "~6,480 KH = 6,800 → trừ DNC (~300) → trừ BL 320"
    INTERACTION: reach number updates when BL/WL toggle changes (simulate: BL off → reach shows 6,800; BL on → 6,480)

  Info note: "ⓘ Giới hạn tần suất nhận tin (frequency cap) được cấu hình tại System Settings" + [Đến System Settings →] link → navigate to /settings

--- RIGHT COLUMN (sticky) ---

CAMPAIGN SUMMARY MINI CARD (top of right column, always visible):
  Background: white, border, rounded-lg, p-4
  Content:
    ⚠ 2 issue (red) · Reach ~6,480 KH (green) | Chi phí SMS ~720,000 VND
  INTERACTION: issue count updates reactively as user fills in content (e.g. when T2 SMS body is filled in, issue count decreases; at 0 issues → [Gửi duyệt] button becomes enabled, badge disappears, summary shows ✓ green)

SECTION 3 — Audience / Phân khúc:
  "Nguồn: Customer 360 · Team Data · BSS · OCS" gray label
  "Cập nhật: 13/05/2026 08:00"
  Search dropdown: [🔍 Tìm phân khúc... ▾]
    INTERACTION: click → opens dropdown with options: Gen Z User (18–25) 18,450 KH | Sắp hết data 8,920 KH | VIP 5,200 KH | ARPU cao 5,600 KH | Nguy cơ rời mạng 2,150 KH. Selecting one → adds segment card below.
  Segment tag cards (already selected):
    [Gen Z User (18–25) · 18,450 KH ×] [Sắp hết data · 8,920 KH ×]
    INTERACTION: [×] on a card → removes it, reach recalculates
  Logic: ● OR ○ AND (toggle, no confirm needed for audience logic)
  ▸ Điều kiện lọc thêm [Mở rộng]
    INTERACTION: click [Mở rộng] → expands to show filter row: [Loại thiết bị ▾] [= ▾] [Android ▾] [×]. Click [×] → removes filter. Button [+ Thêm điều kiện] → adds new filter row.
  Reach: ~6,800 KH [↻ Tính lại] button (no actual recalc needed, just show a 500ms loading spinner then same number)

SECTION 5 — Channel Strategy:
  Collapsed by default.
  ▸ Đặt lịch gửi theo kênh (optional) [Mở rộng]
  INTERACTION: click [Mở rộng] → expands to show per-channel schedule rows:
    Push: ● Không đặt lịch riêng ○ Gửi vào [__:__] [Hằng ngày ▾]
    Zalo OA: same
    SMS: same
    (Only shows channels that have content in Message Matrix)
    Selecting "Gửi vào" radio → shows time input + frequency dropdown inline.
```

---

### SCREEN 4 — /templates

```
Page: Template List

Header: "TEMPLATE" + [+ Tạo Template] → navigate to /templates/new

Search input: filter rows client-side.
Filter row: Kênh [Tất cả ▾] | Trạng thái [Tất cả ▾]

Table: Tên Template | Kênh hỗ trợ | Dùng | Hành động

Rows:
1. Chào mừng SIM | Push · Zalo | 3 lần | [Sửa][Clone][Tắt]
2. Nhắc nạp thẻ | SMS · USSD | 5 lần | [Sửa][Clone][Tắt]
3. Sắp hết data | Push · SMS | 2 lần | [Sửa][Clone][Tắt]
4. Sinh nhật KH | Zalo · Email | 4 lần | [Sửa][Clone][Tắt]
5. Cài app nhắc nhở | Push | 1 lần | [Sửa][Clone][Bật] (row text grayed = Inactive)

INTERACTIONS:
- [Sửa] → navigate to /templates/:id (Template Editor, pre-filled)
- [+ Tạo Template] → navigate to /templates/new (Template Editor, empty)
- [Clone] → shows toast "Đã tạo bản sao: Copy of [Tên]" + new row appears at top of table
- [Tắt] → ConfirmDialog "Tắt template này? Template sẽ không hiện trong dropdown khi tạo campaign." [Hủy] [Tắt]. On confirm → row grays out, button changes to [Bật].
- [Bật] on inactive row → no confirm → row returns to normal, button changes to [Tắt]
- "Dùng" column value → click → Popover showing "Campaign sử dụng: Nhắc nạp tiền, Welcome eSIM"

Page: Template Editor (/templates/new or /templates/:id):

Sub-header: "← Template" → /templates | [Tên template *] input | toggle Active/Inactive | [Lưu Template]
[Mô tả] input below

Progress: Push ● Zalo ● SMS ○ USSD ○ Banner ○ Email ○

Tab bar: [Push ●] [Zalo OA ●] [SMS ○] [USSD ○] [Banner ○] [Email ○] [+ Kênh]
INTERACTION: tabs work same as Campaign Builder Message Matrix. Each tab has compose + preview 2-column layout.

Tab Push compose area:
  THAM SỐ ĐỘNG chips: {{ten_kh}} {{loai_sim}} {{so_dien_thoai}} {{so_du}} {{data_con_lai}} {{ngay_het_han}}
  Click chip → inserts into active textarea.
  Image upload + Title * + Body *
  Preview: phone notification mockup (same as Campaign Builder)
  SAMPLE PARAMS inputs + [↻ Render lại]

[Lưu Template]:
  - Validates: name required → if empty, show inline error "Tên template không được để trống"
  - If any channel tab has content → saves, shows toast "Đã lưu template ✓" → navigate to /templates
  - If all tabs empty (new template) → show warning toast "Template chưa có nội dung kênh nào. Lưu nháp?"
```

---

### SCREEN 5 — /triggers

```
Page: Trigger Management

Header: "TRIGGER MANAGEMENT" + [+ Thêm Trigger] → opens Add Modal

Search + Trạng thái filter [Tất cả ▾]

3 collapsible groups (all expanded by default):
INTERACTION: click group header → toggles expand/collapse

▼ REALTIME (2)
  SIM_ACTIVATED | Kích hoạt SIM | BSS | ● Active | [Sửa][Tắt]
  LOC_TRAVEL_PROV | Đến tỉnh du lịch | OCS | ● Active | [Sửa][Tắt]

▼ NEAR REALTIME (1)
  NO_APP_INSTALL_24H | Chưa cài app sau 24h | CRM | ● Active | [Sửa][Tắt]

▼ OFFLINE / BATCH (3)
  LOW_DATA_BALANCE | Data còn dưới 100MB | OCS | ● Active | [Sửa][Tắt]
  USAGE_DROP_40PCT | Usage giảm >40% | OCS | ◐ Testing (blue) | [Sửa][Bật]
  SIM_NO_TXN_30D | SIM ngừng GD >30 ngày | BSS | ○ Paused (orange) | [Sửa][Bật]

INTERACTIONS:
- [Tắt] → ConfirmDialog: "Tắt trigger SIM_ACTIVATED? Campaign đang dùng: Welcome eSIM Q2/2026 sẽ bị ảnh hưởng." [Hủy] [Tắt]. On confirm → row shows ○ Inactive, button becomes [Bật].
- [Bật] → no confirm → status back to Active, button becomes [Tắt]
- [Sửa] → opens Edit Modal (pre-filled with row data)
- [+ Thêm Trigger] → opens Add Modal

ADD / EDIT TRIGGER MODAL:
Fields:
  - Trigger code * (uppercase only, underscore allowed): text input with validation
  - Loại * dropdown: [Realtime ▾] options: Realtime / Near Realtime / Offline
  - Tên trigger * text input
  - Event source dropdown: [BSS ▾] options: BSS / OCS / CRM / Other
  - Trạng thái toggle: ● Active ○ Inactive
  - Mô tả: textarea (optional)
  - Payload table: rows of [param_name input] [source dropdown] [✕ delete]. [+ Thêm tham số] adds row.

INTERACTION:
  - [Hủy] → closes modal
  - [Lưu]:
    - Validates required fields → inline errors if empty
    - If Trigger code already exists → inline error "Mã trigger đã tồn tại"
    - On success → closes modal + toast "Đã lưu trigger ✓" + new row appears in correct group table
```

---

### SCREEN 6 — /blacklist

```
Page: Blacklist Management

Header: "BLACKLIST"

Filter row: Campaign [Tất cả ▾] | Kênh [Tất cả ▾] | [Lọc] button
INTERACTION: [Lọc] filters table rows.

Table: Số điện thoại | Campaign | Kênh | Hành động
4 rows:
  0987 xxx 001 | Chào mừng SIM | Zalo OA | [Xóa]
  0912 xxx 002 | Hết data | SMS | [Xóa]
  0965 xxx 003 | Hết data | Zalo OA | [Xóa]
  0976 xxx 004 | Nhắc nạp tiền | Push | [Xóa]

Pagination: < 1 2 3 > (decorative)

Bottom buttons: [+ Thêm thủ công] | [📥 Upload danh sách]

INTERACTIONS:
- [Xóa] → ConfirmDialog "Xóa 0987 xxx 001 khỏi blacklist campaign Chào mừng SIM kênh Zalo OA?" [Hủy] [Xóa]. On confirm → row removed, toast "Đã xóa khỏi blacklist".

- [+ Thêm thủ công] → opens Modal "Thêm vào Blacklist":
  Fields: Số điện thoại * (validate: 10 digits starting with 0) | Campaign * dropdown (options: 4 campaigns) | Kênh * dropdown (Push / Zalo OA / SMS / USSD / Banner / Email)
  [Hủy] closes. [Thêm]:
    - Validates fields → inline errors
    - On success → closes modal + new row added to table + toast "Đã thêm vào blacklist ✓"

- [📥 Upload danh sách] → opens Modal "Upload Blacklist":
  Fields: Campaign * dropdown | Kênh * dropdown
  File upload zone: dashed border, "📄 Kéo thả file vào đây hoặc [Chọn file]". Format: CSV, 1 cột so_dien_thoai. [📥 Tải file mẫu] → toast "Đang tải file mẫu..."
  INTERACTION: click [Chọn file] → simulate file selected → show preview result:
    ✅ 245 số hợp lệ | ⚠ 3 số sai định dạng (bỏ qua)
  [Hủy] closes. [Xác nhận Upload] → closes modal + toast "Đã upload 245 số vào blacklist ✓"
```

---

### SCREEN 7 — /customers

```
Page: Customer List

Header: "KHÁCH HÀNG"

Search: [🔍 Tìm số điện thoại...] — filters table client-side

Filter row: Trạng thái SIM [Tất cả ▾] | Cài app [Tất cả ▾] | DNC [Tất cả ▾]

Table: Số điện thoại | Loại SIM | Trạng thái | Cài app | Hành động
  0987 xxx 001 | eSIM | ● Active | Có | [Xem 360]
  0912 xxx 002 | SIM vật lý | ● Active | Không | [Xem 360]
  0965 xxx 003 | eSIM | ○ Inactive | Có | [Xem 360]
  0976 xxx 004 | SIM vật lý | ● Active | Không | [Xem 360]

INTERACTION:
- [Xem 360] on any row → navigate to /customers/:phone/360 (show Customer 360 with that row's data)
- Trạng thái SIM filter → Active = shows rows 1,2,4; Inactive = shows row 3
- Cài app filter → Có = rows 1,3; Không = rows 2,4
```

---

### SCREEN 8 — /customers/:phone/360

```
Page: Customer 360

Sub-header: "← Danh sách khách hàng" → navigate to /customers | Title: "CUSTOMER 360 — 0987 xxx 001"

Two-column layout (left 40%, right 60%):

LEFT — "THÔNG TIN & SỬ DỤNG":
Profile card:
  Loại SIM: eSIM · ● Active
  Gói: D50 · hết 18/05/2026
  Ngày KH: 01/04/2026
  Cài app: Có · Đăng nhập: Có

Usage stats:
  Data còn: 1.2 GB (show a small progress bar: 1.2/50 GB)
  Hôm nay: 320 MB
  Số dư: 45,000 VND
  Nạp tiền: 3 lần (tháng này)
  Sinh nhật: 15/08

RIGHT — "TRẠNG THÁI KÊNH & LỊCH SỬ":

Channel status table:
  Kênh | Trạng thái | Cập nhật
  Zalo | ⛔ Blocked (red chip) | 05/05 09:32
  SMS  | ✅ Active (green chip) | —
  USSD | ✅ Active | —
  Push | ✅ Active | —
Note: "ⓘ Tự cập nhật từ phản hồi Gateway — không chỉnh sửa thủ công"

Throttling box:
  Tin hôm nay: 2/3 (with mini progress bar)
  Cooldown: Không · DNC: Không

Message History list (most recent first, show last 3):
  10/05 · Chào mừng SIM | Zalo · ✓ Delivered (green)
  08/05 · Nạp thẻ lỗi | SMS · ✓ Delivered (green)
  07/05 · Hết data | Zalo · ✕ Blocked (red) → SMS · ✓ Delivered (green) — shown as 2 indented sub-lines

[Xem đầy đủ lịch sử →] button → opens Drawer from right side showing extended list (10 rows, same format). Drawer has [✕] close button.
```

---

### SCREEN 9 — /report

```
Page: Report / Analytics

Header: "ANALYTICS & REPORT" + [Xuất Excel] button → toast "Đang xuất file... Sẽ tải xuống sau vài giây"

Filter bar:
  [7 ngày ▾] dropdown: Hôm nay / 7 ngày / 30 ngày / Tháng này / Tùy chỉnh
  [Campaign: Tất cả ▾] dropdown: 4 campaigns + Tất cả
  [Segment: Tất cả ▾] dropdown: 4 segments + Tất cả
  [So sánh: ON ▾] toggle: ON/OFF. When ON → charts show a second "previous period" data series in dashed line.
  [Kênh: Tất cả ▾]

Tab navigation: [Delivery] [Engagement] [Campaign Comparison] [Segment] [Funnel] [Spam]
INTERACTION: click tab → show that tab's content. Default = Delivery.

--- TAB: Delivery ---
4 KPI cards: Total Sent 124,500 | Delivered 119,940 (96.3%) | Failed 4,560 (3.7% ⚠) | Delivery Rate 96.3%

Area chart "SENT vs DELIVERED" (full width, use recharts AreaChart or a static SVG visual):
  Two series: Sent (blue line), Delivered (green area), Failed (red area). X-axis: 7 days.

Two charts side by side:
  Left: "DELIVERY RATE TREND" — line chart %, dashed red line at 95% target
  Right: "FAILED DELIVERY TREND" — stacked bar, 4 categories with legend:
    ■ Gateway error (red) | ░ User blocked (orange) | ▒ DNC/Blacklist (yellow) | ▓ Network timeout (gray)

--- TAB: Engagement ---
4 KPI cards: Open Rate 38.4% ↑2.1% | CTR 12.1% ↑0.8% | Conversion 8.3% ↑0.4% | Best Channel: Push 97.7%

Horizontal bar chart "CHANNEL PERFORMANCE":
  Push: Open 52% CTR 18% (longest bars)
  Zalo OA: Open 44% CTR 15%
  SMS: Open 22% CTR 8%
  USSD: Open 15% CTR 5%
  Banner: Open 9% CTR 3%
  Email: Open 21% CTR 9%

Engagement Trend line chart: 3 lines (Open Rate, CTR, Conversion) over 7 days.

--- TAB: Campaign Comparison ---
[+ Thêm campaign để so sánh ▾] → adds campaign chip.
Horizontal bar chart comparing up to 4 campaigns by Delivery Rate and Conversion Rate.
Table: Campaign | Sent | Delivered | Open | Convert | Cost SMS. 4 rows.

--- TAB: Segment ---
Table: Segment | Reach | Delivered | Open | Conversion. 4 rows.
Bar chart comparing segments.
Device breakdown: Android 62% | iOS 28% | Khác 10% (horizontal bars).

--- TAB: Funnel ---
Funnel chart (5 steps, decreasing widths):
  Audience đủ điều kiện → Messages Sent → Delivered → Opened → Converted
  Show drop-off % between each step.
Drop-off analysis card at bottom: "Điểm drop lớn nhất: Delivered → Opened (−47.3%) → Gợi ý: Thử A/B test tiêu đề Push"

--- TAB: Spam ---
Line chart: Opt-out & Blacklist trend over 7 days (spike visible on day 5).
Two panels:
  Left: Frequency metrics table (Avg tin/user/ngày: 1.8, tuần: 4.2, Max: 5, User nhận >3: 8.4%) + histogram
  Right: Spam Risk Score card: "● LOW RISK (24/100)" with progress bar. Risk factors listed.
```

---

## GLOBAL COMPONENTS

```
Implement these shared components used across all screens:

1. CONFIRM DIALOG (modal overlay):
   - Backdrop overlay (bg-black/50)
   - White modal, rounded-xl, shadow-xl, max-w-sm, centered
   - Title (bold), body text, two buttons: [Hủy] (outline) + action button (color varies)
   - Press Escape or click backdrop → closes (same as [Hủy])

2. TOAST NOTIFICATIONS:
   - Fixed bottom-right corner, stack vertically
   - Types: success (green left border), error (red), warning (orange), info (blue)
   - Auto-dismiss after 3 seconds
   - Show for: save actions, status changes, confirmations, errors

3. POPOVER:
   - Appears on hover or click of ⓘ / +N trigger chips
   - White bg, shadow, rounded, border
   - Closes on click outside

4. DROPDOWN MENUS:
   - Consistent style: white bg, shadow-lg, rounded-lg, border, py-1
   - Items: px-3 py-2, hover:bg-slate-50
   - Closes on selection or click outside

5. EMPTY STATE:
   - When search/filter returns 0 results: centered icon + "Không tìm thấy kết quả" + "Thử thay đổi điều kiện lọc"

6. STATUS CHIPS:
   - Active: bg-green-100 text-green-700 ● dot
   - Draft: bg-slate-100 text-slate-600 ● dot
   - Pending: bg-yellow-100 text-yellow-700 ⏳
   - Paused: bg-orange-100 text-orange-700 ● dot
   - Inactive: bg-slate-100 text-slate-400

7. COLLAPSIBLE SECTIONS:
   - [Thu gọn] / [Mở rộng] text button top-right of section header
   - Animate expand/collapse (transition)
   - Section header stays visible when collapsed (shows section title + summary)
```

---

## NAVIGATION MAP (complete routing)

```
/dashboard          → Screen 1: Dashboard
/campaigns          → Screen 2: Campaign List
/campaigns/new      → Screen 3: Campaign Builder (empty)
/campaigns/:id/edit → Screen 3: Campaign Builder (pre-filled)
/campaigns/:id/detail → Screen 2B: Campaign Detail View
/templates          → Screen 4A: Template List
/templates/new      → Screen 4B: Template Editor (empty)
/templates/:id      → Screen 4B: Template Editor (pre-filled)
/triggers           → Screen 5: Trigger Management
/blacklist          → Screen 6: Blacklist Management
/customers          → Screen 7: Customer List
/customers/:phone/360 → Screen 8: Customer 360
/report             → Screen 9: Report/Analytics
/settings           → Placeholder screen "Settings (coming soon)"
```
