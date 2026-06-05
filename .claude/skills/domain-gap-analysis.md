---
name: domain_gap_analysis
description: So sánh Domain Brief (mô hình điển hình của domain) với yêu cầu thực tế từ client — tìm ra điểm khác biệt, mâu thuẫn, và sub-domain cần đào sâu trước khi clarify
tools: []
---

# Skill: Domain Gap Analysis

## Mục tiêu

Khi BA đã có Domain Brief từ `ba-research-agent` VÀ có input từ client, skill này giúp xác định:
- Client này **khác gì** so với mô hình điển hình của domain?
- Điểm nào trong Domain Brief **không áp dụng được** cho client này?
- Điểm nào **client chưa đề cập** nhưng domain điển hình thường có — cần hỏi?
- Sub-domain nào cần **đào sâu thêm** dựa trên đặc thù của client?

Output là danh sách gap có độ ưu tiên, dùng trực tiếp để định hướng câu hỏi clarify.

---

## Input bắt buộc

- Domain Brief (file `.claude/output/[tên_dự_án]/research/domain-brief.md`)
- Input từ client (mô tả miệng, PRD sơ bộ, ghi chú, hoặc kết quả buổi họp đầu)

Nếu thiếu một trong hai → không chạy skill này, dùng `requirement-clarification.md` thay thế.

---

## Thinking Pattern

1. Domain Brief mô tả mô hình **điển hình** — client này có theo mô hình đó không, hay có biến thể đặc thù?
2. Actor nào trong domain điển hình **không xuất hiện** trong mô tả của client — bị bỏ qua hay thực sự không có?
3. Quy trình nào client đề cập **mâu thuẫn hoặc khác** với quy trình phổ biến — đây là đặc thù hay misunderstanding?
4. Pain point nào trong domain điển hình client **chưa nhắc đến** — họ đã giải quyết rồi, hay chưa nhận ra?
5. Có sub-domain nào cần **search thêm** vì Domain Brief chưa cover?

---

## Execution

### Bước 1 — Đọc và trích xuất từ Domain Brief

Từ Domain Brief, lập danh sách kiểm tra:

| Hạng mục | Nội dung điển hình theo Domain Brief |
|---|---|
| Actor chính | [list] |
| Quy trình cốt lõi | [list] |
| Pain point phổ biến | [list] |
| Tích hợp thường có | [list] |
| Constraint đặc thù | [list] |

### Bước 2 — Đọc input từ client và map

Với mỗi hạng mục ở trên, đánh dấu:
- **MATCH** — client đề cập và phù hợp với điển hình
- **VARIANT** — client đề cập nhưng khác với điển hình → cần hiểu tại sao
- **MISSING** — không thấy trong input client, nhưng điển hình thường có → cần hỏi
- **NEW** — client đề cập điều gì Domain Brief không có → cần research thêm hoặc là đặc thù riêng

### Bước 3 — Phân loại gap theo mức độ ảnh hưởng

**GAP CRITICAL** — ảnh hưởng trực tiếp đến scope, actor chính, hoặc quy trình cốt lõi:
- Ví dụ: Domain Brief nói có 3 actor nhưng client chỉ đề cập 1 — 2 actor kia ở đâu?
- Ví dụ: Quy trình điển hình có bước phê duyệt nhưng client không nhắc — tự động hóa hay bỏ hẳn?

**GAP MAJOR** — ảnh hưởng đến tính năng hoặc tích hợp:
- Ví dụ: Domain điển hình thường tích hợp hệ thống X nhưng client không đề cập
- Ví dụ: Pain point phổ biến của domain mà client chưa nhắc đến solution

**GAP MINOR** — cần xác nhận nhưng không block:
- Ví dụ: Thuật ngữ client dùng khác với thuật ngữ chuẩn trong domain — cùng khái niệm hay khác?
- Ví dụ: Edge case phổ biến chưa được đề cập

### Bước 4 — Tổng hợp output

```markdown
## Domain Gap Analysis

**Domain:** [tên domain]
**Client:** [tên client / dự án]
**Domain Brief:** [đường dẫn file]

---

### CRITICAL Gaps (phải hỏi trước khi clarify)

| Gap | Điển hình | Client đề cập | Câu hỏi cần đặt |
|---|---|---|---|
| [Gap 1] | [...] | [...] | [Câu hỏi cụ thể] |

### MAJOR Gaps (ảnh hưởng tính năng/tích hợp)

| Gap | Điển hình | Client đề cập | Câu hỏi cần đặt |
|---|---|---|---|
| [Gap 2] | [...] | [...] | [Câu hỏi cụ thể] |

### MINOR Gaps (cần xác nhận, không block)

| Gap | Ghi chú | Câu hỏi cần đặt |
|---|---|---|
| [Gap 3] | [...] | [Câu hỏi cụ thể] |

---

### Sub-domain cần research thêm (nếu có)

- [Sub-domain X] — lý do: client đề cập [...] nhưng Domain Brief chưa cover
```

---

## Rules

- **Không phán xét** — VARIANT không có nghĩa là sai; client có thể có đặc thù hợp lý
- **Câu hỏi phải cụ thể** — không ghi "cần xác nhận actor" mà ghi "Ai phê duyệt đơn hàng trong quy trình của bạn — có phải là [X] không?"
- **Gap CRITICAL phải có trong câu hỏi clarify** — không được bỏ qua dù có vẻ obvious
- **Giới hạn output** — tối đa 3 CRITICAL, 5 MAJOR, 5 MINOR; nếu nhiều hơn thì gộp các gap liên quan lại
- **Không search thêm trong bước này** — nếu cần research thêm thì note vào "Sub-domain cần research thêm" và báo BA, không tự chạy search

---

## Failure Cases

- So sánh máy móc từng điểm mà không suy nghĩ ngữ cảnh → sinh ra gap giả (false gap)
- Câu hỏi quá generic ("Bạn có dùng X không?") → không khai thác được thông tin có giá trị
- Bỏ qua MISSING — chỉ focus vào VARIANT → miss những gì client chưa nhận ra là cần thiết
- Output quá dài → BA không đọc hết trước buổi họp clarify
