---
name: problem_framing
description: Xác định và chuẩn hóa bài toán từ yêu cầu thô, tránh nhầm lẫn giữa problem và solution
tools: []
---

# Skill: Problem Framing

## Mục tiêu

Chuyển yêu cầu mơ hồ thành bài toán rõ ràng trước khi đặt câu hỏi clarify — tránh tình trạng đang phân tích solution thay vì problem thực sự.

## Input

- Yêu cầu thô từ client/PO — có thể là mô tả miệng, ghi chú, ý tưởng ban đầu

## Output

- Problem statement rõ ràng: ai đang gặp vấn đề gì, hậu quả là gì
- Business objective: mục tiêu nghiệp vụ cần đạt
- Success metrics: tiêu chí đo thành công — phải có con số hoặc điều kiện cụ thể

---

## Thinking Pattern

1. **Problem hay solution?** — Input đang mô tả "cần làm X" (solution) hay "đang gặp vấn đề Y" (problem)?
2. **Ai bị ảnh hưởng?** — User nào đang chịu pain point này? Có bao nhiêu nhóm?
3. **Pain point thực sự là gì?** — Đằng sau yêu cầu đó, người dùng đang khó chịu điều gì?
4. **Root cause hay symptom?** — Đây là nguyên nhân gốc hay chỉ là biểu hiện của vấn đề sâu hơn?
5. **Hậu quả nếu không giải quyết?** — Business mất gì? User mất gì?
6. **Đo thành công bằng gì?** — Khi nào thì coi là giải quyết được vấn đề?

---

## Execution

**Bước 1 — Nhận diện loại input**

Xác định input đang ở dạng nào:

| Dạng input | Dấu hiệu | Cần làm |
|---|---|---|
| **Solution** | "Làm app X", "Xây tính năng Y", "Tích hợp Z" | Frame lại thành problem trước |
| **Symptom** | "Khách hàng hay phàn nàn về...", "Mất nhiều thời gian khi..." | Đào sâu tìm root cause |
| **Problem rõ** | "Người dùng không thể... dẫn đến..." | Chuẩn hóa và đo lường |
| **Mơ hồ** | "Cải thiện trải nghiệm", "Làm tốt hơn" | Hỏi lại để cụ thể hóa |

**Bước 2 — Loại bỏ solution khỏi mô tả**

Nếu input có mô tả solution → tạm gác sang một bên, tập trung vào "tại sao cần làm điều đó".

Ví dụ:
- Input: "Làm app giao hàng real-time tracking"
- Sau khi frame: "Khách hàng không biết đơn hàng đang ở đâu → lo lắng, gọi CSKH liên tục → tăng chi phí vận hành"

**Bước 3 — Viết Problem Statement**

Cấu trúc chuẩn:
```
[Nhóm người dùng] đang gặp [vấn đề cụ thể] khi [ngữ cảnh/tình huống],
dẫn đến [hậu quả đo lường được].
```

**Bước 4 — Xác định Business Objective**

Mục tiêu nghiệp vụ cần đạt sau khi giải quyết problem — phải gắn với KPI hoặc giá trị cụ thể, không viết chung chung.

- Sai: "Cải thiện trải nghiệm khách hàng"
- Đúng: "Giảm số cuộc gọi CSKH liên quan đến trạng thái đơn hàng xuống 40%"

**Bước 5 — Đặt Success Metrics**

Tối thiểu 2 metric, phải có điều kiện đo được:

- Metric định lượng: tỉ lệ %, thời gian, số lượng
- Metric định tính (nếu cần): có thể verify qua user feedback hoặc observation

---

## Rules

- **Không assume solution** — problem statement không được ngầm định cách giải quyết
- **Problem phải cụ thể** — "trải nghiệm kém" không phải problem; "mất trung bình 15 phút để hoàn thành đơn hàng" mới là problem
- **Metrics phải đo được** — "tốt hơn", "nhanh hơn" không phải metric
- **Một problem statement** — không gộp nhiều problem vào 1 statement; nếu có nhiều problem → liệt kê riêng, ưu tiên theo impact
- **Ghi rõ assumption** — nếu phải tự suy luận để frame problem → ghi `[Assumption: ...]` để BA xác nhận lại với client

---

## Output Format

```
### Problem Statement

[Nhóm người dùng] đang gặp [vấn đề cụ thể] khi [ngữ cảnh],
dẫn đến [hậu quả cụ thể].

### Business Objective

[Mục tiêu nghiệp vụ gắn với KPI hoặc giá trị đo được]

### Success Metrics

- [Metric 1: điều kiện + cách đo]
- [Metric 2: điều kiện + cách đo]

### Assumptions (nếu có)

- [Assumption 1: ...]
```

---

## Failure Cases

- Viết problem statement có ngầm định solution ("người dùng cần app để...") → giới hạn không gian giải pháp sớm
- Dừng lại ở symptom, không tìm root cause → giải quyết sai vấn đề
- Metrics chung chung ("cải thiện satisfaction") → không biết khi nào thì đạt được
- Gộp nhiều problem vào 1 statement → không ưu tiên được, scope bị phình
- Không ghi assumption → BA và client hiểu khác nhau về bài toán mà không biết
