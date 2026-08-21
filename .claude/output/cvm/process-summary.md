# Process Summary — CVM (Customer Value Management System)

**Ngày tổng kết:** 21/08/2026
**Loại đợt:** SYNC mode — xử lý comment nghiệm thu hệ thống (không phải GENERATE lần đầu; URD/SRS đã có từ v1 đến v3 trước đó)
**Version URD/SRS:** v4 (V4.0)
**Trạng thái session:** HOÀN THÀNH — patch xong, QA CRITICAL đã fix, postcheck PC-01 đã fix, OQ-12 và MA-01 đã đóng. Còn 1 open question (OQ-11) chưa chốt, không chặn cấu trúc tài liệu nhưng chặn 1 phần implement (xem mục 3).

> **Lưu ý:** Đây là tổng kết cho **đợt SYNC ngày 21/08/2026** — chỉ trích xuất những gì thay đổi trong đợt này (v3 → v4). Không diễn giải lại toàn bộ lịch sử v1–v3.

---

## 1. Decision Log

| # | Quyết định | Lý do | Người quyết định | Phase | Ảnh hưởng nếu thay đổi |
|---|---|---|---|---|---|
| D01 | Tạo Traceability Map baseline từ `urd-srs-v3.md` (64 REQ, REQ-CVM-001 đến 064) trước khi chạy SYNC, thay vì bỏ qua bước này | CVM chưa từng có Traceability Map dù đã qua 3 version URD/SRS — SYNC mode bắt buộc phải có map để tra impact | BA (Jun) | Pre-SYNC | Nếu không có map, không thể chạy impact-analysis chính xác cho các lần SYNC sau này |
| D02 | Bật lại campaign từ Paused → **Chờ duyệt** (không phải Active thẳng) khi param/điều kiện lọc trigger bị **Sửa** trong lúc Paused; giữ nguyên rule cũ (Active thẳng) nếu không có thay đổi gì | Đảm bảo Admin review lại nội dung đã sửa trước khi campaign chạy tiếp — tránh gửi tin sai theo cấu hình chưa được duyệt | BA, xác nhận qua AskUserQuestion tại [S1] | SYNC — S1/S3 | Nếu đổi lại thành luôn Active thẳng: mất kiểm soát chất lượng nội dung sau khi sửa trong lúc Paused |
| D03 | Sửa priority trực tiếp trên Campaign List (không chỉ qua Priority Matrix) → bắt buộc chuyển campaign về Chờ duyệt để Admin xác nhận lại | Đồng bộ cơ chế kiểm soát thay đổi — priority ảnh hưởng thứ tự gửi tin, cần review như các thay đổi khác | BA | SYNC — S1/S3 | Nếu bỏ yêu cầu duyệt lại: QTV có thể tự ý đổi thứ tự ưu tiên mà không ai kiểm soát |
| D04 | Điều kiện lọc con của phân khúc dời từ Section 3 (Audience) xuống Section 4 (Message Matrix), khai báo riêng theo từng cặp Trigger × Phân khúc × Kênh | Cho phép điều kiện lọc khác nhau theo từng kênh (ví dụ điều kiện SMS khác Zalo) — Section 3 cũ chỉ hỗ trợ 1 bộ điều kiện dùng chung | BA, xác nhận qua AskUserQuestion tại [S2] | SYNC — S1/S3 | Đây là thay đổi cấu trúc màn hình Campaign Builder lớn nhất đợt này — nếu đảo ngược, phải viết lại toàn bộ UC-CAM-02/03, Screen 3, Screen 4A cùng lúc |
| D05 | Khi thêm kênh mới vào campaign, điều kiện lọc mặc định **kế thừa** từ kênh đã có (không để trống) | Giảm thao tác lặp lại cho QTV khi campaign đã có điều kiện lọc ổn định ở kênh khác | BA, xác nhận qua AskUserQuestion tại [S2] checkpoint | SYNC — S2 | Nếu đổi sang "để trống": QTV phải nhập lại điều kiện lọc từ đầu cho mỗi kênh mới thêm |
| D06 | Bổ sung **Blacklist toàn hệ thống** là danh sách **độc lập** với Blacklist theo campaign — không phải chỉ là chọn "Tất cả" trong bộ lọc nhiều campaign/kênh | BA xác nhận rõ ràng qua AskUserQuestion tại [S1] — đây là NEW_CONFIRMED, không phải suy diễn từ comment gốc | BA | SYNC — S1 | Nếu hiểu nhầm là "chọn tất cả": sẽ không tách được pipeline kiểm tra riêng (DNC → BL toàn hệ thống → BL theo campaign-kênh) |
| D07 | Quyền tạo/xóa Blacklist toàn hệ thống chỉ dành cho **Admin** — QTV chỉ VIEW | BA xác nhận qua checkpoint tại [S2] (câu hỏi "quyền Blacklist toàn hệ thống — chỉ Admin hay cả QTV") | BA | SYNC — S2 | Nếu mở quyền cho QTV: cần viết lại Permission Matrix/RBAC Matrix Khối 4 và audit lại rủi ro lạm dụng chặn số trên toàn hệ thống |
| D08 | Bổ sung **UC-BL-05 (Upload CSV cho Blacklist toàn hệ thống)** — đóng OQ-12, dùng chung giới hạn 100.000 dòng với UC-BL-02, modal 2 tab (Thêm thủ công / Upload CSV) | Ban đầu Change Set chỉ có Thêm thủ công (UC-BL-04); BA quyết định bổ sung Upload CSV để nhất quán với Blacklist theo campaign (đã có cả 2 cách) | BA — quyết định muộn, sau khi QA/postcheck xong | SYNC — đóng OQ-12 sau [S4] | Nếu không bổ sung: Admin phải thêm số thủ công từng dòng cho Blacklist toàn hệ thống, không nhất quán với UX Blacklist theo campaign |
| D09 | Edge case "sửa priority → chuyển Pending → Admin từ chối → về Draft" được xác nhận là **hành vi đúng ý đồ**, không cần cơ chế riêng để giữ Active | QA nêu MA-01 (rủi ro: đổi 1 số ưu tiên có thể khiến campaign dừng hoàn toàn); BA xác nhận đây là chấp nhận được, áp dụng nguyên xi rule "Từ chối → Draft" đã có ở UC-CAM-05 | BA — quyết định sau khi đọc MA-01 | SYNC — đóng MA-01 sau [S4] | Nếu đổi ý sau này: cần thêm state riêng hoặc rule đặc biệt cho case Pending-do-đổi-priority, ảnh hưởng UC-CAM-01 và UC-CAM-05 |
| D10 | Công thức tính SMS segment dùng ngưỡng cố định 70 (có dấu) / 160 (không dấu) cho **mọi** segment, không áp dụng logic segment 2+ ngắn hơn theo chuẩn GSM 7-bit/UCS-2 | Đơn giản hóa để phục vụ mục đích cảnh báo UI cho QTV khi soạn nội dung — không cần độ chính xác kỹ thuật gateway thật | BA (ngầm định qua việc giữ nguyên công thức trong patch, QA ghi nhận là MA-03, không phải lỗi cần sửa) | SYNC — S3, note tại QA MA-03 | Nếu cần khớp chính xác với gateway SMS thật (ước tính cước phí, billing...): phải tính lại công thức theo chuẩn GSM, ảnh hưởng UC-CAM-02 và Screen 3 STT 9 |

**Ghi chú Decision Log:**
- **D04** (dời điều kiện lọc từ Section 3 sang Section 4) là quyết định có rủi ro cao nhất nếu thay đổi muộn — đã lan sang 2 UC, 2 màn hình, và gây ra 5 tham chiếu chết mà QA phải bắt lại (CR-02). Bất kỳ thay đổi tiếp theo về cấu trúc điều kiện lọc nên rà soát lại toàn bộ chuỗi tham chiếu tương tự.
- **D09** và **D10** là 2 quyết định "giữ nguyên, không sửa" — quan trọng để Dev/Tester không tự ý coi đây là bug và tự sửa khác đi.

---

## 2. Assumption Register

| # | Assumption | Loại | Nguồn | Rủi ro nếu sai | Cách validate |
|---|---|---|---|---|---|
| A01 | Đơn vị % và GB áp dụng cho kiểu dữ liệu Số (Number) hiện có trong Bảng toán tử hợp lệ — không phát sinh kiểu dữ liệu mới | Unverified | [ASS-01], Change Set 2026-08-21 | Nếu sai (cần kiểu dữ liệu riêng): phải mở rộng Bảng toán tử hợp lệ, ảnh hưởng UC-CAM-02, UC-CAM-03, validate logic điều kiện lọc | Dev/BA xác nhận lại với PO khi có ví dụ dữ liệu thực tế đầu tiên dùng đơn vị GB |
| A02 | Chỉ xóa được mẫu tin nhắn **chưa được campaign nào tham chiếu** — theo nguyên tắc cột "Dùng" hiện có tại UC-TPL-00 | Unverified | [ASS-02], Change Set 2026-08-21 | Nếu PO muốn cho xóa cả template đã dùng (soft-delete/archive): cần thiết kế lại UC-TPL-01 và ảnh hưởng dữ liệu lịch sử campaign đã Ended | BA xác nhận lại với PO trước khi Dev code chức năng Xóa template |
| A03 | Gửi lại (Retry) sau lần gửi đầu áp dụng ở **mức campaign** — không phải cấu hình chung toàn hệ thống | Unverified | [ASS-03], Change Set 2026-08-21 | Nếu PO muốn cấu hình Retry ở mức hệ thống (áp dụng mọi campaign): cần thêm màn hình cấu hình global, ảnh hưởng Screen Settings | BA xác nhận khi PO review Screen Settings Tab 1 (UC-KH-01, REQ-CVM-065) |
| A04 | Danh sách field cụ thể cần che tại Customer 360/Customer List **chưa chốt** — patch chỉ ghi nguyên tắc chung (che theo bảo mật) + để Open Question, không liệt kê cứng field | Risky | [ASS-04], Change Set 2026-08-21 | Cao — nếu PO/đội bảo mật yêu cầu che thêm field ngoài SĐT (CCCD, địa chỉ...) sau khi Dev đã code chỉ mask SĐT: phải sửa lại UC-KH-01, Screen 7, Screen 8 và có thể ảnh hưởng cả luồng đọc dữ liệu từ BSS/OCS | **Phải validate trước khi Dev bắt đầu code phần che thông tin** — xem OQ-11 tại mục 3 |
| A05 | Công thức SMS segment dùng ngưỡng cố định 70/160 cho mọi segment (không theo chuẩn GSM 7-bit/UCS-2 với segment 2+ ngắn hơn) — chấp nhận sai lệch nhẹ so với gateway thực tế, chỉ dùng để cảnh báo UI | Confirmed | QA report MA-03 — BA xác nhận đây là quyết định nghiệp vụ, không phải gap kỹ thuật bị bỏ sót | Thấp trong phạm vi UI cảnh báo; Trung bình nếu sau này dùng số liệu này để tính cước/billing thật | Không cần validate thêm trừ khi có yêu cầu tính cước SMS chính xác — khi đó cần đối chiếu với gateway provider |
| A06 | Đổi priority trên Campaign List → Admin từ chối → campaign về Draft (mất Active hoàn toàn) là hành vi **đúng ý đồ**, áp dụng nguyên xi rule "Từ chối → Draft" có sẵn | Confirmed | QA report MA-01 — BA xác nhận trực tiếp sau khi đọc cảnh báo rủi ro | Thấp — đã có xác nhận rõ ràng từ BA | Không cần validate thêm; khuyến nghị Dev thêm cảnh báo UI tại bước xác nhận đổi priority để QTV biết rủi ro này (theo khuyến nghị MA-01) |
| A07 | Che thông tin nhạy cảm tại Customer 360/Customer List chỉ mới xác định **Số điện thoại** (mask dạng `09xx xxx 678`) là field chắc chắn phải che; role được xem đầy đủ **chưa được định danh** trong Permission Matrix/RBAC Matrix (hệ thống hiện chỉ có 2 role: Admin Hệ thống, QTV Marketing, chưa rõ ai xem đầy đủ) | Risky | QA report MA-02, URD-SRS v4 Section III (UC-KH-01), Section IV (Screen 7 STT 4, Screen 8 STT 2) | Cao — Dev có thể ngộ nhận đã có quyết định phân quyền (vì câu văn viết "role có quyền xem đầy đủ" như thể đã tồn tại) và code sai role, hoặc bỏ sót logic phân quyền hoàn toàn | **Phải validate trước khi Dev bắt đầu code phần che thông tin** — xem OQ-11 tại mục 3 |

**Tóm tắt:**
- Confirmed: 2 (A05, A06) — đã có xác nhận trực tiếp từ BA sau khi QA nêu vấn đề
- Unverified: 3 (A01, A02, A03) — hợp lý, ghi rõ trong Change Set, nhưng chưa có xác nhận từ PO
- Risky: 2 (A04, A07) — cùng gốc vấn đề (che thông tin C360), impact cao nếu sai, đang gộp lại thành 1 open question duy nhất (OQ-11)

**Ưu tiên validate trước khi Dev bắt đầu:** A04, A07 — cả hai đều thuộc phạm vi "che thông tin nhạy cảm tại Customer 360/Customer List" và đều chặn trực tiếp việc code phần này. Xem chi tiết hành động tại mục 3.1/3.3.

---

## 3. Handoff Note

> Dành cho Dev Lead và Test Lead — đọc trước khi bắt đầu implement hoặc viết test plan cho phần thay đổi từ đợt SYNC 21/08/2026.

### 3.1 Điểm cần làm rõ với BA trước khi implement

| # | Điểm cần làm rõ | Lý do chưa rõ | Section trong URD/SRS | Mức độ khẩn |
|---|---|---|---|---|
| H01 | Danh sách đầy đủ field cần che tại Customer 360/Customer List (ngoài Số điện thoại) và role nào được xem đầy đủ | Change Set ghi rõ [ASS-04]: BA chưa xác định được, chỉ có ứng viên gợi ý (CCCD nếu có, địa chỉ) — chưa chốt | UC-KH-01 (dòng 1530), Screen 7 STT 4, Screen 8 STT 2 | **Cao** — chặn trực tiếp việc code phần che thông tin, không nên code chỉ dựa trên "ứng viên gợi ý" |
| H02 | Role nào (Admin Hệ thống hay QTV Marketing) được xem số điện thoại đầy đủ — hiện Permission Matrix và RBAC Matrix chưa có định nghĩa role này, dù Screen 7/8 đã viết như thể đã có quyết định | QA report MA-02 chỉ ra: câu văn "role có quyền xem đầy đủ" tạo cảm giác đã chốt nhưng thực chất chưa | Permission Matrix (II.3), RBAC Matrix (II.4), Screen 7 STT 4, Screen 8 STT 2 | **Cao** — cùng gốc với H01, gộp chung vào OQ-11 |

### 3.2 Open questions chưa được giải quyết

| # | Câu hỏi | Đặt ra tại | Người cần trả lời | Ảnh hưởng đến |
|---|---|---|---|---|
| Q01 (= OQ-11 trong URD/SRS) | Danh sách chính xác các trường thông tin cần che tại Customer 360/Customer List là gì? Che hoàn toàn hay mask một phần? Áp dụng theo role nào (Admin Hệ thống / QTV Marketing / cả hai đều bị che)? | Change Set 2026-08-21 ([ASS-04], OQ-CVM-NEW-01), khoanh vùng thêm bởi QA report MA-02 | PO / đội bảo mật | UC-KH-01, Screen 7 (Customer List), Screen 8 (Customer 360), Permission Matrix, RBAC Matrix |

**Đây là open question duy nhất còn treo phát sinh từ đợt SYNC này.** (OQ-12 — có cần Upload CSV cho Blacklist toàn hệ thống không — đã được BA đóng, xem D08. MA-01 — edge case đổi priority bị từ chối — đã được BA xác nhận, xem D09.)

### 3.3 Chỗ tài liệu dùng assumption thay vì fact đã xác nhận

| # | Section | Nội dung assumption | Assumption ID | Hành động khuyến nghị |
|---|---|---|---|---|
| X01 | UC-KH-01 Quy tắc nghiệp vụ (dòng 1530); Screen 7 STT 4; Screen 8 STT 2 | Che thông tin nhạy cảm — hiện chỉ chắc chắn về Số điện thoại (mask `09xx xxx 678`); các field khác và role xem đầy đủ chưa chốt | A04, A07 | **Không code phần che field ngoài Số điện thoại cho đến khi OQ-11 được PO/đội bảo mật trả lời.** Test Lead: viết test case cho Số điện thoại trước, để trống test case cho field khác đến khi có câu trả lời |
| X02 | UC-CAM-02 Quy tắc nghiệp vụ (dòng 1088); Screen 3 STT 9 | Công thức SMS segment dùng ngưỡng cố định 70/160, không theo chuẩn GSM 7-bit/UCS-2 thực tế (segment 2+ ngắn hơn segment đầu) | A05 | **Đây KHÔNG phải thiếu sót cần sửa** — là quyết định nghiệp vụ đã được BA xác nhận để đơn giản hóa UI cảnh báo. Dev code đúng theo công thức `ceil(số ký tự / ngưỡng)` ghi trong tài liệu, không tự ý áp chuẩn GSM. Nếu sau này cần tính cước SMS chính xác, đây là điểm cần quay lại đối chiếu với gateway provider |
| X03 | UC-CAM-01 Quy tắc nghiệp vụ; Screen 2 STT 7/9 | Đổi priority → Pending → Admin từ chối → về Draft (dừng gửi tin hoàn toàn) | A06 | **Đây KHÔNG phải bug** — hành vi đúng ý đồ, BA đã xác nhận. Khuyến nghị UI: thêm cảnh báo tại bước xác nhận đổi priority để QTV biết rủi ro trước khi thao tác (theo khuyến nghị MA-01 gốc — kiểm tra xem bước UI này đã được thêm vào patch hay còn treo) |
| X04 | Bảng toán tử hợp lệ; UC-CAM-02, UC-CAM-03 | Đơn vị % và GB dùng chung kiểu dữ liệu Số hiện có, không phải kiểu mới | A01 | Dev implement theo giả định này; nếu phát sinh case dữ liệu GB không khớp validate kiểu Số hiện tại, báo lại BA ngay, đừng tự mở rộng kiểu dữ liệu |
| X05 | UC-TPL-00, UC-TPL-01 | Chỉ xóa được template chưa được campaign nào tham chiếu | A02 | Dev implement theo nguyên tắc cột "Dùng" hiện có; Test Lead cần test case: xóa template đã Dùng phải bị chặn, kèm thông báo rõ lý do |
| X06 | UC-KH-01, Screen Settings Tab 1 (REQ-CVM-065) | Gửi lại (Retry) cấu hình ở mức từng campaign, không phải cấu hình toàn hệ thống | A03 | Dev implement UI cấu hình Retry nằm trong từng campaign (Screen Settings Tab 1 theo context campaign đang mở), không tạo màn hình cấu hình global riêng |

### 3.4 Tính năng bị defer — không nằm trong scope hiện tại

Không có tính năng nào bị defer trong đợt SYNC này — toàn bộ 14 thay đổi trong Change Set đều đã được patch vào v4 (12 MODIFY + 2 ADD, không có REMOVE nào theo `change-set-2026-08-21.md`).

---

## 4. Scope Delta — So sánh Scope trước SYNC (v3) vs Scope sau SYNC (v4)

**Scope nguồn:** `.claude/output/CVM/urd/urd-srs-v3.md` + `.claude/output/CVM/traceability-map.md` (baseline, 64 REQ: REQ-CVM-001 đến 064)
**Scope thực tế:** `.claude/output/CVM/urd/urd-srs-v4.md` (68 REQ: REQ-CVM-001 đến 068)

### Tính năng được thêm vào (scope creep)

| Tính năng thêm | Lý do thêm | Phase thêm vào | Ảnh hưởng đến độ phức tạp |
|---|---|---|---|
| REQ-CVM-065 — Gửi lại (Retry) sau lần gửi đầu thất bại (mức campaign) | Comment nghiệm thu — không có trong URD v3 | [S1]/[S3] — Change Set thay đổi #7 | Trung bình — thêm cấu hình mới vào Frequency Cap, không thêm màn hình mới |
| REQ-CVM-066 — Kiểm tra ký tự SMS có dấu/không dấu (70/160 ký tự/segment) | Comment nghiệm thu, xác nhận NEW_CONFIRMED qua AskUserQuestion | [S1]/[S3] — Change Set thay đổi #14 | Thấp — chỉ thêm validate cảnh báo tại bước soạn nội dung |
| REQ-CVM-067 — Blacklist toàn hệ thống (UC-BL-04 Thêm thủ công + UC-BL-05 Upload CSV) | Comment nghiệm thu, xác nhận NEW_CONFIRMED; UC-BL-05 phát sinh thêm sau khi đóng OQ-12 | [S1]/[S3] cho UC-BL-04; đóng OQ-12 sau [S4] cho UC-BL-05 | **Cao** — thêm danh sách độc lập, quyền riêng (chỉ Admin), pipeline kiểm tra mới (DNC → BL toàn hệ thống → BL theo campaign-kênh), 2 use case mới |
| REQ-CVM-068 — Nhóm mẫu tin nhắn theo Trigger + cho phép Xóa template chưa dùng | Comment nghiệm thu | [S1]/[S3] — Change Set thay đổi #5 | Thấp đến Trung bình — thêm trường phân loại + 1 hành động mới có điều kiện |

### Tính năng bị loại bỏ

| Tính năng loại bỏ | Lý do | Quyết định của ai |
|---|---|---|
| Cooldown liên campaign (Frequency Cap) | Thay bằng giới hạn theo từng kênh + Weekly/Monthly cap — Cooldown không còn phù hợp với mô hình mới | BA (Change Set thay đổi #6) |
| Bắt buộc nhập endDate | Thay bằng tùy chọn "Vô hạn" | BA (Change Set thay đổi #10) |

### Tính năng thay đổi ý nghĩa

| Tính năng | Mô tả ban đầu (v3) | Mô tả cuối (v4) | Lý do thay đổi |
|---|---|---|---|
| Điều kiện lọc con của phân khúc | Khai báo tại Section 3 (Audience), dùng chung cho mọi kênh | Khai báo tại Section 4 (Message Matrix), riêng theo từng cặp Trigger × Phân khúc × Kênh, kế thừa mặc định khi thêm kênh mới | Cho phép điều kiện lọc khác nhau theo kênh — nhu cầu thực tế từ nghiệm thu |
| Bật lại campaign từ Paused | Luôn tự động về Active | Về Chờ duyệt nếu param/điều kiện lọc bị Sửa trong lúc Paused; giữ Active thẳng nếu không đổi gì | Kiểm soát chất lượng nội dung sau khi sửa |
| Sửa độ ưu tiên (Priority) | Chỉ cấu hình tại Priority Matrix (Admin, không cần duyệt lại) | Thêm đường sửa trực tiếp trên Campaign List (cả QTV và Admin) — nhưng bắt buộc quay về Chờ duyệt | Cho phép thao tác nhanh hơn ở danh sách, nhưng giữ kiểm soát qua approval |
| Blacklist theo campaign | Chọn 1 Chiến dịch + 1 Kênh mỗi lần thêm | Chọn nhiều Chiến dịch và nhiều Kênh cùng lúc | Giảm thao tác lặp lại |
| Che thông tin tại Customer 360 | Hiển thị đầy đủ, không phân biệt che/ẩn | Che (mask) Số điện thoại theo role — nguyên tắc chung, danh sách field đầy đủ và role cụ thể còn OQ-11 | Yêu cầu bảo mật dữ liệu từ nghiệm thu — nhưng **chưa hoàn tất**, đây là điểm rủi ro cần theo dõi (xem mục 3) |

### Đánh giá scope delta

- **Scope creep**: Có — Nhẹ đến Trung bình
- **Nhận xét**: Đây là đợt SYNC xử lý comment nghiệm thu chính thức (13 comment gốc), không phải scope creep tự phát — toàn bộ 14 thay đổi đều có nguồn gốc rõ ràng từ input PO/stakeholder qua bảng nghiệm thu, không có tính năng nào BA tự thêm ngoài yêu cầu. Riêng REQ-CVM-067 (Blacklist toàn hệ thống, gồm cả UC-BL-05 phát sinh muộn) là thay đổi phức tạp nhất về mặt kiến trúc nghiệp vụ (danh sách độc lập + pipeline kiểm tra mới) — nên được Dev Lead ưu tiên review kỹ trước khi break-down task.

---

## 5. Tóm tắt trạng thái session

| Hạng mục | Trạng thái | Ghi chú |
|---|---|---|
| [S1] change-handler | Hoàn thành | Change Set 14 thay đổi (12 MODIFY + 2 ADD), 5 điểm mơ hồ đã làm rõ qua AskUserQuestion, 4 assumption ghi rõ (ASS-01 đến ASS-04) |
| [S2] impact-analysis | Hoàn thành | 21 artifact bị ảnh hưởng (6 CRITICAL, 11 MAJOR, 4 MINOR), 3 checkpoint bổ sung đã được BA xác nhận |
| [S3] artifact-patch | Hoàn thành | `urd-srs-v3.md` giữ nguyên, tạo mới `urd-srs-v4.md`, patch khoảng 35 vị trí xuyên Section II/III/IV |
| [S4] verify (ba-qa-agent) | Hoàn thành | 2 CRITICAL đã fix trực tiếp (workflow diagram S151, 5 tham chiếu chết "Section 3"); 3 MAJOR đã xử lý (MA-01 đóng bằng xác nhận BA, MA-02 vẫn còn treo dưới dạng OQ-11, MA-03 xác nhận là quyết định nghiệp vụ hợp lệ); 3 MINOR chưa sửa (không chặn) |
| ba-postcheck-agent | READY (sau fix PC-01) | PC-01 (UC-BL-04 thiếu trong 3 ma trận tổng thể) đã fix. PC-02 (N/A trong cột Mô tả, Screen 1/9) là pre-existing từ trước v4, không thuộc phạm vi đợt SYNC này, để BA quyết định riêng |
| Traceability Map | Đã cập nhật lên Version 2 | REQ-CVM-065 đến 068 đã thêm; version history ghi rõ 2 lần cập nhật (ADD 4 REQ mới, sau đó MODIFY REQ-CVM-067 khi thêm UC-BL-05) |
| **Bàn giao Dev/Tester** | **Sẵn sàng, có 1 điểm chặn cục bộ** | Toàn bộ 14 thay đổi đã dev-ready, TRỪ phần che thông tin C360/Customer List (OQ-11) — không nên code phần này cho đến khi PO/đội bảo mật trả lời |

**Điều kiện để bàn giao:**
- [x] Tất cả assumption Risky đã được thông báo cho Dev Lead (A04, A07 — mục 2 và 3.3 X01)
- [x] Tất cả open questions đã được liệt kê và giao cho đúng người trả lời (OQ-11 → PO/đội bảo mật)
- [ ] Dev Lead và Test Lead đã đọc Handoff Note (mục 3) — cần xác nhận
- [ ] H01/H02 (mức khẩn Cao, cùng thuộc OQ-11) cần PO/đội bảo mật trả lời **trước khi Dev bắt đầu code phần che thông tin C360** — các phần khác của đợt SYNC này có thể bắt đầu ngay, không cần chờ
