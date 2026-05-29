---
name: input_analysis
description: Đọc và phân tích tài liệu có sẵn (PRD, tài liệu cũ, email, file nghiệp vụ) — trích xuất requirement thô, phân loại rõ/chưa rõ/mâu thuẫn để làm input cho clarification
tools: []
---

# Skill: Input Analysis

## Mục tiêu

Khi BA nhận được tài liệu có sẵn (PRD, email mô tả, ghi chú meeting, tài liệu nghiệp vụ cũ) — đọc, trích xuất requirement thô, phân loại những gì đã rõ và những gì còn cần làm rõ. Output làm input cho `requirement-clarification.md`.

## Input

- Tài liệu có sẵn ở bất kỳ dạng nào: PRD, email, ghi chú, file Markdown/Word/PDF, tài liệu hệ thống cũ
- Bối cảnh ngắn gọn từ BA (nếu có): dự án này là gì, đang ở giai đoạn nào

## Output

- Danh sách requirement thô đã trích xuất
- Phân loại rõ ràng / chưa rõ / mâu thuẫn
- Danh sách actor và chức năng nhận diện được
- Danh sách câu hỏi cần clarify chuyển sang `requirement-clarification.md`

---

## Thinking Pattern

1. **Tài liệu này nói về cái gì?** — Mục đích tổng thể, ai viết, viết cho ai
2. **Requirement nào đã được phát biểu rõ?** — Có thể đưa thẳng vào phân tích mà không cần hỏi thêm
3. **Requirement nào mơ hồ?** — Dùng từ chung chung, thiếu điều kiện, thiếu actor
4. **Có mâu thuẫn nào không?** — Hai đoạn nói ngược nhau, hoặc logic không nhất quán
5. **Actor nào được nhắc đến?** — Trực tiếp và ngầm định
6. **Scope đã rõ chưa?** — In scope / out of scope có được đề cập không?

---

## Execution

### Bước 1 — Đọc lướt toàn bộ tài liệu

Không phân tích ngay — chỉ đọc để nắm bức tranh tổng thể:
- Tài liệu này là gì (PRD, proposal, email, meeting note)?
- Có bao nhiêu chủ đề chính?
- Độ chi tiết ở mức nào?

### Bước 2 — Trích xuất requirement thô

Đọc lại từng phần, liệt kê requirement theo dạng bullet:
- Dùng câu ngắn: "[Actor] có thể/cần/phải [hành động]"
- Không diễn giải, không thêm ý — chỉ ghi lại những gì tài liệu đã nói
- Đánh số để tham chiếu dễ

### Bước 3 — Phân loại từng requirement

| Loại | Dấu hiệu | Cần làm |
|---|---|---|
| **Rõ** | Đủ actor, hành động, điều kiện | Đưa thẳng vào phân tích |
| **Chưa rõ** | Thiếu actor, điều kiện mơ hồ, từ chung chung | Ghi câu hỏi clarify |
| **Mâu thuẫn** | Hai chỗ nói ngược nhau | Ghi cả 2, đánh dấu conflict |
| **Ngoài scope** | Tài liệu đề cập nhưng có vẻ không thuộc phạm vi dự án | Ghi chú để xác nhận |

### Bước 4 — Nhận diện actor

- Liệt kê tất cả role/người dùng được nhắc đến — cả trực tiếp ("Quản trị viên") lẫn ngầm định ("người tạo đơn hàng")
- Ghi rõ nếu không chắc: `[Có thể là: ...]`

### Bước 5 — Tổng hợp câu hỏi clarify

Với mỗi requirement "chưa rõ" và "mâu thuẫn" → sinh câu hỏi clarify theo format của `requirement-clarification.md` (CRITICAL / MAJOR / MINOR).

---

## Rules

- **Không diễn giải** — chỉ trích xuất những gì tài liệu đã nói; nếu phải suy luận thì ghi `[Suy luận: ...]`
- **Không bỏ sót** — dù requirement có vẻ hiển nhiên vẫn phải liệt kê để traceability
- **Không fix mâu thuẫn** — chỉ ghi nhận, không tự chọn một phía
- **Ngắn gọn** — mỗi requirement 1 câu; không viết đoạn văn
- **Tách biệt requirement và câu hỏi** — không gộp 2 phần này vào nhau

---

## Output Format

```
## Phân tích tài liệu: [Tên tài liệu / mô tả ngắn]

**Loại tài liệu:** [PRD / Email / Meeting note / Tài liệu hệ thống cũ / ...]
**Số requirement trích xuất:** [N]

---

### Requirement đã trích xuất

| # | Requirement | Loại | Ghi chú |
|---|---|---|---|
| R01 | [Actor] có thể [hành động] | Rõ / Chưa rõ / Mâu thuẫn / Ngoài scope | [Nếu cần] |
| R02 | ... | ... | ... |

---

### Actor nhận diện được

| Actor | Cách nhắc đến trong tài liệu | Ghi chú |
|---|---|---|
| [Tên role] | Trực tiếp / Ngầm định | [Nếu chưa chắc] |

---

### Câu hỏi cần clarify

*(Chuyển sang `requirement-clarification.md` để xử lý tiếp)*

**CRITICAL:**
- [Câu hỏi] — liên quan đến R[số]

**MAJOR:**
- [Câu hỏi] — liên quan đến R[số]

**MINOR:**
- [Câu hỏi] — liên quan đến R[số]

---

### Assumption đang áp dụng

- **[A1]** Giả định rằng [X]. Nếu sai, cần xem lại R[số].
```

---

## Failure Cases

- Diễn giải requirement theo ý mình thay vì trích xuất nguyên văn → sai ngay từ đầu
- Bỏ qua requirement mơ hồ vì "nghĩ là hiểu rồi" → gap bị phát hiện muộn
- Fix mâu thuẫn không hỏi lại → chọn sai phía, requirement bị hiểu lệch
- Không phân loại actor → không biết ai làm gì khi phân tích tiếp
- Gộp câu hỏi clarify vào trong bảng requirement → lộn xộn, khó đọc
- Đọc tài liệu quá chi tiết ngay từ bước đầu → mất thời gian, bỏ sót bức tranh tổng thể
