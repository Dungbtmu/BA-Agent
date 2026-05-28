---
name: urd_permission_matrix
description: Xây dựng ma trận phân quyền (Permission Matrix) và ma trận ủy quyền RBAC cho URD/SRS — section II.3 + II.4
tools: []
---

# Skill: URD Permission Matrix

## Mục tiêu

Định nghĩa rõ ràng role nào được làm gì — Dev implement đúng access control, Tester viết test case kiểm tra phân quyền (positive + negative).

## Input

- `roles`: danh sách role từ stakeholder mapping hoặc RBAC đã xác định
- `function_tree`: cây phân cấp chức năng từ skill `urd-function-tree`

## Output

- Permission Matrix (II.3): bảng khối chức năng × role, dùng ký hiệu X / (X) / –
- RBAC Matrix (II.4): định nghĩa role + quyền chi tiết + nguyên tắc RBAC

## Thinking Pattern

1. Có bao nhiêu role? Mỗi role phạm vi dữ liệu là gì?
2. Ai có quyền xem toàn hệ thống? Ai chỉ xem dữ liệu của mình?
3. Chức năng nào chỉ Admin mới được làm?
4. Có chức năng nào không thể đảo ngược (xóa sau duyệt, sửa sau gửi) cần chặn đặc biệt?
5. Data scope của từng role có khác nhau không? (xem tất cả vs xem của mình)

## Execution

**Bước 1 — Xác định danh sách role**
- Lấy từ stakeholder mapping hoặc mô tả nghiệp vụ
- Mỗi role: tên, mã code, phạm vi dữ liệu, trách nhiệm chính

**Bước 2 — Xây dựng Permission Matrix (II.3)**
- Hàng = khối chức năng + từng chức năng con
- Cột = từng role
- Điền ký hiệu: `X` (được làm), `(X)` (xem/tổng hợp toàn hệ thống), `–` (không được)
- Thêm phần Ghi chú: giải thích nguyên tắc và trường hợp đặc biệt

**Bước 3 — Xây dựng RBAC Matrix (II.4)**
- Định nghĩa từng role với mô tả phạm vi dữ liệu cụ thể
- Bảng quyền chuẩn: VIEW / CREATE / UPDATE / DELETE / EXPORT / CONFIG / ADMIN
- Ma trận role × khối chức năng với quyền chi tiết
- Viết Nguyên tắc RBAC: phân quyền theo vai trò, data scope, kiểm soát thao tác không đảo ngược

**Bước 4 — Đối chiếu với Function Tree**
- Mọi chức năng trong Function Tree phải xuất hiện trong Permission Matrix
- Không có role nào bị bỏ sót

## Rules

- Quy ước ký hiệu phải nhất quán: `X` / `(X)` / `–` — không dùng ký hiệu khác
- Tên role phải nhất quán với Workflow Diagram, Use Case, Giao diện
- Data scope của từng role phải được ghi rõ — không để mơ hồ
- Chức năng không thể đảo ngược phải được ghi chú rõ ràng
- Không tự thêm role không có trong input

## Output Format

```
### II.3. Ma trận phân quyền hệ thống (Permission Matrix)

**Quy ước:**
- `X` : Được thực hiện
- `(X)` : Được xem/tổng hợp (read-only toàn hệ thống)
- `–` : Không được thực hiện

| Khối chức năng | Chức năng | [Role 1] | [Role 2] | [Role 3] |
|----------------|-----------|----------|----------|----------|
| [Tên khối]     | Xem       | (X)      | X        | –        |
|                | Thêm mới  | –        | X        | X        |
|                | Sửa       | –        | X        | X        |
|                | Xóa       | –        | X        | –        |
|                | Xuất Excel| (X)      | X        | X        |

**Ghi chú:**
- [Nguyên tắc phân quyền và trường hợp đặc biệt]

---

### II.4. Ma trận ủy quyền (RBAC – Authorization Matrix)

**II.4.1. Vai trò**

| Role Code | Tên vai trò | Mô tả & Phạm vi dữ liệu |
|-----------|-------------|--------------------------|
| [CODE]    | [Tên]       | [Phạm vi dữ liệu và trách nhiệm cụ thể] |

**II.4.2. Quy ước quyền**

| Ký hiệu | Ý nghĩa |
|---------|---------|
| VIEW    | Xem dữ liệu |
| CREATE  | Thêm mới |
| UPDATE  | Cập nhật |
| DELETE  | Xóa |
| EXPORT  | Xuất Excel / dữ liệu |
| CONFIG  | Cấu hình hệ thống |
| ADMIN   | Quản trị người dùng & phân quyền |

**II.4.3. Ma trận ủy quyền theo khối chức năng**

| Khối chức năng | Chức năng | [Role 1] | [Role 2] | [Role 3] |
|----------------|-----------|----------|----------|----------|
| [Tên khối]     | [Tên CN]  | VIEW     | VIEW, CREATE, UPDATE, DELETE | VIEW, CREATE |

**Nguyên tắc RBAC:**
- **Phân quyền theo vai trò**: Mỗi role chỉ thấy và thao tác đúng phạm vi được giao
- **Phạm vi dữ liệu (Data Scope)**: [Mô tả scope cụ thể từng role]
- **Kiểm soát thao tác không đảo ngược**: [Liệt kê các thao tác cần chặn ở backend + ẩn ở UI]
```

## Failure Cases

- Bỏ sót chức năng trong Function Tree → Permission Matrix không phủ hết scope
- Data scope mơ hồ ("xem theo phân quyền") → Dev không biết implement gì
- Tên role không nhất quán với các section khác → Dev và Tester dùng khác nhau
- Không ghi chú thao tác không đảo ngược → bị miss khi implement
- Nhầm lẫn giữa Permission Matrix (X/(X)/–) và RBAC Matrix (VIEW/CREATE/...) → merge lại thành 1 bảng gây rối
