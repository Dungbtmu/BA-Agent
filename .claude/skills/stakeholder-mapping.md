---
name: stakeholder_mapping
description: Xác định đầy đủ stakeholder, vai trò, mức độ ảnh hưởng và nhu cầu — tránh miss requirement từ nhóm bị bỏ sót
tools: []
---

# Skill: Stakeholder Mapping

## Mục tiêu

Xác định đầy đủ tất cả stakeholder liên quan đến hệ thống — cả người dùng trực tiếp lẫn bên liên quan gián tiếp — để không bỏ sót requirement từ bất kỳ nhóm nào.

## Input

- `context`: mô tả dự án, bài toán, hoặc tài liệu yêu cầu
- `domain`: optional — ngành nghề để suy ra stakeholder điển hình

## Output

- Danh sách stakeholder với vai trò, interest, influence và nhu cầu cốt lõi
- Ma trận Power/Interest để ưu tiên engagement

## Thinking Pattern

1. **Primary users** — Ai thực sự dùng hệ thống hàng ngày? Họ làm gì với nó?
2. **Decision makers** — Ai phê duyệt, ai có quyền thay đổi requirement?
3. **Payers** — Ai trả tiền? Nhu cầu của họ có khác user trực tiếp không?
4. **Operators** — Ai vận hành, maintain hệ thống sau khi go-live?
5. **Indirect stakeholders** — Ai bị ảnh hưởng bởi output của hệ thống mà không trực tiếp dùng?
6. **External parties** — Đối tác, cơ quan quản lý, hệ thống third-party có ràng buộc không?
7. **Conflict** — Nhóm nào có lợi ích mâu thuẫn nhau? Cần xử lý trade-off gì?

## Execution

**Bước 1 — Liệt kê stakeholder**
- Bắt đầu từ user trực tiếp, mở rộng dần ra ngoài
- Dùng các câu hỏi trong Thinking Pattern để không bỏ sót
- Với B2B: phân biệt rõ Admin (bên mua) và End User (bên dùng)
- Với B2C: phân biệt user theo segment hành vi, không chỉ theo demographics

**Bước 2 — Mô tả từng stakeholder**
- Role: họ là ai trong hệ thống
- Interest: họ muốn gì từ hệ thống
- Key requirements: nhu cầu cốt lõi cần đáp ứng
- Pain points hiện tại: đang gặp khó khăn gì mà hệ thống cần giải quyết

**Bước 3 — Đánh giá Power/Interest**
- Power (ảnh hưởng đến quyết định): High / Medium / Low
- Interest (mức độ quan tâm đến outcome): High / Medium / Low
- Phân nhóm: Manage Closely / Keep Satisfied / Keep Informed / Monitor

**Bước 4 — Xác định conflict**
- Nếu 2 nhóm có lợi ích mâu thuẫn → ghi rõ conflict và đề xuất cách xử lý

## Rules

- Bao gồm cả internal (trong tổ chức) và external (đối tác, khách hàng, cơ quan quản lý)
- Không bỏ sót nhóm vận hành — họ hay bị miss nhưng có requirement quan trọng về usability
- Phân biệt rõ "người dùng" và "người mua" trong B2B — nhu cầu thường khác nhau
- Không mô tả chung chung như "người dùng cuối" — phải có tên nhóm cụ thể
- Conflict giữa các stakeholder phải được nêu rõ, không giấu

## Output Format

```
## Stakeholder Map

### Primary Stakeholders

| Stakeholder | Role | Interest | Power | Key Requirements |
|---|---|---|---|---|
| [Tên nhóm] | [Vai trò] | H/M/L | H/M/L | [Nhu cầu cốt lõi] |

### Secondary Stakeholders

| Stakeholder | Role | Interest | Power | Ghi chú |
|---|---|---|---|---|
| [Tên nhóm] | [Vai trò] | H/M/L | H/M/L | [Bị ảnh hưởng như thế nào] |

---

### Power/Interest Matrix

- **Manage Closely** (High Power, High Interest): [Danh sách]
- **Keep Satisfied** (High Power, Low Interest): [Danh sách]
- **Keep Informed** (Low Power, High Interest): [Danh sách]
- **Monitor** (Low Power, Low Interest): [Danh sách]

---

### Conflict cần xử lý

- [Nhóm A] vs [Nhóm B]: [Mô tả conflict] → [Đề xuất hướng xử lý]
```

## Failure Cases

- Chỉ liệt kê user trực tiếp, bỏ qua nhóm vận hành và decision maker → miss requirement quan trọng
- Gộp chung nhiều nhóm user khác nhau thành "người dùng" → mất thông tin về nhu cầu khác biệt
- Không phát hiện conflict giữa stakeholder → solution bị reject muộn trong project
- Mô tả Interest/Power thiếu căn cứ → ma trận không có giá trị để ưu tiên
- Bỏ qua external stakeholder (cơ quan quản lý, đối tác tích hợp) → miss compliance requirement
