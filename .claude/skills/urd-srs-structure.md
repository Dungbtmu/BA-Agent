---
name: urd_srs_structure
description: Orchestrator URD/SRS — cấu trúc tổng thể tài liệu, thứ tự thực hiện các skill con, traceability và checklist trước khi output
tools: []
---

# Skill: URD/SRS Structure (Orchestrator)

## Mục tiêu

Đảm bảo tài liệu URD/SRS được viết đúng cấu trúc, đúng thứ tự, nhất quán và đủ chi tiết để Dev/Tester có thể implement ngay.

---

## Cấu trúc tài liệu chuẩn

```
[TÊN TỔNG CÔNG TY / ĐƠN VỊ]
TÀI LIỆU ĐẶC TẢ YÊU CẦU NGƯỜI DÙNG
[TÊN HỆ THỐNG / MODULE]
[Địa điểm – Tháng MM/YYYY]

CÁC THAY ĐỔI | TRANG KÍ | MỤC LỤC

I.   GIỚI THIỆU
     I.1  Mục đích tài liệu
     I.2  Phạm vi tài liệu
     I.3  Định nghĩa thuật ngữ và từ viết tắt
     I.4  Kiến trúc tổng thể hệ thống

II.  CÁC YÊU CẦU VỀ TỔNG THỂ PHẦN MỀM
     II.1 Sơ đồ quy trình nghiệp vụ (Workflow Diagram)
     II.2 Sơ đồ phân cấp chức năng (Business Function Diagram)
     II.3 Ma trận phân quyền (Permission Matrix)
     II.4 Ma trận ủy quyền (RBAC Matrix)
     II.5 Sơ đồ trình tự (Sequence Diagram)

III. ĐẶC TẢ TÌNH HUỐNG SỬ DỤNG (USE CASE SPECIFICATION)

IV.  GIAO DIỆN CHỨC NĂNG (PROTOTYPE CHÍNH)

C.   YÊU CẦU PHI CHỨC NĂNG
```

---

## Thứ tự thực hiện và skill tương ứng

Các section phụ thuộc nhau theo thứ tự — không được đảo:

```
I.1–I.3  Giới thiệu          → tự viết (không có skill riêng)
I.4      Kiến trúc           → tự viết dựa trên context dự án
   ↓
II.1     Workflow Diagram     → skill: urd-workflow-diagram
   ↓
II.2     Function Tree        → skill: urd-function-tree
   ↓
II.3–II.4 Permission + RBAC  → skill: urd-permission-matrix
   ↓
II.5     Sequence Diagram     → skill: urd-sequence-diagram  (dựa trên II.1)
   ↓
III.     Use Case             → skill: urd-use-case          (dựa trên II.2 + II.3–II.4)
   ↓
IV.      Screen Spec          → skill: urd-screen-spec       (dựa trên III + wireframe)
   ↓
C.       Non-functional       → tự viết dựa trên constraint đã xác định
```

---

## Section I — Giới thiệu (tự viết)

### I.1 Mục đích tài liệu
- Hệ thống làm gì (2–3 câu)
- Đối tượng sử dụng tài liệu (BA, Dev, Tester, SA...)
- Vai trò trong vòng đời dự án

### I.2 Phạm vi tài liệu
- Phạm vi chức năng được bao gồm
- Phạm vi không bao gồm (nếu cần làm rõ ranh giới)

### I.3 Định nghĩa thuật ngữ và từ viết tắt

| STT | Thuật ngữ | Diễn giải |
|-----|-----------|-----------|
| 1   | [Tên]     | [Mô tả đầy đủ] |

### I.4 Kiến trúc tổng thể hệ thống

Vẽ Mermaid block diagram thể hiện các layer và luồng tích hợp. Cấu trúc điển hình:

```
Nguồn dữ liệu đầu vào → Lớp tích hợp → Lớp nghiệp vụ (core) → Lớp giao diện → Hệ thống ngoài
```

Sau sơ đồ: diễn giải từng lớp (tên, mục đích, thành phần chính, luồng vào/ra).

---

## Section C — Yêu cầu phi chức năng (tự viết)

Viết theo các nhóm, mỗi yêu cầu phải có con số hoặc tiêu chí cụ thể:

1. **Kiến trúc hệ thống** — mô hình, yêu cầu triển khai
2. **Ràng buộc thiết kế** — CSDL, toàn vẹn dữ liệu, backup, bảo mật DB
3. **Giao diện người dùng** — ngôn ngữ, design system, thông báo lỗi/thành công
4. **An toàn, bảo mật** — xác thực, mã hóa, phân quyền, log thao tác nhạy cảm
5. **Hiệu năng** (nếu có) — thời gian phản hồi, số user đồng thời
6. **Tích hợp** (nếu có) — hệ thống ngoài, giao thức, xử lý lỗi tích hợp

---

## Traceability — Chuỗi truy vết bắt buộc

```
Business Function → Permission Matrix → Use Case → Giao diện
```

Trước khi output, kiểm tra:
- [ ] Mọi chức năng trong Function Tree có trong Permission Matrix
- [ ] Mọi chức năng trong Permission Matrix có ít nhất 1 Use Case
- [ ] Mọi Use Case có ít nhất 1 màn hình giao diện tương ứng
- [ ] Tên Actor nhất quán xuyên suốt (Workflow, UC, RBAC, Giao diện)
- [ ] Tên chức năng nhất quán (Function Tree, Permission Matrix, UC)
- [ ] Thuật ngữ dùng đúng theo bảng I.3

---

## Checklist chất lượng trước khi output

**Nội dung:**
- [ ] Business Rules trong UC cụ thể, testable — không viết chung chung
- [ ] Validation rule đủ từ cơ bản (bắt buộc, định dạng, độ dài) đến nâng cao (ràng buộc liên field, điều kiện nghiệp vụ)
- [ ] Thông báo lỗi cụ thể — không phải "dữ liệu không hợp lệ"
- [ ] Sequence Diagram có đủ `alt/else` (nhánh hợp lệ + lỗi)
- [ ] Bảng component giao diện đủ: Định dạng, Bắt buộc, Mặc định, Mô tả
- [ ] Log requirement được ghi trong bảng component

**Format:**
- [ ] Đúng cấu trúc section I → II → III → IV → C — không bỏ section nào
- [ ] Sơ đồ Mermaid render được — cú pháp hợp lệ
- [ ] Bảng Markdown đủ cột, không có ô trống không giải thích
- [ ] Không có placeholder vô nghĩa (trừ `[Cần xác nhận: ...]`)
- [ ] Không dùng TBD, N/A trong nội dung hoàn chỉnh

---

## Quy tắc nhất quán toàn tài liệu

- **Không dùng**: TBD, N/A trong nội dung hoàn chỉnh
- **Ghi rõ**: `[Cần xác nhận: ...]` tại chỗ BA chưa có thông tin — không bỏ trống
- **Không bỏ section**: thiếu thông tin → ghi `[Cần xác nhận]`, không bỏ section
