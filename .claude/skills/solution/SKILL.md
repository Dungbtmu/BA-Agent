---
name: solution
description: Orchestrator nhóm Solution — điều phối 5 sub-skill thiết kế giải pháp sau khi requirement đã rõ. Gọi sub-skill phù hợp theo context: stakeholder-mapping, user-persona, context-constraint, assumption-risk, solution-critique. Trigger khi BA cần đề xuất giải pháp, thiết kế user flow, xác định edge cases, trade-offs, hoặc phản biện solution.
tools: []
---

# Skill: Solution Orchestrator

Entry point cho nhóm Solution. Nhận input đã được clarify, tự quyết định sub-skill nào cần chạy theo context.

> Quy tắc chung (IT-BA framing, no-re-ask, assumption, approval gate): xem `.claude/rules/ba-conventions.md`.

---

## 5 Sub-skill và khi nào gọi

| Sub-skill | Gọi khi | Không gọi khi |
|---|---|---|
| [`stakeholder-mapping.md`](references/stakeholder-mapping.md) | Cần xác định actor, vai trò, quyền lợi, mức độ ảnh hưởng của từng bên | Actor đã được clarify rõ ở Phase 1 |
| [`user-persona-identification.md`](references/user-persona-identification.md) | Cần profile chi tiết từng loại user: pain point, goal, behavior | Đã có persona từ trước |
| [`context-constraint-analysis.md`](references/context-constraint-analysis.md) | Cần xác định ràng buộc nghiệp vụ, kỹ thuật, pháp lý ảnh hưởng đến solution | Constraint đã rõ và đã được ghi nhận |
| [`assumption-risk-analysis.md`](references/assumption-risk-analysis.md) | Cần liệt kê assumption đang áp dụng và rủi ro kèm theo | Chỉ viết solution nhanh, không cần formal risk |
| [`solution-critique.md`](references/solution-critique.md) | Cần phản biện nội bộ solution trước khi đề xuất — tìm lỗ hổng, edge case, trade-off | Solution còn đang trong giai đoạn brainstorm |

---

## Thứ tự gợi ý

```
[context-constraint] + [stakeholder-mapping] → hiểu rõ bối cảnh và actor
   ↓
[user-persona] → profile chi tiết từng loại user
   ↓
Thiết kế solution + user flow
   ↓
[assumption-risk] → ghi nhận assumption + rủi ro
   ↓
[solution-critique] → phản biện nội bộ trước khi output
```

Không bắt buộc chạy đủ 5 sub-skill — skip nếu thông tin đã có từ Phase 1 hoặc BA không yêu cầu.

---

## Nguyên tắc

- Output chính của nhóm này là Solution Design doc tại `.claude/output/[tên_dự_án]/solution/`
- Mọi assumption phải được ghi rõ format `[A1] Giả định rằng X. Nếu sai, cần điều chỉnh Y.`
- Solution phải feasible — Dev đọc xong biết làm gì, không cần tự suy đoán thêm
- `solution-critique` nên chạy trước khi handoff sang `ba-devil-advocate-agent` để phản biện chéo
