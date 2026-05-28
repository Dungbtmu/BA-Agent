# Lovable Prompt — CVM System v5
> Cập nhật: Trigger Management (lifecycle stage, priority, campaign linked drill-down) + Campaign Builder Section 4 (segment taxonomy + condition builder)

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
- Admin: full access including Trigger Management
- QTV: create/edit campaigns, view reports

---

## GLOBAL LAYOUT
- Fixed left sidebar: Dashboard, Campaign, Template, Trigger, Customer, Report
- Top navbar: logo left, notification bell + avatar right

---

## SCREENS

---

### SCREEN 1: Dashboard

KPI cards row (3 cards):
- Active Campaigns: 12
- Messages Sent Today: 48,320
- Failure Rate: 2.3%

Table "Active Campaigns":
- Columns: Campaign Name, Trigger Count, Sent, Failed, Actions
- Sample rows:
  - Chào mừng SIM mới | 2 | 1,200 | 12
  - Nạp thẻ lỗi | 1 | 892 | 5
  - Hết data | 3 | 3,100 | 67
- "Xem tất cả" link top right

Feed "Recent Trigger Events":
- Columns: Time, Trigger Name, Campaign Matched
- Sample rows:
  - 10:32 | SIM_ACTIVATE | Chào mừng SIM mới
  - 10:31 | TOP_UP_FAIL | Nạp thẻ lỗi
  - 10:28 | DATA_EMPTY | Hết data
- "Xem log" link top right

---

### SCREEN 2: Campaign List

Header: "Campaign" + primary button "+ Tạo Campaign"

Filters:
- Search bar: "Tìm kiếm tên campaign..."
- Dropdown: Trạng thái (Tất cả / Active / Draft / Paused / Pending / Ended)
- Dropdown: Trigger (Tất cả)

Table columns: Campaign Name, Trigger codes, Logic, Trạng thái, Hiệu lực, Actions

Trigger codes column:
- Show trigger codes as compact badge chips so users can scan which triggers each campaign uses.
- Show max 3 trigger chips inline.
- If campaign has more than 3 triggers, show a "+N" chip.
- Hover a trigger chip → tooltip with trigger name, source, execution type.
- Click a trigger chip → open a small quick-view popover or filter/navigate to Trigger Management for that trigger.
- Hover/click "+N" → popover listing all trigger codes.

Logic column:
- Values: Basic / OR / AND.
- Basic = one trigger only.
- OR/AND = advanced multi-trigger campaign.

Status badges (pill shape):
- Active = green dot
- Draft = gray
- Paused = yellow
- Pending Review = orange
- Ended = muted red

Row actions:
- Active → [Xem] [Sửa] [Dừng]
- Draft → [Xem] [Sửa] [Xóa]
- Paused → [Xem] [Sửa] [Resume]
- Pending → [Xem]
- Ended → [Xem]

"Xem" opens Campaign Detail View read-only, showing all configured trigger, audience, message, channel strategy, delivery schedule, safety rules, and approval info.

Sample rows:
- Welcome eSIM Q2/2026 | [SIM_ACTIVATED] [NO_APP_INSTALL_24H] [LOW_DATA_BALANCE] | OR | Draft | 01/04–30/06 | [Xem] [Sửa]
- Nhắc nạp tiền | [TOP_UP_FAIL] | Basic | Active | 04/05–18/05 | [Xem] [Sửa] [Dừng]
- Hết hạn gói data | [PACKAGE_EXPIRED] | Basic | Paused | 01/05–30/05 | [Xem] [Sửa] [Resume]
- Gói roaming du lịch | [LOC_TRAVEL_PROV] [LOC_TRANSIT_HUB] [+2] | OR | Pending Review | 01/06–30/06 | [Xem]

Pagination: < 1 2 3 >

---

### SCREEN 3: Campaign Builder (Smart UX Revision)

Build this screen as a smart 2-column operational Campaign Builder, not a stepper/wizard.

Goal: support both simple campaigns with 1 trigger and advanced campaigns with multiple triggers. The UI must reduce confusion by separating campaign info, trigger logic, audience, message matrix, channel strategy, trigger-based delivery schedule, and final review.

#### Overall layout

```
[FIXED HEADER]
[LEFT COL ~68% scrollable] | [RIGHT COL ~32% sticky]
```

Left column sections:
1. Thông tin Campaign
2. Trigger & Logic
3. Audience / Phân khúc
4. Message Matrix
5. Channel Strategy
6. Cấu hình gửi theo Trigger
7. Review & Submit

Right sticky column:
- Campaign Summary
- Validation Checklist
- Cost & SMS Risk
- Approval Status

---

#### Fixed Header

- Back link: "← Danh sách Campaign"
- Campaign name display/input: "Welcome eSIM Q2/2026"
- Campaign code: "CVM-WELCOME-ESIM-Q2-2026"
- Status badge: ● Draft
- Buttons: [Lưu Nháp] secondary, [Gửi duyệt →] primary
- Disable [Gửi duyệt →] when validation has blocking issues.

---

**SECTION 1: Thông tin Campaign**
(card with yellow left border, numbered "1.")

Fields:
- Tên campaign *: "Welcome eSIM Q2/2026"
- Mã campaign / kịch bản: "CVM-WELCOME-ESIM-Q2-2026"
- Mục tiêu campaign: "Onboard thuê bao eSIM mới, nhắc cài app, nhắc mua thêm data"
- Thời gian hiệu lực: Từ ngày 01/04/2026 đến ngày 30/06/2026
- Người tạo: QTV Marketing
- Trạng thái: Draft

---

**SECTION 2: Trigger & Logic**
(card with yellow left border, numbered "2.")

Trigger selection mode:
- Segmented control:
  - Basic — 1 trigger chính
  - Advanced — nhiều trigger + AND/OR rõ ràng
- Default selected: Advanced.
- Basic mode allows only 1 trigger and hides AND/OR controls.
- Advanced mode allows multiple triggers and requires explicit OR/AND selection.

Trigger dropdown:
- Searchable dropdown button: "+ Thêm trigger"
- Sample trigger options:
  - BSS · SIM_ACTIVATED · Kích hoạt SIM thành công · Realtime
  - CRM · NO_APP_INSTALL_24H · Chưa cài app sau 24h · Near realtime
  - OCS · LOW_DATA_BALANCE · Data còn dưới 100MB · Offline / Batch

Selected trigger cards:

Card T1:
- Header: ★ T1 · SIM_ACTIVATED
- Source: BSS
- Name: Kích hoạt SIM thành công
- Execution type badge: Realtime — gửi ngay khi event đến
- Payload chips: [ten_kh] [loai_sim] [ngay_kich_hoat] [so_dien_thoai]

Card T2:
- Header: T2 · NO_APP_INSTALL_24H
- Source: CRM
- Name: Chưa cài app sau 24h
- Execution type badge: Near realtime — sau 24h
- Payload chips: [app_installed] [last_active]

Card T3:
- Header: T3 · LOW_DATA_BALANCE
- Source: OCS
- Name: Data còn dưới 100MB
- Execution type badge: Offline / Batch — 09:00 hằng ngày
- Payload chips: [data_con_lai] [ten_goi] [ngay_het_han]

Advanced logic block:
- Radio/segmented control:
  - OR — KH match trigger nào thì gửi message của trigger đó
  - AND — KH phải thỏa tất cả trigger, gửi 1 message chung
- Default selected: OR.
- Show human-readable explanation under the toggle:
  - Nếu KH kích hoạt SIM → gửi Message Welcome
  - Nếu KH chưa cài app 24h → gửi Message Chưa cài app
  - Nếu KH còn data < 100MB → gửi Message Sắp hết data

Behavior:
- Removing a trigger updates Message Matrix and right summary immediately.
- If mode changes from Advanced to Basic, show confirm because it may remove trigger-specific messages.

---

**SECTION 3: Audience / Phân khúc**
(card with yellow left border, numbered "3.")

Use segment data from Customer 360 / Team Data / BSS / OCS. Do not build a full segment rule builder in this phase.

Header area:
- Data source label: "Nguồn phân khúc: Customer 360 / Team Data / BSS / OCS"
- Last updated: "12/05/2026 08:00"
- Search input: "Tìm phân khúc..."

Segment list with checkbox, name, count:
- ☑ Gen Z User (18–25) — 18,450 KH
- ☐ Thuê bao mới kích hoạt — 32,100 KH
- ☑ Sắp hết data — 8,920 KH
- ☐ Chưa cài app MyViettel — 14,300 KH
- ☐ ARPU cao — 5,600 KH
- ☐ Nguy cơ rời mạng — 2,150 KH

Selected badges:
- [Gen Z User ×]
- [Sắp hết data ×]

Segment logic:
- OR: Thuộc bất kỳ phân khúc nào
- AND: Thuộc tất cả phân khúc đã chọn
- Default selected: OR.

Optional additional filter:
- Label: "Điều kiện lọc bổ sung (optional)"
- Example row: [Loại thiết bị] [=] [Android] [×]
- Button: "+ Thêm điều kiện"
- Make it clear this is a filter on selected segments, not segment master creation.

Estimated reach:
- "Ước tính reach: ~6,800 KH ↻"

---

**SECTION 4: Message Matrix**
(card with yellow left border, numbered "4.")

Core model:
- Display label: "Message model: Trigger × Segment × Channel"
- In Advanced OR: each trigger has a separate message per channel.
- In Advanced AND: each channel has one shared message.
- A trigger can have multiple audience variants if selected segments require different wording.

Completion summary:
- Push: 3/3 ✓
- Zalo: 3/3 ✓
- SMS: 2/3 ⚠
- Email: 0/3

Channel tabs:
- [Push] [Zalo OA] [SMS] [Banner] [Email] [+ Thêm kênh]
- Push tab selected by default.

Inside active tab Push, render trigger message cards:

Message card T1:
- Status: ✓ configured
- Header: T1 · SIM_ACTIVATED
- Audience variant: [Gen Z User]
- Template dropdown: "Welcome Push Template"
- Title: "Chào mừng bạn!"
- Body: "SIM eSIM đã kích hoạt thành công. Tải MyViettel..."
- Preview area: Push notification preview

Message card T2:
- Status: ✓ configured
- Header: T2 · NO_APP_INSTALL_24H
- Audience variant: [Gen Z User]
- Template dropdown: "Install App Reminder"
- Title: "Cài MyViettel để nhận ưu đãi"
- Body: "Bạn vẫn chưa cài app MyViettel..."
- Preview area: Push notification preview

Message card T3:
- Status: ✓ configured
- Header: T3 · LOW_DATA_BALANCE
- Audience variant: [Gen Z User]
- Template dropdown: "Low Data Gen Z Push"
- Title: "Sắp hết data rồi 😮"
- Body: "Bạn chỉ còn {{data_con_lai}} MB. Mua thêm gói..."
- Button: "+ Thêm biến thể đối tượng"

When switching to SMS tab:
- Show same trigger rows but SMS-specific content and counters:
  - T1 SMS Welcome: ✓ 92/160 ký tự
  - T2 SMS Chưa cài app: ✓ 118/160 ký tự
  - T3 SMS Sắp hết data: ⚠ Chưa cấu hình

Template behavior:
- Support choosing approved/reusable templates from Template Library.
- Also allow composing content in campaign for now; campaign approval reviews the full campaign, not only template approval.
- Template Library should feel serious: version, channel support, variable list, usage count.

---

**SECTION 5: Channel Strategy**
(card with yellow left border, numbered "5.")

Purpose: prioritize lower-cost channels first, SMS last.

Build this as an ordered fallback flow with draggable/reorderable cards:

Step 1: Push App
- Sends according to the matched trigger schedule.
- Wait for response: [30] phút.
- Stop fallback if customer: [Clicked ▾].

Step 2: Zalo OA / ZNS
- Sends after Push if not clicked after 30 minutes.
- Wait for response: [30] phút.
- Stop fallback if: [Delivered hoặc Clicked ▾].

Step 3: SMS
- Last channel only.
- Sends only if previous channels fail or do not meet stop condition.
- Show estimate: "Ước tính SMS: ~1,200 tin".

Cost-saving checkbox:
- ☑ Không gửi SMS nếu KH đã click/open ở kênh trước

Stop fallback condition checkboxes:
- ☐ Delivered
- ☐ Opened
- ☑ Clicked

Behavior:
- Trigger determines when processing starts.
- Channel Strategy determines which channel sends first/next and after how long.
- Safety rules determine whether the planned send can happen now or must be queued/blocked.

---

**SECTION 6: Cấu hình gửi theo Trigger**
(card with yellow left border, numbered "6.")

Show info note:
"ⓘ Thời điểm gửi phụ thuộc vào trigger; không có lịch gửi chung cho toàn campaign."

Render one schedule card per selected trigger:

T1 · SIM_ACTIVATED
- Kiểu chạy: Realtime
- Xử lý: ngay khi nhận event hợp lệ
- Blackout: queue đến 08:00 nếu rơi 20:00–08:00

T2 · NO_APP_INSTALL_24H
- Kiểu chạy: Near realtime
- Delay: 24h sau SIM_ACTIVATED
- Điều kiện: app_installed = false
- Blackout: queue đến 08:00 nếu rơi 20:00–08:00

T3 · LOW_DATA_BALANCE
- Kiểu chạy: Offline / Batch
- Lịch quét: 09:00 hằng ngày
- Điều kiện: data_con_lai <= 100MB
- Blackout: không chạy batch trong 20:00–08:00

---

**SECTION 7: Review & Submit**
(card with yellow left border, numbered "7.")

Checklist before approval:
- ✓ Có ít nhất 1 trigger
- ✓ Đã chọn phân khúc
- ⚠ SMS còn thiếu message cho T3 LOW_DATA_BALANCE
- ✓ Có channel strategy
- ✓ Có blacklist / DNC check
- ✓ Có frequency cap

Approval note textarea:
- Placeholder/example: "Campaign onboard eSIM Q2, ưu tiên Push/Zalo trước SMS"

Buttons:
- [Lưu Nháp]
- [Gửi duyệt →]

---

#### RIGHT STICKY COLUMN

Build a compact sticky panel that remains visible while scrolling.

Campaign Summary:
- Trigger: Advanced · OR · 3 triggers
- Audience: 2 segments · ~6,800 KH
- Message:
  - Push: 3/3 ✓
  - Zalo: 3/3 ✓
  - SMS: 2/3 ⚠
  - Email: 0/3
- Channel Strategy: Push → Zalo → SMS
- Stop fallback: Clicked
- Risk / Cost:
  - Estimated SMS: ~1,200
  - Estimated cost: ~360,000 VND
- Approval: Draft, Admin duyệt để Active

Interactions:
- Clicking `SMS: 2/3` scrolls to the missing SMS message.
- Cost/SMS risk updates when changing segment, channel strategy, fallback condition, or message completion.

---

### SCREEN 4: Template Management

Header: "Template" + "+ Tạo Template"

Filters:
- Search: "Tìm template..."
- Dropdown: Trạng thái (Tất cả / Active / Inactive)

Table columns: Tên Template, Kênh hỗ trợ, Số lần dùng, Hành động
Sample rows:
- Chào mừng SIM | Zalo OA, SMS | 3 lần | [Sửa] [Clone]
- Nạp thẻ hướng dẫn | SMS, USSD | 5 lần | [Sửa] [Clone]
- Sinh nhật KH | Zalo OA, Email | 2 lần | [Sửa] [Clone]

Click [+ Tạo Template] or [Sửa] → navigate to Template Editor page

---

### SCREEN 4B: Template Editor (2-column layout)

Same layout as Campaign Builder: left col ~65% scrollable, right col ~35% sticky.

#### Fixed Header:
- Back link: "← Danh sách Template"
- Inline editable template name (placeholder: "Tên template...")
- Status toggle: Active / Inactive
- Button: [Lưu Template] (primary)

2-column layout:

Left column — Content Editor:
- Tab bar: [USSD] [Zalo OA] [SMS] [Banner] [Push] [Email]
- Active tab shows textarea for that channel's content
- Dynamic parameter chips below textarea (same as Campaign Builder):
  [{{ten_kh}}] [{{so_du}}] [{{data_con_lai}}] [{{ten_goi}}] [{{ngay_het_han}}]
  Click → insert at cursor
- SMS tab: character counter

Right column — Preview (sticky):
- Channel dropdown (synced with active tab)
- Preview box with rendered content
- Editable sample params

#### LEFT COLUMN — Content Editor:

Channel tabs: [Zalo OA] [SMS] [USSD] [Banner] [Push] [Email] [+ Thêm kênh]
- Tabs are closable (✕) except when only 1 tab remains
- "+ Thêm kênh" → dropdown showing only unselected channels
- No fallback config (template không quản lý fallback, chỉ Campaign mới config)

Inside each channel tab — same content fields as Campaign Builder:

IMAGE UPLOAD (optional) — Zalo OA, Banner, Push, Email only:
- Upload area: [Tải ảnh lên] | [Chọn từ thư viện]
- Before upload: grey placeholder box with 🖼 icon + dashed border
- After upload: thumbnail + [Xóa ảnh] [Đổi ảnh]
- Ratio label per channel:
  - Zalo OA: "Tự do"
  - Banner App/Web: "16:9 khuyến nghị"
  - Push Notification: "1:1 khuyến nghị"
  - Email: "Banner ngang"
- SMS / USSD: show "(Không hỗ trợ hình ảnh)" label

CONTENT FIELDS per channel:
- Zalo OA: textarea (rich text, unlimited)
- SMS: textarea plain text + counter "0 / 160 ký tự" red > 160
- USSD: textarea plain text + warning "⚠ USSD không hỗ trợ tiếng Việt có dấu"
- Banner App/Web: input "Tiêu đề" 0/50 + textarea "Nội dung" + input "CTA Button label"
- Push Notification: input "Tiêu đề" 0/65 + textarea "Nội dung" 0/240
- Email: input "Subject" 0/100 + rich text editor for body

Parameter chips (below content fields):
- Label: "Tham số động:"
- All available params as clickable chips (not filtered by trigger — template is trigger-agnostic):
  [{{ten_kh}}] [{{loai_sim}}] [{{so_du}}] [{{data_con_lai}}] [{{ten_goi}}]
  [{{ngay_het_han}}] [{{ngay_kich_hoat}}] [{{so_dien_thoai}}] [{{so_lan_nap}}]
- Click chip → insert at cursor in active text field

#### RIGHT COLUMN — Preview (sticky):

- Label: "XEM TRƯỚC (PREVIEW)"
- Channel dropdown: syncs with active tab in left column
- Preview box: renders content with sample params, formatted per channel

Preview format per channel (same as Campaign Builder):
- Zalo OA: chat bubble + brand header + optional image at top
- SMS: plain SMS bubble, no image, character counter
- USSD: monospace terminal style, no image, dấu warning
- Banner: card with 16:9 image + title + body + CTA button
- Push: OS notification style, 1:1 thumbnail right, title + body counters
- Email: email client style, subject + header image + body

Sample params (editable inputs — all params, not filtered by trigger):
- ten_kh = [Nguyễn Văn A]
- so_du = [45,000 VNĐ]
- data_con_lai = [1.2 GB]
- ten_goi = [D50]
- ngay_het_han = [18/05/2026]

Preview re-renders on every keystroke or sample param change.
When image uploaded → preview shows thumbnail immediately.

---

### SCREEN 5: Trigger Management — UPDATED v5

Header: "Trigger Management" + primary button "+ Thêm Trigger"

**FILTER BAR (above the priority info banner):**
- Dropdown: "Trạng thái" (Tất cả / Active / Paused / Testing)
- Dropdown: "Giai đoạn VĐ" (Tất cả / G3 Early Use / G4 Tăng trưởng / G5 Suy giảm / G6 Win-back)
- These two filters sit side by side in the same row

**PRIORITY INFO BANNER (below filters, above table):**
- Background: yellow-50 (very light yellow), border: yellow-200, left accent border: yellow-400
- Icon: ⓘ information icon
- Text: "Quy tắc ưu tiên khi xung đột: T-LOC-2 (Di chuyển GT) > T-LOC-1 (Du lịch) > T-JOB > T-AGE — Khi KH nhận 2 trigger cùng lúc, chỉ campaign có priority cao nhất được gửi."

**TABLE — 8 columns:**

Columns: Mã | Tên Trigger | Event Source | Giai đoạn VĐ | Ưu tiên | Campaign Linked | Trạng thái | Hành động

Column details:
- Mã: short code (T1, T2, T3, G5, D1, W1)
- Tên Trigger: text, may include badge/icon for special priority marker
- Event Source: "BSS SIM_ACTIVATED" / "CRM NO_APP_INSTALL_24H" / "OCS LOW_DATA_BALANCE" etc — show source system as small badge (BSS=blue, CRM=violet, OCS=green) + event name
- Giai đoạn VĐ: colored badge pill
  - G3 Early Use → blue (bg-blue-100 text-blue-700)
  - G4 Tăng trưởng → green (bg-green-100 text-green-700)
  - G5 Suy giảm → orange (bg-orange-100 text-orange-700)
  - G6 Win-back → purple (bg-purple-100 text-purple-700)
- Ưu tiên: show priority level as "P1" label + star icons
  - P1 = ★★★★★ (5 filled stars, gold)
  - P2 = ★★★★☆ (4 filled stars)
  - P3 = ★★★☆☆ (3 filled stars)
  - Lower number = higher priority
- Campaign Linked: clickable link text "N campaign(s)" — click opens Campaign Linked modal
  - "0 campaigns" → show as gray non-link text
  - "1 campaign" or more → show as blue underline link
- Trạng thái: pill badge
  - Active = green dot + "Active"
  - Paused = yellow + "Paused"
  - Testing = gray + "Testing"
- Hành động: [Sửa] [Tắt] for Active triggers, [Sửa] [Bật] for Paused/Testing triggers

Sample data rows:
| T1 | Kích hoạt SIM thành công | BSS SIM_ACTIVATED | G3 Early Use | P1 ★★★★★ | Welcome eSIM | Active | [Sửa][Tắt] |
| T2 | Chưa cài app sau 24h | CRM NO_APP_INSTALL_24H | G3 Early Use | P2 ★★★★☆ | Welcome eSIM | Active | [Sửa][Tắt] |
| T3 | Data còn dưới 100MB | OCS LOW_DATA_BALANCE | G3 Early Use | P2 ★★★★☆ | Welcome eSIM | Active | [Sửa][Tắt] |
| G5 | T-LOC-1 · Đến tỉnh du lịch | OCS LOC_TRAVEL_PROV | G4 Tăng trưởng | P2 ★★★★☆ | 1 campaign | Active | [Sửa][Tắt] |
| D1 | Usage giảm >40% vs baseline | OCS USAGE_DROP_40PCT | G5 Suy giảm | P3 ★★★☆☆ | 0 campaigns | Testing | [Sửa][Bật] |
| W1 | SIM ngừng giao dịch >30 ngày | BSS SIM_NO_TXN_30D | G6 Win-back | P2 ★★★★☆ | 1 campaign | Paused | [Sửa][Bật] |

Note: T1/T2/T3 are the sample triggers used by campaign "Welcome eSIM Q2/2026". The "Campaign Linked" cell may show "Welcome eSIM" as a blue link instead of numeric count for demo clarity.

**[Tắt/Bật] behavior:** show confirm dialog before toggling. If trigger has Active campaigns → show warning: "Trigger này đang được dùng bởi N campaign Active. Tắt trigger sẽ ảnh hưởng các campaign này."

---

**MODAL: Thêm / Sửa Trigger — UPDATED v5**

Title: "Thêm Trigger Mới" (or "Sửa Trigger" when editing)

Fields:
- Input: "Tên trigger *" (full width)
- Dropdown: "Event source *" — options: BSS / OCS / CRM / Other
- Textarea: "Mô tả" (optional)

NEW FIELD — Dropdown: "Giai đoạn vòng đời *" (required)
- Options:
  - G3 — Early Use (Ngày 0–30)
  - G4 — Tăng trưởng (Tháng 2+)
  - G5 — Suy giảm
  - G6 — Win-back
- Show colored badge preview next to selected option

NEW FIELD — Input: "Độ ưu tiên *" (required)
- Number input (integer only, min 1)
- Inline tooltip ⓘ: "Số nguyên dương. Số nhỏ hơn = ưu tiên cao hơn. Khi KH nhận nhiều trigger cùng lúc, chỉ campaign của trigger có số ưu tiên nhỏ nhất được gửi."
- Validation: show red border + error message if non-integer or negative

Payload section:
- Label: "Tham số payload"
- Dynamic rows: [Tên tham số *] [Source: BSS / OCS / CRM ▾] [✕]
- Button: "+ Thêm tham số" — appends new row

Toggle: "Trạng thái" → Active / Inactive

Footer: [Hủy] | [Lưu] (primary)

On [Lưu]: close modal + update table row + show toast "Đã lưu trigger ✓"

---

**MODAL: Campaign Linked — NEW v5**

Triggered by: clicking the campaign count link in the "Campaign Linked" column.

Title: 'Campaign đang dùng trigger "[Tên trigger]"'
Subtitle: "N campaign" (count)

Table inside modal — 3 columns:
- Tên campaign
- Trạng thái (pill badge: Active=green, Paused=yellow)
- Hiệu lực (date range or "Liên tục")

Sample data (for trigger "Chào mừng kích hoạt SIM", 2 campaigns):
| Chào mừng SIM | ● Active | 04/05–18/05 |
| Ưu đãi KH mới | ● Active | Liên tục |

Sample data (for trigger "T-LOC-1 · Đến tỉnh du lịch", 1 campaign):
| Ưu đãi du lịch nội địa | ● Active | 01/05–31/05 |

Sample data including a Paused campaign (for full example):
| Chào mừng SIM | ● Active | 04/05–18/05 |
| Ưu đãi KH mới | ● Active | Liên tục |
| Test SIM eSIM | ○ Paused | 01/05–30/05 |

Warning bar (shown only if there is at least 1 Active campaign):
- Background: red-50, border: red-200
- Icon: ⚠
- Text: "Tắt trigger này sẽ ảnh hưởng [N] campaign Active"

Empty state (for trigger with 0 campaigns):
- No table
- Show: "Chưa có campaign nào sử dụng trigger này."

Footer: single button [Đóng] (closes modal, no other actions)

---

### SCREEN 6: Blacklist Management

Sidebar label: "Blacklist"

Header: "Blacklist"

Filters:
- Dropdown: Campaign (Tất cả / [list of campaigns])
- Dropdown: Kênh (Tất cả / Zalo OA / SMS / USSD / Banner / Push / Email)
- Button: [Lọc]

Table columns: Số điện thoại, Campaign, Kênh, Hành động
Sample rows:
- 0987 xxx 001 | Chào mừng SIM | Zalo OA | [Xóa]
- 0912 xxx 002 | Hết data | SMS | [Xóa]
- 0965 xxx 003 | Hết data | Zalo OA | [Xóa]

Pagination: < 1 2 3 >

Footer action buttons:
- [+ Thêm thủ công] (secondary)
- [📥 Upload danh sách] (secondary)

Click [Xóa]: confirm dialog "Xóa số này khỏi blacklist?" → [Hủy] [Xác nhận] → remove row + toast ✓

---

Modal: "+ Thêm thủ công"
- Input: Số điện thoại * (with format validation)
- Dropdown: Campaign *
- Dropdown: Kênh * (Zalo OA / SMS / USSD / Banner / Push / Email)
- Buttons: [Hủy] | [Thêm]
- On save: close modal + add row to table + toast "Đã thêm vào blacklist ✓"

---

Modal: "Upload danh sách"
- Dropdown: Campaign *
- Dropdown: Kênh *
- File upload area:
  - Dashed border box: "📄 Kéo thả file CSV vào đây hoặc [Chọn file]"
  - Below: "Format: 1 cột so_dien_thoai" + [📥 Tải file mẫu] link
- After file selected → show preview below upload area:
  - "✅ 245 số hợp lệ"
  - "⚠ 3 số sai định dạng (sẽ bỏ qua)"
- Buttons: [Hủy] | [Xác nhận Upload]
- On confirm: close modal + update table + toast "Đã upload 245 số ✓"

---

### SCREEN 7A: Customer List

Header: "Khách hàng"
Filters: search + Loại SIM + Gói cước + [Lọc]
Table: Số ĐT, Loại SIM, Gói cước, Ngày KH, [Xem]
Click row or [Xem] → Customer 360

---

### SCREEN 7B: Customer 360

Breadcrumb: "Khách hàng / 0987 xxx 001"

Section 1 — Thông tin khách hàng:
- Số ĐT, Loại SIM, Trạng thái, Gói + hết hạn, Ngày KH SIM, Thiết bị, Cài app, Đăng nhập app

Section 2 — Thống kê sử dụng (2-col table):
- Data còn lại, Lưu lượng hôm nay, Số dư, Số lần nạp, Số lần gia hạn, Cuộc gọi thất bại, Ngày sinh nhật, Nghề nghiệp

Section 3 — Trạng thái kênh (Sync-back) — NEW:
- Label: "TRẠNG THÁI KÊNH"
- Table columns: Kênh, Trạng thái, Cập nhật lần cuối
- Sample rows:
  - Zalo OA | ⛔ Blocked | 05/05/2026 09:32
  - SMS | ✅ Active | —
  - USSD | ✅ Active | —
  - Push | ✅ Active | —
- Info note below table: "ⓘ Trạng thái tự động cập nhật từ phản hồi Gateway. Reset về Active khi KH unblock."
- Blocked status = red badge, Active = green badge

Section 4 — Throttling Status:
- Tin nhắn hôm nay: 2/3 (còn 1 tin)
- Cooldown active: Không
- BSS DNC Flag: Không

Section 5 — Lịch sử nhận tin nhắn:
- Columns: Thời gian, Campaign, Kênh, Trạng thái
- Status values: ✓ Delivered / ✕ Failed / ✓ Fallback / ⛔ Blocked→Fallback
- Sample rows:
  - 10/05 09:32 | Chào mừng SIM | Zalo OA | ✓ Delivered
  - 08/05 14:10 | Nạp thẻ lỗi | SMS | ✓ Delivered
  - 07/05 08:45 | Hết data | Zalo OA | ⛔ Blocked→Fallback
  - 07/05 08:45 | Hết data | SMS | ✓ Delivered

---

### SCREEN 8: Report

Filters: Campaign dropdown + date range + [Lọc]
KPI: Đã gửi 48,320 | Delivered 46,891 | Failed 1,429 | Fallback Rate 2.3%
Bar chart: tin nhắn theo ngày
Horizontal bar: USSD 45% / Zalo OA 30% / SMS 14% / Banner 7% / Push 3% / Email 1%

---

## DESIGN GUIDELINES
- Color: white background, #1E40AF blue primary, gray-100 sidebar
- Sidebar items: Dashboard, Campaign, Template, Trigger, Blacklist, Customer, Report
- Section cards: white background, subtle shadow, yellow/amber left border accent
- Section numbers: bold colored label e.g. "1. Thông tin & Trigger"
- Trigger cards: light amber background, BSS badge = blue, OCS badge = green
- Primary trigger: ★ star badge in gold
- Parameter chips: blue pill, clickable, invalid params = red pill + tooltip
- Image upload area: grey placeholder box with 🖼 icon + dashed border when empty; thumbnail + [Xóa] [Đổi] buttons when filled
- Media Library modal: 4-col grid, selected = blue border + checkmark overlay, [Hủy] | [Chọn ảnh này] footer
- Inside each channel tab in Section 2: 2-column layout — left = content editor, right = preview realtime
- Preview renders different visual format per channel:
  - Zalo OA: chat bubble with brand header + optional image at top
  - SMS: plain phone SMS bubble, no image, character counter
  - USSD: monospace terminal style, plain text + menu numbers, no image, dấu warning
  - Banner: card with 16:9 image top + title + body + CTA button
  - Push: OS notification bar style, small 1:1 thumbnail right, title + body counters
  - Email: email client style, subject line + header image + body
- Channel status badges in Customer 360: ⛔ Blocked = red, ✅ Active = green
- Blacklist upload modal: dashed drop zone, preview result before confirming
- Channel tabs: horizontal tabs, closable, first tab label "Fallback: kênh đầu"
- Right column (Phân khúc + An toàn) fully sticky
- Tables: striped, hover highlight, pointer cursor on rows
- Toast: bottom right, auto-dismiss 3s
- Confirm dialog: centered overlay
- Inline validation: red border + message below field
- Desktop-first, min-width 1440px
- shadcn/ui component style

**NEW in v5 — Trigger Management specific:**
- Lifecycle stage badges: G3=blue (bg-blue-100 text-blue-700), G4=green (bg-green-100 text-green-700), G5=orange (bg-orange-100 text-orange-700), G6=purple (bg-purple-100 text-purple-700)
- Priority stars: gold filled stars (text-yellow-400) for filled, gray for empty (text-gray-300)
- Priority info banner: bg-yellow-50 border border-yellow-200 with left accent border-l-4 border-l-yellow-400
- Campaign Linked link: text-blue-600 underline cursor-pointer; "0 campaigns" = text-gray-400 no-underline no-cursor
- Campaign Linked modal warning bar: bg-red-50 border border-red-200 text-red-700
- Trigger name with ⚡ ƯU TIÊN: inline badge with bg-yellow-100 text-yellow-800 text-xs font-medium

**NEW in v5 — Section 4 Phân khúc specific:**
- Segment badges (Part A): bg-blue-100 text-blue-700 rounded-full px-2 py-1 text-sm with × button
- "+ Chọn phân khúc ▾": secondary outlined button
- Part A / Part B separator: thin gray horizontal rule (border-gray-200)
- Part B subtext "Kết hợp bằng AND": text-gray-500 text-xs italic
- Condition builder rows: same style as v4 — inline dropdowns + remove button
