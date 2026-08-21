---
project: CVM
date: 2026-08-21
trigger_source: BA_INPUT
trigger_ref: "Comment nghiệm thu hệ thống CVM — tổng hợp bảng điều chỉnh/bổ sung + các ý bổ sung ngoài bảng"
ba_confirmed: true
---

## Change Set

| # | REQ ID | Change Type | Old Value | New Value | Confidence | match_status | Note |
|---|---|---|---|---|---|---|---|
| 1 | REQ-CVM-020 | MODIFY | Độ ưu tiên chỉ cấu hình tại Priority Matrix (Screen Settings Tab 3); không có cột trên Danh sách Chiến dịch | Bổ sung cột "Ưu tiên trigger" trên Danh sách Chiến dịch (Screen 2); cho phép sửa trực tiếp thứ tự ưu tiên tại đây; sort danh sách theo Trạng thái + Độ ưu tiên; sửa độ ưu tiên phải qua phê duyệt lại (chuyển về Chờ duyệt) | HIGH | MATCHED | |
| 2 | REQ-CVM-007 | MODIFY | Điều kiện lọc phân khúc con khai báo tại Section 3 (Audience), dùng chung cho mọi kênh trong Message Matrix | Chuyển vị trí khai báo điều kiện lọc con của phân khúc từ Section 3 xuống Section 4 (Message Matrix); cho phép setup điều kiện lọc khác nhau theo từng cặp Trigger × Phân khúc × Kênh | HIGH | MATCHED | Thay đổi cấu trúc màn hình Campaign Builder — ảnh hưởng cả REQ-CVM-009 |
| 3 | REQ-CVM-007, REQ-CVM-031 | MODIFY | Giá trị nhập cho điều kiện lọc chỉ hỗ trợ số/ký tự tự do theo kiểu dữ liệu đã khai báo | Bổ sung hỗ trợ đơn vị **%** và **GB** cho giá trị điều kiện lọc — cần định dạng hiển thị và validate theo đơn vị khi nhập | MEDIUM | MATCHED | [ASS-01] Đơn vị % và GB áp dụng cho kiểu dữ liệu Số (Number) hiện có trong Bảng toán tử hợp lệ, không phải kiểu dữ liệu mới |
| 4 | REQ-CVM-009 | MODIFY | Section 4 Message Matrix không có preview riêng theo phân khúc; không hiển thị tên phân khúc tương ứng | Bổ sung xem trước nội dung tin nhắn cho từng Phân khúc; hiển thị tên Phân khúc tương ứng ngay tại mục 4 | HIGH | MATCHED | |
| 5 | REQ-CVM-021, REQ-CVM-025 | MODIFY | Danh sách Mẫu tin nhắn hiển thị dạng bảng phẳng; hành động chỉ có Sao chép/Bật-Tắt, không có Xóa | Nhóm danh sách mẫu tin nhắn theo Trigger; bổ sung hành động **Xóa mẫu tin nhắn** | HIGH | MATCHED | [ASS-02] Chỉ xóa được mẫu tin chưa được campaign nào tham chiếu (kể cả Draft/Ended) — theo nguyên tắc "Dùng" hiện có tại UC-TPL-00; cần BA xác nhận lại khi patch |
| 6 | REQ-CVM-060 | MODIFY | Frequency Cap gồm Daily cap toàn kênh + Cooldown liên campaign, không phân theo kênh, không có mốc tháng | **Bỏ trường Cooldown**; bổ sung giới hạn tần suất theo **từng kênh**; bổ sung thêm dòng tần suất gửi giới hạn theo **tháng** | HIGH | MATCHED | |
| 7 | REQ-CVM-060 | ADD | Không có khái niệm gửi lại sau lần gửi đầu | Cho phép cấu hình **gửi lại** sau lần gửi đầu tiên — tần suất gửi lại (số lần tối đa, khoảng cách tối thiểu giữa các lần) cho cùng 1 đối tượng trong cùng campaign | MEDIUM | MATCHED | Gắn vào REQ-CVM-060 (Frequency Cap) vì cùng nhóm màn hình Cài đặt giới hạn tần suất; [ASS-03] áp dụng ở mức campaign, không phải toàn hệ thống |
| 8 | REQ-CVM-010, REQ-CVM-034 | MODIFY | Blacklist gắn theo campaign: chọn 1 Chiến dịch + 1 Kênh mỗi lần thêm | Cho phép **chọn nhiều Chiến dịch và nhiều Kênh** cùng lúc khi thêm số vào Blacklist | HIGH | MATCHED | |
| 9 | REQ-CVM-034 | ADD | Không có khái niệm Blacklist toàn hệ thống | Bổ sung loại **Blacklist toàn hệ thống** — danh sách độc lập với Blacklist theo từng campaign, số trong danh sách này bị chặn gửi tin ở tất cả campaign và tất cả kênh | HIGH | NEW_CONFIRMED | BA đã xác nhận: danh sách độc lập, không phải chỉ là chọn "Tất cả" trong bộ lọc nhiều campaign/kênh |
| 10 | REQ-CVM-003, REQ-CVM-019 | MODIFY | Ngày kết thúc (endDate) là trường bắt buộc nhập | Bổ sung tùy chọn ngày kết thúc = **Vô hạn** (không bắt buộc nhập endDate) | HIGH | MATCHED | Ảnh hưởng REQ-CVM-015 (background job kiểm tra endDate) và REQ-CVM-018 (blocking issue endDate quá khứ khi Gửi duyệt) — cần loại trừ case Vô hạn khỏi các rule này |
| 11 | REQ-CVM-001, REQ-CVM-014 | MODIFY | Campaign đã duyệt nhưng chưa tới startDate: không có trạng thái/badge phân biệt, hiển thị như Active bình thường | Giữ nguyên trạng thái **Active**; bổ sung **badge "Chưa tới ngày bắt đầu"** hiển thị tại Danh sách Chiến dịch và Chi tiết Chiến dịch khi startDate còn ở tương lai | HIGH | MATCHED | BA xác nhận: chỉ áp dụng case đã duyệt + chưa tới startDate, không gộp case đã qua endDate |
| 12 | REQ-CVM-017, REQ-CVM-032, REQ-CVM-033 | MODIFY | Bật lại từ Tạm dừng (Paused) → Đang chạy (Active) ngay, không cần duyệt lại nếu không còn cờ lỗi | Bổ sung điều kiện: nếu param/điều kiện lọc **đã bị sửa** trong lúc Tạm dừng (không phải Khóa gắn cờ lỗi, mà là BA chủ động mở/sửa param) → Bật lại phải quay về **Chờ duyệt**, không tự động thành Đang chạy. Nếu không có thay đổi gì trong lúc Tạm dừng → giữ nguyên rule cũ (Bật thẳng Đang chạy) | HIGH | MATCHED | BA xác nhận: chỉ áp dụng khi param bị sửa trong lúc Tạm dừng — cần phân biệt rõ với case Khóa param (đã có policy PARAM_INVALID/FILTER_INVALID khóa vĩnh viễn nút Bật riêng) |
| 13 | REQ-CVM-039 | MODIFY | Customer 360 hiển thị đầy đủ thông tin khách hàng (profile, phân khúc, trạng thái kênh, lịch sử nhận tin) không phân biệt che/ẩn theo field | Che (mask/ẩn) các trường thông tin nhạy cảm trên Customer 360 theo nguyên tắc bảo mật dữ liệu | LOW | MATCHED | [ASS-04] BA chưa xác định được danh sách field cụ thể cần che tại thời điểm này ("có thể sẽ nhiều trường hơn, các thông tin dữ liệu quan trọng"). Patch sẽ ghi nguyên tắc chung + để Open Question liệt kê field cụ thể (ứng viên: SĐT đầy đủ, CCCD nếu có, địa chỉ) — BA cần chốt lại với đội bảo mật/Dev trước khi coi là dev-ready |
| 14 | REQ-CVM-009 | ADD | Không kiểm tra ký tự có dấu/không dấu khi soạn nội dung SMS | Kiểm tra ký tự SMS có dấu/không dấu tại bước soạn nội dung — cảnh báo giới hạn: **70 ký tự** (có dấu) / **160 ký tự** (không dấu) | HIGH | NEW_CONFIRMED | Gắn vào REQ-CVM-009 (Message Matrix) vì cùng bước soạn nội dung theo kênh |

**Assumption:**
- [ASS-01] Đơn vị % và GB áp dụng cho kiểu dữ liệu Số hiện có trong Bảng toán tử hợp lệ (REQ-CVM-031), không phát sinh kiểu dữ liệu mới.
- [ASS-02] Chỉ xóa được mẫu tin nhắn chưa được campaign nào tham chiếu — theo nguyên tắc cột "Dùng" hiện có tại UC-TPL-00.
- [ASS-03] Gửi lại sau lần gửi đầu áp dụng ở mức campaign, không phải cấu hình toàn hệ thống.
- [ASS-04] Danh sách field cụ thể cần che tại Customer 360 chưa chốt — patch ghi nguyên tắc + Open Question, không liệt kê cứng field.

**Open Questions phát sinh (đưa vào URD/SRS sau patch):**
- OQ-CVM-NEW-01: Danh sách chính xác các trường thông tin cần che tại Customer 360 là gì? Che hoàn toàn hay mask một phần? Áp dụng theo role nào?
