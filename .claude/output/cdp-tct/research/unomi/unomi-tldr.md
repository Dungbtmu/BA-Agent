# Unomi — Tóm tắt: học được gì, làm gì tiếp

> Đọc file này thay vì đọc `apache-unomi-research.md`. File gốc chứa nguồn và bằng chứng nếu cần tra lại.

---

## Unomi là gì (1 câu)

Unomi là phần mềm CDP mã nguồn mở của Apache — có sẵn nhiều tính năng hay, nhưng đòi hỏi đội có chuyên môn Java mới dùng được.

**Kết luận:** VNPost **không nên dùng Unomi trực tiếp** — nhưng nên học 5 cách nó thiết kế để áp dụng cho CDP của mình.

---

## 5 thứ cần học từ Unomi

### 1. Cách gộp nhiều ID của cùng 1 khách hàng

**Bài toán thực tế:** Cùng 1 người gửi, VNPost có đến 4 mã khác nhau — mã KHL tại quầy, tài khoản app, mã tài chính, PostID. Hiện tại 4 cái này không nối với nhau được.

**Học từ Unomi:** Không cần gộp ngay — tạo "cầu nối alias". Khi khách hàng dùng cùng số điện thoại ở 2 hệ thống, hệ thống tự nhận ra và nối lại thành 1 hồ sơ duy nhất.

**Xem diagram:** [T1 — Identity Layer](diagrams/t1-identity-layer.png)

---

### 2. Cách tự động làm gì đó khi khách hàng có dấu hiệu bất thường

**Bài toán thực tế:** Làm sao hệ thống tự phát hiện KHL sắp rời bỏ, đơn COD có nguy cơ gian lận, hay địa chỉ nhận hàng có vấn đề — mà không cần người ngồi nhìn data thủ công?

**Học từ Unomi:** Thiết kế theo công thức: **Sự kiện xảy ra → Kiểm tra điều kiện → Tự động làm gì đó**. Ví dụ: KHL giảm sản lượng 30% so với 4 tuần trước → tự động đưa vào danh sách cần chăm sóc.

**Quan trọng:** Hệ thống chỉ phát tín hiệu vào hàng đợi — CRM hay SMS tự lo việc gửi. Không để rule gọi thẳng SMS/Zalo (dễ hỏng).

**Xem diagram:** [T2 — Trigger Engine](diagrams/t2-trigger-engine.png)

---

### 3. Cách để Marketing tự thay đổi quy tắc mà không cần IT

**Bài toán thực tế:** Hôm nay "inactive" = 60 ngày không giao dịch. Tháng sau Marketing muốn đổi thành 45 ngày. Hiện tại phải nhờ IT sửa code và release — mất vài tuần.

**Học từ Unomi:** Tách hệ thống thành 3 tầng:
- **Tầng lõi** — IT làm 1 lần, không đụng vào nữa
- **Tầng cấu hình** — Marketing tự điều chỉnh ngưỡng (60 ngày, 30%, 3 lần thất bại...)
- **Tầng kết nối** — cấu hình kênh output: Zalo, SMS, CRM

**Xem diagram:** [T3 — Business Logic Layer](diagrams/t3-business-logic-layer.png)

---

### 4. Cách xin phép và xóa dữ liệu khách hàng đúng luật

**Bài toán thực tế:** Nghị định 13/2023 yêu cầu VNPost phải xin đồng ý trước khi dùng dữ liệu cá nhân. Và nếu khách hàng yêu cầu xóa, phải xóa trong 72 giờ ở tất cả hệ thống.

**Học từ Unomi:** Có 4 loại đồng ý cần quản lý riêng — marketing, phân tích, vị trí, tài chính. Khách hàng có thể bật tắt từng loại. Khi yêu cầu xóa, hệ thống tự động xóa lan sang tất cả hệ thống liên quan.

**Điểm VNPost cần thêm (Unomi chưa có):** Người gửi và người nhận có quyền khác nhau. Người nhận không có tài khoản VNPost — VNPost chỉ dùng thông tin họ để giao hàng, không được dùng cho marketing.

**Xem diagram:** [T4 — Consent Management](diagrams/t4-consent-management.png)

---

### 5. Cách phân quyền để tỉnh A không xem được dữ liệu tỉnh B

**Bài toán thực tế:** Marketing tỉnh Hà Nội có cần xem được data KH ở TP.HCM không? Và KHL lớn như Shopee giao hàng toàn quốc thì xem được bao nhiêu?

**Học từ Unomi:** Mỗi tỉnh là 1 vùng dữ liệu riêng — không bao giờ lẫn. TCT xem được hết. Tỉnh chỉ xem tỉnh mình. Ban lãnh đạo xem báo cáo tổng hợp — không xem thông tin raw của từng người.

**Xem diagram:** [T5 — Scope/Phân quyền](diagrams/t5-scope-permission.png)

---

## Làm gì tiếp theo

Nghiên cứu này phục vụ cho 5 câu hỏi CRITICAL đang chờ PO/client trả lời trong [Domain Brief](domain-brief.md). Khi có trả lời, các pattern T1-T5 ở trên sẽ là đầu vào cho ba-solution-agent thiết kế kiến trúc CDP.

Câu hỏi nào quan trọng nhất cần chốt trước: **Q9** (phân quyền tỉnh/TCT) và **Q6** (nguồn sự kiện real-time) — hai quyết định này ảnh hưởng trực tiếp đến T5 và T2.
