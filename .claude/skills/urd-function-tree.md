---
name: urd_function_tree
description: Xây dựng sơ đồ phân cấp chức năng (Business Function Diagram) cho URD/SRS — section II.2
tools: []
---

# Skill: URD Function Tree

## Mục tiêu

Liệt kê toàn bộ scope chức năng cần implement dưới dạng cây phân cấp — Dev dùng để ước lượng effort, Tester dùng để lập test plan, BA dùng để đảm bảo không miss tính năng nào.

## Input

- `features`: danh sách tính năng từ Epic/User Stories hoặc mô tả nghiệp vụ
- `actors`: danh sách role sử dụng hệ thống (để xác định khối quản trị)

## Output

- Cây phân cấp chức năng (text-based)
- Diễn giải từng khối chức năng

## Thinking Pattern

1. Các chức năng nhóm lại theo nghiệp vụ nào? (không nhóm theo màn hình hay technical layer)
2. Mỗi khối có những thao tác CRUD nào?
3. Có khối Quản trị hệ thống không? (phân quyền, danh mục...)
4. Có chức năng nào bị miss so với workflow đã vẽ không?
5. Tên khối có nhất quán với tên dùng trong Workflow Diagram không?

## Execution

**Bước 1 — Xác định các khối chức năng**
- Nhóm theo nghiệp vụ, không phải theo màn hình hoặc tech layer
- Mỗi khối là 1 nhóm chức năng phục vụ cùng 1 mục tiêu nghiệp vụ
- Luôn có khối Quản trị hệ thống (quản lý user, phân quyền, danh mục...)

**Bước 2 — Liệt kê chức năng con từng khối**
- Các thao tác cơ bản: Xem danh sách / Thêm mới / Sửa / Xóa / Xuất Excel
- Các chức năng đặc thù của khối (ví dụ: Duyệt, Gửi, Hủy, Kích hoạt...)
- Không bỏ sót chức năng đã xuất hiện trong Workflow Diagram

**Bước 3 — Đối chiếu với Workflow Diagram**
- Mọi hành động trong workflow phải có chức năng tương ứng trong cây
- Nếu phát hiện thiếu → bổ sung vào đúng khối

**Bước 4 — Viết diễn giải từng khối**
- Mục đích: khối này phục vụ nghiệp vụ gì
- Giá trị nghiệp vụ: tại sao quan trọng với người dùng
- Các chức năng con: liệt kê đủ để đối chiếu với Use Case và màn hình

## Rules

- Tên khối phải nhất quán với tên dùng trong Workflow Diagram và các section khác
- Không nhóm theo màn hình (ví dụ: "Trang chủ", "Trang chi tiết") — nhóm theo nghiệp vụ
- Không ghi chức năng kỹ thuật (cache, logging, authentication flow) — đó là phạm vi SA/Dev
- Mọi khối phải có ít nhất 1 Use Case tương ứng ở section III

## Output Format

```
### II.2. Sơ đồ phân cấp chức năng (Business Function Diagram)

​```
[Tên hệ thống]
├── Khối 1: [Tên khối]
│   ├── Xem danh sách
│   ├── Thêm mới
│   ├── Sửa
│   ├── Xóa
│   └── [Chức năng đặc thù]
├── Khối 2: [Tên khối]
│   ├── [Chức năng 1]
│   └── [Chức năng 2]
└── Khối N: Quản trị hệ thống
    ├── Quản lý người dùng & phân quyền
    └── Khai báo danh mục
​```

**Diễn giải:**

**Khối 1: [Tên khối]**
- **Mục đích:** [Phục vụ nghiệp vụ gì]
- **Giá trị nghiệp vụ:** [Tại sao quan trọng với người dùng]
- **Chức năng con:** Xem danh sách, Thêm mới, Sửa, Xóa, [...]
```

## Failure Cases

- Nhóm theo màn hình thay vì nghiệp vụ → cây không reflect business logic
- Bỏ sót chức năng đã có trong Workflow → traceability bị đứt
- Không có khối Quản trị → miss toàn bộ requirement về phân quyền
- Tên khối không nhất quán với Workflow → BA và Dev dùng khác nhau gây nhầm lẫn
- Ghi chức năng kỹ thuật vào cây → vượt phạm vi BA
