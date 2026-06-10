---
name: ui
description: Orchestrator nhóm UI — điều phối 3 sub-skill thiết kế giao diện theo Phase 2. Gọi wireframe-design-system khi phác thảo màn hình, react-ui-generation khi gen prototype, ui-feedback-triage khi nhận feedback stakeholder. Trigger khi BA cần vẽ wireframe, tạo React prototype, hoặc xử lý feedback UI.
tools: []
---

# Skill: UI Orchestrator

Entry point cho toàn bộ nhóm UI. Nhận intent từ BA, gọi đúng sub-skill theo giai đoạn thiết kế.

> Quy tắc chung (IT-BA framing, no-re-ask, assumption, approval gate L1/L2/L3): xem `.claude/rules/ba-conventions.md`.

---

## 3 Sub-skill và khi nào gọi

| Sub-skill | Gọi khi | Không gọi khi |
|---|---|---|
| [`wireframe-design-system.md`](references/wireframe-design-system.md) | Cần phác thảo màn hình, layout, UI flow dạng text — lần đầu hoặc update wireframe | Wireframe đã chốt, chỉ cần code |
| [`react-ui-generation.md`](references/react-ui-generation.md) | Wireframe hoặc solution đã đủ rõ, cần gen React prototype để stakeholder xem trực quan | Wireframe chưa có hoặc chưa chốt |
| [`ui-feedback-triage.md`](references/ui-feedback-triage.md) | Có feedback thô từ stakeholder (comment, mô tả miệng, ghi chú) về UI | Chưa có prototype để feedback |

---

## Thứ tự thực hiện (Phase 2)

```
[4] wireframe-design-system → wireframe text đủ rõ
   ↓ (chỉ tiếp tục khi wireframe/solution đã rõ)
[5] react-ui-generation → React prototype
   ↓
[6] ui-feedback-triage → triage → sửa đúng chỗ
   ↓ (lặp lại [4][5][6] cho đến khi stakeholder chốt)
```

**Quan trọng:** `react-ui-generation` [5] KHÔNG chạy song song với `wireframe-design-system` [4] — phải có wireframe xong trước.

---

## Nguyên tắc

- Approval Gate L3 bắt buộc cho wireframe (render ASCII → feedback → tối đa 3 vòng → chốt → L1)
- Feedback từ Phase 2 chỉ ảnh hưởng wireframe/UI — không tự thay đổi Epic/Story trừ khi BA yêu cầu
- `ui-feedback-triage` map từng feedback về đúng màn hình/component trước khi sửa — không sửa lan sang phần không liên quan
