---
name: urd_review_checklist
description: Checklist review URD/SRS theo từng section và từng vai trò — phát hiện lỗ hổng nghiệp vụ, validate thiếu, logic sai, traceability đứt
tools: []
---

# Skill: URD Review Checklist

## Mục tiêu

Cung cấp checklist có cấu trúc để review URD/SRS một cách toàn diện — không bỏ sót lỗ hổng nghiệp vụ, không miss validate, không để traceability bị đứt.

---

## Checklist theo Section

### Section I — Giới thiệu

- [ ] I.1: Mô tả hệ thống làm gì có đủ rõ để người mới đọc hiểu ngay không?
- [ ] I.2: Phạm vi có ghi rõ cả **trong** và **ngoài** scope không? Ranh giới có rõ không?
- [ ] I.3: Thuật ngữ có đủ không? Có từ nào dùng trong tài liệu mà chưa được định nghĩa ở đây không?
- [ ] I.4: Kiến trúc tổng thể có phản ánh đúng các actor và luồng tích hợp thực tế không?

### Section II.1 — Workflow Diagram

- [ ] Swimlane có đủ lane cho tất cả actor không?
- [ ] Mỗi bước có ghi rõ ai thực hiện không?
- [ ] Có đủ nhánh điều kiện (if/else) cho các trường hợp từ chối, lỗi, ngoại lệ không?
- [ ] Bảng diễn giải có khớp với sơ đồ không — không thiếu bước, không thừa bước?
- [ ] Luồng bắt đầu và kết thúc có rõ ràng không?

### Section II.2 — Business Function Diagram

- [ ] Tất cả chức năng đã được nhóm đúng theo nghiệp vụ, không nhóm theo màn hình?
- [ ] Có chức năng nào được đề cập trong Workflow mà không xuất hiện ở đây không?
- [ ] Cây phân cấp có đủ sâu không — chức năng lá có đủ cụ thể để map vào Use Case không?
- [ ] Tên chức năng có dùng động từ + đối tượng rõ ràng không? (ví dụ: "Tạo đơn hàng", không phải "Đơn hàng")

### Section II.3–II.4 — Permission & RBAC Matrix

- [ ] Tất cả chức năng trong Function Tree có trong Permission Matrix không?
- [ ] Có role nào bị bỏ sót không?
- [ ] Phân quyền có hợp lý về nghiệp vụ không? (ví dụ: Nhân viên không nên có quyền xóa dữ liệu lịch sử)
- [ ] Có quyền nào gây conflict không? (ví dụ: cùng 1 người vừa tạo vừa duyệt — vi phạm nguyên tắc phân tách)
- [ ] RBAC Matrix có khớp với Permission Matrix không — cùng tên role, cùng quyền?

### Section II.5 — Sequence Diagram

- [ ] Có đủ `alt/else` cho nhánh hợp lệ và lỗi không?
- [ ] Thứ tự message có đúng không — không có bước nào xảy ra trước điều kiện tiên quyết?
- [ ] Có bước nào trong Workflow mà không được thể hiện trong Sequence Diagram không?
- [ ] Response của hệ thống sau mỗi action có được mô tả không?

### Section III — Use Case Specification

- [ ] Mỗi chức năng lá trong Function Tree có ít nhất 1 UC không?
- [ ] Actor trong UC có khớp với RBAC Matrix không?
- [ ] Pre-condition có đủ không — có điều kiện nào cần thỏa mãn trước mà chưa ghi không?
- [ ] Alternative flow và Exception có đủ không — đây là nguồn test case ngoài happy path?
- [ ] Business Rules có cụ thể và testable không? ("email hợp lệ" không đủ — phải ghi rõ regex hoặc điều kiện)
- [ ] Post-condition có ghi rõ dữ liệu nào được tạo/cập nhật/xóa không?
- [ ] Thông báo lỗi trong Exception có cụ thể không? ("Dữ liệu không hợp lệ" → không đạt)

### Section IV — Giao diện chức năng

- [ ] Mỗi UC có ít nhất 1 màn hình tương ứng không?
- [ ] Bảng component có đủ 4 cột: Định dạng, Bắt buộc, Mặc định, Mô tả không?
- [ ] Validation rule có đủ từ **cơ bản** đến **nâng cao** không?
  - Cơ bản: bắt buộc, định dạng, độ dài tối thiểu/tối đa
  - Nâng cao: ràng buộc liên field, điều kiện nghiệp vụ, phụ thuộc trạng thái
- [ ] Thông báo lỗi có cụ thể không? ("Ngày kết thúc phải sau ngày bắt đầu" — không phải "Ngày không hợp lệ")
- [ ] Field hiển thị theo điều kiện có ghi rõ điều kiện đó không?
- [ ] Log requirement có được ghi cho các thao tác Sửa/Xóa/Duyệt không?
- [ ] Tên màn hình có khớp với tên dùng trong UC và Workflow không?

### Section C — Yêu cầu phi chức năng

- [ ] Mỗi yêu cầu có con số hoặc tiêu chí cụ thể không? ("nhanh" → không đạt; "< 2 giây" → đạt)
- [ ] Có đề cập bảo mật: xác thực, phân quyền, mã hóa, log thao tác nhạy cảm không?
- [ ] Nếu có tích hợp hệ thống ngoài → có mô tả xử lý lỗi tích hợp không?

---

## Checklist theo vai trò

### BA Leader — kiểm tra logic và tính đầy đủ nghiệp vụ

- [ ] Tất cả quy trình nghiệp vụ quan trọng có được cover không?
- [ ] Có assumption nào chưa được xác nhận với stakeholder không?
- [ ] Có yêu cầu nào mâu thuẫn nhau giữa các section không?
- [ ] Scope có bị phình so với mục tiêu ban đầu không?
- [ ] Các edge case nghiệp vụ quan trọng có được xử lý không? (hủy giữa chừng, timeout, dữ liệu trùng lặp)

### PO/PM — kiểm tra sự rõ ràng và đầy đủ với stakeholder

- [ ] Người không có background kỹ thuật đọc có hiểu được không?
- [ ] Ngôn ngữ có thuần nghiệp vụ không — không dùng từ tiếng Anh không phổ biến?
- [ ] Có yêu cầu nào của stakeholder bị bỏ sót không?
- [ ] Phạm vi (scope) có rõ ràng đủ để tránh tranh cãi sau này không?
- [ ] Tên Actor, tên chức năng có nhất quán xuyên suốt không?

### Dev — kiểm tra tính khả thi và đủ chi tiết để implement

- [ ] Đọc xong có thể implement ngay không, hay còn phải tự quyết định logic?
- [ ] Business Rule nào còn mơ hồ — có thể hiểu theo nhiều cách không?
- [ ] Validate rule có đủ để code không — đủ điều kiện + thông báo lỗi?
- [ ] Trạng thái dữ liệu sau mỗi UC có rõ không — lưu gì, cập nhật gì, xóa gì?
- [ ] Có field nào thiếu thông tin về kiểu dữ liệu, độ dài, format không?
- [ ] Log requirement có đủ không — ai làm, lúc nào, giá trị trước/sau?

### Tester — kiểm tra đủ điều kiện để viết test case

- [ ] Mỗi Business Rule có thể viết thành test case ngay không?
- [ ] Alternative flow và Exception có đủ để viết negative test case không?
- [ ] Thông báo lỗi có cụ thể đủ để verify không?
- [ ] Điều kiện hiển thị/ẩn của field có rõ đủ để test không?
- [ ] Edge case nào có thể xảy ra trong thực tế mà chưa được mô tả?

---

## Lỗ hổng nghiệp vụ phổ biến — chủ động tìm

Đây là các lỗi hay bị bỏ sót nhất trong URD/SRS — check kỹ từng mục:

**Về phân quyền:**
- Thiếu phân quyền theo trạng thái: cùng 1 chức năng nhưng chỉ được làm khi dữ liệu ở trạng thái X
- Thiếu quyền xem vs quyền sửa — không phải ai xem được cũng sửa được
- Không có cơ chế thu hồi quyền hoặc hết hạn phiên

**Về luồng nghiệp vụ:**
- Thiếu luồng hủy / rollback khi quy trình đang chạy dở
- Thiếu xử lý timeout — hệ thống làm gì khi không nhận được response?
- Thiếu xử lý dữ liệu trùng lặp — submit 2 lần, double click
- Thiếu luồng xử lý khi dữ liệu liên quan bị xóa (cascade delete hay block delete?)

**Về validate:**
- Validate ngày tháng: chỉ check định dạng mà không check logic (ngày kết thúc > ngày bắt đầu)
- Validate số: không check âm, không check giới hạn trên
- Validate liên field: field A bắt buộc khi field B có giá trị X
- Validate theo role: cùng field nhưng rule khác nhau cho từng role

**Về thông báo và phản hồi:**
- Thiếu thông báo thành công — người dùng không biết action đã được thực hiện
- Thông báo lỗi chung chung không giúp người dùng tự sửa được
- Thiếu trạng thái loading khi hệ thống đang xử lý

**Về dữ liệu:**
- Thiếu mô tả giá trị mặc định khi tạo mới
- Không rõ dữ liệu nào được giữ lại khi xóa mềm vs xóa cứng
- Thiếu log audit cho thao tác nhạy cảm

---

## Cách dùng

1. Duyệt checklist theo section — đánh dấu pass / fail / không áp dụng
2. Với mỗi mục fail → ghi vào Review Report với mức CRITICAL/MAJOR/MINOR
3. Sau checklist section → chạy qua checklist theo vai trò để bắt thêm góc nhìn bị bỏ sót
4. Cuối cùng → đối chiếu với danh sách lỗ hổng phổ biến để chủ động tìm thêm
