---
name: urd_workflow_diagram
description: Vẽ sơ đồ quy trình nghiệp vụ (Swimlane) và bảng diễn giải cho URD/SRS — section II.1
tools: []
---

# Skill: URD Workflow Diagram

## Mục tiêu

Mô tả toàn bộ luồng nghiệp vụ dưới dạng swimlane diagram + bảng diễn giải — đủ để Dev biết cần implement logic gì, Tester biết cần kiểm thử điều kiện gì.

## Input

- `context`: mô tả nghiệp vụ, user flow, hoặc solution đã có
- `actors`: danh sách actor tham gia quy trình

## Output

- Xác định danh sách quy trình cần vẽ
- Mỗi quy trình: Swimlane diagram (Mermaid) + bảng diễn giải

## Thinking Pattern

1. Có bao nhiêu quy trình độc lập? (1 tổng thể hay nhiều quy trình con?)
2. Quy trình nào là trung tâm của hệ thống?
3. Người dùng thao tác gì ở mỗi bước? Hệ thống xử lý/phản hồi gì?
4. Điều kiện phân nhánh là gì? Ai ra quyết định — người dùng hay hệ thống?
5. Các trường hợp lỗi / ngoại lệ nào cần thể hiện?

## Execution

**Bước 1 — Xác định danh sách quy trình**
- Nếu tất cả chức năng phục vụ 1 luồng trung tâm → 1 quy trình tổng thể
- Nếu có nhiều luồng độc lập (ví dụ: Đặt hàng / Hoàn trả / Quản lý kho) → vẽ riêng từng quy trình
- Ghi tên quy trình trước khi vẽ

**Bước 2 — Vẽ Swimlane Diagram (Mermaid)**
- 2 lane: Người dùng và Hệ thống
- Lane Người dùng: thao tác trên UI + quyết định người dùng tự ra
- Lane Hệ thống: validate, xử lý nghiệp vụ, lưu dữ liệu, phân nhánh theo kết quả
- Node mô tả đủ cụ thể — không viết "Xử lý dữ liệu"; viết rõ "Validate định dạng email, kiểm tra trùng lặp → Hiển thị lỗi nếu không hợp lệ"
- Giữ ở mức business logic — không ghi API endpoint, SQL query, tên biến

**Bước 3 — Viết bảng diễn giải**
- Bước chính (1, 2, 3...) = hành động Người dùng
- Bước con (1.1, 2.1...) = xử lý/phản hồi Hệ thống ngay sau bước chính
- Mỗi nhánh điều kiện có dòng riêng, đặt tên rõ: "Trường hợp [điều kiện]"
- Liệt kê đủ trường hợp lỗi — Tester dùng để viết negative test case

## Rules

- Nhánh điều kiện đặt ở lane đúng: người dùng quyết định → lane Người dùng; hệ thống tự phân nhánh → lane Hệ thống
- Kết thúc luôn có bước "Người dùng nhận kết quả / hoàn tất thao tác"
- Không bỏ qua trường hợp lỗi — đây là input quan trọng nhất cho Tester
- Số lượng quy trình phải được xác định và ghi rõ trước khi vẽ

## Output Format

```
### II.1. Sơ đồ quy trình nghiệp vụ (Workflow Diagram)

**Danh sách quy trình:**
1. [Tên quy trình 1]
2. [Tên quy trình 2] (nếu có)

---

#### Quy trình 1: [Tên quy trình]

**Swimlane Diagram:**

​```mermaid
flowchart TD
    subgraph ND["👤 Người dùng"]
        A([Bắt đầu]) --> B["[Thao tác cụ thể trên UI]"]
        B --> D{"[Điều kiện người dùng ra quyết định]?"}
        D -- Có --> F["[Thao tác tiếp theo — nhánh A]"]
        D -- Không --> G["[Thao tác tiếp theo — nhánh B]"]
        F --> H([Kết thúc])
        G --> H
    end

    subgraph HT["⚙️ Hệ thống"]
        B --> C["[Xử lý nghiệp vụ]\n→ [Phản hồi cụ thể]"]
        D2{"[Điều kiện hệ thống tự phân nhánh]?"} --> I["[Xử lý hợp lệ]\n→ Thông báo thành công"]
        D2 --> J["[Xử lý lỗi]\n→ Thông báo lỗi cụ thể"]
        F --> D2
    end
​```

**Diễn giải luồng quy trình:**

| Bước | Tác nhân | Mô tả |
|------|----------|-------|
| 1 | Người dùng | [Thao tác cụ thể trên giao diện] |
| 1.1 | Hệ thống | [Xử lý nghiệp vụ và phản hồi — ghi rõ điều kiện, kết quả] |
| 2 | Người dùng | [Thao tác tiếp theo] |
| 2.1 | Hệ thống | [Xử lý tương ứng] |
| **[Trường hợp lỗi]** | Hệ thống | [Điều kiện kích hoạt lỗi và thông báo hiển thị] |
```

## Failure Cases

- Vẽ 1 sơ đồ gộp tất cả quy trình → quá phức tạp, không ai đọc được
- Node mô tả chung chung ("Xử lý", "Kiểm tra") → Dev không biết implement gì
- Bỏ qua trường hợp lỗi → Tester thiếu test case quan trọng
- Đặt nhánh điều kiện sai lane → Dev hiểu nhầm ai ra quyết định
- Ghi chi tiết kỹ thuật (SQL, API) → vượt phạm vi BA
