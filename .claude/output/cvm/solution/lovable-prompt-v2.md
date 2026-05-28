# Lovable Prompt — CVM System v2
> Cập nhật: Campaign Builder redesign từ stepper → single page 3-column layout
> Bổ sung: Template Management

---

```
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

Row actions based on status:
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

### SCREEN 3: Campaign Builder (Single Page — 3 Column Layout)

This is the most important screen. DO NOT use a stepper/wizard. Everything is on one page in 3 columns.

#### Overall layout:

```
[FIXED HEADER]
[LEFT COL 30%] | [MIDDLE COL 40%] | [RIGHT COL 30%]
  scrollable   |    scrollable    |    sticky
```

#### Fixed Header:

- Inline editable campaign name input (placeholder: "Tên campaign...")
- Date range picker: Từ ngày → Đến ngày
- Status badge: ● Draft (auto-update)
- Buttons: [Lưu Draft] (secondary) | [Submit →] (primary)
- Back link: "← Danh sách Campaign"

---

#### Left Column — Trigger & Segment:

**TRIGGER section:**
- Label: "TRIGGER"
- Logic dropdown: OR / AND
  - OR tooltip: "1 trigger xảy ra là kích hoạt campaign"
  - AND tooltip: "Tất cả trigger phải xảy ra mới kích hoạt"
- Selected triggers as removable tags:
  - "SIM_ACTIVATE — Kích hoạt SIM mới [✕]"
  - "TOP_UP_FAIL — Nạp thẻ thất bại [✕]"
- Button: "+ Thêm trigger"
  - Click → dropdown with search input + checkbox list:
    - DATA_EMPTY — Hết dung lượng
    - CALL_DROP — Rớt cuộc gọi
    - ROAMING_ON — Bật roaming
    - PACKAGE_EXP — Hết hạn gói
  - Click item → add tag, close dropdown

**Divider**

**SEGMENT section:**
- Label: "SEGMENT"
- Condition rows (removable):
  - [Loại thiết bị ▾] [= ▾] [Android ▾] [✕]
  - [Số ngày kể từ kích hoạt SIM ▾] [≥ ▾] [30] ngày [✕]
  - [Data còn lại ▾] [≤ ▾] [100] MB [✕]
- Button: "+ Thêm điều kiện" → appends new empty row

Attribute groups for segment dropdown:
  Định danh: Loại SIM, Trạng thái SIM
  Hành vi ứng dụng: Đã cài app, Số ngày kể từ cài app, Đã đăng nhập app, Số ngày kể từ kích hoạt SIM
  Sử dụng: Data còn lại (MB/GB), Lưu lượng data hôm nay (MB), Số lần gia hạn liên tiếp (lần), Số lần cuộc gọi thất bại (lần)
  Tài chính: Số dư tài khoản (VNĐ), Số lần nạp tiền (lần)
  Thiết bị: Loại thiết bị (Android/iOS/Other), Cài đặt APN (Có/Chưa)
  Nhân khẩu học: Ngày sinh nhật, Ngày kỷ niệm mua SIM, Nghề nghiệp
  Gói cước: Tên gói hiện tại, Số lần đăng ký gói (lần), Ngày hết hạn gói (ngày)

**Divider**

Estimated reach: "~12,400 khách hàng đủ điều kiện ↻" (updates on any segment change)

---

#### Middle Column — Channels & Messages + Send Config:

**KÊNH & TIN NHẮN section:**
- Label: "KÊNH & TIN NHẮN"

Each channel is a collapsible card with drag handle:
- Drag handle [⠿] on left for reordering
- Channel name + fallback delay input (or "kênh đầu" / "kênh cuối" label)
- [✕] remove button
- Expand/collapse toggle

Inside each expanded channel card:
- Radio: ○ Chọn template có sẵn / ● Soạn mới
- If "Chọn template": searchable dropdown of saved templates → selecting prefills textarea (still editable)
- If "Soạn mới": textarea for message content
- Dynamic parameters as clickable chips below textarea:
  - [{{ten_kh}}] [{{so_du}}] [{{data_con_lai}}] [{{ten_goi}}] [{{ngay_het_han}}]
  - Click chip → insert at cursor position in textarea
- SMS channel only: character counter "X/160 ký tự" — turns red when > 160

Default pre-filled channels (in order):
1. USSD — (kênh đầu, no fallback input)
2. Zalo OA — Fallback sau: 5 phút
3. SMS — Fallback sau: 10 phút
4. Banner App/Web — Fallback sau: 15 phút
5. Push Notification — Fallback sau: 5 phút
6. Email — (kênh cuối, no fallback input)

Button: "+ Thêm kênh"
- Dropdown shows only unselected channels: USSD / Zalo OA / SMS / Banner App/Web / Push Notification / Email

**Divider**

**CẤU HÌNH GỬI section:**
- Label: "CẤU HÌNH GỬI"
- Send type radio:
  - ○ Realtime → show note: ⚠ "Gửi ngay, không áp dụng blackout window"
  - ● Near Realtime → show field: "Gửi sau [10] phút" + note about blackout
  - ○ Offline → show field: "Gửi lúc [09:00] mỗi ngày" + note about blackout
- Input: "Cooldown: [30] phút"
- Time range: "Khung giờ: [08:00] → [22:00]"

---

#### Right Column — Preview (sticky, does not scroll):

- Label: "PREVIEW"
- Channel selector dropdown: [Zalo OA ▾] (shows all channels added in middle col)
- Preview box: renders message content with sample params filled in, formatted per channel
- Sample params (editable inputs):
  - ten_kh = [Nguyễn Văn A]
  - so_du = [45,000 VNĐ]
  - data_con_lai = [1.2 GB]
  - ten_goi = [D50]
- Preview updates realtime as user types in middle column
- SMS warning: "⚠ 142/160 ký tự" shown below preview when SMS selected

---

### SCREEN 4: Template Management

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

### SCREEN 5: Trigger Management

Header: "Trigger Management" + "+ Thêm Trigger"

Filters: Search + Status dropdown

Table: Tên Trigger, Event Source, Trạng thái, Hành động
Sample rows:
- SIM_ACTIVATE | BSS | ● Active | [Sửa] [Tắt]
- TOP_UP_FAIL | BSS | ● Active | [Sửa] [Tắt]
- DATA_EMPTY | OCS | ● Active | [Sửa] [Tắt]
- CALL_DROP | OCS | ○ Inactive | [Sửa] [Bật]

Click [Sửa] or [+ Thêm Trigger] → modal:
- Input: Tên trigger *
- Dropdown: Event source (BSS / OCS / Other)
- Textarea: Mô tả
- Toggle: Active / Inactive
- Buttons: [Hủy] | [Lưu]

On save: close modal + update table row + toast "Đã lưu thành công ✓"

Click [Tắt/Bật]:
- Confirm dialog: "Bạn có chắc muốn tắt trigger này? Campaign đang dùng trigger này sẽ không nhận event mới."
- [Hủy] [Xác nhận] → badge updates immediately

---

### SCREEN 6A: Customer List

Header: "Khách hàng"
Filters: Search phone/ID, Loại SIM dropdown, Gói cước dropdown, [Lọc]
Table: Số điện thoại, Loại SIM, Gói cước, Ngày kích hoạt, [Xem]
Click row or [Xem] → Customer 360

---

### SCREEN 6B: Customer 360

Breadcrumb: "Khách hàng / 0987 xxx 001"

Section 1 — Thông tin khách hàng:
- Số ĐT, Loại SIM, Trạng thái, Gói + hết hạn, Ngày KH SIM, Thiết bị, Cài app, Đăng nhập app

Section 2 — Thống kê sử dụng (2-col table):
- Data còn lại, Lưu lượng hôm nay, Số dư, Số lần nạp, Số lần gia hạn, Cuộc gọi thất bại, Ngày sinh nhật, Nghề nghiệp

Section 3 — Throttling Status:
- Tin nhắn hôm nay: 2/3, Cooldown: Không, Blacklist: Không

Section 4 — Lịch sử nhận tin nhắn:
- Columns: Thời gian, Campaign, Kênh, Trạng thái
- Status: ✓ Delivered / ✕ Failed / ✓ Fallback

---

### SCREEN 7: Report

Filters: Campaign dropdown + date range + [Lọc]

KPI cards: Đã gửi (48,320) | Delivered (46,891) | Failed (1,429) | Fallback Rate (2.3%)

Bar chart: Tin nhắn gửi theo ngày

Horizontal bar chart — Breakdown theo kênh:
- USSD: 45%
- Zalo OA: 30%
- SMS: 14%
- Banner App/Web: 7%
- Push Notification: 3%
- Email: 1%

---

## DESIGN GUIDELINES
- Color: white background, #1E40AF blue primary, gray-100 sidebar
- 3-column campaign builder: left/middle scroll independently, right column sticky
- Collapsible channel cards in middle column with drag handle
- Clickable parameter chips that insert at cursor position
- Preview panel updates in realtime as user types
- Status badges: rounded pill shape
- Tables: striped rows, hover highlight, cursor pointer on clickable rows
- Inline editable fields: show edit border on focus only
- Toast: bottom right, auto-dismiss 3s
- Confirm dialog: centered modal overlay
- Empty states: placeholder + CTA
- Desktop-first, min-width 1440px (3-column layout needs more space)
- shadcn/ui component style
```
