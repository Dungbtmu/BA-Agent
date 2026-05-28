---
name: user_persona_identification
description: Xác định nhóm người dùng theo hành vi và mục tiêu — xây persona đủ cụ thể để thiết kế user flow chính xác
tools: []
---

# Skill: User & Persona Identification

## Mục tiêu

Xác định đúng user và hành vi của họ để thiết kế user flow phù hợp — tránh build tính năng không ai dùng hoặc miss trường hợp sử dụng quan trọng.

## Input

- `context`: mô tả dự án, bài toán, hoặc stakeholder map đã có
- `domain`: optional — ngành nghề để suy ra user segment điển hình

## Output

- Danh sách user segment với đặc điểm hành vi
- Persona cụ thể cho từng segment chính
- Phân biệt rõ Primary / Secondary user

## Thinking Pattern

1. **Segment theo mục tiêu** — Các nhóm user có mục tiêu khác nhau khi dùng hệ thống không?
2. **Segment theo hành vi** — Tần suất dùng? Cách dùng (mobile/desktop, online/offline)? Tech-savvy hay không?
3. **Segment theo quyền** — Ai có quyền gì? Admin vs regular user vs viewer?
4. **Pain point** — Mỗi nhóm đang gặp khó khăn gì cụ thể? Pain của họ có khác nhau không?
5. **Success condition** — Mỗi nhóm coi là "thành công" khi nào? Metric nào họ quan tâm?
6. **Friction** — Điều gì khiến họ không dùng hoặc dùng sai hệ thống?

## Execution

**Bước 1 — Phân nhóm user**
- Liệt kê tất cả người có thể tương tác với hệ thống
- Nhóm theo mục tiêu và hành vi — không nhóm theo demographics đơn thuần
- Xác định: Primary user (dùng nhiều nhất, ảnh hưởng lớn nhất đến thiết kế) vs Secondary user

**Bước 2 — Validate segment**
- Mỗi segment phải có hành vi đủ khác biệt để cần thiết kế riêng
- Nếu 2 segment dùng hệ thống giống hệt nhau → gộp lại
- Nếu 1 segment có quá nhiều behavior khác nhau → tách ra

**Bước 3 — Xây persona cho Primary users**
- Persona là đại diện điển hình của segment — không phải user thực tế
- Tập trung vào: Goal (muốn đạt gì), Context (dùng trong hoàn cảnh nào), Pain (đang bị cản trở gì)
- Thêm "Jobs to be done" — câu dạng "Khi [tình huống], tôi muốn [hành động] để [kết quả]"

**Bước 4 — Xác định implication cho thiết kế**
- Mỗi persona → nêu rõ 1–2 điều ảnh hưởng trực tiếp đến user flow hoặc UI

## Rules

- Không gộp user có mục tiêu khác nhau vào cùng một persona
- Focus vào behavior, không focus vào demographics (tuổi, giới tính) trừ khi thực sự ảnh hưởng đến UX
- Phân biệt rõ B2B (user là nhân viên trong tổ chức) vs B2C (user là cá nhân)
- Persona phải đủ cụ thể để quyết định: tính năng này có cần không, flow này có hợp lý không
- Không xây quá 3–4 persona chính — quá nhiều thì không ai dùng

## Output Format

```
## User Segments

| Segment | Loại | Mô tả hành vi | Tần suất | Thiết bị |
|---|---|---|---|---|
| [Tên segment] | Primary / Secondary | [Hành vi đặc trưng] | Hàng ngày / Tuần / Tháng | Mobile / Desktop / Cả hai |

---

## Personas

### Persona 1: [Tên đại diện] — [Tên segment]

**Bối cảnh:** [Họ là ai, làm việc trong hoàn cảnh nào]

**Goals:**
- [Mục tiêu chính khi dùng hệ thống]

**Pain Points:**
- [Khó khăn hiện tại cần được giải quyết]

**Jobs to be done:**
- Khi [tình huống], tôi muốn [hành động] để [kết quả mong đợi].

**Implication cho thiết kế:**
- [Điều này nghĩa là flow/UI phải làm gì]

---

### Persona 2: [Tên đại diện] — [Tên segment]
[Tương tự]
```

## Failure Cases

- Tạo persona dựa trên demographics (ví dụ: "nam, 25–35 tuổi") thay vì behavior → không dùng được khi thiết kế flow
- Gộp Admin và End User thành một persona → miss toàn bộ requirement của Admin
- Persona quá chung chung ("người dùng muốn trải nghiệm tốt") → không giúp ra quyết định thiết kế
- Không nêu implication → persona được tạo ra nhưng không ảnh hưởng gì đến output
- Tạo quá nhiều persona → không có persona nào được dùng thực sự
