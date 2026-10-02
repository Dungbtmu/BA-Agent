# QA Report — urd-srs-v4.md (V4.18, SYNC mode — Batch A Priority Redesign + Batch B Blacklist/Nhắc lại/Mẫu tin nhắn)

**Phạm vi review theo yêu cầu orchestrator:**
1. Conflict/mâu thuẫn nội bộ phát sinh do patch (Screen 2/2B/3, UC-CAM-05/06/07 đồng bộ cơ chế vào/gỡ nhóm)
2. Tham chiếu chết / cơ chế cũ còn sót (priority score, Priority Matrix, ô nhập số)
3. Lệch STT trong bảng component sau khi thêm field mới
4. Đồng bộ Function Tree (II.2) / Permission Matrix (II.3) / RBAC (II.4) / Sequence Diagram (II.5)
5. Đảo ngược quyết định v4.4 (Template trigger single→multi-select) — còn giả định 1-template-1-trigger ở đâu không
6. Vấn đề khác phát hiện trong quá trình đọc

**Nguồn đối chiếu:** `.claude/output/cvm/solution/clarification.md`, `.claude/output/cvm/solution/priority-redesign-solution.md` (PASS WITH CONDITIONS, OQ-4 còn BLOCKER tạm hoãn).

---

## CRITICAL — Phải sửa trước khi tiếp tục

**[CR-01]** URD được patch dù solution nguồn còn OQ-4 là BLOCKER chưa chốt

- Vị trí: `priority-redesign-solution.md` Mục 7 (Open Questions) và Mục 9 (Bước tiếp theo)
- Vấn đề: Solution doc ghi rõ "OQ-4 (BLOCKER, tạm hoãn): ... **phải chốt trước khi chạy SYNC patch thật**" và bước tiếp theo đề xuất "3. Jun chốt OQ-4 ... điều kiện bắt buộc trước khi patch URD thật. 4. Chuyển sang chế độ SYNC". Tuy nhiên URD V4.18 đã được patch đầy đủ toàn bộ cụm Độ ưu tiên (khái niệm "Vận hành thường trực" được đưa nguyên vẹn vào UC-CAM-02, Screen 3, II.6.8...) — tức là điều kiện tiên quyết do chính solution đặt ra đã bị bỏ qua. Nếu CNTT xác nhận "Vận hành thường trực" thực chất do hệ thống Core tự gửi và CVM không cần giữ khái niệm này, toàn bộ patch vừa thực hiện (field phân loại, nhánh xử lý tie-break created_at, cột Loại hình chiến dịch, Mục 2.5/2.6 của II.6.8...) sẽ phải viết lại lần nữa — đây là rework lớn, không phải patch nhỏ.
- Khuyến nghị: Xác nhận với Jun xem OQ-4 đã được chốt với CNTT chưa trước khi coi URD V4.18 là final. Nếu chưa chốt, đánh dấu rõ trong URD (ví dụ ghi chú tại II.6.8 và UC-CAM-02) rằng khái niệm "Vận hành thường trực" đang ở trạng thái "provisional — chờ xác nhận ranh giới CVM/Core" để Dev không bắt đầu implement phần này trước khi chốt.

**[CR-02]** Function Tree (II.2) hoàn toàn thiếu "Khối 8: Cấu hình Độ ưu tiên"

- Vị trí: URD-II.2 (Sơ đồ phân cấp chức năng, dòng 467–520)
- Vấn đề: Mục III Use Case Specification đã có hẳn `## Khối 8: Cấu hình Độ ưu tiên (Nhóm ưu tiên liên-campaign theo Trigger)` chứa UC-PRIORITY-01, nhưng Function Tree ở II.2 chỉ liệt kê 7 Khối (Quản lý Chiến dịch → Bảng điều hành vận hành), không có Khối 8 nào tương ứng. Đây là đứt gãy trực tiếp chuỗi traceability bắt buộc "Business Function → Permission Matrix → Use Case → Giao diện" mà `output-schema.md` yêu cầu. Checklist urd-review-checklist.md mục II.2 ghi rõ: "Có chức năng nào được đề cập trong Workflow mà không xuất hiện ở đây không?" — đây đúng là trường hợp đó, mức độ nghiêm trọng hơn vì không phải thiếu 1 chức năng con mà thiếu hẳn 1 Khối chức năng cấp cao.
- Khuyến nghị: Bổ sung "Khối 8: Cấu hình Độ ưu tiên" vào cây Function Tree II.2 (chức năng con: "Xem và sắp xếp Nhóm ưu tiên liên-campaign theo Trigger"), kèm đoạn diễn giải mục đích/giá trị nghiệp vụ giống các Khối khác.

**[CR-03]** Permission Matrix (II.3) và RBAC Matrix (II.4) hoàn toàn thiếu dòng cho UC-PRIORITY-01 / Khối 8

- Vị trí: URD-II.3 (dòng 562–608), URD-II.4.3 (dòng 629–656)
- Vấn đề: UC-PRIORITY-01 có quy tắc phân quyền rất đặc thù và quan trọng — "Chỉ Admin được sắp xếp — QTV Marketing không có quyền chỉnh tại màn hình này". Đây chính xác là loại rule mà Permission Matrix và RBAC Matrix tồn tại để chính thức hóa (cùng mức độ với "Duyệt/Từ chối chiến dịch" đã có dòng riêng). Nhưng cả 2 ma trận đều không có dòng nào cho "Cấu hình Độ ưu tiên" / Khối 8. Hệ quả thực tế: Dev dựng RBAC backend dựa theo II.3/II.4 (nguồn trung tâm) có nguy cơ bỏ sót endpoint sắp xếp ưu tiên khỏi access-control list, trong khi QA viết test phân quyền theo ma trận trung tâm cũng sẽ không có test case nào cho UC-PRIORITY-01.
- Khuyến nghị: Thêm dòng "Cấu hình Độ ưu tiên (Nhóm ưu tiên liên-campaign)" vào II.3 với Admin HT = X, QTV Marketing = – (chỉ xem gián tiếp qua link ở Campaign List/Builder, không có quyền truy cập trực tiếp màn Cấu hình). Thêm dòng tương ứng vào II.4.3 với quyền `VIEW, UPDATE` (Admin — sắp xếp thứ tự) / `–` (QTV).

---

## MAJOR — Ảnh hưởng đáng kể, nên sửa

**[MA-01]** II.6.10 (Nhắc lại) chưa đồng bộ với patch Batch B — vẫn mô tả mô hình "1 cặp cấu hình duy nhất" trong khi UC-CAM-02/Screen 3 đã patch thành N cấp độc lập theo Lịch chung/riêng per kênh/biến thể

- Vị trí: URD-II.6.10 (dòng 1027–1050)
- Vấn đề: Theo changelog V4.18 mục (C), Nhắc lại đã được "tách cấu hình theo đúng lựa chọn Lịch chung/Lịch riêng per kênh đã có — Lịch riêng: mỗi kênh có cặp (Số lần, Khoảng cách) độc lập; có Audience Variant: mỗi biến thể trong từng kênh có cặp độc lập". UC-CAM-02 bước 4a và Screen 3 "Kênh & Lịch gửi" STT 8 đã mô tả đúng mô hình phân cấp này. Nhưng II.6.10 — phần được gắn nhãn "Logic Pipeline Kênh — Yêu cầu nghiệp vụ nội bộ cho Dev", tức là nguồn tham chiếu kỹ thuật chính cho Dev implement — vẫn viết: "**Cấu hình**: tại Campaign Builder ... mỗi campaign tự cấu hình riêng theo trigger của mình. QTV bật/tắt 'Cho phép nhắc lại'; nếu bật: Số lần nhắc lại tối đa ... Khoảng cách tối thiểu ..." — mô tả như thể chỉ có 1 cặp giá trị cho toàn campaign, không hề nhắc đến việc tách theo kênh hay biến thể. Cơ chế hoạt động (4 bước) cũng chỉ nói "Campaign gửi tin lần đầu thành công" mà không làm rõ việc đếm số lần/đặt lịch là theo đơn vị nào (toàn campaign, hay theo từng kênh/biến thể độc lập như đã chốt).
- Khuyến nghị: Viết lại phần "Cấu hình" và "Cơ chế hoạt động" của II.6.10 theo đúng mô hình phân cấp 3 cấp (Lịch chung toàn campaign / theo kênh / theo biến thể trong kênh), đồng bộ với UC-CAM-02 bước 4a và Screen 3 Kênh & Lịch gửi STT 8 — đặc biệt làm rõ đơn vị đếm "số lần đã nhắc" và "lịch kiểm tra tiếp theo" là độc lập theo đúng cấp đang áp dụng (ví dụ kênh SMS và Zalo OA tự đếm riêng, không dùng chung 1 bộ đếm).

**[MA-02]** Sequence Diagram II.5.2 không thể hiện bước áp dụng Nhóm ưu tiên liên-campaign để chọn ra đúng 1 campaign

- Vị trí: URD-II.5.2 (Sequence — Sự kiện kích hoạt và Gửi tin nhắn, dòng 759–849), đối chiếu II.6.8
- Vấn đề: II.6.8 nói rõ "Khi một trigger event khiến nhiều campaign Active cùng match một KH, hệ thống **không gửi tất cả** — chỉ chọn **một campaign** theo thứ tự ưu tiên". Đây là bước nghiệp vụ lõi, ảnh hưởng trực tiếp kết quả gửi tin. Nhưng Sequence Diagram II.5.2 bước 3–4 (GD1) chỉ mô tả "Tra cứu danh sách chiến dịch đang hoạt động có lắng nghe sự kiện này → Danh sách chiến dịch phù hợp" rồi nhảy thẳng sang GD2 (kiểm tra điều kiện lọc), không có bước nào thể hiện việc áp dụng Nhóm ưu tiên liên-campaign để thu gọn "danh sách campaign phù hợp" về đúng 1 campaign trước khi xử lý tiếp. Người đọc Sequence Diagram (đặc biệt Dev mới) sẽ hiểu nhầm là toàn bộ campaign phù hợp đều được xử lý tuần tự qua GD2/GD3, trái với mô tả tường minh ở II.6.8. Đây là gap đã tồn tại từ trước patch (cơ chế Cross-campaign Priority cũ cũng không có trong Sequence Diagram) nhưng nay nghiêm trọng hơn vì mô hình mới phức tạp hơn (N nhóm độc lập, tie-break theo loại hình).
- Khuyến nghị: Bổ sung 1 bước vào GD1 (hoặc đầu GD2) trong Sequence Diagram: "Hệ thống áp dụng Nhóm ưu tiên liên-campaign (II.6.8) trên danh sách campaign phù hợp vừa tra cứu → chọn ra đúng 1 campaign để tiếp tục xử lý; các campaign không được chọn ghi log CAMPAIGN_SKIPPED_PRIORITY". Đồng thời cân nhắc bổ sung dòng tương ứng vào bảng điều kiện dừng pipeline II.6.4 ("Không phải campaign có priority cao nhất trong nhóm trigger — Trong pipeline — Campaign không được chọn: ghi log CAMPAIGN_SKIPPED_PRIORITY, không gửi").

**[MA-03]** UC-CAM-02 bước 1 (Tạo mới) không làm rõ field "Độ ưu tiên" hiển thị nội dung gì khi campaign chưa từng tồn tại (chưa Lưu Nháp lần nào)

- Vị trí: URD-III-UC-CAM-02 (Hoạt động bước 1), Screen 3 Section 1 STT 5
- Vấn đề: Bảng trạng thái quyết định nội dung field "Độ ưu tiên" (tại UC-CAM-01 Quy tắc nghiệp vụ, được UC-CAM-02/Screen 3 STT 5 tham chiếu lại) chỉ định nghĩa cho 5 trạng thái: Draft/Pending/Active/Paused/Ended. Nhưng tại thời điểm QTV đang điền form Tạo mới — trước khi bấm [Lưu Nháp] lần đầu tiên — campaign chưa có bản ghi nào trong hệ thống, chưa có trạng thái chính thức nào được gán. URD liệt kê field này ngay trong "Hoạt động bước 1" của luồng Tạo mới mà không nói rõ: field hiển thị dòng chữ "Draft" ngay cả khi campaign chưa được lưu lần nào, hay field này ẩn/để trống cho đến sau lần Lưu Nháp đầu tiên. Status chip Header (STT 4) có mặc định "Draft" nên suy luận hợp lý là field Độ ưu tiên cũng hiển thị theo "Draft" ngay từ đầu, nhưng điều này không được nói tường minh — Dev phải tự suy luận.
- Khuyến nghị: Thêm 1 câu rõ ràng vào Screen 3 Section 1 STT 5 hoặc UC-CAM-02 Quy tắc nghiệp vụ: "Khi đang tạo campaign mới (chưa từng Lưu Nháp), field hiển thị theo đúng nội dung của trạng thái Draft (đồng bộ với Status chip mặc định tại Header STT 4)".

---

## MINOR — Cải thiện chất lượng

**[MI-01]** Số thứ tự Open Questions nhảy từ 11 sang 13 (thiếu OQ số 12)

- Vị trí: URD, mục "Danh sách mục cần xác nhận với PO / Stakeholder" (gần cuối Screen Settings section, "Còn mở")
- Khuyến nghị: Đánh số lại tuần tự 1–12 (hiện tại nhảy từ 11 thẳng sang 13), hoặc nếu mục 12 đã được resolve và xóa khỏi danh sách "Còn mở" thì nên note rõ "(OQ-12 đã resolved, xem phần Đã đóng)" thay vì để trống số — tránh gây nghi ngờ có nội dung bị mất khi bàn giao cho Dev/Tester.

**[MI-02]** Thuật ngữ "created_at" dùng làm tiêu chí tiebreak nhưng chưa định nghĩa trong I.3 Glossary

- Vị trí: URD-II.6.8, UC-PRIORITY-01 Quy tắc nghiệp vụ — nhiều chỗ dùng `created_at` (thời điểm tạo campaign) làm tiêu chí tie-break cho nhóm "Vận hành thường trực"
- Khuyến nghị: `created_at` là thuật ngữ semi-technical (tên field, không phải thuần nghiệp vụ) xuất hiện lặp lại nhiều lần xuyên URD nhưng không có trong I.3.1 Định nghĩa thuật ngữ. Nên thêm định nghĩa ngắn: "created_at: thời điểm campaign được tạo lần đầu trong hệ thống (ghi nhận ngay khi QTV bấm Lưu Nháp lần đầu tiên)" — làm rõ luôn mốc thời gian chính xác, vì nếu hiểu nhầm là "thời điểm Active" thì tie-break sẽ cho kết quả khác.

**[MI-03]** Assumption A9/A10 của solution (risk vận hành — tập trung toàn bộ quyền ghi vào 1 nơi, không có "second Admin" dự phòng) chưa được phản ánh vào URD dưới dạng ghi chú rủi ro

- Vị trí: `priority-redesign-solution.md` Mục 8 (A9, A10) — không có mục tương ứng trong URD
- Khuyến nghị: Đây là rủi ro vận hành đã được chấp nhận có chủ đích (theo solution), không chặn việc patch URD, nhưng nên có ít nhất 1 dòng ghi chú trong UC-PRIORITY-01 hoặc C.4 (An toàn, bảo mật) nhắc rằng cơ chế mới chỉ có 1 điểm ghi duy nhất (Cài đặt, Admin only) để Dev/SA cân nhắc khi thiết kế xử lý sự cố/quyền khẩn cấp. Không bắt buộc sửa ngay, có thể để lại cho `ba-process-summary-agent` ghi vào Decision Log/Risk Register thay vì sửa URD.

**[MI-04]** II.6.8 không nhắc đến phương án migrate dữ liệu priority cũ (Mục 2.8 của solution)

- Vị trí: URD-II.6.8 — không có đoạn nào về migration
- Vấn đề: Solution Mục 2.8 có hẳn 1 phương án migrate dữ liệu one-time (ánh xạ priority cũ → vị trí index mới theo nhóm, tie-break bằng created_at, coi toàn bộ campaign Active hiện có là "Có thời hạn" theo A4). Đây đúng là thuộc phạm vi vận hành/dữ liệu một lần (không phải business rule vĩnh viễn của hệ thống), nên việc URD không đưa vào là hợp lý về mặt phạm vi — nhưng nên xác nhận rõ đây là quyết định có chủ đích (để lại cho Handoff Note/Decision Log xử lý) chứ không phải bị bỏ sót.
- Khuyến nghị: Không bắt buộc sửa URD. Đảm bảo `ba-process-summary-agent` đưa nội dung Mục 2.8 vào Handoff Note cho Dev/SA khi tổng kết, để không bị thất lạc giữa solution doc và URD.

---

## Đối chiếu các điểm không phát hiện vấn đề (để orchestrator yên tâm, không cần re-check)

- **Screen 2 / Screen 2B / Screen 3 STT 5 (hiển thị Độ ưu tiên theo trạng thái)**: đồng bộ tốt — cùng 1 bảng trạng thái (Draft/Pending/Active/Paused/Ended/Vận hành thường trực), cùng wording, không mâu thuẫn.
- **UC-CAM-05/06/07 cơ chế vào/gỡ nhóm**: đồng bộ tốt — UC-CAM-05 (Duyệt → thêm cuối nhóm), UC-CAM-06 (Kill Switch → gỡ khỏi nhóm), UC-CAM-07 (3 nhánh 1a/1b/1c xử lý đúng theo `pre_pause_status`, phân biệt rõ Active thật vs Pending chưa vào nhóm) khớp nhau và khớp với II.6.8/UC-PRIORITY-01.
- **Tham chiếu cơ chế cũ (priority score, 1-9999, Priority Matrix)**: grep toàn văn xác nhận các cụm này chỉ còn xuất hiện trong bảng "CÁC THAY ĐỔI" (lịch sử changelog) — không còn sót trong nội dung nghiệp vụ hiện hành (II.6.8, UC-CAM-*, UC-PRIORITY-01, Screen 2/2B/3/Settings đều đã patch sạch).
- **STT trong Section 1 Campaign Builder**: thêm STT "3b" (Loại hình chiến dịch) giữa STT 3 và STT 4 không gây lệch số các STT phía sau (4, 5, 6, 7 giữ nguyên) — không có tham chiếu chéo nào trỏ sai.
- **Đảo ngược quyết định v4.4 (Template single→multi-select trigger)**: rà soát Template List (cột "Dùng" đếm theo số campaign tham chiếu, không phụ thuộc số trigger), Template Editor (STT 1b, STT 6 — union payload), Campaign Builder (dropdown "Áp dụng từ Template" lọc theo trigger đang soạn) — toàn bộ đã nhất quán với mô hình multi-trigger, không còn giả định "1 template = 1 trigger" ở bất kỳ công thức hay logic nào đã rà soát.
- **Batch B Blacklist (Screen 6B/6C, UC-BL-01/02)**: dropdown Kênh lọc theo union campaign, dropdown Campaign tự ẩn item đã chọn — patch đầy đủ, nhất quán giữa UC và Screen spec.
- **Screen Admin (UC-CAM-05)**: đã tách sạch hoàn toàn khỏi validate priority, không còn tham chiếu nào sót.
- **Khối 3 (Trigger Management)**: xác nhận không có tham chiếu priority nào cần rà soát, đúng theo Assumption A7 của solution.

---

## Tóm tắt

- CRITICAL: 3 issues (CR-01 xác nhận trạng thái OQ-4 trước khi coi patch là final; CR-02, CR-03 đứt traceability Function Tree + Permission/RBAC Matrix)
- MAJOR: 3 issues (MA-01 II.6.10 chưa đồng bộ Nhắc lại; MA-02 Sequence Diagram thiếu bước chọn campaign theo priority; MA-03 UC-CAM-02 bước 1 mơ hồ trạng thái hiển thị khi chưa lưu)
- MINOR: 4 issues (đánh số OQ, định nghĩa glossary, ghi chú risk vận hành, ghi chú migration)
- Đánh giá tổng thể: **Cần sửa trước khi bàn giao Dev/Tester**

**Khuyến nghị bước tiếp theo cho orchestrator:**
1. Xác nhận trạng thái OQ-4 với Jun trước (CR-01) — đây là quyết định business, không phải lỗi soạn thảo, cần Jun xác nhận hướng đi trước khi đầu tư sửa tiếp các CRITICAL khác.
2. Nếu OQ-4 đã chốt (hoặc Jun quyết định chấp nhận rủi ro rework), quay lại `urd-srs-agent` để patch CR-02, CR-03 (bổ sung Function Tree + Permission/RBAC Matrix) — đây là lỗi cấu trúc độc lập với OQ-4, nên sửa bất kể OQ-4 ra sao.
3. Patch MA-01 (II.6.10) cùng đợt vì cùng thuộc Batch B đã patch dở.
4. MA-02, MA-03 có thể xử lý cùng đợt hoặc để version kế tiếp tùy ưu tiên của Jun.
5. MINOR có thể để lại cho `ba-postcheck-agent` hoặc version sau.
