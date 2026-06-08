---
name: urd_sequence_diagram
description: Vẽ sequence diagram và bảng diễn giải cho từng quy trình trong URD/SRS — section II.5
tools: []
---

# Skill: URD Sequence Diagram

## Mục tiêu

Mô tả thứ tự tương tác giữa các thành phần (Người dùng, Hệ thống, CSDL, hệ thống ngoài) cho từng quy trình — đủ để Dev biết luồng xử lý cần implement, Tester viết integration test case.

## Input

- `workflows`: danh sách quy trình đã xác định ở Workflow Diagram (skill `urd-workflow-diagram`)
- `external_systems`: danh sách hệ thống bên ngoài cần tích hợp (nếu có)

## Output

- 1 sequence diagram (Mermaid) + bảng diễn giải cho mỗi quy trình

## Thinking Pattern

1. Quy trình này có bao nhiêu giai đoạn lớn?
2. Participant nào tham gia? Ngoài Người dùng / Hệ thống / CSDL, có hệ thống ngoài không?
3. Điều kiện phân nhánh nào cần `alt/else`?
4. Có bước nào lặp lại cần `loop`?
5. Giai đoạn nào nên nhóm vào cùng 1 `rect` để dễ đọc?

## Execution

**Bước 1 — Xác định participant**
- Cơ bản: Người dùng | Hệ thống | Cơ sở dữ liệu
- Thêm nếu có: hệ thống tích hợp bên ngoài (Payment Gateway, SMS Gateway, BSS...)
- Thêm nếu cần: Sub-system nội bộ nếu có nhiều module tương tác với nhau

**Bước 2 — Chia giai đoạn**
- Nhóm các bước liên quan thành giai đoạn có tên rõ ràng
- Mỗi giai đoạn = 1 `rect` trong Mermaid

**Bước 3 — Vẽ sequence diagram**
- Mũi tên đặc (`->>`) cho yêu cầu chủ động
- Mũi tên đứt (`-->>`) cho phản hồi/trả về
- `alt/else` cho mọi nhánh điều kiện — phải có cả nhánh hợp lệ lẫn lỗi
- `loop` nếu có bước lặp lại
- Không ghi chi tiết kỹ thuật (SQL, API endpoint, tên biến)

**Bước 4 — Viết bảng diễn giải**
- Mỗi mũi tên trên sơ đồ = 1 dòng trong bảng
- Nhánh `alt/else` ghi rõ điều kiện kích hoạt
- Bước tự xử lý nội bộ: cột Từ và Đến đều là "Hệ thống"

## Rules

- Số lượng sequence diagram = số lượng quy trình đã xác định ở II.1
- Tên giai đoạn phải nhất quán với tên bước trong Workflow Diagram
- Mọi nhánh điều kiện trong Workflow phải có `alt/else` tương ứng
- Không ghi chi tiết kỹ thuật — giữ ở mức business
- Participant phải nhất quán xuyên suốt tất cả diagram trong cùng tài liệu

## Output Format

```
### II.5. Sơ đồ trình tự (Sequence Diagram)

#### Quy trình 1: [Tên quy trình]

​```mermaid
sequenceDiagram
    actor ND as Người dùng
    participant HT as Hệ thống
    participant DB as Cơ sở dữ liệu
    participant EXT as [Hệ thống ngoài — nếu có]

    rect rgb(240, 248, 255)
        Note over ND,DB: Giai đoạn 1 — [Tên giai đoạn]
        ND->>HT: [Hành động / yêu cầu]
        HT->>DB: [Truy vấn / lưu dữ liệu]
        DB-->>HT: [Kết quả trả về]
        HT-->>ND: [Phản hồi / hiển thị]
    end

    rect rgb(255, 248, 240)
        Note over ND,DB: Giai đoạn 2 — [Tên giai đoạn]
        alt [Điều kiện hợp lệ]
            ND->>HT: [Hành động]
            HT->>DB: [Lưu dữ liệu]
            DB-->>HT: Thành công
            HT-->>ND: Thông báo thành công
        else [Điều kiện không hợp lệ]
            HT-->>ND: Thông báo lỗi cụ thể
        end
    end
​```

**Diễn giải chi tiết:**

| Giai đoạn | Bước | Từ | Đến | Mô tả |
|-----------|------|----|-----|-------|
| [Tên GĐ 1] | 1 | Người dùng | Hệ thống | [Hành động / yêu cầu gửi đi] |
|            | 2 | Hệ thống | Cơ sở dữ liệu | [Xử lý và truy vấn dữ liệu] |
|            | 3 | Cơ sở dữ liệu | Hệ thống | [Kết quả trả về] |
|            | 4 | Hệ thống | Người dùng | [Phản hồi hiển thị lên màn hình] |
| [Tên GĐ 2] | 5 | Người dùng | Hệ thống | [Nhánh hợp lệ: ...] |
|            | 5a | Hệ thống | Người dùng | [Nhánh lỗi: điều kiện ... → thông báo ...] |
```

## Failure Cases

- Số sequence diagram không khớp số quy trình ở II.1 → traceability bị đứt
- Thiếu `alt/else` cho nhánh lỗi → Tester không có input để viết negative test
- Ghi chi tiết kỹ thuật (SQL, API endpoint) → vượt phạm vi BA
- Participant không nhất quán giữa các diagram → gây nhầm lẫn khi đọc
- Bảng diễn giải không khớp với sơ đồ → Dev không biết tin cái nào
