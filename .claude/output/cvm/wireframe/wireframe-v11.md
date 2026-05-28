# Wireframe CVM System — v11

---

## LOVABLE BUILD INSTRUCTIONS
> Dán toàn bộ file này vào Lovable. Đọc section này TRƯỚC khi đọc bất kỳ screen nào.

### Tech stack
- React + TypeScript + Tailwind CSS + shadcn/ui
- React Router v6 cho navigation
- Recharts cho charts
- Desktop-first, min-width 1440px, ngôn ngữ UI: tiếng Việt

### Design system

```
Colors:
  --bg-page:       #F8FAFC
  --bg-card:       #FFFFFF
  --border:        #E2E8F0
  --text-primary:  #1E293B
  --text-muted:    #64748B
  --brand:         #3B82F6   ← primary button, active nav
  --success:       #22C55E   ← Active status
  --warning:       #EAB308   ← Pending status
  --paused:        #F97316   ← Paused status
  --draft:         #94A3B8   ← Draft status
  --danger:        #EF4444   ← error, kill switch
  --info-bg:       #EFF6FF   ← estimate box, info note
  --amber-accent:  #F59E0B   ← left border card Campaign Builder sections

Typography: Inter. 12px label, 13px table, 14px body, 16px section title, 20px page title.

Cards: bg-white rounded-lg border border-slate-200 shadow-sm p-4
Section cards (Campaign Builder only): bg-white rounded-lg border border-slate-200 shadow-sm p-5 border-l-4 border-l-amber-400

Status chips (pill): rounded-full px-2.5 py-0.5 text-xs font-medium
  Active:  bg-green-100  text-green-700
  Draft:   bg-slate-100  text-slate-600
  Pending: bg-yellow-100 text-yellow-700
  Paused:  bg-orange-100 text-orange-700

Trigger chips: bg-amber-100 text-amber-800 rounded px-2 py-0.5 text-xs font-mono
PARAMS chips:  bg-blue-100  text-blue-700  rounded-full px-2 py-0.5 text-xs cursor-pointer hover:bg-blue-200

Toasts: fixed bottom-4 right-4, stack vertically, auto-dismiss 3s
  success: border-l-4 border-green-500
  error:   border-l-4 border-red-500
  warning: border-l-4 border-orange-500

Confirm dialog: modal centered, backdrop bg-black/40, max-w-sm, rounded-xl shadow-xl
  Always has [Hủy] (outline) + action button (color varies per action)
  Press Escape or click backdrop = same as [Hủy]
```

### Global layout (ALL screens)

```jsx
// Root layout — applies to every screen
<div className="flex h-screen overflow-hidden">
  {/* Left sidebar — fixed, never scrolls */}
  <aside className="w-56 flex-shrink-0 bg-slate-50 border-r border-slate-200 flex flex-col fixed inset-y-0 left-0 z-20">
    <div className="p-4 font-bold text-lg">CVM</div>
    <nav>{/* nav items */}</nav>
  </aside>

  {/* Main area */}
  <div className="flex-1 ml-56 flex flex-col">
    {/* Top header — fixed */}
    <header className="h-14 border-b bg-white flex items-center px-6 fixed top-0 right-0 left-56 z-10">
      {/* breadcrumb left, bell+avatar right */}
    </header>

    {/* Page content — scrollable */}
    <main className="mt-14 flex-1 overflow-y-auto bg-slate-50 p-6">
      {/* screen content here */}
    </main>
  </div>
</div>
```

Nav items: Dashboard → /dashboard | Campaign → /campaigns | Template → /templates | Trigger → /triggers | Khách hàng → /customers | Blacklist → /blacklist | Report → /report | Admin → /admin | Settings → /settings
Active nav item: `bg-blue-50 text-blue-700 font-medium border-l-2 border-blue-600`

### Campaign Builder — layout 2 cột (CRITICAL — KHÔNG được làm 1 cột)

```jsx
// Screen /campaigns/new và /campaigns/:id/edit
// PHẢI dùng layout này — KHÔNG làm single column

<div className="flex flex-col h-full">
  {/* Sub-header fixed bên trong main */}
  <div className="sticky top-0 z-10 bg-white border-b px-6 py-3 flex items-center justify-between">
    {/* breadcrumb + campaign name + status + buttons */}
  </div>

  {/* Body: 2 cột */}
  <div className="flex flex-1 overflow-hidden">
    {/* LEFT: scroll độc lập, chứa S1 S2 S4 S6 */}
    <div className="w-[60%] overflow-y-auto p-6 space-y-6">
      {/* Section 1: Thông tin Campaign */}
      {/* Section 2: Trigger & Logic */}
      {/* Section 4: Message Matrix */}
      {/* Section 6: An toàn */}
    </div>

    {/* RIGHT: sticky, KHÔNG scroll, chứa Summary + Kênh & Lịch gửi + S3 */}
    <div className="w-[40%] border-l bg-slate-50 overflow-y-auto p-6 space-y-4">
      {/* Tóm tắt nhanh campaign — luôn hiển thị đầu cột phải */}
      {/* Kênh & Lịch gửi — thay thế Section 5 Channel Strategy cũ */}
      {/* Section 3: Audience / Phân khúc */}
    </div>
  </div>
</div>
```

### Message Matrix — layout 2 cột + Audience Variant dạng tab (CRITICAL)

```jsx
// Mỗi trigger message card trong Section 4 PHẢI có layout này
// Audience Variant dùng tab ngang — KHÔNG xếp chồng dọc
// Preview on-demand (click [↻ Xem trước]) — KHÔNG cần realtime liên tục

<div className="border rounded-lg overflow-hidden">
  <div className="bg-slate-50 px-4 py-2 border-b font-medium">
    ✓ T1 · SIM_ACTIVATED
    {/* PARAMS chips hiển thị ở header card */}
  </div>
  {/* Tab Audience Variant — ngang */}
  <div className="flex border-b px-4 py-2 gap-2">
    <button className="tab active">Tất cả (dự phòng)</button>
    <button className="tab">VIP</button>
    <button>+ Biến thể đối tượng</button>
  </div>
  <div className="grid grid-cols-[55%_45%]">  {/* 2 cột cố định */}
    <div className="p-4 border-r space-y-3">
      {/* LEFT: image upload, template, title, body của tab đang active */}
    </div>
    <div className="p-4 bg-slate-50">
      {/* RIGHT: channel mockup preview + sample params + nút [↻ Xem trước] */}
    </div>
  </div>
</div>
```

### Section 6 Blackout — CHỈ 2 options (CRITICAL)

```jsx
// Dropdown xử lý Blackout có ĐÚNG 2 options, KHÔNG thêm option nào khác:
const blackoutOptions = [
  { value: 'discard',  label: 'Hủy luôn' },
  { value: 'delay',   label: 'Hoãn đến đầu khung giờ' },
]
// KHÔNG có: "Queue đến giờ cho phép", "Gửi ngay nếu kênh cho phép"
```

### AND mode — ẩn [+ Biến thể đối tượng] (CRITICAL)

```jsx
// Trong Section 4 Message Matrix:
// - OR mode: hiển thị tab Audience Variant + button [+ Biến thể đối tượng] trong mỗi trigger card
// - AND mode: ẩn HOÀN TOÀN tab variant và button này (display:none, không chỉ disabled)
{triggerLogic === 'OR' && (
  <>
    <VariantTabs />
    <button>+ Biến thể đối tượng</button>
  </>
)}

// Khi người dùng chọn AND (nếu đã có variant):
// → mở confirm dialog trước khi switch
// → confirm → xóa tất cả variant (chỉ giữ tab "Tất cả"), chuyển AND
// → hủy → giữ nguyên OR
```

### Navigation map

```
/dashboard              → Screen 1: Dashboard
/campaigns              → Screen 2: Campaign List
/campaigns/new          → Screen 3: Campaign Builder (empty)
/campaigns/:id/edit     → Screen 3: Campaign Builder (pre-filled)
/campaigns/:id/detail   → Screen 2B: Campaign Detail View
/templates              → Screen 4A: Template List
/templates/new          → Screen 4B: Template Editor (empty)
/templates/:id          → Screen 4B: Template Editor (pre-filled)
/triggers               → Screen 5: Trigger Management
/blacklist              → Screen 6: Blacklist Management
/customers              → Screen 7: Customer List
/customers/:phone/360   → Screen 8: Customer 360
/report                 → Screen 9: Report/Analytics
/settings               → Screen Settings: Cấu hình hệ thống (Frequency Cap, Phân quyền)
/admin                  → Screen Admin: Duyệt campaign
```

---

## Thay đổi so với v10 (v11)

| Thành phần | v10 | v11 |
|---|---|---|
| Screen 5 — Mục tiêu | Quản lý trigger: Admin thêm/sửa/bật/tắt | Tra cứu trigger catalog — chỉ đọc cho mọi role |
| Screen 5 — Header | Có nút [+ Thêm Trigger] | Thay bằng banner thông báo catalog |
| Screen 5 — Cột Hành động | [Sửa][Tắt/Bật] cho Admin | Chỉ [Xem] — giống nhau với Admin và QTV |
| Screen 5 — Cửa sổ Thêm/Sửa | Có (Modal 5B đầy đủ) | Xóa hoàn toàn |
| Screen 5 — Modal Xem chi tiết | 4 cột (Code, Tên, Mô tả, Payload) | 3 nhóm A/B/C; Payload bổ sung cột "Ví dụ giá trị"; thêm Kênh hỗ trợ; bỏ Điều kiện kích hoạt |
| Routing /admin | Duyệt campaign + Cấu hình trigger | Chỉ Duyệt campaign |
| Screen Admin — Tab | Tab "Cấu hình Trigger" (full quyền Admin) | Tab "Trigger" (chỉ đọc, giống Screen 5) |
| Campaign Builder S2 — Thẻ trigger | Badge Nguồn + Kiểu chạy (hover) | Thêm nút [Xem params] trên thẻ → mở panel Nhóm B tại chỗ |

---

## Thay đổi so với v9

| Thành phần | v9 | v10 |
|---|---|---|
| Estimate KH & chi phí | Có trong S2 + Tóm tắt nhanh campaign | Bỏ hoàn toàn — trigger có thể chưa hoạt động khi thiết lập |
| Thời gian gửi + Channel Strategy | S4 thời gian gửi (cột trái) + S5 riêng (cột phải) | Gộp thành "Kênh & Lịch gửi" ở cột phải; per kênh có lịch + blackout riêng |
| Biến trigger — Tham số | Tên + Nguồn | Thêm Mô tả + Định dạng; gợi ý khi di chuột vào tham số |
| Event source Trigger | BSS / OCS / CRM / Other | Chỉ BSS / OCS |
| Hướng dẫn khai báo Message Matrix | Không có | Thêm hộp hướng dẫn thu gọn/mở rộng per kênh |
| BL/WL trong campaign | Dùng tệp / Nhập số tự do | Chọn từ danh sách thuê bao theo kênh / Tải lên tệp; đồng bộ 2 chiều với Blacklist Management |
| Phân khúc — Reach | "Cập nhật: [giờ]" | "Reach ước tính tại: [giờ]" + ghi chú phân khúc động |
| Điều kiện lọc | 1 bộ lọc chung cho tất cả phân khúc | Mỗi thẻ phân khúc có điều kiện lọc riêng |
| Bố cục Biến thể đối tượng | Dọc (biến thể xếp chồng) | Ngang dạng tab; xem trước theo yêu cầu (nhấn Xem trước) |
| Admin / Settings | Settings = placeholder coming soon | Admin screen (duyệt campaign + cấu hình trigger) + Settings đầy đủ |

---

## Screen 1: Dashboard

### Mục tiêu
Theo dõi vận hành — quan sát theo thời gian thực sức khỏe hệ thống, hiệu suất campaign và các bất thường cần xử lý ngay.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ [≡] CVM                          LIVE ● 13/05/2026 09:45   🔔  👤       │
├────────────────┬────────────────────────────────────────────────────────┤
│ NAV LEFT       │ MAIN CONTENT (scroll dọc)                              │
│ (fixed)        │                                                        │
│ Dashboard      │ ROW 1 — TOP KPI CARDS (8 card)                        │
│ Campaign       │ ROW 2 — SYSTEM HEALTH (2 biểu đồ + hàng chờ + lỗi)       │
│ Template       │ ROW 3 — CAMPAIGN MONITORING (2 bảng + timeline)        │
│ Trigger        │ ROW 4 — USER JOURNEY FUNNEL                            │
│ Customer       │ ROW 5 — TRIGGER ANALYTICS (top + heatmap + anomaly)    │
│ Blacklist      │                                                        │
│ Report         │                                                        │
│ Admin          │                                                        │
│ Settings       │                                                        │
└────────────────┴────────────────────────────────────────────────────────┘
```

---

### ROW 1 — Top KPI Cards (8 card, 4+4)

```
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Campaign đang    │ │ Trigger kích hoạt│ │ Tin nhắn đã gửi  │ │ Tỉ lệ đã         │
│ chạy             │ │ hôm nay          │ │ hôm nay          │ │ tới đích         │
│   12             │ │   3,842          │ │   48,320         │ │   96.4%          │
│ ↑2 vs hôm qua   │ │ ↑18% vs hôm qua  │ │ ▁▃▅▇▆▅▇          │ │ ▇▇▇▆▇▇▆          │
│ ▁▃▄▅▆▅▇          │ │ ▂▄▄▅▆▇▆          │ │                  │ │ ● SLA: OK        │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘

┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Tin nhắn thất bại│ │ Tỉ lệ chuyển đổi │ │ SLA xử lý        │ │ Bị chặn Blacklist│
│                  │ │                  │ │ realtime         │ │ hôm nay          │
│   1,760          │ │   8.3%           │ │   < 2.1s avg     │ │   4,210          │
│ ⚠ ↑3% vs hôm qua│ │ ↑0.4% vs tuần   │ │ ✅ Target: <3s   │ │ ↑ BSS DNC: 3,840 │
│ ▂▃▂▄▅▆▇ ↑ trend │ │ ▄▅▄▅▆▅▆          │ │ ▂▂▂▃▂▂▃          │ │ CVM BL: 370      │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘
```

**Quy tắc hiển thị thẻ KPI:**
- Tất cả thẻ KPI cập nhật mỗi 60 giây.
- Thẻ KPI có xu hướng xấu (Tin nhắn thất bại tăng, SLA vượt ngưỡng) → viền đỏ + icon ⚠.
- Biểu đồ mini 7 ngày gần nhất; di chuột → hiện "Ngày X: giá trị Y".
- Nhấn vào thẻ KPI → xem chi tiết màn hình tương ứng.
- Công thức các chỉ số:
  - **Tỉ lệ đã tới đích** = Đã tới đích / Đã gửi × 100%
  - **Tỉ lệ chuyển đổi** = Số KH hoàn thành hành động mục tiêu / Đã tới đích × 100% (cửa sổ 24h)
  - **SLA xử lý realtime** = độ trễ p95 hiện tại; ngưỡng cảnh báo: p95 > 3 giây
  - **Bị chặn Blacklist hôm nay** = tin bị suppress bởi DNC + Blacklist campaign trong ngày

---

### ROW 2 — System Health

```
┌─────────────────────────────────────┬─────────────────────────────────────┐
│ LƯỢNG TRIGGER / SỰ KIỆN             │ ĐỘ TRỄ XỬ LÝ                        │
│ Biểu đồ đường — 24h gần nhất           │ Biểu đồ đường — 24h gần nhất           │
│                                     │                                     │
│  4K ┤           ╭──╮                │  3s ┤                               │
│  3K ┤      ╭────╯  ╰──╮            │  2s ┤  ╭──╮          ╭──╮           │
│  2K ┤  ╭───╯           ╰──         │  1s ┤──╯  ╰──────────╯  ╰───────    │
│  1K ┤──╯                           │  0s ┤                               │
│     └──────────────────────────── h│     └────────────────────────────── h│
│     00  04  08  12  16  20  24     │     00  04  08  12  16  20  24      │
│ ● Thời gian thực  ● Gần TT  ● Ngoại tuyến│ ● p50  ● p95  ● p99           │
│                                     │ ─── SLA target (3s)                 │
└─────────────────────────────────────┴─────────────────────────────────────┘

┌─────────────────────────────────────┬─────────────────────────────────────┐
│ HÀNG CHỜ & TỒN ĐỌNG                     │ TOP SYSTEM ERRORS                   │
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
│ CAMPAIGN ĐANG CHẠY NHIỀU NHẤT        │ TRIGGER KÍCH HOẠT NHIỀU NHẤT HÔM NAY │
├──────────────┬──────┬──────┬─────────┼──────────────────────┬──────┬────────┤
│ Campaign     │ Gửi  │ Tỉ lệ│ Xu hướng│ Trigger              │ Fired│ Khớp   │
├──────────────┼──────┼──────┼─────────┼──────────────────────┼──────┼────────┤
│ Nhắc nạp tiền│18,200│96.5% │ ▂▃▄▅▆▇  │ LOW_DATA_BALANCE     │ 1,842│  68%   │
│ Hết hạn data │12,400│95.4% │ ▅▄▅▆▅▆  │ SIM_ACTIVATED        │  940 │  91%   │
│ LOW_DATA batch│9,800│94.2% │ ▆▇▇▆▇▇  │ NO_APP_INSTALL_24H   │  620 │  44%   │
│ Welcome eSIM  │8,100│93.1% │ ▃▄▃▅▄▅  │ TOP_UP_FAIL          │  440 │  82%   │
│              [Xem tất cả →]          │                     [Xem tất cả →]   │
└──────────────────────────────────────┴──────────────────────────────────────┘

DÒNG SỰ KIỆN TRIGGER GẦN ĐÂY
┌─────────────────────────────────────────────────────────────────────────┐
│ 09:44:52  LOW_DATA_BALANCE   ·  0987 xxx 001  →  Khớp · Push đã gửi    │
│ 09:44:38  SIM_ACTIVATED      ·  0912 xxx 002  →  Khớp · Zalo đã gửi    │
│ 09:44:21  NO_APP_INSTALL_24H ·  0965 xxx 003  →  Không khớp             │
│ 09:44:05  TOP_UP_FAIL        ·  0976 xxx 004  →  Bị chặn (Blacklist)    │
│ 09:43:58  LOW_DATA_BALANCE   ·  0988 xxx 005  →  Bị chặn (DNC)          │
│                                                  [Đang chạy ● Tạm dừng]│
└─────────────────────────────────────────────────────────────────────────┘
```

---

### ROW 4 — Phễu hành trình khách hàng

```
┌─────────────────────────────────────────────────────────────────────────┐
│ PHỄU HÀNH TRÌNH KHÁCH HÀNG  ·  Welcome eSIM Q2/2026  ·  [Chọn campaign ▾]│
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SIM kích hoạt              ████████████████████████████  32,100  100%  │
│  ↓ rời bỏ 8%                                                            │
│  Tin nhắn gửi thành công    ███████████████████████████   29,530   92%  │
│  ↓ rời bỏ 61%                                                           │
│  Tin nhắn đã mở             ███████████████             12,200   38%    │
│  ↓ rời bỏ 23%                                                           │
│  Đã cài app                 ████████████                 9,400   29%    │
│  ↓ rời bỏ 20%                                                           │
│  Đã mua gói cước            ████████                     5,800   18%    │
│                                                                         │
│  Tỉ lệ chuyển đổi (SIM → Mua gói): 18.1%  ·  vs tuần trước: ↑1.2%     │
│                                                    [Xem chi tiết →]     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### ROW 5 — Phân tích Trigger

```
┌──────────────────────────────────┬──────────────────────────────────────┐
│ TOP TRIGGERS (7 ngày)            │ PHÁT HIỆN BẤT THƯỜNG TRIGGER         │
├──────────────┬──────┬────────────┼──────────────────────────────────────┤
│ Trigger      │ Fired│ Tỉ lệ khớp │ ⚠ LOW_DATA_BALANCE                   │
├──────────────┼──────┼────────────┤    Volume tăng đột biến +340%        │
│ LOW_DATA     │12,400│  68%  ████ │    So sánh: avg 3,600/ngày (7 ngày)  │
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
- Toàn bộ thẻ KPI và biểu đồ cập nhật mỗi 60 giây; dòng sự kiện trigger cập nhật theo thời gian thực.
- Thẻ KPI vượt ngưỡng xấu → viền đỏ + thông báo cảnh báo.
- `[Đang chạy ● / Tạm dừng]` trên dòng sự kiện: chuyển đổi dừng/tiếp tục luồng để đọc dữ liệu.
- Bản đồ nhiệt: di chuột vào ô → "Thứ X · Giờ Y: Z sự kiện"; thang màu chia đều range [min, max] thành 4 cấp: ░ thấp ▒ trung bình ▓ cao █ rất cao.
- **Phát hiện bất thường**: so sánh volume trigger trong 1 giờ với avg cùng giờ 7 ngày trước; cảnh báo khi lệch >200%. Công thức: `% tăng = (volume hiện tại − avg_7d) / avg_7d × 100%`.
- **Biểu đồ độ trễ xử lý**: p50 = trung vị; p95 = 95% request dưới ngưỡng này; p99 = 99% request; đường SLA 3s là ngưỡng cảnh báo — khi p95 vượt → vùng vượt tô màu đỏ.
- Nhấn vào campaign/trigger bất kỳ trong bảng → chuyển đến màn hình chi tiết tương ứng.

---

## Screen 2: Campaign List

### Mục tiêu
Scan nhanh trạng thái các campaign, lọc linh hoạt, tạo mới hoặc vào sửa với ít thao tác nhất.

### Bố cục

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Campaign                                           [+ Tạo Campaign]     │
├─────────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm tên campaign, mã hoặc trigger code...]                          │
│ FILTER:  [Active]  [Draft]  [Pending]  [Paused]  [Ended]                 │
│ Sắp xếp mặc định: ngày tạo mới nhất lên đầu                             │
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
├──────────────────────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Tết Nguyên Đán 2026              │ [SIM_ACT]  │ 20/01 –    │ ○ Ended    │ [Xem]        │
│ CVM-TET-2026-0001                │ [TOP_UP]   │ 10/02/2026 │ Đã kết thúc│              │
└──────────────────────────────────┴────────────┴────────────┴────────────┴──────────────┘
                                                              < 1 2 3 >  [20/trang ▾]
```

**Màu sắc thẻ trạng thái:**

| Trạng thái | Màu | Hành động |
|---|---|---|
| Active | Xanh lá `#22C55E` | [Xem] [Dừng] |
| Draft | Xám `#94A3B8` | [Xem] [Sửa] |
| Pending | Vàng `#EAB308` | [Xem] |
| Paused | Cam `#F97316` | [Xem] [Bật] |
| Ended | Xám nhạt `#94A3B8` muted | [Xem] — chỉ xem, không có hành động thay đổi trạng thái |

**Cột Trigger:** Hiển thị tối đa 2 thẻ `[CODE]`; nếu campaign có nhiều hơn, hiện `+N ⓘ` — hover/nhấn `ⓘ` mở danh sách đầy đủ dạng cửa sổ nhỏ với tên trigger + nguồn + kiểu chạy.

### Hành vi
- Bộ lọc thẻ: chọn nhiều; nhấn lại để bỏ.
- Tìm kiếm: tức thì theo tên campaign, mã campaign hoặc trigger code.
- `[Dừng]` trên campaign Active = dừng tức thì — hộp thoại xác nhận trước khi thực thi; tin nhắn đang trong hàng chờ sẽ bị hủy.
- `[Bật]` trên campaign Paused: reactivate không cần duyệt lại nếu không có thay đổi nội dung.
- `[Xem]` mở Campaign Detail View chỉ đọc — hiển thị toàn bộ thông tin khai báo.

---

## Screen 2B: Campaign Detail View

### Mục tiêu
Xem toàn bộ thông tin chi tiết một campaign ở chế độ chỉ đọc — dành cho QTV xem lại campaign đang Pending/Active/Paused/Ended, hoặc mở từ nút `[Xem]` trong Campaign List.

### Bố cục

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
│ Độ ưu tiên:       12                                                │
│ Người tạo:        QTV Marketing                                     │
│ Ngày tạo:         10/05/2026 14:32                                  │
│ Ngày gửi duyệt:   12/05/2026 09:15                                  │
│                                                                     │
│ ── 2. Trigger & Logic ───────────────────────────────────────────── │
│ Chế độ:  Advanced  ·  Logic: OR                                     │
│ Trigger:  1  SIM_ACTIVATED · Kích hoạt SIM thành công              │
│           2  NO_APP_INSTALL_24H · Chưa cài app sau 24h              │
│ Ưu tiên khi match nhiều trigger: Chỉ gửi trigger thứ tự 1           │
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
│ │ Body:  SIM eSIM đã kích hoạt thành công. Khám phá VietnamPost App. │   │
│ └──────────────────────────────────────────────────────────────┘   │
│ ┌──────────────────────────────────────────────────────────────┐   │
│ │ T2 · NO_APP_INSTALL_24H  ·  Push                             │   │
│ │ Title: Cài VietnamPost App nhận ưu đãi                             │   │
│ │ Body:  Bạn vẫn chưa cài app VietnamPost App. Cài ngay để...        │   │
│ └──────────────────────────────────────────────────────────────┘   │
│ (Các tab kênh còn lại hiển thị tương tự — nhấn tab để xem)        │
│                                                                     │
│ ── 5. Kênh & Lịch gửi ──────────────────────────────────────────────│
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
- Tiêu đề header hiển thị đúng trạng thái campaign với màu thẻ trạng thái tương ứng.
- Tab kênh trong Message Matrix: nhấn để xem nội dung từng kênh; mặc định hiển thị tab đầu tiên đã có nội dung.
- `[Đóng]` → quay về màn hình trước (Campaign List hoặc Dashboard).
- Với campaign Draft: nút `[Sửa]` hiển thị thay `[Đóng]` → điều hướng sang Campaign Builder.
- Với campaign Active: nút `[Dừng]` hiển thị thêm cạnh `[Đóng]`.
- Với campaign Paused: nút `[Bật]` hiển thị thêm cạnh `[Đóng]`.

---

## Screen 3: Campaign Builder

### Mục tiêu
Tạo hoặc sửa campaign toàn trình. Bố cục 2 cột: trái cuộn (S1/S2/S4/S6), phải cố định (Tóm tắt nhanh campaign + Kênh & Lịch gửi + S3), thanh tiêu đề cố định luôn hiện trạng thái và nút thao tác.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ HEADER FIXED                                                            │
├────────────────────────────────────────┬────────────────────────────────┤
│ LEFT (~60%, scroll)                    │ RIGHT (~40%, sticky)           │
│────────────────────────────────────────│────────────────────────────────│
│ S1. Thông tin Campaign                 │ [CAMPAIGN SUMMARY MINI]        │
│ S2. Trigger & Logic                    │   ⚠ 2 vấn đề · Reach ~6,480 KH │
│ S4. Message Matrix                     │────────────────────────────────│
│   ┌──────────────┬──────────┐          │ Kênh & Lịch gửi                │
│   │ Soạn message│ Preview  │          │   Push · Zalo OA · SMS         │
│   │ (trái ~55%) │(phải 45%)│          │   Lịch: chung / riêng per kênh │
│   └──────────────┴──────────┘          │────────────────────────────────│
│ S6. An toàn                            │ S3. Audience / Phân khúc       │
│                                        │   Gen Z · Sắp hết data         │
│                                        │   Reach ước tính: ~6,800 KH    │
└────────────────────────────────────────┴────────────────────────────────┘
```

**Quy tắc cột phải (cố định):**
- **Tóm tắt nhanh campaign**: hiện số vấn đề + reach cuối — cập nhật tức thì khi thay đổi đối tượng hoặc thiết lập lọc.
- **S3 Phân khúc**: hiển thị các phân khúc đã chọn + lượng tiếp cận; có thể thêm/bỏ phân khúc trực tiếp từ cột phải mà không cần cuộn sang cột trái.
- **Kênh & Lịch gửi**: chọn kênh gửi + cấu hình lịch chung hoặc lịch riêng per kênh (bao gồm cả Blackout per kênh) — mở/đóng khối mở rộng từng kênh độc lập.
- Cột phải **cố định toàn trang** — không cuộn theo nội dung cột trái.

---

### Thanh tiêu đề cố định

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ← Campaign                                                              │
│ Welcome eSIM Q2/2026  ·  CVM-WELCOME-ESIM-Q2-2026        ● Draft        │
│                                 [Lưu Nháp]  [Gửi duyệt →  ⚠ 2 vấn đề]  │
└─────────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- `[Gửi duyệt →]` bị vô hiệu hóa khi còn vấn đề chặn; chỉ báo đỏ hiện số vấn đề.
- Di chuột vào nút bị vô hiệu hóa → hiện danh sách ngắn các vấn đề đang chặn.
- `[Lưu Nháp]` luôn hoạt động.

---

### Section 1: Thông tin Campaign

```
┌────────────────────────────────────────────────────────────────────┐
│ 1. Thông tin Campaign                                    [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ Tên campaign *                                                     │
│ [Welcome eSIM Q2/2026___________________________________________]  │
│                                                                    │
│ Mã kịch bản  (tự sinh, chỉ đọc)                                    │
│  CVM-202506-0042                                                   │
│                                                                    │
│ Mục tiêu                            Thời gian hiệu lực            │
│ [Onboard eSIM, nhắc cài app____]    01/04/2026  –  30/06/2026     │
│                                                                    │
│ Độ ưu tiên                                                         │
│ [12_____]  ← số càng nhỏ càng ưu tiên cao; mặc định = max + 1     │
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
│  1  [SIM_ACTIVATED · Kích hoạt SIM thành công]  [≡] [✕]           │
│  2  [NO_APP_INSTALL_24H · Chưa cài app sau 24h]  [≡] [✕]          │
│                                                                    │
│  Hover chip → tooltip: Nguồn · Kiểu chạy · Tham số               │
│  Kéo [≡] để đổi thứ tự; số nhỏ hơn = ưu tiên cao hơn             │
│                                                                    │
│ Logic:  ● OR   ○ AND                                               │
│                                                                    │
│ ── Khi chọn OR ──────────────────────────────────────────────────  │
│ Mỗi trigger có message riêng. KH match trigger nào → nhận message  │
│ của trigger đó.                                                    │
│ Nếu KH khớp nhiều trigger cùng lúc:                               │
│ ● Chỉ gửi trigger có thứ tự ưu tiên cao nhất (số 1)               │
│ ○ Gửi tất cả trigger đã match                                      │
│                                                                    │
│ ── Khi chọn AND ─────────────────────────────────────────────────  │
│ KH phải thỏa đồng thời tất cả trigger mới được gửi tin.            │
│ Tất cả trigger dùng chung 1 message cho mỗi kênh.                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Mặc định Section 2 hiển thị trạng thái trống với nút `[+ Chọn trigger ▾]` — không điền sẵn trigger.
- Danh sách chọn trigger: tìm kiếm theo mã hoặc tên; hiện danh sách trigger đang hoạt động; chọn → xuất hiện dạng thẻ xếp dọc.
- Thẻ trigger hiển thị `[CODE · Tên trigger ngắn]` + badge Kiểu chạy (Realtime / Near Realtime / Offline); di chuột → tooltip hiện: Nguồn / Kiểu chạy / tóm tắt tham số (ví dụ: "3 tham số: ten_kh, loai_sim, ngay_kich_hoat").
- Mỗi thẻ trigger có nút `[Xem params]` — nhấn → mở panel trượt từ phải hiển thị Nhóm B (bảng tham số đầy đủ 5 cột: Tên tham số, Mô tả, Định dạng, Nguồn, Ví dụ giá trị) của trigger đó; đóng bằng [✕] hoặc click ngoài panel.
- Chế độ Cơ bản: chỉ cho chọn 1 trigger; ẩn Logic OR/AND và khối xung đột.
- Chế độ Nâng cao: cho chọn nhiều trigger; hiện Logic + diễn giải tương ứng.
- Kéo handle `[≡]` để sắp xếp lại; số thứ tự tự cập nhật theo vị trí mới.
- Xóa `[✕]`: các trigger bên dưới tự đánh số lại.

---

### Section 3: Audience / Phân khúc

```
┌────────────────────────────────────────────────────────────────────┐
│ 3. Audience / Phân khúc                                  [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ Nguồn: Customer 360 · Team Data · BSS · OCS                        │
│ Reach ước tính tại: 13/05/2026 08:00                               │
│                                                                    │
│ Chọn phân khúc                                                     │
│ [🔍 Tìm phân khúc... ▾]                                            │
│                                                                    │
│ Đã chọn:                                                           │
│ ┌──────────────────────────────────────────────────────────────┐  │
│ │ Gen Z User (18–25)                                       [×] │  │
│ │ 18,450 KH                                                    │  │
│ │ Điều kiện lọc thêm: [Loại thiết bị=Android]    ▸ [Sửa]      │  │
│ │ → Sau lọc: ~9,200 KH                                         │  │
│ └──────────────────────────────────────────────────────────────┘  │
│ ┌──────────────────────────────────────────────────────────────┐  │
│ │ Sắp hết data                                             [×] │  │
│ │ 8,920 KH                                                     │  │
│ │ Điều kiện lọc thêm: (chưa có)          ▸ [+ Thêm lọc]       │  │
│ └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│ ── Khi click [Sửa] / [+ Thêm lọc] (mở rộng tại chỗ bên dưới card) ─ │
│ [Loại thiết bị ▾]  [= ▾]  [Android ▾]  [×]                        │
│ [+ Thêm điều kiện]                                                 │
│ ⓘ Giá trị được cấu hình sẵn — chỉ chọn, không nhập tự do          │
│ [Áp dụng]  [Hủy]                                                   │
│ ────────────────────────────────────────────────────────────────── │
│                                                                    │
│ Logic: ● Bất kỳ phân khúc nào (OR)   ○ Tất cả phân khúc (AND)    │
│                                                                    │
│ Reach ước tính: ~6,800 KH  [↻ Tính lại]                           │
│ ⓘ Reach ước tính tại thời điểm hiện tại. Phân khúc được đánh giá  │
│   lại khi trigger kích hoạt — KH có thể vào/ra phân khúc theo     │
│   thời gian.                                                       │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Ô tìm kiếm phân khúc: gõ để lọc; chọn → hiện dưới dạng thẻ, không hiện danh sách ô tích sẵn.
- Xóa `[×]` trên thẻ phân khúc → bỏ phân khúc đó; lượng tiếp cận cập nhật lại.
- Điều kiện lọc: riêng theo từng phân khúc (mỗi thẻ có thể mở rộng độc lập); tất cả ô là danh sách chọn giá trị cố định — không nhập tự do.
- Nhấn `[+ Thêm lọc]` hoặc `[Sửa]` → mở rộng tại chỗ dưới thẻ đó; nhấn `[Áp dụng]` → cập nhật thẻ điều kiện + lượng tiếp cận sau lọc trong thẻ phân khúc.
- Reach ước tính là tại thời điểm tính, nhấn `[↻ Tính lại]` để refresh; phân khúc được tái đánh giá tại thời điểm trigger kích hoạt (linh hoạt — không cố định).
- CVM không tự tạo phân khúc — phân khúc do Team Data/BSS/OCS cung cấp.

---

### Section 4: Message Matrix

```
┌────────────────────────────────────────────────────────────────────┐
│ 4. Message Matrix  ·  Trigger × Kênh                     [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ ── Trạng thái chưa có kênh nào ────────────────────────────────── │
│                                                                    │
│         Chưa có kênh nào.                                          │
│         [+ Kênh]  để bắt đầu soạn nội dung.                       │
│                                                                    │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│ ── Sau khi đã thêm kênh ────────────────────────────────────────── │
│                                                                    │
│ TIẾN ĐỘ NỘI DUNG:                                                  │
│ Push ●●  Zalo ●●  SMS ●○  Banner ○○  Email ○○  USSD ○○             │
│ (● = hoàn chỉnh  ○ = chưa cấu hình)                                │
│                                                                    │
│ Tab kênh:  [Push ●● ×]  [Zalo OA ●● ×]  [SMS ●○ ×]  [Banner ○○ ×] │
│            [Email ○○ ×]  [USSD ○○ ×]  [+ Kênh]                    │
│                                                                    │
│ ⓘ Thời gian gửi và giờ giới nghiêm được cấu hình trong khối       │
│   "Kênh & Lịch gửi" ở cột phải.                                    │
│                                                                    │
│ ── Tab Push (đang active) ──────────────────────────────────────   │
│                                                                    │
│ [ℹ Hướng dẫn khai báo Push]  ▸ [Mở]                               │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Title: tối đa 65 ký tự. Hỗ trợ biến {{...}}.                    │
│ • Body: tối đa 240 ký tự. Hỗ trợ biến {{...}}.                    │
│ • Image: optional, tỉ lệ 1:1, tối đa 1MB.                         │
│ • Biến dạng: {{ten_bien}} — chữ thường, không dấu, gạch dưới.     │
│   Ví dụ: {{ten_kh}}, {{loai_sim}}, {{ngay_kich_hoat}}              │
│ • Nếu biến null/trống → hiển thị chuỗi rỗng (không báo lỗi).      │
│ • Đầu mối kênh: Team Mobile / Push Gateway                         │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T1 · SIM_ACTIVATED                                                      │
│ │ PARAMS: [ten_kh] [loai_sim] [ngay_kich_hoat] [so_dt]                       │
│ │ (di chuột → hiện mô tả; nhấn → chèn vào nội dung)                   │
│ ├────────────────────────────────────────────────────────────────────────────┤
│ │ Phân khúc:  [Tất cả (dự phòng) ●]  [VIP ○]  [+ Biến thể đối tượng]        │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG  (tab "Tất cả" active)   │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ Image (1:1, optional)                   │ [Push notification mockup]       │
│ │ ┌──────────────────────────────────┐   │ ┌────────────────────────────┐   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │ │ 🔔 App icon   Title here   │   │
│ │ │     [Chọn thư viện]              │   │ │    Body text rendered...   │   │
│ │ └──────────────────────────────────┘   │ └────────────────────────────┘   │
│ │                                         │                                  │
│ │ Template: [Welcome Push ▾]              │ GIÁ TRỊ MẪU                      │
│ │                                         │ ten_kh    [Nguyễn Văn A  ]       │
│ │ Title *                    0/65         │ loai_sim  [eSIM          ]       │
│ │ [Chào mừng bạn!__________________]     │ ngay_kt   [01/04/2026    ]       │
│ │                                         │ so_dt     [0987 xxx 001  ]       │
│ │ Body *                     0/240        │                                  │
│ │ [SIM {{loai_sim}} đã kích hoạt...]      │ [↻ Xem trước]                   │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T2 · NO_APP_INSTALL_24H                                                 │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Push notification mockup]       │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ 🔔 App icon   Title here   │   │
│ │ Image (1:1, optional)                   │ │    Body text rendered...   │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │ GIÁ TRỊ MẪU                      │
│ │ │     1:1 khuyến nghị              │   │ app_installed  [false     ]       │
│ │ └──────────────────────────────────┘   │ last_active    [10/04/2026]       │
│ │ (Nếu đã upload: ảnh thu nhỏ + [Xóa][Đổi])│                                  │
│ │                                         │ (nhập giá trị mẫu → nhấn [↻ Xem trước] để cập nhật)  │
│ │ Template: [Install App ▾]               │                                  │
│ │                                         │                                  │
│ │ Title *                    0/65         │                                  │
│ │ [Cài VietnamPost App nhận ưu đãi________]    │                                  │
│ │                                         │                                  │
│ │ Body *                     0/240        │                                  │
│ │ [Bạn vẫn chưa cài app VietnamPost App...]    │                                  │
│ │                                         │                                  │
│ │ [+ Biến thể đối tượng]                  │                                  │
│ └─────────────────────────────────────────┴──────────────────────────────────┘
│                                                                    │
│ ── Tab SMS ─────────────────────────────────────────────────────   │
│                                                                    │
│ [ℹ Hướng dẫn khai báo SMS]  ▸ [Mở]                                │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Body: tối đa 160 ký tự/segment. Vượt 160 → tính thêm segment.  │
│ • Chỉ plain text — không hỗ trợ ảnh, link rút gọn tự động.       │
│ • Biến dạng: {{ten_bien}} — nếu biến null → hiển thị chuỗi rỗng.  │
│ • Biến dài có thể đẩy tin vượt 160 ký tự — kiểm tra bộ đếm ký tự.     │
│ • Đầu mối kênh: Team SMS / SMSC                                   │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ✓  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim] [so_dt]     │ [SMS bubble mockup]              │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ VietnamPost                    │   │
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
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [SMS bubble mockup]              │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
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
│ [ℹ Hướng dẫn khai báo USSD]  ▸ [Mở]                               │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Body: tối đa 182 ký tự. Chỉ plain text, không dấu tiếng Việt.  │
│ • Không hỗ trợ ảnh, link, ký tự đặc biệt.                        │
│ • Biến dạng: {{ten_bien}} — giá trị biến cũng phải không dấu.     │
│ • Đầu mối kênh: Team USSD                                         │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim] [so_dt]     │ [USSD terminal mockup]           │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
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
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [USSD terminal mockup]           │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
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
│ [ℹ Hướng dẫn khai báo Zalo OA]  ▸ [Mở]                            │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Nội dung: tối đa 1000 ký tự. Hỗ trợ biến {{...}}.               │
│ • Image: optional, tỉ lệ tự do, khuyến nghị 16:9 hoặc 1:1.       │
│ • Biến dạng: {{ten_bien}} — chữ thường, không dấu, gạch dưới.     │
│ • OA phải được liên kết và phê duyệt trước khi gửi.               │
│ • Đầu mối kênh: Team Zalo OA                                      │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Zalo OA chat bubble mockup]     │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ │ 🟦 VietnamPost         │   │
│ │                                         │ │ [image nếu có]             │   │
│ │ Image (tự do tỉ lệ, optional)           │ │ Nội dung tin nhắn render   │   │
│ │ ┌──────────────────────────────────┐   │ └────────────────────────────┘   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │                                  │
│ │ │     [Chọn thư viện]              │   │ GIÁ TRỊ MẪU                      │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │ (Nếu đã upload: ảnh thu nhỏ + [Xóa][Đổi])│ loai_sim  [eSIM          ]       │
│ │                                         │ (nhập giá trị mẫu → nhấn [↻ Xem trước] để cập nhật)  │
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
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Zalo OA chat bubble mockup]     │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
│ │                                         │ │ 🟦 VietnamPost         │   │
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
│ [ℹ Hướng dẫn khai báo Banner]  ▸ [Mở]                             │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Image: BẮT BUỘC, tỉ lệ 16:9, tối đa 2MB.                       │
│ • Title: tối đa 65 ký tự. Body: tối đa 120 ký tự.                │
│ • CTA Label + CTA URL: bắt buộc (không có CTA → không gửi được).  │
│ • Hỗ trợ biến {{...}} trong Title và Body.                        │
│ • Đầu mối kênh: Team App / Banner                                 │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Banner card mockup]             │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ │ [Image 16:9 full width]    │   │
│ │                                         │ │ Title text                 │   │
│ │ Image **bắt buộc** (16:9)               │ │ Body text                  │   │
│ │ ┌──────────────────────────────────┐   │ │ [CTA Button]               │   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │ └────────────────────────────┘   │
│ │ │     [Chọn thư viện]              │   │                                  │
│ │ │     Bắt buộc · tỉ lệ 16:9       │   │ GIÁ TRỊ MẪU                      │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │ ⚠ Chưa upload image → block lưu       │ loai_sim  [eSIM          ]       │
│ │                                         │ (nhập giá trị mẫu → nhấn [↻ Xem trước] để cập nhật)  │
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
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Banner card mockup]             │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
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
│ [ℹ Hướng dẫn khai báo Email]  ▸ [Mở]                              │
│ ── Khi mở ────────────────────────────────────────────────────── │
│ • Subject: tối đa 100 ký tự. Hỗ trợ biến {{...}}.                │
│ • Body: plain text, không giới hạn. Hỗ trợ biến {{...}}.         │
│ • Header image: optional, banner ngang, tối đa 1MB.               │
│ • Biến dạng: {{ten_bien}} — chữ thường, không dấu, gạch dưới.     │
│ • Đầu mối kênh: Team Email                                        │
│ ─────────────────────────────────────────────────────────────────  │
│                                                                    │
│                                                                    │
│ ┌────────────────────────────────────────────────────────────────────────────┐
│ │ ○  T1 · SIM_ACTIVATED                                                      │
│ ├─────────────────────────────────────────┬──────────────────────────────────┤
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [ten_kh] [loai_sim]             │ [Email client mockup]            │
│ │         [ngay_kich_hoat] [so_dt]        │ ┌────────────────────────────┐   │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ │ From: VietnamPost      │   │
│ │                                         │ │ Subject: [subject render]  │   │
│ │ Header Image (banner ngang, optional)   │ │ [header image nếu có]      │   │
│ │ ┌──────────────────────────────────┐   │ │ Body render plain text     │   │
│ │ │  🖼  Kéo thả hoặc [Tải lên]     │   │ └────────────────────────────┘   │
│ │ │     [Chọn thư viện]              │   │                                  │
│ │ │     Banner ngang, optional       │   │ GIÁ TRỊ MẪU                      │
│ │ └──────────────────────────────────┘   │ ten_kh    [Nguyễn Văn A  ]       │
│ │                                         │ loai_sim  [eSIM          ]       │
│ │ Template: [Email Welcome ▾]             │ (nhập giá trị mẫu → nhấn [↻ Xem trước] để cập nhật)  │
│ │                                         │                                  │
│ │ Subject *                  0/100        │                                  │
│ │ [Chào mừng {{ten_kh}} đến VietnamPost]     │                                  │
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
│ │ SOẠN NỘI DUNG                           │ XEM TRƯỚC                        │
│ ├─────────────────────────────────────────┼──────────────────────────────────┤
│ │ PARAMS: [app_installed] [last_active]   │ [Email client mockup]            │
│ │ (nhấn vào tham số → chèn vào nội dung)        │ ┌────────────────────────────┐   │
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
| Gửi ngay | Ngay khi trigger kích hoạt và KH qua bộ lọc | Thời gian thực, phản hồi tức thì |
| Sau X phút/giờ/ngày kể từ T | T = lúc trigger kích hoạt cho KH đó | Hoãn có chủ ý |
| Vào lúc HH:MM ngày T+N | T+0 = ngày trigger kích hoạt; T+1 = hôm sau | Gửi vào khung giờ cố định |

**Trường ảnh per kênh** (trong cột Soạn, trước Tiêu đề):

| Kênh | Trường ảnh |
|---|---|
| Push | Tải lên tùy chọn · tỉ lệ 1:1 · ảnh thu nhỏ xem trước trong mô phỏng bên phải |
| Zalo OA | Tải lên tùy chọn · tỉ lệ tự do · hiện phía trên bong bóng chat trong xem trước |
| Banner | Tải lên **bắt buộc** · tỉ lệ 16:9 · hiện toàn chiều ngang trong xem trước |
| Email | Tải lên tùy chọn · ảnh banner ngang · hiện ảnh tiêu đề trong xem trước |
| SMS | Ghi chú "(Không hỗ trợ hình ảnh)" — không có tải lên |
| USSD | Ghi chú "(Không hỗ trợ hình ảnh)" — không có tải lên |

**Xem trước per kênh** (cột XEM TRƯỚC, cập nhật khi nhấn [↻ Xem trước]):

| Kênh | Mô hình xem trước |
|---|---|
| Push | Mô phỏng thông báo hệ điều hành: biểu tượng ứng dụng bên trái + ảnh thu nhỏ 1:1 bên phải + Tiêu đề + Nội dung |
| Zalo OA | Bong bóng chat: brand header "VietnamPost" + image phía trên (nếu có) + nội dung |
| SMS | Plain SMS bubble: chỉ text, không ảnh, hiện counter X/160 |
| USSD | Monospace terminal: plain text, menu style |
| Banner | Thẻ 16:9: ảnh toàn ngang + Tiêu đề + Nội dung + nút hành động |
| Email | Giao diện email: thanh tiêu đề thư + ảnh header + nội dung |

**Hành vi:**
- Khối **Thời gian gửi** nằm trên cùng Section 4, cấu hình 1 lần, áp dụng cho toàn bộ trigger và kênh trong campaign.
- **Tham số trigger** hiển thị ngay trong mỗi thẻ tin nhắn, liệt kê đúng danh sách tham số của trigger đó; nhấn vào tham số → chèn `{{tham_so}}` vào vị trí con trỏ trong ô nhập đang được chọn.
- Chế độ AND: mỗi tab kênh chỉ có 1 thẻ duy nhất; tham số hiển thị hợp nhất từ tất cả trigger; nút `[+ Biến thể đối tượng]` bị ẩn hoàn toàn.
- Chế độ OR: mỗi tab kênh có N thẻ tương ứng N trigger; mỗi thẻ hiển thị đúng tham số của trigger đó; nút `[+ Biến thể đối tượng]` hiển thị trong mỗi thẻ.
- Chỉ báo hoàn thành trên tab: `●●` = 2/2 (xanh), `●○` = 1/2 (cam), `○○` = 0/2 (xám).
- Mỗi tab kênh có `[×]` để bỏ kênh đó; nếu tab đã có nội dung → "Bỏ kênh [X]? Nội dung đã soạn sẽ mất." → xác nhận thì xóa tab + bỏ tick kênh trong "Kênh & Lịch gửi".
- `[+ Biến thể đối tượng]` → thêm tab biến thể mới (xem mô hình bên dưới).
- Mỗi thẻ trigger hiển thị **2 cột liền trong thẻ** (Soạn | XEM TRƯỚC); preview on-demand (nhấn `[↻ Xem trước]`).
- SMS: cho nhập vượt 160 ký tự; bộ đếm đỏ + chỉ báo "X tin SMS"; không chặn lưu.
- USSD: plain text 182 ký tự; cảnh báo nếu có ký tự đặc biệt.
- Tab `[+ Kênh]`: chỉ liệt kê kênh chưa có tab; thêm kênh → tự tick kênh đó trong "Kênh & Lịch gửi".

**Biến thể đối tượng — Audience Variant (bố cục tab ngang):**

```
── Tab Push (T1 · SIM_ACTIVATED — đang có 2 biến thể) ───────────────────────────────

┌────────────────────────────────────────────────────────────────────────────────────┐
│ T1 · SIM_ACTIVATED                                                                 │
│ THAM SỐ: [{{ten_kh}}] [{{loai_sim}}] [{{ngay_kich_hoat}}] [{{so_dien_thoai}}]      │
│ (hover chip → tooltip: tên, mô tả, định dạng, ví dụ)                              │
├────────────────────────────────────────────────────────────────────────────────────┤
│ Đối tượng:  [Tất cả (dự phòng) ●]  [VIP]  [+ Biến thể đối tượng]                  │
│              ↑ tab đang active      ↑ tab                                          │
├─────────────────────────────────────────────┬──────────────────────────────────────┤
│ SOẠN NỘI DUNG  (tab "Tất cả" đang active)   │ XEM TRƯỚC                            │
├─────────────────────────────────────────────┼──────────────────────────────────────┤
│ Image (1:1, optional)                       │ [Phone mockup]                       │
│ ┌──────────────────────────────────┐        │ ┌────────────────────────────┐       │
│ │  🖼  Kéo thả hoặc [Tải lên]     │        │ │ 🔔 App icon   Title here   │       │
│ │     [Chọn thư viện]              │        │ │    Body text rendered...   │       │
│ └──────────────────────────────────┘        │ └────────────────────────────┘       │
│                                             │                                      │
│ Template: [Welcome Push ▾]                  │ GIÁ TRỊ MẪU                          │
│ Title *  [Chào mừng bạn!__________]         │ ten_kh    [Nguyễn Văn A  ]           │
│ Body *   [SIM {{loai_sim}} đã kích...]      │ loai_sim  [eSIM          ]           │
│                                             │ ngay_kich_hoat  [01/04/2026]         │
│                                             │ so_dien_thoai  [0987xxx001]          │
│                                             │ [↻ Xem trước]                       │
└─────────────────────────────────────────────┴──────────────────────────────────────┘
```

**Hành vi Audience Variant:**
- Tab "Tất cả (dự phòng)" luôn là tab đầu tiên, không thể xóa, không có `[×]`.
- Nhấn `[+ Biến thể đối tượng]` → thêm tab mới; danh sách chọn phân khúc (liệt kê phân khúc đã chọn ở S3) — bắt buộc chọn trước khi lưu; tên tab = tên phân khúc.
- Nhấn tab → hiện nội dung biến thể đó + khung XEM TRƯỚC bên phải.
- Từ tab biến thể 2 trở đi: có `[×]` → hộp thoại "Xóa biến thể này?" trước khi xóa.
- Thứ tự ưu tiên: tab từ trái → phải (sau Tất cả); KH khớp phân khúc biến thể nào (độ ưu tiên cao hơn) → nhận message đó; không khớp → nhận Biến thể "Tất cả" (dự phòng).
- Tham số trigger dùng chung, hiển thị ở đầu thẻ; không lặp lại trong từng biến thể.
- Xem trước theo yêu cầu: nhập giá trị mẫu rồi nhấn `[↻ Xem trước]`; không tự cập nhật liên tục.
- Chỉ báo hoàn thành trên tab kênh tính là hoàn chỉnh khi tab "Tất cả (dự phòng)" đã có nội dung.

**Cấu trúc thẻ soạn nội dung:**

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
│ Title *                                      │ GIÁ TRỊ MẪU              │
│ [Chào mừng bạn!____________________]         │ ten_kh  [Nguyễn Văn A]   │
│                                              │ loai_sim  [eSIM]         │
│ Body *                                       │ ngay_kich_hoat [01/04]   │
│ [SIM {{loai_sim}} đã kích hoạt thành...]     │ so_dien_thoai [0987xxx]  │
│                                              │                          │
│ PARAMS (T1 · SIM_ACTIVATED):                 │ [↻ Xem trước]           │
│ [{{ten_kh}}] [{{loai_sim}}]                  │                          │
│ [{{ngay_kich_hoat}}] [{{so_dien_thoai}}]     │                          │
└──────────────────────────────────────────────┴──────────────────────────┘
```

---

### Kênh & Lịch gửi _(cột phải Campaign Builder — thay thế Section 5 cũ)_

> **Lưu ý thiết kế:** Logic pipeline kênh (thứ tự fallback, thời gian chờ, điều kiện dừng) được định nghĩa nội bộ, không hiển thị trên giao diện. Xem chi tiết tại [`deferred/channel-pipeline-logic.md`](deferred/channel-pipeline-logic.md).

```
┌────────────────────────────────────────────────────────────────────┐
│ Kênh & Lịch gửi                                          [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ ── Khi chưa có kênh nào trong Message Matrix ──────────────────── │
│ (Chưa có kênh — thêm kênh trong Message Matrix để cấu hình lịch)  │
│                                                                    │
│ ── Sau khi đã có kênh ──────────────────────────────────────────── │
│ Kênh đang có:  Push · Zalo OA · SMS  (sync từ Message Matrix)      │
│                                                                    │
│ ── Lịch gửi ──────────────────────────────────────────────────── │
│ ● Lịch chung (áp dụng cho tất cả kênh)                             │
│ ○ Lịch riêng theo kênh                                             │
│                                                                    │
│ ── Khi chọn Lịch chung ─────────────────────────────────────────  │
│ Thời gian gửi:                                                     │
│ ● Gửi ngay sau khi trigger kích hoạt                               │
│ ○ Sau  [__] [phút / giờ / ngày ▾]  kể từ T                        │
│ ○ Vào lúc  [__:__]  ngày  T + [__]                                 │
│ ⓘ T = thời điểm trigger kích hoạt cho từng KH cụ thể              │
│                                                                    │
│ Giờ giới nghiêm (Blackout):                                        │
│ ○ Không áp dụng                                                    │
│ ● Bật: [22:00] — [08:00]                                           │
│        Xử lý: [Hủy luôn ▾]  (Hủy luôn / Hoãn đến đầu khung giờ) │
│                                                                    │
│ ── Khi chọn Lịch riêng theo kênh ──────────────────────────────── │
│ (Mỗi kênh trong Message Matrix hiển thị 1 accordion row)          │
│                                                                    │
│ ▼ Push                                                             │
│   Thời gian gửi:                                                   │
│   ● Gửi ngay  ○ Sau [__] [phút/giờ/ngày ▾]  ○ Vào [__:__] T+[__] │
│   Giờ giới nghiêm:                                                 │
│   ○ Không áp dụng  ● Bật: [22:00]—[08:00]  [Hủy luôn ▾]          │
│                                                                    │
│ ▶ Zalo OA  (click để mở)                                           │
│ ▶ SMS      (click để mở)                                           │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Định nghĩa thời gian gửi:**

| Tùy chọn | Ý nghĩa | Dùng khi |
|---|---|---|
| Gửi ngay | Ngay khi trigger kích hoạt và KH qua bộ lọc | Thời gian thực, phản hồi tức thì |
| Sau X phút/giờ/ngày kể từ T | T = lúc trigger kích hoạt cho KH đó | Hoãn có chủ ý |
| Vào lúc HH:MM ngày T+N | T+0 = ngày trigger kích hoạt; T+1 = hôm sau | Gửi vào khung giờ cố định |

**Hành vi:**
- "Kênh & Lịch gửi" hoàn toàn phụ thuộc vào Message Matrix — kênh nào có tab trong S4 thì mới xuất hiện ở đây.
- Thêm kênh (`[+ Kênh]` trong S4) → khối cấu hình kênh đó tự sinh trong "Kênh & Lịch gửi".
- Xóa kênh (`[×]` trên tab trong S4) → khối cấu hình kênh đó tự mất khỏi "Kênh & Lịch gửi"; lịch riêng của kênh đó bị xóa.
- Chưa có kênh nào trong S4 → "Kênh & Lịch gửi" hiện note "(Chưa có kênh — thêm kênh trong Message Matrix để cấu hình lịch)".
- Chuyển từ Lịch chung → Lịch riêng: điền sẵn thời gian + giờ giới nghiêm từ lịch chung vào từng kênh; user chỉnh tiếp.
- Chuyển từ Lịch riêng → Lịch chung: hộp thoại "Chuyển về lịch chung sẽ ghi đè cấu hình lịch của tất cả kênh. Tiếp tục?" → xác nhận thì ghi đè.
- Khối kênh: mở/đóng độc lập; mặc định mở kênh đầu tiên.

---

### Section 6: An toàn

```
┌────────────────────────────────────────────────────────────────────┐
│ 6. An toàn                                               [Thu gọn] │
├────────────────────────────────────────────────────────────────────┤
│ ⓘ Giờ giới nghiêm (Blackout) được cấu hình trong "Kênh & Lịch     │
│   gửi" — theo từng kênh hoặc lịch chung.                          │
│                                                                    │
│ GROUP — SUPPRESSION (DNC / Blacklist / Whitelist)                  │
│ ┌────────────────────────────────────────────────────────────────┐ │
│ │ ☑ Check DNC toàn hệ thống (luôn bật mặc định)                 │ │
│ │   Bỏ tick → confirm dialog cảnh báo rủi ro gửi KH đã từ chối  │ │
│ │                                                                │ │
│ │ Blacklist campaign — theo kênh                                 │ │
│ │ ○ Không dùng                                                  │ │
│ │ ● Chọn từ danh sách thuê bao theo kênh                        │ │
│ │ ○ Upload tệp                                                  │ │
│ │                                                                │ │
│ │   ── Khi chọn "Chọn từ danh sách thuê bao theo kênh" ──────── │ │
│ │   Kênh:  [Push]  [Zalo OA]  [SMS]  [+ Thêm kênh]              │ │
│ │   ── Kênh: Push ─────────────────────────────────────────────  │ │
│ │   [🔍 Tìm số thuê bao...]                                       │ │
│ │   ☑ 0987 xxx 001  · Nguyễn Văn A  · Active                    │ │
│ │   ☑ 0912 xxx 002  · Trần Thị B    · Active                    │ │
│ │   ☐ 0965 xxx 003  · Lê Văn C      · Inactive                  │ │
│ │   Đã chọn: 2 số                                                │ │
│ │   ⓘ Danh sách này sẽ tự đồng bộ sang Blacklist Management        │ │
│ │     (nguồn: "Chọn trong campaign CVM-xxx, kênh Push")          │ │
│ │                                                                │ │
│ │   ── Khi chọn "Upload tệp" ──────────────────────────────────  │ │
│ │   Kênh áp dụng: [Push ▾]  [+ Thêm kênh]                       │ │
│ │   [📥 Upload tệp CSV]  [Tải file mẫu]                          │ │
│ │   Tệp hiện tại: BL_ESIM_Q2_2026 · 320 số  [Xem] [Thay]        │ │
│ │                                                                │ │
│ │ Whitelist campaign — theo kênh                                 │ │
│ │ ● Không dùng                                                  │ │
│ │ ○ Chọn từ danh sách thuê bao theo kênh                        │ │
│ │ ○ Upload tệp                                                  │ │
│ │   (giao diện tương tự Blacklist)                               │ │
│ │                                                                │ │
│ │ Reach cuối cùng: ~6,480 KH                                     │ │
│ │ = 6,800 → trừ DNC → trừ BL theo kênh → giao WL theo kênh      │ │
│ └────────────────────────────────────────────────────────────────┘ │
│                                                                    │
│ ⓘ Giới hạn tần suất nhận tin (frequency cap) được cấu hình        │
│    tại Settings → áp dụng cho toàn bộ campaign                    │
│                                    [Đến Settings →]               │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Blacklist campaign có 2 lựa chọn: "Không dùng" / "Chọn từ danh sách thuê bao theo kênh" / "Tải lên tệp".
- Chọn "Chọn từ danh sách thuê bao theo kênh": tìm kiếm và tick số trong từng kênh; mỗi kênh có danh sách riêng.
- Số được chọn tự đồng bộ sang Blacklist Management với nguồn "Chọn trong campaign [MÃ], kênh [X]".
- Tải lên tệp: chọn kênh áp dụng trước, sau đó tải lên file CSV; hỗ trợ nhiều kênh cùng 1 tệp; modal hiển thị hộp hướng dẫn format (1 cột `so_dien_thoai`, 10 chữ số bắt đầu `0`, UTF-8, tối đa 100.000 dòng) và link `[📥 Tải file mẫu (blacklist_mau.csv)]`; sau khi parse hiển thị Hợp lệ / Trùng / Sai định dạng kèm ghi chú lý do sai.
- Chọn nhưng chưa có số/tệp hợp lệ nào → chặn lưu; nút `[Gửi duyệt]` bị vô hiệu hóa.
- Whitelist cùng logic với Blacklist; chọn "Chỉ gửi cho whitelist theo kênh".
- Công thức reach: `Audience → trừ DNC → trừ Campaign BL theo kênh → giao WL theo kênh (nếu bật)`.

**Cửa sổ chọn tệp (Blacklist hoặc Whitelist — Tải lên tệp):**

```
┌──────────────────────────────────────────────────────────────┐
│ Upload tệp Blacklist                                         │
├──────────────────────────────────────────────────────────────┤
│ Kênh áp dụng *                                               │
│ ☑ Push   ☑ Zalo OA   ☐ SMS   ☐ Banner                       │
│                                                              │
│ File CSV *                                                   │
│ ┌──────────────────────────────┐                            │
│ │  📄 Kéo thả file vào đây    │                            │
│ │  hoặc [Chọn file]           │                            │
│ └──────────────────────────────┘                            │
│ ┌──────────────────────────────────────────────────────┐    │
│ │ ℹ Yêu cầu file CSV:                                  │    │
│ │ • 1 cột so_dien_thoai, có/không cần header           │    │
│ │ • Mỗi dòng 1 số — 10 chữ số, bắt đầu bằng 0         │    │
│ │ • Hợp lệ: 0901234567                                  │    │
│ │ • Sai: +84901234567 · 090-123-4567 · 901234567        │    │
│ │ • Tối đa 100.000 dòng · Encoding UTF-8               │    │
│ └──────────────────────────────────────────────────────┘    │
│ [📥 Tải file mẫu (blacklist_mau.csv)]                       │
│                                                              │
│ Hợp lệ: 320  ·  Trùng: 8  ·  Sai định dạng: 2              │
│ ⚠ Sai định dạng: không phải 10 chữ số / không bắt đầu 0    │
│                                                              │
│                        [Hủy]  [Áp dụng]                     │
└──────────────────────────────────────────────────────────────┘
```

---

### User Flow — Tạo Campaign

```
1. Nhập thông tin campaign (tên, mã, mục tiêu, thời gian)
   ↓
2. Chọn chế độ Cơ bản/Nâng cao → thêm trigger → kéo-thả để ưu tiên
   ↓
3. Chọn phân khúc từ danh sách → thêm điều kiện lọc nếu cần
   ↓
4. Chọn kênh gửi + cấu hình Lịch chung hoặc Lịch riêng per kênh (bao gồm Blackout)
   ↓
5. Cấu hình Message Matrix: tab kênh → xem hướng dẫn khai báo → soạn/chọn mẫu nội dung
   ↓
6. Cấu hình An toàn: blackout / DNC / BL campaign / WL
   ↓
7. Lưu nháp hoặc Gửi duyệt (resolve hết vấn đề đang chặn trước)
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
│ Sắp xếp mặc định: số lần dùng nhiều nhất lên đầu                    │
├──────────────────────┬───────────────┬────────┬─────────────────────┤
│ Tên Template         │ Kênh hỗ trợ   │ Dùng   │ Hành động           │
├──────────────────────┼───────────────┼────────┼─────────────────────┤
│ Chào mừng SIM        │ Push · Zalo   │ 3 lần  │ [Sửa] [Nhân bản] [Tắt]│
│ Nhắc nạp thẻ         │ SMS · USSD    │ 5 lần  │ [Sửa] [Nhân bản] [Tắt]│
│ Sắp hết data         │ Push · SMS    │ 2 lần  │ [Sửa] [Nhân bản] [Tắt]│
│ Sinh nhật KH         │ Zalo · Email  │ 4 lần  │ [Sửa] [Nhân bản] [Tắt]│
│ Cài app nhắc nhở     │ Push          │ 1 lần  │ [Sửa] [Nhân bản] [Bật]│
└──────────────────────┴───────────────┴────────┴─────────────────────┘
                                                    < 1 2 >  [20/trang ▾]
```

**Hành vi:**
- Bộ lọc Kênh → lọc template có nội dung kênh đó.
- `[Nhân bản]` → tạo bản sao với tên "Bản sao của [Tên]", mở ngay để sửa.
- `[Tắt/Bật]` → chuyển đổi trạng thái Hoạt động/Tắt; mẫu nội dung không hoạt động không hiện trong danh sách khi tạo campaign.
- "Dùng" = số campaign đang dùng mẫu nội dung này; nhấn → hiện danh sách campaign liên quan.

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
│ │ Image (1:1, optional)                  │ GIÁ TRỊ MẪU            │ │
│ │ [Tải lên] [Chọn thư viện]              │ ten_kh [Nguyễn Văn A]  │ │
│ │                                        │ loai_sim [eSIM]        │ │
│ │ Title *                                │ data_con_lai [120 MB]  │ │
│ │ [Chào mừng {{ten_kh}}!__________]      │                        │ │
│ │                                        │ [↻ Xem trước]         │ │
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
│ │ [____________________________]  0/160  │ GIÁ TRỊ MẪU            │ │
│ │                                        │ ten_kh [Nguyễn Văn A]  │ │
│ │ ⚠ Chỉ plain text, không hỗ trợ ảnh    │ so_du [50,000đ]        │ │
│ │                                        │                        │ │
│ │                                        │ [↻ Xem trước]         │ │
│ └────────────────────────────────────────┴────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Tab kênh: mỗi tab có thẻ soạn nội dung độc lập (cấu trúc Soạn | XEM TRƯỚC giống Message Matrix); `[+ Kênh]` thêm tab mới.
- THAM SỐ ĐỘNG nằm đầu mỗi thẻ; nhấn vào tham số → chèn `{{tham_so}}` vào vị trí con trỏ trong ô nhập đang được chọn.
- Xem trước cập nhật khi nhấn [↻ Xem trước]; nhập tay Giá trị mẫu để kiểm tra nội dung hiển thị đúng.
- Chỉ báo hoàn thành trên tab: `●` = có nội dung, `○` = chưa cấu hình; TIẾN ĐỘ NỘI DUNG tổng hợp tất cả tab.
- Lưu Mẫu nội dung: kiểm tra tên bắt buộc; cảnh báo nếu tab kênh nào đang trống (không chặn lưu).
- Khi người dùng chuyển từ OR sang AND: hiển thị hộp thoại "Chuyển sang AND mode sẽ xóa toàn bộ Biến thể đối tượng đã tạo. Tiếp tục?" — xác nhận thì xóa variant, button `[+ Biến thể đối tượng]` bị ẩn.

---

## Screen 5: Trigger Management

### Mục tiêu
Tra cứu danh sách trigger catalog — chỉ đọc cho mọi role. Trigger là danh mục cố định do Dev/SA quản lý qua deployment.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ TRIGGER MANAGEMENT                                                  │
├─────────────────────────────────────────────────────────────────────┤
│ ℹ Danh sách sự kiện kích hoạt do hệ thống cung cấp.                │
│   Liên hệ Team Kỹ thuật để thêm hoặc điều chỉnh trigger.           │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm trigger code hoặc tên...]    Trạng thái: [Tất cả ▾]        │
└─────────────────────────────────────────────────────────────────────┘
```

### Bảng nhóm theo kiểu chạy

```
▼ REALTIME  (2 trigger)
┌──────────────────────┬───────────────────────┬──────────┬─────────────────────────┬──────────┐
│ Code                 │ Tên                   │ Source   │ Trạng thái              │ Hành động│
├──────────────────────┼───────────────────────┼──────────┼─────────────────────────┼──────────┤
│ SIM_ACTIVATED        │ Kích hoạt SIM         │ BSS      │ ● Active                │ [Xem]    │
│ LOC_TRAVEL_PROV      │ Đến tỉnh du lịch      │ OCS      │ ○ Inactive (Không dùng) │ [Xem]    │
└──────────────────────┴───────────────────────┴──────────┴─────────────────────────┴──────────┘

▼ NEAR REALTIME  (1 trigger)
┌──────────────────────┬───────────────────────┬──────────┬───────────┬──────────┐
│ NO_APP_INSTALL_24H   │ Chưa cài app sau 24h  │ SuperApp │ ● Active  │ [Xem]    │
└──────────────────────┴───────────────────────┴──────────┴───────────┴──────────┘

▼ OFFLINE / BATCH  (2 trigger)
┌──────────────────────┬───────────────────────┬──────────┬───────────┬──────────┐
│ LOW_DATA_BALANCE     │ Data còn dưới 100MB   │ OCS      │ ● Active  │ [Xem]    │
│ SIM_NO_TXN_30D       │ SIM ngừng GD >30 ngày │ BSS      │ ● Active  │ [Xem]    │
└──────────────────────┴───────────────────────┴──────────┴───────────┴──────────┘
```

**Hành vi:**
- Nhấn tiêu đề nhóm `▼ REALTIME` → thu gọn/mở rộng toàn bộ trigger trong nhóm.
- Bộ lọc "Trạng thái" áp dụng cho tất cả nhóm.
- Tìm kiếm tức thì, làm nổi bật kết quả khớp; tự mở rộng nhóm chứa kết quả.
- Trigger Inactive: row grayed out, trạng thái hiển thị chip xám + label "Không còn sử dụng".
- Không có nút [+ Thêm], [Sửa], [Bật], [Tắt] với bất kỳ role nào.

### Modal Xem chi tiết Trigger

```
┌────────────────────────────────────────────────────────────────────┐
│ Chi tiết Sự kiện kích hoạt                                  [✕]   │
├────────────────────────────────────────────────────────────────────┤
│ A. Định danh                                                       │
│ ┌──────────────────────────────────────────────────────────────┐   │
│ │ Code: SIM_ACTIVATED              Kiểu: [Realtime]            │   │
│ │ Tên:  Kích hoạt SIM thành công   Source: BSS                 │   │
│ │ Trạng thái: ● Active                                         │   │
│ │ Mô tả: Fired khi khách hàng kích hoạt SIM mới lần đầu tiên  │   │
│ └──────────────────────────────────────────────────────────────┘   │
│                                                                    │
│ B. Tham số đầu ra                                                  │
│ ┌───────────────┬──────────────────┬────────┬───────┬────────────┐ │
│ │ Tham số       │ Mô tả            │ Định   │ Nguồn │ Ví dụ      │ │
│ │               │                  │ dạng   │       │ giá trị    │ │
│ ├───────────────┼──────────────────┼────────┼───────┼────────────┤ │
│ │ {{ten_kh}}    │ Họ tên đầy đủ KH │ text   │ BSS   │ Nguyễn A   │ │
│ │ {{loai_sim}}  │ Loại SIM         │ text   │ BSS   │ eSIM       │ │
│ │ {{ngay_kh}}   │ Ngày kích hoạt   │ date   │ BSS   │ 15/05/2026 │ │
│ │ {{so_dien_th}}│ Số điện thoại    │ text   │ BSS   │ 0987654321 │ │
│ └───────────────┴──────────────────┴────────┴───────┴────────────┘ │
│ ℹ Nhấn vào tên tham số để copy cú pháp vào clipboard              │
│                                                                    │
│ C. Thông tin vận hành                                              │
│ ┌──────────────────────────────────────────────────────────────┐   │
│ │ Kênh hỗ trợ: SMS  Push Notification  Zalo OA  Banner App    │   │
│ │                                                              │   │
│ │ Campaign đang dùng trigger này:                              │   │
│ │   • Chào mừng SIM mới — Q2 2026      ● Active               │   │
│ │   • Onboard KH eSIM tháng 5          ● Active               │   │
│ │   • Test trigger KH mới (draft)      ○ Draft                │   │
│ └──────────────────────────────────────────────────────────────┘   │
│                                                                    │
│                                              [Đóng]               │
└────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Toàn bộ nội dung chỉ đọc — không có ô nhập liệu, không có nút [Sửa] với bất kỳ role nào.
- Nhấn tên tham số (ví dụ `{{ten_kh}}`) → copy cú pháp vào clipboard + toast nhỏ "Đã copy {{ten_kh}}".
- Trigger Inactive vẫn hiển thị đầy đủ thông tin — QTV cần tra cứu params của trigger deprecated.
- Danh sách campaign hiển thị tất cả trạng thái (Active, Paused, Pending, Draft).

---

## Screen 6: Blacklist Management

### Mục tiêu
Quản lý danh sách số điện thoại bị chặn gửi tin theo campaign và kênh cụ thể (CVM Blacklist — riêng biệt với BSS DNC).

### 6A: Danh sách Blacklist

```
┌──────────────────────────────────────────────────────────────────────────┐
│ BLACKLIST                                                                │
├──────────────────────────────────────────────────────────────────────────┤
│ Campaign: [Tất cả ▾]   Kênh: [Tất cả ▾]   Nguồn: [Tất cả ▾]   [Lọc]   │
│ Sắp xếp mặc định: ngày thêm mới nhất lên đầu                           │
├──────────────────┬───────────────────┬────────┬──────────────────┬───────┤
│ Số điện thoại    │ Campaign          │ Kênh   │ Nguồn            │ Hành  │
│                  │                   │        │                  │ động  │
├──────────────────┼───────────────────┼────────┼──────────────────┼───────┤
│ 0987 xxx 001     │ Chào mừng SIM     │ Zalo OA│ Chọn trong camp. │ [Xóa] │
│ 0912 xxx 002     │ Hết data          │ SMS    │ Upload tệp       │ [Xóa] │
│ 0965 xxx 003     │ Hết data          │ Zalo OA│ Thêm thủ công    │ [Xóa] │
│ 0976 xxx 004     │ Nhắc nạp tiền     │ Push   │ Chọn trong camp. │ [Xóa] │
└──────────────────┴───────────────────┴────────┴──────────────────┴───────┘
│                              < 1 2 3 >                                   │
│                                                                          │
│ [+ Thêm thủ công]                         [📥 Upload danh sách]          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Bộ lọc Campaign + Kênh + Nguồn → lọc tức thì.
- Cột Nguồn: "Chọn trong campaign [MÃ], kênh [X]" / "Tải lên tệp" / "Thêm thủ công".
- Link 2 chiều: số được chọn từ campaign → tự xuất hiện ở đây; xóa từ đây → cảnh báo "Số này đang dùng trong campaign [X] kênh [Y]. Xóa sẽ không ảnh hưởng cấu hình trong campaign." → xác nhận thì xóa.
- `[Xóa]` → hộp thoại xác nhận → xóa ngay khỏi Blacklist Management (không xóa khỏi campaign).
- `[+ Thêm thủ công]` → mở cửa sổ Thêm thủ công.
- `[📥 Tải lên danh sách]` → mở cửa sổ Tải lên.

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
│      [Hủy]    [Xác nhận Tải lên]      │
└──────────────────────────────────────┘
```

**Hành vi:**
- Chọn file → phân tích ngay, hiển thị kết quả xem trước.
- Số sai định dạng: bỏ qua, không chặn tải lên.
- `[Xác nhận Tải lên]` → thêm vào blacklist + thông báo "Đã tải lên 245 số vào blacklist".

---

## Screen 7: Customer List

### Mục tiêu
Tra cứu nhanh danh sách khách hàng, tìm kiếm theo số điện thoại, vào Customer 360 để xem chi tiết.

### Bố cục

```
┌─────────────────────────────────────────────────────────────────────┐
│ KHÁCH HÀNG                                                          │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm số điện thoại...]                                           │
│                                                                     │
│ Trạng thái SIM: [Tất cả ▾]   Cài app: [Tất cả ▾]   DNC: [Tất cả ▾]│
│ Thứ tự hiển thị theo BSS trả về — CVM không sort thêm               │
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
- Bộ lọc Trạng thái SIM: Đang dùng / Không hoạt động / Tạm khóa.
- Bộ lọc Cài app: Có / Không.
- Bộ lọc DNC: Có DNC / Không có DNC.
- `[Xem 360]` → điều hướng sang Customer 360 của số đó.
- Dữ liệu từ BSS/OCS, không chỉnh sửa trực tiếp trong CVM.

---

## Screen 8: Customer 360

### Mục tiêu
Xem toàn bộ thông tin 1 khách hàng: hồ sơ, trạng thái kênh (phản hồi ngược từ Gateway), lịch sử nhận tin, trạng thái giới hạn tần suất.

### Bố cục 2 cột

```
┌─────────────────────────────────────────────────────────────────────┐
│ Khách hàng / 0987xxx001                                             │
│ ← Danh sách khách hàng                                              │
├────────────────────────────────┬────────────────────────────────────┤
│ THÔNG TIN KHÁCH HÀNG           │ TRẠNG THÁI KÊNH & LỊCH SỬ         │
├────────────────────────────────┼────────────────────────────────────┤
│ Số điện thoại  0987xxx001      │ TRẠNG THÁI KÊNH (Sync-back)        │
│ Loại SIM       eSIM            │ ┌──────────┬──────────┬──────────┐ │
│ Trạng thái     Active          │ │ Kênh     │ Trạng thái│ Cập nhật│ │
│ Gói cước       D50 (HH 18/05) │ ├──────────┼──────────┼──────────┤ │
│ Ngày KH SIM    01/04/2026      │ │ Zalo OA  │⛔ Blocked│05/05 09:32│ │
│ Thiết bị       Android         │ │ SMS      │✅ Active │ —        │ │
│ Cài app        Chưa            │ │ USSD     │✅ Active │ —        │ │
│ Đăng nhập app  Chưa            │ │ Push     │✅ Active │ —        │ │
│                                │ └──────────┴──────────┴──────────┘ │
│ THỐNG KÊ SỬ DỤNG               │ ⓘ Tự cập nhật từ phản hồi Gateway. │
│ Data còn lại   85 MB           │   Reset về Active khi KH unblock.  │
│ Lưu lượng hôm nay  320 MB      │                                    │
│ Số dư          45,000 VND      │ THROTTLING                         │
│ Số lần nạp     3               │ Tin nhắn hôm nay   2 / 3 (còn 1)  │
│ Số lần gia hạn 2               │ Cooldown active    Không           │
│ Cuộc gọi thất bại  0           │ BSS DNC Flag       Không           │
│ Ngày sinh nhật 15/08           │                                    │
│ Nghề nghiệp    Nhân viên VP    │ PHÂN KHÚC                          │
│                                │ [Gen Z User] [Sắp hết data]        │
│                                │                                    │
│                                │ LỊCH SỬ NHẬN TIN                   │
│                                │ 10/05 09:32  Chào mừng SIM         │
│                                │              Zalo OA · ✓ Delivered  │
│                                │                                    │
│                                │ 08/05 14:10  Nạp thẻ lỗi           │
│                                │              SMS · ✓ Delivered      │
│                                │                                    │
│                                │ 07/05 08:45  Hết data              │
│                                │              Zalo OA · ⛔ Blocked→Fallback│
│                                │ 07/05 08:46  Hết data              │
│                                │              SMS · ✓ Delivered      │
│                                │                                    │
│                                │ [Xem đầy đủ lịch sử →]             │
└────────────────────────────────┴────────────────────────────────────┘
```

**Hành vi:**
- Toàn bộ dữ liệu chỉ đọc — không chỉnh sửa trong CVM.
- Trạng thái kênh tự cập nhật từ Gateway; reset về Active khi KH unblock.
- Chuyển kênh dự phòng hiển thị 2 dòng riêng: dòng gốc "Bị chặn → Chuyển kênh" + dòng kênh dự phòng bên dưới (thời điểm cách nhau vài giây).
- Phân khúc hiển thị dạng thẻ nhãn; nếu KH không thuộc phân khúc nào → hiển thị "Không có".
- `[Xem đầy đủ lịch sử →]` mở khung trượt từ phải với phân trang và lọc theo kênh và campaign.

---

## Screen 9: Report / Analytics

### Mục tiêu
Phân tích hiệu quả campaign theo thời gian — dành cho QTV và cấp quản lý xem phân tích, so sánh campaign, phân tích theo phân khúc và phát hiện rủi ro spam/fatigue.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────────┐
│ ANALYTICS & REPORT                                       [Xuất Excel]   │
├─────────────────────────────────────────────────────────────────────────┤
│ FILTER BAR                                                              │
│ Thời gian: [7 ngày ▾]  Campaign: [Tất cả ▾]  Segment: [Tất cả ▾]       │
│            [Tháng này]  [So sánh: ON ▾]       Kênh: [Tất cả ▾]         │
├─────────────────────────────────────────────────────────────────────────┤
│ TAB NAVIGATION                                                          │
│ [Hiệu quả gửi tin] [Tương tác] [So sánh Campaign] [Phân khúc] [Phễu] [Spam & Quá tải] │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 1: Hiệu quả gửi tin

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CHỈ SỐ TỔNG HỢP (theo bộ lọc đang chọn)                                │
│ ┌────────────────┐  ┌────────────────┐  ┌────────────┐  ┌────────────┐  │
│ │ Tổng đã gửi   │  │ Đã tới đích    │  │ Thất bại   │  │ Tỉ lệ đã  │  │
│ │   124,500      │  │   119,940      │  │  4,560     │  │ tới đích   │  │
│ │                │  │   96.3%        │  │  3.7% ⚠   │  │  96.3%     │  │
│ └────────────────┘  └────────────────┘  └────────────┘  └────────────┘  │
│ Tỉ lệ đã tới đích = Đã tới đích / Tổng đã gửi × 100%                   │
│ Thất bại = gateway trả lỗi hoặc timeout sau 3 lần retry                 │
├─────────────────────────────────────────────────────────────────────────┤
│ ĐÃ GỬI vs ĐÃ TỚI ĐÍCH — Biểu đồ vùng theo thời gian                    │
│                                                                         │
│  50K ┤                                    ╭──────╮                      │
│  40K ┤               ╭──────────╮        ╭╯      ╰─────╮                │
│  30K ┤          ╭────╯  Đã gửi  ╰────────╯  Đã tới đích╰─────          │
│  20K ┤    ╭─────╯                                                       │
│  10K ┤────╯                                                             │
│      └──────────────────────────────────────────────────────── ngày     │
│        07  08  09  10  11  12  13                                       │
│  ● Đã gửi (line trên)  ● Đã tới đích (vùng xanh)  ● Thất bại (vùng đỏ)│
├─────────────────────────────────────────────────────────────────────────┤
│ XU HƯỚNG TỈ LỆ ĐÃ TỚI ĐÍCH  │ XU HƯỚNG THẤT BẠI (phân loại lý do)     │
│ Biểu đồ đường — % theo ngày  │ Biểu đồ cột chồng                        │
│                              │                                          │
│  98% ┤      ╭───╮            │ 2K ┤  ■■ ■■■ ■■ ■■■ ■■■ ■■ ■■■          │
│  96% ┤──────╯   ╰────────── │ 1K ┤  ░░ ░░░ ░░ ░░░ ░░░ ░░ ░░░          │
│  94% ┤                      │  0 └──────────────────────────── ngày    │
│      └──────────── ngày      │ ■ Lỗi cổng kết nối  ░ Người dùng chặn   │
│ ─── mục tiêu 95%             │ ▒ DNC/Blacklist  ▓ Hết thời gian kết nối│
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 2: Tương tác (Engagement)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CHỈ SỐ TƯƠNG TÁC                                                        │
│ ┌──────────────┐  ┌────────────┐  ┌────────────────┐  ┌──────────────┐  │
│ │ Tỉ lệ mở    │  │ Tỉ lệ nhấp │  │ Tỉ lệ chuyển   │  │ Kênh hiệu   │  │
│ │  38.4%       │  │ (CTR)      │  │ đổi            │  │ quả nhất    │  │
│ │ ↑2.1% vs     │  │  12.1%     │  │  8.3%          │  │ Push 97.7%  │  │
│ │ tuần trước   │  │ ↑0.8%      │  │ ↑0.4%          │  │             │  │
│ └──────────────┘  └────────────┘  └────────────────┘  └──────────────┘  │
│ Tỉ lệ mở = Số mở / Đã tới đích × 100% (Push, Email có tracking)         │
│ Tỉ lệ nhấp = Số click / Đã tới đích × 100%                             │
│ Tỉ lệ chuyển đổi = Hoàn thành mục tiêu / Đã tới đích × 100% (24h)     │
│ Kênh hiệu quả nhất = kênh có Tỉ lệ chuyển đổi cao nhất trong kỳ       │
├─────────────────────────────────────────────────────────────────────────┤
│ HIỆU SUẤT THEO KÊNH — Biểu đồ cột                                       │
│                                                                         │
│ Push    ████████████████████████████████████  Mở 52%  Nhấp 18%        │
│ Zalo OA ████████████████████████████████      Mở 44%  Nhấp 15%        │
│ SMS     ████████████████████                  Mở 22%  Nhấp  8%        │
│ USSD    █████████████                         Mở 15%  Nhấp  5%        │
│ Banner  ████████                              Mở  9%  Nhấp  3%        │
│ Email   ██████████████████                    Mở 21%  Nhấp  9%        │
│                    ■ Tỉ lệ mở  ░ Tỉ lệ nhấp  ▒ Tỉ lệ chuyển đổi      │
├─────────────────────────────────────────────────────────────────────────┤
│ XU HƯỚNG TƯƠNG TÁC — Biểu đồ đường theo ngày                            │
│ ● Tỉ lệ mở  ● Tỉ lệ nhấp  ● Tỉ lệ chuyển đổi  (7 ngày, chọn kênh)    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 3: So sánh Campaign

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SO SÁNH CAMPAIGN — chọn tối đa 5 campaign để so sánh                   │
│ [+ Thêm campaign để so sánh ▾]                                          │
├─────────────────────────────────────────────────────────────────────────┤
│ BIỂU ĐỒ CỘT — Tỉ lệ gửi thành công / Tỉ lệ chuyển đổi theo campaign   │
│                                                                         │
│ Nhắc nạp tiền    ████████████████████████████████  96.5% / 9.1%         │
│ Hết hạn data     ███████████████████████████████   95.4% / 7.8%         │
│ Welcome eSIM     █████████████████████████████     93.1% / 8.3%         │
│ LOW_DATA batch   ████████████████████████████████  94.2% / 6.2%         │
│                     ■ Tỉ lệ thành công  ░ Tỉ lệ chuyển đổi             │
├─────────────────────────────────────────────────────────────────────────┤
│ BẢNG SO SÁNH CHI TIẾT                                                   │
│ ┌─────────────────┬────────┬──────────────┬──────┬──────────┬──────────┐ │
│ │ Campaign        │ Đã gửi │ Đã tới đích  │ Mở   │ Chuyển   │ Chi phí  │ │
│ │                 │        │              │      │ đổi      │ SMS      │ │
│ ├─────────────────┼────────┼──────────────┼──────┼──────────┼──────────┤ │
│ │ Nhắc nạp tiền   │ 32,100 │ 30,980       │38.4% │  9.1%    │ 480K VND │ │
│ │ Hết hạn data    │ 28,400 │ 27,100       │35.2% │  7.8%    │ 360K VND │ │
│ │ Welcome eSIM    │ 20,600 │ 19,400       │42.1% │  8.3%    │ 240K VND │ │
│ │ LOW_DATA batch  │ 24,800 │ 23,900       │29.6% │  6.2%    │ 720K VND │ │
│ └─────────────────┴────────┴──────────────┴──────┴──────────┴──────────┘ │
│ Chi phí SMS = ceil(số ký tự / 160) × đơn giá × số KH thành công qua SMS│
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 4: Phân khúc (Segment)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ HIỆU SUẤT PHÂN KHÚC — Deliverability & Chuyển đổi theo phân khúc        │
│                                                                         │
│ ┌──────────────────┬────────┬──────────────┬──────┬──────────┐          │
│ │ Phân khúc        │ Tiếp   │ Đã tới đích  │ Mở   │ Chuyển   │          │
│ │                  │ cận    │              │      │ đổi      │          │
│ ├──────────────────┼────────┼──────────────┼──────┼──────────┤          │
│ │ Gen Z (18–25)    │ 18,450 │ 17,800       │52.3% │  11.2%   │          │
│ │ Sắp hết data     │  8,920 │  8,600       │44.8% │   8.9%   │          │
│ │ ARPU cao         │  5,600 │  5,480       │38.1% │  14.6%   │          │
│ │ Nguy cơ rời mạng │  2,150 │  2,050       │22.4% │   3.1%   │          │
│ └──────────────────┴────────┴──────────────┴──────┴──────────┘          │
│ Tiếp cận = số KH trong phân khúc thoả điều kiện lọc tại thời điểm chạy │
├─────────────────────────────────────────────────────────────────────────┤
│ SO SÁNH PHÂN KHÚC — Biểu đồ cột                                         │
│                                                                         │
│ Gen Z        ████████████████████████████  52.3% mở  11.2% chuyển đổi │
│ ARPU cao     ██████████████████████        38.1% mở  14.6% chuyển đổi │
│ Sắp hết data ██████████████████████        44.8% mở   8.9% chuyển đổi │
│ Nguy cơ rời  ██████████                    22.4% mở   3.1% chuyển đổi │
│                           ■ Tỉ lệ mở  ░ Tỉ lệ chuyển đổi              │
├─────────────────────────────────────────────────────────────────────────┤
│ PHÂN BỔ THIẾT BỊ (dựa trên push token)                                  │
│ Android: 62%  ████████████████████████████████                          │
│ iOS:     28%  ██████████████                                            │
│ Khác:    10%  █████  (không có push token)                              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 5: Phễu (Funnel)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ PHỄU CHUYỂN ĐỔI  ·  [Chọn campaign ▾]  ·  [Chọn kênh ▾]               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│ Audience đủ điều kiện    ████████████████████████████  32,100  100%     │
│                          ↓ Rời bỏ  7.8%  (−2,510 DNC/BL)               │
│ Đã gửi                   █████████████████████████     29,590   92.2%   │
│                          ↓ Rời bỏ  7.7%  (−2,280 không đến nơi)        │
│ Đã tới đích              ███████████████████████       27,310   85.1%   │
│                          ↓ Rời bỏ 47.3%  (−12,930 không mở)  ← cao nhất│
│ Đã mở / Đã nhấp          ████████████                  14,380   44.8%   │
│                          ↓ Rời bỏ 35.6%  (−5,120 không hành động)      │
│ Đã cài app               ████████                       9,260   28.8%   │
│                          ↓ Rời bỏ 37.3%  (−3,450 không mua)            │
│ Đã mua gói cước          █████                          5,810   18.1%   │
│                                                                         │
│ % Rời bỏ = (bước N − bước N+1) / bước N × 100%                         │
├─────────────────────────────────────────────────────────────────────────┤
│ PHÂN TÍCH ĐIỂM RỜI BỎ                                                   │
│ Điểm rời bỏ lớn nhất: Đã tới đích → Đã mở (−47.3%)                  │
│ → Gợi ý: Thử A/B test tiêu đề Push để tăng tỉ lệ mở                   │
│ → So sánh kênh: Push mở 52% vs SMS mở 22% vs Zalo mở 44%              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Tab 6: Spam & Quá tải

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CHỈ SỐ SPAM & QUÁ TẢI TIN NHẮN                                          │
├─────────────────────────────────────────────────────────────────────────┤
│ XU HƯỚNG OPT-OUT & BỊ CHẶN — Biểu đồ đường theo ngày                   │
│ Tỉ lệ opt-out = Số KH opt-out / Đã tới đích × 100%                     │
│                                                                         │
│  500 ┤                              ╭───╮                               │
│  400 ┤              ╭──╮            │   │ ← Spike cần điều tra          │
│  300 ┤    ╭─────────╯  ╰────────────╯   ╰──                             │
│  200 ┤────╯  Opt-out                                                    │
│  100 ┤───────────────────────────────── Blacklist mới                   │
│      └──────────────────────────────────────────────── ngày             │
│        07  08  09  10  11  12  13                                       │
├─────────────────────────────────────────────────────────────────────────┤
│ CHỈ SỐ TẦN SUẤT                │ ĐIỂM RỦI RO SPAM                       │
│                                │                                        │
│ TB tin/người/ngày:   1.8       │ ● RỦI RO THẤP  (24/100)               │
│ TB tin/người/tuần:   4.2       │ ████░░░░░░░░░░░░░░░░░░░░               │
│ Tối đa tin/người/ngày: 5       │                                        │
│ Người nhận > 3 tin: 8.4%       │ Cảnh báo khi điểm > 60 → gauge cam/đỏ│
│                                │ Trọng số: opt-out (40%)               │
│ PHÂN PHỐI SỐ TIN/NGƯỜI         │ + blacklist mới (30%)                 │
│ (trục x = số tin, trục y = KH) │ + CTR giảm (30%)                      │
│  ████░░░░   1 tin: 42%         │                                        │
│  ████░░░    2 tin: 31%         │ [Xem chi tiết chỉ số →]               │
│  ███░░      3 tin: 18%         │                                        │
│  █          4+ tin:  9%        │                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Hành vi Report
- Bộ lọc áp dụng đồng thời cho tất cả tab và biểu đồ.
- Bộ lọc thời gian: Hôm nay / 7 ngày / 30 ngày / Tháng này / Tùy chỉnh (chọn khoảng ngày).
- `[So sánh: ON]` → hiện thêm đường/cột kỳ trước trên tất cả chart để so sánh trực quan.
- `[Xuất Excel]` → xuất toàn bộ dữ liệu theo bộ lọc và tab hiện tại ra file .xlsx.
- Nhấn vào phân khúc/campaign trong bảng → filter tương ứng áp dụng trên tất cả tab.
- **Phân tích điểm rời bỏ**: tự động highlight bước có tỉ lệ rời bỏ cao nhất (tô nền vàng) + gợi ý hành động cụ thể.

---

## Screen 10: Priority Matrix _(UC-PRIORITY-01)_

### Mục tiêu
Cho phép Admin xem và sắp xếp lại thứ tự ưu tiên của tất cả campaign Active. Hệ thống dùng thứ tự này để chọn campaign khi nhiều campaign cùng match một KH.

### Bố cục

```
┌─────────────────────────────────────────────────────────────────────┐
│ Cài đặt  >  Priority Matrix                                         │
├─────────────────────────────────────────────────────────────────────┤
│ Priority Matrix — Thứ tự ưu tiên Campaign                           │
│ Khi nhiều campaign cùng khớp một khách hàng, hệ thống chọn         │
│ campaign có số ưu tiên nhỏ nhất.                          [ⓘ Help] │
├──────────────────────────────────────────────────────────────────── │
│                                                    [Lưu thứ tự ▼]  │
├────┬──────────────────────────────┬────────────────┬───────────────┤
│    │ Tên campaign                 │ Mã kịch bản    │ Độ ưu tiên    │
├────┼──────────────────────────────┼────────────────┼───────────────┤
│ ≡  │ Welcome eSIM Q2/2026         │ CVM-202506-001 │ [1__]         │
│ ≡  │ Khuyến mãi nạp tiền T5       │ CVM-202505-003 │ [2__]         │
│ ≡  │ Hết data – nhắc mua thêm     │ CVM-202506-004 │ [3__]         │
│ ≡  │ Sinh nhật tháng 6            │ CVM-202506-007 │ [4__]         │
│ ≡  │ Upsell gói 4G Pro            │ CVM-202506-010 │ [5__]         │
├────┴──────────────────────────────┴────────────────┴───────────────┤
│ ★ = campaign đang được ưu tiên cao nhất                             │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Icon `≡` ở cột trái: kéo thả để sắp xếp lại thứ tự — ô Độ ưu tiên tự cập nhật theo vị trí mới
- Ô Độ ưu tiên: có thể nhập số trực tiếp; bảng tự sắp xếp lại sau khi nhập
- Chỉ hiển thị campaign Active; Draft/Pending/Paused/Ended không xuất hiện
- Nhấn `[Lưu thứ tự]` → hộp thoại "Thứ tự ưu tiên mới áp dụng ngay cho sự kiện tiếp theo. Xác nhận?" → lưu → thông báo "Đã cập nhật ✓"
- Campaign mới Active: tự thêm vào cuối danh sách với số ưu tiên = cao nhất hiện tại + 1
- Campaign bị Paused/Ended: tự ẩn khỏi danh sách; số ưu tiên các campaign còn lại không thay đổi

---

## Screen 11: Admin

### Mục tiêu
Màn hình dành riêng cho vai trò Admin — duyệt/từ chối campaign và cấu hình trigger. Admin xem được tất cả màn hình như QTV nhưng không có quyền tạo/sửa campaign.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ ADMIN                                                               │
├─────────────────────────────────────────────────────────────────────┤
│ Tab: [Duyệt Campaign]  [Cấu hình Trigger]                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Tab Duyệt Campaign

```
┌─────────────────────────────────────────────────────────────────────┐
│ CAMPAIGN CHỜ DUYỆT                                                  │
├─────────────────────────────────────────────────────────────────────┤
│ [🔍 Tìm campaign...]                                                 │
├──────────────────────────────┬────────────┬────────────┬────────────────────────┤
│ Tên / Mã Campaign            │ Người tạo  │ Gửi duyệt  │ Hành động              │
├──────────────────────────────┼────────────┼────────────┼────────────────────────┤
│ Welcome eSIM Q2/2026         │ QTV Mkt    │12/05 09:15 │ [Xem] [Duyệt] [Từ chối]│
│ CVM-WELCOME-ESIM-Q2-2026     │            │            │                        │
├──────────────────────────────┼────────────┼────────────┼────────────────────────┤
│ Nhắc nạp tiền T5             │ QTV Sales  │11/05 14:30 │ [Xem] [Duyệt] [Từ chối]│
│ CVM-REMIND-TOPUP-05-2026     │            │            │                        │
└──────────────────────────────┴────────────┴────────────┴────────────────────────┘
                                                          < 1 2 >  [20/trang ▾]
```

**Hành vi:**
- `[Xem]` → mở Campaign Detail View chỉ đọc.
- `[Duyệt]` → hộp thoại xác nhận: "Duyệt campaign [tên]? Campaign sẽ chuyển sang Active ngay." [Hủy] / [Duyệt] → chuyển Active + thông báo "Đã duyệt ✓" + row biến mất khỏi danh sách.
- `[Từ chối]` → hộp thoại nhập lý do (ô văn bản bắt buộc): [Hủy] / [Từ chối] → trạng thái Draft + lý do lưu lại + thông báo "Đã từ chối" + QTV nhận thông báo.
- Nếu không có campaign nào chờ duyệt → hiện trạng thái trống: "Không có campaign nào chờ duyệt."
- Admin không có nút `[+ Tạo Campaign]`, `[Sửa]`, `[Lưu Nháp]`, `[Gửi duyệt]` ở bất kỳ màn hình nào.

### Tab Cấu hình Trigger

Nội dung giống Screen 5: Trigger Management — Admin có đầy đủ quyền thêm/sửa/bật/tắt trigger. QTV khi vào Screen 5 qua nav "Trigger" chỉ thấy bảng danh sách, không có `[+ Thêm Trigger]`, `[Sửa]`, `[Tắt]`.

---

## Screen 12: Settings

### Mục tiêu
Cấu hình hệ thống CVM — chỉ Admin có quyền thay đổi. QTV không thấy màn này trong navigation.

### Bố cục tổng quan

```
┌─────────────────────────────────────────────────────────────────────┐
│ CÀI ĐẶT HỆ THỐNG                                                    │
├─────────────────────────────────────────────────────────────────────┤
│ Tab: [Frequency Cap]  [Phân quyền]  [Priority Matrix]               │
└─────────────────────────────────────────────────────────────────────┘
```

### Tab Frequency Cap

```
┌─────────────────────────────────────────────────────────────────────┐
│ GIỚI HẠN TẦN SUẤT NHẬN TIN — áp dụng toàn hệ thống                 │
├─────────────────────────────────────────────────────────────────────┤
│ Tối đa số tin / KH / ngày:      [3___]  tin                         │
│ Tối đa số tin / KH / tuần:      [7___]  tin                         │
│ Cooldown sau khi đạt giới hạn:  [24__]  giờ                         │
│                                                                     │
│ ⚠ Thay đổi áp dụng cho sự kiện tiếp theo — không hồi tố            │
│                                                 [Lưu cài đặt]      │
└─────────────────────────────────────────────────────────────────────┘
```

**Hành vi:**
- Chỉ nhập số nguyên dương; để trống hoặc nhập 0 → lỗi trực tiếp.
- `[Lưu cài đặt]` → thông báo "Đã lưu cài đặt ✓".

### Tab Phân quyền

```
┌─────────────────────────────────────────────────────────────────────┐
│ PHÂN QUYỀN HỆ THỐNG  (chỉ đọc — thay đổi qua hệ thống IAM)         │
├──────────────────────────────┬────────────────┬─────────────────────┤
│ Chức năng                    │ Admin          │ QTV Marketing       │
├──────────────────────────────┼────────────────┼─────────────────────┤
│ Xem Dashboard                │ ✓              │ ✓                   │
│ Xem tất cả màn hình          │ ✓              │ ✓                   │
│ Tạo / Sửa Campaign           │ ✗              │ ✓                   │
│ Gửi duyệt campaign           │ ✗              │ ✓                   │
│ Duyệt / Từ chối Campaign     │ ✓              │ ✗                   │
│ Xem Trigger catalog           │ ✓ (chỉ xem)    │ ✓ (chỉ xem)         │
│ Quản lý Blacklist            │ ✓              │ ✓                   │
│ Xem Report                   │ ✓              │ ✓                   │
│ Cài đặt hệ thống (Settings)  │ ✓              │ ✗                   │
└──────────────────────────────┴────────────────┴─────────────────────┘
```

### Tab Priority Matrix

Nội dung giống Screen 10: Priority Matrix — đặt ở đây để Admin truy cập từ Settings thay vì nav riêng.

---

## Edge Cases

| Tình huống | Hành vi |
|---|---|
| Chỉ có 1 trigger | Tự động là số 1; ẩn Logic OR/AND và khối xung đột |
| Đổi thứ tự trigger số 1 | Thẻ tham số cập nhật; tham số không còn hợp lệ tô đỏ trong tin nhắn |
| Xóa trigger số 1 | Trigger tiếp theo tự lên số 1; các số bên dưới cập nhật lại |
| Soạn message xong rồi đổi trigger | Giữ nội dung, chỉ tô đỏ tham số không hợp lệ |
| Thêm kênh đã có tab | Không cho thêm; kênh đó bị vô hiệu hóa trong danh sách |
| SMS vượt 160 ký tự | Bộ đếm đỏ + chỉ báo "X tin SMS"; không chặn lưu; cost tính `ceil(ký tự / 160)` |
| SMS 300/160 ký tự | Hiển thị "2 tin SMS"; cost = reach SMS × 2 |
| USSD chưa cấu hình | Chỉ báo `○` trên tab; chỉ báo đỏ trên tiêu đề tăng số vấn đề |
| Gửi duyệt còn issue | Nút disabled + hiện danh sách vấn đề; click issue → cuộn đến đúng mục |
| Tắt trigger đang dùng trong Active campaign | Hộp thoại xác nhận + danh sách campaign bị ảnh hưởng |
| Trigger priority trùng nhau | Cảnh báo trực tiếp; yêu cầu kéo-thả sắp xếp lại trước khi gửi duyệt |
| KH thỏa nhiều trigger cùng lúc (OR mode) | Chỉ gửi message của trigger có thứ tự số 1 |
| Chọn 0 phân khúc | Mặc định gửi T-ALL (tất cả KH); hiển thị hộp thoại xác nhận trước khi lưu |
| Bật blacklist nhưng chưa chọn tệp | Chặn lưu; nút Gửi duyệt bị vô hiệu hóa |
| Bật whitelist nhưng chưa chọn tệp | Blocking validation |
| KH thuộc audience nhưng ngoài whitelist | Loại khỏi reach; không gửi |
| Whitelist có số sai định dạng | Bỏ qua; hiển thị số lượng bị bỏ qua trong cửa sổ xem trước |
| Xóa kênh ở Message Matrix | Xác nhận → bỏ tick kênh đó trong "Kênh & Lịch gửi"; lịch riêng của kênh đó bị xóa |
| Bỏ tick kênh ở "Kênh & Lịch gửi" khi tab đã có nội dung | Hộp thoại cảnh báo nội dung sẽ mất → xác nhận thì ẩn tab và xóa lịch kênh đó |
| Dừng khẩn (Dừng campaign đang Active) | Hộp thoại cảnh báo tin nhắn đang chờ gửi sẽ bị hủy |
| Chuyển Lịch chung → Lịch riêng | Preset giá trị lịch chung vào tất cả kênh; user tự chỉnh từng kênh |
| Chuyển Lịch riêng → Lịch chung | Hộp thoại xác nhận; sau xác nhận: áp lịch chung ghi đè tất cả kênh |
| Phân khúc lọc thêm → lượng tiếp cận sau lọc = 0 | Cảnh báo trực tiếp trong thẻ phân khúc: "Không có KH nào đáp ứng điều kiện lọc này" |
| KH thuộc phân khúc lúc thiết lập nhưng rời phân khúc lúc trigger | KH bị loại khỏi reach — phân khúc được đánh giá lại tại thời điểm trigger |
| Số thuê bao được chọn vào BL campaign đã tồn tại trong Blacklist Management | Không trùng lặp; cập nhật nguồn nếu cần |
| Admin từ chối campaign → QTV sửa và gửi duyệt lại | Trạng thái Draft → QTV sửa → Gửi duyệt → Pending lại trong hàng chờ Admin |
| QTV truy cập /admin hoặc /settings | Chuyển hướng về /dashboard + thông báo "Bạn không có quyền truy cập trang này" |
| Tab biến thể chưa chọn phân khúc | Chặn lưu; không lưu được campaign |
| Thời gian gửi "Vào lúc HH:MM ngày T+0" mà đã qua giờ | Hệ thống xử lý như blackout: đưa vào hàng chờ sang ngày T+1 |

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
                 Kênh & Lịch gửi (chọn kênh + lịch chung / riêng + Blackout)
                 S4. Message Matrix (soạn nội dung theo kênh)
                 Kênh & Lịch gửi (+ Blackout per kênh)
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

---

## INTERACTION MAP

> Toàn bộ thao tác trong mô hình. Định dạng: `[Nút / Phần tử] trên [Màn hình] → [Kết quả]`

---

### Navigation — Sidebar

| Thao tác | Kết quả |
|---|---|
| Nav "Dashboard" | chuyển trang → /dashboard |
| Nav "Campaign" | chuyển trang → /campaigns |
| Nav "Template" | chuyển trang → /templates |
| Nav "Trigger" | navigate → /triggers |
| Nav "Khách hàng" | chuyển trang → /customers |
| Nav "Blacklist" | navigate → /blacklist |
| Nav "Report" | chuyển trang → /report |
| Nav "Admin" | chuyển trang → /admin |
| Nav "Settings" | chuyển trang → /settings |

---

### Screen 1 — Dashboard

| Thao tác | Kết quả |
|---|---|
| Dòng campaign trong bảng "TOP ACTIVE CAMPAIGNS" | chuyển trang → /campaigns |
| `[Xem tất cả →]` dưới bảng TOP ACTIVE CAMPAIGNS | chuyển trang → /campaigns |
| `[Xem chi tiết →]` trong funnel | chuyển trang → /report |
| `[Live ● pause]` trên timeline | chuyển đổi: dừng luồng / tiếp tục luồng |
| `[Xem hàng chờ]` trong Hàng chờ & Tồn đọng | thông báo "Tính năng đang phát triển" |
| `[Xem tất cả →]` trong Top System Errors | thông báo "Tính năng đang phát triển" |

---

### Screen 2 — Campaign List

| Thao tác | Kết quả |
|---|---|
| `[+ Tạo Campaign]` | chuyển trang → /campaigns/new |
| `[Xem]` (bất kỳ dòng) | chuyển trang → /campaigns/:id/detail |
| `[Sửa]` (dòng Draft) | chuyển trang → /campaigns/:id/edit |
| `[Dừng]` (dòng Active) | mở hộp thoại xác nhận: tiêu đề "Dừng campaign?", nội dung "Tin nhắn đang trong hàng chờ sẽ bị hủy. Không thể hoàn tác.", nút [Hủy] / [Xác nhận Dừng] (đỏ) → xác nhận: trạng thái → Paused, button → [Bật], thông báo "Campaign đã dừng" |
| `[Bật]` (dòng Paused) | không cần xác nhận → chuyển Active, nút đổi thành [Dừng], thông báo "Campaign đã kích hoạt lại" |
| Thẻ `+1 ⓘ` trên cột Trigger | mở danh sách nhỏ liệt kê trigger thứ 3: "LOC_TRAVEL_PROV · Đến tỉnh du lịch · OCS · Realtime" |
| Gõ vào ô tìm kiếm | lọc tức thì theo tên campaign / mã / trigger code |
| Nhấn thẻ lọc (Active / Draft / ...) | lọc bảng theo trạng thái; nhấn lại để bỏ |

---

### Screen 2B — Campaign Detail View

| Thao tác | Kết quả |
|---|---|
| `← Campaign` breadcrumb | chuyển trang → /campaigns |
| `[Đóng]` | chuyển trang → /campaigns |
| `[Sửa]` (chỉ hiện khi Draft) | chuyển trang → /campaigns/:id/edit |
| `[Dừng]` (chỉ hiện khi Active) | mở hộp thoại xác nhận giống Campaign List |
| `[Bật]` (chỉ hiện khi Paused) | kích hoạt lại như Campaign List |
| Tab kênh [Push] / [Zalo OA] / [SMS] / ... | hiện nội dung message của tab đó (tất cả 6 tab đều nhấn được) |
| `[Xem]` bên cạnh BL_ESIM_Q2_2026 | mở cửa sổ xem trước tệp blacklist (chỉ đọc): danh sách số + Hợp lệ / Trùng / Sai định dạng. Nút [Đóng] |

---

### Screen 3 — Campaign Builder

| Thao tác | Kết quả |
|---|---|
| `← Campaign` breadcrumb | chuyển trang → /campaigns |
| `[Lưu Nháp]` | thông báo "Đã lưu nháp ✓" |
| `[Gửi duyệt →]` khi còn issue | bị vô hiệu hóa; di chuột → hiện danh sách từng vấn đề |
| `[Gửi duyệt →]` khi hết issue | mở hộp thoại xác nhận: "Gửi campaign để duyệt?" [Hủy] / [Gửi duyệt]. Xác nhận → chuyển trang về /campaigns, trạng thái → Pending, thông báo "Đã gửi duyệt ✓" |
| `[Thu gọn]` trên tiêu đề mục | thu gọn nội dung mục; chữ đổi thành [Mở rộng] |
| `[Mở rộng]` trên section header | mở rộng lại |
| **Section 2 — Trigger** | |
| `[+ Chọn trigger ▾]` | mở danh sách tìm kiếm trigger; chọn → thêm thẻ trigger vào danh sách |
| `[✕]` trên thẻ trigger | xóa trigger đó; các trigger bên dưới tự đánh số lại |
| Kéo `[≡]` trên thẻ trigger | sắp xếp lại; số thứ tự tự cập nhật theo vị trí mới |
| Radio "AND" (nếu đã có biến thể) | mở hộp thoại xác nhận: "Chuyển sang AND mode sẽ xóa toàn bộ Biến thể đối tượng. Tiếp tục?" [Hủy] / [Xác nhận] → xác nhận: xóa biến thể, chuyển sang AND; hủy: giữ OR |
| Radio "AND" (chưa có biến thể) | chuyển ngay, không cần xác nhận |
| **Section 3 — Audience (cột phải)** | |
| `[🔍 Tìm phân khúc... ▾]` | mở danh sách chọn phân khúc; chọn → thêm thẻ phân khúc |
| `[×]` trên thẻ phân khúc | xóa phân khúc; lượng tiếp cận cập nhật lại |
| `[Mở rộng]` Điều kiện lọc | hiện các dòng điều kiện lọc; `[+ Thêm điều kiện]` → thêm dòng mới |
| `[↻ Tính lại]` | hiện vòng xoay 500ms → hiện lại lượng tiếp cận |
| **Section 4 — Message Matrix** | |
| Nhấn tab kênh | hiện thẻ soạn nội dung của tab đó |
| `[×]` trên tab kênh (chưa có nội dung) | xóa tab ngay + bỏ tick kênh trong "Kênh & Lịch gửi" |
| `[×]` trên tab kênh (đã có nội dung) | hộp thoại xác nhận "Bỏ kênh [X]? Nội dung đã soạn sẽ mất." [Hủy] / [Bỏ kênh] → xác nhận: xóa tab + bỏ tick kênh trong "Kênh & Lịch gửi" |
| `[+ Kênh]` | danh sách liệt kê kênh chưa có tab; chọn → thêm tab mới + tự tick kênh trong "Kênh & Lịch gửi" |
| Nhấn vào tham số trong thẻ | chèn `{{tên_param}}` vào vị trí con trỏ trong ô nhập đang được chọn |
| `[Tải lên]` trong ô tải ảnh | mô phỏng: hiện ảnh thu nhỏ + [Xóa][Đổi] |
| `[Chọn thư viện]` | mở cửa sổ thư viện ảnh: lưới ảnh mẫu, chọn → hiện ảnh thu nhỏ trong thẻ |
| `[Xóa]` trên ảnh thu nhỏ | xóa ảnh, trở về trạng thái trống |
| Gõ vào Tiêu đề / Nội dung | bộ đếm ký tự cập nhật tức thì; xem trước không tự cập nhật — nhấn [↻ Xem trước] để làm mới |
| Gõ vào Giá trị mẫu | xem trước KHÔNG tự cập nhật — nhấn [↻ Xem trước] để làm mới |
| `[↻ Xem trước]` | làm mới xem trước với toàn bộ giá trị mẫu hiện tại |
| `[+ Biến thể đối tượng]` (OR mode) | thêm Biến thể 2 bên dưới Biến thể 1 trong thẻ trigger đó |
| `[×]` trên Biến thể 2+ | hộp thoại xác nhận "Xóa biến thể này?" [Hủy] / [Xóa]. Xác nhận → xóa thẻ biến thể |
| Danh sách `[Đối tượng ▾]` trên biến thể | liệt kê phân khúc đã chọn ở S3; chọn để gán |
| **Section 6 — An toàn** | |
| Bỏ tick DNC | hộp thoại xác nhận: "Tắt DNC có thể vi phạm quy định. Chắc chắn?" [Hủy] / [Tắt]. Hủy → ô DNC giữ nguyên tích |
| Radio "Chọn từ danh sách thuê bao theo kênh" (Blacklist) | hiện tab kênh + danh sách thuê bao |
| Radio "Tải lên tệp" (Blacklist) | hiện danh sách chọn kênh + khu vực tải lên |
| Radio "Không dùng" (Blacklist hoặc Whitelist) | ẩn cấu hình; reach cập nhật lại |
| Radio "Chỉ gửi cho whitelist theo kênh" | hiện cấu hình WL tương tự BL |
| `[Đến Settings →]` | chuyển trang → /settings |

---

### Screen 4A — Template List

| Thao tác | Kết quả |
|---|---|
| `[+ Tạo Template]` | chuyển trang → /templates/new |
| `[Sửa]` | chuyển trang → /templates/:id |
| `[Nhân bản]` | thông báo "Đã tạo bản sao: Copy of [Tên]" + dòng mới xuất hiện đầu bảng |
| `[Tắt]` | hộp thoại xác nhận: "Tắt mẫu nội dung? Mẫu sẽ không hiện trong danh sách chọn khi tạo campaign." [Hủy] / [Tắt]. Xác nhận → dòng mờ đi, nút → [Bật] |
| `[Bật]` | không cần xác nhận → dòng trở lại bình thường, nút → [Tắt] |
| Nhấn số "X lần" cột Dùng | Cửa sổ nhỏ: "Campaign sử dụng: [tên campaign 1], [tên campaign 2]" |

### Screen 4B — Template Editor

| Thao tác | Kết quả |
|---|---|
| `← Template` | chuyển trang → /templates |
| Tab kênh | hiện thẻ soạn nội dung tab đó (giống S4 Campaign Builder) |
| `[+ Kênh]` | thêm tab mới |
| `[Lưu Template]` (tên trống) | lỗi trực tiếp "Tên template không được để trống" |
| `[Lưu Template]` (hợp lệ) | thông báo "Đã lưu template ✓" → navigate /templates |

---

### Screen 5 — Trigger Management

| Thao tác | Kết quả |
|---|---|
| `[+ Thêm Trigger]` | mở cửa sổ Thêm Trigger (các ô trống) |
| `[Sửa]` | mở cửa sổ Sửa Trigger (pre-filled) |
| `[Tắt]` trên trigger không dùng trong Active campaign | hộp thoại xác nhận: "Tắt trigger [CODE]?" [Hủy] / [Tắt]. Xác nhận → trạng thái Paused/Không hoạt động, nút → [Bật] |
| `[Tắt]` trên trigger đang dùng trong Active campaign | hộp thoại xác nhận thêm dòng cảnh báo: "Đang dùng trong: Welcome eSIM Q2/2026 (Active)" |
| `[Bật]` | không cần xác nhận → chuyển Active, nút → [Tắt] |
| Nhấn tiêu đề nhóm `▼ REALTIME` | thu gọn/mở rộng nhóm đó |
| **Cửa sổ Trigger** | |
| `[+ Thêm tham số]` | thêm dòng mới vào bảng tham số |
| `[✕]` trên dòng tham số | xóa dòng đó |
| `[Hủy]` | đóng cửa sổ |
| `[Lưu]` (ô còn trống) | lỗi trực tiếp trên các ô bắt buộc |
| `[Lưu]` (hợp lệ) | đóng cửa sổ + cập nhật bảng + thông báo "Đã lưu trigger ✓" |

---

### Screen 6 — Blacklist Management

| Thao tác | Kết quả |
|---|---|
| `[Xóa]` trên dòng | hộp thoại xác nhận: "Xóa [số] khỏi blacklist campaign [X] kênh [Y]?" [Hủy] / [Xóa]. Xác nhận → dòng bị xóa + thông báo "Đã xóa ✓" |
| `[+ Thêm thủ công]` | mở cửa sổ Thêm vào Blacklist |
| `[📥 Tải lên danh sách]` | mở cửa sổ Tải lên Blacklist |
| **Cửa sổ 6B — Thêm thủ công** | |
| `[Hủy]` | đóng cửa sổ |
| `[Thêm]` (ô còn trống) | lỗi trực tiếp |
| `[Thêm]` (hợp lệ) | đóng cửa sổ + thêm dòng vào bảng + thông báo "Đã thêm vào blacklist ✓" |
| **Cửa sổ 6C — Tải lên** | |
| `[Chọn file]` | mô phỏng chọn file → hiện kết quả xem trước: ✅ 245 số hợp lệ / ⚠ 3 số sai định dạng |
| `[📥 Tải file mẫu]` | thông báo "Đang tải file mẫu..." |
| `[Hủy]` | đóng cửa sổ |
| `[Xác nhận Tải lên]` | đóng cửa sổ + thông báo "Đã tải lên 245 số vào blacklist ✓" |

---

### Screen 7 — Customer List

| Thao tác | Kết quả |
|---|---|
| `[Xem 360]` | chuyển trang → /customers/:phone/360 với dữ liệu của dòng đó |
| Gõ vào ô tìm kiếm | lọc tức thì theo số điện thoại |
| Bộ lọc Trạng thái SIM / Cài app / DNC | lọc bảng theo điều kiện chọn |

---

### Screen 8 — Customer 360

| Thao tác | Kết quả |
|---|---|
| `← Danh sách khách hàng` | chuyển trang → /customers |
| `[Xem đầy đủ lịch sử →]` | mở khung trượt từ phải: danh sách 10 dòng lịch sử + lọc theo kênh + phân trang. `[✕]` đóng khung |

---

### Screen 9 — Report

| Thao tác | Kết quả |
|---|---|
| Tab [Delivery] / [Engagement] / [Campaign Comparison] / [Segment] / [Funnel] / [Spam] | hiện nội dung tab tương ứng |
| `[Xuất Excel]` | thông báo "Đang xuất file... Sẽ tải xuống sau vài giây" |
| `[So sánh: BẬT]` bật/tắt | BẬT → hiện thêm đường/cột kỳ trước trên tất cả biểu đồ; TẮT → ẩn |
| Danh sách Thời gian | lọc tất cả biểu đồ theo kỳ chọn |
| Danh sách Campaign | lọc theo campaign |
| Nhấn vào dòng trong bảng Campaign hoặc Phân khúc | làm nổi bật dòng đó; tất cả biểu đồ tự lọc theo lựa chọn |

---

### Screen 10 — Priority Matrix

| Thao tác | Kết quả |
|---|---|
| Nav "Settings" → "Priority Matrix" | chuyển trang → /settings/priority-matrix |
| `[ⓘ Help]` | hiện cửa sổ giải thích: "Số ưu tiên nhỏ hơn = được chọn trước khi nhiều campaign cùng khớp với một KH." Nút [Đóng] |
| Kéo icon `≡` của một dòng lên / xuống | sắp xếp lại vị trí dòng đó; toàn bộ ô Độ ưu tiên tự cập nhật lại theo vị trí mới (1, 2, 3...); ★ luôn theo dòng đầu tiên |
| Nhập số trực tiếp vào ô Độ ưu tiên | khi rời khỏi ô → bảng tự sắp xếp lại theo số vừa nhập; các ô còn lại cập nhật theo; nếu trùng số → tô đỏ cả hai dòng + tooltip "Trùng thứ tự ưu tiên — kéo để sắp xếp lại" |
| `[Lưu thứ tự]` (còn trùng số) | nút bị vô hiệu hóa khi có xung đột trùng số; di chuột → hiện "Hãy giải quyết xung đột trước khi lưu" |
| `[Lưu thứ tự]` (hợp lệ) | mở hộp thoại xác nhận: tiêu đề "Lưu thứ tự ưu tiên?", nội dung "Thứ tự mới sẽ áp dụng ngay cho sự kiện trigger tiếp theo.", nút [Hủy] (viền) / [Xác nhận] (màu chủ đạo). Confirm → thông báo "Đã cập nhật thứ tự ưu tiên ✓"; Hủy → đóng hộp thoại, giữ nguyên |
| Campaign Active mới được tạo | tự động xuất hiện cuối danh sách với Độ ưu tiên = max hiện tại + 1; không cần tải lại trang |
| Campaign bị Paused / Ended | tự ẩn khỏi danh sách; các dòng còn lại giữ nguyên số ưu tiên (không tự re-number) |

---

### Screen 3 — Kênh & Lịch gửi (cột phải Campaign Builder)

| Thao tác | Kết quả |
|---|---|
| Radio "Lịch riêng theo kênh" | hiện khối cấu hình từng kênh đang có trong S4; lấy giá trị từ lịch chung điền sẵn |
| Radio "Lịch chung" (đang ở lịch riêng) | hộp thoại xác nhận: "Chuyển về lịch chung sẽ ghi đè cấu hình lịch của tất cả kênh. Tiếp tục?" [Hủy] / [Xác nhận] |
| Nhấn tiêu đề khối kênh `▶ Push` | mở rộng khối đó; `▼` → thu gọn |
| Thay đổi thời gian gửi (radio) | cập nhật giá trị trong lịch chung hoặc lịch kênh tương ứng |
| Bật/Tắt Blackout trong lịch chung / lịch kênh | bật: hiện ô chọn giờ + danh sách xử lý; tắt: ẩn lại |

---

### Screen 3 — Section 3 Audience (Interaction Map cập nhật)

| Thao tác | Kết quả |
|---|---|
| `[🔍 Tìm phân khúc... ▾]` | mở danh sách chọn phân khúc; chọn → thêm thẻ phân khúc |
| `[×]` trên thẻ phân khúc | xóa phân khúc đó; lượng tiếp cận cập nhật lại |
| `[+ Thêm lọc]` trên thẻ phân khúc (chưa có điều kiện) | mở rộng tại chỗ bên dưới thẻ: hiện các dòng điều kiện lọc |
| `[Sửa]` trên thẻ phân khúc (đã có điều kiện) | mở rộng tại chỗ bên dưới thẻ: hiện các dòng điều kiện lọc với giá trị hiện tại |
| `[+ Thêm điều kiện]` trong mục lọc đang mở | thêm dòng điều kiện lọc mới |
| `[Áp dụng]` trong mục lọc đang mở | đóng lại; cập nhật thẻ điều kiện trong thẻ + lượng tiếp cận sau lọc |
| `[Hủy]` trong mục lọc đang mở | đóng lại; không lưu thay đổi |
| `[↻ Tính lại]` | hiện vòng xoay 500ms → làm mới lượng tiếp cận ước tính |

---

### Screen 3 — Section 4 Message Matrix (Interaction Map cập nhật)

| Thao tác | Kết quả |
|---|---|
| Nhấn tab kênh | hiện thẻ soạn nội dung của tab đó |
| `[+ Kênh]` | danh sách liệt kê kênh chưa có tab; chọn → thêm tab + tự tick kênh trong "Kênh & Lịch gửi" |
| Di chuột vào tham số | hiện tooltip: tên biến / mô tả / định dạng / ví dụ giá trị |
| Nhấn vào tham số | chèn `{{tên_param}}` vào vị trí con trỏ trong ô nhập đang được chọn |
| `[ℹ Hướng dẫn khai báo X]` → `[Mở]` | mở rộng hộp hướng dẫn kênh X; `[Đóng]` → thu lại |
| `[Tải lên]` trong ô tải ảnh | mô phỏng: hiện ảnh thu nhỏ + [Xóa][Đổi] |
| `[Chọn thư viện]` | mở cửa sổ thư viện ảnh: lưới ảnh mẫu, chọn → hiện ảnh thu nhỏ trong thẻ |
| `[Xóa]` trên ảnh thu nhỏ | xóa ảnh, trở về trạng thái trống |
| Gõ vào Tiêu đề / Nội dung | bộ đếm ký tự cập nhật tức thì |
| `[↻ Xem trước]` | làm mới xem trước với toàn bộ giá trị mẫu hiện tại |
| Gõ vào Giá trị mẫu | xem trước không tự cập nhật; cần nhấn `[↻ Xem trước]` |
| **Tab Biến thể đối tượng (Audience Variant)** | |
| Nhấn tab biến thể (ví dụ: "VIP") | hiện nội dung của biến thể đó trong 2 cột Soạn / XEM TRƯỚC |
| `[+ Biến thể đối tượng]` (OR mode) | thêm tab biến thể mới; bắt buộc chọn phân khúc trước khi lưu |
| `[×]` trên tab biến thể 2+ | hộp thoại xác nhận "Xóa biến thể [tên]?" [Hủy] / [Xóa] → xác nhận: xóa tab |
| Danh sách chọn phân khúc trên tab biến thể | liệt kê phân khúc đã chọn ở S3; tên tab cập nhật theo phân khúc |

---

### Screen 3 — Section 6 An toàn (Interaction Map cập nhật)

| Thao tác | Kết quả |
|---|---|
| Bỏ tick DNC | hộp thoại xác nhận: "Tắt DNC có thể vi phạm quy định. Chắc chắn?" [Hủy] / [Tắt]. Hủy → giữ nguyên tích |
| Radio "Chọn từ danh sách thuê bao theo kênh" (BL) | hiện tab kênh + danh sách thuê bao tìm kiếm được |
| Tab kênh trong BL/WL | chuyển sang kênh đó, hiện danh sách thuê bao tương ứng |
| `[+ Thêm kênh]` trong BL | thêm tab kênh mới vào BL |
| Tick / bỏ tick thuê bao trong danh sách | cập nhật "Đã chọn: X số"; đồng bộ sang Blacklist Management |
| Radio "Tải lên tệp" (BL) | hiện danh sách chọn kênh áp dụng + khu vực tải lên |
| `[Xem]` tệp đã tải lên | mở cửa sổ xem trước: danh sách số, Hợp lệ/Trùng/Sai định dạng + [Đóng] |
| Radio "Không dùng" (BL hoặc WL) | ẩn cấu hình BL/WL; reach cập nhật lại |
| Radio "Chỉ gửi cho whitelist theo kênh" | hiện cấu hình WL tương tự BL |
| `[Đến Settings →]` | chuyển trang → /settings |

---

### Screen Admin — /admin

| Thao tác | Kết quả |
|---|---|
| Tab `[Duyệt Campaign]` | hiện bảng campaign chờ duyệt (Pending) |
| Tab `[Trigger]` | hiện màn Trigger Management (giống Screen 5 — chỉ đọc, không có quyền chỉnh sửa) |
| `[Tìm campaign...]` | lọc tức thì theo tên / mã |
| `[Xem]` trên campaign Pending | chuyển trang → /campaigns/:id/detail (chế độ chỉ đọc) |
| `[Duyệt]` | hộp thoại xác nhận: "Duyệt campaign [tên]?" [Hủy] / [Duyệt]. Xác nhận → chuyển sang Active + thông báo "Đã duyệt ✓"; row biến mất khỏi bảng Pending |
| `[Từ chối]` | mở hộp thoại nhập lý do (ô văn bản bắt buộc): [Hủy] / [Từ chối]. Xác nhận → trạng thái Draft + lý do lưu lại + thông báo "Đã từ chối"; QTV nhận thông báo |

---

### Screen Settings — /settings

| Thao tác | Kết quả |
|---|---|
| Tab `[Frequency Cap]` | hiện trang cấu hình giới hạn tần suất |
| Tab `[Phân quyền]` | hiện bảng phân quyền Admin / QTV (chỉ đọc) |
| `[Lưu cài đặt]` (Frequency Cap — giá trị hợp lệ) | thông báo "Đã lưu cài đặt ✓" |
| `[Lưu cài đặt]` (giá trị không hợp lệ) | lỗi trực tiếp trên ô sai |
| Nav "Settings" → "Priority Matrix" | chuyển trang → /settings/priority-matrix |
