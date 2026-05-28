# 📋 PROJECT TRACKING — CVM (Customer Value Management System)

> Cập nhật lần cuối: 07/05/2026 | BA: Jun

---

## I. THÔNG TIN DỰ ÁN

| Hạng mục | Nội dung |
|---|---|
| Tên dự án | CVM — Customer Value Management System |
| Mô tả | Hệ thống quản lý campaign marketing tự động cho viễn thông ảo |
| BA | Jun |
| Trạng thái | 🟡 Đang phân tích |

---

## II. THEO DÕI TÀI LIỆU BA

| STT | Tài liệu | Trạng thái | Version | Ngày hoàn thành | Ghi chú |
|---|---|---|---|---|---|
| 1 | Solution Design | ✅ Xong | v1 | 06/05/2026 | Bao gồm Template Engine + Blacklist |
| 2 | Wireframe | ✅ Xong | v4 | 07/05/2026 | Single page Campaign Builder, 9 màn hình |
| 3 | Lovable Prompt | ✅ Xong | v4 | 07/05/2026 | Sẵn sàng để gen prototype |
| 4 | URD/SRS | ✅ Xong | v1 | 07/05/2026 | Còn 3 mục cần xác nhận (xem Section III) |
| 5 | Backlog | ⏳ Chưa làm | — | — | — |
| 6 | Epic chi tiết | ⏳ Chưa làm | — | — | — |

**Chú thích trạng thái:**
- ✅ Xong — Hoàn chỉnh, sẵn sàng review
- 🔄 Đang làm — Đang trong quá trình
- ⏳ Chưa làm — Chưa bắt đầu
- ⚠️ Cần cập nhật — Có thay đổi cần sync lại

---

## III. THEO DÕI VẤN ĐỀ / ISSUES

| ID | Vấn đề | Loại | Liên quan đến | Người xử lý | Deadline | Trạng thái | Ghi chú |
|---|---|---|---|---|---|---|---|
| CVM-01 | Danh sách đầy đủ Status Code từ từng Gateway (Zalo OA, SMS, USSD) để CVM xử lý đúng logic sync-back và fallback | ❓ Cần xác nhận | URD/SRS — Section C | SA / Dev | — | 🔴 Open | Ảnh hưởng đến logic fallback kênh |
| CVM-02 | Fallback value mặc định cho tham số động khi BSS/OCS không trả về giá trị (VD: {{ten_kh}} = "Quý khách"?) | ❓ Cần xác nhận | URD/SRS — Section C | PO / SA | — | 🔴 Open | Ảnh hưởng đến nội dung tin nhắn khi dữ liệu KH thiếu |
| CVM-03 | Cơ chế Gateway thông báo KH đã unblock Zalo (để CVM auto reset zalo_blocked = false) | ❓ Cần xác nhận | URD/SRS — Section C | SA / Dev | — | 🔴 Open | Nếu Gateway không hỗ trợ → cần cơ chế manual reset |
| CVM-04 | Bảng map Lĩnh vực ↔ Hành vi vi phạm (nếu CVM có module này) | ❓ Cần xác nhận | Segment Builder | PO | — | 🟡 Chờ PO | Hiện tại Segment chỉ lọc theo data KH từ BSS/OCS |
| CVM-05 | Giới hạn tối đa số campaign Active cùng lúc (nếu có) | ❓ Cần xác nhận | Campaign Engine | PO / SA | — | 🟡 Chờ PO | Ảnh hưởng đến performance khi nhiều trigger hit cùng lúc |

**Chú thích trạng thái:**
- 🔴 Open — Chưa có câu trả lời, đang block
- 🟡 Chờ PO/SA/Dev — Đã raise, đang chờ phản hồi
- 🟢 Resolved — Đã có câu trả lời, đã cập nhật tài liệu
- ⬛ Closed — Không áp dụng / bỏ qua có lý do

---

## IV. THEO DÕI THAY ĐỔI YÊU CẦU

| ID | Ngày | Nội dung thay đổi | Yêu cầu bởi | Tác động | Tài liệu cần cập nhật | Trạng thái |
|---|---|---|---|---|---|---|
| CHG-01 | 06/05/2026 | Template tin nhắn quản lý trong CVM (không phụ thuộc hệ thống ngoài) | PO/Jun | Thêm Template Engine, màn hình Template Management | Solution v1, Wireframe v2→v3 | ✅ Đã cập nhật |
| CHG-02 | 06/05/2026 | Mỗi trigger có payload riêng; QTV chọn trigger chính làm nguồn tham số | Jun (phân tích) | Thay đổi UX Campaign Builder, cột trái hiện payload | Wireframe v3 | ✅ Đã cập nhật |
| CHG-03 | 07/05/2026 | Preview nằm trong tab kênh (song song textarea), không ở cột phải | Jun (feedback UX) | Layout Campaign Builder thay đổi | Wireframe v4, Lovable v4 | ✅ Đã cập nhật |
| CHG-04 | 07/05/2026 | Cấu hình gửi đổi từ radio → listbox dropdown | Jun (feedback UX) | UX Section 3 Campaign Builder | Wireframe v4, Lovable v4 | ✅ Đã cập nhật |
| CHG-05 | 07/05/2026 | Bổ sung Blacklist Management (trang riêng) + sync-back trạng thái kênh | Jun (phân tích) | Thêm Screen 6 Blacklist, cập nhật Customer 360 | Wireframe v4, URD/SRS v1 | ✅ Đã cập nhật |

---

## V. LỊCH SỬ PHIÊN BẢN TÀI LIỆU

| Tài liệu | V1 | V2 | V3 | V4 |
|---|---|---|---|---|
| Solution | 06/05 — Base + Template Engine | — | — | — |
| Wireframe | Stepper 7 bước | Single page 3 cột | Bổ sung Trigger payload | 2 cột + Preview trong tab + Blacklist |
| Lovable Prompt | Stepper | 3 cột + Template Mgmt | Trigger payload | 2 cột + Preview per channel + Blacklist |
| URD/SRS | 07/05 — Full document | — | — | — |
