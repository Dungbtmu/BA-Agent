---
name: requirement_clarification
description: Phát hiện các điểm chưa rõ, thiếu thông tin hoặc mâu thuẫn trong requirement — sinh câu hỏi clarify có độ ưu tiên
tools: []
---

# Skill: Requirement Clarification

## Mục tiêu

Làm rõ requirement trước khi phân tích sâu — tránh build sai do hiểu nhầm hoặc thiếu context quan trọng.

## Input

- `requirement`: mô tả yêu cầu ở bất kỳ dạng nào (text, bullet, ghi chú thô)
- `domain_context`: optional — ngữ cảnh dự án, domain, constraints đã biết

## Output

- Danh sách câu hỏi clarify có phân loại CRITICAL / MAJOR / MINOR
- Danh sách assumption đang được áp dụng nếu không hỏi thêm

## Thinking Pattern

1. **Who** — Ai là user thực sự? Có nhiều nhóm user không?
2. **What** — Hành động cụ thể là gì? Có term nào mơ hồ, nhiều nghĩa không?
3. **Why** — Mục tiêu nghiệp vụ đằng sau là gì? Đây là problem hay solution được đề xuất?
4. **Scope** — In scope / out of scope đã rõ chưa? Có trường hợp ngoại lệ nào không được đề cập?
5. **Conflict** — Có yêu cầu nào mâu thuẫn nhau không? Có điều kiện "nếu A thì B" chưa được xử lý?
6. **Constraint ẩn** — Có ràng buộc nào về thời gian, hệ thống hiện tại, quy định pháp lý không được nói ra?

## Execution

**Bước 1 — Scan toàn bộ input**
- Đọc hết requirement, không phân tích ngay
- Đánh dấu những chỗ mơ hồ, thiếu, hoặc mâu thuẫn

**Bước 2 — Phân loại gap**
- CRITICAL: thiếu thông tin này thì không thể thiết kế được (ví dụ: không biết user là ai, không biết trigger của flow)
- MAJOR: có thể tiếp tục nhưng rủi ro assumption sai cao (ví dụ: không rõ edge case quan trọng)
- MINOR: ảnh hưởng nhỏ, có thể assume và ghi chú (ví dụ: label nút bấm, màu sắc)

**Bước 3 — Sinh câu hỏi**
- Mỗi câu hỏi phải gắn với một gap cụ thể
- Hỏi theo dạng mở hoặc đưa ra 2–3 lựa chọn để user trả lời nhanh hơn
- Nếu có nhiều câu hỏi cùng loại CRITICAL → gộp thành nhóm theo chủ đề

**Bước 4 — Ghi assumption**
- Với mỗi câu hỏi MINOR hoặc MAJOR có thể tự assume → ghi rõ assumption thay vì hỏi
- Assumption phải theo dạng: "Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y]."

## Rules

- Không hỏi quá 5 câu CRITICAL trong một lượt — nếu nhiều hơn, chọn câu quan trọng nhất
- Không hỏi những thứ có thể suy ra từ context hoặc domain knowledge phổ biến
- Không hỏi câu hỏi kỹ thuật (API dùng gì, DB nào) — đó là phạm vi SA/Dev
- Assumption phải explicit, không để ẩn trong output
- Nếu không có câu hỏi CRITICAL → tiếp tục phân tích, không chặn workflow

## Output Format

```
## Câu hỏi Clarification

### CRITICAL — Cần trả lời trước khi tiếp tục
1. [Câu hỏi] — *Lý do: ...*
2. [Câu hỏi] — *Lý do: ...*

### MAJOR — Ảnh hưởng lớn đến thiết kế
3. [Câu hỏi] — *Lý do: ...*

### MINOR — Có thể assume, hỏi để confirm
4. [Câu hỏi] — *Lý do: ...*

---

## Assumptions đang áp dụng

- **[A1]** Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
- **[A2]** Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
```

## Failure Cases

- Hỏi quá nhiều câu cùng lúc → user bị overwhelm, không trả lời
- Hỏi câu kỹ thuật thay vì câu nghiệp vụ → sai phạm vi BA
- Không ghi assumption → agent tiếp theo không biết đang dựa trên gì
- Chặn workflow vì còn câu hỏi MINOR → mất thời gian không cần thiết
- Hỏi lại điều user đã nói rõ trong input → mất uy tín
