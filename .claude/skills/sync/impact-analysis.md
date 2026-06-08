---
name: impact_analysis
description: Phân tích impact khi requirement thay đổi — tra cứu Traceability Map, xác định artifact bị ảnh hưởng, phân loại mức độ, output Impact Report cho orchestrator
tools: []
---

# Skill: Impact Analysis

## Mục tiêu

Khi có requirement thay đổi (từ bất kỳ trigger nào), phân tích chính xác:
- Artifact nào bị ảnh hưởng (URD section, Wireframe, Use Case, Story/AC)
- Mức độ ảnh hưởng (CRITICAL / MAJOR / MINOR)
- Thứ tự cập nhật tối ưu để tránh conflict

Output là **Impact Report** — input trực tiếp cho `artifact-patch` skill và orchestrator.

---

## Input

Một trong hai dạng:

### Dạng 1 — BA mô tả thay đổi (free text)

```
"Thay đổi rule phân quyền: Supervisor không còn được duyệt đơn > 500 triệu,
phải chuyển lên Director"
```

### Dạng 2 — Diff từ file PO mới

```
[File cũ]: REQ-CDP-003 — Supervisor duyệt đơn không giới hạn
[File mới]: REQ-CDP-003 — Supervisor chỉ duyệt đơn ≤ 500 triệu; đơn > 500 triệu → Director
```

---

## Quy trình phân tích

### Bước 1 — Parse thay đổi

Từ input (free text hoặc diff), extract:

```
{
  "req_ids": ["REQ-XXX-NNN", ...],       // REQ nào thay đổi (hoặc "NEW" nếu thêm mới)
  "change_type": "MODIFY | ADD | REMOVE",
  "change_summary": "Mô tả ngắn gọn thay đổi",
  "old_value": "...",                     // nếu là MODIFY
  "new_value": "...",                     // nếu là MODIFY hoặc ADD
  "match_status": "MATCHED | NEW_CONFIRMED | AMBIGUOUS"  // từ Change Set của change-handler
}
```

Nếu BA mô tả free text mà không mention REQ ID: tra cứu Traceability Map để tìm REQ ID tương ứng. Nếu không tìm được → hỏi BA confirm trước khi tiếp tục.

**Quy tắc bắt buộc:** Nếu Change Set còn bất kỳ thay đổi nào có `match_status = AMBIGUOUS` → **dừng ngay, hỏi BA xác nhận REQ ID** trước khi phân tích impact. Không được tự suy luận và tiếp tục khi còn AMBIGUOUS — sẽ dẫn đến patch sai artifact.

### Bước 2 — Tra cứu Traceability Map

Đọc file `.claude/output/[tên_dự_án]/traceability-map.md`.

Với mỗi REQ ID bị thay đổi:
1. Tìm dòng trong bảng chính → lấy danh sách UC, WF, URD Section, Story bị ảnh hưởng
2. Tra Reverse Index → xác nhận không bỏ sót artifact nào link đến REQ này

### Bước 3 — Phân loại impact từng artifact

#### Mức CRITICAL — phải cập nhật trước khi tiếp tục

| Điều kiện | Artifact |
|---|---|
| Rule nghiệp vụ cốt lõi thay đổi (phân quyền, luồng chính, điều kiện quyết định) | URD-II.1 (Workflow), URD-II.3 (Permission Matrix), URD-II.4 (RBAC), UC liên quan |
| Actor mới hoặc actor bị xóa | Toàn bộ: Workflow, Permission Matrix, RBAC, UC, Wireframe có actor đó |
| Thay đổi scope (thêm/bớt chức năng chính) | URD-I.2, Function Tree, Permission Matrix, tất cả UC + WF của chức năng đó |

#### Mức MAJOR — nên cập nhật, ảnh hưởng đáng kể

| Điều kiện | Artifact |
|---|---|
| Thay đổi flow (thêm/bớt bước trong luồng chính) | UC Main Flow, Sequence Diagram, Wireframe màn hình liên quan |
| Thay đổi dữ liệu đầu vào/đầu ra của một chức năng | UC, Wireframe (form fields), Story/AC |
| Thay đổi điều kiện validation | UC Alternative Flow, Wireframe (error states), AC |

#### Mức MINOR — cập nhật nhưng không block

| Điều kiện | Artifact |
|---|---|
| Thay đổi tên hiển thị, label UI | Wireframe, Section IV URD |
| Thay đổi mô tả/ghi chú không ảnh hưởng logic | URD section tương ứng, Story description |
| Thêm/bớt thông tin trong section phi chức năng | Section C URD |

### Bước 4 — Xác định thứ tự cập nhật

Artifact phụ thuộc nhau — phải cập nhật đúng thứ tự:

```
1. URD-I (Giới thiệu, Phạm vi)      — nếu scope thay đổi
2. URD-II.1 (Workflow)               — luồng tổng thể
3. URD-II.2 (Function Tree)          — cây chức năng
4. URD-II.3 + II.4 (Permission/RBAC) — phân quyền
5. URD-II.5 (Sequence)               — chi tiết trình tự
6. URD-III (Use Cases)               — đặc tả từng UC
7. Wireframe                         — giao diện
8. URD-IV (Screen Spec)              — đặc tả màn hình
9. Story/AC                          — backlog
10. Traceability Map                 — cập nhật map sau cùng
```

Chỉ cập nhật step N khi step N-1 đã xong — tránh conflict.

---

## Output — Impact Report

```markdown
## Impact Report

**Trigger**: [Mô tả thay đổi]
**Ngày phân tích**: [YYYY-MM-DD]
**REQ bị ảnh hưởng**: [REQ-XXX-NNN, ...]

---

### Tóm tắt

| Mức | Số artifact | Cần BA confirm? |
|---|---|---|
| CRITICAL | N | Không — tự động patch |
| MAJOR | N | Có — báo BA trước khi patch |
| MINOR | N | Không — tự động patch |

---

### Danh sách artifact cần cập nhật

| # | Artifact ID | Loại | Mức | File | Mô tả thay đổi cần làm |
|---|---|---|---|---|---|
| 1 | URD-II.3 | Permission Matrix | CRITICAL | urd-srs-v[N].md | Xóa quyền duyệt đơn > 500M của Supervisor |
| 2 | URD-II.4 | RBAC Matrix | CRITICAL | urd-srs-v[N].md | Cập nhật dòng Supervisor, thêm dòng Director |
| 3 | UC-003 | Use Case | CRITICAL | urd-srs-v[N].md | Cập nhật Main Flow bước 3: thêm nhánh rẽ theo giá trị đơn |
| 4 | WF-002-001 | Wireframe | MAJOR | wireframe-v[N].md | Thêm màn hình confirm escalate to Director |
| 5 | STORY-002-001 | User Story | MAJOR | backlog-v[N].md | Cập nhật AC: thêm case đơn > 500M |

---

### Thứ tự thực hiện

```
Bước 1 (CRITICAL): URD-II.3 → URD-II.4 → UC-003
Bước 2 (MAJOR):    WF-002-001 → URD-IV (nếu có) → STORY-002-001
Bước 3 (MINOR):    [nếu có]
Bước 4 (LUÔN):     Cập nhật Traceability Map
```

---

### Checkpoint cần BA confirm

[Chỉ liệt kê nếu có MAJOR trở lên với ambiguity cao]

- **[?-01]** UC-003 Main Flow bước 3: khi đơn = đúng 500M thì xếp vào nhóm ≤500M hay >500M?
  → Cần BA xác nhận trước khi patch UC-003

---

### Artifact KHÔNG bị ảnh hưởng

UC-001, UC-002, WF-001-001, STORY-001-001 — không link đến REQ-XXX-003
```

---

## Rules

- Không tự suy luận impact khi không có Traceability Map — yêu cầu BA chạy `traceability-map` skill trước
- CRITICAL impact → orchestrator tự động patch, không cần confirm BA
- MAJOR impact với ambiguity → dừng, hỏi BA, sau đó patch
- MINOR impact → tự động patch không cần hỏi
- Luôn cập nhật Traceability Map CUỐI CÙNG — sau khi tất cả artifact đã được patch

## Failure Cases

- Parse sai REQ ID từ free text → tra cứu sai map → patch sai artifact
- Quên kiểm tra Reverse Index → bỏ sót artifact phụ thuộc nhiều REQ
- Đảo thứ tự cập nhật (ví dụ: patch UC trước khi patch Workflow) → UC tham chiếu flow chưa đúng → conflict
- Đánh mức MINOR cho thay đổi phân quyền → artifact quan trọng bị bỏ qua
