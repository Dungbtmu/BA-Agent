---
name: ui_feedback_triage
description: Phân loại feedback UI từ nhiều nguồn — map về đúng component/màn hình cần sửa, tách riêng wireframe vs React
tools: []
---

# Skill: UI Feedback Triage

## Mục tiêu

Biến feedback thô từ nhiều nguồn (BA, PO, khách hàng, dev) thành danh sách thay đổi có cấu trúc — rõ ràng đủ để sửa đúng chỗ, không sửa thừa, không miss.

## Input

- `feedback_raw`: ghi chú feedback thô — có thể là bullet, đoạn văn, hoặc danh sách lộn xộn
- `feedback_source`: ai đưa feedback (BA / PO / Khách hàng / Dev)
- `current_wireframe`: tên/version wireframe hiện tại đang được feedback
- `current_ui`: tên/version React component hiện tại (nếu có)

## Output

- Danh sách thay đổi phân loại theo type và target
- Phân tách rõ: cái nào sửa wireframe, cái nào sửa React component, cái nào cần hỏi lại

## Thinking Pattern

1. **Feedback nói về gì?** — Visual (màu sắc, spacing, font) / Behavior (click xảy ra gì) / Content (text, label, dữ liệu) / Flow (thứ tự màn hình, navigation)
2. **Sửa ở đâu?** — Wireframe (nếu là thay đổi layout/flow/logic) / React (nếu chỉ là visual/content) / Cả hai
3. **Màn hình / component nào bị ảnh hưởng?** — Map feedback về đúng WF-ID hoặc tên component
4. **Rõ chưa?** — Nếu feedback mơ hồ → đặt câu hỏi clarify trước khi assign
5. **Conflict không?** — Feedback từ các nguồn khác nhau có mâu thuẫn nhau không?

## Execution

**Bước 1 — Đọc toàn bộ feedback, không xử lý ngay**
- Đọc hết một lượt
- Đánh dấu những chỗ mơ hồ hoặc có thể conflict

**Bước 2 — Phân loại từng feedback item**

| Type | Định nghĩa | Ví dụ |
|---|---|---|
| **Visual** | Thay đổi giao diện, không thay đổi logic | "Nút submit màu xanh", "Tăng font size" |
| **Behavior** | Thay đổi hành vi khi tương tác | "Click nút X thì hiện popup, không phải chuyển trang" |
| **Content** | Thay đổi text, label, dữ liệu hiển thị | "Đổi 'Lưu' thành 'Xác nhận'", "Thêm cột Ngày tạo" |
| **Flow** | Thay đổi thứ tự màn hình, navigation | "Bỏ màn hình confirm, gộp vào màn hình chính" |
| **Layout** | Thay đổi bố cục, vị trí element | "Chuyển filter từ sidebar sang top bar" |

**Bước 3 — Map về target**
- Visual + Content → chỉ sửa React component
- Behavior + Layout + Flow → sửa Wireframe trước, rồi React theo
- Nếu không rõ → đặt câu hỏi, không tự suy diễn

**Bước 4 — Phát hiện conflict**
- So sánh các feedback item với nhau và với wireframe hiện tại
- Nếu có conflict → liệt kê rõ, không tự quyết định

**Bước 5 — Tổng hợp danh sách việc cần làm**
- Nhóm theo màn hình / component
- Ghi rõ thứ tự: sửa wireframe trước, React sau (nếu có thay đổi flow/layout)

## Rules

- Không tự sửa khi feedback mơ hồ — phải hỏi trước
- Không gộp feedback từ nguồn khác nhau nếu có thể conflict
- Luôn ghi rõ nguồn feedback cho mỗi item — để trace lại khi cần
- Phân biệt rõ "sửa wireframe" vs "sửa React" — không để hai bên lệch nhau
- Nếu một feedback ảnh hưởng nhiều màn hình → liệt kê hết, không chỉ ghi màn hình đầu tiên

## Output Format

```
## Feedback Triage — [Tên dự án] / [Wireframe version]

**Nguồn feedback:** [BA / PO / Khách hàng / Dev]
**Ngày:** [...]
**Tổng số items:** [N]

---

### Cần hỏi lại trước khi sửa

1. "[Trích dẫn feedback gốc]" — *Chưa rõ: ...*

---

### Thay đổi Wireframe + React (Layout / Flow / Behavior)

| # | Feedback | Type | Màn hình / WF-ID | Thay đổi cụ thể |
|---|---|---|---|---|
| 1 | [Trích dẫn] | Flow | WF-001-02 | [Mô tả thay đổi rõ ràng] |
| 2 | [Trích dẫn] | Layout | WF-001-03 | [Mô tả thay đổi rõ ràng] |

### Chỉ thay đổi React (Visual / Content)

| # | Feedback | Type | Component | Thay đổi cụ thể |
|---|---|---|---|---|
| 3 | [Trích dẫn] | Visual | `CampaignCard` | [Mô tả thay đổi rõ ràng] |
| 4 | [Trích dẫn] | Content | `SubmitButton` | Đổi label từ "Lưu" → "Xác nhận" |

---

### Conflict cần xử lý

- Item #2 vs Item #5: [Mô tả mâu thuẫn] — *Cần PO quyết định*

---

### Thứ tự thực hiện

1. Xử lý câu hỏi clarify (nếu có)
2. Sửa Wireframe: [Danh sách WF-ID cần update]
3. Sửa React: [Danh sách component cần update]
```

## Failure Cases

- Tự suy diễn feedback mơ hồ → sửa sai ý
- Quên map feedback về đúng màn hình → sửa nhầm chỗ
- Không phát hiện conflict giữa feedback từ PO và khách hàng → sửa xong phải sửa lại
- Gộp "sửa wireframe" và "sửa React" vào cùng một bước → wireframe và React lệch nhau
- Không ghi nguồn feedback → không trace lại được khi bị hỏi "ai yêu cầu cái này"
