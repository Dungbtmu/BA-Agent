---
name: urd_srs_structure
description: Orchestrator URD/SRS — cấu trúc tổng thể tài liệu, thứ tự thực hiện các skill con, traceability và checklist trước khi output
tools: []
---

# Skill: URD/SRS Structure (Orchestrator)

## Mục tiêu

Đảm bảo tài liệu URD/SRS được viết đúng cấu trúc, đúng thứ tự, nhất quán và đủ chi tiết để Dev/Tester có thể implement ngay.

---

## Cấu trúc tài liệu chuẩn

```
[TÊN TỔNG CÔNG TY / ĐƠN VỊ]
TÀI LIỆU ĐẶC TẢ YÊU CẦU NGƯỜI DÙNG
[TÊN HỆ THỐNG / MODULE]
[Địa điểm – Tháng MM/YYYY]

CÁC THAY ĐỔI | TRANG KÍ | MỤC LỤC

I.   GIỚI THIỆU
     I.1  Mục đích tài liệu
     I.2  Phạm vi tài liệu
     I.3  Định nghĩa thuật ngữ và từ viết tắt
     I.4  Kiến trúc tổng thể hệ thống

II.  CÁC YÊU CẦU VỀ TỔNG THỂ PHẦN MỀM
     II.1 Sơ đồ quy trình nghiệp vụ (Workflow Diagram)
     II.2 Sơ đồ phân cấp chức năng (Business Function Diagram)
     II.3 Ma trận phân quyền (Permission Matrix)
     II.4 Ma trận ủy quyền (RBAC Matrix)
     II.5 Sơ đồ trình tự (Sequence Diagram)

III. ĐẶC TẢ TÌNH HUỐNG SỬ DỤNG (USE CASE SPECIFICATION)

IV.  GIAO DIỆN CHỨC NĂNG (PROTOTYPE CHÍNH)

C.   YÊU CẦU PHI CHỨC NĂNG
```

---

## Thứ tự thực hiện và skill tương ứng

Các section phụ thuộc nhau theo thứ tự — không được đảo:

```
I.1–I.3  Giới thiệu          → tự viết (không có skill riêng)
I.4      Kiến trúc           → tự viết dựa trên context dự án
   ↓
II.1     Workflow Diagram     → skill: urd-workflow-diagram
   ↓
II.2     Function Tree        → skill: urd-function-tree
   ↓
II.3–II.4 Permission + RBAC  → skill: urd-permission-matrix
   ↓
II.5     Sequence Diagram     → skill: urd-sequence-diagram  (dựa trên II.1)
   ↓
III.     Use Case             → skill: urd-use-case          (dựa trên II.2 + II.3–II.4)
   ↓
IV.      Screen Spec          → skill: urd-screen-spec       (dựa trên III + wireframe)
   ↓
C.       Non-functional       → tự viết dựa trên constraint đã xác định
```

---

## Điều kiện skip từng section

Không phải dự án nào cũng cần đủ 9 section. Dùng bảng này để quyết định bỏ qua section nào — nhưng nếu skip thì **phải ghi rõ lý do** trong tài liệu (không bỏ trắng).

| Section | Skip khi | Bắt buộc khi |
|---|---|---|
| I.4 Kiến trúc tổng thể | Dự án nhỏ, single-layer, không có tích hợp ngoài | Có ≥ 2 hệ thống tích hợp hoặc nhiều layer |
| II.1 Workflow Diagram | Hệ thống chỉ có 1 actor, không có quy trình duyệt/phê duyệt | Có ≥ 2 actor tương tác nhau hoặc có flow phê duyệt |
| II.3–II.4 Permission + RBAC | Chỉ có 1 role duy nhất, không có phân quyền | Có ≥ 2 role với quyền khác nhau |
| II.5 Sequence Diagram | Không có tích hợp API ngoài, không có async flow phức tạp | Có call API ngoài, message queue, hoặc flow multi-step phức tạp |
| C. Non-functional | Prototype nội bộ, không deploy production | Bất kỳ hệ thống nào deploy cho user thực |

**Không được skip:**
- II.2 Function Tree — luôn cần để đảm bảo traceability với Permission Matrix và Use Case
- III. Use Case — luôn cần; đây là core của URD/SRS
- IV. Screen Spec — luôn cần nếu có giao diện người dùng

**Khi skip section:** thay vì bỏ trắng, viết:
```
> **[Không áp dụng]** — [Lý do cụ thể: ví dụ "Hệ thống chỉ có 1 actor duy nhất, không có phân quyền"]
```

---

## Section I — Giới thiệu (tự viết)

### I.1 Mục đích tài liệu
- Hệ thống làm gì (2–3 câu)
- Đối tượng sử dụng tài liệu (BA, Dev, Tester, SA...)
- Vai trò trong vòng đời dự án

### I.2 Phạm vi tài liệu
- Phạm vi chức năng được bao gồm
- Phạm vi không bao gồm (nếu cần làm rõ ranh giới)

### I.3 Định nghĩa thuật ngữ và từ viết tắt

| STT | Thuật ngữ | Diễn giải |
|-----|-----------|-----------|
| 1   | [Tên]     | [Mô tả đầy đủ] |

### I.4 Kiến trúc tổng thể hệ thống

Vẽ Mermaid block diagram thể hiện các layer và luồng tích hợp. Cấu trúc điển hình:

```
Nguồn dữ liệu đầu vào → Lớp tích hợp → Lớp nghiệp vụ (core) → Lớp giao diện → Hệ thống ngoài
```

Sau sơ đồ: diễn giải từng lớp (tên, mục đích, thành phần chính, luồng vào/ra).

---

## Section C — Yêu cầu phi chức năng (tự viết)

Viết theo các nhóm, mỗi yêu cầu phải có con số hoặc tiêu chí cụ thể:

1. **Kiến trúc hệ thống** — mô hình, yêu cầu triển khai
2. **Ràng buộc thiết kế** — CSDL, toàn vẹn dữ liệu, backup, bảo mật DB
3. **Giao diện người dùng** — ngôn ngữ, design system, thông báo lỗi/thành công
4. **An toàn, bảo mật** — xác thực, mã hóa, phân quyền, log thao tác nhạy cảm
5. **Hiệu năng** (nếu có) — thời gian phản hồi, số user đồng thời
6. **Tích hợp** (nếu có) — hệ thống ngoài, giao thức, xử lý lỗi tích hợp

---

## Traceability — Chuỗi truy vết bắt buộc

```
Business Function → Permission Matrix → Use Case → Giao diện
```

Trước khi output, kiểm tra:
- [ ] Mọi chức năng trong Function Tree có trong Permission Matrix
- [ ] Mọi chức năng trong Permission Matrix có ít nhất 1 Use Case
- [ ] Mọi Use Case có ít nhất 1 màn hình giao diện tương ứng
- [ ] Tên Actor nhất quán xuyên suốt (Workflow, UC, RBAC, Giao diện)
- [ ] Tên chức năng nhất quán (Function Tree, Permission Matrix, UC)
- [ ] Thuật ngữ dùng đúng theo bảng I.3

---

## Quy tắc viết dễ đọc, dễ hiểu

Tài liệu URD/SRS được đọc bởi Dev, Tester, PO — không phải chỉ BA. Mỗi đoạn mô tả phải tự giải thích được mà không cần hỏi lại.

### 1. Phân tầng mô tả — luồng chính trước, exception sau

**ĐÚNG:**
> Người dùng nhập số điện thoại → hệ thống kiểm tra định dạng → lưu thành công.
>
> **Exception:** Sai định dạng → hiển thị "Số điện thoại không hợp lệ — phải có đúng 10 chữ số bắt đầu bằng 0".

**SAI (nhồi tất cả vào 1 đoạn):**
> Người dùng nhập số điện thoại; nếu đúng 10 chữ số bắt đầu bằng 0 thì lưu, nếu không thì báo lỗi; không chấp nhận dấu cách, dấu gạch ngang...

Ô bảng có trên 4 rule → tách thành sub-list (`-` hoặc đánh số), không để thành 1 khối text liên tục.

### 2. Viết từ góc nhìn người dùng — không viết theo góc kỹ thuật

Mô tả **điều user thấy / làm**, không mô tả cơ chế kỹ thuật bên trong.

| Tránh | Dùng thay |
|---|---|
| "Hệ thống parse file CSV" | "Sau khi chọn file, màn hình hiển thị preview số dòng hợp lệ / sai định dạng / trùng lặp" |
| "Debounce 300ms trước khi gọi API" | "Kết quả tìm kiếm cập nhật sau khi người dùng dừng gõ" |
| "Backend validate token expiry" | "Nếu phiên đăng nhập hết hạn, người dùng được chuyển về màn hình đăng nhập" |

Thông tin kỹ thuật (debounce, timeout, retry) chỉ viết ở **Section C — Yêu cầu phi chức năng**, không lẫn vào mô tả nghiệp vụ.

### 3. Dùng ví dụ minh họa khi mô tả trừu tượng

Bất kỳ rule nào có thể hiểu sai → thêm ví dụ cụ thể kèm theo.

> **Độ ưu tiên**: số nhỏ = ưu tiên cao hơn. *Ví dụ: Campaign A có độ ưu tiên 1 sẽ được gửi trước Campaign B có độ ưu tiên 5.*

Ví dụ dùng dữ liệu thực tế gần với domain dự án — không dùng `foo/bar`, `A/B`, hay số ngẫu nhiên vô nghĩa.

### 4. Giải thích thuật ngữ nghiệp vụ lạ ngay lần đầu xuất hiện

Thuật ngữ domain-specific hoặc viết tắt nội bộ → giải thích inline lần đầu, sau đó dùng bình thường.

> **T-ALL** (Tất cả khách hàng — không lọc phân khúc): khi QTV không chọn phân khúc nào, hệ thống mặc định gửi đến T-ALL.

Đồng thời bổ sung vào bảng **I.3 Định nghĩa thuật ngữ**.

---

## Checklist chất lượng trước khi output

**Nội dung:**
- [ ] Business Rules trong UC cụ thể, testable — không viết chung chung
- [ ] Validation rule đủ từ cơ bản (bắt buộc, định dạng, độ dài) đến nâng cao (ràng buộc liên field, điều kiện nghiệp vụ)
- [ ] Thông báo lỗi cụ thể — không phải "dữ liệu không hợp lệ"
- [ ] Mỗi đoạn mô tả: luồng chính trước, exception sau — không nhồi thành 1 khối text
- [ ] Ô bảng > 4 rule → tách thành sub-list
- [ ] Mô tả viết từ góc nhìn user — không lẫn mô tả kỹ thuật nội bộ vào nghiệp vụ
- [ ] Rule trừu tượng hoặc dễ hiểu sai → có ví dụ minh họa kèm theo
- [ ] Thuật ngữ domain-specific lạ → giải thích inline lần đầu + bổ sung vào I.3
- [ ] Sequence Diagram có đủ `alt/else` (nhánh hợp lệ + lỗi)
- [ ] Bảng component giao diện đủ: Định dạng, Bắt buộc, Mặc định, Mô tả
- [ ] Log requirement được ghi trong bảng component

**Format:**
- [ ] Đúng cấu trúc section I → II → III → IV → C — không bỏ section nào
- [ ] Sơ đồ Mermaid render được — cú pháp hợp lệ
- [ ] Bảng Markdown đủ cột, không có ô trống không giải thích
- [ ] Không có placeholder vô nghĩa (trừ `[Cần xác nhận: ...]`)
- [ ] Không dùng TBD, N/A trong nội dung hoàn chỉnh

---

## Quy tắc nhất quán toàn tài liệu

- **Không dùng**: TBD, N/A trong nội dung hoàn chỉnh
- **Ghi rõ**: `[Cần xác nhận: ...]` tại chỗ BA chưa có thông tin — không bỏ trống
- **Không bỏ section**: thiếu thông tin → ghi `[Cần xác nhận]`, không bỏ section
