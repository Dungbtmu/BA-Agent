# Mini-SRS Delta — Quy trình Đặt phòng họp KDI

**Phiên bản:** 1.0
**Ngày:** 26/05/2026
**Mục đích:** Tài liệu bổ sung, KHÔNG thay thế SRS gốc (Fanxipan v1.0). Chỉ mô tả những điểm giữ nguyên, thay đổi và mới so với SRS cũ trong phạm vi UC 7–11.
**Phạm vi:** Chỉ áp dụng cho UC 7–11 (section 2.3, SRS Fanxipan v1.0). UC 1–6 (văn phòng phẩm), UC 12–14 (xe), UC 15–19 (trình ký) KHÔNG bị ảnh hưởng.

**Tài liệu tham chiếu:**
- SRS Fanxipan v1.0 — tháng 3/2026
- BRD KDI v1.0 — 20/05/2026

---

## UC 7: Đăng ký sử dụng phòng họp — [HAS CHANGES]

### Bảng Delta

| Nội dung | SRS cũ (Fanxipan v1.0) | BRD mới (KDI v1.0) | Delta |
|---|---|---|---|
| Actor | Toàn bộ CBNV | Toàn bộ CBNV | `[SAME]` |
| Số loại phòng | 2 loại: Phòng chung / Phòng lãnh đạo | 3 loại: Phòng thường / Phòng hạn chế / Phòng cần phê duyệt | `[CHANGED]` |
| Phòng thường (Phòng chung) — xử lý khi đặt | Lưu ngay, không cần phê duyệt | Tự xác nhận ngay | `[SAME]` |
| Phòng hạn chế (Phòng lãnh đạo) — điều kiện hiển thị | Chỉ người được phân quyền mới đặt được (vẫn hiển thị) | Chỉ người được phân quyền mới thấy và dùng (ẩn hoàn toàn với người khác) | `[CHANGED]` |
| Phòng cần phê duyệt | Không tồn tại | Tất cả CBNV thấy và đặt được, nhưng chờ BHC duyệt trước khi có hiệu lực | `[NEW]` |
| Luồng phê duyệt sau đặt phòng | Không có — đặt xong lưu ngay | Phòng thường: lưu ngay. Phòng cần phê duyệt: chuyển trạng thái "Chờ phê duyệt" | `[CHANGED]` |
| Kiểm tra trùng lịch (BR-01) | Nếu trùng → báo lỗi, không cho đặt | Không đặt trùng phòng + thời gian; ai đặt trước được ưu tiên (BR-01, BR-02) | `[SAME]` |
| Kiểm tra thời gian quá khứ (BR-03) | Có (ngầm định trong SRS) | Không chọn thời gian quá khứ | `[SAME]` |
| Kiểm tra giờ kết thúc > giờ bắt đầu (BR-04) | Có (ngầm định) | Giờ kết thúc > giờ bắt đầu | `[SAME]` |
| Cảnh báo vượt sức chứa (BR-09) | Không có — chỉ có bộ lọc "Số chỗ ngồi" khi tìm phòng | Nếu số người tham gia > sức chứa phòng → cảnh báo (không chặn) | `[NEW]` |
| Field "Số người tham gia" trong form | Không có field riêng — chỉ có "Số chỗ ngồi" làm bộ lọc tìm phòng | Có field "Số người tham gia" trong form đặt phòng (không bắt buộc) | `[CHANGED]` |
| Field "Yêu cầu hỗ trợ" | Có (trong 33 trường form SRS) | Có, không bắt buộc. Nếu có → gửi thông báo cho BHC chuẩn bị (BR-10) | `[SAME]` logic; `[CHANGED]` trigger email |
| Số lượng trường form | 33 trường (theo SRS cũ) | 8 trường chính được mô tả trong BRD | `[CHANGED]` — BRD liệt kê tối giản; cần xác nhận với PO (xem OQ-01) |
| Lặp lịch: cố định tuần/tháng | Có | Không đề cập trong BRD | `[SAME]` — giữ nguyên theo SRS |
| Lặp lịch: ngẫu nhiên | Có | Không đề cập trong BRD | `[SAME]` — giữ nguyên theo SRS |
| Email sau đặt phòng thành công (phòng thường) | ET5 cho người tham gia; ET6 cho nhóm hỗ trợ | Gửi cho người đặt xác nhận đặt thành công; gửi cho BHC nếu có yêu cầu hỗ trợ | `[CHANGED]` — xem chi tiết tại Phần Email Delta |
| Email khi đặt phòng cần phê duyệt | Không tồn tại | Gửi cho BHC thông báo chờ phê duyệt | `[NEW]` |

### Ghi chú cho Tester — UC 7

- **Critical:** Phòng hạn chế phải ẩn hoàn toàn trong danh sách chọn phòng với CBNV không có quyền. Test case: CBNV thường không thấy phòng hạn chế; CBNV có quyền thấy đúng phòng được chỉ định.
- **Critical:** Phòng cần phê duyệt là loại mới hoàn toàn. Cần test toàn bộ luồng: đặt → trạng thái "Chờ phê duyệt" → BHC duyệt/từ chối → thông báo. Không nhầm với phòng thường.
- **Important:** BR-09 (vượt sức chứa) chỉ cảnh báo, không chặn. Test case: nhập số người > sức chứa → hiện cảnh báo nhưng vẫn cho đặt.
- **Open question (OQ-01):** BRD liệt kê 8 trường form, SRS cũ có 33 trường. Cần PO xác nhận: 8 trường BRD là danh sách đầy đủ hay chỉ là các trường highlight mới? Nếu chưa có câu trả lời, Tester dùng 33 trường SRS làm baseline và bổ sung field "Số người tham gia" từ BRD.

---

## UC 8: Xem lịch — [HAS CHANGES]

### Bảng Delta

| Nội dung | SRS cũ (Fanxipan v1.0) | BRD mới (KDI v1.0) | Delta |
|---|---|---|---|
| Actor | CBNV, Admin, Ban Hành chính | CBNV, Admin, Ban Hành chính | `[SAME]` |
| Chế độ hiển thị lịch | 7 ngày / 1 ngày | Không đề cập thay đổi trong BRD | `[SAME]` — giữ nguyên theo SRS |
| Màu thẻ: Đã đặt | Xanh | Không đề cập thay đổi | `[SAME]` |
| Màu thẻ: Đã hủy | Tím | Không đề cập thay đổi | `[SAME]` |
| Màu thẻ: Chờ phê duyệt | Không có | Cần có trạng thái mới để phân biệt lịch chờ duyệt | `[NEW]` — màu sắc cụ thể cần PO xác nhận (OQ-02) |
| Quyền xem chi tiết thẻ | Chỉ Admin xem được; người thường không phải người đặt thì không vào được | Không đề cập thay đổi trong BRD | `[SAME]` — giữ nguyên theo SRS |
| Hiển thị lịch chờ phê duyệt trên calendar | Không có | Lịch có trạng thái "Chờ phê duyệt" phải hiển thị trên lịch | `[NEW]` |

### Ghi chú cho Tester — UC 8

- **Critical:** Khi có loại phòng cần phê duyệt, trên calendar sẽ xuất hiện lịch ở trạng thái "Chờ phê duyệt". Cần test: khung giờ đó có bị hiển thị là trống không? Người khác có thể đặt trùng khung giờ đang chờ duyệt không? (xem OQ-03)
- **Important:** Màu thẻ cho trạng thái "Chờ phê duyệt" chưa được BRD định nghĩa. Tester cần yêu cầu PO/Designer xác nhận trước khi viết test case UI.

---

## UC 9: Hủy đăng ký — [HAS CHANGES]

### Bảng Delta

| Nội dung | SRS cũ (Fanxipan v1.0) | BRD mới (KDI v1.0) | Delta |
|---|---|---|---|
| Actor — người đặt hủy lịch của mình | Người đặt | Người đặt (CBNV chỉ hủy lịch của mình — BR-11) | `[SAME]` |
| Actor — BHC hủy lịch bất kỳ | Ban Hành chính | BHC được hủy tất cả (BR-11) | `[SAME]` |
| Bắt buộc nhập lý do hủy | Có | Có (BRD 6.7) | `[SAME]` |
| Điều kiện hủy được | Chỉ hủy lịch chưa diễn ra | Chỉ hủy lịch chưa diễn ra (BR-07) | `[SAME]` |
| Hủy lịch đang diễn ra | Không cho phép | Không cho phép (BR-07) | `[SAME]` |
| Hủy lịch đã diễn ra | Không cho phép | Không cho phép | `[SAME]` |
| Hủy lịch đã bị hủy | Không đề cập | BRD bổ sung điều kiện tường minh: chỉ hủy lịch "chưa bị hủy" | `[SAME]` logic; BRD tường minh hơn |
| Sau khi hủy: khung giờ mở lại | Ngầm định | Tường minh trong BRD: sau khi hủy → khung giờ mở lại để người khác đặt | `[SAME]` logic; `[CHANGED]` — BRD yêu cầu xử lý tường minh |
| Hủy lịch lặp: hủy 1 hoặc toàn bộ chuỗi | Có | Có (BRD 6.7) | `[SAME]` |
| Email sau khi hủy — người nhận | ET6 cũ: gửi cho người đặt + toàn bộ người tham gia | BRD: gửi cho người đặt. Phần 6.7 đề cập "người tham gia nhận thông báo" nhưng bảng trigger không liệt kê tường minh | `[CHANGED]` — có mâu thuẫn nội bộ trong BRD; xem OQ-05 |
| Hủy lịch đang ở trạng thái "Chờ phê duyệt" | Không tồn tại | Người đặt có thể hủy lịch đang chờ duyệt (BRD không cấm) | `[NEW]` — cần xác nhận hành vi (OQ-04) |

### Ghi chú cho Tester — UC 9

- **Important:** BRD bổ sung điều kiện "chưa bị hủy" tường minh. Test case: thử hủy một lịch đã ở trạng thái "Đã hủy" → hệ thống phải báo lỗi.
- **Important:** Sau khi hủy, khung giờ phải mở lại ngay. Test case: hủy lịch → thử đặt lại cùng phòng, cùng giờ → phải thành công.
- **Open question (OQ-04):** Khi hủy lịch đang "Chờ phê duyệt", BHC có nhận thông báo không?
- **Open question (OQ-05):** Email hủy lịch trong BRD có mâu thuẫn nội bộ. Phần 6.7 nêu "người tham gia nhận thông báo" nhưng bảng trigger email chỉ liệt kê "người đặt". Cần PO xác nhận trước khi hoàn thiện test case.

---

## UC 10: Chỉnh sửa lịch họp — [HAS CHANGES]

### Bảng Delta

| Nội dung | SRS cũ (Fanxipan v1.0) | BRD mới (KDI v1.0) | Delta |
|---|---|---|---|
| Actor — người đặt sửa lịch của mình | Người đặt | Người đặt (BR-11) | `[SAME]` |
| Actor — BHC sửa lịch bất kỳ | Ban Hành chính (quản lý phòng có thể đổi phòng) | BHC được sửa tất cả (BR-11) | `[SAME]` |
| Điều kiện được phép sửa | Chỉ sửa lịch chưa diễn ra | Chỉ sửa lịch chưa diễn ra hoặc chưa bị hủy (BR-07) | `[SAME]` logic; BRD tường minh hơn |
| Sửa lịch đang diễn ra | Không cho phép | Không cho phép (BR-07) | `[SAME]` |
| Áp dụng lại các rule của UC 7 khi sửa | Có | Có (kiểm tra trùng lịch, thời gian hợp lệ theo BR-01, BR-03, BR-04) | `[SAME]` |
| Email thông báo sau khi sửa | Không nêu rõ trong SRS | Bắt buộc: sau khi sửa → gửi thông báo lại cho người tham gia (BR-08) | `[CHANGED]` — SRS không quy định, BRD bắt buộc |
| Sửa thời gian với phòng cần phê duyệt | Không tồn tại (SRS không có loại phòng này) | Nếu thay đổi thời gian với phòng cần phê duyệt → phải phê duyệt lại (BRD 6.6) | `[NEW]` |
| Cảnh báo vượt sức chứa khi sửa | Không có | Áp dụng BR-09 khi sửa (nếu số người > sức chứa → cảnh báo) | `[NEW]` |

### Ghi chú cho Tester — UC 10

- **Critical:** Sau khi sửa lịch, hệ thống PHẢI gửi email thông báo cho người tham gia (BR-08). Test case bắt buộc: sửa bất kỳ trường nào → kiểm tra email gửi đi.
- **Critical:** Khi sửa thời gian của lịch đặt phòng cần phê duyệt → lịch phải quay về trạng thái "Chờ phê duyệt". Test case: sửa giờ họp trong phòng cần phê duyệt → trạng thái thay đổi → BHC nhận yêu cầu phê duyệt mới.
- **Open question (OQ-06):** Nếu chỉ sửa trường không liên quan đến thời gian (tiêu đề, người tham gia) với phòng cần phê duyệt → có phải phê duyệt lại không? BRD chỉ đề cập "thay đổi thời gian". Cần PO xác nhận.
- **Open question (OQ-07):** Sửa lịch đang ở trạng thái "Chờ phê duyệt" (chưa được duyệt lần đầu): được phép hay không? BRD chưa đề cập.

---

## UC 11: Thiết lập phòng họp — [HAS CHANGES]

### Bảng Delta

| Nội dung | SRS cũ (Fanxipan v1.0) | BRD mới (KDI v1.0) | Delta |
|---|---|---|---|
| Actor | Admin / Ban Hành chính | Admin / Ban Hành chính | `[SAME]` |
| Số loại phòng quản lý được | 2 loại: Phòng thông thường / Phòng giới hạn | 3 loại: Phòng thường / Phòng hạn chế / Phòng cần phê duyệt | `[CHANGED]` |
| Cấu hình Phòng thông thường | Tất cả CBNV đặt được | Tất cả CBNV đặt được, xác nhận ngay | `[SAME]` |
| Cấu hình Phòng hạn chế (Phòng giới hạn) | Chỉ người được phân quyền mới đặt được | Chỉ người được BHC chỉ định mới thấy và dùng (ẩn hoàn toàn) | `[CHANGED]` — hành vi hiển thị thay đổi: từ "thấy nhưng không đặt được" sang "ẩn hoàn toàn" |
| Cấu hình Phòng cần phê duyệt | Không tồn tại | Tất cả CBNV thấy và đặt được, nhưng phải chờ BHC phê duyệt | `[NEW]` |
| Kiểm tra trùng mã phòng khi tạo mới | Có | Không đề cập thay đổi | `[SAME]` — giữ nguyên theo SRS |
| Rule xóa/dừng phòng: có lịch tương lai | Hiển thị danh sách lịch tương lai; hủy hết mới xóa được | Không đề cập thay đổi trong BRD | `[SAME]` — giữ nguyên theo SRS |

### Ghi chú cho Tester — UC 11

- **Critical:** Hành vi của Phòng hạn chế đã thay đổi từ "CBNV thấy nhưng không đặt được" sang "CBNV không thấy trong danh sách chọn phòng". Test case: tạo phòng loại "Hạn chế" → đăng nhập bằng tài khoản CBNV thường → phòng phải ẩn hoàn toàn khi chọn phòng đặt lịch.
- **Critical:** Cấu hình loại phòng mới "Cần phê duyệt" cần test: tạo phòng loại này → CBNV thấy và chọn được → sau khi đặt → lịch ở trạng thái "Chờ phê duyệt" → luồng phê duyệt UC mới kích hoạt đúng.
- **Open question (OQ-08):** Nếu phòng có lịch ở trạng thái "Chờ phê duyệt" → phòng có bị chặn xóa không? BRD chưa đề cập.

---

## UC mới: Phê duyệt lịch đặt phòng — [NEW]

UC này chưa tồn tại trong SRS Fanxipan v1.0. Toàn bộ nội dung dưới đây là mới hoàn toàn, được lấy từ BRD KDI v1.0 (mục 6.8).

### Thông tin chung

| Thuộc tính | Giá trị |
|---|---|
| Tên UC | Phê duyệt lịch đặt phòng |
| Actor chính | Ban Hành chính (BHC) |
| Điều kiện tiên quyết | Có ít nhất 1 lịch đặt phòng loại "Cần phê duyệt" ở trạng thái "Chờ phê duyệt" |
| Điều kiện kết thúc (Phê duyệt) | Lịch chuyển sang trạng thái "Đã xác nhận"; người đặt và người tham gia nhận email |
| Điều kiện kết thúc (Từ chối) | Lịch chuyển sang trạng thái "Bị từ chối"; người đặt nhận email kèm lý do từ chối |

### Flow chi tiết

| Bước | Hành động | Hệ thống xử lý |
|---|---|---|
| 1 | BHC vào màn hình quản lý phê duyệt | Hiển thị danh sách toàn bộ lịch đang ở trạng thái "Chờ phê duyệt" |
| 2 | BHC chọn 1 lịch để xem | Hiển thị đầy đủ thông tin: tiêu đề, phòng, ngày giờ, người đặt, người tham gia, yêu cầu hỗ trợ |
| 3a | BHC nhấn "Phê duyệt" | Lịch chuyển trạng thái → "Đã xác nhận"; hệ thống gửi email cho người đặt và người tham gia |
| 3b | BHC nhấn "Từ chối" | Hệ thống yêu cầu nhập lý do từ chối (bắt buộc) |
| 4b | BHC nhập lý do và xác nhận từ chối | Lịch chuyển trạng thái → "Bị từ chối"; hệ thống gửi email cho người đặt kèm lý do |

### Business Rules áp dụng

| Mã BR | Nội dung |
|---|---|
| BR-06 | Lịch phòng cần phê duyệt chỉ có hiệu lực sau khi BHC phê duyệt |
| BR-02 | Trong thời gian chờ phê duyệt, khung giờ được giữ chỗ — ai đặt trước được ưu tiên |
| BR-11 | Chỉ BHC mới thực hiện được hành động phê duyệt/từ chối |

### Validation Rules

| Điều kiện | Hành vi hệ thống |
|---|---|
| BHC nhấn "Từ chối" mà không nhập lý do | Chặn, hiển thị lỗi: "Vui lòng nhập lý do từ chối" |
| Lịch đã bị hủy bởi người đặt trong khi BHC đang xem | Hệ thống thông báo "Lịch này đã bị hủy" và không cho phép phê duyệt/từ chối |
| Lịch đã được phê duyệt hoặc từ chối (BHC khác xử lý trước) | Hiển thị trạng thái đã xử lý, không cho thao tác lần 2 |

### Email kích hoạt từ UC này

| Trigger | Người nhận | Nội dung email |
|---|---|---|
| BHC phê duyệt | Người đặt | Thông báo đặt phòng đã được phê duyệt: tên phòng, ngày giờ |
| BHC phê duyệt | Người tham gia | Thông báo lịch họp đã được xác nhận |
| BHC từ chối | Người đặt | Thông báo đặt phòng bị từ chối kèm lý do từ chối |

### Ghi chú cho Tester — UC mới Phê duyệt

- **Critical:** Toàn bộ UC này là mới. Cần test end-to-end từ bước đặt phòng → chờ duyệt → phê duyệt/từ chối → email.
- **Critical:** Khi BHC từ chối mà không nhập lý do → hệ thống phải chặn. Test case âm: bỏ trống lý do rồi xác nhận từ chối.
- **Critical:** Race condition: 2 BHC cùng mở 1 lịch chờ duyệt, 1 người phê duyệt, 1 người từ chối đồng thời → hệ thống phải tránh xử lý 2 lần.
- **Important:** Trong thời gian lịch "Chờ phê duyệt", khung giờ đó được giữ chỗ theo BR-02. Test case: thử đặt cùng phòng cùng giờ khi đang có lịch chờ duyệt → hệ thống phải báo trùng.
- **Open question (OQ-09):** Lịch bị từ chối có thể tái đặt không (người dùng sửa và gửi lại yêu cầu)? BRD chưa đề cập.

---

## Email Notification Delta

### Bảng so sánh Email Templates

| Trigger | SRS cũ | BRD mới | Delta |
|---|---|---|---|
| Đặt phòng thành công — gửi cho người tham gia (được mời) | ET4: gửi cho người tham gia — thông báo được mời tham gia cuộc họp | Không đề cập thay đổi trong BRD | `[SAME]` — giữ nguyên ET4 |
| Đặt phòng thường thành công — gửi cho người đặt | ET5 gửi cho người tham gia (không phải người đặt) | BRD: gửi cho người đặt — thông báo đặt phòng thành công | `[CHANGED]` — người nhận thay đổi: từ "người tham gia" → "người đặt" |
| Đặt phòng thường có yêu cầu hỗ trợ | ET6: gửi cho nhóm BP tạp vụ, BP IT | BRD: gửi cho Ban Hành chính — thông báo có yêu cầu hỗ trợ | `[CHANGED]` — người nhận thay đổi: từ "BP tạp vụ/IT" → "Ban Hành chính" |
| Đặt phòng cần phê duyệt | Không tồn tại | Gửi cho Ban Hành chính: thông báo chờ phê duyệt | `[NEW]` |
| BHC phê duyệt | Không tồn tại | Gửi cho người đặt: thông báo đặt phòng thành công | `[NEW]` |
| BHC từ chối | Không tồn tại | Gửi cho người đặt: thông báo từ chối kèm lý do | `[NEW]` |
| CBNV cập nhật lịch | SRS không quy định gửi email sau khi sửa | Gửi cho người đặt: thông báo cập nhật lịch họp (BR-08) | `[CHANGED]` — SRS không có, BRD bắt buộc |
| CBNV hoặc BHC hủy lịch | ET6 cũ: gửi cho người đặt + toàn bộ người tham gia | BRD bảng trigger: gửi cho người đặt. BRD 6.7: "người tham gia nhận thông báo" | `[CHANGED]` — có mâu thuẫn nội bộ trong BRD; xem OQ-05 |

### Ghi chú cho Tester — Email

- **Critical:** ET5 (SRS cũ) gửi cho người tham gia khi đặt thành công đã đổi người nhận sang người đặt. Test case: đặt phòng thường → chỉ người đặt nhận email xác nhận, không phải người tham gia.
- **Critical:** Nhóm nhận hỗ trợ thay đổi từ "BP tạp vụ/IT" sang "Ban Hành chính". Cần cập nhật cấu hình email recipient trong hệ thống.
- **Important:** Mâu thuẫn trong BRD về email hủy lịch: phần 6.7 nêu "người tham gia nhận thông báo" nhưng bảng trigger email chỉ liệt kê "người đặt". Tester cần giữ test case cho cả 2 hành vi và chờ PO xác nhận.
- **Open question (OQ-10):** Sau khi sửa lịch (BR-08), email gửi cho người đặt hay người tham gia hay cả hai? BRD có mâu thuẫn: bảng trigger ghi "người đặt" nhưng BR-08 ghi "cập nhật thông báo cho người tham gia".

---

## Tổng kết Delta

### Thống kê

| Loại | Số lượng |
|---|---|
| `[SAME]` | 20 điểm |
| `[CHANGED]` | 13 điểm |
| `[NEW]` | 10 điểm |
| **Tổng** | **43 điểm** |

### Top 5 điểm Tester cần ưu tiên

**1. Loại phòng thứ 3 — Phòng cần phê duyệt `[NEW]`**
Đây là thay đổi nghiệp vụ lớn nhất. Ảnh hưởng đến UC 7, UC 8, UC 10, UC 11 và tạo ra UC mới (Phê duyệt). Cần test end-to-end toàn bộ luồng trước tiên.

**2. Hành vi hiển thị Phòng hạn chế `[CHANGED]`**
SRS cũ: người không có quyền vẫn thấy phòng nhưng không đặt được. BRD mới: phòng ẩn hoàn toàn. Đây là thay đổi hành vi UI dễ bị bỏ sót.

**3. Email bắt buộc sau khi sửa lịch `[CHANGED]`**
SRS cũ không quy định. BRD mới bắt buộc gửi email sau mỗi lần sửa (BR-08). Toàn bộ test case về chỉnh sửa lịch phải bổ sung bước kiểm tra email.

**4. Phê duyệt lại khi sửa thời gian với phòng cần phê duyệt `[NEW]`**
Khi người dùng sửa giờ họp trên phòng cần phê duyệt, lịch quay về "Chờ phê duyệt". Cần test: trạng thái thay đổi đúng, luồng phê duyệt kích hoạt lại, email đi đúng.

**5. Mâu thuẫn email hủy lịch trong BRD — cần làm rõ trước khi test**
BRD có mâu thuẫn nội bộ về việc người tham gia có nhận email hủy lịch không. Tester phải đặt câu hỏi với PO để làm rõ trước khi hoàn thiện test case cho UC 9.

---

## Open Questions cần PO xác nhận

| STT | UC | Câu hỏi |
|---|---|---|
| OQ-01 | UC 7 | 8 trường BRD có thay thế hoàn toàn 33 trường SRS không, hay BRD chỉ liệt kê các trường highlight mới? |
| OQ-02 | UC 8 | Màu sắc cho trạng thái "Chờ phê duyệt" trên calendar là gì? |
| OQ-03 | UC 8 | Khi có lịch "Chờ phê duyệt", khung giờ đó có bị block không cho người khác đặt không? |
| OQ-04 | UC 9 | Khi hủy lịch đang "Chờ phê duyệt", BHC có nhận thông báo không? |
| OQ-05 | UC 9 | Email hủy lịch: ngoài người đặt, người tham gia có nhận không? (mâu thuẫn nội bộ BRD) |
| OQ-06 | UC 10 | Sửa trường không phải thời gian (tiêu đề, người tham gia) với phòng cần phê duyệt → có phải phê duyệt lại không? |
| OQ-07 | UC 10 | Sửa lịch đang ở trạng thái "Chờ phê duyệt" (chưa được duyệt lần đầu): được phép hay không? |
| OQ-08 | UC 11 | Phòng có lịch ở trạng thái "Chờ phê duyệt" → phòng có bị chặn xóa không? |
| OQ-09 | UC mới | Lịch bị từ chối: người đặt có thể sửa và gửi lại yêu cầu phê duyệt không? |
| OQ-10 | Email | Email sau khi sửa lịch (BR-08) gửi cho người đặt hay người tham gia hay cả hai? |
