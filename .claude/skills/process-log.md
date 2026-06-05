---
name: process_log
description: Hướng dẫn đọc lại artifacts trong BA session để trích xuất decision, assumption, open question — xây dựng process summary có giá trị thực cho Dev/Tester
tools: []
---

# Skill: Process Log

## Mục tiêu

Đọc toàn bộ artifacts của một BA session theo thứ tự thời gian, trích xuất các yếu tố quan trọng (quyết định, assumption, câu hỏi còn mở, thay đổi scope) và tổng hợp thành Process Summary có giá trị thực cho Dev Lead và Test Lead.

---

## Nguồn dữ liệu và cách đọc từng artifact

### 1. Domain Brief (`.claude/output/[tên_dự_án]/research/domain-brief.md`)

**Đọc để tìm:**
- Assumption ban đầu về domain mà BA chấp nhận để bắt đầu
- Thuật ngữ đã được xác lập — mọi biến thể sau này so với bảng này là inconsistency
- Câu hỏi gợi ý đã đặt ra — sau này đã được trả lời chưa?

**Không đọc để tìm:** Chi tiết domain (không trích xuất vào process summary)

---

### 1b. Domain Gap Analysis (`.claude/output/[tên_dự_án]/research/domain-gap-analysis.md`) *(nếu có)*

**Đọc để tìm:**
- **Gap CRITICAL đã được resolve** — quyết định BA chọn hướng nào để xử lý gap → ghi vào Decision Log
- **Gap CRITICAL chưa được resolve** → ghi vào open questions
- **Assumption phát sinh từ gap** — những gì BA giả định để lấp gap mà không có xác nhận từ client → ghi vào Assumption Register (loại Unverified hoặc Risky)
- **Sub-domain cần research thêm** đã được note nhưng chưa thực hiện → ghi vào open questions

**Không đọc để tìm:** Nội dung mô tả domain điển hình (chỉ quan tâm đến kết quả gap, không phải bảng so sánh)

---

### 2. Clarification output (`.claude/output/[tên_dự_án]/solution/` hoặc notes từ ba-clarification-agent)

**Đọc để tìm:**
- **Scope ban đầu** — tính năng nào được đề cập lần đầu; đây là baseline để so sánh scope delta
- **Câu hỏi CRITICAL đã được trả lời** — trở thành fact (Confirmed assumption)
- **Câu hỏi CRITICAL chưa được trả lời** — trở thành open question hoặc Risky assumption
- **Assumption được BA tuyên bố rõ** — ghi nhận ngay vào Assumption Register

**Cách nhận biết scope ban đầu:** Tìm danh sách tính năng, user flow, hoặc "in scope / out of scope" trong output clarification. Nếu không có danh sách rõ, tổng hợp từ các câu hỏi và câu trả lời.

---

### 3. Solution design (`.claude/output/[tên_dự_án]/solution/`)

**Đọc để tìm:**
- **Quyết định về hướng giải pháp** — BA chọn approach A, từ chối approach B → ghi vào Decision Log
- **Trade-off đã được phân tích** — lý do chọn hướng này thay vì alternatives
- **Edge case được defer** — tính năng/trường hợp được ghi "phase sau" hoặc "không xử lý"
- **Assumption được nêu trong section "Assumptions"** — đây là assumption tường minh, ghi vào Assumption Register

**Cách nhận biết quyết định thực sự** (xem phần Decision Extraction bên dưới)

---

### 4. Wireframe (`.claude/output/[tên_dự_án]/wireframe/`)

**Đọc để tìm:**
- **Màn hình bị defer** — có trong wireframe v1 nhưng biến mất ở version sau
- **Thay đổi layout/flow đáng kể** — từ wireframe v1 đến version cuối có gì thay đổi lớn và tại sao
- **Quyết định UI quan trọng** — ví dụ: dùng modal thay vì page mới, gộp 2 màn hình thành 1
- **Assumption về user behavior** — ví dụ: "user thường làm X trước Y" → kiểm tra có evidence không

**Không cần đọc:** Chi tiết component từng màn hình (đó là trong URD/SRS)

---

### 5. URD/SRS version cuối (`.claude/output/[tên_dự_án]/urd/urd-srs-v[N].md`)

**Đọc để tìm:**
- **Scope thực tế** — Function Tree là nguồn scope cuối chuẩn nhất
- **Assumption ẩn** — chỗ tài liệu nói "hệ thống sẽ..." mà chưa có xác nhận từ PO/client
- **`[Cần xác nhận: ...]` còn sót lại** — đây là open questions chính thức
- **Business Rules phụ thuộc assumption** — "nếu user là X thì..." — X đã được xác nhận chưa?
- **Tích hợp hệ thống ngoài** — assumption về khả năng của hệ thống bên ngoài

**Cách quét nhanh assumption ẩn trong URD/SRS:**
1. Tìm tất cả "[Cần xác nhận" → open questions
2. Tìm câu có dạng "Giả sử...", "Trong trường hợp...", "Mặc định..." → kiểm tra có xác nhận không
3. Tìm chỗ mô tả hành vi của hệ thống ngoài (API, bên thứ 3) → thường là assumption chưa xác nhận
4. Tìm Business Rules không có nguồn gốc rõ — không thấy trong clarification hay solution

---

### 6. QA report (`.claude/output/[tên_dự_án]/urd/qa-report-v[N].md`)

**Đọc để tìm:**
- **CRITICAL issues đã được sửa** → xác nhận tài liệu đã update
- **MAJOR/MINOR chưa sửa** → ghi vào open questions, báo Dev/Tester biết
- **Assumption được reviewer phát hiện** → bổ sung vào Assumption Register nếu chưa có
- **Rủi ro được ghi nhận trong report** → bổ sung vào Handoff Note

---

### 7. Postcheck report (`.claude/output/[tên_dự_án]/urd/postcheck-report.md`)

**Đọc để tìm:**
- **Kết luận READY FOR HANDOFF** — xác nhận điều kiện bàn giao đã đủ chưa
- **Điểm lưu ý cuối cùng cho Dev/Tester** — thường có trong section cuối của postcheck
- **Issues được accept (chấp nhận giữ nguyên)** — phải ghi vào Handoff Note để Dev/Tester biết

---

## Decision Extraction — Cách phân biệt quyết định thực sự

### Quyết định thực sự là gì

Một **quyết định** phải thỏa mãn cả 3 tiêu chí:
1. Có **ít nhất 2 lựa chọn** tồn tại tại thời điểm đó (không phải quyết định hiển nhiên)
2. BA/PO **chủ động chọn** một hướng và từ chối các hướng khác
3. **Ảnh hưởng đến scope, flow, hoặc behavior** của hệ thống

**Ví dụ đúng:**
- "Quyết định không build tính năng export PDF trong phase 1 — defer sang phase 2"
- "Chọn flow 1 bước thay vì 2 bước vì user đã xác thực trước đó"
- "Scope giới hạn ở quản lý nội bộ — không tích hợp với hệ thống bên ngoài trong v1"

**Ví dụ không phải quyết định (đừng đưa vào Decision Log):**
- "Hệ thống cho phép user đăng nhập" — hiển nhiên
- "Màu nút là xanh" — chi tiết UI, không ảnh hưởng nghiệp vụ
- "Ba lần nhập sai mật khẩu sẽ khóa tài khoản" — đây là Business Rule, không phải quyết định về direction

### Nguồn quyết định thường xuất hiện ở đâu

| Vị trí trong artifacts | Dấu hiệu quyết định |
|---|---|
| Solution design — section Trade-offs | "Chọn hướng A vì..." |
| Solution design — section Assumptions | "Giả định rằng... để giữ scope gọn" |
| Clarification output | "Câu hỏi về X → PO trả lời sẽ không làm" |
| Wireframe notes | "Bỏ màn hình X, gộp vào Y" |
| QA report — CRITICAL issues | "Sửa flow theo hướng B thay vì A vì..." |
| `[Cần xác nhận: ...]` được resolve | Khi BA quyết định dùng một hướng mà không chờ confirm |

---

## Handoff Note Template — Format chuẩn

Handoff Note được viết cho 2 đối tượng:

### Dành cho Dev Lead

Những điều Dev Lead cần biết trước khi phân công sprint:

```
## Handoff Note — Dev Lead

### Phải làm rõ với BA trước sprint 1
[Liệt kê H01, H02... — các điểm còn mơ hồ ảnh hưởng trực tiếp đến implement]

### Assumption trong tài liệu mà Dev cần theo dõi
[Liệt kê assumption Risky/Unverified ảnh hưởng đến code — kèm section URD/SRS liên quan]

### Open questions chưa có câu trả lời
[Liệt kê Q01, Q02... — ai cần trả lời, deadline mong muốn]

### Tính năng defer — KHÔNG implement trong lần này
[Liệt kê DEF01, DEF02... — để tránh Dev tự ý build thêm]
```

### Dành cho Test Lead

Những điều Test Lead cần biết trước khi viết test plan:

```
## Handoff Note — Test Lead

### Chỗ tài liệu dùng assumption — viết test case cần hỏi lại
[Liệt kê section + assumption + câu hỏi cụ thể nên hỏi BA trước khi viết test case]

### MAJOR/MINOR issues trong QA report chưa được sửa
[Liệt kê từ QA report — Test Lead cần biết những chỗ này có thể chưa chuẩn]

### Edge case được BA chủ động bỏ qua
[Liệt kê edge case nào BA đã biết nhưng quyết định không xử lý — và lý do]
```

---

## Scope Delta Method — Cách so sánh scope

### Bước 1 — Xác định scope ban đầu

Nguồn scope ban đầu (theo thứ tự ưu tiên):
1. Section "In Scope / Out of Scope" trong clarification output
2. Danh sách tính năng trong solution design
3. Epic list trong ba-backlog-agent output (nếu có)
4. Danh sách câu hỏi CRITICAL đã được trả lời trong clarification

Lưu dưới dạng danh sách tính năng/luồng nghiệp vụ cụ thể.

### Bước 2 — Xác định scope cuối

Nguồn scope cuối:
1. Function Tree trong Section II.2 của URD/SRS version cuối — đây là nguồn chuẩn nhất
2. Danh sách màn hình trong Section IV

### Bước 3 — So sánh và phân loại

So sánh item-by-item:

| Trường hợp | Phân loại | Ghi vào |
|---|---|---|
| Có trong scope cuối, không có trong scope đầu | Scope creep | Bảng "Tính năng được thêm vào" |
| Có trong scope đầu, không có trong scope cuối | Bị loại | Bảng "Tính năng bị loại bỏ" |
| Có trong cả hai nhưng mô tả thay đổi đáng kể | Thay đổi ý nghĩa | Bảng "Tính năng thay đổi ý nghĩa" |
| Có trong cả hai, mô tả tương đồng | Không thay đổi | Không ghi vào bảng |

### Bước 4 — Đánh giá mức độ scope delta

- **Không có scope creep**: Scope cuối ≈ Scope đầu, sai lệch < 10%
- **Scope creep nhẹ**: 1–2 tính năng được thêm, không ảnh hưởng timeline đáng kể
- **Scope creep đáng kể**: 3–5 tính năng thêm, hoặc 1 module lớn được thêm
- **Scope creep nghiêm trọng**: > 5 tính năng thêm, hoặc thêm nhóm user mới, hoặc tích hợp hệ thống ngoài không có trong plan ban đầu

---

## Output Format — Cấu trúc file `process-summary.md`

File `process-summary.md` bao gồm 5 phần theo thứ tự:

```
1. Decision Log       — Bảng quyết định (D01, D02, ...)
2. Assumption Register — Bảng assumption (A01, A02, ...) + tóm tắt Confirmed/Unverified/Risky
3. Handoff Note       — 3.1 Điểm cần làm rõ + 3.2 Open questions + 3.3 Assumption trong tài liệu + 3.4 Tính năng defer
4. Scope Delta        — Bảng thêm/loại/thay đổi + đánh giá mức độ
5. Tóm tắt trạng thái — Bảng hạng mục + checklist điều kiện bàn giao
```

Nguyên tắc viết:
- **Ngắn gọn, actionable** — mỗi dòng trong bảng phải là thông tin Dev/Tester có thể hành động được
- **Không copy-paste từ URD/SRS** — tóm tắt, không sao chép
- **Không dùng từ chuyên môn BA** không cần thiết — viết như đang nói chuyện với Dev
- **Ghi ID nhất quán** — D01, A01, H01, Q01, DEF01 để dễ tra cứu

---

## Rules

- Chỉ trích xuất từ artifacts đã có — không bịa thêm assumption hoặc quyết định
- Mỗi entry trong Decision Log phải có thể truy ngược về artifact nguồn
- Assumption "Confirmed" phải có evidence hoặc ghi rõ ai xác nhận và khi nào
- Không đánh giá lại chất lượng tài liệu — đó là việc của `ba-qa-agent`
- Nếu một artifact không tồn tại → ghi "N/A — [artifact] không có trong session này", tiếp tục
- Handoff Note phải viết cho người không tham gia BA session — không dùng ngữ cảnh ngầm định

## Failure Cases

- Liệt kê quyết định hiển nhiên (không có alternatives thực sự) vào Decision Log → làm loãng thông tin
- Copy toàn bộ Assumptions section từ solution design vào mà không phân loại lại → mất giá trị
- Handoff Note viết bằng ngôn ngữ BA/PO nội bộ → Dev/Tester không hiểu được
- Scope delta chỉ so sánh số lượng tính năng mà không so sánh nội dung → bỏ sót scope creep ẩn
- Bỏ qua assumption ẩn trong URD/SRS (không tường minh) → đây là nguồn rủi ro cao nhất
- Không ghi "ảnh hưởng nếu thay đổi" trong Decision Log → Dev không biết quyết định nào quan trọng
- Gộp open questions và assumption vào cùng 1 bảng → mất rõ ràng về loại thông tin
