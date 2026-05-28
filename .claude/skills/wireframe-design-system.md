---
name: wireframe_design_system
description: Chuẩn thiết kế chung cho wireframe và React prototype — layout, component patterns, naming, states bắt buộc. Dự án có thể override bằng file design-guideline.md riêng.
tools: []
---

# Skill: Wireframe Design System

## Mục tiêu

Đảm bảo wireframe và React prototype nhất quán về layout, component, naming và states — bất kể dự án nào. Chuẩn này áp dụng mặc định; dự án có thể override bằng file `output/[tên_dự_án]/design-guideline.md`.

---

## Cơ chế Override

Trước khi bắt đầu wireframe hoặc gen React prototype:

1. Kiểm tra `output/[tên_dự_án]/design-guideline.md` có tồn tại không
2. Nếu **có** → đọc file đó, dùng làm chuẩn chính; chuẩn chung bên dưới chỉ áp dụng cho phần không được override
3. Nếu **không có** → áp dụng toàn bộ chuẩn chung bên dưới

---

## UX Usability Principles

Mục tiêu cốt lõi: **người dùng không cần training vẫn hiểu và thao tác được ngay khi nhìn vào màn hình.**

### 1. Rõ ràng — người dùng luôn biết mình đang ở đâu và cần làm gì

- Tiêu đề trang phải nói rõ màn hình này dùng để làm gì: "Danh sách chiến dịch", "Tạo đơn hàng mới" — không phải tên kỹ thuật như "Campaign Module"
- Nút bấm phải ghi rõ hành động: "Lưu thay đổi", "Gửi yêu cầu" — không phải "OK", "Submit", "Confirm" chung chung
- Icon phải đi kèm text label — không dùng icon đơn thuần khi người dùng chưa quen hệ thống
- Trạng thái hiện tại phải hiển thị rõ: đang ở bước nào, dữ liệu đang ở trạng thái gì

### 2. Nhất quán — thao tác giống nhau ở mọi nơi

- Cùng một hành động luôn dùng cùng một từ ngữ và vị trí xuyên suốt toàn hệ thống
- Màu sắc nhất quán: xanh lá = thành công, đỏ = lỗi/nguy hiểm, vàng = cảnh báo — không đổi ngữ nghĩa giữa các màn hình
- Primary button luôn ở cùng vị trí (góc phải dưới form, góc phải trên list)
- Luồng thao tác quen thuộc: Tạo → Điền form → Lưu / Xem list → Click vào → Xem chi tiết

### 3. Phòng ngừa lỗi — thiết kế để người dùng khó sai hơn là chỉ xử lý khi sai

- Field có định dạng cụ thể → dùng datepicker, dropdown, mask input thay vì text field tự do
- Action nguy hiểm (Xóa, Hủy hợp đồng) → luôn có bước xác nhận với mô tả hậu quả rõ ràng
- Không để người dùng submit form khi thiếu thông tin bắt buộc — disable button hoặc highlight lỗi ngay
- Gợi ý mặc định hợp lý: ngày mặc định = hôm nay, trạng thái mặc định = phổ biến nhất

### 4. Phản hồi tức thì — người dùng luôn biết hệ thống đang làm gì

- Sau mỗi hành động phải có phản hồi ngay: loading spinner, success message, hoặc chuyển màn hình
- Không để màn hình đứng im sau khi bấm nút — người dùng sẽ bấm lại
- Thao tác mất > 2 giây phải có loading indicator
- Thao tác thành công: thông báo rõ "Đã lưu", "Đã gửi" — không chỉ chuyển trang im lặng

### 5. Giảm tải nhận thức — không bắt người dùng nhớ nhiều

- Không bắt người dùng nhớ thông tin từ màn hình trước để dùng ở màn hình sau — hiển thị lại khi cần
- Form dài → chia thành bước (step) hoặc nhóm section có tiêu đề rõ
- Không hiển thị quá nhiều thông tin cùng lúc — ưu tiên thông tin quan trọng nhất, thứ yếu ẩn đi hoặc thu gọn
- Dùng progressive disclosure: hiện thông tin cơ bản trước, chi tiết khi người dùng cần

### 6. Hỗ trợ khi người dùng bị kẹt

- Luồng nào cũng có lối thoát: nút Hủy, nút Quay lại, breadcrumb
- Khi có lỗi → nói rõ lỗi gì và cách sửa, không chỉ hiển thị mã lỗi
- Empty state có hướng dẫn: "Chưa có chiến dịch nào. [+ Tạo chiến dịch đầu tiên]" — không chỉ hiển thị trống

---

## Layout Principles

### Cấu trúc màn hình chuẩn

```
┌─────────────────────────────────┐
│ Header / Page Title + Actions   │
├─────────────────────────────────┤
│ Filter / Search bar (nếu có)    │
├─────────────────────────────────┤
│                                 │
│ Content Area                    │
│ (Table / Form / Cards / List)   │
│                                 │
├─────────────────────────────────┤
│ Pagination / Footer (nếu có)    │
└─────────────────────────────────┘
```

### Nguyên tắc layout

- **Hierarchy rõ**: tiêu đề trang > section heading > label > body text
- **Action placement**: primary action luôn ở góc phải trên (hoặc cuối form)
- **Destructive action** (Xóa, Hủy): tách biệt khỏi primary action, thường ở bên trái
- **Form layout**: single column cho form ngắn (< 6 field); 2 column cho form dài hoặc dense
- **Table**: cột action luôn cố định bên phải; cột ID/mã luôn đầu tiên bên trái

---

## Component Patterns

### Button

| Loại | Dùng khi nào | Ví dụ |
|---|---|---|
| **Primary** | Hành động chính, 1 nút/màn hình | Tạo mới, Lưu, Xác nhận |
| **Secondary** | Hành động phụ | Hủy, Quay lại, Xuất file |
| **Danger** | Hành động không thể hoàn tác | Xóa, Từ chối |
| **Ghost/Link** | Hành động ít quan trọng | Xem chi tiết, Tìm hiểu thêm |

### Form Field

Mỗi field phải có đủ:
- Label (bắt buộc ghi `*` nếu required)
- Placeholder (gợi ý định dạng, không phải label lặp lại)
- Validation message (inline, bên dưới field, màu đỏ)
- Helper text (nếu cần giải thích thêm, màu xám)

### Table

- Có cột checkbox nếu hỗ trợ chọn nhiều
- Header row sticky khi scroll
- Empty state: icon + text mô tả + action gợi ý
- Loading state: skeleton row, không phải spinner toàn trang
- Mỗi row có action menu (3 chấm) hoặc action buttons inline

### Modal / Dialog

- Tiêu đề rõ (động từ + đối tượng: "Tạo chiến dịch", "Xóa tài khoản")
- Tối đa 2 action button: primary (bên phải) + secondary (bên trái)
- Có nút đóng (X) góc trên phải
- Confirm dialog cho destructive action: ghi rõ hậu quả, nút Danger màu đỏ

### Status / Badge

Trạng thái phải có màu nhất quán:

| Trạng thái | Màu | Dùng cho |
|---|---|---|
| Hoạt động / Thành công | Xanh lá | Active, Approved, Completed |
| Chờ xử lý / Nháp | Vàng/Cam | Pending, Draft, In Progress |
| Không hoạt động / Lỗi | Đỏ | Inactive, Rejected, Failed |
| Trung tính / Thông tin | Xám/Xanh dương | Archived, Info |

---

## Naming Convention

### Màn hình (Screen ID)

```
WF-[EpicID]-[ScreenSO]
Ví dụ: WF-001-01, WF-001-02
```

### Component

- Dùng PascalCase cho tên component: `CampaignCard`, `UserTable`
- Dùng camelCase cho props: `isLoading`, `onSubmit`
- Tên phải là danh từ + modifier rõ: `CampaignDetailModal` (không phải `Modal2`)

### Field / Label

- Dùng tiếng Việt cho label hiển thị
- Dùng camelCase tiếng Anh cho field name trong code: `startDate`, `campaignCode`
- Tên field phải nhất quán giữa wireframe, URD/SRS và code

---

## States Bắt Buộc

Mọi màn hình/component phải mô tả đủ các state sau (nếu áp dụng):

| State | Mô tả | Bắt buộc với |
|---|---|---|
| **Default** | Trạng thái bình thường có dữ liệu | Tất cả |
| **Empty** | Chưa có dữ liệu, chưa có kết quả | Table, List, Dashboard |
| **Loading** | Đang tải dữ liệu | Table, Form submit, Page load |
| **Error** | Lỗi hệ thống hoặc validation | Form field, Page load, API call |
| **Disabled** | Không thể tương tác | Button, Field có điều kiện |
| **Read-only** | Hiển thị nhưng không sửa được | Form xem chi tiết, Field theo role |

---

## Error & Feedback Messages

### Nguyên tắc

- **Cụ thể**: "Ngày kết thúc phải sau ngày bắt đầu" — không phải "Dữ liệu không hợp lệ"
- **Hướng dẫn**: nói người dùng cần làm gì, không chỉ nói sai chỗ nào
- **Vị trí**: inline bên dưới field (validation), toast/snackbar (system feedback)

### Loại thông báo

| Loại | Trigger | Hiển thị |
|---|---|---|
| **Validation error** | Submit form sai | Inline dưới field, màu đỏ |
| **Success** | Action thành công | Toast/Snackbar xanh lá, tự đóng sau 3s |
| **Warning** | Action có rủi ro | Banner vàng, cần user confirm |
| **System error** | Lỗi server/network | Toast đỏ + option retry |

---

## Navigation & Flow

- Breadcrumb cho màn hình có hierarchy > 2 cấp
- Back button hoặc breadcrumb — không để người dùng mắc kẹt
- Sau action thành công: redirect rõ ràng (về list, về detail, hay ở lại?)
- Unsaved changes: cảnh báo khi thoát nếu có dữ liệu chưa lưu

---

## Accessibility Cơ Bản

- Field bắt buộc phải có dấu `*` và aria-required
- Error message phải liên kết với field qua aria-describedby
- Button phải có label rõ — không dùng icon đơn thuần không có tooltip
- Contrast tối thiểu: text trên nền phải đủ contrast (WCAG AA)

---

## Cách Dùng

1. Đọc skill này trước khi bắt đầu wireframe hoặc gen React prototype
2. Kiểm tra `output/[tên_dự_án]/design-guideline.md` — nếu có thì ưu tiên file đó
3. Áp dụng chuẩn chung cho mọi phần không được override
4. Khi mô tả component trong wireframe: phải ghi đủ states theo bảng States Bắt Buộc
5. Khi gen React: dùng naming convention và component patterns ở trên

---

## Failure Cases

- Wireframe chỉ mô tả default state, bỏ qua empty/error/loading → tester không có đủ case để test
- Button không phân loại primary/secondary/danger → dev tự quyết định style, UI không nhất quán
- Validation message chung chung → người dùng không biết sửa gì
- Không kiểm tra per-project design-guideline trước khi áp dụng chuẩn chung → override bị bỏ sót
- Nút bấm chỉ có icon không có label → người dùng mới không hiểu phải làm gì
- Không có bước xác nhận cho action nguy hiểm → người dùng xóa nhầm, không khôi phục được
- Sau action thành công không có phản hồi → người dùng bấm lại, gây duplicate
- Empty state không có hướng dẫn → người dùng không biết bắt đầu từ đâu
- Form quá dài không chia bước/nhóm → người dùng bị overwhelmed, bỏ giữa chừng
