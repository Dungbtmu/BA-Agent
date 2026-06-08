---
name: change_handler
description: Nhận và chuẩn hóa trigger thay đổi requirement — từ BA mô tả free text hoặc file PO mới; output là danh sách REQ thay đổi chuẩn hóa cho impact-analysis
tools: []
---

# Skill: Change Handler

## Mục tiêu

Là cổng vào đầu tiên khi có thay đổi requirement. Nhận trigger từ bất kỳ nguồn nào, chuẩn hóa thành danh sách thay đổi có cấu trúc, rồi chuyển sang `impact-analysis` để tính impact.

---

## Hai loại trigger

### Trigger A — BA mô tả trực tiếp

BA nhập mô tả thay đổi dưới dạng tự nhiên:

```
"PO vừa báo: thay đổi rule phân quyền duyệt đơn, Supervisor không còn
duyệt được đơn > 500 triệu nữa"

"Bổ sung thêm chức năng export báo cáo PDF cho module Quản lý đơn hàng"

"Bỏ màn hình xác nhận email — PO nói không cần thiết"
```

### Trigger B — File PO mới/updated

PO gửi file tài liệu mới (PRD update, email, ghi chú) vào:
```
.claude/input/change-requests/[tên-file].[md|txt|pdf]
```

---

## Quy trình xử lý Trigger A — Free text

### Bước 1 — Nhận dạng loại thay đổi

Từ mô tả của BA, xác định:

| Dấu hiệu | Loại |
|---|---|
| "thay đổi", "sửa", "cập nhật", "không còn", "thay thế" | MODIFY |
| "thêm", "bổ sung", "mới", "cần có thêm" | ADD |
| "bỏ", "xóa", "không cần", "loại bỏ" | REMOVE |

### Bước 2 — Tìm REQ ID trong Traceability Map

Đọc `.claude/output/[tên_dự_án]/traceability-map.md`:

1. Tìm REQ có mô tả khớp với nội dung BA đề cập
2. Ghi `match_status` cho từng thay đổi:
   - `MATCHED` — tìm được REQ ID rõ ràng trong Traceability Map
   - `NEW_CONFIRMED` — Change Type = ADD và BA đã xác nhận đây là requirement mới
   - `AMBIGUOUS` — không tìm được REQ ID và chưa được BA xác nhận
3. **Chỉ tạo REQ ID mới khi Change Type = ADD và BA đã xác nhận** — không tự tạo REQ mới cho MODIFY hoặc REMOVE không match
4. Nếu MODIFY hoặc REMOVE không match được REQ ID → đánh dấu `AMBIGUOUS`, hỏi BA xác nhận trước khi tiếp tục
5. SYNC không được chạy sang bước impact-analysis khi còn bất kỳ thay đổi nào có `match_status = AMBIGUOUS`

### Bước 3 — Clarify nếu mơ hồ

Nếu mô tả BA không đủ rõ để xác định impact, hỏi lại đúng 1-2 câu ngắn:

```
"Thay đổi phân quyền Supervisor: rule mới áp dụng cho loại đơn nào?
(tất cả đơn hay chỉ đơn thuộc module Mua sắm?)"
```

Không hỏi quá 2 câu — nếu vẫn chưa rõ, ghi assumption và tiếp tục.

### Bước 4 — Output Change Set

```markdown
## Change Set — [Ngày]

| # | REQ ID | Change Type | Old Value | New Value | Confidence | match_status |
|---|---|---|---|---|---|---|
| 1 | REQ-CDP-003 | MODIFY | Supervisor duyệt đơn không giới hạn | Supervisor chỉ duyệt đơn ≤ 500M | HIGH | MATCHED |
| 2 | REQ-CDP-015 | ADD | N/A | Export báo cáo PDF module Quản lý đơn | MEDIUM | NEW_CONFIRMED |

**Assumption** (nếu có):
- [ASS-01] Giới hạn 500M áp dụng cho tất cả loại đơn, không phân biệt module
```

---

## Quy trình xử lý Trigger B — File mới từ PO

### Bước 1 — Đọc file mới

Đọc file trong `.claude/input/change-requests/`.

Xác định loại tài liệu:
- **PRD update**: tìm phần "Changes" hoặc "Version History" để lấy delta
- **Email/ghi chú thô**: parse toàn bộ nội dung
- **Bảng requirement mới**: so sánh trực tiếp với Traceability Map

### Bước 2 — So sánh với tài liệu hiện tại

Đọc Traceability Map và file URD/SRS hiện tại.

So sánh từng requirement trong file mới với requirement hiện tại:

```
Với mỗi requirement trong file mới:
  1. Tìm REQ ID tương ứng trong Traceability Map (match theo nội dung)
  2. Nếu match → so sánh nội dung:
     - Giống nhau → UNCHANGED (bỏ qua)
     - Khác nhau → MODIFY (ghi lại old/new value)
  3. Nếu không match → ADD (requirement mới)

Với mỗi REQ trong Traceability Map không xuất hiện trong file mới:
  → REMOVE (hỏi BA confirm trước khi đánh dấu REMOVE)
```

### Bước 3 — Highlight delta

Tóm tắt những gì thay đổi để BA review nhanh trước khi chạy impact analysis:

```markdown
## Delta Summary — [Tên file] vs version hiện tại

**MODIFY** (N requirement):
- REQ-CDP-003: [Mô tả thay đổi ngắn]
- REQ-CDP-007: [Mô tả thay đổi ngắn]

**ADD** (N requirement mới):
- REQ-CDP-015 (mới): [Mô tả]

**REMOVE** (N requirement có thể bị xóa — cần xác nhận):
- REQ-CDP-011: không xuất hiện trong file mới — có xóa không?

**UNCHANGED**: N requirement không thay đổi (bỏ qua)
```

### Bước 4 — Chờ BA confirm REMOVE, sau đó output Change Set

Không tự đánh REMOVE mà không có BA confirm — REMOVE một requirement sai có thể xóa artifact còn cần dùng.

---

## Checkpoint và luồng phê duyệt

```
Trigger nhận được
       ↓
Change Handler parse + clarify
       ↓
Delta Summary → BA review (nhanh, < 2 phút)
       ↓
BA confirm → Change Set chính thức
       ↓
Chuyển sang impact-analysis
```

BA review Delta Summary trước khi chạy impact analysis — đây là checkpoint duy nhất trước khi orchestrator bắt đầu patch artifact.

---

## Output chuẩn — Change Set

File lưu tạm tại: `.claude/input/change-requests/change-set-[YYYY-MM-DD].md`

```markdown
---
project: [tên_dự_án]
date: [YYYY-MM-DD]
trigger_source: [BA_INPUT | PO_FILE]
trigger_ref: [mô tả ngắn hoặc tên file]
ba_confirmed: true
---

## Change Set

| # | REQ ID | Change Type | Old Value | New Value | Confidence | match_status | Note |
|---|---|---|---|---|---|---|---|
| 1 | REQ-XXX-003 | MODIFY | ... | ... | HIGH | MATCHED | |
| 2 | REQ-XXX-015 | ADD | N/A | ... | MEDIUM | NEW_CONFIRMED | Assumption: ... |
| 3 | REQ-XXX-011 | REMOVE | ... | N/A | HIGH | MATCHED | BA confirmed |
```

---

## Rules

- Luôn show Delta Summary và chờ BA confirm trước khi output Change Set chính thức
- REMOVE phải luôn có BA confirm — không tự đánh dấu
- Nếu file PO mới có nhiều thay đổi (> 10 REQ) → chunk thành nhiều Change Set nhỏ theo nhóm chức năng
- Confidence mức MEDIUM → ghi rõ assumption để BA biết điểm không chắc
- Change Set phải có `ba_confirmed: true` trước khi chuyển sang impact-analysis

## Failure Cases

- Parse sai loại thay đổi (nhầm MODIFY thành ADD) → Traceability Map tạo REQ trùng
- Không hỏi clarify khi mơ hồ → impact analysis tính sai → patch sai artifact
- Tự đánh REMOVE mà không confirm → xóa artifact đang cần dùng ở nơi khác
- Không lưu Change Set ra file → orchestrator không có input để retry nếu bị gián đoạn
