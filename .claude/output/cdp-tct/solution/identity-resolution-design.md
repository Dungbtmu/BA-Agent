# Identity Resolution Design — CDP VNPost/TCT

**Ngày tạo:** 2026-06-26
**Cập nhật:** 2026-07-29 — đồng bộ với prototype v3 (bản giao diện đã chốt ngày 24/07): thay mô hình một ngưỡng chung bằng bộ 7 rule đang chạy thật, ghi nhận D-01/D-02 đã được thay thế, chuyển luồng tách hồ sơ ra ngoài phạm vi giai đoạn này.
**Phiên bản:** v2.0
**Phụ thuộc vào:** clarification.md, cdp-system-design-brief.md, prototype v3

---

## Lịch sử quyết định

| Mã | Nội dung | Hiệu lực |
|---|---|---|
| D-01 | Identity Resolution do BE xử lý tự động theo confidence score — càng ít thủ công càng tốt | **Đã thay thế 24/07/2026** — xem mô hình hiện hành bên dưới |
| D-02 | Màn "Đối soát hồ sơ" giữ trong sidebar nhưng read-only — đã xóa nút Merge/Dismiss | **Đã thay thế 24/07/2026** |
| D-03 | Tab "Hồ sơ liên kết" thêm vào Customer 360 — hiển thị alias IDs, nguồn, ngày merge, confidence score | Còn hiệu lực |
| D-04 | Kafka ẩn tạm khỏi sidebar | Còn hiệu lực |

### Mô hình hiện hành (thay cho D-01 và D-02)

Hệ thống **tự động gộp** các mã nguồn khớp bằng khóa mạnh — CCCD, mã số thuế, PostID. Các mã còn lại không tự gộp mà đưa vào danh sách nghi trùng để **người dùng xác nhận từng mã một**: tick chọn mã nào thuộc về cùng khách hàng, xem trước hồ sơ chuẩn dự kiến sau khi gộp, rồi mới bấm hợp nhất.

**Lý do thay đổi:** giữ nguyên D-01/D-02 nghĩa là người dùng chỉ được nhìn mà không quyết định được gì ở vùng tin cậy trung bình — trong khi đó chính là vùng cần con người nhất. Dữ liệu VNPost có chất lượng không đồng đều (tên có dấu/không dấu, viết tắt, sai chính tả, số điện thoại dùng chung), nên vùng giữa cần người đối chiếu chứ không thể để hệ thống tự quyết. Mô hình mới vẫn giữ tinh thần "ít thủ công nhất có thể" ở chỗ: khóa mạnh vẫn tự gộp hoàn toàn, người dùng chỉ chạm vào phần hệ thống không đủ cơ sở kết luận.

---

## Solution Overview

**Bài toán cần giải:** ba lỗ hổng khiến Dev không biết code gì (BL-01), hệ thống không giải trình được khi bị kiểm tra (BL-02), và không có cơ chế phục hồi khi gộp nhầm (BL-03).

**Trạng thái xử lý:**

| Mã | Nội dung | Trạng thái |
|---|---|---|
| BL-01 | Luật hợp nhất định danh — định nghĩa hành vi cho từng vùng tin cậy | Đã giải — bộ 7 rule đang chạy trên prototype v3 |
| BL-02 | Nhật ký gộp hồ sơ phục vụ giải trình | Yêu cầu giữ nguyên — màn hiển thị đã dựng nhưng chưa gắn vào điều hướng |
| BL-03 | Luồng tách hồ sơ khi gộp nhầm | **Ngoài phạm vi giai đoạn này** — xem mục riêng |

---

## BL-01 — Luật hợp nhất định danh

### Bộ 7 rule đang áp dụng

| Mã | Trường so khớp | Mức mạnh | Ngưỡng tin cậy | Hành động | Ghi chú |
|---|---|---|---|---|---|
| R1 | Số CCCD/CMND | Rất cao | 98% | Tự gộp | Khớp CCCD tuyệt đối — định danh cá nhân duy nhất |
| R2 | Mã số thuế (doanh nghiệp) | Rất cao | 96% | Tự gộp | Khớp MST — định danh doanh nghiệp duy nhất |
| R3 | PostID | Cao | 95% | Tự gộp | Mã định danh nội bộ VNPost |
| R4 | Số điện thoại + họ tên khớp | Trung bình | 85% | Tự gộp | **Điểm chưa nhất quán — xem bên dưới** |
| R5 | Chỉ số điện thoại trùng | Trung bình | 70% | Chờ xác nhận | Có thể dùng chung số (người thân, shipper) |
| R6 | Tên gần đúng + số điện thoại | Thấp | 62% | Chờ xác nhận | Tên viết khác dấu/viết tắt — cần người đối chiếu |
| R7 | Chỉ tên gần đúng | Rất thấp | 50% | Gợi ý tin cậy thấp | Không đủ cơ sở, chỉ hiển thị làm mờ |

### Ba vùng hành vi

| Vùng tin cậy | Hành vi hệ thống | Hiển thị |
|---|---|---|
| **Từ 90% trở lên** | Tự động gộp vào hồ sơ chuẩn, không cần ai duyệt | Ghi nhận trong tab "Hồ sơ đa nguồn" của Customer 360 |
| **60% đến 89%** | Không tự gộp — đưa vào danh sách nghi trùng, tick sẵn để người dùng xác nhận | Màn "Đối soát định danh" và màn đối chiếu |
| **Dưới 60%** | Không tự gộp, không tick sẵn — làm mờ kèm nhãn "tin cậy thấp" | Chỉ hiện khi người dùng chủ động xem, phải tự tick nếu chắc chắn |

### Điểm chưa nhất quán cần chốt — R4

Rule R4 đặt hành động **tự gộp ở mức 85%**, trong khi ba màn hình khác đều tuyên bố với người dùng là hệ thống chỉ tự gộp từ 90% trở lên và vùng 60–89% cần người xác nhận. Sáu rule còn lại đều nhất quán với ba vùng ở trên; chỉ R4 lệch, nên nhiều khả năng đây là sót khi tinh chỉnh rule chứ không phải thiết kế có chủ đích.

Hai hướng xử lý:

| Hướng | Nội dung | Đánh giá |
|---|---|---|
| **A — Hạ R4 xuống "chờ xác nhận"** | Giữ ngưỡng 85%, đổi hành động thành chờ người xác nhận | **Khuyến nghị.** Số điện thoại tại Việt Nam bị thu hồi và cấp lại sau 3 tháng không dùng; tên tiếng Việt trùng nhau rất nhiều. "Nguyễn Văn A + 09xx" khớp 85% vẫn có thể là hai người khác nhau, và gộp nhầm kéo theo sai điểm rủi ro COD — là quyết định tài chính trực tiếp |
| B — Nâng ngưỡng R4 lên 90% | Giữ tự gộp, chỉ siết ngưỡng cho khớp tuyên bố | Nhất quán về mặt con số nhưng vẫn để số điện thoại + tên làm khóa tự gộp, chưa xử lý được rủi ro số tái sử dụng |

Chưa chốt — xem mục Câu hỏi mở.

### Trạng thái hồ sơ

| Trạng thái | Ai tạo | Hành vi hiển thị |
|---|---|---|
| Đã hợp nhất | Hệ thống (khóa mạnh) hoặc người dùng xác nhận | Hiện trong tab "Hồ sơ đa nguồn" của Customer 360 |
| Chờ xác nhận | Hệ thống (vùng 60–89%) | Hiện trong danh sách nghi trùng, có nhãn số lượng mã chờ xử lý trên thanh điều hướng |
| Không phải cùng người | Người dùng xác nhận | Gỡ cờ nghi trùng của khách hàng đó, không hiện lại trong danh sách |
| Gợi ý tin cậy thấp | Hệ thống (dưới 60%) | Làm mờ trong màn đối chiếu, không tick sẵn |

### Luồng xác nhận thực tế

Người dùng xử lý danh sách nghi trùng theo trình tự sau:

1. Vào **Đối soát định danh** — danh sách khách hàng đang có mã định danh nghi trùng, kèm số lượng mã mạnh và mã yếu của từng người.
2. Chọn một khách hàng để mở màn **Đối chiếu hồ sơ nghi trùng**.
3. Màn này hiển thị hồ sơ chuẩn hiện tại và các mã nguồn đã được hệ thống tự gộp (khóa mạnh), sau đó là bảng đối chiếu toàn bộ mã nghi trùng — mỗi mã một cột, so sánh từng trường cạnh nhau.
4. Mã trong vùng 60–89% được tick sẵn; mã dưới 60% làm mờ, mặc định không tick. Người dùng bỏ tick hoặc tick thêm theo đánh giá của mình.
5. Bấm xem trước — hệ thống dựng **hồ sơ chuẩn dự kiến sau khi gộp**: từng trường lấy giá trị từ nguồn nào, và các chỉ số giao dịch, tài chính được cộng dồn ra sao.
6. Xác nhận hợp nhất, hoặc chọn "Không phải cùng người" nếu kết luận đây là các khách hàng khác nhau.

**Trường hợp cần lưu ý:**

- Danh sách nghi trùng rỗng — hiển thị trạng thái không còn hồ sơ chờ xử lý.
- Cặp hồ sơ có dấu hiệu rủi ro (một bên là người gửi, một bên là người nhận; hoặc số điện thoại dùng chung) — phải hiển thị cảnh báo nổi bật trước khi người dùng quyết định.
- Người dùng không có quyền — màn đối soát không hiện trên thanh điều hướng.

---

## BL-02 — Nhật ký gộp hồ sơ

### Vì sao bắt buộc

Việc gộp hồ sơ ảnh hưởng trực tiếp đến điểm rủi ro COD — một quyết định tài chính. Khi khách hàng khiếu nại hoặc cơ quan quản lý kiểm tra, VNPost phải trả lời được: hồ sơ này được gộp từ những mã nào, dựa trên căn cứ gì, ai quyết định, vào lúc nào.

### Thông tin phải ghi cho mọi lần gộp

| Trường | Mô tả |
|---|---|
| Mã sự kiện | Định danh duy nhất của lần gộp |
| Loại sự kiện | Tự động gộp / Người dùng xác nhận gộp / Xác nhận không phải cùng người |
| Thời điểm | Ngày giờ xảy ra |
| Mã hồ sơ chuẩn | Mã định danh CDP được giữ lại |
| Danh sách mã nguồn được gộp | Toàn bộ mã nguồn hợp nhất vào hồ sơ chuẩn trong lần này |
| Độ tin cậy tại thời điểm quyết định | Ghi riêng cho từng mã nguồn |
| Rule kích hoạt | Mã rule (R1–R7) và trường so khớp tương ứng |
| Hệ thống nguồn | Nguồn phát hiện ra mã trùng |
| Người quyết định | Hệ thống tự động, hoặc tên người dùng xác nhận |
| Lý do | Ghi chú tùy chọn khi người dùng xác nhận |
| Điểm số trước và sau khi gộp | Ảnh chụp các điểm rủi ro rời bỏ, rủi ro COD, gian lận |
| Các trường được hợp nhất | Trường nào lấy giá trị từ mã nguồn nào |

### Quy tắc

- Nhật ký **bất biến** — không được sửa hoặc xóa sau khi ghi. Mọi thay đổi về sau chỉ được ghi thêm bản ghi mới.
- Ghi **đồng bộ** với hành động gộp, không ghi sau hoặc ghi theo lô.
- Thời hạn lưu trữ: đang giả định tối thiểu 5 năm — cần xác nhận theo quy định nội bộ VNPost và Luật Bảo vệ dữ liệu cá nhân số 91/2025/QH15.

### Quyền xem

| Vai trò | Phạm vi |
|---|---|
| Admin, Data Steward | Xem toàn bộ sự kiện gộp |
| CSKH đầy đủ | Chỉ xem tóm tắt các lần gộp liên quan đến khách hàng đang mở |
| CSKH cơ bản, Marketing | Không xem |

### Hiện trạng hiển thị

Màn danh sách nhật ký gộp đã được dựng trong prototype v3 (tab "Lịch sử merge" thuộc màn Đối soát & hợp nhất hồ sơ) nhưng **màn này chưa được gắn vào thanh điều hướng**, nên hiện tại chưa có lối vào từ giao diện. Theo quyết định ngày 29/07, tạm giữ nguyên hiện trạng.

Trong Customer 360, tab **Nhật ký** hiện chỉ hiển thị nguồn dữ liệu đóng góp và thông tin hồ sơ — chưa phải nhật ký gộp theo yêu cầu ở trên. Khi mở lại hạng mục này, cần xác định nơi đặt: tab thứ hai trong màn Đối soát định danh, hoặc bổ sung nội dung vào tab Nhật ký của Customer 360.

---

## BL-03 — Tách hồ sơ khi gộp nhầm (ngoài phạm vi giai đoạn này)

### Quyết định

Giai đoạn này **không xây luồng tách hồ sơ**. Nội dung thiết kế bên dưới được giữ lại nguyên vẹn để dùng khi mở lại hạng mục.

### Cách làm hiện tại

Trong tab "Hồ sơ liên kết" của Customer 360 vẫn giữ nút **Báo cáo** kèm dòng gợi ý cho trường hợp nghi ngờ hồ sơ bị gộp nhầm. Khi người dùng bấm, báo cáo được gửi về hệ thống để kiểm tra; **chưa có luồng xử lý tách hồ sơ đi kèm** trong giai đoạn này.

**Điểm cần lưu ý khi bàn giao:** hiện chưa có màn hình nào để người tiếp nhận xem và xử lý các báo cáo này. Cần thống nhất với vận hành nơi tiếp nhận và cách xử lý ngoài hệ thống, tránh tình trạng báo cáo được ghi nhận nhưng không ai theo dõi.

### Thiết kế giữ lại cho giai đoạn sau

Nguyên tắc: không cấp quyền hoàn tác gộp trực tiếp vì quá nguy hiểm — có thể làm mất dữ liệu đã hợp nhất hợp lệ. Thay vào đó dùng luồng ba bước **Báo cáo → Phê duyệt → Tách có kiểm soát**.

**Bước 1 — Người phát hiện báo cáo:** từ tab Hồ sơ liên kết, chọn dòng nghi ngờ sai, điền lý do bắt buộc kèm bằng chứng tùy chọn, gửi đi. Hệ thống tạo yêu cầu và thông báo cho người có thẩm quyền.

**Bước 2 — Người có thẩm quyền xem xét:** mở yêu cầu, đối chiếu hồ sơ tại thời điểm gộp, nhật ký gộp gốc, lý do báo cáo và tác động hiện tại lên các điểm số. Chọn phê duyệt tách hoặc từ chối, đều phải ghi lý do.

**Bước 3 — Hệ thống thực hiện tách:** tách hồ sơ chuẩn thành các hồ sơ riêng, phân chia lại dữ liệu giao dịch, địa chỉ và điểm số về đúng hồ sơ gốc, tính lại toàn bộ điểm số cho các hồ sơ mới, ghi nhật ký tách và thông báo kết quả.

**Các trường hợp cần xử lý riêng khi mở lại:**

| Tình huống | Hướng xử lý |
|---|---|
| Báo cáo sai một lần gộp vốn đúng | Từ chối yêu cầu, ghi lý do, không tách |
| Hồ sơ đã gộp thêm mã khác sau lần gộp bị nghi ngờ | Cảnh báo chuỗi gộp phức tạp, không tự động tách, cần quy trình riêng |
| Báo cáo trùng cho cùng một lần gộp | Chỉ tạo một yêu cầu, báo cho người gửi sau là đã có yêu cầu đang xử lý |
| Giao dịch dùng chung không phân tách được rõ | Ghi vào cả hai hồ sơ kèm dấu hiệu nhận biết để xử lý thủ công sau |

---

## Phụ thuộc

- Bộ 7 rule và ba vùng hành vi phải được thống nhất giữa cấu hình rule và thông điệp hiển thị cho người dùng trước khi bàn giao — hiện R4 đang lệch.
- Cấu trúc nhật ký gộp cần được xác nhận trước khi thiết kế chi tiết màn hiển thị.
- Bảng phân quyền xem nhật ký cần được xác nhận với PO/client.
- Quy tắc chọn giá trị khi nhiều nguồn cùng cung cấp một trường (hồ sơ chuẩn lấy giá trị từ nguồn nào) hiện được mô tả là "lấy từ nguồn tin cậy cao nhất" nhưng chưa có bộ rule cụ thể — màn cấu hình rule ghi nhận đây là hạng mục mở ở giai đoạn sau.

---

## Đánh đổi

| Hướng | Ưu điểm | Nhược điểm |
|---|---|---|
| **Đang áp dụng: tự gộp khóa mạnh, người xác nhận vùng giữa theo từng mã nguồn** | Giảm rủi ro gộp nhầm ở vùng không chắc chắn; người dùng thấy rõ từng mã nguồn trước khi quyết; có bước xem trước hồ sơ chuẩn | Tăng khối lượng công việc thủ công; phụ thuộc năng lực và thời gian của người đối soát |
| Tự gộp toàn bộ từ 60% trở lên | Ít thủ công nhất | Gộp nhầm xảy ra trước khi có ai kiểm tra — và giai đoạn này không có luồng tách để sửa |
| Chỉ tự gộp từ 85%, bỏ qua vùng 60–84% | Giảm khối lượng đối soát | Bỏ sót nhiều cặp trùng thật, hồ sơ tiếp tục phân mảnh |

---

## Giả định

| Mã | Giả định | Nếu sai — cần điều chỉnh |
|---|---|---|
| A1 | Số lượng mã chờ xác nhận mỗi ngày ở mức người đối soát xử lý được (dưới 50 trong giai đoạn đầu) | Cần bổ sung chức năng duyệt hàng loạt cho nhóm tin cậy cao, hoặc nâng ngưỡng |
| A2 | Nhật ký gộp lưu tối thiểu 5 năm | Điều chỉnh yêu cầu lưu trữ và chính sách xóa dữ liệu |
| A3 | Người đối soát là vai trò riêng biệt, không phải CSKH cấp cao kiêm nhiệm | Cần đơn giản hóa luồng nếu không có vai trò chuyên trách |

---

## Câu hỏi mở

- [ ] **OQ-01**: Rule R4 (số điện thoại + họ tên, ngưỡng 85%) chốt theo hướng nào — hạ xuống "chờ xác nhận" hay nâng ngưỡng lên 90%? Đang khuyến nghị hướng hạ xuống chờ xác nhận.
- [ ] **OQ-02**: Thời hạn lưu giữ nhật ký gộp là bao nhiêu năm theo quy định nội bộ VNPost và Luật Bảo vệ dữ liệu cá nhân số 91/2025/QH15? (Đang giả định 5 năm)
- [ ] **OQ-03**: Nhật ký gộp đặt ở đâu khi mở lại — tab riêng trong màn Đối soát định danh, hay bổ sung vào tab Nhật ký của Customer 360?
- [ ] **OQ-04**: Báo cáo nghi ngờ gộp nhầm được gửi về đâu và ai theo dõi, khi giai đoạn này chưa có luồng tách hồ sơ?
- [ ] **OQ-05**: Khi nhiều hệ thống nguồn cung cấp giá trị khác nhau cho cùng một trường (ví dụ CRM ghi đang hoạt động, CAS ghi ngừng hoạt động; CRM phân loại cá nhân, Portal KHL phân loại doanh nghiệp), CDP dùng rule ưu tiên nguồn nào để quyết định giá trị hiển thị trên hồ sơ chuẩn? Cần chốt cho các trường: loại khách hàng, nhóm khách hàng, trạng thái, hạng khách hàng thân thiết.

---

## Rủi ro

| Mã | Rủi ro | Mức ảnh hưởng | Khả năng xảy ra | Cách giảm thiểu |
|---|---|---|---|---|
| R1 | Số lượng mã chờ xác nhận lớn, người đối soát không xử lý kịp, hàng chờ tồn đọng | Cao | Trung bình | Theo dõi thời gian xử lý trung bình từ ngày đầu vận hành; đặt cam kết xử lý trong 2 ngày làm việc |
| R2 | **Xác nhận nhầm hai người khác nhau thành một** | Cao | Trung bình | Bắt buộc hiển thị cảnh báo với cặp có dấu hiệu rủi ro; bắt buộc xem trước hồ sơ chuẩn trước khi hợp nhất |
| R3 | Nhật ký gộp bị thay đổi hoặc mất, không giải trình được trước cơ quan quản lý | Cao | Thấp | Lưu trên vùng riêng chỉ cho phép ghi thêm; kiểm tra tính toàn vẹn định kỳ (thuộc phạm vi SA/Dev) |
| R4 | Rule R4 giữ nguyên tự gộp ở 85%, hệ thống tự gộp cả những cặp chỉ khớp số điện thoại và tên | Cao | Trung bình | Chốt OQ-01 trước khi bàn giao Dev |

**Lưu ý về R2:** ở phiên bản trước, rủi ro này được giảm nhẹ bằng khả năng báo cáo và tách hồ sơ ngay sau khi gộp nhầm. Giai đoạn này **không còn cơ chế sửa sai đó** — một lần xác nhận nhầm sẽ tồn tại cho đến khi có luồng tách hồ sơ. Vì vậy cảnh báo trước khi hợp nhất và bước xem trước hồ sơ chuẩn không còn là tính năng hỗ trợ mà là hàng rào duy nhất.

---

## Hiện trạng giao diện (prototype v3, chốt 24/07/2026)

| Màn | Trạng thái |
|---|---|
| Đối soát định danh — danh sách khách hàng có mã nghi trùng | Đang chạy, có trên thanh điều hướng, hiển thị số lượng chờ xử lý |
| Đối chiếu hồ sơ nghi trùng — bảng so sánh, tick chọn, xem trước, hợp nhất | Đang chạy |
| Rule hợp nhất định danh — bảng 7 rule, chỉ xem | Đang chạy, nằm trong nhóm Cấu hình |
| Customer 360 — tab Hồ sơ liên kết (alias + nút Báo cáo) | Đang chạy |
| Customer 360 — tab Hồ sơ đa nguồn (so sánh từng trường theo nguồn) | Đang chạy |
| Nhật ký gộp hồ sơ | Đã dựng nhưng chưa có lối vào |
| Luồng tách hồ sơ | Ngoài phạm vi giai đoạn này |
