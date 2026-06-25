# Brainstorm Board — CDP VNPost/TCT

> Khai thác sâu từ nghiên cứu Unomi. Đọc file này để biết **quyết định gì cần đưa ra tiếp theo** trước khi vào thiết kế solution.  
> Nguồn gốc chi tiết → `apache-unomi-research.md` · Tóm tắt dễ đọc → `unomi-tldr.md`

---

## Mục 1 — Rủi ro thiết kế cần xử lý ngay

### R1 — Gộp ID nhầm người: hậu quả domino sang UC-04 và UC-06

Khi gộp ID (pattern T1), nếu hai người khác nhau dùng cùng số điện thoại — vì người cũ đổi số, người mới đăng ký lại — hệ thống sẽ ghép lịch sử của hai người thành một hồ sơ. Người mới "thừa kế" lịch sử xấu của người cũ.

**Hậu quả:**
- UC-04: người mới bị gắn nhãn "địa chỉ rủi ro COD" oan vì lịch sử thất bại COD của người trước dùng số đó
- UC-06: người mới bị tính vào "1 số điện thoại liên kết nhiều tên" vì hồ sơ ghép có cả tên người cũ lẫn tên người mới

**Gợi ý:** Quy tắc gộp phải khớp tối thiểu 2 tiêu chí cùng lúc (ví dụ: cùng SĐT VÀ cùng địa chỉ, hoặc cùng SĐT VÀ tên gần giống). Gộp chỉ dựa vào 1 tiêu chí duy nhất là rủi ro cao.

**Câu hỏi cần hỏi PO:** VNPost có dữ liệu về ngày khách hàng bắt đầu dùng SĐT đó không?

---

### R2 — Nhãn phân loại tự động không tự gỡ khi KH cải thiện

Khi hệ thống gắn nhãn "sắp rời bỏ" hoặc "địa chỉ rủi ro", không có cơ chế nào tự gỡ nhãn đó khi khách hàng đã thay đổi hành vi. Một doanh nghiệp giảm sản lượng vì dịch bệnh 3 tháng sẽ bị Marketing gửi ưu đãi giữ chân liên tục dù đã hồi phục — gây khó chịu, lãng phí ngân sách.

**Gợi ý:** Mỗi nhãn cần có "ngày hết hạn" — ví dụ nhãn "sắp rời bỏ" tự xóa sau 8 tuần nếu không còn dấu hiệu. Cho phép nhân viên kinh doanh bác bỏ nhãn khi có thông tin bối cảnh mà hệ thống không biết.

---

### R3 — Marketing thay đổi ngưỡng khiến báo cáo không so sánh được

Nếu Marketing được quyền tự đổi ngưỡng phát hiện (ví dụ: "inactive" từ 60 ngày đổi thành 45 ngày), hai chiến dịch chạy ở hai thời điểm khác nhau dùng định nghĩa khác nhau. Kết quả chiến dịch không so sánh được vì cùng từ "inactive" nhưng nghĩa đã thay đổi.

**Gợi ý:** Lưu lại lịch sử thay đổi ngưỡng với dấu thời gian. Báo cáo phải hiển thị rõ "áp dụng ngưỡng nào, từ ngày nào".

---

### R4 — Tích hợp nguồn dữ liệu mới mà chưa xin phép KH

VNPost có 16 hệ thống nguồn dữ liệu. Khi tích hợp thêm nguồn mới, không có cơ chế tự động kiểm tra xem khách hàng đã đồng ý cho phép dùng loại dữ liệu này chưa — dễ vi phạm Nghị định 13/2023 mà không biết.

**Gợi ý:** Mỗi nguồn dữ liệu cần được gắn nhãn loại dữ liệu (nhân khẩu học / hành vi giao dịch / vị trí địa lý...). Trước khi tích hợp nguồn mới, phải đối chiếu với danh sách đồng ý hiện có của KH.

---

### R5 — Phân quyền theo tỉnh chưa đủ chi tiết

Phân quyền theo tỉnh (pattern T5) chưa tính đến cấp bưu cục và bưu tá. Nếu bưu tá Hà Nội quận Đống Đa xem được toàn bộ dữ liệu KH tỉnh Hà Nội, đây vẫn là rủi ro quyền riêng tư — bưu tá chỉ nên xem thông tin của đơn hàng họ đang xử lý.

**Câu hỏi cần hỏi PO:** Đơn vị phân quyền nhỏ nhất là gì — tỉnh, bưu cục, hay bưu tá?

---

## Mục 2 — Những thứ Unomi không có, VNPost cần thêm

### P6 — Dữ liệu từ cuộc gặp trực tiếp giữa bưu tá và người nhận

Unomi thiết kế cho tương tác số (click, form, sự kiện trên app). VNPost có kênh đặc thù mà Unomi không tính đến: **bưu tá gặp trực tiếp người nhận**. Cuộc gặp đó tạo ra thông tin quan trọng: người nhận có nhà không, thái độ nhận hàng, lý do từ chối, ghi chú "thường vắng nhà giờ trưa"...

Đây là nguồn thông tin duy nhất về hành vi ngoài đời thực của KH — không hệ thống số nào khác của VNPost có. Nếu bỏ qua, CDP thiếu một chiều dữ liệu không thể bù đắp, đặc biệt ảnh hưởng UC-04.

**Gợi ý:** Cần luồng nhập liệu từ bưu tá ngay sau mỗi lần giao hàng (app di động đơn giản). Kết quả giao hàng + lý do thất bại trở thành sự kiện trong hồ sơ KH.

---

### P7 — Khách hàng doanh nghiệp có nhiều chi nhánh

Unomi mô hình hoá "1 người — 1 hồ sơ". VNPost có KHL là doanh nghiệp với nhiều chi nhánh — công ty mẹ ở Hà Nội, chi nhánh ở Đà Nẵng và TP.HCM, mỗi nơi có người liên hệ riêng nhưng hợp đồng chung.

UC-02 (phát hiện KHL sắp rời bỏ) sẽ bị nhầm nếu chỉ nhìn vào một chi nhánh — chi nhánh Đà Nẵng giảm sản lượng không có nghĩa là công ty sắp rời bỏ VNPost.

**Gợi ý:** Thiết kế mô hình phân cấp: "tài khoản doanh nghiệp" (cấp hợp đồng) chứa nhiều "điểm giao dịch" (cấp chi nhánh). UC-02 phải nhìn tổng ở cả hai cấp.

---

### P8 — Khách giao dịch tại quầy nhưng không có tài khoản

Nhiều người gửi hàng qua quầy CAS mà không tạo tài khoản — chỉ để lại tên và SĐT trên vận đơn. Họ vẫn là KH thực tế, có thể gửi hàng 10 lần/năm. Nếu CDP chỉ phân tích KH đã đăng ký, bỏ sót một lượng lớn người dùng thực tế.

Đặc biệt: UC-04 (rủi ro COD) có thể bỏ sót người nhận vãng lai có lịch sử thất bại nhưng chưa có tài khoản.

**Gợi ý:** Cần quy trình tạo "hồ sơ tạm" cho KH vãng lai dựa trên SĐT. Hồ sơ này có thể được nâng cấp thành hồ sơ chính thức khi KH tạo tài khoản sau này. Pattern T1 (gộp ID) cần tính đến việc hợp nhất hồ sơ tạm vào hồ sơ chính thức.

---

## Mục 3 — Thứ tự xây dựng nếu ngân sách hạn chế

| Thứ tự | Làm gì | Tại sao trước |
|---|---|---|
| **1 — Bắt buộc đầu tiên** | T4: Quản lý đồng ý xử lý dữ liệu | Điều kiện tiên quyết để xây bất cứ thứ gì khác — nếu chưa có mà đã tích hợp 16 nguồn thì mọi phân tích đều có nguy cơ vi phạm Nghị định 13 |
| **2 — Nền tảng** | T1: Gộp ID — hồ sơ KH hợp nhất | Không có T1, mọi phân tích hoạt động trên dữ liệu phân mảnh — 1 KH có 3 ID = 3 người khác nhau, số liệu sai từ gốc |
| **3 — Quick win** | T2 (chỉ UC-04 + UC-06 trước) | Có tác động tài chính đo được ngay: giảm COD thất bại, giảm gian lận. Thuyết phục lãnh đạo tiếp tục đầu tư |
| **Làm sau** | T3 (Marketing tự cấu hình) | Chỉ có giá trị khi đã có ít nhất 1 use case chạy ổn định |
| **Làm sau** | T5 (Phân quyền tỉnh) | Phụ thuộc câu trả lời của Q9 từ client |
| **Phiên bản 2** | UC-02 + UC-03 (giữ chân KHL) | Tác động dài hơi hơn, phức tạp hơn, làm sau khi cơ sở hạ tầng đã ổn định |

---

## Mục 4 — Câu hỏi cần hỏi thêm PO/client

Ngoài 5 câu CRITICAL đã có trong Domain Brief, cần thêm:

**Q10 — SĐT tái sử dụng: VNPost có biết không?**
Nhà mạng Việt Nam thu hồi và cấp lại SĐT sau 3 tháng không dùng. Nếu không có cách phát hiện SĐT đã đổi chủ, rủi ro gộp nhầm ID (R1) rất cao.
→ Mức độ: Cao — ảnh hưởng trực tiếp thiết kế T1.

**Q11 — Kết quả giao hàng chi tiết lưu ở đâu?**
Lý do giao hàng thất bại (không có nhà / từ chối nhận / địa chỉ sai...) hiện đang lưu ở hệ thống nào, dạng gì? Hay bưu tá chỉ ghi trên giấy?
→ Mức độ: Cao — ảnh hưởng thiết kế P6 và UC-04.

**Q12 — KHL doanh nghiệp quản lý theo công ty hay chi nhánh?**
Portal KHL hiện quản lý theo công ty mẹ hay từng chi nhánh? Hợp đồng ký ở cấp nào?
→ Mức độ: Trung bình — ảnh hưởng P7 và UC-02.

**Q13 — Ngưỡng phát hiện (30%, 60 ngày, 3 lần...) đã được kiểm chứng chưa?**
Đây là con số ước đoán ban đầu hay đã được kiểm chứng từ dữ liệu lịch sử? Ai có quyền điều chỉnh sau khi hệ thống đi vào vận hành?
→ Mức độ: Trung bình — ảnh hưởng thiết kế T2 và T3.

**Q14 — Khi phát hiện bất thường, ai làm gì tiếp theo? ← Quan trọng nhất**
Ví dụ khi UC-06 phát hiện "1 SĐT liên kết hơn 5 tên": thông báo cho bộ phận nào, họ xem xét ở màn hình nào, có thể bác bỏ cảnh báo không, hành động cuối cùng là gì (chặn đơn / gắn cờ / chuyển điều tra)? Hiện tại đang làm thủ công thế nào?
→ Mức độ: Cao — nếu không rõ thì hệ thống phát hiện ra mà không ai xử lý cũng vô ích.

---

## Mục 5 — Edge case từng use case

### UC-02 — Phát hiện KHL sắp rời bỏ

| Edge case | Mô tả | Gợi ý xử lý |
|---|---|---|
| Giảm có kế hoạch | KHL đã thông báo với nhân viên KD là tháng này giảm hàng tồn kho, tháng sau tăng lại. Hệ thống vẫn gắn nhãn "sắp rời bỏ" → gửi ưu đãi → KH thấy lạ vì đã báo rồi | Cho phép nhân viên KD ghi chú "giảm có kế hoạch" để tạm thời loại KH khỏi danh sách cảnh báo |
| Doanh nghiệp theo mùa | Công ty bán đồ Tết — sản lượng giảm 80% từ tháng 2 đến tháng 10 mỗi năm. Hoàn toàn bình thường | Phát hiện hành vi theo mùa từ lịch sử năm trước, không tính là dấu hiệu rời bỏ |
| Chuyển kênh, không rời bỏ | KHL giảm giao hàng qua quầy CAS vì đã ký hợp đồng pickup mới. Tổng sản lượng tăng nhưng dữ liệu CAS giảm | Phân tích tổng sản lượng trên tất cả kênh, không chỉ 1 kênh |

### UC-03 — Kéo lại KH cũ (không giao dịch 60 ngày)

| Edge case | Mô tả | Gợi ý xử lý |
|---|---|---|
| KH có khiếu nại đang mở | KH ngừng giao dịch vì đang chờ giải quyết khiếu nại. Gửi ưu đãi lúc này càng làm KH bực bội hơn | Kiểm tra khiếu nại mở trước khi đưa vào danh sách UC-03 |
| KH bán hàng theo mùa | Người bán online chỉ hoạt động dịp lễ Tết — không giao dịch 60 ngày là bình thường | Phân tích lịch sử theo mùa trước khi gắn nhãn "inactive" |

### UC-04 — Địa chỉ nhận hàng rủi ro COD

| Edge case | Mô tả | Gợi ý xử lý |
|---|---|---|
| Địa chỉ tập thể | Ký túc xá, toà nhà văn phòng nhiều công ty — 3 người khác nhau từ chối COD vì lý do riêng → địa chỉ bị gắn cờ → ảnh hưởng đến hàng trăm người khác | Đặt ngưỡng cao hơn hoặc tính tỷ lệ (số lần thất bại / tổng số lần giao) thay vì đếm tuyệt đối |
| Thất bại do bưu tá | Bưu tá không đến giao nhưng ghi là "KH không nhận" → địa chỉ bị oan | Cần trường "lý do thất bại" có phân loại rõ ràng, không chỉ đếm số lần thất bại |
| Sai thông tin từ người gửi | Người gửi ghi sai địa chỉ hoặc tên người nhận — không phải lỗi của địa chỉ | Chỉ tính thất bại khi lý do là "người nhận từ chối" hoặc "không liên lạc được" |

### UC-06 — Phát hiện gian lận

| Edge case | Mô tả | Gợi ý xử lý |
|---|---|---|
| Đại lý giao hàng hộ | Shipper chuyên nghiệp dùng SĐT của mình nhận hàng cho nhiều KH khác nhau — hoàn toàn hợp lệ nhưng liên kết hơn 5 tên | Loại trừ SĐT đã đăng ký là tài khoản đại lý/shipper |
| Hộ gia đình nhiều thế hệ | Cùng SĐT nhà nhưng ông bà, bố mẹ, con cái mỗi người nhận hàng với tên riêng → 6 tên 1 SĐT | Kết hợp thêm điều kiện: địa chỉ nhận hàng có khác nhau không |
| Tên viết không nhất quán | "Nguyễn Thị A", "Thi A Nguyen", "A Nguyen", "Ms. A" — cùng 1 người nhưng hệ thống đếm là 4 tên khác nhau | Dùng đối chiếu tên gần đúng (fuzzy matching) thay vì đếm tên khác biệt hoàn toàn |

---

## Mục 6 — Quyết định BA cần đưa ra

| Quyết định | Ảnh hưởng | Trạng thái |
|---|---|---|
| Quy tắc gộp ID: cần tối thiểu bao nhiêu tiêu chí trùng khớp? | Chất lượng toàn bộ dữ liệu CDP | Chờ trả lời Q10 |
| Đơn vị phân quyền nhỏ nhất là gì (tỉnh / bưu cục / bưu tá)? | Thiết kế T5 | Chờ trả lời Q9 |
| Ngưỡng phát hiện có được thay đổi sau vận hành không? | Thiết kế T2 và T3, ảnh hưởng báo cáo lịch sử | Khuyến nghị: có, nhưng phải ghi lịch sử thay đổi |
| Khi phát hiện bất thường, hành động cuối cùng là gì? | Luồng nghiệp vụ sau UC-04 và UC-06 | Chờ trả lời Q14 |
| Có xử lý KHL doanh nghiệp đa chi nhánh trong giai đoạn đầu không? | Phạm vi UC-02, tăng độ phức tạp đáng kể | Gợi ý: để phiên bản 2 |

---

## Open Questions

- [ ] OQ-1: SĐT có được coi là định danh tin cậy để gộp ID không? VNPost có dữ liệu ngày bắt đầu dùng SĐT không?
- [ ] OQ-2: Kết quả giao hàng chi tiết (lý do thất bại) của bưu tá hiện lưu ở hệ thống nào, dạng gì?
- [ ] OQ-3: Portal KHL quản lý theo công ty hay chi nhánh? Hợp đồng ký ở cấp nào?
- [ ] OQ-4: Ngưỡng phát hiện (30%, 60 ngày, 3 lần, 5 tên) đã được kiểm chứng từ dữ liệu lịch sử chưa?
- [ ] OQ-5: Khi hệ thống phát hiện gian lận (UC-06), quy trình xử lý hiện tại là gì và ai quyết định cuối cùng?
- [ ] OQ-6: Có bao nhiêu tài khoản đại lý/shipper chuyên nghiệp trong hệ thống? (cần loại trừ khỏi UC-06)
