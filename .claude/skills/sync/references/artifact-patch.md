---
name: artifact_patch
description: Patch đúng phần bị ảnh hưởng trong từng artifact (URD section, Wireframe, Use Case, Story/AC) theo Impact Report — không viết lại toàn bộ tài liệu
tools: []
---

# Skill: Artifact Patch

## Mục tiêu

Nhận Impact Report từ `impact-analysis`, thực hiện patch chính xác từng artifact theo đúng thứ tự. Nguyên tắc cốt lõi: **chỉ sửa đúng phần bị ảnh hưởng — không viết lại toàn bộ tài liệu**.

---

## Input

- Impact Report từ `impact-analysis`
- File artifact hiện tại trong `.claude/output/[tên_dự_án]/`
- Traceability Map

---

## Quy tắc patch chung

1. **Đọc artifact hiện tại trước** — không patch mù mà không biết nội dung hiện tại
2. **Xác định đúng vị trí** — heading, bảng, dòng cụ thể cần sửa
3. **Patch minimal** — chỉ thay đổi nội dung liên quan, giữ nguyên phần còn lại
4. **Tăng version** — mỗi lần patch URD/SRS → tăng version tài liệu và cập nhật version history
5. **Ghi change log** — ghi rõ đã sửa gì, tại section nào, lý do

---

## Patch từng loại artifact

### 1. URD/SRS Section Patch

**File**: `.claude/output/[tên_dự_án]/urd/urd-srs-v[N].md`

#### URD-II.1 — Workflow Diagram

Khi flow tổng thể thay đổi:
- Tìm swimlane hoặc bước bị ảnh hưởng trong Mermaid diagram
- Thêm/sửa/xóa node và arrow đúng bước đó
- Cập nhật chú thích nếu có
- KHÔNG vẽ lại toàn bộ diagram

```
Patch pattern:
  OLD: A --> B: Supervisor duyệt
  NEW: A --> B: Supervisor duyệt (nếu ≤ 500M)
       A --> C: Director duyệt (nếu > 500M)
```

#### URD-II.2 — Function Tree

Khi thêm/bớt chức năng:
- Tìm nhánh cây bị ảnh hưởng
- Thêm node mới vào đúng vị trí trong cây
- Cập nhật ID chức năng (không đánh số lại toàn bộ)

#### URD-II.3 — Permission Matrix

Khi thay đổi phân quyền:
- Tìm dòng chức năng bị ảnh hưởng
- Sửa đúng ô tại giao chức năng × role
- Nếu thêm role mới → thêm cột mới, giữ nguyên các dòng hiện tại

```
Patch pattern (bảng Markdown):
  OLD: | Duyệt đơn | ✓ | ✓ | - |
  NEW: | Duyệt đơn ≤ 500M | ✓ | - | - |
       | Duyệt đơn > 500M | - | - | ✓ | (dòng mới)
```

#### URD-II.4 — RBAC Matrix

Tương tự Permission Matrix nhưng theo nhóm quyền hệ thống.

#### URD-II.5 — Sequence Diagram

Khi flow trình tự thay đổi:
- Xác định đoạn sequence bị ảnh hưởng
- Thêm/sửa message giữa các lifeline đúng chỗ
- Giữ nguyên các đoạn sequence không liên quan

#### URD-III — Use Case

Khi UC bị ảnh hưởng:

```
Các field cần kiểm tra và patch:
- Actor: thêm/bớt actor nếu có thay đổi phân quyền
- Precondition: cập nhật nếu điều kiện đầu vào thay đổi
- Main Flow: sửa đúng bước bị ảnh hưởng
- Alternative Flow: thêm/sửa nhánh rẽ mới
- Postcondition: cập nhật nếu kết quả thay đổi
```

Không sửa các field không bị ảnh hưởng bởi REQ thay đổi.

#### URD-IV — Screen Specification

Khi màn hình thay đổi:
- Tìm màn hình theo WF ID hoặc tên màn hình
- Sửa đúng component/row bị ảnh hưởng trong bảng 6 cột
- Thêm row mới cho component mới, không thêm cột mới vào bảng
- Cập nhật Validation Rules nếu có thay đổi

#### Section C — Phi chức năng

Chỉ patch khi thay đổi liên quan đến performance, bảo mật, hoặc constraint hệ thống.

---

### 2. Wireframe Patch

**File**: `.claude/output/[tên_dự_án]/wireframe/wireframe-v[N].md`

#### Thêm màn hình mới

```markdown
## [WF-NNN-NNN] [Tên màn hình mới]

**Trigger**: [Từ màn hình nào / hành động nào]
**Actor**: [Ai sử dụng màn hình này]
**REQ nguồn**: [REQ-XXX-NNN]

### Layout

[Mô tả layout text-based]

### Components

| Component | Loại | Mô tả | State |
|---|---|---|---|
| ... | ... | ... | ... |

### Interaction

- [Hành động] → [Kết quả]
```

#### Sửa màn hình hiện có

- Tìm heading màn hình theo WF ID
- Chỉ sửa phần bị ảnh hưởng: thêm/bớt component, thay đổi interaction
- Cập nhật REQ nguồn nếu có REQ mới link vào màn hình này

---

### 3. Story/AC Patch

**File**: `.claude/output/[tên_dự_án]/backlog/backlog-v[N].md`

Nếu dự án đang dùng file `backlog.md` (legacy không có version number):
1. Đổi tên thành `backlog-v1.md` trước khi patch
2. Sau khi patch, lưu thành `backlog-v2.md`
3. Không tự tìm hay patch file `backlog.md` trực tiếp — phải version hóa trước

#### Sửa Acceptance Criteria

Khi rule nghiệp vụ thay đổi:
- Tìm Story liên quan theo STORY ID
- Sửa/thêm AC bị ảnh hưởng
- Đảm bảo AC vẫn theo format: "Given [context] When [action] Then [expected result]"

```
OLD AC-002-001-003:
  Given Supervisor đã login
  When Supervisor duyệt đơn
  Then hệ thống ghi nhận và chuyển trạng thái "Đã duyệt"

NEW AC-002-001-003:
  Given Supervisor đã login và đơn có giá trị ≤ 500 triệu
  When Supervisor duyệt đơn
  Then hệ thống ghi nhận và chuyển trạng thái "Đã duyệt"

ADD AC-002-001-004 (mới):
  Given Supervisor đã login và đơn có giá trị > 500 triệu
  When Supervisor cố gắng duyệt đơn
  Then hệ thống từ chối và hiển thị "Đơn vượt hạn mức, cần Director duyệt"
```

#### Thêm Story mới

Khi REQ ADD kéo theo User Story mới:

```markdown
### [STORY-NNN-NNN] [Tên story]

**Epic**: [EPIC-NNN]
**REQ nguồn**: [REQ-XXX-NNN]
**As a** [actor], **I want to** [hành động] **so that** [lợi ích]

**Acceptance Criteria**:
- AC-NNN-NNN-001: Given... When... Then...
- AC-NNN-NNN-002: Given... When... Then...
```

---

## Quy trình thực hiện patch

```
Bước 1: Đọc Impact Report — lấy danh sách artifact + thứ tự
Bước 2: Với từng artifact theo thứ tự ưu tiên:
   a. Đọc file artifact hiện tại
   b. Xác định vị trí cần patch (section, bảng, dòng)
   c. Thực hiện patch minimal
   d. Verify: không sót, không thừa, không conflict với phần còn lại
Bước 3: Tăng version file URD/SRS, cập nhật version history
Bước 4: Cập nhật Traceability Map (luôn làm cuối cùng)
Bước 5: Output Patch Summary
```

---

## Patch Summary — output bắt buộc

Sau khi patch xong, tạo báo cáo ngắn:

```markdown
## Patch Summary — [YYYY-MM-DD]

**Change Set**: [Tên/ref của Change Set]
**Tổng artifact đã patch**: N

| # | Artifact | File | Section/Vị trí | Thay đổi | Status |
|---|---|---|---|---|---|
| 1 | URD-II.3 | urd-srs-v2.md → v3.md | Section II.3, dòng "Duyệt đơn" | Tách thành 2 dòng theo giá trị đơn | DONE |
| 2 | UC-003 | urd-srs-v3.md | Section III, UC-003 Main Flow bước 3 | Thêm nhánh rẽ Director | DONE |
| 3 | WF-002-001 | wireframe-v2.md → v3.md | WF-002-001 Interaction | Thêm route escalate to Director | DONE |
| 4 | STORY-002-001 | backlog-v1.md | AC-002-001-003 + thêm AC-002-001-004 | Cập nhật điều kiện + thêm case mới | DONE |
| 5 | Traceability Map | traceability-map.md | Bảng chính + Reverse Index | Cập nhật REQ-CDP-003 links | DONE |

**Version bump**: urd-srs-v2.md → urd-srs-v3.md, wireframe-v2.md → v3.md

**Cần BA review**:
- [Nếu có điểm không chắc trong quá trình patch]
```

---

## Rules

- KHÔNG patch nếu Impact Report chưa có `ba_confirmed: true`
- KHÔNG viết lại toàn bộ tài liệu chỉ vì 1 section thay đổi
- Version bump: chỉ bump version khi URD/SRS hoặc Wireframe thực sự bị sửa — không bump nếu chỉ sửa backlog
- Luôn giữ file cũ — không xóa `urd-srs-v2.md` khi tạo `urd-srs-v3.md`
- Nếu patch tạo ra conflict với phần khác trong cùng file → dừng, báo BA, không tự xử lý

## Failure Cases

- Patch sai section (nhầm II.3 thành II.4) → sai tài liệu, khó phát hiện
- Quên cập nhật version history → version hygiene fail khi postcheck chạy
- Patch xong không cập nhật Traceability Map → map lệch với tài liệu thực tế
- Xóa file cũ sau khi tạo file mới → mất lịch sử version
- Tự xử lý conflict thay vì báo BA → có thể sửa sai logic nghiệp vụ
