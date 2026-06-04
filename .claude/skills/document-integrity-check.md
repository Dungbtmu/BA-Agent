---
name: document_integrity_check
description: Kiểm tra tính toàn vẹn cấu trúc và traceability của tài liệu URD/SRS sau khi content đã được review — dùng bởi ba-postcheck-agent
tools: []
---

# Skill: Document Integrity Check

## Mục tiêu

Cung cấp thuật toán và tiêu chí cụ thể để kiểm tra tính toàn vẹn **cấu trúc và metadata** của tài liệu URD/SRS đã hoàn chỉnh — đảm bảo tài liệu "sạch" trước khi handoff cho Dev/Tester.

Skill này **không** đánh giá chất lượng nghiệp vụ (đó là việc của `urd-review-checklist`).

---

## Kiểm tra 1 — Section Completeness

### Tiêu chí

Tài liệu URD/SRS hợp lệ phải có đủ tất cả section sau, đúng thứ tự:

```
I.    Giới thiệu
  I.1   Mục đích
  I.2   Phạm vi (bao gồm cả In Scope và Out of Scope)
  I.3   Thuật ngữ và định nghĩa
  I.4   Kiến trúc tổng thể
II.   Yêu cầu tổng thể
  II.1  Workflow Diagram
  II.2  Business Function Diagram (Function Tree)
  II.3  Permission Matrix
  II.4  RBAC Matrix
  II.5  Sequence Diagram
III.  Use Case Specification
IV.   Giao diện chức năng
C.    Yêu cầu phi chức năng
```

### Cách kiểm tra

1. Tìm từng heading section trong tài liệu theo cú pháp Markdown (`##`, `###`)
2. Đánh dấu PASS nếu heading tồn tại và có nội dung (không để trống)
3. Đánh dấu FAIL nếu:
   - Heading không tồn tại trong tài liệu
   - Heading tồn tại nhưng không có nội dung bên dưới
   - Nội dung chỉ là dấu gạch ngang, khoảng trắng, hoặc comment trống

### Kiểm tra bổ sung — `[Cần xác nhận: ...]`

Quét toàn bộ tài liệu tìm chuỗi `[Cần xác nhận`:
- Mỗi lần xuất hiện = 1 mục chưa được giải quyết
- **Trong URD/SRS:** WARN (không phải FAIL) nếu còn `[Cần xác nhận: ...]` — placeholder này hợp lệ trong tài liệu đang làm việc, miễn là được liệt kê rõ trong danh sách "Mục cần xác nhận" cuối tài liệu
- **Trong tài liệu handoff cuối cho Dev/Test:** FAIL nếu còn mục critical chưa xác nhận (ví dụ: actor, phân quyền, quy trình chính còn `[Cần xác nhận]`)
- Liệt kê đầy đủ: section chứa nó, nội dung của placeholder, số lượng, phân loại WARN hay FAIL

### Rules

- Section I.2 FAIL nếu chỉ có In Scope mà không có Out of Scope (hoặc ngược lại)
- Section I.3 FAIL nếu bảng thuật ngữ trống hoặc không có cột Thuật ngữ + Định nghĩa
- Section II.3 và II.4 được coi là 2 bảng riêng — thiếu 1 trong 2 là FAIL
- Section C FAIL nếu không có ít nhất 1 tiêu chí có con số đo được (ví dụ: "< 2 giây")

---

## Kiểm tra 2 — Traceability Audit

### Counting Algorithm — Đếm chức năng lá trong Function Tree

**Định nghĩa chức năng lá**: node trong Function Tree không có node con — tức là chức năng cụ thể nhất, không thể chia nhỏ thêm được.

**Thuật toán đếm:**

```
Bước 1: Xác định cấu trúc cây
  - Tìm heading "Business Function Diagram" hoặc "Cây chức năng"
  - Nhận diện các cấp độ:
      - Cấp 1: Nhóm chức năng chính (ví dụ: "Quản lý người dùng")
      - Cấp 2: Chức năng con (ví dụ: "1.1 Tạo tài khoản")
      - Cấp N: Chức năng lá (không còn dòng con thụt vào sâu hơn)

Bước 2: Nhận diện chức năng lá trong Markdown
  - Nếu dùng bullet list: dòng không có dòng con thụt vào sâu hơn = lá
  - Nếu dùng numbered list: mục cuối cùng trong chuỗi số = lá
      Ví dụ: 1.1.1 là lá nếu không có 1.1.1.1
  - Nếu dùng bảng Mermaid: node không có mũi tên đi ra = lá

Bước 3: Đếm và liệt kê
  - Ghi danh sách tên từng chức năng lá
  - Tổng cộng: N chức năng lá
```

**Ví dụ minh họa:**
```
Quản lý đơn hàng               ← KHÔNG phải lá (có con)
  1.1 Tạo đơn hàng             ← LÁ (không có con)
  1.2 Cập nhật đơn hàng        ← LÁ (không có con)
  1.3 Hủy đơn hàng             ← LÁ (không có con)
Quản lý thanh toán             ← KHÔNG phải lá (có con)
  2.1 Xử lý thanh toán         ← KHÔNG phải lá (có con)
      2.1.1 Thanh toán online   ← LÁ
      2.1.2 Thanh toán COD      ← LÁ
  2.2 Hoàn tiền                ← LÁ

→ Tổng chức năng lá: 6 (1.1, 1.2, 1.3, 2.1.1, 2.1.2, 2.2)
```

### Đếm Permission Matrix

- Đếm số **dòng chức năng** trong bảng Permission Matrix (không đếm dòng header, không đếm dòng nhóm/group nếu có)
- Mỗi dòng chức năng = 1 entry

### Đếm Use Case

- Đếm số **UC ID** duy nhất trong Section III (ví dụ: UC-001, UC-002...)
- Không đếm sub-flow hoặc alternative flow như UC riêng biệt

### Đếm màn hình

- Đếm số **heading màn hình** duy nhất trong Section IV
- Mỗi màn hình = 1 heading cấp 3 hoặc cấp 4 (tùy cách viết tài liệu)
- Không đếm dialog/popup nhỏ được mô tả inline trong màn hình chính là màn hình riêng biệt (trừ khi tài liệu khai báo là màn hình độc lập)

### Tiêu chí PASS/FAIL

| Điều kiện | Kết quả |
|---|---|
| Số chức năng lá == Số entry Permission Matrix | PASS |
| Số chức năng lá > Số entry Permission Matrix | FAIL — có chức năng chưa được phân quyền |
| Số chức năng lá < Số entry Permission Matrix | FAIL — Permission Matrix có entry thừa không tồn tại trong Function Tree |
| Số entry Permission Matrix <= Số UC | PASS (mỗi chức năng có ít nhất 1 UC) |
| Số entry Permission Matrix > Số UC | FAIL — có chức năng chưa có UC |
| Số UC <= Số màn hình | PASS (mỗi UC có ít nhất 1 màn hình) |
| Số UC > Số màn hình | FAIL — có UC chưa có màn hình tương ứng |

**Ghi kết quả theo format:**
```
Function Tree (lá): 12
Permission Matrix (entries): 12  → Delta: 0 ✓
Use Case (UC IDs): 15            → Delta: +3 (bình thường — 1 chức năng có thể có nhiều UC)
Màn hình (Section IV): 10        → Delta: -5 (FAIL — 5 UC thiếu màn hình)
```

---

## Kiểm tra 3 — Naming Consistency

### Phạm vi quét

Quét các thực thể sau xuyên suốt tài liệu:

**Actor names** — tên vai trò người dùng:
- Tìm tất cả cách gọi actor trong: Workflow (swimlane label), Permission Matrix (header cột), RBAC Matrix, Use Case (Actor field), Section IV (mô tả permission, log)
- Flag nếu cùng 1 role có nhiều tên khác nhau

**Function names** — tên chức năng:
- So sánh tên chức năng lá trong Function Tree với: tên dòng trong Permission Matrix, tên UC trong Section III, tên màn hình/tiêu đề trong Section IV
- Flag nếu cùng 1 chức năng có tên khác nhau ở các section

**Screen names** — tên màn hình:
- So sánh tên màn hình trong Section IV với: tên màn hình được đề cập trong UC (Pre/Post-condition, Main Flow), tên màn hình trong Workflow
- Flag nếu cùng 1 màn hình có tên khác nhau

**Thuật ngữ nghiệp vụ**:
- Ưu tiên kiểm tra các thuật ngữ đã định nghĩa trong Section I.3
- Flag nếu từ được định nghĩa trong I.3 nhưng tài liệu dùng từ khác với nghĩa tương đương

### Phân loại lỗi

- **Khác ngôn ngữ**: "Người dùng" vs "User" → FAIL
- **Khác từ cùng nghĩa**: "Đăng ký" vs "Ghi danh" → FAIL
- **Khác format/viết hoa**: "Quản lý đơn hàng" vs "quản lý đơn hàng" → WARNING (không làm FAIL nhưng nên thống nhất)
- **Viết tắt không khai báo**: "QL" thay cho "Quản lý" → FAIL nếu viết tắt chưa được định nghĩa trong I.3

---

## Kiểm tra 4 — Version Hygiene

### Kiểm tra file name

Format chuẩn: `urd-srs-v[N].md` trong đó `[N]` là số nguyên dương (1, 2, 3...).

- FAIL nếu file name không theo format này
- FAIL nếu có ký tự đặc biệt ngoài dấu gạch ngang
- FAIL nếu version dùng chữ thay vì số (ví dụ: `urd-srs-vFinal.md`)

### Kiểm tra header tài liệu

Tìm bảng thông tin tài liệu ở đầu file (thường có các field: Tên tài liệu, Version, Ngày, Tác giả):

- Xác định version trong header (ví dụ: `| Version | 2.0 |` → version là `2`)
- So sánh với số `[N]` trong file name
- FAIL nếu không khớp (ví dụ: file name `urd-srs-v2.md` nhưng header ghi `Version: 1.0`)
- FAIL nếu tài liệu không có header thông tin

### Kiểm tra version history

Tìm bảng lịch sử phiên bản (thường ở cuối Section I hoặc trước Section I):

- FAIL nếu không có bảng version history
- FAIL nếu version hiện tại không có dòng entry trong bảng
- FAIL nếu ngày của version hiện tại là ngày trong tương lai (so với ngày audit)
- WARNING nếu có khoảng nhảy version bất thường (ví dụ: v1 → v3, bỏ v2)

---

## Kiểm tra 5 — Placeholder Detection

### Danh sách pattern cần tìm

**Placeholder tuyệt đối cấm** (FAIL ngay):

| Pattern | Lý do |
|---|---|
| `TBD` (dạng chữ in hoặc thường) | To Be Determined — chưa được quyết định |
| `TODO` (dạng chữ in hoặc thường) | Còn việc chưa làm |
| `...` (ba dấu chấm) trong nội dung bảng hoặc mô tả | Nội dung bỏ lửng |
| `[tên chức năng]`, `[tên màn hình]`, `[tên actor]` | Template chưa được điền |
| Ô trống trong bảng Section IV (bất kỳ cột nào) | Cấm theo output-schema |

**Placeholder có điều kiện** (FAIL nếu dùng sai chỗ):

| Pattern | Cho phép ở đâu | Cấm ở đâu |
|---|---|---|
| `[Cần xác nhận: ...]` | Trong URD/SRS với WARN — xem quy tắc chi tiết ở Kiểm tra 1 | Trong tài liệu handoff Dev/Test nếu là thông tin critical chưa được xử lý |
| `N/A` | Cột Bắt buộc và Mặc định của Label, Button, Icon, Badge trong Section IV | Cột Mô tả của bất kỳ component nào trong Section IV |

### Cách quét

1. Tìm kiếm toàn văn bản theo từng pattern
2. Với mỗi lần xuất hiện: xác định section, tên bảng/bảng, và nội dung xung quanh
3. Phân loại: cấm tuyệt đối hay có điều kiện
4. Ghi vào danh sách lỗi với vị trí đầy đủ

---

## Output Format

```
## Kết quả Document Integrity Check

### Bảng tổng hợp

| # | Kiểm tra | Kết quả | Ghi chú tóm tắt |
|---|---|---|---|
| 1 | Section Completeness | PASS / FAIL | [Số section FAIL, số [Cần xác nhận] còn lại] |
| 2 | Traceability Audit | PASS / FAIL | [Bảng đếm: lá N · PM N · UC N · Screen N] |
| 3 | Naming Consistency | PASS / FAIL | [Số biến thể tên phát hiện] |
| 4 | Version Hygiene | PASS / FAIL | [File name vs header mismatch nếu có] |
| 5 | Placeholder Detection | PASS / FAIL | [Số và loại placeholder còn lại] |

---

### Chi tiết discrepancies

[Chỉ liệt kê các mục FAIL — với vị trí cụ thể và hành động cần sửa]

**[DC-01]** [Kiểm tra N] — [Mô tả lỗi]
- Vị trí: [Section / bảng / dòng]
- Hành động: [Cụ thể cần làm gì]

---

### Phán quyết

**READY FOR HANDOFF** — Tất cả 5 kiểm tra đều PASS.
hoặc
**NEEDS FIX** — [N] kiểm tra FAIL. Cần sửa [N] lỗi trước khi handoff.
```

---

## Rules

- Đếm chính xác theo thuật toán — không ước tính, không làm tròn
- Khi không chắc 1 node có phải lá không → kiểm tra kỹ thụt lề và cấu trúc con; nếu vẫn không rõ → ghi chú và hỏi BA
- Naming consistency: chỉ flag khi KHÁC BIỆT RÕ RÀNG — không flag sự khác biệt về dấu câu không đáng kể
- Version hygiene: so sánh số version, không so sánh format minor (1.0 vs 1 được coi là tương đương nếu số chính khớp)
- Không đánh giá "đúng sai nghiệp vụ" của bất kỳ nội dung nào — chỉ kiểm tra cấu trúc

## Failure Cases

- Đếm nhầm node trung gian thành chức năng lá → kết quả traceability sai
- Flag tất cả `N/A` là lỗi mà không phân biệt cột Bắt buộc/Mặc định với cột Mô tả → false positive
- Bỏ sót section vì heading dùng dấu La Mã khác format (ví dụ: "Phần I" thay vì "I. Giới thiệu") → phải kiểm tra cả 2 pattern
- Không phân biệt `[Cần xác nhận: ...]` với template placeholder khác → nhầm lẫn phân loại
- Quét placeholder không case-insensitive → bỏ sót "tbd", "Tbd", "TBD" là 3 variant của cùng 1 pattern
