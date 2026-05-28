---
name: context_constraint_analysis
description: Phân tích ràng buộc kỹ thuật, business và pháp lý — xác định giới hạn mà solution phải tuân theo trước khi thiết kế
tools: []
---

# Skill: Context & Constraint Analysis

## Mục tiêu

Xác định rõ giới hạn cứng (hard constraint) và giới hạn mềm (soft constraint) để solution được đề xuất là khả thi ngay từ đầu — tránh thiết kế xong mới phát hiện không thể implement.

## Input

- `context`: mô tả dự án, bài toán, hoặc tài liệu yêu cầu
- `domain`: optional — ngành nghề (Fintech, EdTech, Logistics...)
- `existing_systems`: optional — hệ thống đang tồn tại cần tích hợp hoặc kế thừa

## Output

- Danh sách constraint phân loại theo nhóm
- Phân biệt rõ hard constraint (không thể thay đổi) vs soft constraint (có thể thương lượng)

## Thinking Pattern

1. **Technical** — Hệ thống hiện tại có gì? Phải tích hợp với gì? Có giới hạn về platform, device, offline/online không?
2. **Business** — Deadline? Budget? Team size? Phạm vi giai đoạn này vs roadmap dài hạn?
3. **Legal / Compliance** — Có quy định ngành nào áp dụng không? Dữ liệu người dùng có ràng buộc gì (PDPA, GDPR, quy định ngân hàng...)?
4. **Operational** — Ai vận hành? Trình độ người dùng? Có SLA, uptime requirement không?
5. **Dependency** — Solution phụ thuộc vào team nào, hệ thống nào? Họ có sẵn sàng không?
6. **Hard vs Soft** — Cái nào không thể thay đổi? Cái nào có thể negotiate nếu cần?

## Execution

**Bước 1 — Thu thập constraint từ context**
- Đọc kỹ input, tìm những từ chỉ giới hạn: "phải", "không được", "chỉ", "bắt buộc", "theo quy định"
- Tìm cả constraint ẩn không được nói ra (ví dụ: hệ thống legacy → không thể thay đổi DB schema)

**Bước 2 — Phân loại**
- Technical: platform, integration, performance, data format
- Business: timeline, budget, scope, team
- Legal/Compliance: quy định ngành, bảo mật dữ liệu, audit trail
- Operational: người vận hành, SLA, quy trình vận hành hiện tại

**Bước 3 — Đánh giá hard vs soft**
- Hard: vi phạm là không thể deliver (ví dụ: phải tuân thủ PCI-DSS cho payment)
- Soft: có thể điều chỉnh nếu có lý do đủ mạnh (ví dụ: timeline có thể dời nếu scope giảm)

**Bước 4 — Highlight impact lên solution**
- Với mỗi hard constraint → nêu rõ nó ảnh hưởng gì đến thiết kế
- Với soft constraint → ghi chú điều kiện có thể thay đổi

## Rules

- Không nhầm constraint với requirement (requirement là "cần làm gì", constraint là "phải làm trong giới hạn gì")
- Không đánh giá constraint kỹ thuật chi tiết (architecture, infra) — chỉ ghi nhận ở mức nghiệp vụ
- Hard constraint phải được nêu rõ trước khi đề xuất solution
- Nếu không có đủ thông tin để xác định constraint → ghi là "Cần xác nhận" thay vì bỏ qua

## Output Format

```
## Constraint Analysis

### Hard Constraints — Không thể thay đổi

**Technical**
- [Constraint cụ thể] → *Impact: ...*

**Business**
- [Constraint cụ thể] → *Impact: ...*

**Legal / Compliance**
- [Constraint cụ thể] → *Impact: ...*

**Operational**
- [Constraint cụ thể] → *Impact: ...*

---

### Soft Constraints — Có thể negotiate

- [Constraint] → *Điều kiện thay đổi: ...*

---

### Cần xác nhận

- [Thông tin còn thiếu để xác định constraint]
```

## Failure Cases

- Nhầm constraint với feature requirement → làm lệch hướng phân tích
- Bỏ qua constraint pháp lý vì không được nói rõ trong input → rủi ro compliance
- Không phân biệt hard/soft → solution bị reject vì vi phạm hard constraint có thể tránh được
- Đánh giá quá sâu về kỹ thuật (latency, database index) → vượt phạm vi BA
- Không nêu impact lên solution → constraint được liệt kê nhưng không ai dùng
