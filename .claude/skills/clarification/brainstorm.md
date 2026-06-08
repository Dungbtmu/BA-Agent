# Brainstorm Skill — Deep Interview + Capture Idea

> Skill này chạy **trước** `ba-clarification-agent` khi BA có idea thô hoặc yêu cầu mơ hồ cần khai thác sâu trước khi phân tích chính thức. Output là Brainstorm Board — checkpoint bắt buộc trước khi chuyển sang clarification.

Quy tắc chung (IT-BA framing, no-re-ask, assumption, approval gate L1/L2/L3, OQ format): xem `@../shared/../../../.claude/rules/ba-conventions.md`.

---

## Mục tiêu

Khai thác idea thô thành **Brainstorm Board có cấu trúc** qua phỏng vấn 7 section (hỏi từng section một, không dồn batch). Output ghi lại: user types, capabilities P0/P1/P2, Core Flows với numbered steps + ASCII diagram, System Behavior Deep Dive (decision points, scenario matrix, state transitions, interrupted transaction handling), Validation/Limits/Wording (exact values), assumptions, risks, open questions.

Brainstorm là **checkpoint riêng** — KHÔNG nhảy thẳng sang clarification/solution mà không có BA approve output này trước.

---

## Khi nào dùng skill này

- BA đưa ý tưởng thô, mô tả mơ hồ chưa đủ để clarification
- Cần khai thác sâu về edge case, validation, wording trước khi viết requirement
- Feature phức tạp có external redirect, multi-role, state machine, async flow

---

## Inputs

```
/brainstorm                          # interactive — hỏi idea trước
/brainstorm <mô tả idea>             # idea text trực tiếp
/brainstorm <mô tả idea> --shallow   # fast mode, 1 batch câu hỏi
```

---

## Phase A — Nhận diện & Phát hiện độ phức tạp (silent)

1. **Resolve idea source:** Nếu không có arg → hỏi "Bạn brainstorm gì?". Chờ reply.
2. **Auto-derive feature slug** — extract noun phrase chính, kebab-case ASCII, tối đa 30 ký tự. Đề xuất trong L1, BA override được.
3. **Auto-derive idea slug** — semantic slug từ chủ đề. Fallback `idea-{NNN}`. Trùng → suffix `-v2`.
4. **Phát hiện complexity signals** từ content:
   - External redirect/OAuth/payment/webhook → `has_external_redirect = true`
   - signup/checkout/subscribe/verify/callback → `has_async_flow = true`
   - admin/user/guest / ≥2 roles / free/paid → `has_multi_role = true`
   - pending→active / draft→published / entity status → `has_state_machine = true`
   - rate limit/quota/captcha/lockout → `has_throttle_rules = true`
   - Flag từng để quyết định artifact bắt buộc tương ứng.

---

## Phase B — Phỏng vấn 7 Section (hỏi từng section, chờ reply)

> Mỗi section: 2–5 câu hỏi tối đa, in 1 message, wait reply. Push for exact values. BA `skip` → điền `<!-- TBD: ... -->` + ghi OQ.

### Section 1 — Overview
1. Feature này làm gì (1–2 câu từ góc user)?
2. Vấn đề/pain cụ thể đang giải? Ai bị ảnh hưởng?
3. Why now? (request từ ai, deadline, tín hiệu thị trường)

### Section 2 — Users & Access
1. Roles nào dùng (admin, free, paid, guest...)?
2. Gating: cần subscription/verified/role gì để truy cập?
3. Entry point: user vào feature qua đâu (menu, button, deep link, thông báo)?
4. Số lượng user dự kiến (để ước lượng capacity)?

### Section 3 — Core Flow (Happy Path)
1. Walk-through từng bước: user làm gì → system làm gì → user thấy gì (success state)?
2. Có sub-flow khác không (signup vs login, new vs returning, upgrade vs downgrade)?
3. Output cuối user thấy gì? Có notification/email gửi đi không?

### Section 4 — Detailed Flow Deep Dive *(chỉ chạy nếu complexity signal trigger từ Phase A)*

**4a. System actions (business level):** Mỗi bước nghiệp vụ system làm gì? Dùng action verb nghiệp vụ: "validate email format", "check email tồn tại", "tạo user record", "gửi verification email", "gọi Google OAuth", "ghi audit log". KHÔNG hỏi function name / service class / API endpoint. Loại thông tin nghiệp vụ nào cần lưu (liệt kê tên field nghiệp vụ, vd email, status, ngày tạo — KHÔNG hỏi column type / schema). Có gọi dịch vụ bên ngoài nào (chỉ tên dịch vụ + mục đích nghiệp vụ — KHÔNG hỏi endpoint/SDK)?

**4b. Decision points:** If/else nghiệp vụ nào trong flow? Condition + path YES/NO? Có tính toán/business rule gì?

**4c. State transitions:** Entity nào có status? Liệt kê: `entity: stateA → stateB → stateC`. Trigger từng transition? Có thể quay lại không?

**4d. Interrupted transactions** *(bắt buộc nếu `has_external_redirect || has_async_flow`)*:
- User đóng browser/app giữa flow → state gì còn lại, resume kiểu gì?
- External service fail/timeout → retry? State?
- User start flow mới trong khi cái cũ còn pending → behavior?
- Link/token expired → flow?
- 2 device cùng action → ai thắng?

**4e. ASCII flow diagram (L3 iterate)** *(bắt buộc nếu `has_external_redirect || has_async_flow || branching ≥2`)*:
- Skill vẽ v1 từ answers section 3+4a+4b.
- Hiển thị: "Diagram này đúng không? Sửa gì?" → iterate tối đa 3 vòng.
- Diagram phải show: user vs system action, decision với condition, external call, data change, error path.
- Dùng box-drawing `┌ ─ ┐ │ ▼` trong code block — KHÔNG dùng mermaid.

**4f. Scenario matrix** *(bắt buộc nếu `has_multi_role || ≥2 input states`)*:
- Liệt kê combo (from_state × to_state × rule) → action + result.
- Skill draft từ flow, hỏi BA confirm/correct.

### Section 5 — Validation, Limits & Wording
1. Required fields + format + min/max?
2. Limits/quotas (số chính xác): rate limit X/phút, max Y items, retry Z, lockout sau N lần fail?
3. Business rules: conditions, calculations, state-transition rules?
4. **Exact error messages** cho từng error case (string đúng wording, tiếng Việt tự nhiên)?
5. **Exact success messages** cho từng confirmation state?
6. **Exact info/neutral messages** (vd "Đã gửi email xác nhận tới {email}…")?

> Push for exact values: "Rate limit bao nhiêu/phút?" → "Lockout sau bao nhiêu fail?" → "Câu error chính xác là gì?". Vague vẫn vague sau 1 lần re-ask → TBD + flag OQ.

### Section 6 — System Context *(business-level, KHÔNG technical)*
1. Cần lưu thêm loại thông tin nghiệp vụ nào (vd "danh sách thiết bị", "lịch sử đăng nhập", "trạng thái subscription") — chỉ liệt kê **thông tin gì**, KHÔNG hỏi DB schema?
2. Có dịch vụ bên ngoài nào cần dùng (email service, OAuth, payment, SMS, captcha) — **tên dịch vụ + mục đích nghiệp vụ**, KHÔNG hỏi SDK/endpoint?
3. Notification gửi qua kênh nào (email/push/in-app/SMS) + trigger khi nào?
4. Có xử lý nền/scheduled không (vd cleanup token hết hạn mỗi ngày, gửi digest tuần) — chỉ **nhu cầu nghiệp vụ**?
5. Có cần real-time không (vd thông báo ngay khi event xảy ra) — chỉ **nhu cầu nghiệp vụ**?

### Section 7 — Edge Cases, Risks, Open Questions
1. Mất kết nối giữa chừng?
2. Dịch vụ bên ngoài bị lỗi?
3. Concurrent: 2 user cùng action lên cùng resource?
4. Pending/abandoned transactions — TTL, cleanup, resume path?
5. Top 3 rủi ro nghiệp vụ (adoption/vendor/compliance/process/timeline/data) — khả năng (thường/thỉnh thoảng/hiếm), hậu quả nghiệp vụ, cách phòng?
6. Đang chưa rõ gì → liệt kê thành open questions?

---

## Phase C — Tổng hợp & Quality Gate

Sau khi thu thập đủ answers, tổng hợp thành nội dung theo template:
- **Mục 5 Core Flows**: numbered steps + ASCII diagram embedded per flow
- **Mục 6.1 Decision Points**: bảng `ID | Flow | Khi nào | YES | NO`
- **Mục 6.2 Scenario matrix** (nếu trigger): bảng `From | To | Rule | Action | Result`
- **Mục 6.3 State transitions** (nếu trigger): bảng `Entity | Từ | Sang | Trigger | Quay lại?`
- **Mục 6.4 Interrupted-tx** (nếu trigger): bảng 4 cột
- **Mục 7.3 Wording**: 3 nhóm bảng riêng — error / success / info
- **Mục 9 Risks**: IT-BA format (Khả năng / Hậu quả nghiệp vụ / Cách phòng)

### Quality checklist gate (self-check trước L1)
- [ ] Mỗi flow ở Mục 5 có numbered steps user + system actions
- [ ] Flow phức tạp có ASCII diagram đi kèm
- [ ] Mục 6.1 Decision Points có tối thiểu các nhánh chính của flow
- [ ] Interrupted flow handling documented (nếu external redirect)
- [ ] Scenario matrix cover all combo (nếu multi-state)
- [ ] State transitions mapped (nếu có entity status)
- [ ] Mục 7.2 limits/quotas có exact numbers — KHÔNG "phù hợp"
- [ ] Mục 7.3 error/success/info messages là exact strings
- [ ] Risks dùng IT-BA framing — KHÔNG phải bug/infra
- [ ] Open questions có ID `OQ-1, OQ-2, ...`

Fail check → in checklist gap + đề xuất "hỏi thêm Q-X" trước proceed. BA có thể chọn "proceed anyway with TBD".

---

## Phase D — Approval Gate & Write

### L3 — Iterate ASCII flow diagram
Render trong chat → wait BA feedback → tối đa 3 vòng → chốt → tiếp tục L1.

### L1 — Plan preview

> Em sẽ tạo mới file `.claude/output/[dự_án]/brainstorm/{idea-slug}.md` với:
>
> **Nội dung sẽ có:**
> - {liệt kê 4–8 bullet bằng từ nghiệp vụ: "luồng đăng nhập có 5 bước", "bảng scenario 3 role", "wording error cho trường hợp sai mật khẩu", "lockout sau 5 lần fail", ...}
>
> **Câu hỏi mở:** {N} đã chốt trong session; còn {M} OQ để dành cho clarification/solution.
>
> Apply? (Y / sửa)

**CẤM trong L1:** bảng log dev, tag flag, từ technical (matrix, diagram, flag, schema).
**GIỮ:** số liệu nghiệp vụ cụ thể (lockout 5 lần, link 24h).

### L2 — Diff confirm (khi update brainstorm đã có)
Hiển thị diff trước khi ghi, hỏi Apply? (Y / n / sửa).

### Write
Lưu tại `.claude/output/[tên_dự_án]/brainstorm/{idea-slug}.md`.

Nếu chưa xác định tên dự án → hỏi BA hoặc dùng tên thư mục phù hợp với input.

---

## Phase E — Resolve Open Questions

Sau khi Write → chạy `resolve-oqs` skill (`@../shared/resolve-oqs.md`):
- Collect tất cả OQ trong doc
- Prompt BA: resolve all / skip / chọn IDs
- Loop 1-by-1 → cascade scan section bị ảnh hưởng
- Changelog sau mỗi resolve

---

## Output report cuối session

```
✅ Brainstorm captured: .claude/output/[dự_án]/brainstorm/{slug}.md
   Mode: deep | Sections: {N} | OQs: {M} | Quality gate: pass/partial

Bước tiếp theo được đề xuất:
  → ba-clarification-agent — làm rõ yêu cầu chính thức (inherit {M} OQ còn hold)
  → ba-solution-agent — nếu đã đủ rõ để thiết kế solution

BA approve trước khi tiếp tục.
```

---

## Shallow mode (`--shallow`)

Bỏ qua Phase B multi-section. Hỏi 1 batch 6 câu. Bỏ qua mandatory artifacts. Output ghi nhãn `mode: shallow`. Recommend trong report: "shallow mode — nên chạy lại deep nếu feature đi xa hơn prototype".

---

## Gotchas

- **Auto-derived feature slug có thể sai** — luôn show L1, BA override được
- **Vague answer** ("show error", "có rate limit") → re-ask 1 lần cụ thể hơn → vẫn vague → TBD + OQ; KHÔNG block workflow
- **BA bỏ giữa chừng** (skip section 5+) → proceed quality gate, in checklist gap, để BA quyết "write minimal" hay "tiếp tục"
- **Section 4 chỉ chạy nếu có complexity signal** — KHÔNG force BA trả lời 6 sub-questions cho feature đơn giản
- **Push exact values KHÔNG phải grilling** — re-ask 1 lần, tôn trọng BA; vague vẫn vague → TBD
- **No-re-ask**: continuation mode (file brainstorm đã có) → đọc kỹ doc trước khi hỏi, chỉ hỏi gap
- **Brainstorm là checkpoint** — KHÔNG auto-trigger clarification/solution sau khi write

---

## References

- `@../../../.claude/rules/ba-conventions.md`
- `@../shared/resolve-oqs.md`
- `@./brainstorm-example.md` — ví dụ output Brainstorm Board hoàn chỉnh (payment checkout)
