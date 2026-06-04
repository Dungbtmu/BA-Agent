# Skill: Review Report Fix Handoff

Skill này hướng dẫn cách parse bảng handoff trong review report và convert thành patch plan có thể thực thi.

---

## Schema finding hợp lệ

Skill này chấp nhận **2 dạng schema** từ Agent Review Agent:

### Schema 8 cột (đầy đủ)

```
Finding ID:          F-001
Severity:            CRITICAL / MAJOR / MINOR
Target Agent System: ba-agent
Target File:         .claude/agents/[tên-agent].md
Target Section:      [Tên section hoặc dòng tham chiếu]
Action Type:         UPDATE / ADD / DELETE / MOVE / ASK_CONFIRM
Fix Instruction:     [Hướng dẫn cụ thể cần làm]
Acceptance Criteria: [Điều kiện để xác nhận đã sửa đúng]
```

### Schema 7 cột (không có cột `Severity`)

```
Finding ID:          CRITICAL-001 / MAJOR-001 / MINOR-001
Target Agent System: ba-agent
Target File:         .claude/agents/[tên-agent].md
Target Section:      [Tên section hoặc dòng tham chiếu]
Action Type:         UPDATE / ADD / DELETE / MOVE / ASK_CONFIRM
Fix Instruction:     [Hướng dẫn cụ thể cần làm]
Acceptance Criteria: [Điều kiện để xác nhận đã sửa đúng]
```

Khi gặp schema 7 cột, **derive severity từ prefix của `Finding ID`**:
- Prefix `CRITICAL-` → Severity = CRITICAL
- Prefix `MAJOR-` → Severity = MAJOR
- Prefix `MINOR-` → Severity = MINOR
- Finding ID không có prefix rõ ràng → đánh dấu ASK_CONFIRM, hỏi BA xác nhận severity trước khi sort.

**Thứ tự sort sau khi derive:** CRITICAL → MAJOR → MINOR → theo Finding ID trong cùng severity.

**Trường bắt buộc để APPLY (áp dụng cho cả 2 dạng):** `Target File` + `Action Type` + `Fix Instruction` + `Acceptance Criteria`

Finding thiếu bất kỳ trường bắt buộc nào → SKIPPED, báo lý do.

Finding có `Target Agent System ≠ ba-agent` → bỏ qua hoàn toàn, không báo SKIPPED.

---

## Xử lý từng Action Type

| Action Type | Mô tả | Cần confirm? |
|---|---|---|
| `UPDATE` | Sửa nội dung hiện có tại Target Section | Không — thực thi ngay |
| `ADD` | Thêm section hoặc nội dung mới vào Target File | Không — thực thi ngay |
| `DELETE` | Xóa nội dung tại Target Section | **Luôn hỏi user trước** |
| `MOVE` | Di chuyển nội dung sang file/section khác | **Luôn hỏi user trước** |
| `ASK_CONFIRM` | Finding cần judgement call — không tự quyết | **Luôn hỏi user trước** |

`DELETE` và `MOVE` tự động được xử lý như `ASK_CONFIRM` — không bao giờ thực thi trực tiếp.

---

## Quy trình convert finding → patch plan

### Bước 1 — Đọc và lọc

Đọc toàn bộ section `## Fix Handoff For Target Agent`. Với mỗi finding:

1. Kiểm tra `Target Agent System = ba-agent` — nếu không, bỏ qua
2. Kiểm tra đủ 4 trường bắt buộc
3. Kiểm tra `Target File` tồn tại trong hệ thống
4. Phân loại:
   - **APPLY** — đủ điều kiện, Action Type là UPDATE hoặc ADD
   - **ASK_CONFIRM** — Action Type là DELETE, MOVE, hoặc ASK_CONFIRM; hoặc thiếu trường bắt buộc
   - **SKIPPED** — `Target File` không tồn tại, hoặc finding mơ hồ không thể thực thi

### Bước 2 — Sắp xếp thứ tự apply

1. CRITICAL trước → MAJOR → MINOR
2. Trong cùng severity: theo thứ tự Finding ID
3. Nếu 2 finding cùng Target File → apply lần lượt, kiểm tra không conflict trước

### Bước 3 — Thực thi

Với mỗi finding **APPLY**:
- `UPDATE`: tìm đúng Target Section → sửa theo Fix Instruction
- `ADD`: tìm đúng vị trí được chỉ định → chèn nội dung mới

Sau mỗi finding: kiểm tra Acceptance Criteria — nếu chưa đạt → đánh dấu NEEDS_REVIEW trong Fix Summary.

Nếu không tìm được Target Section trong file → đánh dấu SKIPPED (file đã thay đổi so với lúc review), không tự đoán vị trí.

Với mỗi finding **ASK_CONFIRM**:
- Trình bày finding cho user: Finding ID, nội dung Fix Instruction, lý do cần xác nhận
- Chờ user quyết định: apply / skip / sửa instruction
- Không tự thực thi khi chưa có quyết định

### Bước 4 — Tạo Fix Summary

Sau khi xử lý xong toàn bộ finding, output Fix Summary theo format trong agent.

---

## Nguyên tắc

- **Không review lại** — không đánh giá finding đúng hay sai, chỉ thực thi
- **Không tranh luận** — finding mơ hồ → SKIPPED, không tự suy diễn
- **Không sửa ngoài phạm vi** — chỉ sửa đúng Target File và Target Section được chỉ định
- **Không tự thêm finding** — chỉ xử lý finding có trong bảng handoff
- **Atomic per finding** — một finding fail không làm dừng các finding khác
- **Acceptance Criteria là tiêu chuẩn nghiệm thu** — phải kiểm tra sau mỗi fix, ghi kết quả vào Fix Summary
