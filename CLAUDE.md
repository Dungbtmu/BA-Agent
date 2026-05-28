# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Repo này là **BA Agent System** — hỗ trợ BA tạo tài liệu phân tích nghiệp vụ cho mọi dự án, mọi domain. Output chính là **URD/SRS dev/test-ready**. Các artifact phụ trợ (Epic, User Story, AC, Wireframe, React prototype) chỉ tạo khi BA yêu cầu rõ. Không có code production để build, test, hay compile.

BA agent tập trung vào **nghiệp vụ và giao diện mức BA** — bao gồm cả gen React prototype để stakeholder feedback — không xử lý kiến trúc kỹ thuật hay production code.

---

## Danh sách Agent

Gọi agent bằng cách mô tả yêu cầu — hệ thống tự nhận dạng intent và dispatch đúng agent. Chi tiết keyword → agent xem trong `agent-workflow.md`.

### Phase 1 — Làm rõ & Giải pháp

| Agent | Dùng khi nào |
|---|---|
| `ba-research-agent` | Domain mới, chưa có kiến thức — research nghiệp vụ, actor, pain point, thuật ngữ. Output: Domain Brief |
| `ba-clarification-agent` | Có yêu cầu thô (mô tả miệng, ghi chú, tài liệu chưa đầy đủ) — làm rõ trước khi thiết kế |
| `ba-solution-agent` | Đã rõ yêu cầu — đề xuất giải pháp, thiết kế user flow, xác định edge cases và trade-offs |
| `ba-backlog-agent` | *(Optional)* Cần chia solution thành Epic + User Stories + AC rõ ràng, testable |

### Phase 2 — Thiết kế giao diện

| Agent | Dùng khi nào |
|---|---|
| `ba-wireframe-agent` | Phác thảo màn hình, layout, UI flow dạng text từ User Stories hoặc solution |
| `ui-react-agent` | Gen React prototype để stakeholder xem trực quan và đưa feedback |
| `ui-feedback-agent` | Nhận feedback UI thô — phân loại, map về đúng màn hình/component, chỉ định sửa đúng chỗ |

### Phase 3 — Tài liệu

| Agent | Dùng khi nào |
|---|---|
| `urd-srs-agent` | Viết tài liệu URD/SRS đầy đủ, dev/test-ready — output chính của hệ thống |
| `ba-qa-agent` | Review tài liệu — phát hiện logic gaps, missing cases, conflict, risk. Tự động chạy sau `urd-srs-agent` |

---

## Workflow 3 Phase

```
Phase 1 — Làm rõ & Giải pháp
  [0] ba-research-agent    (optional — chỉ khi domain mới)
   ↓
  [1] ba-clarification-agent
   ↓
  [2] ba-solution-agent
   ↓
  [3] ba-backlog-agent     (optional — chỉ khi BA cần Epic/Story/AC)

Phase 2 — Thiết kế giao diện (vòng lặp đến khi chốt)
  [4] ba-wireframe-agent   ←→ song song với
  [5] ui-react-agent
   ↓
  [6] ui-feedback-agent → triage → sửa đúng chỗ → lặp lại

Phase 3 — Tài liệu (vòng lặp đến khi chốt)
  [7] urd-srs-agent → viết URD/SRS
   ↓ (tự động)
  [8] ba-qa-agent → review → CRITICAL: sửa đúng section → lặp lại
                           → MAJOR/MINOR: báo BA quyết định
                           → Không issue: chốt, lưu file version cuối
```

Có thể bắt đầu từ bất kỳ phase nào nếu đã có input phù hợp.

---

## Rules chi tiết

Các quy tắc được tách thành file riêng trong `.claude/rules/`:

- [`project-context.md`](.claude/rules/project-context.md) — Bối cảnh hệ thống, phạm vi BA, team, cấu trúc thư mục
- [`ba-persona.md`](.claude/rules/ba-persona.md) — Phong cách, tư duy, năng lực BA Senior
- [`language.md`](.claude/rules/language.md) — Quy tắc ngôn ngữ (tiếng Việt có dấu bắt buộc)
- [`ba-workflow.md`](.claude/rules/ba-workflow.md) — Quy trình GENERATE / REFINE / REVIEW
- [`agent-workflow.md`](.claude/rules/agent-workflow.md) — Quy trình phối hợp subagent, intent recognition, Phase 1-2-3
- [`output-schema.md`](.claude/rules/output-schema.md) — Tiêu chuẩn chất lượng URD/SRS (output chính) + Epic, Story, Wireframe (phụ trợ)
