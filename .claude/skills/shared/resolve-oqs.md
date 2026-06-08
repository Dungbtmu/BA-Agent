---
name: resolve_oqs
description: Skill quản lý và resolve Open Questions sau khi Write doc — collect OQ từ doc hiện tại + upstream, prompt BA resolve từng cái, cascade scan các section bị ảnh hưởng, update upstream docs nếu cần. Chạy sau Write, trước suggest downstream.
tools: [Read, Edit, Write]
---

# Skill: Resolve Open Questions (Phase E)

## Mục tiêu

Không để OQ debt tích lũy qua nhiều phase mà không có tracking. Sau mỗi lần Write doc → buộc collect + resolve hoặc acknowledge hold ngay, trước khi suggest downstream skill.

---

## Trigger

Skill này chạy ngay sau khi Write doc thành công — cả create lẫn update. Áp dụng cho mọi BA output: clarification output, solution doc, brainstorm doc, URD/SRS.

---

## Bước 1 — Collect OQs

Gom 2 nguồn:

**1. Own OQs** — parse mục "Open Questions" / "Câu hỏi mở" trong doc vừa write. Format nhận dạng: `- [ ] OQ-N:` hoặc `- [ ] OQ:`.

**2. Inherited OQs** — từ upstream docs còn `[ ]` (unresolved) hoặc `[~]` (deferred). Upstream chain theo từng loại doc:

| Doc hiện tại | Upstream cần scan |
|---|---|
| Brainstorm | (không có — brainstorm là gốc) |
| Clarification output | (không có — Phase 1 gốc) |
| Solution doc | Clarification output nếu có |
| URD/SRS | Solution doc, Clarification output |
| Process Summary | Tất cả upstream |

Bỏ qua `[x]` (resolved). Chỉ gom `[ ]` và `[~]`.

---

## Bước 2 — Prompt BA

Nếu N == 0 → skip Phase E, đi thẳng final report.

Nếu N > 0, in:

```
📋 Còn {N} câu hỏi mở:

Từ doc hiện tại:
  - OQ-{id}: {text}

Inherited từ {upstream_path}:
  - OQ-{id}: {text}

Resolve ngay? (Y / skip / {id cụ thể vd "OQ-3,OQ-4"})
  Y     → hỏi từng OQ một
  skip  → giữ OQ, downstream inherit
  ids   → chỉ resolve OQ được chỉ định
```

---

## Bước 3 — Resolve Loop (một OQ mỗi lần)

Với mỗi OQ được chọn:

1. In OQ + 1–2 dòng context từ section liên quan.
2. Chờ BA trả lời. 4 dạng reply:
   - **Answer cụ thể** → mark `[x]`, ghi "Resolved: {answer}"
   - **`skip` / `hold`** → giữ `[ ]`, note "hold tới {next_skill}"
   - **`không cần` / `oos`** → mark `[~]`, note "out of scope"
   - **`không biết` / `hỏi sau`** → giữ `[ ]`, không retry trong session này
3. Sau khi có answer → chạy **cascade scan** (Bước 3.5).
4. Nếu OQ inherited từ upstream → propose L2 diff để update upstream doc (mark `[x]`/`[~]` + note "resolved via {current_skill}").

---

## Bước 3.5 — Cascade Scan (KHÔNG được skip)

Sau mỗi OQ resolved, quét rộng hơn để propagate vào các section liên quan. KHÔNG để OQ "resolved" trên giấy nhưng các section khác vẫn ghi assumption cũ.

**Scan trong doc hiện tại:**
1. Direct reference: pattern `OQ-{id}`, `chờ OQ-{id}`, `pending OQ-{id}` → propose L2 diff xóa reference hoặc thay bằng resolution.
2. Next Steps: bullet `- Resolve OQ-{id}` → propose xóa.
3. Topic-based: dùng bảng mapping dưới → đọc section có thể chứa assumption cũ → nếu conflict với answer → propose L2 diff.

**Scan downstream docs** (nếu doc hiện tại không phải cuối chain):

| Doc hiện tại | Scan downstream |
|---|---|
| Clarification output | Solution doc (nếu có) |
| Solution doc | URD/SRS (nếu có) |
| URD/SRS | Process Summary (nếu có) |
| Brainstorm | Clarification output, Solution doc, URD/SRS |

**Topic → Section mapping (heuristic):**

| OQ topic keywords | Sections cần check |
|---|---|
| actor, vai trò, người dùng, role, RBAC | Phân quyền, Permission Matrix, RBAC, UC Actor |
| phạm vi, scope, bao gồm, không bao gồm | Phạm vi tài liệu (I.2), Giả định |
| constraint, ràng buộc, hạn chế | Yêu cầu phi chức năng (C), Giả định |
| trigger, sự kiện, khi nào | Workflow Diagram, Use Case trigger |
| thông báo, notification, email, SMS | UC Extension, Screen Spec |
| dữ liệu, thông tin cần lưu, field | Use Case business rule, Screen Spec validation |
| timeline, deadline, thời gian | Giả định, Risk |
| tích hợp, hệ thống ngoài, API bên ngoài | Kiến trúc tổng thể (I.4), Sequence Diagram (II.5) |

Nếu không chắc section nào bị ảnh hưởng → hỏi BA "Mục nào còn liên quan?"

**Tóm tắt impact trước khi loop diff:**
```
🔗 OQ-{id} resolved → {K} sections/docs cần update:

Doc hiện tại:
  - Mục {section}: {1-line preview thay đổi}

Downstream:
  - {path} Mục {section}: {preview}

Apply lần lượt? (Y / skip-all / chọn id)
```

---

## Bước 4 — Changelog

Sau khi resolve ≥1 OQ, append vào doc hiện tại:

```yaml
changelog:
  - {date} | resolve-oqs | resolved OQ-{ids}: {short summary}
```

Mỗi upstream/downstream doc bị update cũng có changelog entry riêng.

---

## Bước 5 — Final Report

```
✅ {Doc_type} finalized: {path}
   Resolved OQs trong session: {R}/{N}
   Còn hold: {M} (sẽ inherit khi chạy {next_skill})

Recommended next:
  - {next_skill}   — {mô tả ngắn}
```

Nếu BA skip Phase E:
```
⚠️  {N} OQ vẫn hold. Khi chạy {next_skill}, sẽ inherit list này.
```

---

## Rules

- Resolve loop **một OQ mỗi lần** — KHÔNG dồn batch.
- L2 diff trước mọi Edit (doc hiện tại + upstream + downstream).
- Push exact values — vague answer → retry 1 lần với câu cụ thể hơn. Vẫn vague → giữ `[ ]`, không force.
- Side-effect updates KHÔNG silent — luôn propose L2 trước.
- **Cascade scan bắt buộc** — mark `[x]` xong mà không scan = OQ "resolved" trên giấy nhưng bug nghiệp vụ vẫn còn.
- KHÔNG hỏi lại OQ đã `[x]` trong upstream.

## Anti-patterns

- ❌ Output final report ngay sau Write, bỏ qua OQs.
- ❌ Gom "Resolve OQs" vào "Recommended next" list (làm mất ưu tiên).
- ❌ Ép BA resolve hết — BA có quyền skip hoặc hold.
- ❌ Update upstream OQ silent mà không có L2 diff.
- ❌ Mark `[x]` nhưng bỏ qua cascade scan.

## References

- @.claude/rules/ba-conventions.md — Section 5 (OQ format chuẩn)
