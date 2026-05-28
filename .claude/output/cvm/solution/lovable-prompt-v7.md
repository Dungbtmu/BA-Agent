# Lovable Prompt — CVM System v7
> Cập nhật từ v5: USSD trong Message Matrix, bỏ Review & Submit, thêm Safety/Whitelist, Channel Strategy theo kênh đã thêm, SMS cost theo 160 ký tự/segment, campaign-level trigger priority

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

Goal: support both simple campaigns with 1 trigger and advanced campaigns with multiple triggers. The UI must reduce confusion by separating campaign info, trigger logic, audience, message matrix, channel strategy/safety, and trigger-based delivery schedule.

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
5. Channel Strategy & An toàn
6. Cấu hình gửi theo Trigger

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
- In Advanced OR mode, show a conflict handling block:
  - Radio option selected by default: "Chỉ gửi trigger ưu tiên cao nhất"
  - Secondary option: "Gửi tất cả trigger đã match (không khuyến nghị)"
  - Show a draggable campaign priority list:
    - [drag] P1 T3 LOW_DATA_BALANCE — Data còn dưới 100MB
    - [drag] P2 T2 NO_APP_INSTALL_24H — Chưa cài app sau 24h
    - [drag] P3 T1 SIM_ACTIVATED — Kích hoạt SIM thành công
  - Allow changing priority via drag-and-drop or P1/P2/P3 dropdown on each row.
  - Show note: "Priority này chỉ áp dụng trong campaign, không đổi master trigger."
- If a subscriber matches all 3 triggers at the same time in OR mode, send only the message for the highest-priority trigger by default.
- If Advanced logic = AND, hide the conflict handling block because AND uses one shared message per channel.
- If two triggers have the same campaign priority, show inline warning and block approval until order is clear.

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
- Banner: 1/3 ⚠
- Email: 0/3
- USSD: 0/3

Channel tabs:
- [Push] [Zalo OA] [SMS] [Banner] [Email] [USSD] [+ Thêm kênh]
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
  - T3 SMS Sắp hết data: ⚠ 300/160 ký tự = 2 SMS segment
- SMS must allow content longer than 160 characters. Do not block saving.
- Billing/cost calculation for SMS uses `ceil(character_count / 160)`.
- Show an inline cost panel for long SMS:
  - "Reach SMS dự kiến: ~1,200 KH"
  - "300/160 ký tự = 2 SMS segment"
  - "Chi phí: 1,200 × 2 segment = 2,400 SMS"
  - "Ước tính: ~720,000 VND"

When switching to USSD tab:
- Render the same trigger rows but with USSD-specific content:
  - Plain text/menu textarea only
  - Character counter, e.g. 0/182 ký tự
  - No image upload controls
  - Warning: "USSD không hỗ trợ ảnh; cần kiểm tra tiếng Việt có dấu"
  - Preview: terminal/menu-style USSD screen
- Sample rows:
  - T1 USSD Welcome: ⚠ Chưa cấu hình
  - T2 USSD Chưa cài app: ⚠ Chưa cấu hình
  - T3 USSD Sắp hết data: ⚠ Chưa cấu hình

Template behavior:
- Support choosing approved/reusable templates from Template Library.
- Also allow composing content in campaign for now; campaign approval reviews the full campaign, not only template approval.
- Template Library should feel serious: version, channel support, variable list, usage count.
- Every channel tab must use a 2-column content layout:
  - Left column: content configuration
  - Right column: `XEM TRƯỚC` preview and `SAMPLE PARAMS`
- The right preview must render in the actual visual format of the selected channel, not as generic text.
- Sample params are editable inputs and update preview in realtime.
- Parameter chips are generated from the selected trigger payload. Clicking a chip inserts it into the focused field.
- In OR mode, each trigger message card shows only that trigger's payload parameters.
- In AND mode, there is one shared message per channel and parameter chips combine payload parameters from all selected triggers, grouped by trigger.
- Payload parameter groups:
  - T1 SIM_ACTIVATED: {{ten_kh}}, {{loai_sim}}, {{ngay_kich_hoat}}, {{so_dien_thoai}}
  - T2 NO_APP_INSTALL_24H: {{app_installed}}, {{last_active}}
  - T3 LOW_DATA_BALANCE: {{data_con_lai}}, {{ten_goi}}, {{ngay_het_han}}
- Channel-specific previews:
  - Push: OS notification with app icon, title, body, optional 1:1 thumbnail
  - Zalo OA: brand header + chat bubble + optional image
  - SMS: plain text + character counter + SMS segment/cost
  - Banner: 16:9 image + title + body + CTA
  - Email: subject + header image + rich text body
  - USSD: terminal/menu preview, plain text only
- "+ Thêm kênh" only shows channels not already present as tabs. Existing channels are disabled.

Advanced AND mode rendering:
- Show label: "AND mode — nhiều trigger = 1 tin nhắn chung".
- Channel tabs can include: [Zalo OA (Kênh đầu)] [SMS ×] [USSD ×] [Push ×] [Banner ×] [Email ×] [+ Thêm kênh].
- Do not render T1/T2/T3 separate message cards.
- Render exactly one `Message chung` card for the active channel.
- The shared message card title should say: "Message chung cho 3 trigger".
- Show parameter chips grouped by T1/T2/T3 as listed above.
- Each channel still has different content fields and preview format:
  - Zalo OA: upload image ratio free, rich/plain message body, preview with blue `Viettel Telecom` brand header, image area, and chat bubble content.
  - SMS: no image, plain textarea, `X/160 ký tự`, segment/cost if over 160, preview as `SMS · Viettel` phone-message card.
  - USSD: no image, menu/plain textarea, warning for Vietnamese diacritics, preview as black terminal with green text.
  - Push: image 1:1 upload, title input, body textarea, preview as OS notification with app icon and thumbnail.
  - Banner: image 16:9 upload, title, body, CTA input, preview as 16:9 banner card with CTA button.
  - Email: header image, subject input, rich text body, preview as email client with subject, image, and body.

---

**SECTION 5: Channel Strategy & An toàn**
(card with yellow left border, numbered "5.")

Split this section into two visual blocks: `5A. Channel Strategy` and `5B. An toàn & Chống Spam`.

5A. Channel Strategy:
- Purpose: prioritize lower-cost channels first, SMS last by default.
- The channel list depends on channels already added in Section 4 Message Matrix.
- If Message Matrix has Push, Zalo OA, SMS, Banner, Email, USSD tabs, those channels become available here.
- Build this as an ordered fallback flow with draggable/reorderable cards.

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
- Show estimate: "Ước tính SMS: ~1,200 KH × 2 segment = 2,400 SMS".

Unused channels:
- Show chips: [Banner] [Email] [USSD]
- Button: "+ Thêm vào strategy"
- User can drag chips/cards into the ordered strategy.

Cost-saving checkbox:
- ☑ Không gửi SMS nếu KH đã click/open ở kênh trước

Stop fallback condition checkboxes:
- ☐ Delivered
- ☐ Opened
- ☑ Clicked

Behavior:
- Trigger determines when processing starts.
- Channel Strategy determines which channel sends first/next and after how long.
- User can freely reorder channels via drag-and-drop.
- If a channel is removed from Message Matrix, ask confirmation and remove it from Channel Strategy.
- If a new channel is added in Message Matrix, show it as unused until QTV drags it into the strategy.

5B. An toàn & Chống Spam:
- Render an operational safety card similar to the reference screenshot.

Blackout block:
- Label: "Giờ giới nghiêm (Blackout)"
- Time inputs: [22:00] - [08:00] with clock icons
- Dropdown selected: "Hủy luôn (Discard)"
- Options: Queue đến giờ cho phép / Hủy luôn (Discard) / Gửi ngay nếu kênh cho phép

Frequency cap block:
- Label: "Giới hạn nhận tin"
- Inline inputs: "Gửi [1] lần / [30] ngày"

Push notification limit block:
- Label: "Giới hạn Push Notification"
- Numeric inputs:
  - "Tối đa [3] push / ngày"
  - "Tối đa [10] push / tuần"
  - "Tối đa [30] push / tháng"
- Checked checkbox: "Áp dụng theo từng thuê bao"
- If Push is not present in Section 4 Message Matrix, render this block disabled with note: "Chỉ áp dụng khi campaign có kênh Push".

DNC block:
- Label: "Blacklist / Suppression"
- Checked checkbox: "Check blacklist DNC toàn hệ thống"
- Green status: "Enabled"
- Treat this as global/system-level suppression.

Campaign blacklist block:
- Radio options:
  - "Không dùng blacklist riêng" selected by default
  - "Dùng blacklist riêng cho campaign này"
- When "Không dùng blacklist riêng" is selected, hide the selected file dropdown, [Xem danh sách], [Thay tệp], [Upload tệp mới], preview area, and result text.
- Only when "Dùng blacklist riêng cho campaign này" is selected, show:
  - Selected campaign blacklist dropdown: "BL_ESIM_Q2_2026 · 320 KH · cập nhật 12/05/2026 09:00"
  - Buttons: [Xem danh sách] [Thay tệp] [Upload tệp mới]
  - Result text: "Sau khi áp blacklist campaign: loại 320 KH"
- Explain visually that campaign blacklist excludes customers only from this campaign.

Whitelist block:
- Radio options:
  - "Không dùng whitelist" selected by default
  - "Chỉ gửi cho tệp whitelist được chọn"
- When "Không dùng whitelist" is selected, hide the selected file dropdown, [Xem danh sách], [Thay tệp], [Upload tệp mới], preview area, and reach result text.
- Only when "Chỉ gửi cho tệp whitelist được chọn" is selected, show:
  - Selected whitelist dropdown: "WL_ESIM_Q2_2026 · 1,250 KH · cập nhật 12/05/2026 08:00"
  - Buttons: [Xem danh sách] [Thay tệp] [Upload tệp mới]
  - Result text: "Sau khi áp whitelist: reach còn ~1,120 KH"
- Explain visually that whitelist narrows Audience; only customers in both Audience and whitelist receive messages.

Whitelist modal:
- Title: "Chọn tệp whitelist"
- Search input: "Tìm tên tệp whitelist..."
- Radio list:
  - WL_TEST_INTERNAL | 35 KH | 10/05/2026 14:00
  - WL_ESIM_Q2_2026 | 1,250 KH | 12/05/2026 08:00 selected
  - WL_HIGH_ARPU_DATA | 860 KH | 11/05/2026 09:30
- Preview area:
  - "Số hợp lệ: 1,250 · Trùng: 12 · Sai định dạng: 3"
  - Show masked phone rows
- Footer buttons: [Hủy] [Áp dụng whitelist]

Campaign blacklist modal:
- Title: "Chọn tệp blacklist campaign"
- Search input: "Tìm tên tệp blacklist..."
- Radio list:
  - BL_TEST_INTERNAL | 40 KH | 10/05/2026 15:00
  - BL_ESIM_Q2_2026 | 320 KH | 12/05/2026 09:00 selected
  - BL_COMPLAINT_USERS | 180 KH | 11/05/2026 10:30
- Preview area:
  - "Số hợp lệ: 320 · Trùng: 8 · Sai định dạng: 2"
  - Show masked phone rows
- Footer buttons: [Hủy] [Áp dụng blacklist campaign]

Safety behavior:
- Safety rules determine whether the planned send can happen now or must be queued/blocked/discarded.
- Push notification limits control the maximum number of Push messages each subscriber can receive per day, week, and month.
- If a subscriber exceeds the Push quota, do not send Push to that subscriber and continue Channel Strategy fallback to the next eligible channel.
- Global DNC/system blacklist is applied before campaign-specific blacklist.
- Campaign blacklist is optional and applies only to the current campaign.
- If campaign blacklist mode is selected but no file is selected, show blocking validation.
- Final reach formula: Audience ban đầu → trừ Global DNC/blacklist hệ thống → trừ Campaign blacklist → giao với Whitelist nếu bật whitelist.
- If whitelist mode is selected but no file is selected, show blocking validation.
- If DNC is unchecked, show confirm dialog because this is risky.
- Updating whitelist recalculates reach and sticky summary immediately.

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

Do not render Section 7 Review & Submit. Approval actions stay in the fixed header only.

#### RIGHT STICKY COLUMN

Build a compact sticky panel that remains visible while scrolling.

Campaign Summary:
- Trigger: Advanced · OR · 3 triggers
- Audience: 2 segments · ~6,800 KH
- Message:
  - Push: 3/3 ✓
  - Zalo: 3/3 ✓
  - SMS: 2/3 ⚠
  - Banner: 1/3 ⚠
  - Email: 0/3
  - USSD: 0/3
- Channel Strategy: Push → Zalo → SMS
- Stop fallback: Clicked
- Safety:
  - DNC: Enabled
  - Whitelist: WL_ESIM_Q2_2026
  - Reach sau whitelist: ~1,120 KH
- Risk / Cost:
  - Estimated SMS: 2,400 segment
  - Estimated cost: ~720,000 VND
- Approval: Draft, Admin duyệt để Active

Interactions:
- Clicking `SMS: 2/3` scrolls to the missing SMS message.
- Clicking `Banner: 1/3` or `USSD: 0/3` scrolls to the corresponding channel tab.
- Cost/SMS risk updates when changing segment, whitelist, channel strategy, fallback condition, SMS character count, or message completion.

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

### SCREEN 5: Trigger Management — UPDATED v7

Header: "Trigger Management" + primary button "+ Thêm Trigger"

**FILTER BAR:**
- Dropdown: "Trạng thái" (Tất cả / Active / Paused / Testing)
- Dropdown: "Loại thông báo" (Tất cả / Realtime / Near Realtime / Offline)
- These two filters sit side by side in the same row

Do not render lifecycle-stage filter, priority info banner, campaign linked filter, or any trigger master priority UI.

**TABLE — 6 columns:**

Columns: Trigger Code | Tên Trigger | Event Source | Loại thông báo | Trạng thái | Hành động

Column details:
- Trigger Code: uppercase code such as SIM_ACTIVATED, NO_APP_INSTALL_24H, LOW_DATA_BALANCE
- Tên Trigger: text
- Event Source: "BSS SIM_ACTIVATED" / "CRM NO_APP_INSTALL_24H" / "OCS LOW_DATA_BALANCE" etc — show source system as small badge (BSS=blue, CRM=violet, OCS=green) + event name
- Loại thông báo: badge with Realtime / Near Realtime / Offline
- Trạng thái: pill badge
  - Active = green dot + "Active"
  - Paused = yellow + "Paused"
  - Testing = gray + "Testing"
- Hành động: [Sửa] [Tắt] for Active triggers, [Sửa] [Bật] for Paused/Testing triggers

Sample data rows:
| SIM_ACTIVATED | Kích hoạt SIM thành công | BSS SIM_ACTIVATED | Realtime | Active | [Sửa][Tắt] |
| NO_APP_INSTALL_24H | Chưa cài app sau 24h | CRM NO_APP_INSTALL_24H | Near Realtime | Active | [Sửa][Tắt] |
| LOW_DATA_BALANCE | Data còn dưới 100MB | OCS LOW_DATA_BALANCE | Offline | Active | [Sửa][Tắt] |
| LOC_TRAVEL_PROV | Đến tỉnh du lịch | OCS LOC_TRAVEL_PROV | Realtime | Active | [Sửa][Tắt] |
| USAGE_DROP_40PCT | Usage giảm >40% baseline | OCS USAGE_DROP_40PCT | Offline | Testing | [Sửa][Bật] |
| SIM_NO_TXN_30D | SIM ngừng giao dịch >30 ngày | BSS SIM_NO_TXN_30D | Offline | Paused | [Sửa][Bật] |

**[Tắt/Bật] behavior:** show confirm dialog before toggling.

---

**MODAL: Thêm / Sửa Trigger — UPDATED v7**

Title: "Thêm Trigger Mới" (or "Sửa Trigger" when editing)

Fields:
- Input: "Trigger code *"
  - Example value: "SIM_ACTIVATED"
  - Validation: uppercase letters, numbers, and underscores only
  - Must be unique
- Input: "Tên trigger *" (full width)
- Dropdown: "Event source *" — options: BSS / OCS / CRM / Other
- Textarea: "Mô tả" (optional)
- Dropdown: "Loại thông báo *" — options: Realtime / Near Realtime / Offline

Do not render fields "Giai đoạn vòng đời" or "Độ ưu tiên" in the trigger modal. Trigger priority is configured only inside Campaign Builder at campaign level.

Payload section:
- Label: "Tham số payload"
- Dynamic rows: [Tên tham số *] [Source: BSS / OCS / CRM ▾] [✕]
- Button: "+ Thêm tham số" — appends new row

Toggle: "Trạng thái" → Active / Inactive

Footer: [Hủy] | [Lưu] (primary)

On [Lưu]: close modal + update table row + show toast "Đã lưu trigger ✓"

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

**NEW in v7 — Trigger Management specific:**
- Trigger Code column uses monospace uppercase code chips.
- Loại thông báo badges: Realtime=green, Near Realtime=blue, Offline=gray.
- Do not show lifecycle stage badges, trigger master priority stars, priority info banner, or Campaign Linked column/modal.

**NEW in v5 — Section 4 Phân khúc specific:**
- Segment badges (Part A): bg-blue-100 text-blue-700 rounded-full px-2 py-1 text-sm with × button
- "+ Chọn phân khúc ▾": secondary outlined button
- Part A / Part B separator: thin gray horizontal rule (border-gray-200)
- Part B subtext "Kết hợp bằng AND": text-gray-500 text-xs italic
- Condition builder rows: same style as v4 — inline dropdowns + remove button
