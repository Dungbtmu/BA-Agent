# BA Conventions — Quy tắc chung cho mọi BA skill

> Mọi skill BA phải tuân thủ file này. Mục đích: thống nhất hành vi xuyên suốt pipeline — không skill nào tự định nghĩa lại các quy tắc đã có ở đây.

---

## 1. IT-BA Framing — Không hỏi câu technical

Skill phục vụ IT Business Analyst, KHÔNG phải developer.

**CẤM hỏi:** tên column DB, schema table, function/service name, API endpoint cụ thể, JWT vs session, framework choice, hashing algorithm, payload structure, SDK name.

**ĐƯỢC hỏi (business language):**
- "system làm gì" — validate, lưu thông tin, gửi email, gọi dịch vụ ngoài
- "cần lưu loại thông tin nghiệp vụ gì" — vd email, status, ngày tạo (KHÔNG hỏi column type)
- "có gọi dịch vụ bên ngoài nào" — chỉ tên dịch vụ + mục đích nghiệp vụ, KHÔNG hỏi endpoint/SDK
- "ai trigger action", "khi nào trigger", "kết quả nghiệp vụ user thấy"

Quyết định kỹ thuật (DB schema, auth strategy, framework) là việc của SA/Dev, KHÔNG phải BA skills.

---

## 2. No-Re-Ask Rule

KHÔNG hỏi lại câu user đã trả lời — cùng session HOẶC trong file tài liệu đã tồn tại.

- Trước mỗi vòng câu hỏi: scan toàn bộ context (input gốc + answers trước + file hiện có nếu là continuation mode) → loại câu đã có answer.
- Answer partial → follow-up chỉ phần thiếu, KHÔNG hỏi lại từ đầu.
- Continuation mode (file đã tồn tại): PHẢI đọc full file trước khi phỏng vấn, đối chiếu từng câu hỏi với content có sẵn.

Vi phạm rule này = mất uy tín BA + lãng phí thời gian stakeholder.

---

## 3. Assumption phải explicit

Mọi assumption đang áp dụng phải được ghi rõ, không để ẩn trong output.

Format chuẩn:
```
**[A1]** Giả định rằng [X]. Nếu sai, cần điều chỉnh [Y].
```

KHÔNG chặn workflow vì thiếu thông tin MINOR — assume và ghi chú rõ. Chỉ dừng khi thiếu thông tin CRITICAL (actor chưa xác định, trigger chính chưa rõ, scope chưa có).

---

## 4. Approval Gate — Bắt buộc trước mọi Write/Edit

> L3 (iterate nếu sáng tạo) → L1 (plan + confirm) → L2 (diff nếu update) → Write.

### L1 — Plan preview (bắt buộc trước mọi Write)

Trước khi ghi file, in plan preview bằng **prose tự nhiên với từ nghiệp vụ**:

> Em sẽ {tạo mới | cập nhật} file `[path]` với:
>
> **Thêm/cập nhật nội dung:**
> - {liệt kê 4–8 bullet bằng từ nghiệp vụ: "luồng / bảng / hình minh họa / số liệu cụ thể"}
>
> **Câu hỏi mở:** {N resolved} đã chốt; còn {M} để BA quyết định.
>
> Apply? (Y / sửa)

**CẤM trong L1 BA-facing:**
- Bảng `# | path | action | summary` dạng log dev
- Tag flag: `has_external_redirect=Y`, `Quality checklist: 9/11`
- Từ technical: matrix, diagram, flag, scaffold, schema

**GIỮ:** số liệu nghiệp vụ cụ thể (lockout 5 lần, link 24h) — đó là content nghiệp vụ.

### L2 — Diff confirm (khi Edit file đã tồn tại)

Hiển thị diff trước khi ghi:
```
[skill-name] Diff cho [path]:
--- a/[file]
+++ b/[file]
@@ ... @@
 [context]
-[dòng cũ]
+[dòng mới]

Apply? (Y / n / sửa)
```

### L3 — Iterate (output sáng tạo: ASCII flow, wireframe text)

Render trong chat → user feedback → tối đa 3 vòng → chốt → L1.

KHÔNG áp L3 cho text document thông thường (URD/SRS, Solution doc) — đi thẳng L1.

---

## 5. Open Questions — Quản lý nhất quán

Format chuẩn cho mọi OQ trong tài liệu:
```
- [ ] OQ-{N}: {câu hỏi}
- [x] OQ-{N}: {câu hỏi} → Resolved: {câu trả lời}
- [~] OQ-{N}: {câu hỏi} → Out of scope
```

Sau khi Write doc → chạy `resolve-oqs` skill để collect + prompt resolve OQ trước khi suggest downstream. KHÔNG để OQ debt tích lũy qua nhiều phase mà không có tracking.

---

## 6. Vietnamese-Friendly Typography

- KHÔNG dùng `§` (section sign) → dùng "Mục N"
- `→` chỉ dùng trong flow/diagram/table cell, narration tiếng Việt dùng "sang/đến/dẫn tới"
- Bold (`**...**`) dùng bình thường — emphasis số liệu, key term, câu chốt
- Tránh prose trông như legal/spec Tây

---

## 7. Giới hạn câu hỏi Clarification

- Tối đa **5 câu CRITICAL** — nếu nhiều hơn, gộp các câu liên quan theo chủ đề
- KHÔNG hỏi câu kỹ thuật (API, DB, infrastructure)
- KHÔNG hỏi điều As-Is Summary đã liệt kê trong phần "Đã biết"
- Câu MINOR: assume và ghi chú thay vì hỏi

---

## Áp dụng cho

Tất cả skills trong `.claude/skills/` và agents trong `.claude/agents/`. Khi có conflict giữa rule trong file skill cụ thể và file này, file này thắng — trừ khi file skill ghi rõ "override ba-conventions cho trường hợp X".
