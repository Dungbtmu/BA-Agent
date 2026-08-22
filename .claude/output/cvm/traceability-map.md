# Traceability Map — CVM (Customer Value Management System)

> **Lưu ý:** Dự án CVM hiện chưa có Epic/User Story/Backlog (`.claude/output/CVM/` không có thư mục `backlog/`) — cột **Story** để trống toàn bộ map này.
> Baseline được tạo từ `urd-srs-v3.md` (2589 dòng, đã qua QA) — dùng làm nền tính impact cho các thay đổi phát sinh từ comment nghiệm thu sắp xử lý.

## Traceability Matrix — CVM
Version: 3 | Cập nhật: 2026-08-21

| REQ ID | Mô tả ngắn | UC | WF | URD Section | Story | Ghi chú |
|---|---|---|---|---|---|---|
| REQ-CVM-001 | Quy trình tổng thể tạo và vận hành campaign (đăng nhập → cấu hình → gửi duyệt → duyệt → chạy → dừng/kích hoạt lại) | UC-CAM-02, UC-CAM-05, UC-CAM-06, UC-CAM-07, UC-CAM-08 | Screen 2, Screen 3, Screen Admin | URD-II.1 | | Workflow diagram tổng — là khung tham chiếu cho toàn bộ Khối 1 |
| REQ-CVM-002 | Xem danh sách campaign — tìm kiếm, filter trạng thái, phân trang, hành động theo trạng thái | UC-CAM-01 | Screen 2 | URD-III-UC-CAM-01, URD-IV-Screen2 | | |
| REQ-CVM-003 | Tạo campaign mới — cấu hình đầy đủ thông tin cơ bản, trigger, audience, nội dung, kênh & lịch, an toàn; Lưu Nháp hoặc Gửi duyệt; **hỗ trợ endDate = Vô hạn (v4)** | UC-CAM-02 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 | | REQ lõi nhất hệ thống — gộp nhiều field nhưng đây là 1 hành vi nghiệp vụ thống nhất (tạo campaign); các quy tắc phụ tách REQ riêng bên dưới |
| REQ-CVM-004 | Mã kịch bản tự sinh theo rule `CVM-YYYYMM-SEQ4`, chỉ đọc, đảm bảo unique | UC-CAM-02 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 (Header Fixed STT 3, Section 1 STT 2) | | |
| REQ-CVM-005 | Chế độ trigger Basic (1 trigger) / Advanced (nhiều trigger + Logic OR/AND); chuyển đổi giữa 2 chế độ có confirm khi mất dữ liệu | UC-CAM-02, UC-CAM-03 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 (Section 2) | | |
| REQ-CVM-006 | Audience Variant (biến thể đối tượng theo phân khúc) — chỉ khả dụng khi đồng thời Logic trigger OR và Logic phân khúc OR; chuyển đổi logic có confirm xóa | UC-CAM-02 | Screen 3 | URD-I.3, URD-III-UC-CAM-02, URD-IV-Screen3 (Section 2 STT 4, Section 4 STT 13-14) | | Định nghĩa thuật ngữ tại I.3.1 mục 10 gắn liền quy tắc nghiệp vụ này |
| REQ-CVM-007 | Điều kiện lọc **theo Trigger × Phân khúc × Kênh (v4 — chuyển từ Section 3 xuống Section 4 Message Matrix)** — thuộc tính lấy động theo trigger đã chọn (group theo trigger, không gộp trùng tên), toán tử khai báo thẳng theo thuộc tính, giá trị render theo kiểu (hỗ trợ đơn vị % và GB cho kiểu Số — v4), validate BETWEEN "đến" ≥ "từ"; kế thừa mặc định khi thêm kênh mới; mỗi accordion vẫn giữ logic AND cố định giữa các điều kiện lọc con (không đổi qua các version) — thay đổi thực sự là **phạm vi áp dụng**: từ 1 bộ điều kiện dùng chung mọi kênh (Section 3) sang riêng theo từng kênh (Section 4) | UC-CAM-02 | Screen 3, Screen 2B | URD-III-UC-CAM-02, URD-IV-Screen3 (Section 4 STT 4b), URD-IV-Screen2B (Section 4 STT 7) | | Liên quan chặt UC-TRG-05 (nơi khai báo thuộc tính lọc); v3 ở Section 3, v4 chuyển sang Section 4 theo comment nghiệm thu; **v4.5** — rà soát toàn diện đóng nốt 4 vị trí còn sót mô hình cũ (Screen 3 STT 3, UC-CAM-02 Hoạt động, Screen 2B STT 6/7, UC-TRG-05); đồng thời bỏ toàn bộ Reach ước tính (Estimate box, Section 3/6) vì hệ quả trực tiếp — không còn 1 con số ước tính chung đủ chính xác khi điều kiện lọc đã tách theo kênh |
| REQ-CVM-008 | Kênh & Lịch gửi — lịch chung/riêng per kênh, đồng bộ 2 chiều với Message Matrix, Blackout xử lý Hủy luôn/Hoãn đến đầu khung giờ | UC-CAM-02 | Screen 3 | URD-II.6.5, URD-III-UC-CAM-02, URD-IV-Screen3 (Kênh & Lịch gửi) | | |
| REQ-CVM-009 | Soạn nội dung Message Matrix theo kênh (Push/Zalo/SMS/USSD/Banner/Email) — thêm/xóa tab kênh, PARAMS chips, template dropdown, preview realtime, giới hạn ký tự/ảnh theo kênh; **preview + tên phân khúc theo từng phân khúc (v4)**; **SMS tự phát hiện có dấu (70 ký tự/segment) / không dấu (160 ký tự/segment) (v4)** | UC-CAM-02 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 (Section 4) | | |
| REQ-CVM-010 | Cấu hình An toàn — DNC mặc định bật (confirm khi tắt), Blacklist/Whitelist campaign 3 hình thức (Không dùng/Chọn từ danh sách/Upload tệp), đồng bộ 2 chiều với Blacklist Management | UC-CAM-02 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 (Section 6) | | |
| REQ-CVM-011 | Validate tham chiếu param trong nội dung message tại Lưu Nháp và Gửi duyệt — chặn cả 2 thao tác nếu param không thuộc trigger đã chọn (PARAM_INVALID nguồn 2) | UC-CAM-02, UC-CAM-08 | Screen 3 | URD-III-UC-CAM-02 (bước 7a), URD-III-UC-CAM-08, URD-Quy-tắc-Khối-3 | | Bổ sung V3.35 — 1 trong 2 nguồn phát sinh cờ PARAM_INVALID |
| REQ-CVM-012 | Sửa campaign đang Draft — pre-fill dữ liệu, cảnh báo trigger bị tắt, hiển thị banner PARAM_INVALID/FILTER_INVALID | UC-CAM-03 | Screen 3 | URD-III-UC-CAM-03, URD-IV-Screen3 (Header Fixed STT 7) | | |
| REQ-CVM-013 | Xem chi tiết campaign chỉ đọc — toàn bộ cấu hình, nút hành động biến đổi theo trạng thái | UC-CAM-04 | Screen 2B | URD-III-UC-CAM-04, URD-IV-Screen2B | | |
| REQ-CVM-014 | Duyệt / Từ chối campaign bởi Admin — phê duyệt chuyển Active, từ chối chuyển Draft kèm lý do bắt buộc | UC-CAM-05 | Screen Admin | URD-III-UC-CAM-05, URD-IV-Screen-Admin | | |
| REQ-CVM-015 | Background job kiểm tra endDate — tự động chuyển Ended cho campaign Active/Pending/Paused quá hạn, hủy message trong queue | UC-CAM-05 | — | URD-II.6.4, URD-III-UC-CAM-05 | | Logic nội bộ không phơi UI — Dev implement theo II.6.4 |
| REQ-CVM-016 | Dừng campaign ngay lập tức (Kill Switch) — confirm bắt buộc, hủy toàn bộ message trong queue, không thu hồi message đã Delivered | UC-CAM-06 | Screen 2, Screen 2B | URD-III-UC-CAM-06, URD-IV-Screen2 (STT 8), URD-IV-Screen2B (STT 3) | | |
| REQ-CVM-017 | Kích hoạt lại campaign từ Paused — không cần duyệt lại nếu không có cờ lỗi và không có param/điều kiện lọc trigger bị sửa trong lúc Paused (**v4** — nếu có sửa thì chuyển về Chờ duyệt thay vì Active thẳng); bị khóa vĩnh viễn nếu còn PARAM_INVALID/FILTER_INVALID, chỉ resume qua [Sửa] → Draft → gửi duyệt lại | UC-CAM-07 | Screen 2, Screen 2B | URD-III-UC-CAM-07, URD-IV-Screen2 (STT 9), URD-IV-Screen2B (STT 3) | | |
| REQ-CVM-018 | Gửi duyệt campaign — danh sách đầy đủ issue blocking (tên trống, thời gian hiệu lực, endDate quá khứ **[loại trừ case Vô hạn — v4]**, BETWEEN ngược, PARAM_INVALID cả 2 nguồn, FILTER_INVALID...) | UC-CAM-08 | Screen 3 | URD-III-UC-CAM-08, URD-IV-Screen3 (Header Fixed STT 6) | | REQ tổng hợp toàn bộ điều kiện chặn Gửi duyệt — điểm neo quan trọng để tính impact khi thêm/bớt 1 loại issue |
| REQ-CVM-019 | Enddate không được ở trong quá khứ — blocking issue riêng khi Gửi duyệt (**không áp dụng cho campaign chọn Vô hạn — v4**); startDate vẫn cho phép quá khứ | UC-CAM-02, UC-CAM-08 | Screen 3 | URD-III-UC-CAM-02, URD-III-UC-CAM-08, URD-IV-Screen3 (Section 1 STT 4) | | |
| REQ-CVM-020 | Độ ưu tiên campaign — mặc định = max priority của campaign Active + 1, cho phép trùng số, tie-break theo created_at; **sửa priority trực tiếp trên Campaign List (v4) — bắt buộc chuyển về Chờ duyệt, khác Priority Matrix (Admin tự sắp xếp không cần duyệt lại)** | UC-CAM-01, UC-CAM-02, UC-PRIORITY-01 | Screen 2, Screen 3, Screen Settings | URD-II.6.8, URD-III-UC-CAM-01, URD-III-UC-CAM-02, URD-III-UC-PRIORITY-01, URD-IV-Screen2 (STT 7), URD-IV-Screen3 (Section 1 STT 5), URD-IV-Screen-Settings (Tab 3) | | v4: UC-CAM-01 bổ sung vào cột UC vì đã có luồng sửa priority tại Campaign List |
| REQ-CVM-021 | Xem danh sách template — tìm kiếm, filter kênh/trạng thái, cột Dùng (đếm campaign tham chiếu kể cả Draft/Ended); **nhóm hiển thị theo Trigger áp dụng (v4)** | UC-TPL-00 | Screen 4A | URD-III-UC-TPL-00, URD-IV-Screen4A | | |
| REQ-CVM-022 | Tạo template mới — nhiều tab kênh, chip Global Params (union payload trigger Active), preview realtime không giá trị mẫu; tab kênh trống là blocking issue; **trường Trigger áp dụng không bắt buộc (v4) — chỉ để nhóm hiển thị, không giới hạn phạm vi dùng** | UC-TPL-01 | Screen 4B | URD-III-UC-TPL-01, URD-IV-Screen4B | | |
| REQ-CVM-023 | Xem chi tiết template chỉ đọc — layout 2 cột, toàn bộ input disabled | UC-TPL-02 | Screen 4C | URD-III-UC-TPL-02, URD-IV-Screen4C | | |
| REQ-CVM-024 | Sửa template — template đang dùng trong campaign Active vẫn sửa được, không ảnh hưởng campaign Active (đã copy tại thời điểm tạo) | UC-TPL-03 | Screen 4B | URD-III-UC-TPL-03 | | |
| REQ-CVM-025 | Sao chép template (tạo bản sao độc lập) / Bật-Tắt template (Inactive ẩn khỏi dropdown chọn khi soạn campaign) | UC-TPL-04 | Screen 4A | URD-III-UC-TPL-04, URD-IV-Screen4A (STT 8) | | |
| REQ-CVM-026 | Xem danh sách trigger — bảng phẳng, filter Kiểu chạy + Trạng thái, phân quyền hành động khác nhau QTV/Admin | UC-TRG-00 | Screen 5, Screen Admin (Tab Trigger) | URD-III-UC-TRG-00, URD-IV-Screen5A, URD-IV-Screen-Admin (Tab Trigger) | | |
| REQ-CVM-027 | Xem chi tiết trigger — Nhóm A Định danh (chỉ đọc mọi role), Nhóm B Tham số đầu ra, Nhóm C Điều kiện lọc phân khúc | UC-TRG-02 | Screen 5B, Screen Admin (T-DETAIL) | URD-III-UC-TRG-02, URD-IV-Screen5B, URD-IV-Screen-Admin (T-DETAIL) | | |
| REQ-CVM-028 | Khai báo trigger mới bởi Admin — Trigger Code/Tên/Kiểu chạy/Nguồn sự kiện bắt buộc, tùy chọn khai báo tham số + điều kiện lọc ngay trong form | UC-TRG-03 | Screen Admin (T-NEW) | URD-III-UC-TRG-03, URD-IV-Screen-Admin (T-NEW) | | |
| REQ-CVM-029 | Thêm/Sửa/Khóa tham số đầu ra của trigger — không xóa cứng; sửa Tên giữ nguyên techName (không gắn cờ); Khóa qua dialog xác nhận (gắn cờ PARAM_INVALID) | UC-TRG-04 | Screen Admin (T-DETAIL Nhóm B) | URD-III-UC-TRG-04, URD-IV-Screen-Admin (Nhóm B) | | |
| REQ-CVM-030 | Thêm/Sửa/Khóa điều kiện lọc phân khúc của trigger — không xóa cứng; sửa Tên giữ nguyên techName (không gắn cờ); sửa Kiểu/Toán tử/Giá trị hoặc Khóa gắn cờ FILTER_INVALID | UC-TRG-05 | Screen Admin (T-DETAIL Nhóm C) | URD-III-UC-TRG-05, URD-IV-Screen-Admin (Nhóm C) | | |
| REQ-CVM-031 | Bảng toán tử hợp lệ theo kiểu dữ liệu (14 toán tử × 5 nhóm kiểu) — ràng buộc form khai báo điều kiện lọc và render lưới an toàn tại Campaign Builder; **đơn vị % và GB cho kiểu Số (v4)** | UC-TRG-05 | Screen Admin (T-DETAIL/T-NEW), Screen 3 | URD-III-Bảng-toán-tử-hợp-lệ, URD-IV-Screen3 (Section 4 STT 4b) | | |
| REQ-CVM-032 | Policy PARAM_INVALID (nguồn 1 — trigger thay đổi sau khi campaign đã cấu hình) — điều kiện áp dụng cờ, auto-Paused cho Active/Pending, chỉ gắn cờ cho Draft/Paused/Ended, thông báo nội bộ, khóa vĩnh viễn nút Bật | UC-TRG-04, UC-CAM-07, UC-CAM-08 | Screen 3, Screen 2, Screen 2B, Screen Admin | URD-Quy-tắc-Khối-3, URD-III-UC-CAM-07, URD-III-UC-CAM-08 | | Policy nghiệp vụ trung tâm — nhiều UC/Screen phụ thuộc |
| REQ-CVM-033 | Policy FILTER_INVALID (song song PARAM_INVALID cho điều kiện lọc) — điều kiện áp dụng cờ, auto-Paused, thông báo nội bộ, khóa vĩnh viễn nút Bật | UC-TRG-05, UC-CAM-07, UC-CAM-08 | Screen 3, Screen 2, Screen 2B, Screen Admin | URD-Quy-tắc-Khối-3, URD-III-UC-CAM-07, URD-III-UC-CAM-08 | | |
| REQ-CVM-034 | Xem danh sách Blacklist — filter Campaign/Kênh/Phạm vi (v4); **gộp hiển thị theo Số điện thoại, 1 dòng/số, chip Campaign/Kênh, bỏ cột Nguồn (v4.9)**; đồng bộ 2 chiều với campaign; hiển thị cả bản ghi Blacklist toàn hệ thống (v4, luôn tách dòng riêng) | UC-BL-00 | Screen 6A | URD-III-UC-BL-00, URD-IV-Screen6A | | |
| REQ-CVM-035 | Thêm thủ công số vào Blacklist — nhập nhiều số, validate realtime, xử lý case 0 số được lưu mới (không đóng modal, báo lỗi rõ); **chọn nhiều Campaign + nhiều Kênh cùng lúc, tạo N×M bản ghi (v4)** | UC-BL-01 | Screen 6B | URD-III-UC-BL-01, URD-IV-Screen6B | | |
| REQ-CVM-036 | Upload CSV vào Blacklist — file mẫu, parse preview (Hợp lệ/Sai định dạng/Trùng), tối đa 100.000 dòng; **chọn nhiều Campaign + nhiều Kênh cùng lúc, tạo N×M bản ghi (v4)** | UC-BL-02 | Screen 6C | URD-III-UC-BL-02, URD-IV-Screen6C | | |
| REQ-CVM-037 | Xóa số khỏi Blacklist — confirm dialog, áp dụng cặp campaign-kênh cụ thể hoặc bản ghi toàn hệ thống (**v4**); **2 lớp xóa (v4.9)**: [×] trên chip gỡ đúng 1 tổ hợp, [Xóa] toàn dòng gỡ mọi tổ hợp của số trong phạm vi đang xem; không ảnh hưởng DNC BSS | UC-BL-03 | Screen 6A | URD-III-UC-BL-03, URD-IV-Screen6A (STT 8) | | |
| REQ-CVM-038 | Xem danh sách & tìm kiếm khách hàng — dữ liệu chỉ đọc từ BSS/OCS, filter Trạng thái SIM/Cài app; **Số điện thoại che một phần theo role (v4)** | UC-KH-00 | Screen 7 | URD-III-UC-KH-00, URD-IV-Screen7 | | |
| REQ-CVM-039 | Xem Customer 360 — profile, phân khúc, trạng thái kênh, throttling, lịch sử nhận tin (kể cả fallback 2 dòng); **che thông tin nhạy cảm theo nguyên tắc bảo mật, tối thiểu Số điện thoại (v4) — danh sách đầy đủ field còn OQ-11 chờ PO/bảo mật chốt** | UC-KH-01 | Screen 8 | URD-III-UC-KH-01, URD-IV-Screen8 | | |
| REQ-CVM-040 | Xem Báo cáo & Xuất Excel — bộ lọc 4 chiều dùng chung, so sánh kỳ trước tự tính, 6 tab phân tích | UC-RPT-01 | Screen 9 | URD-III-UC-RPT-01, URD-IV-Screen9 | | |
| REQ-CVM-041 | Quy tắc nguồn dữ liệu Report — 3 nguồn (message_log/Gateway callback/BSS event join), kênh không đo được hiển thị N/A | UC-RPT-01 | Screen 9 | URD-III-UC-RPT-01 (Nguồn dữ liệu), URD-IV-Screen9 | | |
| REQ-CVM-042 | Tab 1 Hiệu quả gửi tin — Sent/Delivered/Delivery Rate/Failed/Failure Rate | UC-RPT-01 | Screen 9 (Tab 1) | URD-III-UC-RPT-01 (Tab 1), URD-IV-Screen9 (Tab 1) | | |
| REQ-CVM-043 | Tab 2 Tương tác — Opened/Open Rate/Clicked/CTR/Converted/Conversion Rate, phân biệt kênh đo được theo metric | UC-RPT-01 | Screen 9 (Tab 2) | URD-III-UC-RPT-01 (Tab 2), URD-IV-Screen9 (Tab 2) | | |
| REQ-CVM-044 | Tab 3 So sánh Campaign — Sent/Delivered/Open Rate/Conversion Rate ngang hàng, click áp dụng filter | UC-RPT-01 | Screen 9 (Tab 3) | URD-III-UC-RPT-01 (Tab 3), URD-IV-Screen9 (Tab 3) | | |
| REQ-CVM-045 | Tab 4 Phân khúc — Reach (snapshot audience_size)/Delivered/Open Rate/Conversion Rate theo phân khúc | UC-RPT-01 | Screen 9 (Tab 4) | URD-III-UC-RPT-01 (Tab 4), URD-IV-Screen9 (Tab 4) | | |
| REQ-CVM-046 | Tab 5 Phễu chuyển đổi — 6 bước phễu, công thức rời bỏ, gợi ý theo bước rời bỏ lớn nhất | UC-RPT-01 | Screen 9 (Tab 5) | URD-III-UC-RPT-01 (Tab 5), URD-IV-Screen9 (Tab 5) | | |
| REQ-CVM-047 | Tab 6 Spam & Quá tải — Opt-out/Opt-out Rate/Blacklist mới/tần suất trung bình, ngưỡng cảnh báo riêng từng chỉ số (< 3% / 3-4.9% / ≥5%) | UC-RPT-01 | Screen 9 (Tab 6) | URD-III-UC-RPT-01 (Tab 6), URD-IV-Screen9 (Tab 6) | | |
| REQ-CVM-048 | Xuất Excel — 3 sheet (Metadata/Dữ liệu chính/Kỳ trước), quy tắc đặt tên file | UC-RPT-01 | Screen 9 | URD-III-UC-RPT-01 (Xuất Excel) | | |
| REQ-CVM-049 | Dashboard vận hành realtime — 7 thẻ KPI, sức khỏe hệ thống, campaign monitoring, phân tích trigger; cập nhật polling 60s + stream realtime | UC-DSH-01 | Screen 1 | URD-III-UC-DSH-01, URD-IV-Screen1 | | |
| REQ-CVM-050 | Row 1 — 7 KPI Cards, công thức và ngưỡng cảnh báo từng thẻ (đồng nhất khung "hôm nay" 00:00) | UC-DSH-01 | Screen 1 (Row 1) | URD-III-UC-DSH-01 (Row 1), URD-IV-Screen1 (ROW 1) | | |
| REQ-CVM-051 | Row 2 — Sức khỏe hệ thống: biểu đồ 3 nhóm trigger 24h, Hàng đợi & Tồn đọng (Pending blackout/Scheduled future/Oldest pending với ngưỡng cam/đỏ) | UC-DSH-01 | Screen 1 (Row 2) | URD-III-UC-DSH-01 (Row 2), URD-IV-Screen1 (ROW 2) | | |
| REQ-CVM-052 | Row 3 — Campaign Monitoring: bảng Top Campaign, bảng Top Trigger (toggle Hôm nay/7 ngày), Dòng sự kiện trigger realtime (FIFO 100 dòng, Đang chạy/Tạm dừng) | UC-DSH-01 | Screen 1 (Row 3) | URD-III-UC-DSH-01 (Row 3), URD-IV-Screen1 (ROW 3) | | |
| REQ-CVM-053 | Row 4 — Phát hiện bất thường trigger: công thức % lệch so với trung bình 7 ngày, ngưỡng > 200% | UC-DSH-01 | Screen 1 (Row 4) | URD-III-UC-DSH-01 (Row 4), URD-IV-Screen1 (ROW 4) | | |
| REQ-CVM-054 | Cấu hình Priority Matrix — Admin kéo thả/nhập số sắp xếp thứ tự ưu tiên campaign Active, cảnh báo trùng số, tiebreak created_at | UC-PRIORITY-01 | Screen Settings (Tab 3) | URD-III-UC-PRIORITY-01, URD-IV-Screen-Settings (Tab 3) | | |
| REQ-CVM-055 | Logic Pipeline Kênh — thứ tự xử lý, kiểm tra trạng thái kênh, gửi qua Gateway, ghi log kết quả (Dev-internal) | — | — | URD-II.6.1 | | Không phơi UI — không gắn UC/Screen theo đúng bản chất mục II.6; [Cần xác nhận] có nên gắn UC-CAM-02 làm nguồn gián tiếp không |
| REQ-CVM-056 | Fallback kênh — thứ tự mặc định Push→Zalo→SMS→USSD→Banner→Email, chỉ fallback kênh có nội dung, không fallback khi DNC/Blacklist | — | — | URD-II.6.2 | | Dev-internal — [Cần xác nhận] liên kết UC nguồn |
| REQ-CVM-057 | Retry khi Gateway lỗi — phân loại lỗi tạm thời/vĩnh viễn, exponential backoff, lưu cả failure_reason và gateway_status_code | — | — | URD-II.6.3 | | Dev-internal |
| REQ-CVM-058 | Điều kiện dừng pipeline — 8 điều kiện (trước startDate, Delivered, hết kênh, DNC/BL, SIM không hợp lệ, Throttle, Kill Switch, endDate) | UC-CAM-06 | — | URD-II.6.4, URD-III-UC-CAM-06 | | Bổ sung V3.8/V3.9 — liên kết Kill Switch (UC-CAM-06) và background job endDate |
| REQ-CVM-059 | Sync-back trạng thái kênh và lịch sử nhận tin về Customer 360 sau mỗi lần Gateway trả kết quả | UC-KH-01 | Screen 8 | URD-II.6.6, URD-III-UC-KH-01, URD-IV-Screen8 (STT 6, 7) | | |
| REQ-CVM-060 | Throttling & Frequency Cap — Daily/Weekly/Monthly cap toàn kênh + cap riêng theo từng kênh, để trống = không giới hạn (**v4** — bỏ Cooldown, thêm Weekly/Monthly + theo kênh), hiển thị tại Customer 360 | UC-KH-01 | Screen 8, Screen Settings (Tab 1) | URD-II.6.7, URD-III-UC-KH-01, URD-IV-Screen8 (STT 4), URD-IV-Screen-Settings (Tab 1) | | |
| REQ-CVM-061 | Cross-campaign Priority — chọn 1 campaign khi nhiều campaign cùng match 1 KH, theo priority score + tiebreak created_at | UC-PRIORITY-01 | Screen Settings (Tab 3) | URD-II.6.8, URD-III-UC-PRIORITY-01 | | |
| REQ-CVM-062 | Deduplication Event — chống xử lý trùng event dựa trên event_id, TTL 24 giờ | — | — | URD-II.6.9 | | Dev-internal, không có UC/Screen tương ứng — [Cần xác nhận] |
| REQ-CVM-063 | Quy tắc ánh xạ trạng thái SIM từ BSS sang CVM (Active/Suspended/Inactive) — riêng cho SIM vật lý và eSIM | UC-KH-00, UC-KH-01 | Screen 7, Screen 8 | URD-II.7, URD-III-UC-KH-00, URD-III-UC-KH-01, URD-IV-Screen7 (STT 6), URD-IV-Screen8 (STT 2) | | |
| REQ-CVM-064 | Nguồn dữ liệu xác định "Đã cài app" — từ BSS `app_install_log`, CVM không lưu, query realtime | UC-KH-00, UC-KH-01 | Screen 7, Screen 8 | URD-II.7.4, URD-III-UC-KH-00, URD-III-UC-KH-01 | | Có Open question SA/Dev về API/batch — [Cần xác nhận] |
| REQ-CVM-065 | **Nhắc lại (Re-engagement)** — sau khi gửi tin THÀNH CÔNG cho KH theo 1 trigger, nếu KH vẫn thoả điều kiện trigger gốc tại thời điểm kiểm tra thì gửi thêm 1-N lần nhắc; cấu hình riêng theo TỪNG campaign tại Campaign Builder (không phải System Settings): bật/tắt, số lần tối đa, khoảng cách tối thiểu (giờ); khác với Retry kỹ thuật (chỉ xảy ra khi gửi Failed) | UC-CAM-02 | Screen 3 | URD-II.6.10, URD-III-UC-CAM-02, URD-IV-Screen3 (Kênh & Lịch gửi STT 8) | | v4.1 — sửa lại từ v4.0: bản đầu đặt sai vị trí (System Settings) và sai ý nghĩa (retry kỹ thuật) so với yêu cầu thực tế của BA |
| REQ-CVM-066 | Kiểm tra ký tự SMS có dấu/không dấu khi soạn nội dung — 70 ký tự/segment (có dấu) / 160 ký tự/segment (không dấu), tự phát hiện realtime | UC-CAM-02 | Screen 3 | URD-III-UC-CAM-02, URD-IV-Screen3 (Section 4 STT 9) | | Mới thêm v4 — comment nghiệm thu "check ký tự sms có dấu/không dấu" |
| REQ-CVM-067 | Blacklist toàn hệ thống — danh sách độc lập với Blacklist theo campaign, áp dụng mọi campaign/mọi kênh, chỉ Admin thao tác; hỗ trợ cả Thêm thủ công (UC-BL-04) và Upload CSV (UC-BL-05) | UC-BL-04, UC-BL-05 | Screen 6A | URD-III-UC-BL-04, URD-III-UC-BL-05, URD-IV-Screen6A (STT 13) | | Mới thêm v4 — comment nghiệm thu "BL toàn hệ thống"; kiểm tra trước Blacklist theo campaign-kênh trong pipeline (xem II.6.4); UC-BL-05 bổ sung sau khi đóng OQ-12 |
| REQ-CVM-068 | Trigger cho mẫu tin nhắn — **bắt buộc chọn đúng 1 trigger** (single-select, v4.4; ban đầu v4.0 là multi-select không bắt buộc dùng để nhóm hiển thị) tại Template Editor, dùng để lấy đúng tham số động của trigger đó khi soạn nội dung; hiển thị dạng 1 chip Trigger tại Template List (v4.3 đổi từ nhóm collapsible thành cột đơn giản, v4.4 bỏ luôn "+N" vì chỉ còn 1 trigger); cho phép **Xóa cứng template ở mọi mức độ sử dụng** kể cả đang được campaign tham chiếu (v4.4 — ban đầu v4.0 chỉ xóa được khi Dùng = 0), có dialog cảnh báo khi Dùng > 0, không ảnh hưởng nội dung đã copy vào campaign | UC-TPL-00, UC-TPL-01 | Screen 4A, Screen 4B | URD-III-UC-TPL-00, URD-III-UC-TPL-01, URD-IV-Screen4A, URD-IV-Screen4B | | Mới thêm v4 — comment nghiệm thu "nhóm lại theo trigger, cho phép xóa mẫu tin nhắn"; v4.3 sửa cách hiển thị; v4.4 sửa bản chất nghiệp vụ (single-select bắt buộc + xóa không giới hạn theo Dùng) |

---

## Reverse Index

### Use Cases

| UC ID | Phụ thuộc REQ |
|---|---|
| UC-CAM-01 | REQ-CVM-002, REQ-CVM-020 |
| UC-CAM-02 | REQ-CVM-001, REQ-CVM-003, REQ-CVM-004, REQ-CVM-005, REQ-CVM-006, REQ-CVM-007, REQ-CVM-008, REQ-CVM-009, REQ-CVM-010, REQ-CVM-011, REQ-CVM-019, REQ-CVM-020, REQ-CVM-065, REQ-CVM-066 |
| UC-CAM-03 | REQ-CVM-012 |
| UC-CAM-04 | REQ-CVM-013 |
| UC-CAM-05 | REQ-CVM-001, REQ-CVM-014, REQ-CVM-015 |
| UC-CAM-06 | REQ-CVM-001, REQ-CVM-016, REQ-CVM-058 |
| UC-CAM-07 | REQ-CVM-001, REQ-CVM-017, REQ-CVM-032, REQ-CVM-033 |
| UC-CAM-08 | REQ-CVM-001, REQ-CVM-011, REQ-CVM-018, REQ-CVM-019, REQ-CVM-032, REQ-CVM-033 |
| UC-TPL-00 | REQ-CVM-021, REQ-CVM-068 |
| UC-TPL-01 | REQ-CVM-022, REQ-CVM-068 |
| UC-TPL-02 | REQ-CVM-023 |
| UC-TPL-03 | REQ-CVM-024 |
| UC-TPL-04 | REQ-CVM-025 |
| UC-TRG-00 | REQ-CVM-026 |
| UC-TRG-02 | REQ-CVM-027 |
| UC-TRG-03 | REQ-CVM-028 |
| UC-TRG-04 | REQ-CVM-029, REQ-CVM-032 |
| UC-TRG-05 | REQ-CVM-030, REQ-CVM-031, REQ-CVM-033 |
| UC-BL-00 | REQ-CVM-034 |
| UC-BL-01 | REQ-CVM-035 |
| UC-BL-02 | REQ-CVM-036 |
| UC-BL-03 | REQ-CVM-037 |
| UC-BL-04 | REQ-CVM-067 |
| UC-BL-05 | REQ-CVM-067 |
| UC-KH-00 | REQ-CVM-038, REQ-CVM-063, REQ-CVM-064 |
| UC-KH-01 | REQ-CVM-039, REQ-CVM-059, REQ-CVM-060, REQ-CVM-063, REQ-CVM-064 |
| UC-RPT-01 | REQ-CVM-040, REQ-CVM-041, REQ-CVM-042, REQ-CVM-043, REQ-CVM-044, REQ-CVM-045, REQ-CVM-046, REQ-CVM-047, REQ-CVM-048 |
| UC-DSH-01 | REQ-CVM-049, REQ-CVM-050, REQ-CVM-051, REQ-CVM-052, REQ-CVM-053 |
| UC-PRIORITY-01 | REQ-CVM-020, REQ-CVM-054, REQ-CVM-061 |

### Wireframes / Screens

| Screen | Phụ thuộc REQ |
|---|---|
| Screen 1 (Dashboard) | REQ-CVM-049, REQ-CVM-050, REQ-CVM-051, REQ-CVM-052, REQ-CVM-053 |
| Screen 2 (Campaign List) | REQ-CVM-001, REQ-CVM-002, REQ-CVM-016, REQ-CVM-017, REQ-CVM-020, REQ-CVM-032, REQ-CVM-033 |
| Screen 2B (Campaign Detail View) | REQ-CVM-007, REQ-CVM-013, REQ-CVM-016, REQ-CVM-017, REQ-CVM-032, REQ-CVM-033 |
| Screen 3 (Campaign Builder) | REQ-CVM-001, REQ-CVM-003, REQ-CVM-004, REQ-CVM-005, REQ-CVM-006, REQ-CVM-007, REQ-CVM-008, REQ-CVM-009, REQ-CVM-010, REQ-CVM-011, REQ-CVM-012, REQ-CVM-018, REQ-CVM-019, REQ-CVM-020, REQ-CVM-031, REQ-CVM-032, REQ-CVM-033, REQ-CVM-065, REQ-CVM-066 |
| Screen 4A (Template List) | REQ-CVM-021, REQ-CVM-025, REQ-CVM-068 |
| Screen 4B (Template Editor) | REQ-CVM-022, REQ-CVM-024, REQ-CVM-068 |
| Screen 4C (Template Detail View) | REQ-CVM-023 |
| Screen 5A (Trigger Management — QTV list) | REQ-CVM-026 |
| Screen 5B (Modal chi tiết Trigger — QTV) | REQ-CVM-027 |
| Screen 6A (Blacklist — Danh sách) | REQ-CVM-034, REQ-CVM-035, REQ-CVM-036, REQ-CVM-037, REQ-CVM-067 |
| Screen 6B (Modal Thêm thủ công) | REQ-CVM-035 |
| Screen 6C (Modal Upload danh sách) | REQ-CVM-036 |
| Screen 7 (Customer List) | REQ-CVM-038, REQ-CVM-063, REQ-CVM-064 |
| Screen 8 (Customer 360) | REQ-CVM-039, REQ-CVM-059, REQ-CVM-060, REQ-CVM-063, REQ-CVM-064 |
| Screen 9 (Report / Analytics) | REQ-CVM-040, REQ-CVM-041, REQ-CVM-042, REQ-CVM-043, REQ-CVM-044, REQ-CVM-045, REQ-CVM-046, REQ-CVM-047, REQ-CVM-048 |
| Screen Admin (Duyệt Campaign) | REQ-CVM-014 |
| Screen Admin (Tab Trigger — danh sách) | REQ-CVM-026 |
| Screen Admin (T-DETAIL) | REQ-CVM-027, REQ-CVM-029, REQ-CVM-030, REQ-CVM-031, REQ-CVM-032, REQ-CVM-033 |
| Screen Admin (T-NEW) | REQ-CVM-028, REQ-CVM-031 |
| Screen Settings (Tab 1 — Frequency Cap) | REQ-CVM-060 |
| Screen Settings (Tab 3 — Priority Matrix) | REQ-CVM-020, REQ-CVM-054, REQ-CVM-061 |

### URD Sections

| URD Section | Phụ thuộc REQ |
|---|---|
| URD-II.1 (Workflow Diagram) | REQ-CVM-001 |
| URD-II.6.1 (Tổng quan pipeline) | REQ-CVM-055 |
| URD-II.6.2 (Fallback kênh) | REQ-CVM-056 |
| URD-II.6.3 (Retry Gateway) | REQ-CVM-057 |
| URD-II.6.4 (Điều kiện dừng pipeline) | REQ-CVM-015, REQ-CVM-058 |
| URD-II.6.5 (Kênh & Lịch gửi override) | REQ-CVM-008 |
| URD-II.6.6 (Sync-back Customer 360) | REQ-CVM-059 |
| URD-II.6.7 (Throttling & Frequency Cap) | REQ-CVM-060 |
| URD-II.6.8 (Cross-campaign Priority) | REQ-CVM-020, REQ-CVM-061 |
| URD-II.6.9 (Deduplication Event) | REQ-CVM-062 |
| URD-II.6.10 (Nhắc lại — Re-engagement) | REQ-CVM-065 |
| URD-II.7 (Ánh xạ trạng thái SIM) | REQ-CVM-063 |
| URD-II.7.4 (Nguồn dữ liệu Đã cài app) | REQ-CVM-064 |
| URD-Quy-tắc-Khối-3 (PARAM_INVALID / FILTER_INVALID) | REQ-CVM-011, REQ-CVM-032, REQ-CVM-033 |
| URD-III-Bảng-toán-tử-hợp-lệ | REQ-CVM-031 |
| URD-III-UC-CAM-01 | REQ-CVM-002, REQ-CVM-020 |
| URD-III-UC-CAM-02 | REQ-CVM-001, REQ-CVM-003 → REQ-CVM-011, REQ-CVM-019, REQ-CVM-020, REQ-CVM-065, REQ-CVM-066 |
| URD-III-UC-CAM-03 | REQ-CVM-012 |
| URD-III-UC-CAM-04 | REQ-CVM-013 |
| URD-III-UC-CAM-05 | REQ-CVM-001, REQ-CVM-014, REQ-CVM-015 |
| URD-III-UC-CAM-06 | REQ-CVM-001, REQ-CVM-016, REQ-CVM-058 |
| URD-III-UC-CAM-07 | REQ-CVM-001, REQ-CVM-017, REQ-CVM-032, REQ-CVM-033 |
| URD-III-UC-CAM-08 | REQ-CVM-001, REQ-CVM-011, REQ-CVM-018, REQ-CVM-019, REQ-CVM-032, REQ-CVM-033 |
| URD-III-UC-TPL-00 | REQ-CVM-021, REQ-CVM-068 |
| URD-III-UC-TPL-01 | REQ-CVM-022, REQ-CVM-068 |
| URD-III-UC-TPL-02 | REQ-CVM-023 |
| URD-III-UC-TPL-03 | REQ-CVM-024 |
| URD-III-UC-TPL-04 | REQ-CVM-025 |
| URD-III-UC-TRG-00 | REQ-CVM-026 |
| URD-III-UC-TRG-02 | REQ-CVM-027 |
| URD-III-UC-TRG-03 | REQ-CVM-028 |
| URD-III-UC-TRG-04 | REQ-CVM-029, REQ-CVM-032 |
| URD-III-UC-TRG-05 | REQ-CVM-030, REQ-CVM-031, REQ-CVM-033 |
| URD-III-UC-BL-00 | REQ-CVM-034 |
| URD-III-UC-BL-01 | REQ-CVM-035 |
| URD-III-UC-BL-02 | REQ-CVM-036 |
| URD-III-UC-BL-03 | REQ-CVM-037 |
| URD-III-UC-BL-04 | REQ-CVM-067 |
| URD-III-UC-BL-05 | REQ-CVM-067 |
| URD-III-UC-KH-00 | REQ-CVM-038, REQ-CVM-063, REQ-CVM-064 |
| URD-III-UC-KH-01 | REQ-CVM-039, REQ-CVM-059, REQ-CVM-060, REQ-CVM-063, REQ-CVM-064 |
| URD-III-UC-RPT-01 | REQ-CVM-040 → REQ-CVM-048 |
| URD-III-UC-DSH-01 | REQ-CVM-049 → REQ-CVM-053 |
| URD-III-UC-PRIORITY-01 | REQ-CVM-020, REQ-CVM-054, REQ-CVM-061 |
| URD-IV-Screen1 | REQ-CVM-049 → REQ-CVM-053 |
| URD-IV-Screen2 | REQ-CVM-002, REQ-CVM-016, REQ-CVM-017, REQ-CVM-020, REQ-CVM-032, REQ-CVM-033 |
| URD-IV-Screen2B | REQ-CVM-013, REQ-CVM-016, REQ-CVM-017, REQ-CVM-032, REQ-CVM-033 |
| URD-IV-Screen3 | REQ-CVM-003 → REQ-CVM-012, REQ-CVM-018 → REQ-CVM-020, REQ-CVM-031 → REQ-CVM-033, REQ-CVM-065, REQ-CVM-066 |
| URD-IV-Screen4A | REQ-CVM-021, REQ-CVM-025, REQ-CVM-068 |
| URD-IV-Screen4B | REQ-CVM-022, REQ-CVM-024, REQ-CVM-068 |
| URD-IV-Screen4C | REQ-CVM-023 |
| URD-IV-Screen5A | REQ-CVM-026 |
| URD-IV-Screen5B | REQ-CVM-027 |
| URD-IV-Screen6A | REQ-CVM-034, REQ-CVM-035, REQ-CVM-036, REQ-CVM-037, REQ-CVM-067 |
| URD-IV-Screen6B | REQ-CVM-035 |
| URD-IV-Screen6C | REQ-CVM-036 |
| URD-IV-Screen7 | REQ-CVM-038, REQ-CVM-063, REQ-CVM-064 |
| URD-IV-Screen8 | REQ-CVM-039, REQ-CVM-059, REQ-CVM-060, REQ-CVM-063, REQ-CVM-064 |
| URD-IV-Screen9 | REQ-CVM-040 → REQ-CVM-048 |
| URD-IV-Screen-Admin | REQ-CVM-014, REQ-CVM-026 → REQ-CVM-033 |
| URD-IV-Screen-Settings | REQ-CVM-020, REQ-CVM-054, REQ-CVM-060, REQ-CVM-061 |

---

## Version History

| Ngày | Action | REQ ID | Ghi chú |
|---|---|---|---|
| 2026-08-21 | INIT | ALL | Tạo map từ urd-srs-v3.md (baseline trước khi xử lý comment nghiệm thu) |
| 2026-08-21 | MODIFY | REQ-CVM-003, REQ-CVM-007, REQ-CVM-009, REQ-CVM-017, REQ-CVM-018, REQ-CVM-019, REQ-CVM-020, REQ-CVM-021, REQ-CVM-022, REQ-CVM-031, REQ-CVM-034, REQ-CVM-035, REQ-CVM-036, REQ-CVM-037, REQ-CVM-038, REQ-CVM-039, REQ-CVM-060 | SYNC theo Change Set 2026-08-21 (comment nghiệm thu CVM) — patch urd-srs-v3.md → v4.md, xem chi tiết mục CÁC THAY ĐỔI trong urd-srs-v4.md |
| 2026-08-21 | ADD | REQ-CVM-065, REQ-CVM-066, REQ-CVM-067, REQ-CVM-068 | 4 requirement mới từ Change Set 2026-08-21: Gửi lại (Retry), kiểm tra ký tự SMS có dấu/không dấu, Blacklist toàn hệ thống (UC-BL-04 mới), nhóm Template theo Trigger + Xóa |
| 2026-08-21 | MODIFY | REQ-CVM-068 | v4.3: BA yêu cầu bỏ nhóm collapsible theo Trigger tại Template List, thay bằng cột Trigger dạng chip (giống Campaign List) — mô tả REQ và Screen 4A STT 2–6, 10 cập nhật lại |
| 2026-08-21 | MODIFY | REQ-CVM-068 | v4.4: 2 thay đổi bản chất nghiệp vụ theo yêu cầu BA — (1) Trigger đổi từ multi-select không bắt buộc sang single-select bắt buộc, mục đích thực sự là lấy đúng tham số của 1 trigger để soạn nhanh (không phải để nhóm hiển thị); (2) Xóa template cho phép cả khi đang được dùng (Dùng > 0), có cảnh báo campaign ảnh hưởng, không đổi nội dung đã copy vào campaign. Cập nhật UC-TPL-00, UC-TPL-01, Screen 4A STT 6, Screen 4B STT 1b/4/6 |
| 2026-08-21 | MODIFY | REQ-CVM-067 | Sau QA/postcheck: đóng OQ-12 (Có — bổ sung Upload CSV cho Blacklist toàn hệ thống), thêm UC-BL-05; đồng thời fix PC-01 (postcheck) — bổ sung UC-BL-04 vào Function Tree/Permission Matrix/RBAC Matrix (II.2/II.3/II.4), trước đó chỉ có ở Section III/IV |
| 2026-08-21 | MODIFY | REQ-CVM-065 | Sửa lại (v4.1): bản v4.0 đặt sai vị trí (System Settings) và sai ý nghĩa (retry kỹ thuật khi Failed) — đổi thành "Nhắc lại" (Re-engagement), cấu hình riêng theo từng campaign tại Campaign Builder, chỉ nhắc khi KH vẫn thoả điều kiện trigger gốc. UC/WF/URD Section đổi từ UC-KH-01/Screen 8/II.6.7 sang UC-CAM-02/Screen 3/II.6.10 (mới) |
| 2026-08-22 | MODIFY | REQ-CVM-034, REQ-CVM-037 | v4.9: Blacklist Management gộp bảng theo Số điện thoại (1 dòng/số/phạm vi, chip Campaign/Kênh tối đa 2+"+N") theo yêu cầu BA — trước dàn trải mỗi tổ hợp campaign-kênh 1 dòng. Bỏ cột Nguồn. Xóa mở rộng 2 lớp: [×] per-chip (gỡ 1 tổ hợp) và [Xóa] toàn dòng (gỡ mọi tổ hợp trong phạm vi đang xem). Blacklist toàn hệ thống vẫn tách dòng riêng, không gộp với dòng Theo Campaign. Dữ liệu lưu trữ nội bộ không đổi — chỉ đổi cách UI gộp hiển thị. Cập nhật UC-BL-00, UC-BL-03, Screen 6A |
| 2026-08-22 | MODIFY | REQ-CVM-009, REQ-CVM-006 | v4.8: Preview Section 4 (Campaign Builder) đổi sang hiển thị đồng thời toàn bộ Audience Variant dạng N mockup card xếp dọc — trước đây preview theo tab biến thể đang active, QTV phải click từng tab mới xem hết; nay hiển thị cố định cùng lúc, áp dụng riêng theo từng tab kênh. Cột trái (soạn nội dung) vẫn giữ theo tab biến thể như cũ; SAMPLE PARAMS dùng chung 1 bộ cho mọi card. Cập nhật UC-CAM-02 Quy tắc nghiệp vụ, Screen 3 Section 4 STT 4a/11/12 |
| 2026-08-22 | MODIFY | REQ-CVM-065 | v4.7: Nhắc lại (Re-engagement) đổi đơn vị "Khoảng cách tối thiểu giữa các lần nhắc" từ giờ sang ngày theo yêu cầu BA — giới hạn nhập 1–365 ngày (riêng cho ô này, tách khỏi ngưỡng 1–9999 dùng chung với "Số lần nhắc lại tối đa"). Cập nhật II.6.10, UC-CAM-02 (bước 4a + Quy tắc nghiệp vụ), Screen 3 STT 8 |
| 2026-08-22 | MODIFY | — (NFR, không có REQ ID) | v4.6: fix gap sót từ đợt bỏ Reach ước tính (v4.5) — xóa dòng NFR "Tính toán reach" tại C.5 Yêu cầu về hiệu năng. Phát hiện qua rà soát đối chiếu toàn diện 12 nghiệp vụ đã patch V4.0-V4.5 với Function Tree/Permission/RBAC/Sequence/Workflow/Glossary/Report. 3 gap còn lại thuộc phạm vi diagram (Workflow Diagram II.1, Sequence Diagram II.5.2 — thiếu bước Blacklist toàn hệ thống) — BA quyết định không patch đợt này |
| 2026-08-21 | MODIFY | REQ-CVM-007 | v4.5: rà soát toàn diện theo yêu cầu BA — đóng nốt 4 vị trí URD còn mô tả điều kiện lọc theo mô hình cũ (Section 3, dùng chung mọi kênh) sau khi v4.0 đã dời sang Section 4: Screen 3 STT 3, UC-CAM-02 Hoạt động (bước 3 + 5b mới), Screen 2B STT 6/7, UC-TRG-05 (3 vị trí tham chiếu). Xác nhận logic AND/OR không đổi — chỉ đổi phạm vi áp dụng (dùng chung → riêng theo kênh). Hệ quả: bỏ toàn bộ Reach ước tính khỏi Campaign Builder (Estimate box, Section 3 STT 1/6, Section 6 STT 5, Sequence Diagram II.5.1, Glossary I.3.1 mục 6) — không còn 1 con số ước tính chung đủ chính xác khi điều kiện lọc đã tách theo từng kênh; giữ nguyên Reach trong Report/Conversion Rate (REQ-CVM-045, khác bản chất — snapshot sau khi chạy) |
