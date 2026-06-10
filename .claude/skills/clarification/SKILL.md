---
name: requirement_clarification
description: Orchestrator của nhóm Clarification — tự phân tích context, quyết định gọi sub-skill nào (input-analysis, as-is-analysis, domain-gap-analysis, problem-framing), tổng hợp thành danh sách câu hỏi clarify không trùng lặp
tools: []
---

# Skill: Requirement Clarification (Orchestrator)

## Mục tiêu

Entry point duy nhất cho toàn bộ nhóm Clarification. Nhận input ở bất kỳ dạng nào, tự phân tích context, gọi đúng sub-skill theo thứ tự, tổng hợp thành danh sách câu hỏi clarify có độ ưu tiên — không trùng, không thừa.

**`ba-clarification-agent` chỉ cần đọc skill này — không cần đọc 4 sub-skill riêng lẻ.**

> Quy tắc chung (IT-BA framing, no-re-ask, assumption, approval gate, OQ format): xem `ba-conventions.md`. Skill này không lặp lại.

---

## Sub-skills và khi nào gọi

| Sub-skill | Gọi khi | Không gọi khi |
|---|---|---|
| `references/input-analysis.md` | Input là tài liệu có sẵn (PRD, email, ghi chú, file cũ) | Input là mô tả miệng thuần túy |
| `references/as-is-analysis.md` | BA cung cấp hiện trạng thực tế (hệ thống cũ, quy trình đang chạy) | Không có thông tin hiện trạng |
| `references/domain-gap-analysis.md` | Có Domain Brief từ `ba-research-agent` | Không có Domain Brief |
| `references/problem-framing.md` | Input mơ hồ, mô tả solution thay vì problem, hoặc thiếu business objective | Input đã có problem statement rõ ràng |

---

## Quy trình thực hiện

### Bước 1 — Nhận dạng context

Đọc toàn bộ input, xác định:

```
□ Loại input:
  □ Tài liệu có sẵn (PRD, email, ghi chú...)  → gọi input-analysis
  □ Mô tả miệng / ý tưởng thô                 → không gọi input-analysis
  □ Kết hợp cả hai

□ Context bổ sung có hay không:
  □ Có hiện trạng thực tế từ BA               → gọi as-is-analysis
  □ Có Domain Brief                            → gọi domain-gap-analysis
  □ Input mơ hồ hoặc là solution-framed        → gọi problem-framing
```

### Bước 2 — Chạy sub-skills theo thứ tự

Thứ tự bắt buộc khi có nhiều sub-skill:

```
1. input-analysis      (nếu có tài liệu)      → trích xuất requirement thô
2. as-is-analysis      (nếu có hiện trạng)    → xác định đã biết gì / còn mở gì
3. domain-gap-analysis (nếu có Domain Brief)  → gap domain chuẩn vs yêu cầu client
4. problem-framing     (nếu input mơ hồ)      → chuẩn hóa bài toán
```

Lý do thứ tự: mỗi bước xử lý input cho bước sau. `as-is-analysis` cần biết requirement thô trước khi so với hiện trạng. `domain-gap-analysis` cần biết cả requirement lẫn hiện trạng trước khi so với domain chuẩn. `problem-framing` cần đủ context trước khi frame bài toán.

Nếu chỉ có 1 sub-skill áp dụng → chạy sub-skill đó, bỏ qua phần còn lại.

### Bước 3 — Tổng hợp câu hỏi clarify

Sau khi chạy xong các sub-skill có liên quan, tổng hợp câu hỏi từ tất cả output:

**Loại bỏ trùng lặp:**
- Nếu `as-is-analysis` và `domain-gap-analysis` cùng raise một vấn đề → gộp thành 1 câu hỏi duy nhất, ghi rõ nguồn
- Nếu `as-is-analysis` đã trả lời một câu hỏi từ `domain-gap-analysis` → xóa câu hỏi đó khỏi danh sách

**Phân loại lại:**
- CRITICAL: thiếu thông tin này thì không thể thiết kế được (actor chưa xác định, trigger chính chưa rõ, scope chưa có)
- MAJOR: có thể tiếp tục nhưng assumption sai thì ảnh hưởng lớn
- MINOR: có thể assume và ghi chú, không block workflow

**Giới hạn:**
- Tối đa 5 câu CRITICAL — nếu nhiều hơn, gộp các câu liên quan theo chủ đề
- Không hỏi những gì đã rõ trong As-Is Summary (phần "Đã biết")
- Không hỏi câu kỹ thuật (API, DB, infrastructure) — đó là phạm vi SA/Dev

### Bước 4 — Ghi assumption

Với mỗi câu hỏi MINOR hoặc MAJOR có thể tự assume → ghi assumption thay vì hỏi:

```
Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
```

---

## Output Format

```markdown
## Clarification Output

### Context phát hiện
- Loại input: [Tài liệu / Mô tả miệng / Kết hợp]
- Sub-skills đã chạy: [input-analysis / as-is-analysis / domain-gap-analysis / problem-framing]
- Đã biết từ hiện trạng: [Tóm tắt nếu có as-is-analysis]

---

### Problem Statement
*(Chỉ có nếu đã chạy problem-framing)*
[Nhóm người dùng] đang gặp [vấn đề cụ thể] khi [ngữ cảnh],
dẫn đến [hậu quả cụ thể].

---

### Câu hỏi Clarification

#### CRITICAL — Cần trả lời trước khi tiếp tục
1. [Câu hỏi] — *Lý do: ...* *(Nguồn: domain-gap / as-is / input-analysis)*

#### MAJOR — Ảnh hưởng lớn đến thiết kế
2. [Câu hỏi] — *Lý do: ...*

#### MINOR — Có thể assume
3. [Câu hỏi] — *Lý do: ...*

---

### Assumptions đang áp dụng

- **[A1]** Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
- **[A2]** Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
```

---

## Rules

- Không hỏi lại những gì As-Is Summary đã liệt kê trong phần "Đã biết" — xem thêm no-re-ask rule trong `ba-conventions.md`
- Không hỏi câu kỹ thuật (API, DB, infrastructure) — xem IT-BA framing trong `ba-conventions.md`
- Không chặn workflow vì còn câu hỏi MINOR — assume và ghi chú theo format `ba-conventions.md`
- Assumption phải explicit, không để ẩn trong output
- Nếu không có câu hỏi CRITICAL → tiếp tục phân tích ngay, không chờ
- Sau khi output Clarification → chạy `resolve-oqs` skill để track OQ trước khi suggest downstream

## Failure Cases

- Gọi tất cả sub-skill mà không kiểm tra điều kiện → tốn thời gian, sinh câu hỏi thừa
- Không gộp câu hỏi trùng từ nhiều sub-skill → BA nhận 2 câu hỏi giống nhau
- Hỏi lại điều As-Is đã trả lời → vi phạm no-re-ask rule
- Không ghi assumption → agent tiếp theo không biết đang dựa trên gì
- Chạy sub-skill sai thứ tự → output kém chất lượng (ví dụ: domain-gap trước as-is thì không biết loại câu hỏi nào đã được trả lời)

## References

- @.claude/rules/ba-conventions.md — IT-BA framing, no-re-ask, assumption format, approval gate, OQ format
- @../shared/resolve-oqs.md — OQ tracking + cascade scan sau khi output
