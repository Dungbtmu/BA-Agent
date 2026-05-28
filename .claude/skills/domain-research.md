---
name: domain_research
description: Hướng dẫn research domain mới — cách search hiệu quả, đánh giá nguồn, và tổng hợp thành Domain Brief chuẩn để BA bắt đầu dự án
tools: [WebSearch]
---

# Skill: Domain Research & Synthesis

## Mục tiêu

Giúp BA nhanh chóng có đủ kiến thức nền về một domain mới — đủ để đặt câu hỏi đúng với client và không bị lạc trong buổi clarification đầu tiên.

## Input

- Tên domain / lĩnh vực
- Mô tả sơ bộ dự án (nếu có)
- Bối cảnh thị trường (Việt Nam / SEA / global)

## Output

Domain Brief đầy đủ 7 mục, sẵn sàng làm input cho `ba-clarification-agent`.

---

## Thinking Pattern

1. Domain này thuộc lĩnh vực gì? Có sub-domain nào cần đào sâu không?
2. Ai là người dùng cuối — cá nhân hay doanh nghiệp? B2B hay B2C?
3. Quy trình cốt lõi của domain này là gì — nếu không có phần mềm thì họ đang làm tay như thế nào?
4. Điều gì hay gây ra sự khác biệt lớn giữa các doanh nghiệp trong cùng domain?
5. Sản phẩm nào trên thị trường đang giải quyết bài toán này — họ làm tốt điểm gì, yếu điểm gì?

---

## Execution

### Bước 1 — Xác định phạm vi trước khi search

- Domain chính là gì? Có cần thu hẹp sub-domain không?
- Bối cảnh địa lý: ưu tiên Việt Nam / SEA trừ khi BA chỉ định global
- Liệt kê 3–5 keyword search sẽ dùng trước khi bắt đầu — tránh search lan man

### Bước 2 — Search theo thứ tự ưu tiên

| Mục tiêu | Query gợi ý |
|---|---|
| Tổng quan domain | `[domain] how it works overview` / `[domain] là gì` |
| Quy trình nghiệp vụ | `[domain] business process workflow` / `[domain] vận hành như thế nào` |
| Actor / người dùng | `[domain] system users roles stakeholders` |
| Pain point | `[domain] challenges problems pain points` / `[domain] khó khăn thường gặp` |
| Benchmark sản phẩm | `[domain] software solutions popular` / `phần mềm [domain] Việt Nam` |
| Thuật ngữ | `[domain] glossary terminology` / `thuật ngữ [domain]` |

**Quy tắc search:**
- Mỗi mục search tối đa 2–3 lần — nếu không ra kết quả tốt thì đổi keyword, không search lặp vô tận
- Ưu tiên nguồn: trang chuyên ngành, báo cáo thị trường, tài liệu sản phẩm thực tế — tránh nguồn wiki chung chung
- Ghi lại URL nguồn cho thông tin quan trọng

### Bước 3 — Đánh giá và lọc thông tin

Trước khi tổng hợp, lọc theo tiêu chí:
- **Độ tin cậy**: nguồn có uy tín trong ngành không?
- **Độ mới**: thông tin có còn phù hợp với thực tế hiện tại không?
- **Độ phù hợp**: áp dụng được cho bối cảnh dự án (VN/SEA) không?

Nếu tìm thấy thông tin mâu thuẫn giữa các nguồn → ghi cả 2 quan điểm, không tự chọn 1 cái.

### Bước 4 — Tổng hợp Domain Brief

Viết ngắn gọn, thực dụng — mỗi mục tối đa 5–7 bullet points:

**1. Tổng quan domain**
- Domain này làm gì, phục vụ ai
- Các thành phần / module chính thường có
- Đặc thù của thị trường VN/SEA (nếu khác với global)

**2. Các actor điển hình**
- Liệt kê role theo nhóm: người dùng trực tiếp, người quản lý, hệ thống ngoài
- Ghi rõ mỗi role quan tâm đến điều gì nhất

**3. Quy trình nghiệp vụ phổ biến**
- 3–5 quy trình cốt lõi, mô tả ngắn từng bước chính
- Đánh dấu quy trình nào thường được tự động hóa bằng phần mềm

**4. Pain point thường gặp**
- Vấn đề phổ biến mà doanh nghiệp trong domain này hay gặp
- Đây là nguồn để đặt câu hỏi đúng khi gặp client

**5. Thuật ngữ chuyên ngành**
- Tiếng Việt + tiếng Anh tương ứng
- Ưu tiên thuật ngữ hay gây nhầm lẫn hoặc có nhiều cách gọi khác nhau

**6. Benchmark sản phẩm**
- 3–5 sản phẩm / hệ thống điển hình trên thị trường
- Mỗi cái: tên, điểm nổi bật, phân khúc khách hàng
- Ghi URL nếu có

**7. Câu hỏi gợi ý để clarify với client**
- 5–10 câu hỏi BA nên hỏi ngay buổi đầu
- Tập trung vào: scope, actor, quy trình đặc thù của doanh nghiệp này, tích hợp hệ thống ngoài, constraint
- Ưu tiên câu hỏi về những điểm thường khác nhau giữa các doanh nghiệp trong cùng domain

---

## Rules

- **Không bịa** — chỉ viết những gì tìm được từ search hoặc kiến thức có căn cứ; nếu là assumption thì ghi rõ `[Assumption: ...]`
- **Không copy-paste** — tổng hợp bằng ngôn ngữ của mình, không paste nguyên văn từ nguồn
- **Thực tế hơn lý thuyết** — ưu tiên thông tin áp dụng được ngay, không cần giải thích hàn lâm
- **Dẫn nguồn** — ghi URL cho thông tin quan trọng để BA đọc thêm khi cần
- **Kết nối bước tiếp** — cuối Domain Brief ghi: "Bước tiếp theo: dùng file này làm input cho `ba-clarification-agent`"

## Failure Cases

- Search quá rộng, không thu hẹp sub-domain → Domain Brief chung chung, không dùng được
- Chỉ dùng nguồn tiếng Anh global, bỏ qua đặc thù thị trường VN → câu hỏi clarify không phù hợp thực tế
- Liệt kê thuật ngữ không giải thích → BA đọc vẫn không hiểu
- Câu hỏi clarify quá generic ("scope là gì?") → không khai thác được thông tin có giá trị từ client
- Domain Brief quá dài, mỗi mục hàng chục dòng → BA không đọc hết trước buổi meeting
