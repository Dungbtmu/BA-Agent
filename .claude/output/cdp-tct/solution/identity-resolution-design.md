# Identity Resolution Design — CDP VNPost/TCT

**Ngày tạo:** 2026-06-26
**Cập nhật:** 2026-07-29 (v2.1) — đối chiếu với tài liệu gốc `CDP.md` (Chương 6 + mục 7.4): áp dụng bảng ngưỡng tin cậy 4 vùng theo mục 6.6.2, bổ sung bộ luật đối sánh đầy đủ theo mục 6.6.1, gắn truy vết tới mã FR-IDR, ghi nhận các điểm bản đang chạy còn lệch.
**Nguồn chuẩn:** `.claude/input/CDP/CDP.md` — Chương 6 (Mô hình dữ liệu và Hợp nhất định danh), mục 7.4 (Phân hệ 3: Hợp nhất định danh), mục 8.9/8.14/8.15 (audit, rủi ro, báo cáo).
**Phụ thuộc vào:** clarification.md, cdp-system-design-brief.md, prototype v3

> Khi nội dung tài liệu này khác với `CDP.md`, tài liệu gốc là chuẩn.

---

## Truy vết tới tài liệu gốc

| Nội dung trong tài liệu này | Mã yêu cầu | Mục trong CDP.md |
|---|---|---|
| Luật đối sánh tuyệt đối | FR-IDR-01 | 6.6.1 |
| Luật đối sánh xác suất | FR-IDR-02 | 6.6.2 |
| Điểm tin cậy và phân loại kết quả | FR-IDR-11 | 6.6.2 |
| Gộp hồ sơ | FR-IDR-06 | 6.8.1 |
| Tách hồ sơ | FR-IDR-07 | 6.8.3 |
| Định danh dùng chung | FR-IDR-08 | 6.8.2, 6.9 case 2/4 |
| Phân biệt vai trò người gửi/người nhận | FR-IDR-09 | 6.8.2, 6.9 case 7/8 |
| Danh sách rà soát thủ công | FR-IDR-12 | 6.6.2 |
| Xử lý xung đột dữ liệu | FR-IDR-13 | 6.9 case 19, 6.10 |
| Nhật ký hợp nhất định danh | FR-IDR-14, FR-GOV-03 | 8.9 nhóm sự kiện 8 |
| Báo cáo gộp/tách hồ sơ | — | 8.15 báo cáo 6 |

---

## Lịch sử quyết định

| Mã | Nội dung | Hiệu lực |
|---|---|---|
| D-01 | Identity Resolution do BE xử lý tự động theo confidence score — càng ít thủ công càng tốt | Đã thay thế 24/07/2026 |
| D-02 | Màn "Đối soát hồ sơ" giữ trong sidebar nhưng read-only — đã xóa nút Merge/Dismiss | Đã thay thế 24/07/2026 |
| D-03 | Tab "Hồ sơ liên kết" thêm vào Customer 360 | Còn hiệu lực |
| D-04 | Kafka ẩn tạm khỏi sidebar | Còn hiệu lực |
| D-05 | Ngưỡng tin cậy và luật đối sánh **áp dụng theo tài liệu gốc** (mục 6.6.1 và 6.6.2), không dùng ngưỡng tự đặt | Chốt 29/07/2026 |
| D-06 | ~~Luồng tách hồ sơ hoãn sang giai đoạn sau~~ | Đã thay thế 30/07/2026 |
| D-07 | Luồng tách hồ sơ **làm trong giai đoạn này, theo đúng tài liệu gốc**: người phụ trách dữ liệu tự tách trực tiếp, bắt buộc điền lý do, hệ thống ghi nhật ký. Không có bước báo cáo và phê duyệt riêng | Chốt 30/07/2026 |

### Mô hình hiện hành (thay cho D-01 và D-02)

Hệ thống tự động gộp các mã nguồn khớp bằng khóa mạnh. Các mã còn lại không tự gộp mà đưa vào danh sách chờ duyệt để người dùng xác nhận từng mã một: tick chọn mã nào thuộc về cùng khách hàng, xem trước hồ sơ chuẩn dự kiến sau khi gộp, rồi mới bấm hợp nhất.

**Lý do thay đổi:** giữ nguyên D-01/D-02 nghĩa là người dùng chỉ được nhìn mà không quyết định được gì ở vùng tin cậy trung bình — trong khi đó chính là vùng cần con người nhất. Mô hình mới cũng khớp với mục 6.6.2 và 6.8.1 trường hợp 5 của tài liệu gốc, vốn đã quy định vùng giữa phải qua hàng đợi phê duyệt của Data Steward.

---

## Solution Overview

| Mã | Nội dung | Trạng thái |
|---|---|---|
| BL-01 | Luật hợp nhất định danh — định nghĩa hành vi cho từng vùng tin cậy | Đã giải theo tài liệu gốc |
| BL-02 | Nhật ký gộp hồ sơ phục vụ giải trình | Yêu cầu giữ nguyên — màn hiển thị đã dựng nhưng chưa gắn vào điều hướng |
| BL-03 | Luồng tách hồ sơ khi gộp nhầm | Đã giải theo tài liệu gốc — người phụ trách dữ liệu tách trực tiếp |

---

## BL-01 — Luật hợp nhất định danh

### Bốn vùng tin cậy (theo mục 6.6.2)

| Điểm tin cậy | Ý nghĩa | Hành vi hệ thống |
|---|---|---|
| **Từ 95% trở lên** | Gần như chắc chắn cùng khách hàng | Tự động gộp, với điều kiện không có xung đột dữ liệu và không vi phạm consent |
| **85% đến dưới 95%** | Khả năng cao là cùng khách hàng | Đưa vào danh sách chờ duyệt để Data Steward/Admin xác nhận |
| **70% đến dưới 85%** | Có liên quan nhưng chưa đủ chắc chắn | Lưu quan hệ nghi vấn trong Identity Graph, **không gộp**, không đưa vào hàng đợi duyệt |
| **Dưới 70%** | Không đủ căn cứ | Không gộp; chỉ dùng làm tín hiệu phân tích nếu phù hợp |

### Luật đối sánh tuyệt đối (theo mục 6.6.1)

| # | Điều kiện khớp | Mức ưu tiên | Hành động | Đã có trong bản đang chạy |
|---|---|---|---|---|
| 1 | Trùng CCCD hợp lệ | Rất cao | Gộp hồ sơ cá nhân, áp dụng masking/mã hóa | Có |
| 2 | Trùng mã số thuế hợp lệ | Rất cao | Gộp vào hồ sơ doanh nghiệp/KHL | Có |
| 3 | Trùng mã số thuế + tên doanh nghiệp gần giống | Rất cao | Gộp, lưu tên chuẩn và tên thay thế | **Chưa** |
| 4 | Trùng số điện thoại + email | Rất cao | Tự động gộp nếu không xung đột vai trò | **Chưa** |
| 5 | Trùng PostID | Cao | Gộp tài khoản số vào hồ sơ khách hàng | Có |
| 6 | Trùng email hợp lệ, không phải email dùng chung | Cao | Gộp hoặc đề xuất gộp | **Chưa** |
| 7 | Trùng mã khách hàng CRM | Cao | Gộp vào hồ sơ tương ứng | **Chưa** |
| 8 | Trùng mã KHL | Cao | Gộp vào hồ sơ doanh nghiệp/KHL | **Chưa** |
| 9 | Trùng User ID App đã xác thực qua số điện thoại/PostID | Cao | Gộp hành vi app vào hồ sơ | **Chưa** |
| 10 | Trùng số điện thoại đã chuẩn hóa, không thuộc danh sách dùng chung | Cao | Gợi ý gộp; chỉ tự động gộp khi có thêm tín hiệu hỗ trợ và đạt ngưỡng | Có |

### Trường hợp cấm gộp tự động (theo mục 6.8.2)

Không được gộp tự động khi chỉ trùng: mã vận đơn · địa chỉ · địa chỉ IP · Device ID. Cũng không gộp tự động khi số điện thoại là hotline/tổng đài/số doanh nghiệp, khi người gửi và người nhận chỉ trùng một thông tin phụ, hoặc khi thiếu consent cho mục đích kích hoạt.

Riêng tên: **không dùng tên làm khóa gộp độc lập** trong mọi trường hợp (mục 6.9 case 11); tên chỉ là tín hiệu hỗ trợ đi kèm định danh khác.

### Đối sánh xác suất (FR-IDR-02 — chưa triển khai)

Tài liệu gốc quy định 10 tín hiệu dùng cho đối sánh xác suất: IP mạng bưu cục · thiết bị/Device ID · Cookie ID · khung giờ gửi hàng · địa chỉ gửi thường dùng · tuyến gửi thường xuyên · loại hàng và trọng lượng thường gặp · tên gần giống · lịch sử chăm sóc tương đồng · hành vi app/web.

Không tín hiệu nào trong nhóm này được dùng làm khóa gộp độc lập — chỉ cộng điểm vào confidence score. Hạng mục này có độ ưu tiên Medium và **chưa được triển khai** trong bản đang chạy.

### Trạng thái hồ sơ

| Trạng thái | Ai tạo | Hành vi hiển thị |
|---|---|---|
| Đã hợp nhất | Hệ thống (từ 95%) hoặc người dùng xác nhận (85–94%) | Hiện trong tab "Hồ sơ đa nguồn" của Customer 360 |
| Chờ duyệt | Hệ thống (85–94%) | Hiện trong danh sách đối soát, có nhãn số lượng chờ xử lý |
| Quan hệ nghi vấn | Hệ thống (70–84%) | Chỉ lưu trong Identity Graph, không đưa vào hàng đợi duyệt |
| Không phải cùng người | Người dùng xác nhận | Gỡ cờ nghi trùng, không hiện lại trong danh sách |
| Đã tách | Người phụ trách dữ liệu tách hồ sơ gộp nhầm | Các mã nguồn được trả về hồ sơ riêng; ghi nhật ký tách; hiện dấu hiệu đã tách trong tab hồ sơ liên kết |

### Luồng xác nhận

1. Vào **Đối soát định danh** — danh sách khách hàng có mã định danh chờ duyệt.
2. Chọn một khách hàng để mở màn **Đối chiếu hồ sơ nghi trùng**.
3. Màn hiển thị hồ sơ chuẩn hiện tại, các mã nguồn hệ thống đã tự gộp, rồi tới bảng đối chiếu các mã chờ duyệt — mỗi mã một cột, so sánh từng trường cạnh nhau.
4. Người dùng tick chọn các mã xác nhận là cùng một khách hàng.
5. Bấm xem trước — hệ thống dựng **hồ sơ chuẩn dự kiến sau khi gộp**: từng trường lấy giá trị từ nguồn nào, các chỉ số giao dịch và tài chính cộng dồn ra sao.
6. Xác nhận hợp nhất, hoặc chọn "Không phải cùng người".

**Trường hợp cần lưu ý:**

- Danh sách chờ duyệt rỗng — hiển thị trạng thái không còn hồ sơ cần xử lý.
- Cặp hồ sơ có dấu hiệu rủi ro (một bên người gửi, một bên người nhận; hoặc số điện thoại dùng chung) — phải hiển thị cảnh báo nổi bật trước khi người dùng quyết định.
- Người dùng không có quyền — màn đối soát không hiện trên thanh điều hướng.

### Bản đang chạy còn lệch so với tài liệu gốc

| Điểm lệch | Bản đang chạy (prototype v3) | Tài liệu gốc |
|---|---|---|
| Ngưỡng tự động gộp | 90% | **95%** |
| Ngưỡng chờ duyệt | 60–89% | **85–94%** |
| Vùng "quan hệ nghi vấn, chưa gộp" | Không có | **70–84%** |
| Ngưỡng loại bỏ | Dưới 60% | **Dưới 70%** |
| Luật số điện thoại + họ tên, ngưỡng 85% | Tự động gộp | **Chờ duyệt** (thuộc vùng 85–94%) |
| Luật "chỉ tên gần đúng" | Tồn tại như một luật riêng | Tên **không được** làm khóa gộp độc lập |
| 6 luật đối sánh còn thiếu | Chưa có | Xem bảng luật đối sánh, cột cuối |
| Đối sánh xác suất | Chưa có | FR-IDR-02, ưu tiên Medium |

Toàn bộ các điểm trên cần được đưa vào phạm vi điều chỉnh trước khi bàn giao Dev.

---

## BL-02 — Nhật ký gộp hồ sơ

### Vì sao bắt buộc

Việc gộp hồ sơ ảnh hưởng trực tiếp đến điểm rủi ro COD — một quyết định tài chính. Khi khách hàng khiếu nại hoặc cơ quan quản lý kiểm tra, VNPost phải trả lời được: hồ sơ này được gộp từ những mã nào, dựa trên căn cứ gì, ai quyết định, vào lúc nào. Đây là yêu cầu bắt buộc theo FR-IDR-14 và FR-GOV-03 (ghi nhật ký **không thể xóa** đối với thao tác merge/unmerge).

### Thông tin phải ghi cho mọi lần gộp

Mục 8.9 nhóm sự kiện 8 quy định tối thiểu: Customer ID trước và sau, lý do, người thực hiện, kết quả xử lý. Chi tiết hóa cho CDP VNPost:

| Trường | Mô tả |
|---|---|
| Mã sự kiện | Định danh duy nhất của lần gộp |
| Loại sự kiện | Tự động gộp / Người dùng xác nhận gộp / Xác nhận không phải cùng người |
| Thời điểm | Ngày giờ xảy ra |
| Mã hồ sơ chuẩn | Mã định danh CDP được giữ lại |
| Danh sách mã nguồn được gộp | Toàn bộ mã nguồn hợp nhất vào hồ sơ chuẩn trong lần này |
| Điểm tin cậy tại thời điểm quyết định | Ghi riêng cho từng mã nguồn |
| Luật kích hoạt | Điều kiện khớp và trường so khớp tương ứng |
| Hệ thống nguồn | Nguồn phát hiện ra mã trùng |
| Người quyết định | Hệ thống tự động, hoặc tên người dùng xác nhận |
| Lý do | Ghi chú tùy chọn khi người dùng xác nhận |
| Điểm số trước và sau khi gộp | Ảnh chụp các điểm rủi ro rời bỏ, rủi ro COD, gian lận |
| Các trường được hợp nhất | Trường nào lấy giá trị từ mã nguồn nào |

### Quy tắc

- Nhật ký **bất biến** — không sửa, không xóa sau khi ghi. Thay đổi về sau chỉ được ghi thêm bản ghi mới.
- Ghi **đồng bộ** với hành động gộp, không ghi sau hoặc ghi theo lô.
- Thời hạn lưu trữ: đang giả định tối thiểu 5 năm — cần xác nhận theo quy định nội bộ VNPost và Luật Bảo vệ dữ liệu cá nhân số 91/2025/QH15.

### Quyền xem

| Vai trò | Phạm vi |
|---|---|
| Admin, Data Steward | Xem toàn bộ sự kiện gộp |
| CSKH đầy đủ | Chỉ xem tóm tắt các lần gộp liên quan đến khách hàng đang mở |
| CSKH cơ bản, Marketing | Không xem |

### Báo cáo gộp/tách hồ sơ

Ngoài nhật ký chi tiết, mục 8.15 báo cáo 6 yêu cầu một **báo cáo tổng hợp**: số hồ sơ đã gộp/tách, số trường hợp đang chờ duyệt, số trường hợp gộp sai, lý do xử lý. Đối tượng sử dụng là Data Steward và bộ phận quản trị dữ liệu. Hạng mục này **chưa có** trong bản đang chạy.

### Hiện trạng hiển thị

Màn danh sách nhật ký gộp đã được dựng trong prototype v3 nhưng **chưa gắn vào thanh điều hướng**, nên hiện chưa có lối vào từ giao diện. Theo quyết định ngày 29/07, tạm giữ nguyên hiện trạng. Trong Customer 360, tab "Nhật ký" hiện chỉ hiển thị nguồn dữ liệu đóng góp và thông tin hồ sơ — chưa phải nhật ký gộp theo yêu cầu ở trên.

---

## BL-03 — Tách hồ sơ khi gộp nhầm

### Căn cứ

Tách hồ sơ là **chức năng bắt buộc** theo tài liệu gốc, xuất hiện ở 12 vị trí:

- **FR-IDR-07 — Tách hồ sơ khách hàng, độ ưu tiên High**, tác nhân là **người phụ trách dữ liệu / hệ thống**
- Mục 6.8: "Merge và Unmerge là chức năng bắt buộc để duy trì tính chính xác của Customer 360"
- Mục 6.8.3: bảng 6 trường hợp tách hồ sơ kèm yêu cầu kiểm soát
- Mục 6.9 case 8: người gửi và người nhận bị gộp nhầm thì áp dụng tách hồ sơ
- Mục 8.7: mọi thao tác gộp và tách phải có nhật ký kèm lý do
- Mục 8.9 nhóm sự kiện 8: nhật ký ghi mã khách hàng trước và sau, lý do, người thực hiện, kết quả
- Mục 8.14 rủi ro 4: "Gộp nhầm hồ sơ khách hàng" ở mức Cao, biện pháp kiểm soát gồm tách hồ sơ
- Mục 8.15 báo cáo 6: báo cáo tổng hợp số hồ sơ đã gộp và đã tách

Tài liệu gốc **không quy định** bước báo cáo hay cấp phê duyệt riêng — người phụ trách dữ liệu là tác nhân thực hiện, điều kiện duy nhất là ghi nhật ký kèm lý do.

### Luồng tách hồ sơ (áp dụng giai đoạn này)

1. Người phụ trách dữ liệu mở hồ sơ khách hàng, vào tab hồ sơ liên kết, xem danh sách mã nguồn đã được gộp vào hồ sơ chuẩn.
2. Chọn mã nguồn cần tách ra. Có thể chọn nhiều mã trong một lần tách.
3. Hệ thống hiển thị **xem trước kết quả tách**: hồ sơ chuẩn còn lại những gì, hồ sơ mới nhận những gì, điểm số dự kiến sau khi tính lại.
4. Người phụ trách dữ liệu **điền lý do — bắt buộc**, chọn trường hợp tách theo bảng dưới.
5. Xác nhận. Hệ thống tách hồ sơ, **trả lại mã nguồn tương ứng**, phân chia lại dữ liệu giao dịch, địa chỉ và điểm số về đúng hồ sơ gốc, **không làm mất lịch sử vận đơn**.
6. Hệ thống tính lại toàn bộ điểm số cho các hồ sơ sau khi tách.
7. Hệ thống ghi nhật ký tách và cập nhật dấu hiệu đã tách trong tab hồ sơ liên kết của cả hai hồ sơ.

**Sáu trường hợp tách hồ sơ theo mục 6.8.3:**

| # | Trường hợp | Cách xử lý | Yêu cầu kiểm soát |
|---|---|---|---|
| 1 | Gộp nhầm hai khách hàng cá nhân | Tách thành hai mã khách hàng riêng, trả lại mã nguồn tương ứng | Lưu nhật ký và lý do tách |
| 2 | Gộp nhầm cá nhân với doanh nghiệp | Tách hồ sơ cá nhân và hồ sơ doanh nghiệp | Cập nhật lại luật đối sánh theo mã số thuế và mã khách hàng lớn |
| 3 | Gộp nhầm người gửi và người nhận | Tách vai trò và lịch sử giao dịch tương ứng | Không làm mất lịch sử vận đơn |
| 4 | Số điện thoại dùng chung cho nhiều người | Tách hồ sơ, đánh dấu số điện thoại là dùng chung | Không dùng số này để gộp tự động nữa |
| 5 | Email dùng chung | Tách hồ sơ, đánh dấu email là dùng chung | Chỉ dùng email làm kênh liên hệ, không làm khóa gộp mạnh |
| 6 | Khách hàng yêu cầu chỉnh sửa hoặc xóa dữ liệu | Tách, xóa hoặc ẩn danh theo quy trình quyền chủ thể dữ liệu | Bảo đảm tuân thủ và lưu vết xử lý |

### Nhật ký tách hồ sơ

| Trường | Mô tả |
|---|---|
| Mã sự kiện | Định danh duy nhất của lần tách |
| Thời điểm | Ngày giờ thực hiện |
| Mã hồ sơ trước khi tách | Hồ sơ chuẩn đang chứa các mã nguồn |
| Mã hồ sơ sau khi tách | Danh sách các hồ sơ hình thành sau khi tách |
| Mã nguồn được trả lại | Từng mã nguồn về hồ sơ nào |
| Liên kết tới lần gộp gốc | Trỏ về bản ghi nhật ký gộp tương ứng |
| Trường hợp tách | Một trong sáu trường hợp ở bảng trên |
| Lý do | Bắt buộc, do người thực hiện điền |
| Người thực hiện | Tên người phụ trách dữ liệu |
| Điểm số đã tính lại | Ghi nhận có tính lại hay không, và giá trị sau khi tính |

Nhật ký tách là **bất biến** như nhật ký gộp — chỉ ghi thêm, không sửa, không xóa. Bản ghi nhật ký gộp gốc **không bị xóa** khi tách; chuỗi sự kiện gộp rồi tách được giữ nguyên để truy vết.

### Vai trò nút Báo cáo trong hồ sơ khách hàng

| Vai trò | Hành vi |
|---|---|
| Người phụ trách dữ liệu, quản trị | Tách trực tiếp, không cần qua nút Báo cáo |
| Chăm sóc khách hàng, kinh doanh, vận hành | Bấm **Báo cáo** kèm lý do — hệ thống ghi nhận và chuyển cho người phụ trách dữ liệu xem xét |
| Tiếp thị | Không thấy nút này |

Báo cáo không tự tạo ra thao tác tách. Người phụ trách dữ liệu xem báo cáo, tự đánh giá và quyết định tách hay không.

### Trường hợp để giai đoạn sau

| Tình huống | Cách xử lý giai đoạn này |
|---|---|
| Hồ sơ đã qua nhiều lần gộp, cần tách một mã nằm giữa chuỗi | Hệ thống **cảnh báo chuỗi gộp phức tạp** và không cho tách trực tiếp; ghi vào danh sách chờ xử lý riêng |
| Giao dịch dùng chung không phân tách được rõ | Ghi vào cả hai hồ sơ kèm dấu hiệu nhận biết, để người phụ trách dữ liệu xử lý tay sau |

Luồng ba bước có phê duyệt — người phát hiện báo cáo, người có thẩm quyền duyệt, hệ thống thực hiện — được ghi nhận là **phương án chặt hơn**, để dành nếu sau này thấy quyền tách trực tiếp bị dùng sai hoặc số lần tách tăng bất thường.

---

## Phụ thuộc

- Bản đang chạy phải được điều chỉnh theo bảng ngưỡng 4 vùng và bộ luật đối sánh của tài liệu gốc trước khi bàn giao Dev.
- Cấu trúc nhật ký gộp cần được xác nhận trước khi thiết kế chi tiết màn hiển thị.
- Bảng phân quyền xem nhật ký cần được xác nhận với PO/client.
- Quy tắc chọn giá trị khi nhiều nguồn cùng cung cấp một trường: tài liệu gốc đã có bảng nguồn ưu tiên ở mục 6.10 cho 12 nhóm dữ liệu. Cần đối chiếu bảng này với cách bản đang chạy đang chọn giá trị ("lấy từ nguồn tin cậy cao nhất").
- Ba yêu cầu ưu tiên High liên quan chưa có trong bản đang chạy: đánh dấu định danh dùng chung (FR-IDR-08), xử lý xung đột dữ liệu định danh (FR-IDR-13), phân quyền theo đơn vị/tỉnh/thành (FR-GOV-14).

---

## Đánh đổi

| Hướng | Ưu điểm | Nhược điểm |
|---|---|---|
| **Áp dụng: tự gộp từ 95%, người xác nhận vùng 85–94%, lưu nghi vấn 70–84%** | Bám đúng tài liệu gốc; giảm mạnh rủi ro gộp nhầm; vùng nghi vấn vẫn được lưu để phân tích về sau | Số cặp được gộp tự động ít hơn; hồ sơ phân mảnh lâu hơn ở giai đoạn đầu |
| Ngưỡng cũ đang chạy (tự gộp từ 90%, xác nhận từ 60%) | Hàng đợi duyệt phủ rộng hơn, gộp được nhiều hơn | Lệch tài liệu gốc; vùng 60–84% đưa vào duyệt trong khi tài liệu gốc coi là chưa đủ căn cứ |
| Tự gộp toàn bộ từ 60% | Ít thủ công nhất | Gộp nhầm xảy ra trước khi có ai kiểm tra |

Về luồng tách hồ sơ:

| Hướng | Ưu điểm | Nhược điểm |
|---|---|---|
| **Áp dụng: người phụ trách dữ liệu tách trực tiếp, bắt buộc lý do và nhật ký** | Đúng tài liệu gốc; khối lượng nhỏ; vá lại lớp phòng vệ cho rủi ro gộp nhầm ngay giai đoạn này | Quyền tách nằm ở một vai trò, phụ thuộc vào sự cẩn thận của người thực hiện |
| Luồng ba bước có phê duyệt | Kiểm soát chặt hơn, khó dùng sai quyền | Tài liệu gốc không yêu cầu; khối lượng lớn hơn nhiều, từng là lý do khiến hạng mục bị hoãn |
| Không làm luồng tách | Tiết kiệm nhất ở giai đoạn đầu | Lệch một yêu cầu ưu tiên cao; rủi ro gộp nhầm mất lớp phòng vệ mà tài liệu gốc chỉ định |

---

## Giả định

| Mã | Giả định | Nếu sai — cần điều chỉnh |
|---|---|---|
| A1 | Số lượng mã chờ duyệt mỗi ngày ở mức người đối soát xử lý được | Cần bổ sung chức năng duyệt hàng loạt, hoặc xem lại ngưỡng |
| A2 | Nhật ký gộp lưu tối thiểu 5 năm | Điều chỉnh yêu cầu lưu trữ và chính sách xóa dữ liệu |
| A3 | Data Steward là vai trò riêng biệt, không phải CSKH cấp cao kiêm nhiệm | Cần đơn giản hóa luồng nếu không có vai trò chuyên trách |

---

## Câu hỏi mở

- [ ] **OQ-01**: Thời hạn lưu giữ nhật ký gộp là bao nhiêu năm theo quy định nội bộ VNPost và Luật Bảo vệ dữ liệu cá nhân số 91/2025/QH15? (Đang giả định 5 năm)
- [ ] **OQ-02**: Nhật ký gộp và tách, cùng báo cáo tổng hợp gộp/tách, đặt ở đâu — tab riêng trong màn Đối soát định danh, hay bổ sung vào tab Nhật ký của Customer 360?
- [ ] **OQ-03**: Vùng 70–84% ("quan hệ nghi vấn, chưa gộp") được lưu trong Identity Graph — người dùng nghiệp vụ có cần nhìn thấy nhóm này ở đâu không, hay chỉ phục vụ phân tích nội bộ?
- [ ] **OQ-04**: Bảng nguồn ưu tiên ở mục 6.10 của tài liệu gốc đã đủ để quyết định giá trị hiển thị trên hồ sơ chuẩn chưa, hay cần bổ sung rule cho các trường: loại khách hàng, nhóm khách hàng, trạng thái, hạng khách hàng thân thiết?
- [ ] **OQ-05**: Quyền tách hồ sơ có cần giới hạn theo cấp không — mọi người phụ trách dữ liệu đều tách được, hay chỉ người được chỉ định riêng? Tài liệu gốc chỉ ghi tác nhân là người phụ trách dữ liệu, không phân cấp.
- [ ] **OQ-06**: Khi hồ sơ đã qua nhiều lần gộp và cần tách một mã nằm giữa chuỗi thì tách đến đâu — chỉ lần gộp gần nhất, hay tách được mã bất kỳ trong chuỗi? Giai đoạn này đang cảnh báo và không cho tách trực tiếp.
- [ ] **OQ-07**: Có cần ngưỡng cảnh báo khi số lần tách tăng bất thường không? Đây là dấu hiệu luật đối sánh đang gộp sai quá nhiều.

---

## Rủi ro

| Mã | Rủi ro | Mức ảnh hưởng | Khả năng xảy ra | Cách giảm thiểu |
|---|---|---|---|---|
| R1 | Số lượng mã chờ duyệt lớn, người đối soát không xử lý kịp, hàng chờ tồn đọng | Cao | Trung bình | Theo dõi thời gian xử lý trung bình từ ngày đầu vận hành; đặt cam kết xử lý trong 2 ngày làm việc |
| R2 | **Xác nhận nhầm hai người khác nhau thành một** | Cao | Trung bình | Bắt buộc hiển thị cảnh báo với cặp có dấu hiệu rủi ro; bắt buộc xem trước hồ sơ chuẩn trước khi hợp nhất |
| R3 | Nhật ký gộp bị thay đổi hoặc mất, không giải trình được trước cơ quan quản lý | Cao | Thấp | Lưu trên vùng riêng chỉ cho phép ghi thêm; kiểm tra tính toàn vẹn định kỳ (thuộc phạm vi SA/Dev) |
| R4 | Bản đang chạy giữ ngưỡng cũ và bộ luật thiếu, dẫn tới hành vi gộp khác với đặc tả khi bàn giao Dev | Cao | Cao | Điều chỉnh theo bảng ngưỡng và bộ luật ở mục BL-01 trước khi bàn giao |
| R5 | Quyền tách hồ sơ bị dùng sai, hoặc tách nhầm một lần gộp vốn đúng | Trung bình | Trung bình | Bắt buộc điền lý do và chọn trường hợp tách; ghi nhật ký bất biến; có báo cáo tổng hợp số lần gộp và tách để phát hiện bất thường; giữ nguyên nhật ký gộp gốc để đối chiếu |

**Lưu ý về R2:** tài liệu gốc chỉ định tách hồ sơ là biện pháp kiểm soát cho rủi ro gộp nhầm (mục 8.14 rủi ro 4). Giai đoạn này **đã có luồng tách**, nên một lần xác nhận nhầm vẫn sửa được. Tuy vậy cảnh báo trước khi hợp nhất và bước xem trước hồ sơ chuẩn vẫn là hàng rào đầu tiên và bắt buộc phải có — sửa sau khi gộp nhầm luôn tốn hơn ngăn từ đầu, vì điểm rủi ro thu hộ có thể đã được dùng để ra quyết định nghiệp vụ trong khoảng thời gian hồ sơ còn sai.

---

## Hiện trạng giao diện (prototype v3, chốt 24/07/2026)

| Màn | Trạng thái |
|---|---|
| Đối soát định danh — danh sách khách hàng có mã nghi trùng | Đang chạy, cần cập nhật theo ngưỡng mới |
| Đối chiếu hồ sơ nghi trùng — bảng so sánh, tick chọn, xem trước, hợp nhất | Đang chạy, cần cập nhật theo ngưỡng mới |
| Rule hợp nhất định danh — bảng luật, chỉ xem | Đang chạy, thiếu 6 luật và sai hành động ở luật số điện thoại + họ tên |
| Customer 360 — tab Hồ sơ liên kết (định danh thay thế + nút Báo cáo) | Đang chạy |
| Customer 360 — tab Hồ sơ đa nguồn (so sánh từng trường theo nguồn) | Đang chạy |
| Nhật ký gộp hồ sơ | Đã dựng nhưng chưa có lối vào |
| Báo cáo gộp/tách hồ sơ | Chưa có |
| Đối sánh xác suất | Chưa có |
| Luồng tách hồ sơ | **Chưa có — cần bổ sung:** chọn mã nguồn để tách, xem trước kết quả, chọn trường hợp tách, điền lý do bắt buộc |
| Tách hồ sơ trong chuỗi gộp nhiều lần | Để giai đoạn sau — giai đoạn này chỉ cảnh báo và không cho tách trực tiếp |
