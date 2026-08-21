# QA Report — urd-srs-v4.md (SYNC mode, Change Set 2026-08-21)

**Phạm vi review:** Tập trung vào 14 thay đổi patch từ `change-set-2026-08-21.md` và vùng nội dung liên quan trực tiếp (workflow diagram, UC-CAM-01/02/03/05/07/08, UC-PRIORITY-01, UC-TPL-00, UC-BL-00~04, UC-KH-01, II.6.4/II.6.7, Quy tắc nghiệp vụ chung Khối 3, Screen 2/2B/3/4A/6/7/8, Settings Tab 1). Không review lại các phần không bị patch (Dashboard, Report Tab 2-6, kiến trúc, NFR...).

---

## Review Report

### CRITICAL — Phải sửa trước khi tiếp tục

**[CR-01]** Workflow Diagram (Section II.1) vẫn mô tả "Bật lại" là hành động luôn tự động, không đề cập nhánh mới "Bật lại → Chờ duyệt khi param/điều kiện lọc trigger bị Sửa trong lúc Paused"
- Vị trí: Section II.1, Mermaid diagram node `S151` (dòng 379) và bảng diễn giải bước 15.1 (dòng 443)
- Vấn đề: Thay đổi #3 trong Change Set ("Bật lại từ Paused → Chờ duyệt khi param/điều kiện lọc trigger bị Sửa") đã được patch đầy đủ vào UC-CAM-07, Screen 2 STT 9, Screen 2B STT 3 — nhưng Workflow Diagram tổng thể (nơi Dev/Tester đọc đầu tiên để hiểu bức tranh toàn cảnh) vẫn giữ nguyên câu "Kích hoạt lại campaign ngay lập tức, không cần phê duyệt lại; chuyển về trạng thái Đang chạy → quay lại bước 11" — không có nhánh rẽ nào giữa "không sửa param" và "có sửa param". Người đọc chỉ theo Workflow Diagram (không đọc kỹ UC-CAM-07) sẽ hiểu sai: Bật lại luôn luôn tự động, không bao giờ về Chờ duyệt. Đây là mâu thuẫn logic trực tiếp giữa Section II (tổng thể) và Section III (chi tiết UC) — đúng loại lỗi mà checklist review liệt kê là "yêu cầu mâu thuẫn giữa các section".
- Khuyến nghị: Thêm nhánh quyết định vào node B15/S151 trong Mermaid: `B15["15. Kích hoạt lại — campaign"] --> S151b{"15.1 Param/điều kiện lọc trigger đã bị Sửa trong lúc Paused?"}`, rẽ 2 nhánh: "Không" → chuyển Active ngay (như cũ); "Có" → chuyển Pending, quay lại bước 9 (Chờ phê duyệt). Cập nhật bảng diễn giải bước 15.1 tương ứng, có thể thêm bước 15.2 mô tả nhánh Pending.

**[CR-02]** Tham chiếu "Section 3" bị bỏ sót ở 5 vị trí sau khi điều kiện lọc con đã dời sang Section 4 — tham chiếu chết, chỉ sai người dùng đến đúng chỗ cần sửa
- Vị trí:
  1. UC-CAM-03, Exception "campaign còn cờ FILTER_INVALID" (dòng 1102): "...vui lòng cập nhật điều kiện lọc **ở Section 3** trước khi gửi duyệt lại"
  2. Quy tắc nghiệp vụ chung — Khối 3, policy FILTER_INVALID (dòng 1395): "...cập nhật hoặc gỡ điều kiện lọc lỗi **ở Section 3** → [Lưu nháp]..."
  3. Screen 2 STT 9 — Cột HÀNH ĐỘNG (dòng 1910): tooltip "...vui lòng sửa điều kiện lọc **ở Section 3** trước"
  4. Screen 2B STT 3 — Nút hành động (dòng 1925): tooltip giống hệt dòng 1910
  5. Screen 3 STT 7 — Banner cảnh báo FILTER_INVALID (dòng 1950): "...vui lòng cập nhật điều kiện lọc **ở Section 3** trước khi gửi duyệt lại"
- Vấn đề: Thay đổi #5 trong Change Set đã di chuyển toàn bộ "điều kiện lọc con của phân khúc" từ Section 3 (Audience) sang Section 4 (Message Matrix) — điều này đã được patch đúng ở Section 3 STT 5 (dòng 1984, ghi rõ "đã di chuyển... xem Section 4") và Section 4 STT 4b (dòng 1996, đặc tả đầy đủ). Nhưng cụm từ hướng dẫn user "sửa ở Section 3" tại 5 vị trí trên (banner cảnh báo, tooltip nút Bật, Exception UC-CAM-03) chưa được cập nhật theo — dẫn QTV đến sai vị trí màn hình khi cố xử lý lỗi FILTER_INVALID. Sau patch, Section 3 chỉ còn chọn phân khúc + Logic phân khúc, không còn accordion điều kiện lọc nào để sửa.
- Khuyến nghị: Đổi cả 5 vị trí từ "Section 3" thành "Section 4 (Message Matrix)", đồng thời làm rõ QTV cần mở đúng tab kênh + card trigger + phân khúc tương ứng để tìm accordion điều kiện lọc bị lỗi (vì giờ điều kiện lọc phân tán theo từng cặp Trigger × Phân khúc × Kênh, không còn 1 chỗ duy nhất như Section 3 cũ). Cân nhắc bổ sung: nếu campaign có nhiều kênh/phân khúc, QTV cần kiểm tra từng tổ hợp — nên nói rõ trong thông báo lỗi thay vì chỉ trỏ chung "Section 4".

---

### MAJOR — Ảnh hưởng đáng kể, nên sửa

**[MA-01]** Edge case chưa đặc tả: sửa priority tại Campaign List → chuyển Pending → Admin **từ chối** thì campaign về đâu, có mất trạng thái Active không?
- Vị trí: UC-CAM-01 (Hoạt động bước 5, Quy tắc nghiệp vụ), Screen 2 STT 7/9
- Vấn đề: Theo thay đổi #4, sửa priority trên Campaign List bắt buộc chuyển campaign Active → Pending để Admin duyệt lại. Nhưng UC-CAM-05 (Duyệt/Từ chối) quy định rõ: "Từ chối: Campaign → Draft". Áp dụng nguyên xi, nếu Admin từ chối lần duyệt-lại-vì-đổi-priority này, campaign đang chạy (Active) sẽ rơi thẳng về **Draft** — nghĩa là dừng gửi tin hoàn toàn, mất trạng thái Active chỉ vì QTV/Admin đổi 1 con số ưu tiên. Đây là hệ quả nặng hơn nhiều so với mục đích ban đầu (chỉ đổi thứ tự ưu tiên), và tài liệu không xác nhận đây có phải hành vi mong muốn hay không — khác biệt rõ với luồng "sửa nội dung Paused" (nơi Từ chối cũng về Draft nhưng ít nhất QTV chủ động biết mình đang sửa nội dung). Trường hợp đổi priority, QTV có thể không lường trước rủi ro "mất Active" chỉ vì 1 thao tác nhỏ trên danh sách.
- Khuyến nghị: Xác nhận với PO: (a) Admin từ chối campaign đang ở Pending-do-đổi-priority thì có về Draft như luồng thông thường không, hay cần cơ chế riêng (ví dụ giữ nguyên Active với priority cũ nếu Admin từ chối)? (b) Nếu về Draft là đúng ý đồ, cần bổ sung rõ trong UC-CAM-01 Quy tắc nghiệp vụ + thông báo cảnh báo cho QTV/Admin ngay tại bước 5b (confirm dialog đổi priority) rằng "nếu Admin từ chối, campaign sẽ dừng hoàn toàn (chuyển Draft), không chỉ đơn thuần giữ nguyên Active với priority cũ".

**[MA-02]** Cụm "role có quyền xem đầy đủ" (Customer 360/Customer List che số điện thoại) không được định danh trong Permission Matrix hoặc RBAC Matrix
- Vị trí: UC-KH-01 Quy tắc nghiệp vụ (dòng 1509), Screen 7 STT 4 (dòng 2221), Screen 8 STT 2 (dòng 2239)
- Vấn đề: Cả 3 vị trí đều viết "hiển thị đầy đủ chỉ với role có quyền xem đầy đủ" nhưng hệ thống chỉ có 2 role (Admin HT, QTV Marketing), và cả UC-KH-00 lẫn UC-KH-01 đều liệt kê Tác nhân là "QTV Marketing, Admin Hệ thống" ngang hàng, không phân biệt quyền xem theo role ở Permission Matrix (II.3) hay RBAC (II.4). Nội dung ngầm định có sự phân biệt (Admin xem đầy đủ, QTV bị che — hoặc ngược lại) nhưng không nơi nào trong tài liệu nói rõ đó là role nào. OQ-11 (mục cần xác nhận #11) chỉ hỏi "danh sách field cần che" và "che theo role nào" gộp chung — nhưng phần đặc tả Screen 7/8 đã viết như thể đã có quyết định ("chỉ với role có quyền xem đầy đủ") trong khi thực chất chưa xác định role đó là ai. Rủi ro: Dev đọc Screen 7/8 có thể hiểu nhầm là đã chốt xong, chỉ còn thiếu danh sách field, bỏ sót việc role phân quyền cũng còn treo.
- Khuyến nghị: Sửa câu văn tại 3 vị trí để nhất quán với OQ-11 — ví dụ đổi thành "hiển thị đầy đủ chỉ với role được xác định sau khi PO/đội bảo mật chốt (xem OQ-11)" thay vì ngầm định đã có role cụ thể. Đảm bảo OQ-11 hỏi rõ 2 câu tách bạch: (1) field nào cần che, (2) role nào (trong 2 role Admin HT / QTV Marketing) được xem đầy đủ — hiện OQ-11 đã hỏi câu 2 nhưng cách viết trong bảng component lại tạo cảm giác đã được trả lời.

**[MA-03]** SMS 71 ký tự có dấu = 2 segment: nhất quán về công thức nhưng thiếu ngưỡng biên dưới 70 ký tự chính xác — cần xác nhận có tính theo bội số cố định hay không
- Vị trí: UC-CAM-02 Quy tắc nghiệp vụ (dòng 1088), Screen 3 STT 9 (dòng 2001)
- Vấn đề: Công thức `ceil(số ký tự / ngưỡng tương ứng)` được nêu nhất quán ở cả 2 nơi, ví dụ đúng (71 ký tự có dấu → ceil(71/70) = 2). Tuy nhiên đây là mô hình SMS segment đơn giản hóa — thực tế gateway SMS thường dùng ngưỡng segment tiếp theo nhỏ hơn ngưỡng đơn (ví dụ chuẩn GSM: segment đầu 160 ký tự nhưng segment 2+ chỉ 153 ký tự do header UDH). Tài liệu không nói rõ đây là quyết định nghiệp vụ đơn giản hóa (chấp nhận sai lệch nhỏ so với chuẩn kỹ thuật SMS thật) hay là gap chưa nhận ra.
- Khuyến nghị: Không phải lỗi phải sửa ngay (đây có thể là quyết định nghiệp vụ hợp lý để đơn giản hóa UI), nhưng nên bổ sung 1 dòng ghi chú xác nhận: "Công thức segment dùng ngưỡng cố định 70/160 cho mọi segment (không áp dụng logic segment 2+ ngắn hơn theo chuẩn GSM 7-bit/UCS-2) — quyết định đơn giản hóa, ước tính có thể lệch nhẹ so với gateway thực tế, chấp nhận được vì chỉ dùng để cảnh báo UI". Nếu đây không phải chủ đích ban đầu, cần confirm lại với PO/gateway provider.

---

### MINOR — Cải thiện chất lượng

**[MI-01]** UC-BL-04 (Blacklist toàn hệ thống) không có Exception ứng với "tổ hợp trùng nhưng có 1 phần hợp lệ mới" — chỉ có 2 case toàn-trùng và toàn-hợp-lệ
- Vị trí: UC-BL-04 Hoạt động, dòng 1476
- Vấn đề: Đặc tả có Exception "0 số được lưu mới" nhưng không nói rõ case hỗn hợp (một số trùng, một số mới) có toast hiển thị đúng "Đã thêm X số" (X = số mới thực sự) hay không — dù suy luận từ câu "lọc bỏ số sai định dạng và số đã có sẵn... lưu các số còn lại" là có, nhưng không viết tường minh như UC-BL-01 (đã có ví dụ N×M rõ ràng).
- Khuyến nghị: Bổ sung 1 câu ví dụ tương tự UC-BL-01 để nhất quán mức độ chi tiết, ví dụ: "Nhập 5 số, 2 số đã có sẵn trong Blacklist toàn hệ thống → chỉ 3 số được thêm mới, toast 'Đã thêm 3 số vào Blacklist toàn hệ thống ✓'".

**[MI-02]** Priority Matrix (UC-PRIORITY-01) và sửa priority tại Campaign List (UC-CAM-01) dùng 2 cơ chế xác nhận khác nhau (Admin tự sắp xếp không cần duyệt lại vs. bắt buộc về Pending) — đã giải thích lý do nhưng chưa nói rõ nếu **QTV** vào Priority Matrix
- Vị trí: UC-PRIORITY-01 Quy tắc nghiệp vụ (dòng 1816), Tác nhân (dòng 1811)
- Vấn đề: UC-PRIORITY-01 ghi Tác nhân chỉ có "Admin Hệ thống" — nhất quán với Permission Matrix (Cấu hình Priority Matrix không có trong bảng quyền của QTV). Không phải lỗi, nhưng nên xác nhận thêm: câu giải thích "Admin là người duyệt campaign nên tự sắp xếp lại thứ tự không cần tự duyệt lại chính mình" ngầm định lý do vì sao 2 cơ chế khác nhau — logic hợp lý, nhưng vì UC-CAM-01 cho phép **cả QTV lẫn Admin** sửa priority tại Campaign List, trong khi QTV không có quyền vào Priority Matrix, nên có sự bất đối xứng: Admin có 2 đường sửa priority (Matrix — không cần duyệt; Campaign List — chuyển Pending), QTV chỉ có 1 đường (Campaign List — luôn chuyển Pending). Đây là thiết kế nhất quán về mặt logic (đã note rõ), chỉ gợi ý nên thêm 1 câu tổng kết ngắn gọn trong UC-CAM-01 hoặc UC-PRIORITY-01 liệt kê rõ bảng "ai sửa priority ở đâu, kết quả gì" để Dev/Tester không cần tự suy luận từ 2 UC riêng lẻ.

**[MI-03]** Cụm "Section 3" xuất hiện tại dòng 1928 (Screen 2B STT 6) và dòng 1392/1984/1996 mang nghĩa đúng (mô tả nhất quán Campaign Detail hiển thị theo đúng Section 3 hiện tại — chỉ còn chọn phân khúc, không phải điều kiện lọc) — không phải lỗi, nhưng dễ gây nhầm lẫn khi đọc lướt do gần các đoạn CR-02 nói trên
- Vị trí: dòng 1928
- Khuyến nghị: Không cần sửa nội dung, nhưng khi Dev/BA fix CR-02 nên rà lại toàn bộ các cụm "Section 3" một lượt để chắc chắn không sửa nhầm các chỗ dùng đúng nghĩa (như dòng 1928 — nói về badge trigger nguồn, không liên quan điều kiện lọc).

---

### Tóm tắt

- CRITICAL: 2 issues
- MAJOR: 3 issues
- MINOR: 3 issues
- Đánh giá tổng thể: **Cần sửa trước khi tiếp tục** — 2 CRITICAL đều là tham chiếu/mâu thuẫn phát sinh trực tiếp từ patch SYNC vừa thực hiện (CR-01: Workflow Diagram chưa theo kịp UC-CAM-07 mới; CR-02: 5 vị trí trỏ sai "Section 3" thay vì "Section 4" sau khi di chuyển điều kiện lọc). Cả 2 đều sửa cục bộ, không cần viết lại nội dung lớn. Sau khi patch CR-01/CR-02, khuyến nghị xử lý MA-01 (cần hỏi PO) trước khi coi tài liệu là dev-ready cho luồng sửa priority.

**Bước tiếp theo đề xuất:** Quay lại `urd-srs-agent` để patch đúng 2 vị trí CRITICAL (Workflow Diagram bước 15 + 5 chỗ "Section 3"→"Section 4"), sau đó chạy lại `ba-qa-agent` xác nhận hết CRITICAL trước khi chuyển `ba-postcheck-agent`.
