---
name: assumption_risk_analysis
description: Xác định giả định đang được áp dụng và đánh giá rủi ro theo impact/likelihood — dùng trong solution design và review
tools: []
---

# Skill: Assumption & Risk Analysis

## Mục tiêu

Expose assumption ẩn và đánh giá rủi ro để stakeholder có thể ra quyết định có thông tin đầy đủ — tránh build dựa trên nền móng sai.

## Input

- `context`: mô tả bài toán, solution đề xuất, hoặc tài liệu cần review
- `existing_assumptions`: optional — assumption đã được ghi nhận từ bước trước

## Output

- Danh sách assumption có phân loại (Confirmed / Unverified / Risky)
- Danh sách risk có đánh giá impact × likelihood và mitigation

## Thinking Pattern

1. **Assumption về user** — Đang giả định user sẽ hành xử như thế nào? Có evidence không?
2. **Assumption về hệ thống** — Đang giả định hệ thống hiện tại có khả năng gì? Đã kiểm chứng chưa?
3. **Assumption về dữ liệu** — Dữ liệu cần thiết có sẵn không? Đúng format không? Đủ chất lượng không?
4. **Assumption về nghiệp vụ** — Quy trình hiện tại có đúng như mô tả không? Có exception nào không?
5. **Nếu assumption sai** — Impact là gì? Cần làm lại bao nhiêu? Có cách validate không?
6. **Risk từ bên ngoài** — Dependency, thay đổi quy định, timeline, resource?

## Execution

**Bước 1 — Liệt kê assumption**
- Quét toàn bộ context, tìm những chỗ "đang ngầm hiểu là..."
- Phân loại:
  - **Confirmed**: đã có evidence hoặc stakeholder xác nhận
  - **Unverified**: hợp lý nhưng chưa kiểm chứng
  - **Risky**: có thể sai, impact cao nếu sai

**Bước 2 — Xác định risk**
- Mỗi assumption Unverified/Risky → tạo risk tương ứng
- Thêm risk từ dependency, timeline, con người, quy định
- Đánh giá: Impact (H/M/L) × Likelihood (H/M/L)
- Ưu tiên xử lý: High Impact + High Likelihood trước

**Bước 3 — Đề xuất mitigation**
- Với mỗi risk HIGH → phải có mitigation cụ thể
- Mitigation ưu tiên: validate sớm > contingency plan > accept risk

## Rules

- Tách rõ assumption và fact đã được xác nhận
- Risk phải cụ thể — không viết "có thể xảy ra vấn đề"
- Mỗi risk phải có ít nhất một mitigation action
- Không đánh giá risk kỹ thuật (performance, security architecture) — đó là phạm vi SA/Dev
- Nếu không có assumption nào → ghi rõ "Không phát hiện assumption ẩn" thay vì bỏ trống

## Output Format

```
## Assumptions

| # | Assumption | Loại | Ghi chú |
|---|---|---|---|
| A1 | [Nội dung assumption] | Confirmed / Unverified / Risky | [Evidence hoặc lý do] |
| A2 | ... | ... | ... |

---

## Risk Register

| # | Rủi ro | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| R1 | [Mô tả rủi ro cụ thể] | H / M / L | H / M / L | [Hành động cụ thể] |
| R2 | ... | ... | ... | ... |

**Ưu tiên xử lý ngay:** R1, R3 (High Impact × High Likelihood)
```

## Failure Cases

- Liệt kê assumption hiển nhiên, không có giá trị phân tích (ví dụ: "user cần internet")
- Risk mơ hồ như "có thể có vấn đề" — không đủ để stakeholder hành động
- Bỏ qua assumption về dữ liệu — đây là nguồn rủi ro hay bị miss nhất
- Không có mitigation cho risk HIGH → không có giá trị thực tế
- Nhầm risk kỹ thuật (infra, latency) với risk nghiệp vụ
