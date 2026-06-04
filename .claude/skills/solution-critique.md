---
name: solution_critique
description: Framework phản biện có cấu trúc cho solution BA — tìm điểm yếu, assumption nguy hiểm, scope creep, và alternative đơn giản hơn trước khi đi vào Phase 2
tools: []
---

# Skill: Solution Critique

## Mục tiêu

Cung cấp framework phản biện có cấu trúc để đánh giá chất lượng một solution BA trước khi đầu tư vào wireframe và tài liệu — tránh build sai hướng, build quá phức tạp, hoặc bỏ qua rủi ro quan trọng.

## Input

- `solution`: output từ `ba-solution-agent` — Solution Overview, User Flow, Dependencies, Trade-offs, Assumptions
- `context`: bối cảnh dự án, domain, constraint (nếu có thêm thông tin ngoài solution)

## Output

- Danh sách issues phân loại BLOCKING / MAJOR / MINOR từ 4 lens
- Phán quyết cuối: PASS / PASS WITH CONDITIONS / BLOCK
- Điều kiện cụ thể để tiếp tục (nếu không PASS)

---

## Thinking Pattern

Với mỗi lens, áp dụng tư duy **"Điều gì có thể sai?"** thay vì **"Điều này có đúng không?"** — phản biện chủ động, không thụ động.

1. **User Lens** — Solution có thực sự phù hợp với cách người dùng suy nghĩ và hành động không?
2. **Business Lens** — Solution có đáng làm không? Có cách nào đơn giản hơn không?
3. **Feasibility Lens** — Solution có thể thực hiện được không với những gì hiện có?
4. **Risk Lens** — Điều tệ nhất có thể xảy ra là gì?

---

## Execution

### Lens 1 — User Lens: Flow có khớp mental model thực tế không?

**Câu hỏi gợi ý:**
- Người dùng nào sẽ dùng flow này? Có ai không dùng được không (người mới, người lớn tuổi, người thiếu dữ liệu)?
- Có bước nào mà người dùng sẽ không hiểu cần làm gì tiếp theo?
- Có bước nào mà người dùng phải nhớ thông tin từ bước trước để thực hiện?
- Friction lớn nhất trong flow là ở đâu? Người dùng có thể bỏ cuộc ở điểm nào?
- Flow có đòi hỏi người dùng thay đổi thói quen hiện tại không? Rào cản đó lớn đến mức nào?
- Có actor nào trong nghiệp vụ thực tế nhưng không xuất hiện trong flow không?

**Dấu hiệu cần flag:**
- Flow có bước mơ hồ (người dùng phải tự quyết định cách làm)
- Không có xử lý khi người dùng bỏ giữa chừng
- Một nhóm người dùng quan trọng bị bỏ qua hoàn toàn
- Flow chỉ đúng với happy path, không có alternative cho người dùng edge

---

### Lens 2 — Business Lens: ROI có rõ không? Có cách đơn giản hơn không?

**Câu hỏi gợi ý:**
- Tính năng này giải quyết bài toán nghiệp vụ nào cụ thể? Đo bằng metric gì?
- Nếu không làm tính năng này, business mất gì? Mất bao nhiêu (ước lượng)?
- Có cách nào đơn giản hơn 50% mà vẫn đạt 80% mục tiêu nghiệp vụ không?
- Scope của solution có phình hơn so với bài toán gốc không? Phần nào thêm vào mà không ai yêu cầu?
- Có tính năng nào trong solution mà không có ai thực sự cần không (nice-to-have bị đưa vào must-have)?
- Timeline và effort có tương xứng với business value dự kiến không?

**Dấu hiệu cần flag:**
- Không có metric đo thành công rõ ràng
- Solution giải quyết nhiều vấn đề cùng lúc (scope creep ẩn)
- Có tính năng phức tạp mà không rõ ai sẽ dùng
- Alternative đơn giản hơn chưa được cân nhắc nghiêm túc trong Trade-offs

---

### Lens 3 — Feasibility Lens: Assumption kỹ thuật nào chưa kiểm chứng?

**Câu hỏi gợi ý:**
- Solution đang ngầm giả định hệ thống hiện tại có dữ liệu gì? Đã kiểm chứng chưa?
- Dependency nào trong solution chưa được xác nhận là sẵn sàng (hệ thống thứ ba, API, data source)?
- Có bước nào trong flow mà Dev sẽ hỏi lại "dữ liệu này lấy ở đâu"?
- Có flow nào đòi hỏi real-time hoặc đồng bộ phức tạp mà chưa được đánh giá feasibility?
- Assumption nào trong phần Assumptions của solution được đánh dấu Risky nhưng chưa có plan validate?
- Nếu dependency quan trọng nhất bị fail hoặc delay 2 tháng, flow có bị vỡ không?

**Dấu hiệu cần flag:**
- Assumption đánh dấu Risky mà không có mitigation
- Flow phụ thuộc vào hệ thống ngoài mà chưa có xác nhận từ bên đó
- Dữ liệu cần cho flow chưa rõ là có sẵn hay cần thu thập mới
- Solution giả định nghiệp vụ hiện tại hoạt động đúng như mô tả mà chưa verify

---

### Lens 4 — Risk Lens: Điều gì tệ nhất có thể xảy ra?

**Câu hỏi gợi ý:**
- Nếu người dùng dùng sai cách (không phải ác ý, chỉ nhầm lẫn), hệ thống có safeguard không?
- Edge case nào nguy hiểm nhất cho business (tài chính, dữ liệu, uy tín, compliance)?
- Có scenario nào mà dữ liệu đầu vào hợp lệ nhưng kết quả đầu ra sai về mặt nghiệp vụ?
- Nếu solution này được dùng ở scale lớn hơn dự kiến 10 lần, điều gì sẽ vỡ đầu tiên?
- Có quy định pháp lý hoặc chính sách nội bộ nào mà solution có thể vi phạm không?
- Nếu phải rollback sau khi đã deploy, quy trình có đơn giản không hay sẽ gây hại cho dữ liệu?

**Dấu hiệu cần flag:**
- Không có xử lý khi dữ liệu đầu vào bị lỗi ở giữa flow
- Không rõ ai có quyền hủy/sửa khi có lỗi nghiệp vụ xảy ra sau khi submit
- Solution liên quan đến tiền, sức khỏe, hoặc dữ liệu nhạy cảm nhưng không có audit trail
- Không có kế hoạch khi dependency bị gián đoạn

---

## Phân loại Issue

**BLOCKING**: Nếu không giải quyết, solution không thể triển khai hoặc sẽ fail sau khi go-live.
- Ví dụ: flow giả định dữ liệu mà hệ thống không có; dependency chắc chắn không sẵn sàng; vi phạm compliance rõ ràng

**MAJOR**: Ảnh hưởng đáng kể đến UX hoặc business value — nên sửa trước Phase 2 để tránh wireframe sai hướng.
- Ví dụ: friction lớn trong flow chính; scope phình đáng kể; một nhóm người dùng quan trọng bị bỏ qua; assumption Risky chưa có plan validate

**MINOR**: Gợi ý cải tiến không ảnh hưởng đến tính khả thi — có thể sửa trong hoặc sau Phase 2.
- Ví dụ: alternative đơn giản hơn đáng cân nhắc; cách diễn đạt flow chưa tối ưu; metric thành công cần cụ thể hơn

---

## Rules

- Phán biện phải bám vào nội dung solution thực tế — không phán biện giả định
- Câu hỏi phải cụ thể, có thể trả lời được — không hỏi kiểu "có chắc không?"
- Mỗi issue phải có khuyến nghị hành động — không chỉ nêu vấn đề
- Không đánh giá chất lượng kỹ thuật (code, DB, infra) — đó là phạm vi SA/Dev
- Không re-design solution — chỉ chỉ ra điểm yếu và đề xuất hướng điều chỉnh
- Nếu solution chắc chắn không có issue ở một lens → ghi "Không phát hiện vấn đề đáng kể" thay vì bỏ qua lens đó
- Phán quyết BLOCK phải có ít nhất 1 BLOCKING issue — không BLOCK vì nhiều MAJOR

---

## Output Format

```
## Solution Critique Report

**Solution:** [Tên / mô tả ngắn]

---

### Lens 1 — User

[Phát hiện cụ thể từ góc nhìn người dùng, hoặc "Không phát hiện vấn đề đáng kể"]

### Lens 2 — Business

[Phát hiện cụ thể từ góc nhìn business, hoặc "Không phát hiện vấn đề đáng kể"]

### Lens 3 — Feasibility

[Phát hiện cụ thể từ góc nhìn feasibility, hoặc "Không phát hiện vấn đề đáng kể"]

### Lens 4 — Risk

[Phát hiện cụ thể từ góc nhìn rủi ro, hoặc "Không phát hiện vấn đề đáng kể"]

---

## Issues

| # | Lens | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|---|
| BL-01 | [Lens] | [Mô tả vấn đề cụ thể] | BLOCKING | [Hành động cần làm] |
| MA-01 | [Lens] | [Mô tả vấn đề] | MAJOR | [Hành động khuyến nghị] |
| MI-01 | [Lens] | [Mô tả vấn đề] | MINOR | [Gợi ý cải tiến] |

---

## Phán quyết

**Kết quả:** PASS / PASS WITH CONDITIONS / BLOCK

- BLOCKING: [số] issues
- MAJOR: [số] issues
- MINOR: [số] issues

**Điều kiện để tiếp tục (nếu PASS WITH CONDITIONS hoặc BLOCK):**
1. [Điều kiện cụ thể — hành động cần hoàn thành và cách verify]
2. [...]

**Bước tiếp theo:**
- PASS → `ba-wireframe-agent`
- PASS WITH CONDITIONS → Giải quyết điều kiện → xác nhận BA → `ba-wireframe-agent`
- BLOCK → Quay lại `ba-solution-agent`, tập trung redesign phần: [chỉ định cụ thể]
```

---

## Failure Cases

- Phán biện chung chung kiểu "cần làm rõ hơn" mà không chỉ ra cụ thể cái gì
- Đặt câu hỏi mà không thể trả lời được từ context có sẵn — làm BA mất thời gian
- Flag mọi thứ là BLOCKING vì muốn tỏ ra nghiêm khắc — mất uy tín và làm chậm workflow
- Bỏ qua Lens nào đó vì "không có gì để nói" — mỗi lens phải được xem xét dù kết quả là không có issue
- Đề xuất giải pháp thay thế hoàn toàn khác — đó là phạm vi `ba-solution-agent`, không phải skill này
- Nhầm lẫn giữa feasibility kỹ thuật (performance, security architecture) và feasibility nghiệp vụ (dữ liệu có sẵn, dependency rõ ràng)
