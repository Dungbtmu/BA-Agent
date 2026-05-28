# Lovable Prompt — CVM System v9
> Cập nhật từ v8: Thêm Screen 2B Campaign Detail View (chỉ đọc); bổ sung card soạn nội dung + preview đầy đủ cho Zalo OA, Banner, Email trong S4; AND mode ẩn [+ Biến thể đối tượng] + confirm dialog khi chuyển OR→AND.

---

Build a high-fidelity wireframe web app for a CVM (Customer Value Management) system used by a virtual telecom operator. This is an internal tool for Marketing Admins and Campaign Operators (QTV).

---

## TECH STACK
- React + Tailwind CSS
- Fixed sidebar navigation layout
- Clean, modern B2B SaaS UI (similar to Braze, Klaviyo)
- Language: Vietnamese UI labels

---

## ROLES (assume full access for this prototype)
- Admin: full access including Trigger Management and System Settings
- QTV: create/edit campaigns, view reports

---

## GLOBAL LAYOUT
- Fixed left sidebar: Dashboard, Campaign, Template, Trigger, Blacklist, Customer, Report, Settings
- Top navbar: logo left, notification bell + avatar right

---

## SCREENS

---

### SCREEN 1: Dashboard

#### ROW 1 — KPI Cards (8 cards, arranged 4+4)

Render two rows of 4 cards each. Each card auto-refreshes every 60 seconds.

**Row 1 — 4 cards:**

Card 1 — Active Campaigns:
- Value: 12
- Delta: ↑2 vs hôm qua
- Sparkline: 7-day line chart (small, inline)
- Hover sparkline → tooltip showing date + value

Card 2 — Trigger Fired Today:
- Value: 3,842
- Delta: ↑18% vs hôm qua
- Sparkline: 7-day line chart

Card 3 — Messages Sent Today:
- Value: 48,320
- No delta label
- Sparkline: 7-day line chart

Card 4 — Delivery Rate:
- Value: 96.4%
- Status: SLA OK (green badge)
- Sparkline: 7-day line chart

**Row 2 — 4 cards:**

Card 5 — Failed Messages:
- Value: 1,760
- Delta: ⚠ ↑3% vs hôm qua
- Trend direction: bad (upward failure)
- Card border: red (`border-red-500`) when value exceeds threshold
- Sparkline: 7-day line chart, red fill

Card 6 — Conversion Rate:
- Value: 8.3%
- Delta: ↑0.4% vs tuần
- Sparkline: 7-day line chart

Card 7 — Realtime SLA:
- Value: < 2.1s avg
- Status: ✅ Target < 3s (green)
- Sparkline: 7-day line chart

Card 8 — Blacklist Blocked Today:
- Value: 4,210
- Breakdown (two sub-lines): BSS DNC: 3,840 · CVM BL: 370
- Sparkline: 7-day line chart

**Card rules:**
- Auto-refresh every 60s; show last-updated timestamp at card footer
- Red border (`border-red-500`) when metric crosses a configured bad threshold (e.g. Failed Messages rising trend)
- Click any card → drill-down view (modal or linked screen)

---

#### ROW 2 — System Health (4 panels, equal-width grid)

**Panel 1 — Trigger/Event Volume:**
- Area/line chart, 24h timeline (x-axis: hours 00–23)
- 3 lines: Realtime / Near Realtime / Offline
- Legend below chart with color keys

**Panel 2 — Processing Latency:**
- Line chart, 24h timeline
- 3 lines: p50 / p95 / p99
- Horizontal dashed reference line: SLA target 3s (labeled "SLA 3s")
- Y-axis in seconds

**Panel 3 — Queue & Backlog:**
- Label: "Processing Queue"
- Count: 342 events
- Progress bar: 68% capacity (amber at 60%+, red at 80%+)
- Three sub-metrics below bar:
  - Pending blackout: 1,240
  - Scheduled future: 8,900
  - Oldest pending: 02:15 ago
- Note: "flush lúc 08:00"
- Button: [Xem queue] — links to queue detail

**Panel 4 — Top System Errors:**
- List of 3 most recent error types:
  - ⛔ GATEWAY_ZALO_TIMEOUT · 128 lần · Zalo OA API timeout >5s · Lần gần nhất: 09:32
  - ⚠ SMS_SEGMENT_EXCEED · 45 lần · Tin nhắn vượt 2 segment
  - ⚠ DEDUP_COLLISION · 12 lần · Event ID trùng bị bỏ qua
- Link at bottom: [Xem tất cả →]
- Error severity: ⛔ = red, ⚠ = amber

---

#### ROW 3 — Campaign Monitoring (2 side-by-side tables + 1 realtime timeline)

**Table 1 — Top Active Campaigns:**
Columns: Campaign | Sent | Rate | Trend
Sample rows:
- Nhắc nạp tiền | 18,200 | 96.5% | ▂▃▄▅▆▇ (sparkline mini)
- Chào mừng SIM | 12,400 | 95.1% | ▁▂▃▄▅▆

**Table 2 — Trigger Fired Nhiều Nhất Hôm Nay:**
Columns: Trigger | Fired | Match Rate
Sample rows:
- LOW_DATA_BALANCE | 1,842 | 68%
- SIM_ACTIVATED | 934 | 91%

**Timeline — Recent Trigger Events (realtime stream):**
- Live feed of trigger events, newest at top
- Each row: timestamp · trigger code · masked phone number · outcome badge
- Sample rows:
  - 09:44:52 LOW_DATA_BALANCE · 0987 xxx 001 → Matched · Push sent
  - 09:44:38 SIM_ACTIVATED · 0912 xxx 002 → Matched · Zalo sent
  - 09:44:21 NO_APP_INSTALL_24H · 0965 xxx 003 → No match
  - 09:44:05 TOP_UP_FAIL · 0976 xxx 004 → Blacklist blocked
- Outcome badges: "Matched" = green · "No match" = gray · "Blacklist blocked" = red
- Toggle button: [Live ●] / [Pause] — pausing stops the feed; click again to resume

---

#### ROW 4 — User Journey Funnel

- Campaign selector dropdown at top-right of panel
- Horizontal funnel bars (full width, stepped narrowing)
- Funnel steps with absolute count and percentage:
  - SIM Activated: 32,100 (100%)
  - Message Delivered: 29,530 (92%) · drop label: "↓ 8%"
  - Message Opened: 12,200 (38%) · drop label: "↓ 61%"
  - App Installed: 9,400 (29%) · drop label: "↓ 23%"
  - Package Purchased: 5,800 (18%) · drop label: "↓ 20%"
- Footer: Conversion rate: 18.1% · vs tuần trước: ↑1.2%
- Color: funnel bars in blue gradient; drop labels in red

---

#### ROW 5 — Trigger Analytics (2 side-by-side panels)

**Left panel — Top Triggers 7 ngày:**
Table with inline bar: Trigger | Fired | Match Rate | Bar
Sample rows:
- LOW_DATA | 12,400 | 68% | ████████░░
- SIM_ACTIVATED | 6,100 | 91% | █████████░

**Right panel — Anomaly Detection:**
- ⚠ Alert row: LOW_DATA_BALANCE — Volume tăng đột biến +340%
  - Detail: avg 3,600/ngày · Hôm nay: 12,400 · 09:00–09:45
  - Note: "→ Có thể do batch job OCS re-run"
  - Link: [Xem chi tiết →]
- ✅ Row: "Các trigger còn lại: bình thường"

**Trigger Heatmap — 7 ngày × 24 giờ (full-width below the two panels):**
- Rows: Mon–Sun (y-axis)
- Columns: 00–23 (x-axis)
- Cell density using 5 levels: ░ (thấp) ▒ (trung bình) ▓ (cao) █ (rất cao) ██ (đột biến)
- Color scale: white → light amber → amber → deep amber → red for anomaly cells
- Hover any cell → tooltip: "Thứ X · Giờ Y: Z triggers"
- Anomaly detection: cells where volume > 200% of same-hour 7-day baseline are highlighted with red border + ⚠ icon overlay
- Legend at bottom-right: 5 density squares + labels

---

### SCREEN 2: Campaign List

**Header:** "Campaign" + primary button [+ Tạo Campaign]

**Search bar (full width):** placeholder "Tìm theo tên campaign, mã hoặc trigger code..." — search is realtime (filter-as-you-type)

**Filter tags (multi-select inline chips, below search):**
- [Tất cả ×] [Active] [Draft] [Pending] [Paused]
- [Realtime] [Offline] [Có SMS] [Chưa đủ nội dung]
- Click a tag to select/deselect. Active selection = filled chip. Multiple tags can be active simultaneously.
- When "Tất cả" is active, all other tags are deselected. Selecting any other tag deactivates "Tất cả".

**Table columns:** TÊN / MÃ CAMPAIGN | TRIGGER | HIỆU LỰC | TRẠNG THÁI | HÀNH ĐỘNG

**Column: TÊN / MÃ CAMPAIGN**
- Two-line cell: campaign name on line 1 (normal weight), campaign code on line 2 (smaller, text-gray-500 monospace)

**Column: TRIGGER**
- Show maximum 2 trigger chips inline: `[CODE]` style chips, amber background
- If campaign has more than 2 triggers: show `+N ⓘ` chip (outlined)
  - Hover or click `+N ⓘ` → popover listing all triggers: code + name + source + kiểu chạy
- Hover any individual chip → tooltip: trigger name + source + kiểu chạy
- No Logic column (removed in v8)

**Status badges (pill shape):**
- Active = filled green pill (`#22C55E`) → actions: [Xem] [Dừng]
- Draft = filled gray pill (`#94A3B8`) → actions: [Xem] [Sửa]
- Pending = filled yellow pill (`#EAB308`) → actions: [Xem]
- Paused = filled orange pill (`#F97316`) → actions: [Xem] [Bật]

**Action: [Dừng]** — Kill Switch:
- Click → confirm dialog: "Dừng campaign sẽ hủy tất cả tin nhắn đang trong queue. Bạn có chắc chắn?"
- Confirm → campaign status changes to Paused immediately

**Action: [Bật]** (on Paused) — Reactivate:
- No approval required if campaign content has not changed since last activation
- If content changed since last approval → prompt QTV that re-approval is needed

**Sample rows:**
- Welcome eSIM Q2/2026 / CVM-WELCOME-ESIM-Q2-2026 | [SIM_ACT] [NO_APP] +1 ⓘ | 01/04–30/06 | ● Draft | [Xem][Sửa]
- Nhắc nạp tiền / CVM-REMIND-TOPUP-05-2026 | [TOP_UP] | 01/05–31/05 | ● Active | [Xem][Dừng]
- Hết hạn gói data / CVM-DATA-EXPIRE-05-2026 | [PKG_EXP] | 01/05–31/05 | ● Paused | [Xem][Bật]
- Chào mừng du lịch / CVM-TRAVEL-PROMO-Q2 | [LOC_TRV] | 15/05–30/06 | ⏳ Pending | [Xem]

**Pagination:** < 1 2 3 > · [20/trang ▾] select

---

### SCREEN 2B: Campaign Detail View

Opened when user clicks [Xem] from Campaign List. Read-only view of all campaign configuration.

**Header:**
- Back link: "← Campaign" → navigates back to Campaign List
- Campaign name + code: "Welcome eSIM Q2/2026 · CVM-WELCOME-ESIM-Q2-2026"
- Status badge (matches campaign status color — e.g. ⏳ Pending = yellow)
- Buttons (right, vary by status):
  - All statuses: [Đóng] → navigates back to Campaign List
  - Draft only: [Sửa] → navigates to Campaign Builder (edit mode)
  - Active: [Dừng] (danger) → same Kill Switch confirm dialog as Campaign List
  - Paused: [Bật] → same reactivate behavior as Campaign List

**Content (single scrollable column, sections with labeled dividers — all read-only, no input fields):**

**Section 1 — Thông tin Campaign:**
Two-column label-value grid:
- Tên campaign: Welcome eSIM Q2/2026
- Mã kịch bản: CVM-WELCOME-ESIM-Q2-2026
- Mục tiêu: Onboard eSIM, nhắc cài app
- Thời gian hiệu lực: 01/04/2026 – 30/06/2026
- Người tạo: QTV Marketing
- Ngày tạo: 10/05/2026 14:32
- Ngày gửi duyệt: 12/05/2026 09:15

**Section 2 — Trigger & Logic:**
- Chế độ: Advanced · Logic: OR
- Trigger chips (read-only): ★ P1 [SIM_ACTIVATED · Kích hoạt SIM] · P2 [NO_APP_INSTALL_24H · Chưa cài app 24h]
- Ưu tiên: Chỉ gửi trigger P1
- Ước tính: ~6,800 KH × 2 kênh = ~13,600 tin (blue info box)

**Section 3 — Audience:**
- Phân khúc: tag cards [Gen Z User (18–25)] [Sắp hết data] — no [×] remove buttons
- Logic: OR
- Điều kiện lọc: Loại thiết bị = Android
- Reach: ~6,800 KH

**Section 4 — Message Matrix:**
- Thời gian gửi: Gửi ngay sau khi trigger kích hoạt
- Channel tabs: [Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○]
  - Click tab → show message content for that channel (all 6 tabs clickable)
  - Default active tab: first tab with content (Push)
  - Each tab shows read-only message cards per trigger (no compose area, no preview — just label-value display):
    - Push tab sample:
      - Card: T1 · SIM_ACTIVATED | Title: "Chào mừng bạn!" | Body: "SIM eSIM đã kích hoạt thành công. Khám phá MyViettel."
      - Card: T2 · NO_APP_INSTALL_24H | Title: "Cài MyViettel nhận ưu đãi" | Body: "Bạn vẫn chưa cài app MyViettel..."
    - Tabs without content: show empty state "Chưa cấu hình nội dung cho kênh này"

**Section 5 — Channel Strategy:**
- Đặt lịch theo kênh: Không (tất cả theo thời gian gửi message)

**Section 6 — An toàn:**
- Blackout: Bật · 22:00 – 08:00 · Hủy luôn
- DNC toàn hệ thống: ✓ Bật
- Blacklist campaign: BL_ESIM_Q2_2026 · 320 KH — [Xem] button → opens same file preview modal (read-only in this context)
- Whitelist: Không dùng
- Reach cuối cùng: ~6,480 KH (green bold)

---

### SCREEN 3: Campaign Builder

#### Overall Layout

**2-column layout. NO Section 7. Min-width 1440px.**

```
[FIXED HEADER — full width]
┌──────────────────────────────────────┬─────────────────────────────┐
│ LEFT COLUMN (~60%, scrollable)       │ RIGHT COLUMN (~40%, sticky) │
│                                      │                             │
│  Section 1: Thông tin Campaign       │  Campaign Summary mini      │
│  Section 2: Trigger & Logic          │  ──────────────────────     │
│  Section 4: Message Matrix           │  Section 3: Audience        │
│    └─ internal 2-col: Soạn | Preview │  ──────────────────────     │
│  Section 6: An toàn                  │  Section 5: Channel Strategy│
└──────────────────────────────────────┴─────────────────────────────┘
```

**Left column rules:**
- Scrollable independently
- Contains S1, S2, S4, S6 in order
- S4 Message Matrix internally uses its own 2-column layout (Soạn | Preview) — see Section 4 spec

**Right column rules:**
- Sticky — does not scroll with left column
- Top: Campaign Summary mini (always visible)
- Middle: Section 3 — Audience / Phân khúc (full interactive, not read-only)
- Bottom: Section 5 — Channel Strategy accordion
- Right column sections are fully interactive (user can add/remove segments, toggle schedules from here)

**Campaign Summary mini** (top of right column, above S3):
- Issue badge: "⚠ 2 issue cần giải quyết" (red) or "✅ Sẵn sàng gửi duyệt" (green)
- Reach cuối: "~6,480 KH (sau DNC + BL)"
- Chi phí SMS ước tính: "~720,000 VND"
- Nội dung: "Push ●● Zalo ●● SMS ●○" (completion dots)
- Updates realtime when any section changes

---

#### Fixed Header

- Back link (left): "← Campaign"
- Center: campaign name (editable inline) · campaign code (text-gray-500 monospace, smaller)
- Status badge: ● Draft
- Buttons (right):
  - [Lưu Nháp] — secondary button
  - [Gửi duyệt →] — primary button
    - Show issue count badge when validation has blocking issues: "⚠ 2 issue"
    - Disable button when blocking issues exist
    - Hover disabled button → tooltip listing each issue, e.g. "Chưa cấu hình nội dung SMS · Chưa chọn tệp whitelist"
    - Click an issue in tooltip → scroll to the relevant section

---

#### SECTION 1: Thông tin Campaign

Card with yellow/amber left border, labeled **"1. Thông tin Campaign"**

Fields (vertical stack):
- **Tên campaign \*** — text input; example: "Welcome eSIM Q2/2026"
- **Mã kịch bản** — text input (auto-generated or editable); example: "CVM-WELCOME-ESIM-Q2-2026"
- **Mục tiêu** — text input; example: "Onboard eSIM, nhắc cài app"
- **Thời gian hiệu lực** — date range picker; example: 01/04/2026 – 30/06/2026
- **Người tạo** — read-only text; example: "QTV Marketing"

---

#### SECTION 2: Trigger & Logic

Card with yellow/amber left border, labeled **"2. Trigger & Logic"**

**Default state (no triggers selected):**
- Show only button: [+ Chọn trigger ▾]
- Note below: "Chưa có trigger. Nhấn để bắt đầu."
- Do NOT pre-fill any trigger cards by default (v8 change from v7)

**Mode toggle (segmented control):**
- ○ Basic (1 trigger) · ● Advanced (nhiều trigger + logic)
- Basic mode: allow only 1 trigger; hide Logic OR/AND block and conflict resolution block
- Advanced mode: allow multiple triggers; show Logic block

**After triggers are selected — trigger chip list (vertical, compact):**

Each selected trigger renders as a single compact chip row (not a full card):
- Format: `[★/P-number] [TRIGGER_CODE · Tên trigger đầy đủ] [≡] [✕]`
- ★ gold badge marks P1 (primary trigger)
- `[≡]` drag handle on right side — drag to reorder priority; the ★ P1 marker follows the top position automatically
- `[✕]` remove button; if P1 is removed, the next trigger automatically becomes P1

Hover any chip → tooltip panel showing:
- Source (BSS / OCS / CRM)
- Kiểu chạy (Realtime / Near Realtime / Offline)
- Payload params list (e.g. ten_kh, loai_sim, ngay_kich_hoat)

**Sample after selection (Advanced mode):**
- ★ P1 [SIM_ACTIVATED · Kích hoạt SIM thành công] [≡] [✕]
- P2 [NO_APP_INSTALL_24H · Chưa cài app sau 24h] [≡] [✕]

Button below list: [+ Chọn trigger ▾] — opens searchable dropdown

**Logic block (Advanced mode only — shown below trigger list):**

Radio group:
- ● OR — gửi message của trigger nào match
- ○ AND — KH phải thỏa tất cả trigger

Under OR selection, show human-readable explanation:
- "Nếu KH kích hoạt SIM → gửi Message Welcome"
- "Nếu KH chưa cài app 24h → gửi Message Chưa cài app"

Under OR + multi-trigger: show conflict resolution sub-block:
- Radio (default selected): ● Chỉ gửi trigger ưu tiên cao nhất (★ P1)
- Radio: ○ Gửi tất cả trigger đã match

Under AND selection, show explanation:
- "KH phải thỏa đồng thời tất cả trigger; dùng chung 1 message cho mỗi kênh"

**Switching OR → AND (if Audience Variants already exist in any channel tab):**
- Click AND radio → open confirm dialog before switching:
  - Title: "Chuyển sang AND mode?"
  - Body: "Chuyển sang AND mode sẽ xóa toàn bộ Biến thể đối tượng đã tạo ở tất cả tab kênh. Hành động này không thể hoàn tác."
  - Buttons: [Hủy] (cancel — stay on OR) | [Xác nhận] (proceed — delete all variants, switch to AND)
- If no variants exist: switch immediately without dialog.
- In AND mode: button [+ Biến thể đối tượng] is hidden in ALL trigger cards across ALL channel tabs.

**Estimate block (conditional — only render when BOTH triggers AND audience are configured):**
- "Ước tính tin nhắn: ~6,800 KH × 2 kênh = ~13,600 tin"
- "Trong đó SMS: ~1,200 KH × 2 segment = ~2,400 SMS"
- "Chi phí SMS: ~720,000 VND"
- Note: "ⓘ Tính dựa trên audience hiện tại và kênh đã cấu hình"

---

#### SECTION 3: Audience / Phân khúc

> **Layout note:** Section 3 renders in the RIGHT sticky column, not the left scroll column.

Card with yellow/amber left border, labeled **"3. Audience / Phân khúc"**

**Data source line (read-only):**
- "Nguồn phân khúc: Customer 360 / Team Data / BSS / OCS"
- "Cập nhật: 13/05/2026 08:00"

**Segment selector — dropdown search (v8 change: NOT a checkbox list):**
- Button: [🔍 Tìm phân khúc... ▾]
- Opens dropdown with search input; type to filter segment names
- Click a segment → add as tag card below (do not keep a visible checkbox list)

**Selected segment tag cards (horizontal wrap):**
Each tag card shows:
- Segment name · số KH · [×] remove button
- Example: "Gen Z User (18–25) · 18,450 KH [×]" · "Sắp hết data · 8,920 KH [×]"

**Segment logic radio:**
- ● Bất kỳ phân khúc nào (OR)
- ○ Tất cả phân khúc (AND)

**Điều kiện lọc thêm — collapsible accordion (optional):**
- Label: "Điều kiện lọc thêm (optional) [Mở rộng]"
- When expanded: render filter rows
  - Each row: [Loại thiết bị ▾] [= ▾] [Android ▾] [×]
  - Button: [+ Thêm điều kiện]
  - ALL fields in filter rows are dropdown-select only; no free-text input
  - Note below: "ⓘ Giá trị được cấu hình sẵn — chỉ chọn, không nhập tự do"

**Estimated reach:**
- "Ước tính reach: ~6,800 KH" + [↻ Tính lại] button
- Recalculates in realtime when segment selection or filter conditions change

**Note at bottom:**
- "CVM không tạo segment master — segment do Team Data/BSS/OCS cung cấp."

---

#### SECTION 4: Message Matrix

Card with yellow/amber left border, labeled **"4. Message Matrix · Trigger × Kênh"**

---

**THỜI GIAN GỬI block (at the very top of Section 4, applies to the entire campaign, rendered once):**

Label: "Thời gian gửi"

Radio group (vertical):
- ● Gửi ngay sau khi trigger kích hoạt
- ○ Sau [__] [phút / giờ / ngày ▾] kể từ T
- ○ Vào lúc [__:__] ngày T + [__]

Note below: "ⓘ T = thời điểm trigger kích hoạt cho từng KH cụ thể"

Behavior definitions (show as collapsible tooltip/info block):
- "Gửi ngay": send immediately when trigger fires and subscriber passes filters
- "Sau X kể từ T": intentional delay from the moment the trigger fires for that subscriber
- "Vào lúc HH:MM ngày T+N": T+0 = date trigger fires; T+1 = next day; etc.

---

**TIẾN ĐỘ NỘI DUNG (progress summary — shown below THỜI GIAN GỬI, above tabs):**
- Render inline completion dots per channel:
  - Push ●● · Zalo ●● · SMS ●○ · Banner ○○ · Email ○○ · USSD ○○
  - ● = hoàn chỉnh · ○ = chưa cấu hình
- Legend: "● = hoàn chỉnh · ○ = chưa cấu hình"

---

**Channel tabs:**
[Push ●●] [Zalo OA ●●] [SMS ●○] [Banner ○○] [Email ○○] [USSD ○○] [+ Kênh]

- Completion badge per tab: ●● = all variants complete (green) · ●○ = partial (amber) · ○○ = none (gray)
- [+ Kênh] dropdown lists only channels NOT already present as tabs; channels already added are disabled in dropdown

---

**Tab Push — Trigger Message Cards (OR mode — one card per trigger):**

Each trigger message card uses a permanent 2-column layout (NO [Preview] button needed):

Card header (full width): status icon (✓/⚠/○) + trigger code · e.g. "✓ T1 · SIM_ACTIVATED"

LEFT sub-column (~55%) — Soạn nội dung:
- PARAMS chips row (trigger-specific payload):
  - Chips: [ten_kh] [loai_sim] [ngay_kich_hoat] [so_dien_thoai]
  - Blue pill chips; click → insert {{param_name}} at cursor in focused textarea
  - Invalid params (if trigger changes): red pill + tooltip
- Image upload field (channel-dependent, positioned before Title):
  - Push: upload optional · label "1:1 khuyến nghị" · empty = dashed 🖼 placeholder · filled = thumbnail + [Xóa][Đổi]
  - Zalo OA: upload optional · label "Tự do" · same states
  - Banner: upload REQUIRED · label "16:9 bắt buộc" · blocking validation if empty
  - Email: upload optional · label "Banner ngang"
  - SMS: show text label "(Không hỗ trợ hình ảnh)" — no upload control
  - USSD: show text label "(Không hỗ trợ hình ảnh)" — no upload control
- Template dropdown: select from library or compose inline
- Title input * (character counter 0/65) — Push/Zalo/Banner/Email only
- Body textarea * (character counter varies per channel)
- [+ Biến thể đối tượng] button at bottom

RIGHT sub-column (~45%) — XEM TRƯỚC (live, always visible):
- Channel mockup rendered in realistic visual format (updates in realtime on every keystroke)
- Image appears in mockup as soon as uploaded in left column
- SAMPLE PARAMS section below mockup: editable inputs for each payload param
  - e.g. ten_kh = [Nguyễn Văn A], loai_sim = [eSIM], ngay_kich_hoat = [01/04/2026]
  - Changing any sample param re-renders the preview immediately (no button needed)

Channel-specific preview format in right sub-column:
- Push: OS notification bar — app icon (left) + 1:1 thumbnail (right, if uploaded) + Title + Body
- Zalo OA: chat bubble — "Viettel Telecom" brand header + image above bubble (if uploaded) + message content
- SMS: plain phone SMS bubble — text only, no image, character counter X/160 shown below
- USSD: monospace black terminal — plain text + menu-style layout, no image, dấu warning shown
- Banner: card — 16:9 image full width at top + Title + Body below + CTA button
- Email: email client view — subject bar + header image (if uploaded) + body content

**AND mode:** instead of N trigger cards, render 1 card labeled "Message chung" per tab; PARAMS chips combine all trigger payloads

---

**Audience Variant (when [+ Biến thể đối tượng] is clicked):**

The trigger card expands into a grouped layout:
- Card header: trigger code + PARAMS chips (shown once at top, shared across all variants)
- Variants listed vertically below header:

**Biến thể 1 (fallback — mandatory):**
- Label: "Biến thể 1"
- Đối tượng dropdown: [Tất cả ▾] — "Tất cả" is only available for Biến thể 1
- No [×] remove button — Biến thể 1 cannot be deleted
- Uses same permanent 2-column layout (Soạn | Preview live) as the base trigger card
- Left: image upload (channel-dependent) · Template dropdown · Title · Body · PARAMS chips
- Right: channel mockup preview + sample params (always visible, no click needed)

**Biến thể 2+ (optional):**
- Label: "Biến thể 2"
- Đối tượng dropdown: [VIP ▾] — lists segments chosen in Section 3; excludes "Tất cả"
- [×] remove button — click → confirm dialog before deleting
- Uses same permanent 2-column layout (Soạn | Preview live) as the base trigger card
- Left: image upload (channel-dependent) · Template dropdown · Title · Body · PARAMS chips
- Right: channel mockup preview + sample params (always visible, no click needed)

Button below all variants: [+ Biến thể đối tượng]

Note at bottom of expanded card:
- "ⓘ KH match nhiều biến thể → áp dụng biến thể có thứ tự cao nhất (trái → phải); Biến thể 1 luôn là fallback, không thể xóa"

Completion badge logic: card is considered complete (●) when Biến thể 1 has content

---

**Note on preview behavior:**
- Preview is always visible in the right sub-column of each trigger card — no click required.
- Preview renders in realtime as the user types in Title, Body, or changes sample params.
- When an image is uploaded in the left sub-column, it appears immediately in the right sub-column mockup.
- The right sub-column preview format matches exactly the channel of the active tab (Push shows Push notification, SMS shows SMS bubble, etc.).

---

**Tab SMS — per trigger card:**
- PARAMS chips (trigger-specific)
- Textarea: plain text
- Character counter: X/160; counter turns red when > 160
- Badge: "Y SMS segment" where Y = ceil(chars/160)
- Inline cost panel: "Reach SMS: ~1,200 KH · Y segment = ceil(Y) SMS · ~VND"
- Exceeding 160 chars is allowed; do not block save

**Tab USSD — per trigger card:**
- Plain text textarea only (max 182 chars shown in counter)
- Warning: "⚠ Chỉ plain text, không hỗ trợ ảnh; kiểm tra ký tự đặc biệt"
- No image upload

---

**Tab Zalo OA — per trigger card (same 2-column Soạn | Preview layout):**

LEFT sub-column:
- PARAMS chips (trigger-specific payload) — same click-to-insert behavior
- Image upload: optional, free ratio. Empty = dashed 🖼 placeholder + [Tải lên][Chọn thư viện]. Filled = thumbnail + [Xóa][Đổi].
- Template dropdown
- Nội dung * textarea (counter 0/1000)
- [+ Biến thể đối tượng] button (hidden in AND mode)

RIGHT sub-column — XEM TRƯỚC (live):
- Zalo OA chat bubble mockup: "🟦 Viettel Telecom" brand header at top, optional image below header (if uploaded), message text content below image
- SAMPLE PARAMS inputs (editable, re-renders preview on change)

---

**Tab Banner — per trigger card (same 2-column Soạn | Preview layout):**

LEFT sub-column:
- PARAMS chips
- Image upload: **REQUIRED**, ratio 16:9. Show blocking validation warning "⚠ Chưa upload ảnh — bắt buộc" if empty. Filled = thumbnail + [Xóa][Đổi].
- Template dropdown
- Title * input (counter 0/65)
- Body * textarea (counter 0/120)
- CTA Label * input (e.g. "Khám phá ngay")
- CTA URL * input (validates URL format)
- [+ Biến thể đối tượng] button (hidden in AND mode)
- Blocking validation: if image is missing → contributes to issue count; [Gửi duyệt] disabled

RIGHT sub-column — XEM TRƯỚC (live):
- Banner card mockup: 16:9 image area full width at top (gray placeholder if not uploaded, actual image if uploaded), Title below, Body text, CTA button at bottom
- SAMPLE PARAMS inputs

---

**Tab Email — per trigger card (same 2-column Soạn | Preview layout):**

LEFT sub-column:
- PARAMS chips
- Header image upload: optional, banner ratio (landscape). Label "Banner ngang, optional". Empty = dashed 🖼. Filled = thumbnail + [Xóa][Đổi].
- Template dropdown
- Subject * input (counter 0/100)
- Body * textarea — plain text (label "Nội dung plain text")
- [+ Biến thể đối tượng] button (hidden in AND mode)

RIGHT sub-column — XEM TRƯỚC (live):
- Email client mockup: "From: Viettel Telecom" row, "Subject: [rendered subject]" row, header image below (if uploaded), body text below
- SAMPLE PARAMS inputs

---

#### SECTION 5: Channel Strategy

> **Layout note:** Section 5 renders in the RIGHT sticky column, below Section 3.

Card with yellow/amber left border, labeled **"5. Channel Strategy"**

**Important (v8 change): Remove all pipeline/fallback-order UI. No drag-and-drop channel ordering. No fallback wait-time configuration.**

Show only this note at top:
> "ⓘ Pipeline kênh (thứ tự fallback, thời gian chờ) được định nghĩa nội bộ bởi hệ thống, không cấu hình tại đây."

**Accordion: "Đặt lịch gửi theo kênh (optional)" [Mở rộng]**

When accordion is expanded, render one schedule row per channel that has a configured tab in Section 4. Do NOT show channels without content in S4.

For each channel (e.g. Push / Zalo OA / SMS):
- Channel name label
- Radio options:
  - ● Không đặt lịch riêng (theo thời gian gửi ở S4)
  - ○ Gửi vào [__:__] [Hằng ngày ▾]
- When "Gửi vào [__:__]" is selected: that channel sends only within the selected time window, regardless of when the trigger fires; this setting is independent of the send-time block in S4

---

#### SECTION 6: An toàn

Card with yellow/amber left border, labeled **"6. An toàn"**

---

**GROUP A — BLACKOUT:**

Label: "Giờ giới nghiêm (Blackout)"

Toggle: ● Bật · ○ Tắt

When enabled, show the following in order (vertical stack):

1. **Section label**: "Giờ giới nghiêm (Blackout)" — bold, above the inputs

2. **Time range row** — two separate time inputs side by side, connected by a dash:
   - Input 1: `22:00` + clock icon (⊙) on the right inside the input
   - Separator: ` — ` between the two inputs
   - Input 2: `08:00` + clock icon (⊙) on the right inside the input
   - Both inputs are individually editable (not a single text string)
   - Style: rounded border input, same width, clean minimal look

3. **Violation handling dropdown** — on its own line below the time inputs (NOT inline):
   - Default selected: "Hủy luôn"
   - Full width dropdown with chevron (∨) on right
   - Options (exactly 2):
     - Hủy luôn
     - Delay đến đầu khung giờ

---

**GROUP B — SUPPRESSION:**

**DNC toàn hệ thống:**
- Checkbox: ☑ Check DNC toàn hệ thống — checked and enabled by default
- If user unchecks → show confirm dialog: "Tắt DNC toàn hệ thống có thể vi phạm quy định. Bạn có chắc chắn?"

**Blacklist campaign:**
- Radio:
  - ○ Không dùng (default)
  - ● Dùng tệp riêng: [BL_ESIM_Q2_2026 · 320 KH ▾] + [Xem][Thay][Upload mới]
- When "Dùng tệp riêng" is selected but no file chosen → blocking validation; [Gửi duyệt] disabled

**Whitelist:**
- Radio:
  - ● Không dùng (default)
  - ○ Chỉ gửi cho whitelist: hides all whitelist controls until this option is selected
- When enabled but no file chosen → blocking validation; [Gửi duyệt] disabled

**Frequency Cap — NOT shown at campaign level (v8 removal):**
- Show note only: "ⓘ Giới hạn tần suất nhận tin được cấu hình tại System Settings → áp dụng cho toàn bộ campaign"
- Link: [Đến System Settings →]

**Reach summary (updated dynamically):**
- Formula display: "~6,480 KH = 6,800 → trừ DNC → trừ BL 320 → chưa dùng WL"
- Updates in realtime when DNC, BL, or WL changes

---

**Modal: Chọn tệp (Blacklist hoặc Whitelist — same modal, different title):**
- Title: "Chọn tệp blacklist campaign" or "Chọn tệp whitelist"
- Search input: "Tìm tên tệp..."
- Radio list: each row — tên tệp · số KH · ngày cập nhật
- Preview area:
  - "✅ X số hợp lệ · ⚠ Y số sai định dạng · Trùng: Z"
  - 2–3 masked phone sample rows
- Footer: [Hủy] · [Áp dụng]

---

**User flow note (as tooltip or info card below all sections):**
Steps: 1. Thông tin → 2. Trigger → 3. Audience → 4. Message Matrix → 5. Đặt lịch kênh → 6. An toàn → 7. Lưu nháp hoặc Gửi duyệt

---

### SCREEN 4: Template Management

#### 4A — Danh sách Template

Header: "TEMPLATE" + primary button [+ Tạo Template]

Filters:
- Search: [🔍 Tìm tên template...] (full width)
- Dropdown: Kênh (Tất cả / Push / Zalo OA / SMS / USSD / Banner / Email)
- Dropdown: Trạng thái (Tất cả / Active / Inactive)
- Filter Kênh → show only templates that have content for the selected channel

Table columns: Tên Template | Kênh hỗ trợ | Dùng | Hành động

Sample rows:
- Chào mừng SIM | Push · Zalo | 3 lần | [Sửa] [Clone] [Tắt]
- Nhắc nạp thẻ | SMS · USSD | 5 lần | [Sửa] [Clone] [Tắt]
- Sắp hết data | Push · SMS | 2 lần | [Sửa] [Clone] [Tắt]
- Sinh nhật KH | Zalo · Email | 4 lần | [Sửa] [Clone] [Tắt]
- Cài app nhắc nhở | Push | 1 lần | [Sửa] [Clone] [Bật]

**Behaviors:**
- [Clone]: create a copy named "Copy of [Tên]", immediately open in editor
- [Tắt]: set template to Inactive; Inactive templates do not appear in campaign dropdowns; button changes to [Bật]
- "Dùng" count = number of campaigns using this template; click number → popover listing campaign names
- Click [+ Tạo Template] or [Sửa] → navigate to Template Editor (4B)

Pagination: < 1 2 > · [20/trang ▾]

---

#### 4B — Tạo / Sửa Template

**Header:**
- Back link: ← Template
- Inline editable template name (placeholder: "Tên template \*")
- Status badge: ● Active
- Button: [Lưu Template] (primary)
- Below name: optional description input (placeholder: "Mô tả (optional)")

---

**TIẾN ĐỘ NỘI DUNG (progress bar — all channels, shown above tabs):**
- Push ● · Zalo ● · SMS ○ · USSD ○ · Banner ○ · Email ○
- ● = tab has content · ○ = tab empty/unconfigured

---

**Channel tabs (top of content area):**
[Push ●] [Zalo OA ●] [SMS ○] [USSD ○] [Banner ○] [Email ○] [+ Kênh]

- Badge per tab: ● green = has content · ○ gray = not configured
- [+ Kênh]: only list channels not already present; existing channels are disabled in dropdown

---

**Each tab = one card with 2-column layout (Soạn | Preview):**

Template is trigger-agnostic — THAM SỐ ĐỘNG includes ALL possible params, not filtered by trigger.

**Left column — Soạn:**

**THAM SỐ ĐỘNG (at the very top of each tab card):**
- Label: "THAM SỐ ĐỘNG"
- All available param chips:
  [{{ten_kh}}] [{{loai_sim}}] [{{so_dien_thoai}}] [{{so_du}}] [{{data_con_lai}}] [{{ngay_het_han}}]
- Click chip → inserts `{{param_name}}` at cursor in focused input/textarea

**Image upload (Push / Zalo OA / Banner / Email only):**
- [Tải lên] [Chọn thư viện]
- Empty: dashed border + 🖼 icon placeholder
- Filled: thumbnail + [Xóa][Đổi]
- Ratio label per channel:
  - Push: "1:1 khuyến nghị"
  - Zalo OA: "Tự do"
  - Banner: "16:9 khuyến nghị"
  - Email: "Banner ngang"
- SMS / USSD: "(Không hỗ trợ hình ảnh)" label only

**Content fields per channel:**
- Push: Title \* (counter 0/65) · Body \* (counter 0/240)
- Zalo OA: rich text textarea (unlimited)
- SMS: plain text textarea + counter "0/160 ký tự" (red when > 160)
- USSD: plain text textarea + warning "⚠ Chỉ plain text, không hỗ trợ ảnh"
- Banner: Tiêu đề input (0/50) · Nội dung textarea · CTA label input
- Email: Subject input (0/100) · rich text body editor

**Right column — Preview:**
- Channel-specific mockup renders live content:
  - Push: OS notification bar — app icon + 1:1 thumbnail right + title + body
  - Zalo OA: chat bubble + "Viettel Telecom" brand header + optional image at top
  - SMS: plain SMS bubble, no image, character counter at bottom
  - USSD: monospace terminal, plain text + menu, no image
  - Banner: 16:9 image + title + body + CTA button
  - Email: email client — subject bar + header image + body
- SAMPLE PARAMS (editable inputs below mockup):
  - ten_kh / loai_sim / data_con_lai / so_du / ngay_het_han
- [↻ Render lại] button
- Preview re-renders realtime on every keystroke and param change

**Behaviors:**
- [Lưu Template]: validate template name required; warn about empty tabs (do not block save)
- Tab completion badge updates as soon as content is entered

---

### SCREEN 5: Trigger Management

Header: "TRIGGER MANAGEMENT" + primary button [+ Thêm Trigger]

Filters:
- Search: [🔍 Tìm trigger code hoặc tên...] — realtime; auto-expands group containing matched result and highlights matched text
- Dropdown: Trạng thái (Tất cả / Active / Testing / Paused) — applies across all groups

---

**Table grouped by type — collapsible sections:**

Each group has a clickable header row: ▼ GROUP_NAME (N trigger) → click to collapse/expand

**▼ REALTIME (2 trigger)**
| SIM_ACTIVATED | Kích hoạt SIM | BSS | ● Active | [Sửa][Tắt] |
| LOC_TRAVEL_PROV | Đến tỉnh du lịch | OCS | ● Active | [Sửa][Tắt] |

**▼ NEAR REALTIME (1 trigger)**
| NO_APP_INSTALL_24H | Chưa cài app sau 24h | CRM | ● Active | [Sửa][Tắt] |

**▼ OFFLINE / BATCH (3 trigger)**
| LOW_DATA_BALANCE | Data còn dưới 100MB | OCS | ● Active | [Sửa][Tắt] |
| USAGE_DROP_40PCT | Usage giảm >40% | OCS | ○ Testing | [Sửa][Bật] |
| SIM_NO_TXN_30D | SIM ngừng GD >30 ngày | BSS | ○ Paused | [Sửa][Bật] |

**Column details:**
- Trigger Code: monospace uppercase chip (amber background)
- Event Source: small badge — BSS=blue · OCS=green · CRM=violet
- Loại thông báo: Realtime=green badge · Near Realtime=blue badge · Offline/Batch=gray badge
- Trạng thái: Active=green dot · Testing=gray · Paused=yellow

**[Tắt/Bật] behavior:**
- Click → confirm dialog
- If trigger is currently used in any Active campaign: show additional warning listing affected campaign names
- Status toggle updates immediately after confirm

---

**Modal: Thêm / Sửa Trigger**

Title: "Thêm Trigger Mới" (or "Sửa Trigger")

Fields (row 1 — 2 columns same row):
- **Trigger code \*** — uppercase, numbers, underscores only; must be unique; example: "SIM_ACTIVATED"
- **Loại \*** — dropdown: Realtime / Near Realtime / Offline

Field (row 2 — full width):
- **Tên trigger \*** — text input

Fields (row 3 — 2 columns same row):
- **Event source \*** — dropdown: BSS / OCS / CRM / Other
- **Trạng thái** — radio: ● Active · ○ Inactive

Field (row 4):
- **Mô tả** — textarea (optional)

**Payload section:**
- Label: "Payload"
- Dynamic rows: [Tên tham số \*] [Source ▾: BSS/OCS/CRM] [✕]
- [+ Thêm tham số] appends a new row

**Note:** Trigger priority is NOT configured here — priority is set only inside the Campaign Builder per campaign.

Footer: [Hủy] | [Lưu] (primary)
On [Lưu]: close modal + update table + toast "Đã lưu trigger ✓"

---

### SCREEN 6: Blacklist Management

#### 6A — Danh sách Blacklist

Header: "BLACKLIST"

Filters:
- Dropdown: Campaign [Tất cả ▾]
- Dropdown: Kênh [Tất cả ▾]
- Button: [Lọc]

Table columns: Số điện thoại | Campaign | Kênh | Hành động

Sample rows:
- 0987 xxx 001 | Chào mừng SIM | Zalo OA | [Xóa]
- 0912 xxx 002 | Hết data | SMS | [Xóa]
- 0965 xxx 003 | Hết data | Zalo OA | [Xóa]
- 0976 xxx 004 | Nhắc nạp tiền | Push | [Xóa]

[Xóa] behavior:
- Confirm dialog: "Xóa số này khỏi blacklist của campaign [X] kênh [Y]?"
- Confirm → remove row + toast "Đã xóa ✓"

Footer buttons: [+ Thêm thủ công] · [📥 Upload danh sách]

---

#### 6B — Modal: Thêm thủ công

Fields:
- **Số điện thoại \*** — with format validation (10 digits, starts with 0)
- **Campaign \*** — dropdown
- **Kênh \*** — dropdown: Zalo OA / SMS / USSD / Push / Banner / Email

Footer: [Hủy] | [Thêm] → close + toast "Đã thêm vào blacklist ✓"

---

#### 6C — Modal: Upload danh sách

Fields:
- **Campaign \*** — dropdown
- **Kênh \*** — dropdown

File upload:
- Dashed drop zone: "📄 Kéo thả file vào đây hoặc [Chọn file]"
- Below: "Format: 1 cột so_dien_thoai" + [📥 Tải file mẫu] link

After file is selected, show preview:
- "✅ 245 số hợp lệ"
- "⚠ 3 số sai định dạng (bỏ qua)"

Footer: [Hủy] | [Xác nhận Upload] → close + toast "Đã upload 245 số ✓"

---

### SCREEN 7: Customer List

Header: "KHÁCH HÀNG"

Filters (single row):
- Search: [🔍 Tìm số điện thoại...] (realtime)
- Dropdown: Trạng thái SIM [Tất cả ▾]
- Dropdown: Cài app [Tất cả ▾]
- Dropdown: DNC [Tất cả ▾]

Table columns: Số điện thoại | Loại SIM | Trạng thái | Cài app | Hành động

Sample rows:
- 0987 xxx 001 | eSIM | ● Active | Có | [Xem 360]
- 0912 xxx 002 | SIM vật lý | ● Active | Không | [Xem 360]
- 0965 xxx 003 | eSIM | ○ Inactive | Có | [Xem 360]
- 0976 xxx 004 | SIM vật lý | ● Active | Không | [Xem 360]

Pagination: < 1 2 ... > · [20/trang ▾]

Note: data sourced from BSS/OCS; no editing allowed in CVM.

---

### SCREEN 8: Customer 360

Breadcrumb: ← Danh sách khách hàng

Title: "CUSTOMER 360 — 0987 xxx 001"

**2-column layout:**

---

**LEFT COLUMN — Thông tin & Sử dụng:**

Profile block:
- Loại SIM: eSIM
- Gói + hết hạn: D50 / 31/05/2026
- Ngày kích hoạt: 01/05/2026
- Cài app: Có
- Đăng nhập app lần cuối: 05/05/2026

Sử dụng block:
- Data còn lại: 1.2 GB
- Lưu lượng hôm nay: 340 MB
- Số dư: 45,000 VND
- Nạp tiền gần nhất: 10/04/2026
- Sinh nhật: 15/06/1998

---

**RIGHT COLUMN — Trạng thái Kênh & Lịch sử:**

**TRẠNG THÁI KÊNH (sync-back from Gateway):**
- Read-only table: Kênh | Trạng thái | Cập nhật lần cuối
- Sample rows:
  - Zalo | ⛔ Blocked | 05/05 09:32
  - SMS | ✅ Active | —
  - USSD | ✅ Active | —
  - Push | ✅ Active | —
- ⛔ Blocked = red badge · ✅ Active = green badge
- Note: "ⓘ Tự cập nhật từ phản hồi Gateway. Reset về Active khi KH unblock."
- No manual edit allowed

**THROTTLING:**
- Tin hôm nay: 2/3
- Cooldown: Không
- DNC: Không

**LỊCH SỬ NHẬN TIN:**
Columns: Ngày | Campaign | Kênh | Trạng thái

Sample rows:
- 10/05 | Chào mừng SIM | Zalo | ✓ Delivered
- 08/05 | Nạp thẻ lỗi | SMS | ✓ Delivered
- 07/05 | Hết data | Zalo | ✕ Blocked → SMS | ✓ Delivered (render as 2 sub-rows, indented second row)

Link: [Xem đầy đủ lịch sử →] — opens drawer or full page with pagination + filter by kênh and campaign

---

### SCREEN 9: Report / Analytics

Header: "ANALYTICS & REPORT" + button [Xuất Excel]

**Filter bar (applies to all tabs simultaneously):**
- Thời gian: [7 ngày ▾] — options: Hôm nay / 7 ngày / 30 ngày / Tháng này / Tùy chỉnh date range
- Campaign: [Tất cả ▾]
- Segment: [Tất cả ▾]
- Quick presets: [Tháng này]
- Toggle: [So sánh: ON ▾] — when ON, show previous period data as ghost line/bar on all charts
- Kênh: [Tất cả ▾]

**[Xuất Excel]:** exports .xlsx with current filter state and active tab data

---

**6 Tabs:** [Delivery] [Engagement] [Campaign Comparison] [Segment] [Funnel] [Spam & Fatigue]

---

**Tab 1: Delivery Analytics**

KPI cards row (4 cards):
- Total Sent: 124,500
- Delivered: 119,940 (96.3%)
- Failed: 4,560 (3.7% ⚠ — amber if trending up)
- Delivery Rate: 96.3%

Charts:
- Area chart: Sent vs Delivered vs Failed per day (7-day x-axis)
- Line chart: Delivery Rate % trend per day + dashed horizontal target line at 95%
- Stacked bar: Failed Delivery by reason category — Gateway error / User blocked / DNC-Blacklist / Network timeout

---

**Tab 2: Engagement Analytics**

KPI cards row (4 cards):
- Open Rate: 38.4% ↑2.1%
- CTR: 12.1% ↑0.8%
- Conversion Rate: 8.3% ↑0.4%
- Best Channel: Push 97.7%

Charts:
- Horizontal grouped bar: Channel Performance — Open % and CTR per channel
  - Push 52% / 18%
  - Zalo OA 44% / 15%
  - SMS 22% / 8%
  - USSD 15% / 5%
  - Banner 9% / 3%
  - Email 21% / 9%
- Line chart: Engagement Trend (Open/CTR/Conversion) per day; filter by kênh available inline

---

**Tab 3: Campaign Comparison**

- Multi-select dropdown: [+ Thêm campaign để so sánh ▾] — max 5 campaigns simultaneously
- Grouped bar chart: Delivery Rate vs Conversion Rate per selected campaign
- Detail table: Campaign | Sent | Delivered | Open | Convert | Cost SMS
  - Sample: Nhắc nạp tiền | 32,100 | 30,980 | 38.4% | 9.1% | 480K VND

---

**Tab 4: Segment Analytics**

Table: Segment | Reach | Delivered | Open | Conversion
- Sample: Gen Z | 18,450 | 17,800 | 52.3% | 11.2%

Grouped bar chart: Open Rate + Conversion Rate per segment (side-by-side bars)

Device Breakdown pie or donut chart:
- Android: 62%
- iOS: 28%
- Khác: 10%

---

**Tab 5: Funnel Analytics**

- Campaign selector dropdown (top right)
- Channel selector dropdown (top right, next to campaign)
- Horizontal funnel with drop-off labels:
  - Audience đủ điều kiện: 32,100 (100%)
  - ↓ Drop 7.8% (−2,510 DNC/BL)
  - Messages Sent: 29,590 (92.2%)
  - ↓ Drop 7.7% (−2,280 undelivered)
  - Delivered: 27,310 (85.1%)
  - ↓ Drop 47.3% (−12,930 not opened) ← LARGEST DROP — highlight in red/bold
  - Opened/Clicked: 14,380 (44.8%)
  - ↓ Drop 35.6% (−5,120 no action)
  - App Installed: 9,260 (28.8%)
  - ↓ Drop 37.3% (−3,450 no purchase)
  - Package Purchased: 5,810 (18.1%)

Drop-off analysis panel below funnel:
- Highlight largest drop automatically with a callout card:
  "Điểm drop lớn nhất: Delivered → Opened (−47.3%)"
  "→ Gợi ý: A/B test tiêu đề Push"
- One action suggestion per largest drop stage

---

**Tab 6: Spam & Fatigue Report**

Line chart: Opt-out & Blacklist Trend (2 lines, 7-day) — highlight spike markers with ⚠ icon when values jump sharply

Frequency Metrics section:
- Avg tin/user/ngày: 1.8
- Avg tin/user/tuần: 4.2
- Max tin/user/ngày: 5
- User nhận > 3 tin: 8.4%
- Histogram: phân phối số tin/user
  - 1 tin: 42% | 2 tin: 31% | 3 tin: 18% | 4+ tin: 9%

Spam Risk Indicator:
- Score display: 24/100
- Badge: ● LOW RISK (green)
- Progress bar: 24% filled (amber at 40–60, red at 60+)
- Trigger alert when score > 60
- Contributing factors listed (as small info rows):
  - Opt-out rate / BL trend / avg msg per user / failed delivery ratio
- Link: [Xem chi tiết chỉ số →]

---

## EDGE CASES

Implement the following behaviors:

| Situation | Expected Behavior |
|---|---|
| Only 1 trigger in Advanced mode | Automatically P1; hide Logic OR/AND block and conflict resolution block |
| Change P1 trigger | PARAMS chips update; invalid params in message body highlight red |
| Delete P1 trigger | Next trigger in list automatically becomes P1 |
| Edit message then change trigger | Keep message content; only highlight invalid param chips in red |
| Add channel already present as tab | Channel is disabled in [+ Kênh] dropdown |
| SMS > 160 chars | Counter turns red; badge shows "X segment"; allow save; cost recalculates |
| SMS 300 chars | Show badge "2 SMS segment" |
| USSD tab not configured | Tab badge = ○; contributes to header issue count |
| [Gửi duyệt] with blocking issues | Button disabled; hover tooltip lists each issue; click issue → scroll to section |
| Deactivate trigger used in Active campaign | Confirm dialog + list of affected Active campaign names |
| Trigger priority collision (same rank) | Inline warning; require re-ordering before approval allowed |
| Subscriber matches multiple triggers (OR mode) | Send only P1 trigger message |
| No segment selected | Default T-ALL; require confirm before saving |
| Blacklist/Whitelist enabled but no file | Blocking validation; [Gửi duyệt] disabled |
| Subscriber in audience but outside whitelist | Excluded from reach; no message sent |
| Whitelist file with invalid phone format | Skip invalid rows; show count in modal preview |
| Kill Switch ([Dừng]) | Confirm dialog warns queued messages will be cancelled |
| Send time T+0 window already passed | Queue message to T+1 |
| Audience Variant: subscriber matches multiple variants | Apply the highest-priority variant (leftmost in order); Biến thể 1 is always fallback |

---

## DESIGN GUIDELINES

**Colors:**
- Background: white
- Primary: `#1E40AF` (blue)
- Sidebar: gray-100
- Section card left border accent: yellow/amber

**Components:**
- Section cards: white background + subtle shadow + amber left border (4px)
- Section number labels: bold colored, e.g. "1. Thông tin Campaign"
- Trigger chips (compact): amber background (`bg-amber-100 text-amber-800`); P1 chip has gold ★ prefix
- BSS source badge: blue · OCS badge: green · CRM badge: violet
- PARAMS chips: blue pill (`bg-blue-100 text-blue-700`), clickable; invalid = red pill (`bg-red-100 text-red-700`) + tooltip
- Image upload: dashed border + 🖼 placeholder before upload; thumbnail + [Xóa][Đổi] after upload
- Media Library modal: 4-column image grid; selected = blue border + checkmark; footer: [Hủy] | [Chọn ảnh này]

**Channel Previews (render in realistic format):**
- Push: OS notification bar — app icon + 1:1 thumbnail right + title + body
- Zalo OA: chat bubble + "Viettel Telecom" brand header + optional image at top
- SMS: plain phone SMS bubble, no image, character counter
- USSD: monospace terminal, plain text + menu, no image, dấu warning shown
- Banner: 16:9 image + title + body + CTA button
- Email: email client — subject bar + header image + body

**Status indicators:**
- Channel status in Customer 360: ⛔ Blocked = red badge · ✅ Active = green badge
- Campaign status: Active=green · Draft=gray · Pending=yellow · Paused=orange
- Trigger group type badges: Realtime=green · Near Realtime=blue · Offline=gray

**Tables:**
- Striped rows; hover = highlight; pointer cursor on clickable rows

**Toasts:**
- Bottom right; auto-dismiss after 3s; success = green · error = red

**Dialogs:**
- Confirm dialog: centered overlay with title + description + [Hủy] [Xác nhận] buttons

**Validation:**
- Inline: red border + error message below field
- Blocking issues: surface in header button tooltip + issue counter badge

**Responsive:**
- Desktop-first, minimum width 1440px
- Component library: shadcn/ui style

**Sidebar navigation items:** Dashboard / Campaign / Template / Trigger / Blacklist / Customer / Report / Settings
