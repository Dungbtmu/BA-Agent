---
name: traceability_map
description: Xây dựng và duy trì Traceability Map — bảng link từ REQ → Use Case → Wireframe → URD section → Story/AC; nền tảng để orchestrator tính impact khi requirement thay đổi
tools: []
---

# Skill: Traceability Map

## Mục tiêu

Tạo và duy trì một bản đồ traceability rõ ràng, machine-readable — linking từng requirement đến tất cả artifact phụ thuộc. Khi requirement thay đổi, orchestrator tra cứu map này để biết chính xác cần cập nhật gì.

---

## ID Schema

### Requirement ID

Format: `REQ-[DỰ ÁN]-[NNN]`

- `[DỰ ÁN]` — prefix 2-4 ký tự viết hoa, ví dụ: `CDP`, `CVM`, `BSS`
- `[NNN]` — số thứ tự 3 chữ số, bắt đầu từ `001`

Ví dụ: `REQ-CDP-001`, `REQ-CVM-042`

### Artifact ID (đã có trong hệ thống)

| Artifact | ID format | Ví dụ |
|---|---|---|
| Use Case | `UC-NNN` | `UC-001` |
| Wireframe | `WF-NNN-NNN` | `WF-001-002` |
| URD Section | `URD-[section]` | `URD-II.1`, `URD-III-UC001`, `URD-IV-WF001` |
| User Story | `STORY-NNN-NNN` | `STORY-001-002` |
| Acceptance Criteria | `AC-NNN-NNN-NNN` | `AC-001-002-001` |

---

## Cấu trúc Traceability Map

File lưu tại: `.claude/output/[tên_dự_án]/traceability-map.md`

### Format bảng chính

```markdown
## Traceability Matrix — [Tên dự án]
Version: [N] | Cập nhật: [YYYY-MM-DD]

| REQ ID | Mô tả ngắn | UC | WF | URD Section | Story | Ghi chú |
|---|---|---|---|---|---|---|
| REQ-XXX-001 | [Mô tả 1 dòng] | UC-001, UC-002 | WF-001-001 | URD-II.1, URD-III-UC001, URD-IV-WF001 | STORY-001-001 | |
| REQ-XXX-002 | [Mô tả 1 dòng] | UC-003 | WF-002-001, WF-002-002 | URD-III-UC003, URD-IV-WF002 | STORY-002-001, STORY-002-002 | |
```

### Reverse index — tra cứu theo artifact

Phần này giúp orchestrator tra ngược: artifact này phụ thuộc vào REQ nào.

```markdown
## Reverse Index

### Use Cases
| UC ID | Phụ thuộc REQ |
|---|---|
| UC-001 | REQ-XXX-001 |
| UC-002 | REQ-XXX-001, REQ-XXX-003 |

### Wireframes
| WF ID | Phụ thuộc REQ |
|---|---|
| WF-001-001 | REQ-XXX-001 |
| WF-002-001 | REQ-XXX-002 |

### URD Sections
| URD Section | Phụ thuộc REQ |
|---|---|
| URD-II.1 | REQ-XXX-001, REQ-XXX-002 |
| URD-III-UC001 | REQ-XXX-001 |
| URD-IV-WF001 | REQ-XXX-001 |

### Stories
| STORY ID | Phụ thuộc REQ |
|---|---|
| STORY-001-001 | REQ-XXX-001 |
```

---

## Quy trình xây dựng Traceability Map ban đầu

### Bước 1 — Thu thập requirement

Đọc tất cả tài liệu input (PRD, clarification output, solution output) và trích xuất danh sách requirement:

```
1. Mỗi yêu cầu chức năng rõ ràng = 1 REQ ID
2. Mỗi rule nghiệp vụ quan trọng = 1 REQ ID
3. Không tạo REQ cho: mô tả UI thuần túy, yêu cầu phi chức năng (dùng section C URD)
```

### Bước 2 — Link sang artifact

Với mỗi REQ ID:
1. Tìm Use Case nào trong URD mô tả nghiệp vụ này → ghi vào cột UC
2. Tìm Wireframe nào thể hiện chức năng này → ghi vào cột WF
3. Tìm URD Section nào chứa nội dung liên quan → ghi vào cột URD Section
4. Tìm Story nào implement requirement này → ghi vào cột Story

### Bước 3 — Tạo Reverse Index

Từ bảng chính, đảo ngược: với mỗi artifact ID, liệt kê tất cả REQ mà nó phụ thuộc.

### Bước 4 — Validate

- Mỗi REQ phải có ít nhất 1 UC và 1 URD Section
- Mỗi UC phải có ít nhất 1 REQ nguồn (không có UC "mồ côi")
- Mỗi WF phải có ít nhất 1 REQ nguồn
- Flag bất kỳ artifact nào không có REQ nguồn → cần BA xác nhận tại sao tồn tại

---

## Quy trình cập nhật khi có thay đổi

Khi `impact-analysis` gửi danh sách REQ thay đổi:

### Nếu REQ bị sửa (MODIFY)
1. Cập nhật mô tả ngắn trong bảng chính
2. Kiểm tra lại link: có artifact nào cần thêm/bớt không?
3. Cập nhật Reverse Index nếu link thay đổi
4. Ghi vào version history: `| [YYYY-MM-DD] | MODIFY | REQ-XXX-NNN | [Mô tả thay đổi] |`

### Nếu REQ mới (ADD)
1. Thêm dòng mới vào bảng chính với ID tiếp theo
2. Link sang artifact mới (sẽ được tạo bởi artifact-patch)
3. Cập nhật Reverse Index
4. Ghi vào version history: `| [YYYY-MM-DD] | ADD | REQ-XXX-NNN | [Mô tả] |`

### Nếu REQ bị xóa (REMOVE)
1. KHÔNG xóa dòng REQ — đánh dấu `[DEPRECATED]` trong cột Ghi chú
2. Giữ nguyên link để biết artifact nào bị ảnh hưởng
3. Ghi vào version history: `| [YYYY-MM-DD] | REMOVE | REQ-XXX-NNN | [Lý do] |`
4. Sau khi artifact đã được cleanup, mới remove khỏi bảng

---

## Version History trong map file

```markdown
## Version History

| Ngày | Action | REQ ID | Ghi chú |
|---|---|---|---|
| 2026-06-05 | INIT | ALL | Tạo map từ clarification + solution output |
| 2026-06-10 | MODIFY | REQ-CDP-003 | PO đổi rule phân quyền |
| 2026-06-12 | ADD | REQ-CDP-015 | Bổ sung yêu cầu export báo cáo |
```

---

## Rules

- Một REQ có thể link đến nhiều UC, WF, Story — quan hệ N-N là bình thường
- Một artifact (UC, WF) cũng có thể phụ thuộc nhiều REQ — đây là lý do phải có Reverse Index
- KHÔNG tự suy luận link nếu không chắc — ghi `[Cần xác nhận]` vào cột tương ứng
- Map phải được cập nhật ĐỒNG THỜI với artifact — không để map lệch phase so với tài liệu thực tế
- Sau mỗi lần orchestrator chạy patch, phải cập nhật version history trong map

## Failure Cases

- Bỏ sót artifact khi link → impact analysis tính thiếu, bỏ sót file cần cập nhật
- REQ quá thô (1 REQ = cả một module) → impact tính quá rộng, mọi thứ đều bị flag là bị ảnh hưởng
- REQ quá chi tiết (1 REQ = 1 trường trong form) → map quá dài, khó maintain
- Không cập nhật Reverse Index → orchestrator tra cứu theo artifact sẽ sai
