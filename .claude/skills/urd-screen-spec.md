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
- `wireframe`: wireframe đã gen mới nhất từ `ba-wireframe-agent` hoặc React prototype từ `ui-react-agent` — **bắt buộc bám theo phiên bản mới nhất**; nếu có nhiều phiên bản, chỉ dùng phiên bản cuối cùng đã được stakeholder xác nhận
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
- Nếu đã có wireframe/prototype → **bắt buộc đối chiếu với phiên bản mới nhất**; số lượng màn hình trong URD/SRS phải bằng số màn hình trong wireframe/prototype mới nhất
- Đặt tên màn hình nhất quán với tên dùng trong wireframe và UC

**Bước 2 — Đặc tả từng màn hình**
- Mô tả ngắn: màn hình phục vụ mục đích gì, ai dùng
- Liên kết UC tương ứng
- Tham chiếu wireframe (WF-ID hoặc tên màn hình trong prototype) — bắt buộc nếu đã có

**Bước 3 — Viết bảng component**

Mỗi component trên màn hình phải có 1 dòng riêng — **không gộp nhiều component vào 1 dòng**.

Bảng gồm 6 cột bắt buộc:

| TT | Tên thành phần | Định dạng | Bắt buộc | Mặc định | Mô tả |
|-----|----------------|-----------|----------|----------|-------|

- **TT**: số thứ tự, bắt đầu từ 1, tăng dần theo chiều từ trên xuống dưới, trái sang phải trên màn hình
- **Tên thành phần**: tên rõ ràng, đặc trưng — không dùng tên chung chung như "Button 1", "Field A"
- **Định dạng**: Text / Listbox / Date / DateRange / Button / Icon / Checkbox / Radio / Textarea / Search / Table / Badge / Label / Tab / Modal
- **Bắt buộc**: Có / Không — *không được để trống; nếu không áp dụng (ví dụ: Label, Button) → ghi `N/A`*
- **Mặc định**: giá trị khi màn hình load lần đầu — *nếu không có giá trị mặc định → ghi `N/A`*
- **Mô tả**: validation rule + behavior khi tương tác + điều kiện hiển thị/ẩn theo role/trạng thái + thông báo lỗi cụ thể khi vi phạm

**Bước 4 — Xác định log requirement**
- Thao tác Sửa/Xóa/Duyệt → phải ghi log: ai làm, lúc nào, giá trị trước/sau
- Ghi rõ trong cột Mô tả của component/button tương ứng

**Bước 5 — Kiểm tra traceability**
- Mọi UC có ít nhất 1 màn hình tương ứng
- Tên màn hình nhất quán với UC, Workflow, Wireframe

## Rules

- **Bám theo wireframe/prototype mới nhất** — số màn hình phải khớp; nếu prototype đã được cập nhật sau khi wireframe text ra đời, dùng prototype làm nguồn chính
- **Không gộp component** — mỗi dòng trong bảng là 1 component; gộp nhiều field vào 1 dòng làm Dev/Tester không biết đặc tả cho field nào
- **N/A** chỉ dùng cho cột Bắt buộc và Mặc định khi component không có khái niệm đó (Label, Button, Icon, Badge); không dùng N/A cho cột Mô tả hoặc Định dạng
- Thông báo lỗi phải cụ thể: "Ngày kết thúc phải sau ngày bắt đầu" — không phải "Ngày không hợp lệ"
- Validation rule phải có cả điều kiện và thông báo lỗi — thiếu 1 trong 2 là chưa đủ
- Field chỉ hiển thị theo điều kiện phải ghi rõ điều kiện đó
- Log requirement không được bỏ sót — đây là yêu cầu audit thường bị miss

## Output Format

```
### IV. Giao diện chức năng

---

#### [Tên màn hình]
*(Liên kết UC: UC-[MÃ]-[SỐ] | Tham chiếu: WF-[ID] hoặc [Tên màn hình trong prototype])*

[Mô tả ngắn: màn hình này phục vụ mục đích gì, actor nào sử dụng]

| TT | Tên thành phần | Định dạng | Bắt buộc | Mặc định | Mô tả |
|----|----------------|-----------|----------|----------|-------|
| 1  | [Tên component] | Text / Listbox / Date / Button / ... | Có / Không / N/A | [Giá trị mặc định] / N/A | [Validation rule; behavior khi tương tác; điều kiện hiển thị/ẩn theo role/trạng thái; thông báo lỗi cụ thể; log nếu có] |
| 2  | ... | ... | ... | ... | ... |
```

**Lưu ý:**
- Liệt kê hết tất cả component trên màn hình từ trên xuống dưới, trái sang phải
- Mỗi component 1 dòng — không gộp
- Cột Bắt buộc và Mặc định: ghi `N/A` nếu không áp dụng (Label, Button, Icon, Badge)
- Cột Mô tả không được để trống và không được ghi `N/A`

## Failure Cases

- **Số màn hình trong URD/SRS ít hơn trong wireframe/prototype** → một số màn hình không có đặc tả, Dev phải tự đoán
- **Gộp nhiều component vào 1 dòng** → Dev không biết validation áp dụng cho field nào; Tester không test được từng field
- Viết wireframe cũ, bỏ qua prototype đã được update → đặc tả lệch với giao diện thực tế stakeholder đã duyệt
- Thông báo lỗi chung chung ("Dữ liệu không hợp lệ") → Dev tự đặt thông báo, Tester không biết verify gì
- Thiếu điều kiện hiển thị/ẩn của field → Dev tự quyết định, sai với nghiệp vụ
- Không ghi log requirement → bị miss khi implement, phát hiện muộn
- Cột Mô tả để trống hoặc ghi N/A → không có giá trị cho Dev/Tester
- Màn hình không liên kết UC → traceability bị đứt
- Tên màn hình không khớp với UC và Wireframe → BA, Dev, Tester dùng tên khác nhau
