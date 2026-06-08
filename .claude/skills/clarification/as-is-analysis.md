---
name: as_is_analysis
description: Phân tích hiện trạng thực tế dự án (hệ thống cũ, quy trình đang chạy, constraint hiện có) — thu hẹp phạm vi Clarification bằng cách xác định rõ cái gì đã biết, cái gì cần giữ, cái gì cần thay đổi
tools: []
---

# Skill: As-Is Analysis

## Mục tiêu

Khi BA cung cấp thông tin về hiện trạng thực tế (hệ thống cũ, quy trình đang chạy, dữ liệu hiện có, constraint kỹ thuật/nghiệp vụ), skill này giúp:

- Xác định rõ **cái gì đã biết** — không cần hỏi lại trong Clarification
- Xác định **cái gì phải giữ nguyên** — constraint không thể thay đổi
- Xác định **cái gì cần thay đổi** — điểm đau, gap giữa hiện tại và mục tiêu
- Thu hẹp phạm vi câu hỏi cho `ba-clarification-agent` — chỉ hỏi những gì thực sự chưa biết

Output là **As-Is Summary** — dùng làm input bổ sung cho `ba-clarification-agent` song song với `domain-gap-analysis`.

---

## Input bắt buộc

Hiện trạng do BA cung cấp — có thể ở bất kỳ dạng nào:

- Mô tả miệng: "Hệ thống cũ đang dùng Excel, quy trình duyệt đơn qua email"
- Tài liệu có sẵn: quy trình nội bộ, SOP, màn hình chụp hệ thống cũ
- Ghi chú từ buổi khảo sát hiện trạng với client
- Danh sách pain point do BA hoặc PO liệt kê

Nếu BA không cung cấp hiện trạng → **không chạy skill này**. Bỏ qua và để `ba-clarification-agent` chạy bình thường với `domain-gap-analysis`.

---

## Thinking Pattern

1. Hệ thống / quy trình hiện tại đang làm được gì? Ai đang dùng? Dùng như thế nào?
2. Điểm nào đang hoạt động tốt — client muốn giữ lại?
3. Điểm nào đang gây đau — client muốn thay đổi?
4. Có constraint nào không thể thay đổi không? (hệ thống legacy phải tích hợp, quy định nội bộ, dữ liệu cũ phải migrate...)
5. Thông tin nào từ hiện trạng đã trả lời được câu hỏi mà Clarification định hỏi?

---

## Execution

### Bước 1 — Trích xuất từ hiện trạng

Đọc toàn bộ input hiện trạng, trích xuất và phân loại:

```
Actors hiện tại:      [Ai đang tham gia quy trình hiện tại]
Quy trình hiện tại:   [Các bước đang được thực hiện như thế nào]
Công cụ/hệ thống:     [Đang dùng gì — Excel, phần mềm X, email, giấy tờ...]
Dữ liệu hiện có:      [Dữ liệu gì đang được lưu, ở đâu, format nào]
Constraint cứng:      [Điều gì không thể thay đổi — pháp lý, hệ thống legacy, ngân sách...]
Pain point rõ ràng:   [Điều gì đang gây vấn đề, client đã nói rõ]
```

### Bước 2 — Phân loại theo trạng thái TO-BE

Với mỗi thành phần trong hiện trạng, đánh dấu:

- **KEEP** — giữ nguyên trong hệ thống mới (quy trình đang tốt, không cần thay)
- **IMPROVE** — giữ nhưng cải thiện (logic đúng nhưng thực hiện kém hiệu quả)
- **REPLACE** — thay thế hoàn toàn (đang gây pain, cần làm lại)
- **MIGRATE** — dữ liệu hoặc quy trình cần migrate sang hệ thống mới
- **CONSTRAINT** — không thể thay đổi dù muốn (ghi rõ lý do)

### Bước 3 — Xác định thông tin đã biết

Liệt kê những câu hỏi mà hiện trạng đã trả lời — để `ba-clarification-agent` KHÔNG hỏi lại:

```
Đã biết:
- Actor chính: [X, Y, Z] — xác nhận từ hiện trạng
- Quy trình duyệt: [mô tả] — đang chạy như vậy, muốn giữ logic nhưng tự động hóa
- Dữ liệu cần migrate: [danh sách] — đã có, cần import vào hệ thống mới
```

### Bước 4 — Xác định điểm còn mở

Những gì hiện trạng CHƯA trả lời — đây là phạm vi thực sự cần Clarification hỏi:

```
Còn mở (cần Clarification hỏi):
- [Câu hỏi 1]: hiện trạng không đề cập đến [X]
- [Câu hỏi 2]: có mâu thuẫn giữa [A] và [B] trong mô tả hiện trạng
- [Câu hỏi 3]: pain point [Y] đã rõ nhưng chưa biết client muốn giải quyết theo hướng nào
```

---

## Output — As-Is Summary

```markdown
## As-Is Summary — [Tên dự án]

### Hiện trạng tóm tắt

| Thành phần | Hiện trạng | Trạng thái TO-BE | Ghi chú |
|---|---|---|---|
| [Actor A] | [Mô tả hiện tại] | KEEP / IMPROVE / REPLACE | |
| [Quy trình X] | [Mô tả hiện tại] | IMPROVE | Pain point: [...] |
| [Hệ thống Y] | Excel / phần mềm Z | REPLACE | Cần migrate dữ liệu |
| [Rule nghiệp vụ B] | [Mô tả] | CONSTRAINT | Lý do: quy định pháp lý |

---

### Đã biết — KHÔNG cần hỏi lại trong Clarification

- **Actor**: [danh sách] đã xác nhận
- **Quy trình cốt lõi**: [mô tả ngắn] — giữ logic, cải thiện cách thực hiện
- **Dữ liệu**: [loại dữ liệu] cần migrate từ [nguồn]
- **Constraint**: [danh sách] — cứng, không thể thay đổi

---

### Còn mở — Clarification cần tập trung hỏi

| # | Điểm còn mở | Lý do chưa rõ | Mức độ |
|---|---|---|---|
| 1 | [Câu hỏi cụ thể] | [Hiện trạng không đề cập / mâu thuẫn / ...] | CRITICAL |
| 2 | [Câu hỏi cụ thể] | [...] | MAJOR |
| 3 | [Câu hỏi cụ thể] | [...] | MINOR |

---

### Pain point đã xác nhận

| Pain point | Mức độ ảnh hưởng | Hướng giải quyết mong muốn (nếu client đã nói) |
|---|---|---|
| [Pain 1] | Cao / Trung bình / Thấp | [Nếu có] |
| [Pain 2] | ... | [Cần hỏi thêm] |
```

---

## Cách `ba-clarification-agent` dùng output này

Khi có As-Is Summary, `ba-clarification-agent` phải:

1. **Đọc phần "Đã biết"** trước — loại bỏ những câu hỏi không cần thiết
2. **Ưu tiên phần "Còn mở"** — đây là danh sách câu hỏi đã được lọc, tập trung vào đó
3. **Dùng pain point** để contextualize câu hỏi — không hỏi chung chung mà hỏi có dẫn dắt từ hiện trạng
4. **Kết hợp với `domain-gap-analysis`** — gap từ domain chuẩn + điểm mở từ hiện trạng = danh sách câu hỏi clarify tổng hợp, không trùng lặp

---

## Rules

- Không phán xét "hiện trạng sai" hay "không chuyên nghiệp" — chỉ ghi nhận thực tế
- CONSTRAINT phải ghi rõ lý do — không ghi chung chung "không thể thay đổi"
- Phần "Đã biết" phải thực sự rõ ràng từ input — không tự suy luận rồi ghi là đã biết
- Nếu input hiện trạng mâu thuẫn nội bộ → flag ngay trong output, không tự chọn một phiên bản
- Giới hạn "Còn mở": tối đa 3 CRITICAL, 5 MAJOR — nếu nhiều hơn thì gộp các điểm liên quan

## Failure Cases

- Ghi nhận thông tin chưa chắc vào "Đã biết" → Clarification bỏ qua câu hỏi quan trọng
- Không flag mâu thuẫn trong hiện trạng → Clarification hỏi nhầm chiều
- CONSTRAINT không có lý do → Dev/BA sau này không biết tại sao không thể thay đổi
- Output quá dài, không tóm gọn → `ba-clarification-agent` đọc không hiệu quả
