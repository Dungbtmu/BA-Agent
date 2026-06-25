# Wireframe CDP VNPost — v1

**Dự án:** CDP — Customer Data Platform (VNPost/TCT)
**Phiên bản:** v1
**Ngày:** 25/06/2026
**Trạng thái:** Dashboard (WF-001-01) đã BA confirm — các màn hình còn lại đang chờ review

---

## Danh sách màn hình

| WF-ID | Tên màn hình | Trạng thái |
|---|---|---|
| WF-001-01 | Dashboard — Tổng quan hệ thống | Đã confirm |
| WF-001-02 | Tìm kiếm & Danh sách khách hàng | Chờ review |
| WF-001-03 | Customer 360 — Hồ sơ cá nhân (Individual) | Chờ review |
| WF-001-04 | Customer 360 — Hồ sơ doanh nghiệp (Business/KHL) | Chờ review |
| WF-001-05 | Segment Builder | Chờ review |
| WF-001-06 | Danh sách Segment | Chờ review |

---

## Ghi chú chung về thiết kế

- Left sidebar navigation cố định: Tổng quan / Tìm kiếm KH / Phân khúc / Báo cáo / Cài đặt [Admin]
- Customer 360 layout 3 cột: trái (định danh) + giữa (tabs nội dung) + phải (điểm số & phân khúc)
- Masking theo phân quyền role: CCCD chỉ Admin xem; SĐT masked với CSKH cơ bản; COD Risk Score và Fraud Score ẩn với CSKH/Marketing
- Hồ sơ doanh nghiệp khác hồ sơ cá nhân: có tab Người liên hệ, Hợp đồng; không có CCCD

---

## WF-001-01: Dashboard — Tổng quan hệ thống

**Mục tiêu:** Cung cấp cái nhìn tổng quan về trạng thái hệ thống, số liệu KPIs và các cảnh báo quan trọng ngay khi đăng nhập.

**Trạng thái:** Đã BA confirm

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Tổng quan hệ thống                         📅 Hôm nay: 24/06/2026  ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ ║
║  ────    ║  │ Tổng KH      │ │ KH Mới       │ │ KH Doanh     │ │ Segment    │ ║
║          ║  │ (active)     │ │ (30 ngày)    │ │ Nghiệp (KHL) │ │ Active     │ ║
║ 🏠 Tổng  ║  │  2,847,312   │ │   +12,450    │ │    48,920    │ │    127     │ ║
║    quan  ║  │ ▲ 3.2% / T   │ │ ▲ vs T trước │ │ ▲ 1.8% / T  │ │  +5 mới   │ ║
║          ║  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ ║
║ 🔍 Tìm   ║                                                                      ║
║    kiếm  ║  ┌──────────────────────────────────┐ ┌──────────────────────────┐  ║
║    KH    ║  │ HOẠT ĐỘNG GẦN ĐÂY               │ │ CẢNH BÁO HỆ THỐNG       │  ║
║          ║  │                                  │ │                          │  ║
║ 👥 Phân  ║  │ 10:24 Segment "KH VIP Hà Nội"   │ │ ⚠ 234 KH COD risk > 80  │  ║
║    khúc  ║  │       được tạo bởi Marketing    │ │   → [Xem danh sách]      │  ║
║          ║  │ 09:51 Hồ sơ #C000192 cập nhật   │ │                          │  ║
║ 📊 Báo   ║  │       — CSKH Bắc Ninh           │ │ ⚠ 12 profile chờ xác    │  ║
║    cáo   ║  │ 09:30 Merge tự động: 3→1         │ │   minh danh tính         │  ║
║          ║  │       (PostID thống nhất)        │ │   → [Xem hàng đợi]       │  ║
║ ⚙️ Cài   ║  │ 08:15 Import CAS hoàn tất        │ │                          │  ║
║    đặt   ║  │       15,420 bản ghi mới         │ │ ✅ 8/9 nguồn bình thường │  ║
║  [Admin] ║  │                                  │ │ ⚠ PNS/DingDong: chậm    │  ║
║          ║  │ [Xem tất cả hoạt động]           │ │   → [Chi tiết]           │  ║
║          ║  └──────────────────────────────────┘ └──────────────────────────┘  ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| MetricCard — Tổng KH | Card số liệu | Tổng KH (active) | Hiển thị tổng KH đang hoạt động và xu hướng tháng | Default (có số) / Loading (skeleton) |
| MetricCard — KH Mới | Card số liệu | KH Mới (30 ngày) | Hiển thị số KH mới trong 30 ngày và so sánh với tháng trước | Default / Loading |
| MetricCard — KH DN | Card số liệu | KH Doanh Nghiệp (KHL) | Hiển thị tổng KH doanh nghiệp và xu hướng | Default / Loading |
| MetricCard — Segment | Card số liệu | Segment Active | Hiển thị số segment đang hoạt động và số mới tạo | Default / Loading |
| ActivityFeed | Danh sách hoạt động | Hoạt động gần đây | Ghi nhật ký thay đổi hệ thống theo thời gian thực | Default (có dữ liệu) / Empty (chưa có hoạt động) / Loading |
| SystemAlert | Khung cảnh báo | Cảnh báo hệ thống | Hiển thị cảnh báo COD risk, profile chờ xác minh, trạng thái nguồn dữ liệu | Default / Warning (có cảnh báo) / OK (không có cảnh báo) |
| AlertLink — Xem danh sách | Link hành động | Xem danh sách | Điều hướng đến danh sách KH COD risk cao | Default / Hover |
| AlertLink — Xem hàng đợi | Link hành động | Xem hàng đợi | Điều hướng đến danh sách profile chờ xác minh | Default / Hover |
| SidebarNav | Navigation | — | Điều hướng chính giữa các module | Default / Active (module hiện tại) / Hover |
| UserMenu | Dropdown | Xin chào [Tên] ▼ | Thông tin người dùng, đăng xuất | Default / Open |

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Click "Xem tất cả hoạt động" | Tải trang nhật ký hoạt động đầy đủ | Chuyển sang màn hình Activity Log | Toast lỗi nếu không tải được |
| Click "Xem danh sách" (COD risk) | Tải danh sách KH có COD Risk > 80 | Chuyển sang Tìm kiếm KH với bộ lọc COD risk đã áp sẵn | Toast lỗi |
| Click "Xem hàng đợi" (xác minh) | Tải danh sách profile chờ xác minh | Chuyển sang màn hình quản lý hàng đợi xác minh | Toast lỗi |
| Click "Chi tiết" (PNS/DingDong chậm) | Tải chi tiết trạng thái nguồn dữ liệu | Hiện modal hoặc trang chi tiết kết nối nguồn dữ liệu | Toast lỗi |
| Trang tải lần đầu | Gọi API lấy số liệu KPIs + hoạt động + cảnh báo | Hiển thị đầy đủ 4 metric cards + feed + cảnh báo | Hiển thị trạng thái lỗi từng card riêng biệt, không sập cả trang |

### Navigation

- **Entry point:** Sau đăng nhập thành công
- **Next:** Tìm kiếm KH, Phân khúc, Báo cáo, Cài đặt (qua sidebar)
- **Alternative paths:** Click vào cảnh báo → màn hình liên quan; Click metric card → màn hình chi tiết tương ứng

---

## WF-001-02: Tìm kiếm & Danh sách khách hàng

**Mục tiêu:** Cho phép người dùng tra cứu khách hàng theo nhiều tiêu chí và xem danh sách kết quả để điều hướng vào hồ sơ chi tiết.

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Tìm kiếm khách hàng                                                 ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║  ┌─────────────────────────────────────────────────────────────┐    ║
║  ────    ║  │ 🔍  Nhập SĐT / Họ tên / CCCD / Mã vận đơn / PostID / Mã KHL│    ║
║          ║  │                                                    [Tìm kiếm]│    ║
║ 🏠 Tổng  ║  └─────────────────────────────────────────────────────────────┘    ║
║    quan  ║                                                                      ║
║          ║  ▼ Lọc nâng cao                                                      ║
║ 🔍 Tìm   ║  ┌─────────────────────────────────────────────────────────────┐    ║
║    kiếm  ║  │ Loại KH: [ Tất cả ▼]  Tỉnh/TP: [ Tất cả ▼]                │    ║
║    KH    ║  │ Nhóm phân khúc: [ Tất cả ▼]   Trạng thái: [ Tất cả ▼]     │    ║
║  [active]║  │                              [Áp dụng lọc]  [Xóa bộ lọc]   │    ║
║          ║  └─────────────────────────────────────────────────────────────┘    ║
║ 👥 Phân  ║                                                                      ║
║    khúc  ║  Kết quả: 3 khách hàng khớp với "Nguyễn Văn A"    [Xuất Excel]     ║
║          ║                                                                      ║
║ 📊 Báo   ║  ┌─────┬──────────────┬───────────┬──────────┬──────────┬───────┐  ║
║    cáo   ║  │     │ Họ tên       │ SĐT       │ Loại KH  │ Tỉnh/TP  │       │  ║
║          ║  ├─────┼──────────────┼───────────┼──────────┼──────────┼───────┤  ║
║ ⚙️ Cài   ║  │ □   │ Nguyễn Văn A │ 090***123 │ Cá nhân  │ Hà Nội   │ [Xem] │  ║
║    đặt   ║  │ □   │ Nguyễn Văn A │ 091***456 │ Cá nhân  │ TP.HCM   │ [Xem] │  ║
║  [Admin] ║  │ □   │ Cty TNHH A   │ 024***789 │ Doanh    │ Đà Nẵng  │ [Xem] │  ║
║          ║  │     │              │           │ nghiệp   │          │       │  ║
║          ║  └─────┴──────────────┴───────────┴──────────┴──────────┴───────┘  ║
║          ║                                                                      ║
║          ║  ← Trang 1/1  |  Hiển thị 3/3 kết quả                              ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| SearchInput | Input text | Nhập SĐT / Họ tên / CCCD / Mã vận đơn / PostID / Mã KHL | Ô tìm kiếm đa tiêu chí, hỗ trợ 6 loại định danh | Default / Focus / Loading (đang tìm) / Error (tìm lỗi) |
| SearchButton | Button Primary | Tìm kiếm | Kích hoạt tìm kiếm | Default / Loading / Disabled (ô trống) |
| AdvancedFilterPanel | Collapsible Panel | Lọc nâng cao | Mở rộng/thu gọn bộ lọc bổ sung | Collapsed (mặc định) / Expanded |
| FilterLoaiKH | Dropdown | Loại KH | Lọc theo cá nhân / doanh nghiệp / tất cả | Default / Open |
| FilterTinhTP | Dropdown | Tỉnh/TP | Lọc theo tỉnh thành phố | Default / Open |
| FilterPhanKhuc | Dropdown | Nhóm phân khúc | Lọc theo segment đã tạo | Default / Open |
| FilterTrangThai | Dropdown | Trạng thái | Lọc KH đang hoạt động / không hoạt động | Default / Open |
| ApplyFilterButton | Button Secondary | Áp dụng lọc | Thực thi bộ lọc đã chọn | Default / Disabled (chưa chọn gì) |
| ClearFilterButton | Button Ghost | Xóa bộ lọc | Xóa toàn bộ bộ lọc về mặc định | Default / Disabled (không có filter) |
| ResultSummary | Text | Kết quả: N khách hàng | Hiển thị số kết quả và từ khóa tìm kiếm | Default / Empty |
| ExportButton | Button Secondary | Xuất Excel | Xuất danh sách kết quả ra file Excel | Default / Loading / Disabled (không có kết quả) |
| CustomerTable | Table | — | Hiển thị danh sách KH tìm được với các thông tin tóm tắt | Default / Loading (skeleton rows) / Empty |
| ViewButton | Button Ghost | Xem | Mở hồ sơ Customer 360 của KH được chọn | Default / Hover |
| Pagination | Phân trang | Trang N/M | Điều hướng giữa các trang kết quả | Default / Disabled (1 trang) |

### States màn hình

- **Default (chưa tìm):** Ô tìm kiếm trống, bảng kết quả chưa hiển thị, có placeholder gợi ý
- **Loading:** Sau khi nhấn Tìm kiếm — hiển thị skeleton rows trong bảng
- **Có kết quả:** Bảng hiện danh sách KH, thông báo số lượng kết quả
- **Không có kết quả:** Bảng hiện empty state "Không tìm thấy khách hàng khớp. Thử từ khóa khác hoặc điều chỉnh bộ lọc."
- **Lỗi:** Toast đỏ "Không thể thực hiện tìm kiếm. Vui lòng thử lại." + nút Thử lại

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Nhập từ khóa + Enter hoặc click Tìm kiếm | Gọi API tìm kiếm với từ khóa; SĐT masked nếu không phải Admin | Bảng hiện kết quả, summary hiện số lượng | Toast đỏ + nút Thử lại |
| Click "▼ Lọc nâng cao" | Mở rộng panel bộ lọc | Panel filter hiển thị | — |
| Chọn bộ lọc + click Áp dụng lọc | Gọi API tìm kiếm kết hợp từ khóa + bộ lọc | Bảng cập nhật kết quả theo bộ lọc | Toast đỏ |
| Click "Xóa bộ lọc" | Reset toàn bộ dropdown về "Tất cả" | Bộ lọc sạch; nếu đang có từ khóa thì tìm lại không có filter | — |
| Click [Xem] tại một hàng KH | Điều hướng sang Customer 360 | Mở trang hồ sơ KH tương ứng (cá nhân hoặc doanh nghiệp tùy loại KH) | Toast lỗi nếu hồ sơ không tải được |
| Click Xuất Excel | Tạo file Excel từ danh sách kết quả hiện tại | File tải xuống tự động | Toast lỗi "Không thể xuất file. Thử lại." |
| Click số trang trong Pagination | Tải trang kết quả tương ứng | Bảng cập nhật danh sách trang mới | Toast lỗi |

### Navigation

- **Previous:** Dashboard hoặc bất kỳ màn hình nào qua sidebar
- **Next:** WF-001-03 (Customer 360 cá nhân) hoặc WF-001-04 (Customer 360 doanh nghiệp) khi click [Xem]
- **Alternative paths:** Sidebar → Phân khúc, Báo cáo

---

## WF-001-03: Customer 360 — Hồ sơ cá nhân (Individual)

**Mục tiêu:** Cung cấp toàn bộ thông tin 360 độ về một khách hàng cá nhân để CSKH/Marketing/Admin tra cứu, xem lịch sử giao dịch, điểm số và phân khúc — hiển thị phân quyền theo role.

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Tìm kiếm KH  >  Customer 360  >  Nguyễn Văn An (#C000192)          ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║ ┌──────────────────┐ ┌──────────────────────────────┐ ┌───────────┐ ║
║  ────    ║ │ CỘT TRÁI         │ │ CỘT GIỮA                     │ │ CỘT PHẢI │ ║
║          ║ │ ĐỊNH DANH        │ │ CHI TIẾT (Tabs)              │ │ ĐIỂM SỐ  │ ║
║ 🏠 Tổng  ║ │                  │ │                              │ │ & PHÂN   │ ║
║    quan  ║ │ [Avatar viết tắt]│ │ [Giao dịch][COD][Timeline]   │ │ KHÚC     │ ║
║          ║ │ Nguyễn Văn An    │ │ [Khiếu nại][Loyalty][Consent]│ │          │ ║
║ 🔍 Tìm   ║ │ #C000192         │ │                              │ │ CLV Score│ ║
║    kiếm  ║ │ ● Đang hoạt động │ ├──────────────────────────────┤ │   742    │ ║
║    KH    ║ │                  │ │ [Tab: Giao dịch — đang chọn] │ │  ●●●●○   │ ║
║          ║ │ ── Liên hệ ───── │ │                              │ │  (Cao)   │ ║
║ 👥 Phân  ║ │ SĐT: 090***123   │ │  Tổng đơn:        47        │ │          │ ║
║    khúc  ║ │  [Hiện SĐT thật] │ │  Tổng giá trị:  12.4M đ    │ │ COD Risk │ ║
║          ║ │  [chỉ Admin/CSKH]│ │  Giao dịch cuối: 15/06/2026 │ │   [ẩn]   │ ║
║ 📊 Báo   ║ │ Email: nguy@...  │ │                              │ │ [Admin/  │ ║
║    cáo   ║ │                  │ │ ┌──────┬────────┬──────────┐  │ │ Rủi ro]  │ ║
║          ║ │ ── Định danh ─── │ │ │Mã VĐ │ Ngày   │ Trạng th.│  │ │          │ ║
║ ⚙️ Cài   ║ │ PostID: VNP00192 │ │ ├──────┼────────┼──────────┤  │ │ Fraud    │ ║
║    đặt   ║ │ CCCD: [*** ẩn ***│ │ │VE123.│ 10/06  │ Đã giao  │  │ │ Score    │ ║
║  [Admin] ║ │ chỉ Admin xem]   │ │ │VE124.│ 05/06  │ Đang chờ │  │ │   [ẩn]   │ ║
║          ║ │                  │ │ │VE125.│ 28/05  │ Đã giao  │  │ │ [Admin/  │ ║
║          ║ │ ── Phân loại ─── │ │ └──────┴────────┴──────────┘  │ │ Rủi ro]  │ ║
║          ║ │ Loại: Cá nhân    │ │  [Xem thêm 44 giao dịch]     │ │          │ ║
║          ║ │ Hạng loyalty: V. │ │                              │ │ Phân khúc│ ║
║          ║ │ Tỉnh/TP: Hà Nội  │ │                              │ │ KH VIP HN│ ║
║          ║ │                  │ │                              │ │ KH thân  │ ║
║          ║ │ [Lịch sử merge]  │ │                              │ │ thiết    │ ║
║          ║ └──────────────────┘ └──────────────────────────────┘ └───────────┘ ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Phân quyền hiển thị theo role

| Thông tin | CSKH cơ bản | CSKH đầy đủ / Marketing | Admin |
|---|---|---|---|
| Họ tên, địa chỉ, email | Hiện | Hiện | Hiện |
| SĐT (thật) | Masked (090***123) | Có thể hiện (click) | Hiện đầy đủ |
| CCCD | Ẩn hoàn toàn | Ẩn hoàn toàn | Hiện đầy đủ |
| COD Risk Score | Ẩn | Ẩn | Hiện |
| Fraud Score | Ẩn | Ẩn | Hiện |
| Giao dịch, Loyalty, Consent | Hiện | Hiện | Hiện |

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| CustomerAvatar | Avatar | [Chữ cái viết tắt họ tên] | Nhận diện nhanh khách hàng | Default |
| CustomerStatusBadge | Badge | Đang hoạt động / Ngưng hoạt động | Trạng thái hiện tại của KH | Active (xanh) / Inactive (xám) |
| PhoneDisplay | Text + Link | SĐT: 090***123 [Hiện] | Hiển thị SĐT với masking và nút hiện theo quyền | Masked / Revealed (nếu có quyền) / Hidden |
| CCCDDisplay | Text | CCCD: [*** ẩn ***] | Hiển thị CCCD — chỉ Admin mới thấy giá trị thật | Hidden (non-Admin) / Revealed (Admin) |
| TabNavigation | Tabs | Giao dịch / COD / Timeline / Khiếu nại / Loyalty / Consent | Chuyển nội dung chi tiết giữa các lĩnh vực | Default / Active (tab đang chọn) |
| TransactionSummary | Card số liệu | Tổng đơn / Tổng giá trị / Giao dịch cuối | Tóm tắt lịch sử giao dịch | Default / Loading |
| TransactionTable | Table | Mã VĐ / Ngày / Trạng thái | Danh sách giao dịch gần nhất | Default / Loading / Empty |
| ShowMoreButton | Button Ghost | Xem thêm N giao dịch | Tải thêm lịch sử giao dịch | Default / Loading |
| CLVScoreCard | Card điểm số | CLV Score + mức độ | Điểm giá trị vòng đời khách hàng | Default / Loading |
| CODRiskCard | Card điểm số | COD Risk Score | Điểm rủi ro COD — ẩn với CSKH/Marketing | Visible (Admin) / Hidden (non-Admin) |
| FraudScoreCard | Card điểm số | Fraud Score | Điểm gian lận — ẩn với CSKH/Marketing | Visible (Admin) / Hidden (non-Admin) |
| SegmentBadgeList | Danh sách badge | [Tên segment] | Các phân khúc khách hàng đang thuộc về | Default / Empty (chưa có segment) |
| MergeHistoryLink | Link Ghost | Lịch sử merge | Xem lịch sử hợp nhất profile (Identity Graph) | Default / Hover |
| Breadcrumb | Navigation | Tìm kiếm KH > Customer 360 > [Tên KH] | Định hướng và quay lại | Default |

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Click tab (Giao dịch / COD / Timeline / Khiếu nại / Loyalty / Consent) | Tải nội dung tab tương ứng | Tab được highlight, nội dung thay đổi | Hiển thị lỗi inline trong tab "Không tải được dữ liệu. [Thử lại]" |
| Click "Hiện" bên cạnh SĐT | Kiểm tra quyền; nếu đủ quyền → gọi API lấy SĐT thật | SĐT thật hiển thị, nút đổi thành "Ẩn" | Toast lỗi "Không thể hiển thị. Thử lại." |
| Click "Xem thêm N giao dịch" | Tải thêm trang giao dịch | Bảng giao dịch mở rộng thêm các hàng mới | Toast lỗi + nút Thử lại |
| Click "Lịch sử merge" | Mở modal hoặc trang lịch sử hợp nhất profile | Hiện danh sách các lần merge và nguồn dữ liệu | Toast lỗi |
| Click breadcrumb "Tìm kiếm KH" | Quay lại màn hình tìm kiếm | Mở lại trang tìm kiếm (giữ nguyên từ khóa đã tìm) | — |
| Trang tải lần đầu | Gọi song song API lấy định danh, điểm số, tab mặc định (Giao dịch) | Toàn bộ 3 cột hiển thị với dữ liệu | Từng phần lỗi riêng biệt — không sập cả trang |

### Navigation

- **Previous:** WF-001-02 — Tìm kiếm khách hàng
- **Next:** Không có màn hình tiếp theo cố định — KH ở lại trang hoặc quay lại tìm kiếm
- **Alternative paths:** Click tab → nội dung trong cùng màn hình thay đổi; Click "Lịch sử merge" → modal/trang merge history; Sidebar → module khác

---

## WF-001-04: Customer 360 — Hồ sơ doanh nghiệp (Business/KHL)

**Mục tiêu:** Hiển thị hồ sơ 360 độ của một khách hàng doanh nghiệp (KHL) với các thông tin đặc thù như người liên hệ, hợp đồng, và chỉ số giao dịch quy mô lớn.

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Tìm kiếm KH  >  Customer 360  >  Cty TNHH ABC (#KHL00451)          ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║ ┌──────────────────┐ ┌──────────────────────────────┐ ┌───────────┐ ║
║  ────    ║ │ CỘT TRÁI         │ │ CỘT GIỮA                     │ │ CỘT PHẢI │ ║
║          ║ │ ĐỊNH DANH DN     │ │ CHI TIẾT (Tabs)              │ │ ĐIỂM SỐ  │ ║
║ 🏠 Tổng  ║ │                  │ │                              │ │ & PHÂN   │ ║
║    quan  ║ │ [Logo/Icon DN]   │ │ [Giao dịch][COD][Timeline]   │ │ KHÚC     │ ║
║          ║ │ Cty TNHH ABC     │ │ [Khiếu nại][Hợp đồng]       │ │          │ ║
║ 🔍 Tìm   ║ │ #KHL00451        │ │ [Người liên hệ]              │ │ CLV Score│ ║
║    kiếm  ║ │ ● Đang hoạt động │ │                              │ │  1,248   │ ║
║    KH    ║ │                  │ ├──────────────────────────────┤ │  ●●●●●   │ ║
║          ║ │ ── Thông tin DN──│ │ [Tab: Người liên hệ — active]│ │ (Rất cao)│ ║
║ 👥 Phân  ║ │ Mã KHL: KHL00451 │ │                              │ │          │ ║
║    khúc  ║ │ MST: 0123456789  │ │ ┌──────────┬────────┬──────┐  │ │ Hạng KHL │ ║
║          ║ │ Ngành: TMĐT      │ │ │ Họ tên   │ Chức vụ│ SĐT  │  │ │  Vàng    │ ║
║ 📊 Báo   ║ │ Quy mô: Vừa     │ │ ├──────────┼────────┼──────┤  │ │          │ ║
║    cáo   ║ │                  │ │ │ Trần Văn B│ GĐ    │●●●●  │  │ │ Phân khúc│ ║
║          ║ │ ── Địa chỉ ───── │ │ │ Lê Thị C  │ KT    │●●●●  │  │ │ DN VIP   │ ║
║ ⚙️ Cài   ║ │ 45 Lý Thường    │ │ │ Nguyễn D  │ Sale  │●●●●  │  │ │ TP.HCM   │ ║
║    đặt   ║ │ Kiệt, Đà Nẵng   │ │ └──────────┴────────┴──────┘  │ │          │ ║
║  [Admin] ║ │                  │ │                              │ │ COD Risk │ ║
║          ║ │ ── Hợp đồng ──── │ │  [+ Thêm người liên hệ]     │ │   [ẩn]   │ ║
║          ║ │ HĐ hiện hành: 2  │ │                              │ │ [Admin]  │ ║
║          ║ │ Sắp hết hạn:     │ │                              │ │          │ ║
║          ║ │ HĐ-2026-09-30    │ │                              │ │          │ ║
║          ║ │ hết 30/09/2026   │ │                              │ │          │ ║
║          ║ └──────────────────┘ └──────────────────────────────┘ └───────────┘ ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Khác biệt so với hồ sơ cá nhân

| Điểm khác biệt | Hồ sơ cá nhân | Hồ sơ doanh nghiệp |
|---|---|---|
| Trường định danh | CCCD (ẩn theo quyền), PostID | MST, Mã KHL, Ngành, Quy mô (không có CCCD) |
| Tabs chi tiết | Giao dịch / COD / Timeline / Khiếu nại / Loyalty / Consent | Giao dịch / COD / Timeline / Khiếu nại / Hợp đồng / Người liên hệ |
| Avatar | Viết tắt họ tên cá nhân | Icon tòa nhà / Logo DN |
| Cột phải | CLV Score / COD Risk / Fraud / Phân khúc | CLV Score / Hạng KHL / COD Risk (Admin) / Phân khúc |
| Thông tin tóm tắt trái | Loại, Hạng loyalty, Tỉnh/TP | Tên DN, MST, Ngành, Quy mô, Địa chỉ, Thống kê hợp đồng |

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| BusinessAvatar | Avatar/Icon | [Icon tòa nhà hoặc logo DN] | Nhận diện doanh nghiệp | Default |
| CustomerStatusBadge | Badge | Đang hoạt động / Ngưng hoạt động | Trạng thái hiện tại của DN | Active (xanh) / Inactive (xám) |
| BusinessInfoBlock | Info block | Mã KHL / MST / Ngành / Quy mô | Thông tin pháp lý và phân loại doanh nghiệp | Default |
| ContractSummary | Info block | HĐ hiện hành / Sắp hết hạn | Tóm tắt tình trạng hợp đồng — cảnh báo khi gần hết hạn | Default / Warning (gần hết hạn < 30 ngày) |
| TabNavigation | Tabs | Giao dịch / COD / Timeline / Khiếu nại / Hợp đồng / Người liên hệ | Chuyển nội dung chi tiết | Default / Active |
| ContactTable | Table | Họ tên / Chức vụ / SĐT | Danh sách người liên hệ của DN với SĐT masked | Default / Loading / Empty |
| AddContactButton | Button Secondary | + Thêm người liên hệ | Mở form thêm người liên hệ mới | Default / Disabled (nếu không có quyền) |
| KHLRankCard | Card | Hạng KHL | Hạng doanh nghiệp trong chương trình KHL (Đồng/Bạc/Vàng/Kim cương) | Default / Loading |
| CLVScoreCard | Card điểm số | CLV Score | Điểm giá trị vòng đời doanh nghiệp | Default / Loading |
| CODRiskCard | Card điểm số | COD Risk Score | Ẩn với CSKH/Marketing, hiện với Admin | Visible (Admin) / Hidden |
| SegmentBadgeList | Danh sách badge | [Tên segment] | Các phân khúc DN đang thuộc về | Default / Empty |
| Breadcrumb | Navigation | Tìm kiếm KH > Customer 360 > [Tên DN] | Định hướng và quay lại | Default |

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Click tab Người liên hệ | Tải danh sách người liên hệ của DN | Bảng hiện danh sách với SĐT masked | Lỗi inline trong tab "Không tải được. [Thử lại]" |
| Click tab Hợp đồng | Tải danh sách hợp đồng của DN | Bảng hiện danh sách hợp đồng, badge màu vàng cho HĐ gần hết hạn | Lỗi inline trong tab |
| Click "+ Thêm người liên hệ" | Mở modal form thêm người liên hệ | Modal hiện với các trường: Họ tên, Chức vụ, SĐT, Email | Toast lỗi nếu modal không mở được |
| Submit form thêm người liên hệ | Validate dữ liệu, lưu người liên hệ mới | Modal đóng, danh sách cập nhật, toast xanh "Đã thêm người liên hệ" | Lỗi inline tại field sai định dạng |
| Click tab Giao dịch / COD / Timeline / Khiếu nại | Tương tự hồ sơ cá nhân | Nội dung tab thay đổi | Lỗi inline trong tab |
| Trang tải lần đầu | Gọi song song API định danh DN, điểm số, tab mặc định | Toàn bộ 3 cột hiển thị | Từng phần lỗi riêng biệt |

### Navigation

- **Previous:** WF-001-02 — Tìm kiếm khách hàng
- **Next:** Không có màn hình tiếp theo cố định
- **Alternative paths:** Tab Hợp đồng → chi tiết hợp đồng; Tab Người liên hệ → form thêm mới; Sidebar → module khác

---

## WF-001-05: Segment Builder

**Mục tiêu:** Cho phép người dùng (Marketing/Admin) tạo phân khúc khách hàng mới bằng giao diện visual rule, không cần SQL, với ước tính số lượng khách hàng theo thời gian thực.

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Phân khúc  >  Tạo phân khúc mới                                    ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║  Tên phân khúc: *                                                    ║
║  ────    ║  ┌────────────────────────────────────────────────────────────────┐  ║
║          ║  │ _KH VIP Hà Nội 2026___________________________________________  │  ║
║ 🏠 Tổng  ║  └────────────────────────────────────────────────────────────────┘  ║
║    quan  ║                                                                      ║
║          ║  Mô tả (không bắt buộc):                                            ║
║ 🔍 Tìm   ║  ┌────────────────────────────────────────────────────────────────┐  ║
║    kiếm  ║  │ _KH cá nhân tại HN, CLV > 700, active 3 tháng gần nhất_____   │  ║
║    KH    ║  └────────────────────────────────────────────────────────────────┘  ║
║          ║                                                                      ║
║ 👥 Phân  ║  ── Điều kiện lọc ────────────────────────────────────────────────── ║
║    khúc  ║                                                                      ║
║  [active]║  Khớp với: ( ● Tất cả điều kiện [AND]  ○ Bất kỳ điều kiện [OR] )   ║
║          ║                                                                      ║
║ 📊 Báo   ║  ┌──── Nhóm điều kiện 1 ─────────────────────────────────────────┐  ║
║    cáo   ║  │                                                    [+ Thêm đk] │  ║
║          ║  │  [Thuộc tính ▼]       [Toán tử ▼]     [Giá trị           ][×] │  ║
║ ⚙️ Cài   ║  │  Tỉnh/TP cư trú       =               Hà Nội                   │  ║
║    đặt   ║  │                                                                │  ║
║  [Admin] ║  │  [Thuộc tính ▼]       [Toán tử ▼]     [Giá trị           ][×] │  ║
║          ║  │  Loại khách hàng      =               Cá nhân                  │  ║
║          ║  │                                                                │  ║
║          ║  │  [Thuộc tính ▼]       [Toán tử ▼]     [Giá trị           ][×] │  ║
║          ║  │  CLV Score            ≥               700                      │  ║
║          ║  └────────────────────────────────────────────────────────────────┘  ║
║          ║                                                                      ║
║          ║  [+ Thêm nhóm điều kiện]                                            ║
║          ║                                                                      ║
║          ║  ┌────────────────────────────────────────────────────────────────┐  ║
║          ║  │  Ước tính: ~14,320 khách hàng         ↻ Cập nhật ước tính     │  ║
║          ║  └────────────────────────────────────────────────────────────────┘  ║
║          ║                                                                      ║
║          ║                                     [Hủy]  [Lưu nháp]  [Lưu & Kích ║
║          ║                                                          hoạt]       ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| SegmentNameInput | Input text | Tên phân khúc * | Nhập tên phân khúc, bắt buộc, unique | Default / Focus / Error (trùng tên) / Error (để trống) |
| SegmentDescInput | Textarea | Mô tả | Mô tả tùy chọn để làm rõ mục đích phân khúc | Default / Focus |
| LogicSelector | Radio group | Tất cả điều kiện [AND] / Bất kỳ điều kiện [OR] | Chọn logic kết hợp giữa các điều kiện | AND selected (mặc định) / OR selected |
| ConditionGroup | Container | Nhóm điều kiện N | Nhóm chứa các hàng điều kiện; có thể có nhiều nhóm | Default / Expanded |
| AttributeDropdown | Dropdown | [Thuộc tính ▼] | Chọn thuộc tính KH từ danh sách 13 Block (A-M) | Default / Open / Loading |
| OperatorDropdown | Dropdown | [Toán tử ▼] | Chọn phép so sánh phù hợp với kiểu dữ liệu thuộc tính (=, ≥, ≤, chứa, trong danh sách...) | Default / Open / Disabled (chưa chọn thuộc tính) |
| ValueInput | Input / Dropdown / DatePicker | [Giá trị] | Nhập hoặc chọn giá trị so sánh, kiểu input thay đổi theo thuộc tính | Default / Focus / Error (giá trị không hợp lệ) |
| RemoveConditionButton | Button Icon | × | Xóa hàng điều kiện khỏi nhóm | Default / Hover / Disabled (nhóm chỉ còn 1 điều kiện) |
| AddConditionButton | Button Ghost | + Thêm điều kiện | Thêm hàng điều kiện mới vào nhóm | Default / Hover |
| AddGroupButton | Button Ghost | + Thêm nhóm điều kiện | Thêm nhóm điều kiện mới (logic khác biệt) | Default / Hover |
| EstimatePanel | Info panel | Ước tính: ~N khách hàng | Hiển thị ước tính real-time số KH khớp điều kiện | Default / Loading (đang tính) / Error |
| RefreshEstimateButton | Button Ghost | ↻ Cập nhật ước tính | Gọi lại API ước tính thủ công | Default / Loading |
| CancelButton | Button Secondary | Hủy | Hủy tạo phân khúc, quay lại danh sách | Default |
| SaveDraftButton | Button Secondary | Lưu nháp | Lưu phân khúc ở trạng thái Nháp, chưa kích hoạt | Default / Loading |
| SaveActivateButton | Button Primary | Lưu & Kích hoạt | Lưu và kích hoạt phân khúc ngay | Default / Loading / Disabled (form chưa hợp lệ) |

### States màn hình

- **Default (mới tạo):** Form trống, ô Thuộc tính/Toán tử/Giá trị chưa có giá trị, ước tính hiện 0
- **Đang điền điều kiện:** Khi người dùng chọn thuộc tính, Toán tử auto-cập nhật theo kiểu dữ liệu; Giá trị auto-suggest nếu là danh sách
- **Ước tính loading:** Khi điều kiện thay đổi và hệ thống đang tính lại
- **Form lỗi:** Highlight field thiếu/sai khi nhấn Lưu — inline error message
- **Trùng tên:** Inline error "Tên phân khúc đã tồn tại. Vui lòng chọn tên khác."

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Chọn Thuộc tính | Tải danh sách toán tử phù hợp với kiểu dữ liệu thuộc tính | Dropdown Toán tử cập nhật; ô Giá trị đổi kiểu phù hợp | Toast lỗi nếu không tải được danh sách thuộc tính |
| Thay đổi bất kỳ điều kiện | Sau 1 giây không có thao tác, tự động gọi API ước tính | Ô Ước tính cập nhật con số mới | Ô Ước tính hiện "Không thể tính. [↻ Thử lại]" |
| Click "↻ Cập nhật ước tính" | Gọi API ước tính thủ công với điều kiện hiện tại | Con số ước tính cập nhật | Toast lỗi |
| Click "+ Thêm điều kiện" | Thêm hàng mới vào nhóm điều kiện | Hàng mới xuất hiện với 3 dropdown trống | — |
| Click "×" xóa điều kiện | Xóa hàng điều kiện | Hàng biến mất, ước tính tự cập nhật | — |
| Click "+ Thêm nhóm điều kiện" | Thêm nhóm mới bên dưới | Nhóm điều kiện 2 xuất hiện với 1 hàng trống | — |
| Click "Hủy" | Kiểm tra có dữ liệu chưa lưu không | Nếu form trống: quay lại danh sách. Nếu có dữ liệu: confirm dialog "Bỏ phân khúc này?" | — |
| Click "Lưu nháp" | Validate tên (bắt buộc); lưu với trạng thái Nháp | Toast xanh "Đã lưu nháp", chuyển sang trang danh sách | Toast lỗi nếu tên trùng hoặc lỗi server |
| Click "Lưu & Kích hoạt" | Validate toàn bộ form; lưu và kích hoạt phân khúc | Toast xanh "Phân khúc đã được kích hoạt", chuyển sang danh sách | Inline error tại field lỗi; toast lỗi server |

### Navigation

- **Previous:** WF-001-06 — Danh sách Segment (qua nút "+ Tạo phân khúc mới")
- **Next:** WF-001-06 — Danh sách Segment sau khi lưu thành công
- **Alternative paths:** Click "Hủy" → WF-001-06; Sidebar → module khác

---

## WF-001-06: Danh sách Segment

**Mục tiêu:** Hiển thị toàn bộ phân khúc đã tạo, cho phép tìm kiếm, lọc và thực hiện các thao tác quản lý phân khúc (xem, sửa, nhân bản, tạm dừng, xóa).

### Layout

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║  [Logo VNPost CDP]   CDP — Customer Data Platform           [?] Trợ giúp        ║
║                                                   Xin chào, Nguyễn Thị Lan ▼   ║
╠══════════╦═════════════════════════════════════════════════════════════════════╣
║          ║  Danh sách phân khúc                   [+ Tạo phân khúc mới]        ║
║  ĐIỀU    ║                                                                      ║
║  HƯỚNG   ║  🔍 [Tìm theo tên phân khúc...              ]  Trạng thái: [Tất cả▼]║
║  ────    ║                                                                      ║
║          ║  ┌─────┬──────────────────────┬───────────┬──────────┬──────────┬──┐ ║
║ 🏠 Tổng  ║  │     │ Tên phân khúc        │ Số KH     │ Cập nhật │ Trạng    │  │ ║
║    quan  ║  │     │                      │           │ lần cuối │ thái     │  │ ║
║          ║  ├─────┼──────────────────────┼───────────┼──────────┼──────────┼──┤ ║
║ 🔍 Tìm   ║  │ □   │ KH VIP Hà Nội 2026  │ 14,320    │ 24/06    │ ● Active │[·]│ ║
║    kiếm  ║  │ □   │ KH Doanh nghiệp     │ 48,920    │ 24/06    │ ● Active │[·]│ ║
║    KH    ║  │     │ toàn quốc           │           │          │          │  │ ║
║          ║  │ □   │ KH COD rủi ro cao   │ 2,341     │ 23/06    │ ● Active │[·]│ ║
║ 👥 Phân  ║  │ □   │ KH không giao dịch  │ 8,720     │ 20/06    │ ⏸ Tạm    │[·]│ ║
║    khúc  ║  │     │ 6 tháng gần nhất    │           │          │ dừng     │  │ ║
║  [active]║  │ □   │ Segment thử nghiệm  │ 134       │ 10/06    │ 📝 Nháp  │[·]│ ║
║          ║  └─────┴──────────────────────┴───────────┴──────────┴──────────┴──┘ ║
║ 📊 Báo   ║                                                                      ║
║    cáo   ║  ← Trang 1/26  |  Hiển thị 5/127 phân khúc       [25 mục/trang ▼]  ║
║          ║                                                                      ║
║ ⚙️ Cài   ║  Khi click [·] — menu hành động xuất hiện:                           ║
║    đặt   ║  ┌──────────────────┐                                                ║
║  [Admin] ║  │ Xem chi tiết     │                                                ║
║          ║  │ Chỉnh sửa        │                                                ║
║          ║  │ Nhân bản         │                                                ║
║          ║  │ Tạm dừng / Kích  │                                                ║
║          ║  │ hoạt lại         │                                                ║
║          ║  │ ─────────────    │                                                ║
║          ║  │ Xóa (màu đỏ)    │                                                ║
║          ║  └──────────────────┘                                                ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

### Components

| Component | Type | Label | Mục đích | States |
|---|---|---|---|---|
| PageTitle | Heading | Danh sách phân khúc | Tiêu đề trang | Default |
| CreateButton | Button Primary | + Tạo phân khúc mới | Điều hướng sang WF-001-05 Segment Builder | Default / Hover / Disabled (nếu không có quyền) |
| SearchInput | Input text | Tìm theo tên phân khúc... | Lọc danh sách theo tên phân khúc | Default / Focus / Loading |
| StatusFilter | Dropdown | Trạng thái: Tất cả ▼ | Lọc theo trạng thái Active / Tạm dừng / Nháp | Default / Open |
| SegmentTable | Table | — | Danh sách phân khúc với thông tin tóm tắt | Default / Loading (skeleton) / Empty |
| CheckboxColumn | Checkbox | □ | Chọn nhiều phân khúc để thao tác hàng loạt | Unchecked / Checked / Indeterminate (một số được chọn) |
| SegmentNameCell | Text + Link | [Tên phân khúc] | Tên phân khúc, click để xem chi tiết | Default / Hover |
| CustomerCountCell | Text | [Số KH] | Số lượng khách hàng trong phân khúc | Default |
| LastUpdatedCell | Text | [Ngày cập nhật] | Ngày phân khúc được tính toán lại lần cuối | Default |
| StatusBadge | Badge | Active / Tạm dừng / Nháp | Trạng thái phân khúc với màu sắc nhất quán | Active (xanh lá) / Tạm dừng (vàng) / Nháp (xám) |
| ActionMenu | Button Icon + Dropdown | [···] | Menu hành động: Xem / Sửa / Nhân bản / Tạm dừng hoặc Kích hoạt lại / Xóa | Default / Open |
| DeleteOption | Menu item Danger | Xóa | Xóa phân khúc (có xác nhận) | Default / Hover (màu đỏ) |
| Pagination | Phân trang | Trang N/M | Điều hướng giữa các trang | Default / Disabled (1 trang) |
| PageSizeSelector | Dropdown | 25 mục/trang ▼ | Thay đổi số dòng hiển thị mỗi trang (10/25/50) | Default / Open |

### States màn hình

- **Default (có dữ liệu):** Bảng hiện danh sách phân khúc với đầy đủ thông tin
- **Loading:** Skeleton rows trong bảng khi tải trang hoặc thay đổi bộ lọc
- **Empty (chưa có phân khúc):** Empty state "Chưa có phân khúc nào. [+ Tạo phân khúc đầu tiên]"
- **Empty (không có kết quả lọc):** "Không tìm thấy phân khúc phù hợp. Thử từ khóa hoặc bộ lọc khác."
- **Lỗi tải:** "Không thể tải danh sách phân khúc. [Thử lại]"

### User Interactions

| Trigger | Process | Success Outcome | Error Handling |
|---|---|---|---|
| Click "+ Tạo phân khúc mới" | Điều hướng sang WF-001-05 | Mở màn hình Segment Builder | — |
| Nhập từ khóa vào ô tìm kiếm | Lọc real-time sau 300ms dừng gõ | Bảng cập nhật, chỉ hiện phân khúc tên chứa từ khóa | Toast lỗi nếu API lọc thất bại |
| Chọn Trạng thái trong dropdown | Lọc danh sách theo trạng thái | Bảng cập nhật chỉ hiện phân khúc có trạng thái được chọn | Toast lỗi |
| Click tên phân khúc | Điều hướng sang trang chi tiết phân khúc | Mở trang chi tiết phân khúc (ngoài scope wireframe v1) | Toast lỗi |
| Click [···] → Chỉnh sửa | Mở Segment Builder ở chế độ edit, tải lại điều kiện của phân khúc | Mở WF-001-05 với dữ liệu phân khúc đã điền sẵn | Toast lỗi |
| Click [···] → Nhân bản | Tạo bản sao phân khúc với tên "[Tên gốc] — bản sao" | Bản sao xuất hiện trong danh sách, trạng thái Nháp | Toast lỗi |
| Click [···] → Tạm dừng | Đổi trạng thái phân khúc sang Tạm dừng | Badge đổi thành vàng "Tạm dừng", toast xanh xác nhận | Toast lỗi |
| Click [···] → Kích hoạt lại | Đổi trạng thái phân khúc sang Active | Badge đổi thành xanh "Active", toast xanh xác nhận | Toast lỗi |
| Click [···] → Xóa | Hiện confirm dialog "Xóa phân khúc này? Hành động không thể hoàn tác." với nút Xóa (đỏ) và Hủy | Phân khúc bị xóa khỏi danh sách, toast xanh "Đã xóa phân khúc" | Toast lỗi nếu xóa thất bại |
| Click số trang hoặc thay đổi số mục/trang | Tải trang dữ liệu tương ứng | Bảng cập nhật | Toast lỗi |

### Navigation

- **Previous:** Dashboard hoặc bất kỳ màn hình nào qua sidebar "Phân khúc"
- **Next:** WF-001-05 (click "+ Tạo phân khúc mới" hoặc "Chỉnh sửa")
- **Alternative paths:** Click tên phân khúc → trang chi tiết; Sidebar → module khác

---

## User Flow tổng

```
[Dashboard]
    |
    ├─ Sidebar "Tìm kiếm KH" → [Tìm kiếm & Danh sách KH]
    │                               |
    │                               ├─ Click [Xem] KH cá nhân → [Customer 360 — Cá nhân]
    │                               │                               |
    │                               │                               └─ Tab: Giao dịch / COD /
    │                               │                                   Timeline / Khiếu nại /
    │                               │                                   Loyalty / Consent
    │                               │
    │                               └─ Click [Xem] KH doanh nghiệp → [Customer 360 — DN]
    │                                                               |
    │                                                               └─ Tab: Giao dịch / COD /
    │                                                                   Timeline / Khiếu nại /
    │                                                                   Hợp đồng / Người liên hệ
    |
    └─ Sidebar "Phân khúc" → [Danh sách Segment]
                                |
                                ├─ Click "+ Tạo phân khúc mới" → [Segment Builder]
                                │                               |
                                │                               ├─ Lưu & Kích hoạt → [Danh sách Segment]
                                │                               ├─ Lưu nháp → [Danh sách Segment]
                                │                               └─ Hủy → [Danh sách Segment]
                                │
                                └─ Click [···] Chỉnh sửa → [Segment Builder] (edit mode)

Cảnh báo Dashboard:
    ├─ "Xem danh sách" (COD risk) → [Tìm kiếm KH] với bộ lọc áp sẵn
    └─ "Xem hàng đợi" (xác minh) → [Danh sách chờ xác minh] (ngoài scope v1)
```

---

## Acceptance Criteria Coverage

| Nhóm chức năng | Màn hình phủ |
|---|---|
| Tổng quan số liệu KPIs hệ thống (tổng KH, KH mới, KH DN, Segment) | WF-001-01 |
| Cảnh báo COD risk cao, profile chờ xác minh, nguồn dữ liệu | WF-001-01 |
| Hoạt động gần đây (audit log tóm tắt) | WF-001-01 |
| Tìm kiếm KH theo SĐT, tên, CCCD, mã vận đơn, PostID, mã KHL | WF-001-02 |
| Lọc nâng cao theo loại KH, tỉnh/TP, phân khúc, trạng thái | WF-001-02 |
| Xuất danh sách kết quả ra Excel | WF-001-02 |
| Masking SĐT theo role (hiện thật với Admin/CSKH đầy đủ, masked với CSKH cơ bản) | WF-001-02, WF-001-03 |
| Hồ sơ 360 khách hàng cá nhân: định danh, liên hệ, PostID, CCCD (ẩn theo quyền) | WF-001-03 |
| Lịch sử giao dịch cá nhân (tab Giao dịch) | WF-001-03 |
| Điểm CLV, COD Risk (ẩn), Fraud Score (ẩn với CSKH/Marketing) | WF-001-03 |
| Phân khúc khách hàng đang thuộc về | WF-001-03, WF-001-04 |
| Tabs: COD / Timeline / Khiếu nại / Loyalty / Consent | WF-001-03 |
| Hồ sơ 360 khách hàng doanh nghiệp: tên DN, MST, Mã KHL, ngành, quy mô | WF-001-04 |
| Tóm tắt hợp đồng, cảnh báo gần hết hạn | WF-001-04 |
| Tab Người liên hệ của doanh nghiệp (không có ở hồ sơ cá nhân) | WF-001-04 |
| Tab Hợp đồng của doanh nghiệp (không có ở hồ sơ cá nhân) | WF-001-04 |
| Hạng KHL trong phần điểm số | WF-001-04 |
| Tạo phân khúc bằng visual rule (không SQL), chọn thuộc tính + toán tử + giá trị | WF-001-05 |
| Logic AND/OR giữa các điều kiện | WF-001-05 |
| Ước tính số KH real-time khi thay đổi điều kiện | WF-001-05 |
| Lưu nháp và Lưu & Kích hoạt phân khúc | WF-001-05 |
| Danh sách phân khúc với trạng thái Active / Tạm dừng / Nháp | WF-001-06 |
| Tìm kiếm và lọc phân khúc theo tên và trạng thái | WF-001-06 |
| Thao tác hàng loạt: Xem / Sửa / Nhân bản / Tạm dừng / Kích hoạt lại / Xóa | WF-001-06 |
| Xác nhận trước khi xóa phân khúc (destructive action) | WF-001-06 |
