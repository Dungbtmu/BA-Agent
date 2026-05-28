---
name: urd_use_case
description: Viết đặc tả Use Case chi tiết cho URD/SRS — section III, dựa trên Business Function Diagram
tools: []
---

# Skill: URD Use Case Specification

## Mục tiêu

Mô tả chi tiết từng tình huống sử dụng — Dev biết chính xác cần implement logic gì (pre/post condition, business rule), Tester biết cần kiểm thử điều kiện gì (trigger, alternative flow, error case).

## Input

- `function_tree`: cây phân cấp chức năng từ skill `urd-function-tree`
- `permission_matrix`: role và quyền từ skill `urd-permission-matrix`
- `workflows`: workflow diagram đã có để hiểu luồng tổng thể

## Output

- Bộ Use Case đầy đủ, mỗi chức năng trong Function Tree có ít nhất 1 UC

## Thinking Pattern

1. UC này thuộc khối chức năng nào? Actor là role nào theo Permission Matrix?
2. Trigger là gì — người dùng vào màn hình từ đâu? Điều kiện kích hoạt là gì?
3. Pre-condition cần thỏa mãn trước khi UC bắt đầu?
4. Luồng chính (happy path) gồm mấy bước?
5. Có Alternative flow không? (khi người dùng chọn nhánh khác)
6. Exception nào có thể xảy ra? Hệ thống xử lý thế nào?
7. Business Rule nào ràng buộc UC này? Rule có specific và testable không?
8. Post-condition sau khi UC hoàn thành — dữ liệu nào thay đổi?

## Execution

**Bước 1 — Liệt kê UC cần viết**
- Duyệt qua toàn bộ Function Tree
- Mỗi chức năng lá (Xem / Thêm / Sửa / Xóa / Xuất...) → 1 UC
- Đặt mã UC: UC-[MÃ KHỐI]-[SỐ THỨ TỰ] (ví dụ: UC-CM-01)

**Bước 2 — Viết từng UC theo bảng chuẩn**
- Tên Actor phải khớp với Permission Matrix và RBAC
- Activities: bước lẻ (1, 2, 3...) = hành động người dùng; bước chữ (1a, 2a...) = phản hồi hệ thống
- Alternative và Exception phải có đủ — đây là nguồn để Tester viết test case ngoài happy path
- Business Rules: mỗi rule phải cụ thể, có thể viết thành test case ngay

**Bước 3 — Kiểm tra traceability**
- Mọi chức năng trong Function Tree có UC tương ứng
- Tên Actor nhất quán với RBAC Matrix
- Business Rules không mâu thuẫn với Workflow Diagram

## Rules

- Mã UC phải unique và nhất quán — UC-[MÃ KHỐI]-[SỐ] không được trùng
- Actor phải dùng đúng tên role trong RBAC — không tự đặt tên mới
- Business Rule không được viết chung chung ("đảm bảo chính xác", "validate đầy đủ") — phải có điều kiện cụ thể và thông báo lỗi khi vi phạm
- Alternative flow ≠ Exception: Alternative là luồng hợp lệ khác; Exception là điều kiện lỗi
- Post-condition phải ghi rõ trạng thái dữ liệu sau UC — Dev dùng để biết cần lưu gì

## Output Format

```
### III. Đặc tả tình huống sử dụng (Use Case Specification)

---

#### [Tên module]: UC-[MÃ]-[SỐ]: [Tên Use Case]

| Nội dung | Mô tả |
|----------|-------|
| **Tên** | [Tên đầy đủ của use case] |
| **Mục tiêu** | [Mục tiêu nghiệp vụ và giá trị mang lại] |
| **Tác nhân** | [Tên role — phải khớp với RBAC Matrix] |
| **Trigger** | [Điều kiện / sự kiện kích hoạt use case] |
| **Tiền điều kiện** | - [Điều kiện phải thỏa mãn trước khi UC bắt đầu] |
| **Hậu điều kiện** | - [Trạng thái hệ thống sau khi UC hoàn thành]<br>- [Dữ liệu nào được tạo/cập nhật/xóa] |
| **Hoạt động** | 1. [Bước người dùng thực hiện]<br>1a. [Hệ thống xử lý / phản hồi]<br>2. [Bước tiếp theo]<br>2a. [Hệ thống xử lý / phản hồi]<br>**[Alternative]**: [Luồng thay thế hợp lệ]<br>**[Exception]**: [Điều kiện lỗi → hệ thống xử lý thế nào] |
| **Quy tắc nghiệp vụ** | - [Rule cụ thể, testable: điều kiện + thông báo lỗi khi vi phạm]<br>- [Rule 2...]  |
```

## Failure Cases

- Business Rule chung chung ("validate đúng định dạng") → Tester không biết viết test case gì
- Thiếu Exception → miss toàn bộ negative test case
- Actor không khớp với RBAC → Dev implement sai phân quyền
- Không có Post-condition → Dev không biết cần lưu gì sau UC
- Mã UC trùng lặp → tài liệu không trace được
- Nhầm Alternative flow với Exception → luồng hợp lệ bị xử lý như lỗi
