---
name: urd_screen_spec
description: Đặc tả giao diện chức năng (màn hình, component, validation) cho URD/SRS — section IV
tools: []
---

# Skill: URD Screen Specification

## Mục tiêu

Đặc tả từng màn hình đủ chi tiết để Dev implement UI đúng behavior, Tester viết test case kiểm tra từng field/action mà không cần hỏi lại BA.

## Input

- `use_cases`: danh sách UC từ skill `urd-use-case` — mỗi UC cần ít nhất 1 màn hình
- `wireframe`: wireframe đã có từ `ba-wireframe-agent` (nếu có) — dùng làm tham chiếu layout
- `actors`: danh sách role để biết màn hình nào hiển thị cho ai

## Output

- Đặc tả đầy đủ từng màn hình: mô tả, bảng component, hành vi từng field

## Thinking Pattern

1. Màn hình này liên kết với UC nào? Actor là ai?
2. Layout tổng thể trông như thế nào?
3. Từng field: type gì, bắt buộc không, mặc định là gì, validate rule gì?
4. Field nào chỉ hiển thị theo điều kiện (role, trạng thái dữ liệu)?
5. Button / action nào cần log (ai làm, lúc nào, giá trị trước/sau)?
6. Thông báo lỗi cụ thể là gì — không phải "dữ liệu không hợp lệ"?

## Execution

**Bước 1 — Liệt kê màn hình cần đặc tả**
- Duyệt qua danh sách UC — mỗi UC cần ít nhất 1 màn hình
- Nếu đã có wireframe → dùng làm tham chiếu layout, không cần mô tả lại từ đầu
- Đặt tên màn hình nhất quán với tên dùng trong wireframe và UC

**Bước 2 — Đặc tả từng màn hình**
- Mô tả ngắn: màn hình phục vụ mục đích gì, ai dùng
- Liên kết UC tương ứng
- Mô tả layout tổng thể (không cần chi tiết nếu đã có wireframe)

**Bước 3 — Viết bảng component**
Mỗi field/component phải có:
- **Định dạng**: Text / Listbox / Date / Button / Icon / Checkbox / Radio / Textarea / Search
- **Bắt buộc**: Có / Không
- **Mặc định**: giá trị khi màn hình load lần đầu
- **Mô tả**: validation rule + behavior + điều kiện hiển thị/ẩn + thông báo lỗi cụ thể

**Bước 4 — Xác định log requirement**
- Thao tác Sửa/Xóa/Duyệt → phải ghi log: ai làm, lúc nào, giá trị trước/sau
- Ghi rõ trong cột Mô tả của component/button tương ứng

**Bước 5 — Kiểm tra traceability**
- Mọi UC có ít nhất 1 màn hình tương ứng
- Tên màn hình nhất quán với UC, Workflow, Wireframe

## Rules

- Thông báo lỗi phải cụ thể: "Ngày kết thúc phải sau ngày bắt đầu" — không phải "Ngày không hợp lệ"
- Validation rule phải có cả điều kiện và thông báo lỗi — thiếu 1 trong 2 là chưa đủ
- Field chỉ hiển thị theo điều kiện phải ghi rõ điều kiện đó
- Không viết "N/A" trong cột Bắt buộc hoặc Mặc định — phải có giá trị rõ ràng
- Log requirement không được bỏ sót — đây là yêu cầu audit thường bị miss

## Output Format

```
### IV. Giao diện chức năng (Prototype chính)

---

#### [Tên màn hình]
*(Liên kết UC: UC-[MÃ]-[SỐ])*

[Mô tả ngắn: màn hình này phục vụ mục đích gì, actor nào sử dụng]

[Tham chiếu wireframe: WF-[ID] — nếu có]

| STT | Tên thành phần | Định dạng | Bắt buộc | Mặc định | Mô tả |
|-----|----------------|-----------|----------|----------|-------|
| 1 | [Tên field] | Text / Listbox / Date / Button / ... | Có / Không | [Giá trị mặc định] | [Validation rule; behavior khi tương tác; điều kiện hiển thị/ẩn; thông báo lỗi cụ thể khi vi phạm; log requirement nếu có] |
```

## Failure Cases

- Thông báo lỗi chung chung ("Dữ liệu không hợp lệ") → Dev tự đặt thông báo, Tester không biết verify gì
- Thiếu điều kiện hiển thị/ẩn của field → Dev tự quyết định, sai với nghiệp vụ
- Không ghi log requirement → bị miss khi implement, phát hiện muộn
- Dùng "N/A" trong cột Bắt buộc → Dev không biết có cần validate không
- Màn hình không liên kết UC → traceability bị đứt
- Tên màn hình không khớp với UC và Wireframe → BA, Dev, Tester dùng tên khác nhau
