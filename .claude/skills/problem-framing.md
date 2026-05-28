---
name: problem_framing
description: Xác định và chuẩn hóa bài toán từ yêu cầu thô, tránh nhầm lẫn giữa problem và solution
tools: []
---

# Skill: Problem Framing

## Mục tiêu

Chuyển yêu cầu mơ hồ thành bài toán rõ ràng, có thể đo lường.

## Input

- idea: string
- domain_context: optional

## Output

- problem_statement: string
- business_objective: string
- success_metrics: list

## Thinking Pattern

1. Đây là problem hay solution?
2. Ai là user bị ảnh hưởng?
3. Pain point là gì?
4. Root cause hay symptom?
5. Nếu không giải quyết thì sao?
6. Đo thành công bằng gì?

## Execution

- Phân tích input
- Loại bỏ phần mô tả solution nếu có
- Viết lại thành problem rõ ràng
- Xác định objective và metric

## Rules

- Không assume solution
- Problem phải cụ thể
- Metrics phải measurable

## Output Format

```json
{
  "problem_statement": "...",
  "business_objective": "...",
  "success_metrics": ["..."]
}
```
