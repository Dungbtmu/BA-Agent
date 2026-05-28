# Lovable Prompt — CVM System v4
> Cập nhật: Layout 2 cột (học từ giao diện tham khảo) + giữ điểm mạnh v3

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

Table columns: Campaign Name, Trigger Count, Trạng thái, Hiệu lực, Actions

Status badges (pill shape):
- Active = green dot
- Draft = gray
- Paused = yellow
- Pending Review = orange
- Ended = muted red

Row actions:
- Active → [Sửa] [Dừng]
- Draft → [Sửa] [Xóa]
- Paused → [Sửa] [Resume]
- Pending → [Xem]
- Ended → [Xem]

Sample rows:
- Chào mừng SIM mới | 2 | Active | 04/05–18/05
- Nạp thẻ lỗi | 1 | Active | 04/05–18/05
- Ưu đãi sinh nhật | 1 | Draft | Chưa set
- Hết hạn gói data | 3 | Paused | 01/05–30/05

Pagination: < 1 2 3 >

---

### SCREEN 3: Campaign Builder (2-column layout)

DO NOT use a stepper/wizard. Use a 2-column layout as described below.

#### Overall layout:
```
[FIXED HEADER]
[LEFT COL ~65% scrollable] | [RIGHT COL ~35% sticky]
```

Right column contains ONLY: Section 4 (Phân khúc) + Section 5 (An toàn).
Preview is INSIDE each channel tab in Section 2, NOT in the right column.

---

#### Fixed Header:
- Back link: "← Danh sách Campaign"
- Inline editable campaign name (placeholder: "Tên chiến dịch...")
- Status badge: ● Draft (auto-update)
- Buttons right side: [Lưu Nháp] (secondary) | [Gửi Duyệt →] (primary blue)

---

#### LEFT COLUMN (scrollable) — 3 numbered sections:

---

**SECTION 1: Thông tin & Trigger**
(card with yellow left border, numbered "1.")

- Input: "Tên chiến dịch *" (full width)
- Input: "Mã kịch bản (ID)" (full width)
- Date range: "Thời gian hiệu lực" → [Từ ngày____] [Đến ngày____]

Divider

- Label: "Chọn Trigger kích hoạt"
- Dropdown: searchable trigger list:
  - BSS: Kích hoạt SIM thành công
  - BSS: Nạp thẻ thất bại
  - OCS: Hết dung lượng data
  - OCS: Rớt cuộc gọi
  - OCS: Hết hạn gói

Each selected trigger renders as a card below the dropdown:
- Card header: [★ or blank] [Trigger name — source badge BSS/OCS] [✕ remove]
  - ★ = primary trigger (trigger chính)
- Card body (always visible, not collapsible):
  - Label: "Tham số:"
  - Parameter tags with source badges:
    - SIM_ACTIVATE: [ten_kh BSS] [loai_sim BSS] [ngay_kich_hoat BSS] [so_dien_thoai BSS]
    - TOP_UP_FAIL: [so_lan_nap_sai BSS] [so_tien_nap BSS] [ten_kh BSS]
    - DATA_EMPTY: [data_con_lai OCS] [ten_goi OCS] [ngay_het_han OCS]
- Card has light yellow/amber background to stand out

Button: "+ Thêm trigger" (only shown if not all triggers selected)

"Logic kết hợp" toggle (shown only when 2+ triggers selected):
- [OR (Hoặc)] [AND (Và)] — toggle button style, OR selected by default

"Trigger chính" dropdown (shown only when 2+ triggers selected):
- Label: "Trigger chính:" with ⓘ tooltip: "Tham số trong tin nhắn sẽ lấy từ trigger này"
- Dropdown listing selected triggers

Behavior:
- 1 trigger only → auto primary, hide logic toggle + trigger chính dropdown
- Change primary → ★ badge moves, parameter chips in section 2 update
- Remove primary trigger → auto-select next as primary

---

**SECTION 2: Kênh & Tin nhắn**
(card with yellow left border, numbered "2.")

Row 1:
- Dropdown: "Ngày gửi (T = ngày phát sinh trigger)"
  - Options: Ngày T — Gửi ngay / Ngày T+1 / Ngày T+2 / Ngày T+3
- Time range: "Khung giờ gửi tin nhắn" → Từ [08:00] đến [20:00]

Divider

Channel tabs: [Zalo OA] [SMS] [USSD] [Banner] [Push] [Email] [+ Thêm kênh]
- Tabs are closable (✕ on each tab except when only 1 tab remains)
- "+ Thêm kênh" opens dropdown showing only unselected channels
- First tab = primary channel (label: "Fallback: kênh đầu"), subsequent tabs show fallback delay input

Inside each channel tab — IMPORTANT: 2-column layout side by side:
```
[LEFT HALF: soạn nội dung] | [RIGHT HALF: preview realtime]
```

LEFT HALF of tab (input area):

For non-first tabs:
- Small input row at top: "Fallback sau: [5] phút"

- Radio: ○ Chọn template có sẵn / ● Soạn mới
- If "Chọn template": searchable dropdown → prefills content (editable)

IMAGE UPLOAD (optional) — Zalo OA, Banner, Push, Email only:
- Upload area: [🖼 Tải ảnh lên] | [Chọn từ thư viện]
- Before upload: grey placeholder box with 🖼 icon + dashed border
- After upload: thumbnail + [Xóa] [Đổi] buttons
- Ratio label: Zalo OA "Tự do" / Banner "16:9" / Push "1:1" / Email "Banner ngang"
- SMS / USSD: show "(Không hỗ trợ hình ảnh)"

Parameter chips:
- Label: "Tham số (tên_trigger_chính):"
- Chips from primary trigger payload only — click to insert at cursor
- Invalid params in content → red highlight + tooltip

Content fields:
- Zalo OA: textarea, unlimited
- SMS: textarea + counter "0/160" red > 160
- USSD: textarea + warning "⚠ Không hỗ trợ tiếng Việt có dấu"
- Banner: input Tiêu đề 0/50 + textarea Nội dung + input CTA label
- Push: input Tiêu đề 0/65 + textarea Nội dung 0/240
- Email: input Subject 0/100 + rich text editor

RIGHT HALF of tab (preview area — updates realtime):

Label: "Xem trước"

Preview format per channel:
- Zalo OA: chat bubble, brand header, image at top (placeholder if none), message text
- SMS: phone SMS bubble, plain text, no image, counter "X/160"
- USSD: monospace terminal, plain text + menu numbers, no image, dấu warning
- Banner: card — 16:9 image top (placeholder if none), title, body, CTA button
- Push: OS notification bar — app icon, title, body, 1:1 thumbnail right (placeholder if none), counters
- Email: email client — subject, header image (placeholder if none), body

Sample params (editable, below preview box):
- Only params from primary trigger
- ten_kh = [Nguyễn Văn A], loai_sim = [eSIM], ngay_kich_hoat = [01/04/2026]...
- Edit any param → preview re-renders immediately
- When image uploaded → preview updates immediately

Media Library Modal (opens on "Chọn từ thư viện"):
- Header: "Thư viện ảnh" + [Tải ảnh mới lên]
- Search bar + 4-column image grid
- Click → blue border + checkmark selected state
- Footer: [Hủy] | [Chọn ảnh này]

---

**SECTION 3: Cấu hình gửi**
(card with yellow left border, numbered "3.")

- Label: "Loại gửi"
- Listbox dropdown (single select):
  - Realtime
  - Near Realtime
  - Offline

Conditional fields shown below the listbox based on selection:

If "Realtime" selected:
- Show info note: ⚠ "Gửi ngay lập tức, không áp dụng blackout window"
- No extra fields

If "Near Realtime" selected:
- Input: "Gửi sau [10] phút kể từ trigger"
- Info note: "Áp dụng blackout window. Nếu rơi vào khung giờ cấm → delay đến đầu khung giờ hôm sau"

If "Offline" selected:
- Time picker: "Gửi lúc [09:00] mỗi ngày"
- Info note: "Áp dụng blackout window"

---

#### RIGHT COLUMN (sticky) — Section 4 + Section 5 only:

No preview panel in right column. Preview is inside each channel tab in Section 2.

---

**SECTION 4: Phân khúc**
(card, numbered "4.")

- Dropdown: "Chọn Segment động" (User Gen Z 18-25 / All Users / Custom...)
- Label: "Hoặc tự định nghĩa điều kiện:"
- Condition rows (removable):
  - [Loại thiết bị ▾] [= ▾] [Android ▾] [✕]
  - [Số ngày kể từ kích hoạt SIM ▾] [≥ ▾] [30] ngày [✕]
  - [Data còn lại ▾] [≤ ▾] [100] MB [✕]
- Button: "+ Thêm điều kiện" → appends new empty row
- Input: "Control Group: [5] %" with tooltip "Giữ lại X% KH không gửi để đo Uplift"
- Estimated reach: "~12,400 khách hàng đủ điều kiện ↻"

Attribute groups for segment dropdown:
  Định danh: Loại SIM, Trạng thái SIM
  Hành vi ứng dụng: Đã cài app, Số ngày kể từ cài app, Đã đăng nhập app, Số ngày kể từ kích hoạt SIM
  Sử dụng: Data còn lại (MB/GB), Lưu lượng data hôm nay (MB), Số lần gia hạn liên tiếp (lần), Số lần cuộc gọi thất bại (lần)
  Tài chính: Số dư tài khoản (VNĐ), Số lần nạp tiền (lần)
  Thiết bị: Loại thiết bị (Android/iOS/Other), Cài đặt APN (Có/Chưa)
  Nhân khẩu học: Ngày sinh nhật, Ngày kỷ niệm mua SIM, Nghề nghiệp
  Gói cước: Tên gói hiện tại, Số lần đăng ký gói (lần), Ngày hết hạn gói (ngày)

---

**SECTION 5: An toàn & Chống Spam**
(card, numbered "5.")

- Label: "Giờ giới nghiêm (Blackout)"
- Time range: [22:00] - [08:00]
- Dropdown: Xử lý khi rơi vào blackout: "Hủy luôn (Discard)" / "Delay đến đầu khung giờ"

- Label: "Giới hạn nhận tin"
- Inputs: Gửi [1] lần / [30] ngày

- Checkbox: ☑ Kiểm tra Blacklist DNC

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

## SCREEN 4: Template Management

Header: "Template" + primary button "+ Tạo Template"

Filters:
- Search: "Tìm template..."
- Dropdown: Trạng thái (Tất cả / Active / Inactive)

Table columns: Tên Template, Kênh hỗ trợ, Số lần dùng, Hành động

Sample rows:
- Chào mừng SIM | Zalo, SMS | 3 lần | [Sửa] [Clone]
- Nạp thẻ hướng dẫn | SMS, USSD | 5 lần | [Sửa] [Clone]
- Sinh nhật KH | Zalo, Email | 2 lần | [Sửa] [Clone]

Click [+ Tạo Template] or [Sửa] → navigate to Template Editor page

---

### SCREEN 4B: Template Editor (2-column layout)

Fixed header:
- Inline editable template name
- Status toggle: Active / Inactive
- Button: [Lưu Template]
- Back link: "← Template"

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

---

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

---

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

### SCREEN 5: Trigger Management

Header: "Trigger Management" + "+ Thêm Trigger"
Table: Tên Trigger, Event Source, Trạng thái, [Sửa] [Tắt/Bật]

Modal (add/edit):
- Input: Tên trigger *
- Dropdown: Event source (BSS / OCS / Other)
- Textarea: Mô tả
- Payload section: list of [Tên tham số *] [BSS / OCS ▾] [✕] rows + "+ Thêm tham số"
- Toggle: Active / Inactive
- [Hủy] | [Lưu] → close modal + update table + toast ✓

[Tắt/Bật]: confirm dialog before toggling

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
