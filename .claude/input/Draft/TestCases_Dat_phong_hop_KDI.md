# Test Cases — Quy trình Đặt phòng họp KDI Group

**Phiên bản:** 1.1
**Ngày:** 26/05/2026
**Nguồn:** Mini-SRS Delta v1.0 + SRS Fanxipan v1.0 + BRD KDI v1.0
**Phạm vi:** Toàn bộ 19 UC — Văn phòng phẩm (UC 1–6) + Đặt phòng họp (UC 7–11 + UC Phê duyệt) + Đăng ký xe (UC 11X–14X) + Trình ký văn bản (UC 15–19)

> **Lưu ý phân biệt nguồn tham chiếu:**
> - UC 7–11 + UC Phê duyệt: dựa theo Mini-SRS Delta (thay đổi từ BRD). Phần [CHANGED]/[NEW] theo BRD mới.
> - UC 1–6, 11X–14X, 15–19: dựa hoàn toàn theo SRS Fanxipan v1.0 (không thay đổi theo BRD).
> - Test case đánh dấu **[Cần PO xác nhận]** hoặc **[OQ-xx]** cần làm rõ trước khi execute.
> - Suffix **X** trong ID (TC-UC11X, 12X, 13X, 14X) dùng cho Đăng ký xe, để phân biệt với UC 11 Thiết lập phòng họp.

---

## UC 7: Đăng ký sử dụng phòng họp

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-UC7-001 | Happy path: Đặt phòng thường thành công | 1. CBNV đăng nhập hệ thống. 2. Truy cập form đặt phòng. 3. Nhập tiêu đề cuộc họp. 4. Chọn phòng thường. 5. Chọn ngày họp. 6. Nhập giờ bắt đầu và giờ kết thúc. 7. Nhấn "Đặt phòng". | Tiêu đề: "Họp Q2 Review". Phòng: Phòng A (loại thường). Ngày: 2026-06-01. Giờ bắt đầu: 09:00. Giờ kết thúc: 10:00. | Lịch được lưu ngay với trạng thái "Đã xác nhận". Email xác nhận gửi cho người đặt. Không gửi email cho người tham gia ở bước này. | High |
| TC-UC7-002 | Happy path: Đặt phòng cần phê duyệt | 1. CBNV đăng nhập hệ thống. 2. Truy cập form đặt phòng. 3. Nhập tiêu đề. 4. Chọn phòng cần phê duyệt. 5. Chọn ngày, giờ bắt đầu, giờ kết thúc. 6. Nhấn "Đặt phòng". | Tiêu đề: "Họp Board". Phòng: Phòng Hội đồng (loại cần phê duyệt). Ngày: 2026-06-02. Giờ bắt đầu: 14:00. Giờ kết thúc: 16:00. | Lịch chuyển trạng thái "Chờ phê duyệt". Email thông báo gửi cho Ban Hành chính. Lịch KHÔNG lưu ngay là "Đã xác nhận". | High |
| TC-UC7-003 | CBNV thấy và chọn được phòng cần phê duyệt | 1. CBNV thường đăng nhập. 2. Truy cập form đặt phòng. 3. Mở dropdown chọn phòng. | Tài khoản: CBNV thường (không có quyền đặc biệt). | Phòng cần phê duyệt hiển thị trong danh sách và có thể chọn được (không bị ẩn, không bị disable). | High |
| TC-UC7-004 | Phòng hạn chế ẩn hoàn toàn với người không có quyền | 1. CBNV không có quyền phòng hạn chế đăng nhập. 2. Truy cập form đặt phòng. 3. Mở dropdown chọn phòng. | Tài khoản: CBNV không có quyền phòng hạn chế. Hệ thống có Phòng VIP (loại hạn chế). | Phòng hạn chế KHÔNG xuất hiện trong danh sách. Người dùng không thấy tên phòng này ở bất kỳ đâu. | High |
| TC-UC7-005 | Người có quyền thấy phòng hạn chế | 1. CBNV có quyền phòng hạn chế đăng nhập. 2. Truy cập form đặt phòng. 3. Mở dropdown chọn phòng. | Tài khoản: CBNV có quyền phòng hạn chế. | Phòng hạn chế xuất hiện trong danh sách và có thể chọn được. | High |
| TC-UC7-006 | Negative: Trùng lịch phòng thường (BR-01) | 1. CBNV A đặt Phòng B ngày 2026-06-03 từ 09:00–10:00 thành công. 2. CBNV B thử đặt cùng Phòng B, ngày 2026-06-03, từ 09:30–10:30. 3. Nhấn "Đặt phòng". | Phòng: Phòng B. Lịch đã tồn tại: 09:00–10:00. Lịch thử đặt: 09:30–10:30 (trùng). | Hệ thống báo lỗi trùng lịch. Không lưu lịch mới. Người đặt trước (CBNV A) được ưu tiên (BR-02). | High |
| TC-UC7-007 | Negative: Chọn thời gian trong quá khứ (BR-03) | 1. CBNV đăng nhập. 2. Truy cập form đặt phòng. 3. Chọn ngày và giờ đã qua. 4. Nhấn "Đặt phòng". | Ngày họp: 2026-05-01 (đã qua). Giờ bắt đầu: 09:00. Giờ kết thúc: 10:00. | Hệ thống báo lỗi không cho chọn thời gian quá khứ. Không lưu lịch. | High |
| TC-UC7-008 | Negative: Giờ kết thúc < giờ bắt đầu (BR-04) | 1. CBNV đăng nhập. 2. Nhập thông tin form đặt phòng. 3. Nhập giờ kết thúc nhỏ hơn giờ bắt đầu. 4. Nhấn "Đặt phòng". | Giờ bắt đầu: 10:00. Giờ kết thúc: 09:00. | Hệ thống báo lỗi "Giờ kết thúc phải lớn hơn giờ bắt đầu". Không lưu lịch. | High |
| TC-UC7-009 | Edge case: Giờ kết thúc bằng giờ bắt đầu (BR-04) | 1. CBNV đăng nhập. 2. Nhập thông tin form. 3. Nhập giờ kết thúc bằng giờ bắt đầu. 4. Nhấn "Đặt phòng". | Giờ bắt đầu: 10:00. Giờ kết thúc: 10:00. | Hệ thống báo lỗi. Không lưu lịch. | High |
| TC-UC7-010 | Edge case: Cảnh báo vượt sức chứa nhưng vẫn cho đặt (BR-09) | 1. CBNV đăng nhập. 2. Chọn phòng có sức chứa 10 người. 3. Nhập số người tham gia là 15. 4. Nhấn "Đặt phòng". | Phòng: sức chứa 10 người. Số người tham gia: 15. | Hệ thống hiển thị cảnh báo "Số người tham gia vượt quá sức chứa phòng" NHƯNG KHÔNG chặn. Lịch vẫn được lưu thành công. | High |
| TC-UC7-011 | Số người tham gia đúng sức chứa — không cảnh báo | 1. CBNV đăng nhập. 2. Chọn phòng có sức chứa 10 người. 3. Nhập số người tham gia là 10. 4. Nhấn "Đặt phòng". | Phòng: sức chứa 10 người. Số người tham gia: 10. | Không hiển thị cảnh báo vượt sức chứa. Lịch lưu thành công. | Medium |
| TC-UC7-012 | Field "Số người tham gia" không bắt buộc | 1. CBNV đăng nhập. 2. Điền đủ các trường bắt buộc. 3. Để trống trường "Số người tham gia". 4. Nhấn "Đặt phòng". | Tiêu đề: "Họp nội bộ". Phòng: Phòng A. Ngày: 2026-06-05. Giờ: 08:00–09:00. Số người tham gia: (để trống). | Lịch lưu thành công. Không có thông báo lỗi về trường số người. | Medium |
| TC-UC7-013 | Negative: Không nhập tiêu đề cuộc họp (bắt buộc) | 1. CBNV đăng nhập. 2. Để trống "Tiêu đề cuộc họp". 3. Điền đầy đủ các trường khác. 4. Nhấn "Đặt phòng". | Tiêu đề: (để trống). Phòng: Phòng A. Ngày: 2026-06-05. Giờ: 08:00–09:00. | Hệ thống báo lỗi "Tiêu đề cuộc họp là bắt buộc". Không lưu lịch. | High |
| TC-UC7-014 | Negative: Không chọn phòng họp (bắt buộc) | 1. CBNV đăng nhập. 2. Nhập tiêu đề. 3. Không chọn phòng họp. 4. Nhấn "Đặt phòng". | Tiêu đề: "Họp kế hoạch". Phòng: (không chọn). | Hệ thống báo lỗi "Phòng họp là bắt buộc". Không lưu lịch. | High |
| TC-UC7-015 | Edge case [OQ-01]: Form thiếu Ngày/Giờ — có cho đặt không? | 1. CBNV đăng nhập. 2. Nhập tiêu đề và chọn phòng. 3. Để trống Ngày họp, Giờ bắt đầu, Giờ kết thúc. 4. Nhấn "Đặt phòng". | Tiêu đề: "Họp tự do". Phòng: Phòng A. Ngày/Giờ: (để trống). | **[Cần PO xác nhận]** BRD liệt kê Ngày/Giờ là "không bắt buộc" nhưng logic kiểm tra trùng lịch cần thời gian. Cần PO xác định hành vi. | High |
| TC-UC7-016 | Happy path: Đặt lịch lặp cố định hàng tuần | 1. CBNV đăng nhập. 2. Chọn phòng thường, điền thông tin họp. 3. Chọn lặp "Hàng tuần". 4. Chọn ngày kết thúc. 5. Nhấn "Đặt phòng". | Phòng: Phòng B. Ngày bắt đầu: Thứ Hai 2026-06-01. Giờ: 08:00–09:00. Lặp: hàng tuần, kết thúc 2026-06-29. | Hệ thống tạo 5 lịch lặp (các thứ Hai trong tháng 6). Mỗi lịch trạng thái "Đã xác nhận". | Medium |
| TC-UC7-017 | Edge case: Lịch lặp có 1 buổi trùng với lịch đã tồn tại | 1. CBNV A đặt Phòng C thứ Tư 2026-06-10 09:00–10:00. 2. CBNV B tạo lịch lặp hàng tuần thứ Tư cho Phòng C từ 09:00–10:00, bắt đầu 2026-06-03, kết thúc 2026-06-17. 3. Nhấn "Đặt phòng". | Phòng: Phòng C. Lịch lặp: thứ Tư hàng tuần 09:00–10:00. Lịch trùng: thứ Tư 2026-06-10 đã được đặt. | Hệ thống báo lỗi trùng lịch tại ngày 2026-06-10. Hiển thị rõ lịch bị lỗi theo BRD 6.5. | High |
| TC-UC7-018 | Field "Yêu cầu hỗ trợ" có nội dung — email gửi đúng nơi | 1. CBNV đăng nhập. 2. Điền đầy đủ form. 3. Nhập nội dung vào "Yêu cầu hỗ trợ". 4. Nhấn "Đặt phòng". | Yêu cầu hỗ trợ: "Cần máy chiếu và microphone". Phòng: Phòng A. Ngày: 2026-06-05. | Email thông báo yêu cầu hỗ trợ gửi đến Ban Hành chính (KHÔNG gửi BP tạp vụ/IT như SRS cũ). | High |
| TC-UC7-019 | Email sau đặt phòng thường thành công — gửi đúng người | 1. CBNV A đặt phòng thường thành công. 2. Kiểm tra hộp thư. | Người đặt: CBNV A (nhanvien-a@kdi.vn). Người tham gia: CBNV B (nhanvien-b@kdi.vn). | Email xác nhận gửi cho CBNV A (người đặt). CBNV B (người tham gia) KHÔNG nhận email xác nhận ở bước này. | High |
| TC-UC7-020 | Negative: Số người tham gia nhập số âm | 1. CBNV đăng nhập. 2. Nhập số người tham gia là -5. 3. Nhấn "Đặt phòng". | Số người tham gia: -5. | Hệ thống báo lỗi định dạng không hợp lệ. Không lưu lịch. | Medium |

---

## UC 8: Xem lịch

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-UC8-001 | Happy path: Xem lịch theo chế độ 7 ngày | 1. CBNV đăng nhập. 2. Truy cập màn hình xem lịch. 3. Chọn chế độ xem "7 ngày". | Ngày hiện tại: 2026-05-26. | Calendar hiển thị 7 ngày liên tiếp. Các lịch trong 7 ngày xuất hiện đúng khung giờ. | High |
| TC-UC8-002 | Happy path: Xem lịch theo chế độ 1 ngày | 1. CBNV đăng nhập. 2. Truy cập màn hình xem lịch. 3. Chọn chế độ xem "1 ngày". | Ngày: 2026-06-01. | Calendar hiển thị chi tiết lịch trong 1 ngày. Các lịch xuất hiện đúng khung giờ. | High |
| TC-UC8-003 | Lịch "Đã đặt" hiển thị màu xanh | 1. Tạo lịch phòng thường thành công. 2. CBNV xem calendar ngày có lịch đó. | Lịch phòng thường trạng thái "Đã xác nhận". | Lịch hiển thị với màu xanh trên calendar. | Medium |
| TC-UC8-004 | Lịch "Đã hủy" hiển thị màu tím | 1. Hủy một lịch đã đặt. 2. CBNV xem calendar ngày có lịch đó. | Lịch đã ở trạng thái "Đã hủy". | Lịch hiển thị với màu tím trên calendar. | Medium |
| TC-UC8-005 | Lịch "Chờ phê duyệt" phải hiển thị trên calendar — không bỏ trống khung giờ | 1. CBNV đặt phòng cần phê duyệt thành công. 2. Bất kỳ CBNV nào xem calendar ngày có lịch đó. | Lịch trạng thái "Chờ phê duyệt", ngày 2026-06-02, giờ 14:00–16:00. | Lịch hiển thị trên calendar tại khung giờ 14:00–16:00. Khung giờ KHÔNG hiển thị là trống. Thẻ lịch có dấu hiệu phân biệt trạng thái "Chờ phê duyệt". | High |
| TC-UC8-006 | Admin xem chi tiết thẻ lịch của người khác | 1. Admin đăng nhập. 2. Xem calendar. 3. Nhấn vào thẻ lịch của CBNV B. | Tài khoản: Admin. Thẻ lịch: đặt bởi CBNV B. | Admin xem được đầy đủ thông tin chi tiết lịch. | High |
| TC-UC8-007 | CBNV thường xem chi tiết thẻ lịch của người khác — bị chặn | 1. CBNV A đăng nhập. 2. Xem calendar. 3. Nhấn vào thẻ lịch do CBNV B đặt. | Tài khoản: CBNV A (không phải Admin, không phải người đặt). Thẻ lịch: đặt bởi CBNV B. | CBNV A không thể xem chi tiết. Hệ thống từ chối hoặc không có action khi nhấn. | High |
| TC-UC8-008 | Người đặt xem chi tiết lịch của chính mình | 1. CBNV A đặt lịch thành công. 2. CBNV A mở calendar, nhấn vào thẻ lịch của mình. | Tài khoản: CBNV A (người đặt lịch). | CBNV A xem được đầy đủ thông tin chi tiết lịch của mình. | High |
| TC-UC8-009 | Edge case: Lịch "Chờ phê duyệt" không chiếm khung giờ của phòng khác | 1. Đặt Phòng Hội đồng trạng thái "Chờ phê duyệt" ngày 2026-06-02 14:00–16:00. 2. Xem calendar của Phòng A cùng khung giờ. | Phòng Hội đồng: lịch Chờ phê duyệt 14:00–16:00. Phòng A: không có lịch. | Calendar Phòng A không hiển thị lịch từ Phòng Hội đồng. Khung giờ 14:00–16:00 của Phòng A hiển thị trống. | Medium |
| TC-UC8-010 | Edge case: Lịch "Chờ phê duyệt" giữ chỗ — thử đặt trùng cùng phòng cùng giờ | 1. Phòng Hội đồng có lịch "Chờ phê duyệt" 14:00–16:00. 2. CBNV khác vào form đặt phòng, chọn cùng phòng, cùng giờ. | Phòng Hội đồng. Lịch chờ phê duyệt: 14:00–16:00. Lịch mới thử đặt: 15:00–17:00 (trùng). | Hệ thống báo trùng lịch. Không cho đặt thêm vào khung giờ đang được giữ chỗ (BR-02). | High |

---

## UC 9: Hủy đăng ký

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-UC9-001 | Happy path: Người đặt hủy lịch của mình | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra. 3. Nhấn "Hủy". 4. Nhập lý do hủy. 5. Xác nhận hủy. | Tài khoản: CBNV A (người đặt). Lịch: ngày 2026-06-10, chưa diễn ra. Lý do: "Thay đổi kế hoạch". | Lịch chuyển trạng thái "Đã hủy". Khung giờ mở lại ngay lập tức. | High |
| TC-UC9-002 | Happy path: BHC hủy lịch của người khác (BR-11) | 1. BHC đăng nhập. 2. Truy cập lịch của CBNV B. 3. Nhấn "Hủy". 4. Nhập lý do. 5. Xác nhận. | Tài khoản: BHC. Lịch của: CBNV B, ngày 2026-06-10. Lý do: "Phòng cần sử dụng cho sự kiện khẩn cấp". | Lịch chuyển "Đã hủy". Khung giờ mở lại ngay. | High |
| TC-UC9-003 | Negative: Hủy mà không nhập lý do | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra. 3. Nhấn "Hủy". 4. Để trống lý do. 5. Xác nhận. | Lý do hủy: (để trống). | Hệ thống báo lỗi "Lý do hủy là bắt buộc". Không thực hiện hủy. | High |
| TC-UC9-004 | Negative: CBNV thường hủy lịch của người khác — bị chặn | 1. CBNV A đăng nhập. 2. Truy cập lịch của CBNV B. 3. Thử nhấn "Hủy". | Tài khoản: CBNV A (không phải BHC, không phải người đặt). Lịch của: CBNV B. | Hệ thống không cho phép. Nút "Hủy" không hiển thị hoặc bị vô hiệu hóa. | High |
| TC-UC9-005 | Negative: Hủy lịch đã diễn ra (BR-07) | 1. CBNV A đăng nhập. 2. Chọn lịch đã qua. 3. Thử nhấn "Hủy". | Lịch: ngày 2026-05-20 (đã qua). Trạng thái: "Đã xác nhận". | Hệ thống không cho phép hủy. Nút "Hủy" ẩn hoặc vô hiệu hóa. | High |
| TC-UC9-006 | Negative: Hủy lịch đã ở trạng thái "Đã hủy" | 1. CBNV A đăng nhập. 2. Tìm lịch đã bị hủy trước đó. 3. Thử nhấn "Hủy" lần nữa. | Lịch: trạng thái "Đã hủy". | Hệ thống không cho phép hủy lần 2. Nút "Hủy" ẩn hoặc vô hiệu hóa. | High |
| TC-UC9-007 | Hủy lịch đang "Chờ phê duyệt" — được phép | 1. CBNV A đặt phòng cần phê duyệt, lịch ở "Chờ phê duyệt". 2. CBNV A chọn lịch đó. 3. Nhấn "Hủy". 4. Nhập lý do. 5. Xác nhận. | Tài khoản: CBNV A (người đặt). Lịch: "Chờ phê duyệt". Lý do: "Đổi kế hoạch". | Hệ thống cho phép hủy. Lịch chuyển "Đã hủy". Khung giờ mở lại. | High |
| TC-UC9-008 | Khung giờ mở lại ngay sau hủy — người khác đặt được ngay | 1. CBNV A hủy lịch Phòng D ngày 2026-06-10 09:00–10:00. 2. Ngay sau đó, CBNV B thử đặt Phòng D cùng ngày cùng giờ. | Phòng D, ngày 2026-06-10, 09:00–10:00. | CBNV B đặt được thành công. Không còn báo lỗi trùng lịch. | High |
| TC-UC9-009 | Email sau hủy — Hành vi 1: gửi cho người đặt [OQ-05 — cần PO xác nhận] | 1. CBNV A hủy lịch thành công. 2. Kiểm tra hộp thư người đặt và người tham gia. | Người đặt: CBNV A. Người tham gia: CBNV B, CBNV C. | **[Cần PO xác nhận — mâu thuẫn nội bộ BRD]** Hành vi 1 (theo bảng trigger BRD): Email hủy gửi cho CBNV A (người đặt). CBNV B, C không nhận. | High |
| TC-UC9-010 | Email sau hủy — Hành vi 2: gửi cho người tham gia [OQ-05 — cần PO xác nhận] | 1. CBNV A hủy lịch thành công. 2. Kiểm tra hộp thư người tham gia. | Người đặt: CBNV A. Người tham gia: CBNV B (nhanvien-b@kdi.vn), CBNV C (nhanvien-c@kdi.vn). | **[Cần PO xác nhận — mâu thuẫn nội bộ BRD]** Hành vi 2 (theo phần 6.7 BRD): Email hủy gửi cho CBNV B và CBNV C (người tham gia). | High |
| TC-UC9-011 | Happy path: Hủy 1 buổi trong chuỗi lịch lặp | 1. CBNV A có chuỗi lịch lặp hàng tuần. 2. Chọn 1 buổi cụ thể. 3. Nhấn "Hủy". 4. Chọn "Hủy buổi này". 5. Nhập lý do. 6. Xác nhận. | Chuỗi lịch lặp hàng tuần 5 buổi. Buổi hủy: buổi thứ 3. Lý do: "Nghỉ lễ". | Chỉ buổi thứ 3 chuyển "Đã hủy". 4 buổi còn lại vẫn "Đã xác nhận". Khung giờ buổi 3 mở lại. | High |
| TC-UC9-012 | Happy path: Hủy toàn bộ chuỗi lịch lặp | 1. CBNV A có chuỗi lịch lặp. 2. Chọn 1 buổi. 3. Nhấn "Hủy". 4. Chọn "Hủy toàn bộ chuỗi". 5. Nhập lý do. 6. Xác nhận. | Chuỗi lịch lặp: 5 buổi. Lý do: "Dự án kết thúc". | Toàn bộ 5 buổi chuyển "Đã hủy". Tất cả khung giờ mở lại. | High |

---

## UC 10: Chỉnh sửa lịch họp

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-UC10-001 | Happy path: Người đặt sửa thông tin lịch phòng thường | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra của mình. 3. Nhấn "Chỉnh sửa". 4. Thay đổi tiêu đề. 5. Lưu. | Tài khoản: CBNV A (người đặt). Lịch: phòng thường, chưa diễn ra. Tiêu đề mới: "Họp Q2 Review v2". | Lịch cập nhật thành công. Email thông báo thay đổi gửi cho người tham gia (BR-08). | High |
| TC-UC10-002 | Happy path: BHC sửa lịch của người khác (BR-11) | 1. BHC đăng nhập. 2. Chọn lịch của CBNV B chưa diễn ra. 3. Nhấn "Chỉnh sửa". 4. Thay đổi giờ họp. 5. Lưu. | Tài khoản: BHC. Lịch của CBNV B: phòng thường 09:00–10:00. Giờ mới: 10:00–11:00 (không trùng). | Lịch cập nhật thành công. Email thông báo gửi người tham gia (BR-08). | High |
| TC-UC10-003 | Negative: CBNV thường sửa lịch của người khác — bị chặn | 1. CBNV A đăng nhập. 2. Truy cập lịch của CBNV B. 3. Thử nhấn "Chỉnh sửa". | Tài khoản: CBNV A (không phải BHC, không phải người đặt). Lịch của: CBNV B. | Hệ thống không cho phép. Nút "Chỉnh sửa" ẩn hoặc vô hiệu hóa. | High |
| TC-UC10-004 | Negative: Sửa lịch đã diễn ra (BR-07) | 1. CBNV A đăng nhập. 2. Chọn lịch đã qua. 3. Thử nhấn "Chỉnh sửa". | Lịch: ngày 2026-05-20 (đã qua). | Hệ thống không cho phép sửa. Nút "Chỉnh sửa" ẩn hoặc vô hiệu hóa. | High |
| TC-UC10-005 | Negative: Sửa giờ gây trùng lịch (BR-01) | 1. CBNV A đăng nhập. 2. Chọn lịch của mình. 3. Sửa giờ sang khung giờ đã có lịch khác. 4. Lưu. | Lịch hiện tại: Phòng A, 14:00–15:00. Lịch tồn tại: Phòng A, 13:30–14:30. Giờ mới: 13:00–14:00 (trùng). | Hệ thống báo lỗi trùng lịch. Không lưu thay đổi. | High |
| TC-UC10-006 | Negative: Sửa sang thời gian quá khứ (BR-03) | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra. 3. Sửa ngày sang ngày đã qua. 4. Lưu. | Ngày mới: 2026-05-01 (đã qua). | Hệ thống báo lỗi không cho chọn thời gian quá khứ. Không lưu. | High |
| TC-UC10-007 | Negative: Sửa giờ kết thúc bằng giờ bắt đầu (BR-04) | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra. 3. Sửa giờ kết thúc bằng giờ bắt đầu. 4. Lưu. | Giờ bắt đầu: 10:00. Giờ kết thúc mới: 10:00. | Hệ thống báo lỗi "Giờ kết thúc phải lớn hơn giờ bắt đầu". Không lưu. | High |
| TC-UC10-008 | Sửa thời gian lịch phòng cần phê duyệt → quay về "Chờ phê duyệt" | 1. Lịch phòng cần phê duyệt đã ở trạng thái "Đã xác nhận". 2. Người đặt sửa giờ họp. 3. Lưu thay đổi. | Lịch: Phòng Hội đồng, trạng thái "Đã xác nhận". Giờ mới: 15:00–17:00 (không trùng). | Lịch quay về trạng thái "Chờ phê duyệt". Email gửi BHC để phê duyệt lại. | High |
| TC-UC10-009 | Edge case [OQ-06]: Sửa nội dung không phải thời gian của lịch phòng cần phê duyệt | 1. Lịch phòng cần phê duyệt trạng thái "Đã xác nhận". 2. Người đặt chỉ sửa tiêu đề (không thay đổi giờ). 3. Lưu. | Lịch: Phòng Hội đồng "Đã xác nhận". Tiêu đề mới: "Họp Board Q2". Giờ giữ nguyên. | **[Cần PO xác nhận — OQ-06]** Kỳ vọng: sửa tiêu đề KHÔNG kích hoạt lại phê duyệt, lịch giữ "Đã xác nhận". | Medium |
| TC-UC10-010 | Edge case: Cảnh báo vượt sức chứa khi sửa số người (BR-09) | 1. CBNV A đăng nhập. 2. Chọn lịch chưa diễn ra. 3. Sửa số người từ 5 lên 15. 4. Phòng có sức chứa 10. 5. Lưu. | Phòng: sức chứa 10 người. Số người cũ: 5. Số người mới: 15. | Hệ thống hiển thị cảnh báo vượt sức chứa NHƯNG không chặn. Lịch lưu thành công. | High |
| TC-UC10-011 | Email bắt buộc sau khi sửa — gửi cho người tham gia (BR-08) | 1. CBNV A sửa lịch thành công. 2. Kiểm tra hộp thư người tham gia. | Người tham gia: CBNV B (nhanvien-b@kdi.vn), CBNV C (nhanvien-c@kdi.vn). | Email thông báo thay đổi gửi đến CBNV B và CBNV C. Nội dung email phản ánh thông tin mới. | High |
| TC-UC10-012 | Edge case [OQ-10]: Email sửa lịch — gửi cho danh sách người tham gia cũ hay mới khi danh sách thay đổi? | 1. CBNV A sửa lịch và thay đổi danh sách người tham gia (thêm CBNV D, bỏ CBNV C). 2. Lưu. | Danh sách cũ: CBNV B, CBNV C. Danh sách mới: CBNV B, CBNV D. | **[Cần PO xác nhận — OQ-10]** Kỳ vọng hợp lý: email gửi cho danh sách mới (B, D). CBNV C có thể cần nhận thông báo bị xóa. BRD không quy định rõ. | Medium |

---

## UC 11: Thiết lập phòng họp

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-UC11-001 | Tạo mới Phòng thường — happy path | 1. Đăng nhập Admin/BHC. 2. Vào màn hình danh sách phòng họp. 3. Nhấn "Thêm mới". 4. Nhập thông tin, chọn Loại phòng = "Phòng thường". 5. Nhấn Lưu. | Mã: PH-001. Tên: Phòng Hội đồng A. Địa điểm: Tầng 3, KDI. Loại: Phòng thường. Sức chứa: 20. | Lưu thành công. Phòng xuất hiện trong danh sách. Hiển thị cho tất cả CBNV. Đặt ngay không cần phê duyệt. | High |
| TC-UC11-002 | Tạo mới Phòng hạn chế — happy path | 1. Đăng nhập Admin/BHC. 2. Vào màn hình danh sách phòng họp. 3. Nhấn "Thêm mới". 4. Nhập thông tin, chọn Loại phòng = "Phòng hạn chế". 5. Nhấn Lưu. | Mã: PH-002. Tên: Phòng Ban Giám đốc. Địa điểm: Tầng 5, KDI. Loại: Phòng hạn chế. Sức chứa: 10. | Lưu thành công. CBNV không có quyền KHÔNG thấy phòng này ở bất kỳ màn hình nào. CBNV có quyền thấy và đặt được. | High |
| TC-UC11-003 | Tạo mới Phòng cần phê duyệt — happy path | 1. Đăng nhập Admin/BHC. 2. Vào màn hình danh sách phòng họp. 3. Nhấn "Thêm mới". 4. Nhập thông tin, chọn Loại phòng = "Phòng cần phê duyệt". 5. Nhấn Lưu. | Mã: PH-003. Tên: Phòng Đào tạo Lớn. Địa điểm: Tầng 1, KDI. Loại: Phòng cần phê duyệt. Sức chứa: 50. | Lưu thành công. Phòng hiển thị với tất cả CBNV. Khi CBNV đặt lịch, lịch ở trạng thái "Chờ phê duyệt", không xác nhận ngay. | High |
| TC-UC11-004 | Sửa thông tin phòng họp — happy path | 1. Đăng nhập Admin/BHC. 2. Vào danh sách phòng họp. 3. Nhấn vào cột Tên phòng họp. 4. Sửa Tên và Sức chứa. 5. Nhấn Lưu. | Tên cũ: Phòng Hội đồng A. Tên mới: Phòng Hội đồng A — 2026. Sức chứa mới: 25. | Thông tin phòng cập nhật. Danh sách phản ánh ngay lập tức. Lịch đã đặt trước đó không bị ảnh hưởng. | High |
| TC-UC11-005 | Tìm kiếm phòng theo tên | 1. Đăng nhập Admin/BHC. 2. Vào danh sách phòng họp. 3. Nhập từ khóa vào ô tìm kiếm Tên phòng. 4. Xem kết quả. | Từ khóa: "Hội đồng". Dữ liệu: Phòng Hội đồng A, Phòng Hội đồng B, Phòng Đào tạo. | Chỉ hiển thị Phòng Hội đồng A và B. Phòng Đào tạo không xuất hiện. | Medium |
| TC-UC11-006 | Tìm kiếm phòng theo địa điểm | 1. Đăng nhập Admin/BHC. 2. Vào danh sách phòng họp. 3. Nhập từ khóa vào ô tìm kiếm Địa điểm. 4. Xem kết quả. | Từ khóa: "Tầng 3". Dữ liệu: 2 phòng ở Tầng 3, 1 phòng ở Tầng 5. | Chỉ hiển thị 2 phòng thuộc Tầng 3. Phòng Tầng 5 không xuất hiện. | Medium |
| TC-UC11-007 | Xóa phòng không có lịch tương lai — happy path | 1. Đăng nhập Admin/BHC. 2. Chọn phòng không có lịch tương lai. 3. Thực hiện Xóa. 4. Xác nhận. | Mã phòng: PH-099 (chỉ có lịch trong quá khứ). | Xóa thành công ngay lập tức. Phòng biến mất khỏi danh sách. Không còn trong form đặt lịch. | High |
| TC-UC11-008 | Dừng phòng không có lịch tương lai — happy path | 1. Đăng nhập Admin/BHC. 2. Chọn phòng không có lịch tương lai. 3. Thực hiện Dừng. | Mã phòng: PH-098. | Trạng thái phòng cập nhật thành "Dừng hoạt động". Phòng không còn khả dụng để đặt lịch mới. | High |
| TC-UC11-009 | Xóa phòng có lịch họp tương lai — phải hủy hết trước | 1. Đăng nhập Admin/BHC. 2. Chọn phòng có ít nhất 2 lịch tương lai. 3. Thực hiện Xóa. | Mã phòng: PH-010. Lịch tương lai: Lịch A ngày 2026-06-01 10:00, Lịch B ngày 2026-06-15 14:00. | Hệ thống KHÔNG xóa ngay. Hiển thị danh sách các lịch tương lai. Yêu cầu hủy hết lịch trước khi xóa. | High |
| TC-UC11-010 | Xóa phòng sau khi đã hủy hết lịch tương lai | 1. Từ kết quả TC-UC11-009, hủy từng lịch tương lai. 2. Quay lại danh sách phòng. 3. Thực hiện lại Xóa phòng PH-010. 4. Xác nhận. | Mã phòng: PH-010. Sau khi hủy: 0 lịch tương lai còn lại. | Hệ thống xóa phòng thành công. Phòng biến mất khỏi danh sách. | High |
| TC-UC11-011 | Dừng phòng có lịch tương lai — phải hủy hết trước | 1. Đăng nhập Admin/BHC. 2. Chọn phòng có lịch tương lai. 3. Thực hiện Dừng. | Mã phòng: PH-020 có lịch ngày 2026-07-01 14:00. | Hệ thống KHÔNG dừng ngay. Hiển thị danh sách lịch tương lai. Yêu cầu hủy hết trước khi Dừng. | High |
| TC-UC11-012 | Negative: Tạo phòng với mã đã tồn tại | 1. Đăng nhập Admin/BHC. 2. Vào Thêm mới phòng họp. 3. Nhập Mã phòng trùng. 4. Điền các trường còn lại. 5. Nhấn Lưu. | Mã phòng nhập: PH-001 (đã tồn tại). Tên: Phòng Kiểm tra Trùng. | Hệ thống báo lỗi "Mã phòng PH-001 đã tồn tại". KHÔNG lưu. Dữ liệu form giữ nguyên. | High |
| TC-UC11-013 | Negative: Tạo phòng thiếu trường bắt buộc | 1. Đăng nhập Admin/BHC. 2. Vào Thêm mới. 3. Để trống Mã phòng. 4. Điền các trường còn lại. 5. Nhấn Lưu. | Mã phòng: (để trống). Tên: Phòng Kiểm tra. | Hệ thống báo lỗi validation tại trường Mã phòng. Không lưu. Dữ liệu form giữ nguyên. | High |
| TC-UC11-014 | Negative: CBNV thường truy cập màn hình quản lý phòng họp | 1. Đăng nhập CBNV thông thường. 2. Cố truy cập URL màn hình danh sách phòng họp. | Tài khoản: nhanvien01. | Hệ thống chặn truy cập. Hiển thị "Không có quyền truy cập" hoặc chuyển hướng về trang chủ. Menu quản lý phòng không hiển thị trong navigation. | High |
| TC-UC11-015 | Edge case: Verify Phòng hạn chế ẩn hoàn toàn với CBNV không có quyền | 1. Admin/BHC tạo Phòng hạn chế (PH-002). 2. Đăng nhập CBNV không có quyền phòng hạn chế. 3. Vào danh sách phòng. 4. Tìm kiếm theo tên "Ban Giám đốc". 5. Vào form đặt lịch họp, tìm kiếm phòng. | Tài khoản CBNV: nhanvien02 (không có quyền phòng hạn chế). Phòng: PH-002. | Phòng hạn chế KHÔNG xuất hiện ở BẤT KỲ điểm nào: danh sách phòng, kết quả tìm kiếm, dropdown chọn phòng. Khác hoàn toàn với SRS cũ (SRS cũ: thấy nhưng không đặt; BRD mới: ẩn hoàn toàn). | High |
| TC-UC11-016 | Edge case: Verify Phòng hạn chế hiển thị đúng với CBNV CÓ quyền | 1. Admin/BHC tạo Phòng hạn chế (PH-002). 2. Cấp quyền cho CBNV đặc biệt. 3. Đăng nhập CBNV có quyền. 4. Vào danh sách phòng và form đặt lịch. | Tài khoản: truongphong01 (có quyền phòng hạn chế). | Phòng hạn chế HIỂN THỊ trong danh sách và form. CBNV đặt được bình thường. Lịch xác nhận ngay (không cần phê duyệt). | High |
| TC-UC11-017 | Edge case [OQ-08]: Xóa phòng có lịch ở trạng thái "Chờ phê duyệt" | 1. Admin/BHC tạo Phòng cần phê duyệt (PH-003). 2. CBNV đặt lịch → trạng thái "Chờ phê duyệt", chưa phê duyệt. 3. Admin/BHC thực hiện Xóa phòng PH-003. | Mã phòng: PH-003. Lịch: "Chờ phê duyệt", ngày 2026-06-20. | **[OQ-08 — Chờ PO xác nhận]** Kịch bản A: hệ thống coi lịch "Chờ phê duyệt" là lịch tương lai → chặn xóa. Kịch bản B: bỏ qua lịch chưa phê duyệt → cho xóa ngay. Cần PO xác nhận trước khi execute. | High |
| TC-UC11-018 | Edge case: Verify luồng phê duyệt hoạt động đúng sau khi tạo Phòng cần phê duyệt | 1. Admin/BHC tạo Phòng cần phê duyệt (PH-003). 2. CBNV đặt lịch tại PH-003. 3. Kiểm tra trạng thái lịch. 4. BHC phê duyệt. 5. Kiểm tra lại trạng thái lịch và email. | CBNV đặt lịch: nhanvien03. Phòng: PH-003. Lịch: 2026-06-10 09:00–11:00. | Sau bước 3: lịch "Chờ phê duyệt". Sau bước 4: lịch "Đã xác nhận", CBNV nhận email thông báo. | High |
| TC-UC11-019 | Nhấn vào cột Tên phòng họp để mở màn hình chi tiết | 1. Đăng nhập Admin/BHC. 2. Vào danh sách phòng. 3. Nhấn vào ô Tên phòng của 1 phòng bất kỳ. | Phòng bất kỳ có sẵn. | Màn hình chi tiết/hiệu chỉnh mở ra. Hiển thị đầy đủ thông tin. Cột Tên phòng phải là vùng nhấn được rõ ràng. | Medium |
| TC-UC11-020 | Tìm kiếm không có kết quả — edge case | 1. Đăng nhập Admin/BHC. 2. Vào danh sách phòng họp. 3. Nhập từ khóa không khớp bất kỳ phòng nào. | Từ khóa: "XXXXXXXX99999". | Danh sách trống. Hiển thị thông báo "Không tìm thấy phòng phù hợp". Không có lỗi hệ thống. | Medium |

---

## UC mới: Phê duyệt lịch đặt phòng

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|----|---------------|------------|-----------|-----------------|----------|
| TC-APPROVE-001 | Happy path: BHC phê duyệt lịch chờ phê duyệt | 1. BHC đăng nhập. 2. Truy cập màn hình quản lý phê duyệt. 3. Xem danh sách lịch "Chờ phê duyệt". 4. Chọn 1 lịch. 5. Xem đầy đủ thông tin. 6. Nhấn "Phê duyệt". | Tài khoản: BHC. Lịch: Phòng Hội đồng, ngày 2026-06-02, 14:00–16:00. | Lịch chuyển trạng thái "Đã xác nhận". Email thông báo phê duyệt gửi cho người đặt và người tham gia. | High |
| TC-APPROVE-002 | Happy path: BHC từ chối lịch với lý do | 1. BHC đăng nhập. 2. Vào màn hình quản lý phê duyệt. 3. Chọn lịch "Chờ phê duyệt". 4. Nhấn "Từ chối". 5. Nhập lý do. 6. Xác nhận. | Tài khoản: BHC. Lý do từ chối: "Phòng dành cho sự kiện công ty". | Lịch chuyển trạng thái "Bị từ chối". Email gửi người đặt kèm lý do. | High |
| TC-APPROVE-003 | Negative: Từ chối mà không nhập lý do | 1. BHC đăng nhập. 2. Chọn lịch "Chờ phê duyệt". 3. Nhấn "Từ chối". 4. Để trống lý do. 5. Xác nhận. | Lý do từ chối: (để trống). | Hệ thống báo lỗi "Lý do từ chối là bắt buộc". Không thực hiện từ chối. | High |
| TC-APPROVE-004 | Negative: CBNV thường không thể phê duyệt/từ chối (BR-11) | 1. CBNV thường đăng nhập. 2. Thử truy cập màn hình quản lý phê duyệt. | Tài khoản: CBNV thường (không phải BHC). | Hệ thống không cho truy cập màn hình phê duyệt. Hiển thị thông báo từ chối quyền hoặc chuyển hướng. | High |
| TC-APPROVE-005 | Negative: Lịch đã bị hủy trước khi BHC xử lý | 1. CBNV A đặt lịch → "Chờ phê duyệt". 2. CBNV A hủy lịch trong khi BHC chưa xử lý. 3. BHC mở lịch đó và thử phê duyệt. | Lịch: đã chuyển từ "Chờ phê duyệt" sang "Đã hủy". | Hệ thống hiển thị "Lịch này đã bị hủy". Nút "Phê duyệt" và "Từ chối" bị vô hiệu hóa. | High |
| TC-APPROVE-006 | Negative: Race condition — Lịch đã được xử lý bởi BHC khác | 1. BHC1 và BHC2 cùng mở cùng 1 lịch "Chờ phê duyệt". 2. BHC1 phê duyệt thành công. 3. BHC2 cũng thử nhấn "Phê duyệt". | Tài khoản BHC1 và BHC2. Lịch: cùng 1 lịch "Chờ phê duyệt". | BHC2 bị chặn. Hệ thống hiển thị "Lịch này đã được xử lý". Không thực hiện lại. | High |
| TC-APPROVE-007 | Edge case: Race condition — BHC1 đang từ chối trong khi BHC2 phê duyệt | 1. BHC1 nhấn "Từ chối", đang nhập lý do. 2. BHC2 nhấn "Phê duyệt" thành công trước. 3. BHC1 hoàn tất và submit. | BHC2 hoàn tất trước. | BHC1 nhận thông báo "Lịch này đã được xử lý". Hành động từ chối của BHC1 không có hiệu lực. Trạng thái theo hành động BHC2. | High |
| TC-APPROVE-008 | BR-02: Lịch "Chờ phê duyệt" giữ chỗ — đặt trùng cùng phòng cùng giờ bị từ chối | 1. Phòng Hội đồng có lịch "Chờ phê duyệt" 14:00–16:00. 2. CBNV B vào form đặt phòng, chọn cùng phòng, cùng giờ. 3. Nhấn "Đặt phòng". | Phòng Hội đồng. Lịch chờ phê duyệt: 14:00–16:00. Lịch mới: 14:30–15:30 (trùng). | Hệ thống báo lỗi trùng lịch. Không cho đặt thêm vào khung giờ đang được giữ chỗ. | High |
| TC-APPROVE-009 | Sau khi từ chối — khung giờ mở lại cho người khác đặt | 1. BHC từ chối lịch Phòng Hội đồng 14:00–16:00. 2. CBNV B thử đặt cùng phòng cùng giờ sau khi từ chối. | Lịch: "Bị từ chối". Phòng Hội đồng 14:00–16:00. | CBNV B đặt được thành công. Khung giờ đã được giải phóng. | High |
| TC-APPROVE-010 | Email phê duyệt — gửi đúng đối tượng | 1. BHC phê duyệt lịch thành công. 2. Kiểm tra hộp thư người đặt và người tham gia. | Người đặt: CBNV A. Người tham gia: CBNV B, CBNV C. | Email "Lịch họp đã được xác nhận" gửi cho CBNV A (người đặt) VÀ CBNV B, CBNV C (người tham gia). | High |
| TC-APPROVE-011 | Email từ chối — gửi đúng đối tượng và có lý do | 1. BHC từ chối lịch với lý do. 2. Kiểm tra hộp thư người đặt. | Người đặt: CBNV A. Lý do từ chối: "Phòng dành cho sự kiện công ty". | Email "Lịch họp bị từ chối" kèm lý do gửi cho CBNV A. Người tham gia không nhận email từ chối. | High |
| TC-APPROVE-012 | Danh sách phê duyệt chỉ hiển thị lịch "Chờ phê duyệt" | 1. BHC đăng nhập. 2. Truy cập màn hình quản lý phê duyệt. 3. Kiểm tra danh sách hiển thị. | Hệ thống có lịch ở trạng thái: "Đã xác nhận", "Đã hủy", "Bị từ chối", "Chờ phê duyệt". | Chỉ các lịch trạng thái "Chờ phê duyệt" xuất hiện trong danh sách. | Medium |
| TC-APPROVE-013 | Xem đầy đủ thông tin lịch trước khi quyết định | 1. BHC đăng nhập. 2. Vào danh sách phê duyệt. 3. Chọn 1 lịch. 4. Xem màn hình chi tiết. | Lịch: Phòng Hội đồng, ngày 2026-06-02, 14:00–16:00, người đặt: CBNV A, người tham gia: CBNV B. | Màn hình hiển thị đầy đủ: tiêu đề, phòng, ngày giờ, người đặt, người tham gia, số người, yêu cầu hỗ trợ (nếu có). | Medium |

---

---

# PHẦN 2: VĂN PHÒNG PHẨM (UC 1–6)

> **Nguồn:** SRS Fanxipan v1.0, phần 2.1. Không có thay đổi theo BRD.

## UC 1: Tạo mới đề nghị mua văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC1-001 | Happy path: Tạo mới tờ trình hợp lệ và nhấn Lưu | 1. Đăng nhập hệ thống. 2. Nhấn "Tạo mới" đề nghị mua VPP. 3. Nhập Nội dung. 4. Chọn Chi phí sử dụng. 5. Chọn Tên hàng, nhập Số lượng. 6. Nhấn Lưu. | Nội dung: "Mua văn phòng phẩm tháng 5"; Chi phí: "Trong ngân sách"; Tên hàng: bút bi; Số lượng: 10 | Tờ trình lưu trạng thái "Đang soạn". Hiển thị tab Quy trình và nút Trình phê duyệt. Log ghi nhận "Tạo mới tờ trình". | High |
| TC-UC1-002 | Happy path: Trình phê duyệt thành công | 1. Mở tờ trình trạng thái Đang soạn. 2. Cấu hình đủ người phê duyệt ở tab Quy trình. 3. Nhấn Trình phê duyệt. | Tờ trình hợp lệ, quy trình đủ người phê duyệt | Tờ trình chuyển trạng thái "Chờ phê duyệt". Gửi email ET1 cho người phê duyệt bước đầu tiên. | High |
| TC-UC1-003 | Happy path: Thông tin Đơn vị và Người tạo tự động điền | 1. Đăng nhập. 2. Vào màn hình Tạo mới đề nghị mua VPP. 3. Quan sát các trường Đơn vị, Người tạo. | Người dùng đã đăng nhập đúng tài khoản | Trường Đơn vị và Người tạo tự động điền theo thông tin người dùng. Không cho phép chỉnh sửa. | High |
| TC-UC1-004 | Happy path: Hệ thống tự động tính toán giá trị | 1. Chọn Tên hàng từ danh mục. 2. Nhập Số lượng. 3. Quan sát các trường Đơn giá, Giá mua dự kiến, VAT, Tổng số tiền. | Tên hàng: bút bi (đơn giá 5.000 VNĐ, VAT 10%); Số lượng: 20 | Đơn giá = 5.000. Giá mua dự kiến chưa VAT = 100.000. VAT = 10%. Giá mua dự kiến đã VAT = 110.000. Tổng = 110.000. | High |
| TC-UC1-005 | Happy path: Xuất PDF tờ trình sau khi trình phê duyệt | 1. Trình phê duyệt thành công. 2. Nhấn xuất PDF. | Tờ trình trạng thái Chờ phê duyệt trở đi | File PDF xuất đầy đủ thông tin và quy trình phê duyệt (bước thực hiện, thời gian, người thực hiện). | Medium |
| TC-UC1-006 | Negative: Lưu khi thiếu trường bắt buộc Nội dung | 1. Vào Tạo mới. 2. Bỏ trống Nội dung. 3. Nhập các trường còn lại hợp lệ. 4. Nhấn Lưu. | Nội dung: để trống; Tên hàng: bút bi; Số lượng: 5 | Hệ thống không lưu. Hiển thị thông báo lỗi tại trường Nội dung. | High |
| TC-UC1-007 | Negative: Lưu khi bảng danh sách đề xuất mua trống | 1. Vào Tạo mới. 2. Nhập Nội dung, chọn Chi phí sử dụng. 3. Không thêm dòng nào vào bảng. 4. Nhấn Lưu. | Bảng đề xuất: rỗng | Hệ thống không lưu. Hiển thị thông báo lỗi yêu cầu nhập ít nhất một mặt hàng. | High |
| TC-UC1-008 | Negative: Trình phê duyệt khi quy trình thiếu người phê duyệt | 1. Tạo và lưu tờ trình. 2. Không cấu hình đủ người phê duyệt trong tab Quy trình. 3. Nhấn Trình phê duyệt. | Quy trình thiếu người phê duyệt ở ít nhất một bước | Hệ thống hiển thị cảnh báo. Không gửi yêu cầu phê duyệt. Tờ trình vẫn trạng thái Đang soạn. | High |
| TC-UC1-009 | Negative: Nhấn Cancel khi đang tạo mới | 1. Vào Tạo mới. 2. Nhập một số thông tin. 3. Nhấn Cancel. | Nội dung: "test" (chưa lưu) | Màn hình Tạo mới đóng. Dữ liệu không được lưu. Không có tờ trình trong danh sách. | Medium |
| TC-UC1-010 | Negative: Cố chỉnh sửa tờ trình ở trạng thái Chờ phê duyệt | 1. Trình phê duyệt tờ trình. 2. Cố chỉnh sửa thông tin. | Tờ trình trạng thái Chờ phê duyệt | Hệ thống không cho phép chỉnh sửa. Nút Lưu ẩn hoặc bị vô hiệu hóa. | High |
| TC-UC1-011 | Edge case: Thêm nhiều mặt hàng — kiểm tra Tổng số tiền tự động tính đúng | 1. Thêm 5 dòng mặt hàng khác nhau. 2. Nhập số lượng từng dòng. 3. Quan sát trường Tổng số tiền. | 5 mặt hàng với đơn giá và VAT khác nhau | Tổng số tiền = tổng cộng Giá mua dự kiến đã VAT của tất cả dòng. Khớp phép tính thủ công. | High |
| TC-UC1-012 | Edge case: Chọn Chi phí sử dụng = "Ngoài ngân sách" | 1. Vào Tạo mới. 2. Chọn Chi phí sử dụng "Ngoài ngân sách". 3. Lưu và trình phê duyệt. | Chi phí sử dụng: "Ngoài ngân sách" | Lưu thành công. Khi trình phê duyệt, quy trình Ngoài định mức (có thêm bước Trưởng Ban/Khối/TGĐ) được kích hoạt. | Medium |

---

## UC 2: Phê duyệt/Trả lại tờ trình đề nghị mua văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC2-001 | Happy path: Phụ trách bộ phận phê duyệt — còn cấp tiếp theo | 1. Đăng nhập Phụ trách bộ phận. 2. Vào tờ trình đang chờ xử lý bước mình phụ trách. 3. Nhấn Phê duyệt. | Tờ trình trạng thái Chờ phê duyệt, đang ở bước 1/2 | Tờ trình chuyển tiếp sang bước phê duyệt tiếp theo. Gửi email ET1 cho người phê duyệt bước tiếp. | High |
| TC-UC2-002 | Happy path: Cấp phê duyệt cuối cùng phê duyệt — hoàn thành | 1. Đăng nhập người phê duyệt bước cuối. 2. Vào tờ trình đang chờ xử lý. 3. Nhấn Phê duyệt. | Tờ trình đang ở bước phê duyệt cuối cùng | Tờ trình chuyển trạng thái Hoàn thành. Gửi email ET2 cho người trình. | High |
| TC-UC2-003 | Happy path: Trả lại kèm ý kiến | 1. Đăng nhập người phê duyệt. 2. Nhấn Trả lại. 3. Nhập ý kiến. 4. Xác nhận. | Ý kiến: "Số lượng vượt mức quy định, cần điều chỉnh" | Tờ trình chuyển trạng thái Trả lại. Gửi email ET3 cho người trình. | High |
| TC-UC2-004 | Happy path: Log ghi nhận đầy đủ sau phê duyệt | 1. Phê duyệt tờ trình. 2. Kiểm tra log. | Tờ trình vừa được phê duyệt | Log ghi nhận: hành động (Phê duyệt), ý kiến (nếu có), thời gian, người xử lý. | Medium |
| TC-UC2-005 | Negative: Trả lại mà không nhập ý kiến | 1. Đăng nhập người phê duyệt. 2. Nhấn Trả lại. 3. Để trống Ý kiến. 4. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Hiển thị lỗi yêu cầu nhập ý kiến. | High |
| TC-UC2-006 | Negative: Người không phải người phê duyệt cố phê duyệt | 1. Đăng nhập nhân viên thường. 2. Truy cập tờ trình đang chờ phê duyệt. | Người dùng không phải người phê duyệt bước hiện tại | Không hiển thị nút Phê duyệt/Trả lại. Chỉ xem được thông tin. | High |
| TC-UC2-007 | Negative: Người không có quyền truy cập cố mở tờ trình | 1. Đăng nhập người dùng không liên quan. 2. Truy cập URL chi tiết tờ trình. | Người dùng không phải người tạo, không được chia sẻ, không phải người phê duyệt | Hệ thống từ chối truy cập. Không hiển thị nội dung tờ trình. | High |
| TC-UC2-008 | Negative: Người phê duyệt bước 2 cố phê duyệt khi tờ trình đang ở bước 1 | 1. Đăng nhập người phê duyệt bước 2. 2. Truy cập tờ trình đang ở bước 1. | Tờ trình đang ở bước 1 | Không hiển thị nút Phê duyệt/Trả lại. Chỉ xem được. | High |
| TC-UC2-009 | Edge case: Tờ trình ngoài ngân sách — kiểm tra đúng luồng phê duyệt | 1. Tạo tờ trình Chi phí "Ngoài ngân sách". 2. Trình phê duyệt. 3. Kiểm tra bước quy trình. | Chi phí: "Ngoài ngân sách" | Quy trình bao gồm bước Trưởng Ban/Khối/TGĐ. Email ET1 gửi đúng người phê duyệt đầu tiên. | High |
| TC-UC2-010 | Edge case: Người được chia sẻ tờ trình chỉ xem, không thao tác | 1. Người tạo chia sẻ tờ trình. 2. Người được chia sẻ đăng nhập và mở tờ trình. | Người dùng được chia sẻ quyền xem | Xem được đầy đủ thông tin. Không hiển thị nút Phê duyệt/Trả lại. | Medium |
| TC-UC2-011 | Edge case: Email ET2 gửi đúng sau khi hoàn thành | 1. Người phê duyệt cuối phê duyệt. 2. Kiểm tra hộp thư người trình. | Tờ trình hoàn thành phê duyệt | Email ET2 gửi đúng địa chỉ người tạo tờ trình. Nội dung đúng mẫu ET2. | Medium |
| TC-UC2-012 | Edge case: Email ET3 gửi đúng sau khi bị trả lại | 1. Người phê duyệt trả lại. 2. Kiểm tra hộp thư người trình. | Tờ trình bị trả lại | Email ET3 gửi đúng địa chỉ người trình, kèm ý kiến người phê duyệt. | Medium |

---

## UC 3: Màn hình quản lý danh sách tờ trình đề nghị mua văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC3-001 | Happy path: Ban hành chính xem danh sách tờ trình đã phê duyệt | 1. Đăng nhập Ban hành chính. 2. Truy cập màn hình Đề nghị mua VPP từ leftmenu. | Có nhiều tờ trình đã phê duyệt | Danh sách hiển thị đầy đủ các cột: STT, Số ký hiệu, Tiêu đề, Phòng ban, Trạng thái, Người trình, Ngày trình, Ngày phê duyệt. | High |
| TC-UC3-002 | Happy path: Tìm kiếm theo từ khóa (số ký hiệu) | 1. Nhập từ khóa là số ký hiệu tờ trình. 2. Quan sát kết quả. | Từ khóa: mã ký hiệu đã biết | Danh sách lọc đúng tờ trình có số ký hiệu khớp. | High |
| TC-UC3-003 | Happy path: Lọc theo Tháng | 1. Chọn tháng cụ thể ở bộ tìm kiếm. 2. Quan sát kết quả. | Tháng: 04/2026 | Chỉ hiển thị tờ trình phê duyệt trong tháng 04/2026. | High |
| TC-UC3-004 | Happy path: Lọc theo Phòng ban | 1. Chọn phòng ban trong bộ lọc. 2. Quan sát kết quả. | Phòng ban: Phòng Kế toán và Phòng IT | Chỉ hiển thị tờ trình thuộc các phòng ban đã chọn. | High |
| TC-UC3-005 | Happy path: Chọn 1 tờ trình Chưa tạo đơn — hiện nút Tạo đơn đặt hàng | 1. Tích chọn 1 tờ trình trạng thái "Chưa tạo đơn đặt hàng". 2. Quan sát giao diện. | Tờ trình: trạng thái Chưa tạo đơn | Nút Tạo đơn đặt hàng xuất hiện. | High |
| TC-UC3-006 | Happy path: Nhấn vào dòng để xem chi tiết | 1. Nhấn vào một cột bất kỳ trên dòng tờ trình. | Bất kỳ tờ trình nào | Điều hướng đến màn hình chi tiết tờ trình đó. | High |
| TC-UC3-007 | Happy path: Tích chọn toàn bộ qua checkbox header | 1. Nhấn checkbox ở header. 2. Quan sát trạng thái các dòng. | Danh sách có nhiều tờ trình | Tất cả dòng được tích chọn. Nút Tạo đơn đặt hàng hiển thị (nếu tất cả chưa tạo đơn). | Medium |
| TC-UC3-008 | Negative: Người dùng thường không thấy màn hình này | 1. Đăng nhập nhân viên thường. 2. Kiểm tra leftmenu. | Tài khoản không thuộc Ban hành chính | Màn hình không xuất hiện tại leftmenu. Không thể truy cập. | High |
| TC-UC3-009 | Negative: Chọn tờ trình Đã tạo đơn — không hiện nút Tạo đơn đặt hàng | 1. Tích chọn tờ trình trạng thái "Đã tạo đơn đặt hàng". 2. Quan sát giao diện. | Tờ trình: trạng thái Đã tạo đơn | Nút Tạo đơn đặt hàng không hiển thị. | High |
| TC-UC3-010 | Negative: Tìm kiếm không tồn tại | 1. Nhập từ khóa không khớp bất kỳ tờ trình nào. | Từ khóa: "XXXX999" | Danh sách rỗng hoặc thông báo không tìm thấy. Không lỗi hệ thống. | Medium |
| TC-UC3-011 | Edge case: Lọc kết hợp nhiều điều kiện cùng lúc | 1. Chọn Tháng + Phòng ban + Người trình + Trạng thái cùng lúc. 2. Nhấn tìm kiếm. | Tháng: 03/2026; Phòng ban: Kế toán; Trạng thái: Chưa tạo đơn | Danh sách đúng với tờ trình thỏa mãn đồng thời tất cả điều kiện lọc. | Medium |
| TC-UC3-012 | Edge case: Lọc theo khoảng Ngày trình (từ ngày–đến ngày) | 1. Nhập ngày bắt đầu và kết thúc ở trường Ngày trình. 2. Quan sát kết quả. | Ngày trình: 01/03/2026 – 31/03/2026 | Chỉ hiển thị tờ trình được trình trong khoảng đó. | Medium |

---

## UC 4: Tạo đơn đặt hàng văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC4-001 | Happy path: Tạo đơn từ 1 tờ trình đã phê duyệt | 1. Đăng nhập Ban hành chính. 2. Chọn 1 tờ trình "Chưa tạo đơn". 3. Nhấn Tạo đơn đặt hàng. 4. Kiểm tra data mapping. 5. Nhấn Lưu. | Tờ trình đã phê duyệt, chưa tạo đơn | Dữ liệu VPP từ tờ trình tự động mapping sang đơn. Đơn lưu trạng thái Đang soạn. Tab Quy trình và nút Trình phê duyệt hiển thị. Log ghi nhận. | High |
| TC-UC4-002 | Happy path: Tạo đơn từ nhiều tờ trình cùng lúc | 1. Chọn 3 tờ trình "Chưa tạo đơn". 2. Nhấn Tạo đơn đặt hàng. 3. Kiểm tra danh mục tổng hợp. | 3 tờ trình từ 3 phòng ban khác nhau | Dữ liệu VPP từ cả 3 tờ trình được tổng hợp vào 1 đơn. Mapping chính xác, không mất dữ liệu. | High |
| TC-UC4-003 | Happy path: Trình phê duyệt đơn thành công | 1. Mở đơn Đang soạn. 2. Cấu hình đủ người phê duyệt. 3. Nhấn Trình phê duyệt. | Đơn hợp lệ, quy trình đủ người | Đơn chuyển trạng thái Chờ phê duyệt. Gửi email cho người phê duyệt đầu tiên. | High |
| TC-UC4-004 | Happy path: Trường Đơn vị và Người tạo tự động điền | 1. Tạo đơn mới. 2. Quan sát trường Đơn vị, Người tạo. | Người dùng là nhân viên Ban hành chính | Tự động điền theo thông tin đăng nhập. Không cho chỉnh sửa. | High |
| TC-UC4-005 | Happy path: Xuất PDF đơn đặt hàng | 1. Trình phê duyệt thành công. 2. Nhấn xuất PDF. | Đơn trạng thái Chờ phê duyệt trở đi | File PDF xuất đầy đủ thông tin đơn và quy trình phê duyệt. | Medium |
| TC-UC4-006 | Negative: Lưu khi thiếu trường bắt buộc | 1. Mở tạo đơn. 2. Xóa/bỏ trống trường bắt buộc. 3. Nhấn Lưu. | Trường bắt buộc để trống | Hệ thống không lưu. Hiển thị lỗi tại trường thiếu. | High |
| TC-UC4-007 | Negative: Trình phê duyệt khi thiếu người phê duyệt | 1. Tạo và lưu đơn. 2. Không cấu hình đủ người phê duyệt. 3. Nhấn Trình phê duyệt. | Quy trình thiếu người phê duyệt | Hệ thống cảnh báo. Không gửi yêu cầu. Đơn vẫn Đang soạn. | High |
| TC-UC4-008 | Negative: Nhấn Cancel khi đang tạo đơn | 1. Đang trên màn hình tạo đơn. 2. Chỉnh sửa một số thông tin. 3. Nhấn Cancel. | Dữ liệu chưa lưu | Màn hình đóng. Dữ liệu không lưu. | Medium |
| TC-UC4-009 | Negative: Người dùng thường không thể tạo đơn | 1. Đăng nhập nhân viên thường. 2. Cố truy cập chức năng tạo đơn. | Tài khoản không thuộc Ban hành chính | Không thể truy cập màn hình. Không hiển thị nút Tạo đơn. | High |
| TC-UC4-010 | Edge case: Chỉnh sửa đơn trạng thái Đang soạn | 1. Tạo và lưu đơn (Đang soạn). 2. Mở lại. 3. Chỉnh sửa. 4. Lưu. | Đơn trạng thái Đang soạn | Cho phép chỉnh sửa và lưu thành công. Thông tin mới được cập nhật. | High |
| TC-UC4-011 | Edge case: Cố chỉnh sửa đơn trạng thái Chờ phê duyệt | 1. Trình phê duyệt đơn. 2. Cố chỉnh sửa. | Đơn trạng thái Chờ phê duyệt | Không cho phép chỉnh sửa. Các trường hiển thị read-only. | High |
| TC-UC4-012 | Edge case: Mapping từ nhiều tờ trình có cùng mã SP | 1. Chọn 2 tờ trình có cùng mã SP. 2. Tạo đơn. 3. Kiểm tra bảng danh mục. | 2 tờ trình đều có "bút bi" với số lượng khác nhau | Dữ liệu từ cả 2 tờ trình hiển thị đúng (gộp hoặc tách riêng tùy cấu hình SRS). Không mất dữ liệu. | High |

---

## UC 5: Phê duyệt/Trả lại đơn đặt hàng văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC5-001 | Happy path: Trưởng Ban hành chính phê duyệt — chuyển tiếp lãnh đạo | 1. Đăng nhập Trưởng Ban hành chính. 2. Vào đơn chờ xử lý bước mình phụ trách. 3. Nhấn Phê duyệt. | Đơn đang ở bước Trưởng Ban hành chính | Đơn chuyển tiếp sang bước Lãnh đạo có thẩm quyền. Gửi email ET1 cho Lãnh đạo. | High |
| TC-UC5-002 | Happy path: Lãnh đạo phê duyệt — hoàn thành | 1. Đăng nhập Lãnh đạo. 2. Vào đơn chờ xử lý. 3. Nhấn Phê duyệt. | Đơn đang ở bước Lãnh đạo | Đơn chuyển trạng thái "Đã phê duyệt". Gửi email ET2 cho người trình. | High |
| TC-UC5-003 | Happy path: Trưởng Ban hành chính trả lại | 1. Đăng nhập Trưởng BHC. 2. Nhấn Trả lại. 3. Nhập ý kiến. 4. Xác nhận. | Ý kiến: "Đơn giá không khớp báo giá, cần cập nhật" | Đơn chuyển trạng thái Trả lại. Gửi email ET3 cho người trình. | High |
| TC-UC5-004 | Happy path: Lãnh đạo trả lại | 1. Đăng nhập Lãnh đạo. 2. Nhấn Trả lại. 3. Nhập ý kiến. | Ý kiến: "Tổng giá trị vượt ngân sách duyệt" | Đơn chuyển trạng thái Trả lại. Gửi email ET3 cho người trình. | High |
| TC-UC5-005 | Happy path: Log ghi nhận đầy đủ | 1. Phê duyệt hoặc trả lại đơn. 2. Kiểm tra log. | Cả hai hành động Phê duyệt và Trả lại | Log ghi nhận: Hành động, Ý kiến, Thời gian xử lý, Người xử lý. | Medium |
| TC-UC5-006 | Negative: Trả lại không nhập ý kiến | 1. Đăng nhập người phê duyệt. 2. Nhấn Trả lại. 3. Để trống Ý kiến. 4. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Hiển thị lỗi yêu cầu nhập ý kiến. | High |
| TC-UC5-007 | Negative: Người không phải người phê duyệt bước hiện tại | 1. Đăng nhập Lãnh đạo khi đơn đang ở bước Trưởng BHC. 2. Mở đơn. | Đơn đang ở bước Trưởng BHC | Không hiển thị nút Phê duyệt/Trả lại. Chỉ xem. | High |
| TC-UC5-008 | Negative: Người không có quyền cố truy cập | 1. Đăng nhập tài khoản không liên quan. 2. Cố truy cập URL đơn. | Tài khoản không liên quan đến đơn | Hệ thống từ chối. Không hiển thị nội dung đơn. | High |
| TC-UC5-009 | Edge case: Người được chia sẻ chỉ xem | 1. Người tạo chia sẻ đơn cho đồng nghiệp. 2. Đồng nghiệp mở đơn. | Người dùng được chia sẻ quyền xem | Xem được toàn bộ. Không có nút Phê duyệt/Trả lại. | Medium |
| TC-UC5-010 | Edge case: Email ET1 gửi đúng khi chuyển bước | 1. Trưởng BHC phê duyệt xong. 2. Kiểm tra hộp thư Lãnh đạo. | Bước Trưởng BHC vừa phê duyệt | Email ET1 gửi đúng địa chỉ Lãnh đạo. Nội dung đúng mẫu, có tên đơn. | Medium |
| TC-UC5-011 | Edge case: Trạng thái sau khi Lãnh đạo phê duyệt là "Đã phê duyệt" | 1. Lãnh đạo phê duyệt đơn. 2. Kiểm tra trạng thái trên danh sách. | Đơn vừa được phê duyệt bước cuối | Trạng thái hiển thị: "Đã phê duyệt". | High |
| TC-UC5-012 | Edge case: Đơn bị trả lại — người tạo có thể chỉnh sửa và trình lại | 1. Đơn ở trạng thái Trả lại. 2. Người tạo chỉnh sửa. 3. Trình phê duyệt lại. | Đơn trạng thái Trả lại | Hệ thống cho phép chỉnh sửa và trình lại. Đơn chuyển Chờ phê duyệt, quy trình khởi động lại từ đầu. | High |

---

## UC 6: Quản lý danh mục văn phòng phẩm

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC6-001 | Happy path: Tạo mới danh mục VPP với trường bắt buộc | 1. Đăng nhập Ban hành chính. 2. Vào Quản lý danh mục VPP. 3. Nhấn Tạo mới. 4. Nhập Mã SP và Tên sản phẩm. 5. Nhấn Lưu. | Mã SP: "BP001"; Tên sản phẩm: "Bút bi xanh" | Danh mục lưu thành công. Danh sách cập nhật bản ghi vừa tạo. | High |
| TC-UC6-002 | Happy path: Tạo mới với đầy đủ tất cả trường | 1. Nhấn Tạo mới. 2. Nhập đủ: Mã SP, Tên sản phẩm, Xuất xứ, Quy cách, Đơn vị, Giá bán chưa VAT, VAT, Hình ảnh. 3. Nhấn Lưu. | Mã SP: "BP002"; Tên: "Bút bi đỏ"; Giá: 3.000; VAT: 10% | Tất cả thông tin lưu chính xác. Danh sách phản ánh đầy đủ dữ liệu. | High |
| TC-UC6-003 | Happy path: Chỉnh sửa danh mục VPP | 1. Chọn bản ghi trong danh sách. 2. Nhấn Edit. 3. Cập nhật Giá bán chưa VAT. 4. Nhấn Lưu. | Giá cũ: 3.000; Giá mới: 4.000 | Cập nhật thành công. Danh sách hiển thị thông tin mới. | High |
| TC-UC6-004 | Happy path: Xóa danh mục không trong phiếu chờ phê duyệt | 1. Tích chọn bản ghi không liên quan đến phiếu chờ phê duyệt. 2. Nhấn Xóa. 3. Xác nhận. | Mã SP không thuộc phiếu đang Chờ phê duyệt | Bản ghi bị xóa. Danh sách cập nhật. Tờ trình cũ đã tham chiếu không bị ảnh hưởng. | High |
| TC-UC6-005 | Happy path: Xóa nhiều bản ghi cùng lúc | 1. Tích chọn 3 bản ghi hợp lệ. 2. Nhấn Xóa. 3. Xác nhận. | 3 mã SP không trong phiếu chờ phê duyệt | 3 bản ghi bị xóa. Danh sách cập nhật. | Medium |
| TC-UC6-006 | Negative: Tạo mới thiếu Mã SP | 1. Nhấn Tạo mới. 2. Để trống Mã SP. 3. Nhập Tên sản phẩm. 4. Nhấn Lưu. | Mã SP: để trống; Tên: "Bút chì" | Hệ thống không lưu. Hiển thị lỗi yêu cầu nhập Mã SP. | High |
| TC-UC6-007 | Negative: Tạo mới thiếu Tên sản phẩm | 1. Nhấn Tạo mới. 2. Nhập Mã SP. 3. Để trống Tên sản phẩm. 4. Nhấn Lưu. | Mã SP: "BP003"; Tên: để trống | Hệ thống không lưu. Hiển thị lỗi yêu cầu nhập Tên sản phẩm. | High |
| TC-UC6-008 | Negative: Xóa bản ghi đang trong phiếu Chờ phê duyệt | 1. Tích chọn bản ghi có Mã SP đang trong tờ trình Chờ phê duyệt. 2. Nhấn Xóa. | Mã SP thuộc tờ trình đang Chờ phê duyệt | Hệ thống cảnh báo không cho phép xóa. Bản ghi không bị xóa. | High |
| TC-UC6-009 | Negative: Người dùng thường không thấy màn hình Quản lý danh mục | 1. Đăng nhập nhân viên thường. 2. Kiểm tra leftmenu. | Tài khoản không thuộc Ban hành chính | Màn hình không xuất hiện trong leftmenu. Không thể truy cập. | High |
| TC-UC6-010 | Edge case: Xóa nhiều bản ghi trong đó có 1 bản đang trong phiếu chờ phê duyệt | 1. Tích chọn 3 bản ghi, trong đó 1 bản đang trong phiếu Chờ phê duyệt. 2. Nhấn Xóa. | 2 bản hợp lệ + 1 bản trong phiếu chờ | Hệ thống cảnh báo bản không hợp lệ, không xóa bản đó. 2 bản hợp lệ xóa thành công. | High |
| TC-UC6-011 | Edge case: Tờ trình cũ tham chiếu mã SP bị xóa không bị ảnh hưởng | 1. Xóa 1 mã SP đã xuất hiện trong tờ trình đã Hoàn thành. 2. Mở tờ trình cũ. | Mã SP đã xóa; Tờ trình trạng thái Hoàn thành | Tờ trình cũ vẫn hiển thị đầy đủ thông tin mặt hàng. Dữ liệu lịch sử không mất. | High |
| TC-UC6-012 | Edge case: Cập nhật đơn giá trong danh mục — tờ trình mới lấy giá mới | 1. Chỉnh sửa Giá bán chưa VAT của 1 mã SP. 2. Tạo mới tờ trình, chọn đúng mã SP đó. 3. Kiểm tra đơn giá trong tờ trình. | Giá cũ: 3.000; Giá mới: 5.000 | Tờ trình mới lấy đơn giá 5.000. Tờ trình cũ đã tạo trước không thay đổi. | High |

---

# PHẦN 3: ĐĂNG KÝ XE (UC 11X–14X)

> **Nguồn:** SRS Fanxipan v1.0, phần 2.4 (Quy trình Đăng ký xe / Điều xe). Không có thay đổi theo BRD.
> Suffix **X** dùng để phân biệt với UC 11 Thiết lập phòng họp.

## UC 11X: Đăng ký sử dụng xe

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC11X-001 | Happy path: Tạo phiếu đăng ký xe và lưu trạng thái Đang soạn | 1. Đăng nhập. 2. Nhấn Tạo mới. 3. Chọn Loại: Đăng ký sử dụng xe. 4. Nhập Mục đích sử dụng, Thời gian đi, Địa điểm đón. 5. Nhấn Lưu. | Mục đích: "Họp khách hàng tại Hà Nội"; Thời gian đi: 02/06/2026 08:00; Địa điểm đón: "16A Nguyễn Công Trứ" | Phiếu lưu thành công trạng thái Đang soạn. Hiển thị nút Trình phê duyệt. | High |
| TC-UC11X-002 | Happy path: Trình phê duyệt thành công | 1. Mở phiếu Đang soạn. 2. Nhấn Trình phê duyệt. | Dữ liệu hợp lệ | Phiếu chuyển trạng thái Chờ phê duyệt. Gửi email ET1 tới Phụ trách bộ phận. | High |
| TC-UC11X-003 | Negative: Lưu thiếu trường bắt buộc "Mục đích sử dụng" | 1. Tạo phiếu. 2. Để trống Mục đích sử dụng. 3. Nhấn Lưu. | Mục đích: để trống | Hệ thống không lưu. Hiển thị lỗi yêu cầu nhập Mục đích sử dụng. | High |
| TC-UC11X-004 | Negative: Lưu thiếu trường bắt buộc "Thời gian đi" | 1. Tạo phiếu. 2. Để trống Thời gian đi. 3. Nhấn Lưu. | Thời gian đi: để trống | Hệ thống không lưu. Hiển thị lỗi trường bắt buộc. | High |
| TC-UC11X-005 | Negative: Lưu thiếu trường bắt buộc "Địa điểm đón" | 1. Tạo phiếu. 2. Để trống Địa điểm đón. 3. Nhấn Lưu. | Địa điểm đón: để trống | Hệ thống không lưu. Hiển thị lỗi. | High |
| TC-UC11X-006 | Negative: Nhấn Cancel khi đang tạo phiếu | 1. Tạo mới, nhập một số thông tin. 2. Nhấn Cancel. | Một số trường đã nhập | Màn hình đóng. Không lưu dữ liệu. Không tạo phiếu mới. | Medium |
| TC-UC11X-007 | Edge case: Trường Thời gian về không bắt buộc | 1. Tạo phiếu với đủ trường bắt buộc. 2. Để trống Thời gian về. 3. Nhấn Lưu. | Thời gian về: để trống | Phiếu lưu thành công. | Medium |
| TC-UC11X-008 | Edge case: Trường Ghi chú và Nơi đến không bắt buộc | 1. Tạo phiếu, nhập đủ trường bắt buộc, để trống Ghi chú và Nơi đến. 2. Nhấn Lưu. | Ghi chú: để trống; Nơi đến: để trống | Phiếu lưu thành công. | Low |
| TC-UC11X-009 | Edge case: Chỉnh sửa phiếu trạng thái Đang soạn | 1. Lưu phiếu (Đang soạn). 2. Mở lại. 3. Thay đổi Mục đích sử dụng. 4. Nhấn Lưu. | Mục đích mới: "Đưa đón lãnh đạo" | Cập nhật thành công. Phiếu vẫn Đang soạn với thông tin mới. | Medium |
| TC-UC11X-010 | Edge case: Không cho phép chỉnh sửa sau khi phiếu đã Chờ phê duyệt | 1. Trình phê duyệt thành công. 2. Mở lại phiếu đang Chờ phê duyệt. 3. Thử chỉnh sửa. | Phiếu trạng thái Chờ phê duyệt | Các trường không cho phép chỉnh sửa. Nút Lưu/Trình phê duyệt không hiển thị. | High |
| TC-UC11X-011 | Edge case: Kiểm tra log sau khi tạo phiếu | 1. Tạo và lưu phiếu thành công. 2. Kiểm tra log. | Phiếu vừa tạo | Log ghi nhận: Người thực hiện, Thời gian, Hành động "Tạo mới". | Medium |
| TC-UC11X-012 | Edge case: Trường thông tin người dùng (Đơn vị, Người tạo) tự động điền và không sửa được | 1. Mở form tạo mới. 2. Quan sát trường Đơn vị, Người tạo. 3. Thử chỉnh sửa. | Người dùng đăng nhập: tài khoản hợp lệ | Tự động điền đúng thông tin người đăng nhập. Không cho phép chỉnh sửa. | Medium |

---

## UC 12X: Phê duyệt/Trả lại phiếu đăng ký sử dụng xe

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC12X-001 | Happy path: Phụ trách bộ phận phê duyệt thành công | 1. Đăng nhập Phụ trách bộ phận. 2. Vào Cần xử lý. 3. Chọn phiếu Chờ phê duyệt. 4. Nhấn Phê duyệt. | Phiếu hợp lệ trạng thái Chờ phê duyệt | Phiếu chuyển trạng thái Chờ điều xe. Chuyển tiếp sang Cán bộ quản lý xe. | High |
| TC-UC12X-002 | Happy path: Trả lại phiếu kèm ý kiến | 1. Đăng nhập Phụ trách bộ phận. 2. Chọn phiếu. 3. Nhấn Trả lại. 4. Nhập Ý kiến. 5. Xác nhận. | Ý kiến: "Mục đích chưa rõ, vui lòng bổ sung" | Phiếu chuyển trạng thái Trả lại. Kết thúc luồng. Người tạo nhận email ET3. | High |
| TC-UC12X-003 | Negative: Trả lại không nhập Ý kiến | 1. Nhấn Trả lại. 2. Để trống Ý kiến. 3. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Hiển thị lỗi bắt buộc nhập Ý kiến. | High |
| TC-UC12X-004 | Negative: Người không phải Phụ trách bộ phận cố phê duyệt | 1. Đăng nhập nhân viên thường. 2. Truy cập phiếu Chờ phê duyệt. | Tài khoản không có quyền phê duyệt | Không hiển thị nút Phê duyệt/Trả lại. Chỉ xem. | High |
| TC-UC12X-005 | Negative: Tài khoản không có quyền truy cập phiếu | 1. Đăng nhập tài khoản không liên quan. 2. Thử truy cập URL phiếu. | Tài khoản không phải người tạo, không được chia sẻ | Hệ thống từ chối. Không hiển thị nội dung. | High |
| TC-UC12X-006 | Edge case: Người tạo chỉ xem khi phiếu đang Chờ phê duyệt | 1. Đăng nhập người tạo phiếu. 2. Mở phiếu Chờ phê duyệt. | Phiếu do tài khoản này tạo ra | Xem được thông tin. Không hiển thị nút Phê duyệt/Trả lại. | Medium |
| TC-UC12X-007 | Edge case: Sau phê duyệt, trạng thái phiếu là "Chờ điều xe" | 1. Phụ trách phê duyệt. 2. Kiểm tra trạng thái phiếu. | Phiếu vừa phê duyệt | Trạng thái: Chờ điều xe. Phiếu xuất hiện trong danh sách Cán bộ quản lý xe. | High |
| TC-UC12X-008 | Edge case: Sau Trả lại, người tạo có thể chỉnh sửa và trình lại | 1. Phiếu bị Trả lại. 2. Người tạo mở phiếu, chỉnh sửa, Trình phê duyệt lại. | Phiếu trạng thái Trả lại | Phiếu trình lại thành công. Chuyển sang Chờ phê duyệt. | Medium |
| TC-UC12X-009 | Edge case: Log ghi nhận khi Phê duyệt | 1. Phụ trách phê duyệt. 2. Kiểm tra log. | Hành động: Phê duyệt | Log ghi: Hành động, Ý kiến (nếu có), Thời gian, Người xử lý. | Medium |
| TC-UC12X-010 | Edge case: Log ghi nhận khi Trả lại | 1. Phụ trách trả lại kèm ý kiến. 2. Kiểm tra log. | Ý kiến: "Cần bổ sung thông tin" | Log ghi: Hành động Trả lại, Ý kiến, Thời gian, Người xử lý. | Medium |
| TC-UC12X-011 | Edge case: Màn hình Cần xử lý chỉ hiện phiếu thuộc thẩm quyền người dùng | 1. Đăng nhập Phụ trách bộ phận A. 2. Xem danh sách Cần xử lý. | Có nhiều phiếu nhiều phòng ban | Chỉ hiển thị phiếu mà người dùng là người xử lý bước hiện tại. | Medium |
| TC-UC12X-012 | Edge case: Phiếu đã Trả lại không còn trong danh sách Cần xử lý | 1. Phiếu bị Trả lại. 2. Đăng nhập Phụ trách bộ phận. 3. Xem danh sách Cần xử lý. | Phiếu trạng thái Trả lại | Phiếu không còn trong danh sách Cần xử lý. | Medium |

---

## UC 13X: Cán bộ phụ trách thực hiện điều xe

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC13X-001 | Happy path: Điều xe công ty — tự chọn tài xế | 1. Đăng nhập Cán bộ quản lý xe. 2. Mở phiếu Chờ điều xe. 3. Chọn xe từ danh sách. 4. Chọn tài xế thủ công. 5. Nhấn Xác nhận. | Xe: Toyota Innova; Tài xế: Nguyễn Văn A | Phiếu chuyển trạng thái Đã điều xe. Người đăng ký nhận email ET7 với thông tin xe và tài xế. | High |
| TC-UC13X-002 | Happy path: Điều xe — xe đã có tài xế gán sẵn | 1. Mở phiếu Chờ điều xe. 2. Chọn xe đã cấu hình tài xế. 3. Nhấn Xác nhận. | Xe: 30A-123.45 (đã gán tài xế Trần Văn B) | Hệ thống tự điền tài xế Trần Văn B. Phiếu chuyển Đã điều xe. | High |
| TC-UC13X-003 | Happy path: Điều xe bằng Thẻ taxi | 1. Mở phiếu Chờ điều xe. 2. Chọn hình thức Thẻ taxi. 3. Nhập số Thẻ taxi. 4. Nhấn Xác nhận. | Thẻ taxi: "TAXI-2026-001" | Phiếu chuyển Đã điều xe. Email ET7 gửi người đăng ký. | High |
| TC-UC13X-004 | Negative: Xác nhận điều xe khi chưa chọn xe | 1. Mở phiếu Chờ điều xe. 2. Không chọn xe. 3. Nhấn Xác nhận. | Trường Chọn xe: để trống | Hệ thống không thực hiện. Hiển thị cảnh báo trường bắt buộc. | High |
| TC-UC13X-005 | Negative: Trả lại phiếu không nhập Ý kiến | 1. Nhấn Trả lại. 2. Để trống Ý kiến. 3. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Yêu cầu nhập Ý kiến. | High |
| TC-UC13X-006 | Negative: Người không thuộc Cán bộ quản lý xe cố thực hiện điều xe | 1. Đăng nhập nhân viên thường. 2. Mở phiếu Chờ điều xe. | Tài khoản nhân viên thường | Không hiển thị nút Xác nhận điều xe. | High |
| TC-UC13X-007 | Edge case: Ghi chú không bắt buộc khi điều xe | 1. Chọn xe hợp lệ. 2. Để trống Ghi chú. 3. Nhấn Xác nhận. | Ghi chú: để trống | Điều xe thành công. | Low |
| TC-UC13X-008 | Edge case: Hệ thống không kiểm tra xe có đang bận không (theo SRS) | 1. Chọn xe đang được điều cho phiếu khác trong cùng khung giờ. 2. Nhấn Xác nhận. | Xe đã được điều cho phiếu khác trùng giờ | Hệ thống cho phép điều xe thành công (không block). Cán bộ chịu trách nhiệm kiểm tra ngoài hệ thống. | High |
| TC-UC13X-009 | Edge case: Sau Trả lại, phiếu về trạng thái Trả lại, người tạo thấy ý kiến | 1. Cán bộ trả lại phiếu với Ý kiến. 2. Người tạo kiểm tra phiếu. | Ý kiến: "Không có xe khả dụng ngày này" | Phiếu chuyển trạng thái Trả lại. Người tạo thấy ý kiến của cán bộ. | Medium |
| TC-UC13X-010 | Edge case: Thông tin xe tự động hiển thị khi chọn xe | 1. Cán bộ chọn xe từ droplist. 2. Quan sát trường Loại xe, Biển số, Hãng xe. | Xe: Toyota Innova 30A-123.45 | Hệ thống tự động điền Loại xe, Biển số, Hãng xe. Không cho chỉnh sửa các trường này. | Medium |
| TC-UC13X-011 | Edge case: Log ghi nhận khi Xác nhận điều xe | 1. Cán bộ điều xe thành công. 2. Kiểm tra log. | Hành động: Xác nhận | Log ghi: Hành động Xác nhận, Ý kiến (nếu có), Thời gian, Người xử lý. | Medium |
| TC-UC13X-012 | Edge case: Trường Thẻ taxi bắt buộc khi chọn hình thức Thẻ taxi | 1. Chọn hình thức Thẻ taxi. 2. Để trống số Thẻ taxi. 3. Nhấn Xác nhận. | Thẻ taxi: để trống | Hệ thống không thực hiện. Yêu cầu nhập số Thẻ taxi. | High |

---

## UC 14X: Cán bộ xác nhận hoàn thành chuyến đi

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC14X-001 | Happy path: Xác nhận hoàn thành chuyến đi thành công | 1. Đăng nhập Cán bộ phụ trách điều xe. 2. Mở phiếu Đã điều xe. 3. Nhấn Xác nhận hoàn thành. | Phiếu trạng thái Đã điều xe | Phiếu chuyển trạng thái Hoàn thành. | High |
| TC-UC14X-002 | Happy path: Cập nhật Thời gian về trước khi xác nhận hoàn thành | 1. Mở phiếu Đã điều xe. 2. Chỉnh sửa Thời gian về (muộn hơn dự kiến). 3. Nhấn Xác nhận hoàn thành. | Thời gian về mới: 02/06/2026 18:30 (thay vì 17:00) | Thời gian về cập nhật. Phiếu chuyển Hoàn thành với thời gian về mới. | High |
| TC-UC14X-003 | Negative: Người không thuộc nhóm phụ trách xe cố xác nhận hoàn thành | 1. Đăng nhập nhân viên thường. 2. Mở phiếu Đã điều xe. | Tài khoản nhân viên thường | Không hiển thị nút Xác nhận hoàn thành. | High |
| TC-UC14X-004 | Negative: Không thể xác nhận hoàn thành khi phiếu chưa ở trạng thái Đã điều xe | 1. Mở phiếu trạng thái Chờ điều xe. 2. Tìm nút Xác nhận hoàn thành. | Phiếu trạng thái Chờ điều xe | Không hiển thị nút Xác nhận hoàn thành. | High |
| TC-UC14X-005 | Negative: Tài khoản không liên quan không truy cập được phiếu | 1. Đăng nhập tài khoản không liên quan. 2. Thử truy cập phiếu Đã điều xe. | Tài khoản ngoài luồng xử lý | Hệ thống từ chối truy cập. | High |
| TC-UC14X-006 | Edge case: Thời gian về có thể để trống khi xác nhận hoàn thành | 1. Mở phiếu Đã điều xe. 2. Không cập nhật Thời gian về. 3. Nhấn Xác nhận hoàn thành. | Thời gian về: để trống hoặc giữ nguyên | Xác nhận thành công. Phiếu chuyển Hoàn thành. | Medium |
| TC-UC14X-007 | Edge case: Phiếu Hoàn thành không thể xác nhận lại | 1. Xác nhận hoàn thành thành công. 2. Mở lại phiếu Hoàn thành. 3. Tìm nút Xác nhận hoàn thành. | Phiếu trạng thái Hoàn thành | Không hiển thị nút. Phiếu ở trạng thái cuối. | Medium |
| TC-UC14X-008 | Edge case: Người tạo xem phiếu đã Hoàn thành với thông tin đầy đủ | 1. Đăng nhập người tạo. 2. Vào Tờ trình của tôi. 3. Mở phiếu Hoàn thành. | Phiếu trạng thái Hoàn thành | Hiển thị đầy đủ: thông tin xe, tài xế, Thời gian về cập nhật. Không hiển thị nút xử lý. | Medium |
| TC-UC14X-009 | Edge case: Log ghi nhận khi Xác nhận hoàn thành | 1. Xác nhận hoàn thành. 2. Kiểm tra log. | Hành động: Xác nhận hoàn thành | Log ghi: Hành động, Ý kiến (nếu có), Thời gian, Người xử lý. | Medium |
| TC-UC14X-010 | Edge case: Tab Quy trình hiển thị đầy đủ lịch sử toàn bộ luồng | 1. Mở phiếu Hoàn thành. 2. Xem tab Quy trình. | Phiếu đã qua: Tạo → Phê duyệt → Điều xe → Hoàn thành | Tab Quy trình hiển thị đầy đủ các bước, người xử lý, thời gian, ý kiến. | Medium |
| TC-UC14X-011 | Edge case: Cán bộ không thể chỉnh sửa thông tin đăng ký ban đầu ở bước này | 1. Mở phiếu Đã điều xe. 2. Thử chỉnh sửa Mục đích sử dụng. | Phiếu trạng thái Đã điều xe | Mục đích sử dụng và các trường gốc không cho phép chỉnh sửa. Chỉ Thời gian về có thể cập nhật. | Medium |
| TC-UC14X-012 | Edge case: Xác nhận hoàn thành ngay sau khi vừa điều xe | 1. Điều xe thành công. 2. Ngay lập tức xác nhận hoàn thành cùng phiếu. | Phiếu vừa chuyển sang Đã điều xe | Hệ thống chấp nhận. Phiếu chuyển Hoàn thành. | Low |

---

## E2E — Quy trình Đăng ký xe đầu đến cuối

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-E2E-XE-001 | E2E Happy path: Toàn bộ quy trình từ đăng ký đến hoàn thành | 1. [CBNV] Tạo phiếu đăng ký xe, Lưu, Trình phê duyệt. 2. [Phụ trách BP] Nhận ET1, Phê duyệt. 3. [Cán bộ xe] Chọn xe + tài xế, Xác nhận điều xe. 4. [CBNV] Nhận ET7. 5. [Cán bộ xe] Cập nhật Thời gian về, Xác nhận hoàn thành. | CBNV: Nguyễn Văn A; Xe: Toyota Innova 30A-123.45; Tài xế: Trần Văn C | Bước 1: Phiếu Đang soạn → Chờ phê duyệt. Bước 2: Phiếu Chờ điều xe. Bước 3: Phiếu Đã điều xe, ET7 gửi. Bước 5: Phiếu Hoàn thành. Log đầy đủ. | Critical |
| TC-E2E-XE-002 | E2E: Trả lại → Chỉnh sửa → Trình lại → Hoàn thành | 1. CBNV tạo, trình. 2. Phụ trách Trả lại kèm ý kiến. 3. CBNV chỉnh sửa, trình lại. 4. Phụ trách Phê duyệt. 5. Điều xe. 6. Hoàn thành. | Lần 1 thiếu Nơi đến; Lần 2 bổ sung "Sân bay Nội Bài" | Sau bước 2: Phiếu Trả lại, ET3 gửi. Sau bước 3: Phiếu Chờ phê duyệt lần 2. Toàn bộ quy trình tiếp tục bình thường. | High |
| TC-E2E-XE-003 | E2E: Trạng thái phiếu hiển thị đúng từng bước | 1. Tạo phiếu. 2. Trình phê duyệt. 3. Được phê duyệt. 4. Điều xe. 5. Hoàn thành. Kiểm tra trạng thái sau mỗi bước. | Một phiếu xuyên suốt 5 bước | Trạng thái đúng theo thứ tự: Đang soạn → Chờ phê duyệt → Chờ điều xe → Đã điều xe → Hoàn thành. | High |
| TC-E2E-XE-004 | E2E: Email gửi đúng người đúng bước | 1. Thực hiện toàn bộ quy trình E2E. 2. Kiểm tra hộp thư từng actor. | Các tài khoản: CBNV, Phụ trách BP, Cán bộ xe | Trình phê duyệt: Phụ trách BP nhận ET1. Điều xe thành công: CBNV nhận ET7. Không có email gửi nhầm hoặc thiếu bước. | High |

---

# PHẦN 4: TRÌNH KÝ VĂN BẢN (UC 15–19)

> **Nguồn:** SRS Fanxipan v1.0, phần 2.5 (Quy trình Trình ký văn bản). Không có thay đổi theo BRD.

## UC 15: Tạo đề xuất trình ký hợp đồng/văn bản

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC15-001 | Happy path: Tạo mới đề xuất trình ký thành công và lưu nháp | 1. Đăng nhập. 2. Nhấn Tạo mới. 3. Nhập Nội dung, Loại hợp đồng, tải Tài liệu trình ký. 4. Nhấn Lưu. | Nội dung: "Hợp đồng dịch vụ Marketing Q2"; Loại: "Hợp đồng dịch vụ Marketing"; File PDF hợp lệ. | Tờ trình lưu trạng thái "Đang soạn". Nút Thiết lập vị trí ký và Trình phê duyệt hiển thị. | High |
| TC-UC15-002 | Happy path: Trình phê duyệt thành công | 1. Mở tờ trình Đang soạn. 2. Kiểm tra quy trình đủ người phê duyệt. 3. Nhấn Trình phê duyệt. | Tờ trình hợp lệ, quy trình đầy đủ | Tờ trình chuyển "Chờ phê duyệt". Gửi email ET1 cho người phê duyệt bước đầu tiên. | High |
| TC-UC15-003 | Happy path: Tạo đề xuất nhóm Vận hành có Giá trị tham chiếu | 1. Đăng nhập. 2. Tạo mới, chọn Loại yêu cầu "Vận hành". 3. Chọn "Hợp đồng liên quan đến mảng điện". 4. Nhập Giá trị tham chiếu. 5. Lưu. | Loại HĐ: "Hợp đồng liên quan đến mảng điện"; Giá trị: 500.000.000 | Trường Giá trị tham chiếu lưu thành công. Quy trình tự động lấy theo Loại hợp đồng + Giá trị. | High |
| TC-UC15-004 | Negative: Thiếu trường bắt buộc Nội dung | 1. Tạo mới. 2. Để trống Nội dung. 3. Nhấn Lưu. | Nội dung: để trống | Hệ thống không lưu. Hiển thị lỗi tại trường Nội dung. | High |
| TC-UC15-005 | Negative: Thiếu Tài liệu trình ký | 1. Tạo mới. 2. Nhập đủ trường text. 3. Không upload Tài liệu trình ký. 4. Nhấn Lưu. | Tài liệu trình ký: không upload | Hệ thống không lưu. Hiển thị lỗi yêu cầu upload tài liệu. | High |
| TC-UC15-006 | Negative: Trình phê duyệt khi quy trình thiếu người phê duyệt | 1. Tạo và lưu tờ trình. 2. Bỏ trống người phê duyệt ở một bước. 3. Nhấn Trình phê duyệt. | Quy trình: thiếu người phê duyệt bước 2 | Hệ thống cảnh báo. Không gửi. Trạng thái vẫn Đang soạn. | High |
| TC-UC15-007 | Negative: Nhấn Cancel khi đang tạo mới | 1. Mở form tạo mới. 2. Nhập một số thông tin. 3. Nhấn Cancel. | Nội dung: "Test" (chưa lưu) | Màn hình đóng. Không có tờ trình nào được lưu. | Medium |
| TC-UC15-008 | Edge case: Trường Giá trị tham chiếu chỉ hiển thị khi chọn loại HĐ điện | 1. Chọn Loại HĐ khác (không phải HĐ điện). 2. Quan sát. 3. Chuyển sang "HĐ liên quan đến mảng điện". | Loại 1: "HĐ tư vấn"; Loại 2: "HĐ liên quan đến mảng điện" | Bước 2: Trường Giá trị tham chiếu không hiển thị. Bước 3: Trường Giá trị tham chiếu xuất hiện và bắt buộc nhập. | High |
| TC-UC15-009 | Edge case: Trường Đơn vị và Người tạo không cho chỉnh sửa | 1. Đăng nhập user A. 2. Mở form tạo mới. 3. Thử chỉnh sửa Đơn vị hoặc Người tạo. | Tài khoản: user A thuộc phòng ban B | Tự động điền theo đăng nhập. Không cho phép chỉnh sửa. | Medium |
| TC-UC15-010 | Edge case: Cố chỉnh sửa tờ trình trạng thái Chờ phê duyệt | 1. Trình phê duyệt thành công. 2. Cố chỉnh sửa nội dung. | Tờ trình trạng thái Chờ phê duyệt | Hệ thống không cho phép. Tab Quy trình và nút Trình phê duyệt không còn hiển thị. | High |
| TC-UC15-011 | Edge case: Quy trình tự động thay đổi khi đổi loại hợp đồng | 1. Tạo mới, chọn Loại yêu cầu "Phát triển & Quản lý nghỉ dưỡng", chọn loại HĐ. 2. Quan sát tab Quy trình. | Loại yêu cầu: "Phát triển & Quản lý nghỉ dưỡng" | Luồng phê duyệt hiển thị đúng quy trình cấu hình cho loại yêu cầu. | High |
| TC-UC15-012 | Edge case: Ghi log khi tạo mới tờ trình | 1. Tạo mới và lưu thành công. 2. Kiểm tra log. | Tờ trình lưu thành công | Log ghi: Người tạo, Thời gian, Hành động "Tạo mới tờ trình". | Medium |

---

## UC 16: Thiết lập vị trí ký

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC16-001 | Happy path: Thiết lập vị trí ký thành công cho 1 người ký | 1. Mở tờ trình Đang soạn (trình ký văn bản). 2. Nhấn Thiết lập vị trí ký. 3. Chọn tài liệu. 4. Nhấn vào tên người ký. 5. Kéo thả chữ ký vào vị trí mong muốn. 6. Nhấn Lưu. | Tờ trình Đang soạn; PDF hợp lệ; 1 người ký. | Vị trí ký lưu thành công. Màn hình thiết lập đóng lại. | High |
| TC-UC16-002 | Happy path: Thiết lập vị trí ký cho nhiều người ký | 1. Mở tờ trình có nhiều người ký. 2. Nhấn Thiết lập vị trí ký. 3. Lần lượt đặt chữ ký từng người ở vị trí khác nhau. 4. Nhấn Lưu. | Bước ký có 2 người: Lãnh đạo A và Lãnh đạo B | Hệ thống lưu vị trí ký riêng cho từng người. | High |
| TC-UC16-003 | Happy path: Đặt 1 chữ ký ở nhiều vị trí trên nhiều trang | 1. Chọn người ký. 2. Đặt ký ở trang 1. 3. Chuyển trang 2, đặt thêm chữ ký. 4. Nhấn Lưu. | Tài liệu nhiều trang; 1 người ký đặt 2 vị trí | Hệ thống lưu cả 2 vị trí trên 2 trang cho cùng 1 người. | Medium |
| TC-UC16-004 | Negative: Nút Thiết lập vị trí ký không hiện với người phê duyệt | 1. Đăng nhập người phê duyệt. 2. Mở tờ trình đang Chờ phê duyệt. 3. Quan sát giao diện. | Tài khoản: người phê duyệt bước 1 | Nút Thiết lập vị trí ký không xuất hiện. | High |
| TC-UC16-005 | Negative: Nút không hiện khi tờ trình đã qua bước ký | 1. Mở tờ trình đã đến bước Lãnh đạo ký duyệt. 2. Đăng nhập người trình. 3. Quan sát. | Tờ trình đã đến bước ký duyệt | Nút Thiết lập vị trí ký không hiển thị. | High |
| TC-UC16-006 | Negative: Tờ trình không thuộc nhóm Trình ký văn bản | 1. Mở tờ trình đăng ký xe Đang soạn. 2. Quan sát. | Tờ trình loại Đăng ký xe | Nút Thiết lập vị trí ký không hiển thị. | High |
| TC-UC16-007 | Edge case: Người trình thiết lập khi tờ trình Đang soạn | 1. Tạo tờ trình trình ký văn bản, Lưu (Đang soạn). 2. Nhấn Thiết lập vị trí ký. | Tờ trình Đang soạn | Nút hiển thị và chức năng hoạt động bình thường. | High |
| TC-UC16-008 | Edge case: Tài liệu trình ký có nhiều file | 1. Tạo tờ trình với 2 file tài liệu. 2. Mở thiết lập vị trí ký. 3. Quan sát danh sách tài liệu. | Tài liệu: 2 file PDF | Danh sách hiển thị đủ 2 tài liệu. Có thể chuyển sang từng tài liệu để thiết lập độc lập. | Medium |
| TC-UC16-009 | Edge case: Không thiết lập vị trí ký — chữ ký chèn mặc định cuối trang cuối | 1. Tạo tờ trình, không thiết lập vị trí ký. 2. Hoàn thành luồng đến bước ký. 3. Lãnh đạo ký số. | Không có thiết lập vị trí ký | Chữ ký chèn mặc định cuối trang cuối tài liệu. | Medium |
| TC-UC16-010 | Edge case: Kéo thả tự do ở bất kỳ vị trí nào trên trang | 1. Mở thiết lập. 2. Kéo thả chữ ký đến góc trên trái, giữa trang, góc dưới phải. | 3 vị trí khác nhau | Cho phép đặt ở bất kỳ vị trí nào. Vị trí ghi nhận chính xác. | Medium |
| TC-UC16-011 | Edge case: Chức năng thiết lập vị trí ký không ghi log (theo SRS) | 1. Thực hiện thiết lập thành công. 2. Kiểm tra lịch sử tờ trình. | Thiết lập thành công | Không có log cho hành động thiết lập vị trí ký. | Low |

---

## UC 17: Xác nhận/Phê duyệt/Đồng ý/Trả lại tờ trình trình ký văn bản

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC17-001 | Happy path: Trưởng phòng xác nhận thành công | 1. Đăng nhập Trưởng phòng. 2. Vào Cần xử lý. 3. Mở tờ trình ở bước mình. 4. Nhấn Xác nhận. | Tờ trình đang ở bước Trưởng phòng | Kết quả bước: Xác nhận. Chuyển sang bước Phòng ban liên quan thẩm định. | High |
| TC-UC17-002 | Happy path: Tất cả người Phòng ban liên quan phê duyệt | 1. Đăng nhập lần lượt từng thành viên phòng ban liên quan. 2. Mỗi người mở tờ trình và Phê duyệt. | Bước thẩm định: 2 người, cả 2 Phê duyệt | Sau khi người cuối phê duyệt, tờ trình chuyển sang bước Tổ Thư ký/Trợ lý soát xét. | High |
| TC-UC17-003 | Happy path: Tổ Thư ký/Trợ lý đồng ý | 1. Đăng nhập Thư ký/Trợ lý. 2. Mở tờ trình ở bước soát xét. 3. Nhấn Đồng ý. | Tờ trình đang ở bước Tổ Thư ký/Trợ lý | Tờ trình chuyển sang bước Lãnh đạo ký duyệt (UC 19). | High |
| TC-UC17-004 | Happy path: Trưởng phòng trả lại kèm ý kiến | 1. Đăng nhập Trưởng phòng. 2. Nhấn Trả lại. 3. Nhập lý do. 4. Xác nhận. | Lý do: "Nội dung HĐ chưa đúng mẫu" | Tờ trình chuyển trạng thái "Trả lại". Kết thúc quy trình. Gửi email ET3 cho người trình. | High |
| TC-UC17-005 | Negative: Trả lại không nhập ý kiến | 1. Đăng nhập người phê duyệt. 2. Nhấn Trả lại. 3. Để trống Ý kiến. 4. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Hiển thị lỗi yêu cầu nhập Ý kiến. | High |
| TC-UC17-006 | Negative: Người không phải người xử lý bước hiện tại | 1. Đăng nhập người phê duyệt bước 2 khi tờ trình ở bước 1. | Tờ trình đang ở bước 1 | Không hiển thị nút chức năng. Chỉ xem. | High |
| TC-UC17-007 | Negative: Người không có quyền truy cập cố mở tờ trình | 1. Đăng nhập nhân viên không liên quan. 2. Truy cập URL trực tiếp. | Tài khoản không liên quan | Hệ thống không cho truy cập. Thông báo lỗi hoặc chuyển hướng. | High |
| TC-UC17-008 | Edge case: Phòng ban liên quan — 1 người trả lại thì toàn bộ bị trả lại | 1. Bước thẩm định: 2 người. Người 1 Phê duyệt. Người 2 Trả lại kèm lý do. | Bước 2 người; người 1 PD; người 2 TL | Tờ trình chuyển trạng thái Trả lại ngay khi người 2 nhấn trả lại. Email ET3 gửi người trình. | High |
| TC-UC17-009 | Edge case: Còn người chưa xử lý ở bước thẩm định — tờ trình chưa chuyển bước | 1. Bước thẩm định: 2 người. Chỉ 1 người Phê duyệt. | Bước 2 người; chỉ 1 người PD | Tờ trình vẫn ở bước hiện tại. Không chuyển sang Tổ Thư ký. | High |
| TC-UC17-010 | Edge case: Người được chia sẻ chỉ xem | 1. Người trình chia sẻ tờ trình. 2. Người được chia sẻ mở tờ trình. | Người được chia sẻ không phải người phê duyệt | Chỉ xem được thông tin. Không có nút thao tác. | Medium |
| TC-UC17-011 | Edge case: Log ghi nhận đầy đủ khi xác nhận/phê duyệt/trả lại | 1. Thực hiện hành động Xác nhận. 2. Kiểm tra lịch sử. | Bất kỳ hành động | Log ghi: Hành động, Ý kiến, Thời gian, Người xử lý. | Medium |
| TC-UC17-012 | Edge case: Màn hình Cần xử lý hiển thị đúng danh sách | 1. Đăng nhập người phê duyệt bước hiện tại. 2. Vào Cần xử lý. | Có 3 tờ trình chờ xử lý bởi người này | Đúng 3 tờ trình hiển thị. Tờ trình không thuộc bước của người dùng không xuất hiện. | High |

---

## UC 18: Yêu cầu điều chỉnh tờ trình trình ký văn bản

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC18-001 | Happy path: Tổ Thư ký/Trợ lý yêu cầu điều chỉnh thành công | 1. Đăng nhập Tổ Thư ký/Trợ lý. 2. Mở tờ trình ở bước soát xét. 3. Nhấn Yêu cầu điều chỉnh. 4. Nhập ý kiến. 5. Xác nhận. | Ý kiến: "Cần bổ sung phụ lục giá trị HĐ" | Tờ trình chuyển trạng thái "Yêu cầu điều chỉnh". Người trình được phép chỉnh sửa lại. | High |
| TC-UC18-002 | Happy path: Người trình chỉnh sửa và gửi lại — nhảy thẳng về Tổ Thư ký | 1. Tờ trình trạng thái "Yêu cầu điều chỉnh". 2. Người trình chỉnh sửa nội dung/tài liệu. 3. Gửi lại. | Người trình cập nhật tài liệu trình ký mới | Tờ trình chuyển trực tiếp sang bước Tổ Thư ký/Trợ lý soát xét. Không qua lại Trưởng phòng hay Phòng ban. | High |
| TC-UC18-003 | Negative: Yêu cầu điều chỉnh không nhập ý kiến | 1. Đăng nhập Thư ký/Trợ lý. 2. Nhấn Yêu cầu điều chỉnh. 3. Để trống ý kiến. 4. Xác nhận. | Ý kiến: để trống | Hệ thống không thực hiện. Hiển thị lỗi yêu cầu nhập ý kiến. | High |
| TC-UC18-004 | Negative: Người không phải Tổ Thư ký/Trợ lý không thể yêu cầu điều chỉnh | 1. Đăng nhập Trưởng phòng. 2. Mở tờ trình đang ở bước soát xét. | Tài khoản: Trưởng phòng | Nút Yêu cầu điều chỉnh không hiển thị. | High |
| TC-UC18-005 | Negative: Người trình không thể chỉnh sửa khi tờ trình đang Chờ phê duyệt bình thường | 1. Tờ trình trạng thái Chờ phê duyệt (bước Trưởng phòng). 2. Người trình cố chỉnh sửa. | Trạng thái: Chờ phê duyệt, chưa qua soát xét | Không cho phép chỉnh sửa. | High |
| TC-UC18-006 | Edge case: Tổ Thư ký 2 người — 1 Đồng ý + 1 Yêu cầu điều chỉnh | 1. Bước soát xét 2 người. 2. Thư ký 1 Đồng ý. 3. Thư ký 2 Yêu cầu điều chỉnh. | Bước: 2 người | Tờ trình chuyển trạng thái "Yêu cầu điều chỉnh". (Cần xác nhận thêm với PO/Dev) | High |
| TC-UC18-007 | Edge case: Sau khi gửi lại, các bước trước không lặp lại | 1. Hoàn thành flow điều chỉnh → người trình gửi lại. 2. Kiểm tra quy trình. | Tờ trình sau khi gửi lại | Tờ trình nhảy thẳng bước Tổ Thư ký. Không thông báo chờ xử lý gửi đến Trưởng phòng hay Phòng ban. | High |
| TC-UC18-008 | Edge case: Người trình upload file mới khi điều chỉnh | 1. Tờ trình trạng thái "Yêu cầu điều chỉnh". 2. Người trình xóa file cũ, upload file mới. 3. Gửi lại. | File mới: phiên bản HĐ đã sửa | Tờ trình lưu tài liệu mới. Tổ Thư ký nhận bản tài liệu cập nhật khi soát xét lại. | High |
| TC-UC18-009 | Edge case: Log ghi nhận khi yêu cầu điều chỉnh | 1. Yêu cầu điều chỉnh thành công. 2. Kiểm tra log. | Yêu cầu điều chỉnh thành công | Log ghi: Hành động "Yêu cầu điều chỉnh", Ý kiến, Thời gian, Người xử lý. | Medium |
| TC-UC18-010 | Edge case: Nút Yêu cầu điều chỉnh chỉ hiển thị đúng bước | 1. Đăng nhập Thư ký. 2. Mở tờ trình đang ở bước Trưởng phòng (không phải soát xét). | Tờ trình bước Trưởng phòng | Nút Yêu cầu điều chỉnh không hiển thị. | High |

---

## UC 19: Lãnh đạo có thẩm quyền ký duyệt văn bản

| ID | Test Scenario | Test Steps | Test Data | Expected Result | Priority |
|---|---|---|---|---|---|
| TC-UC19-001 | Happy path: Lãnh đạo ký số thành công — đã thiết lập vị trí ký | 1. Đăng nhập Lãnh đạo. 2. Vào Cần xử lý. 3. Mở tờ trình ở bước ký duyệt. 4. Nhấn Ký số. 5. Nhập ghi chú. 6. Xác nhận. | Tờ trình có thiết lập vị trí ký; Lãnh đạo có chữ ký số hợp lệ | Chữ ký chèn đúng vị trí đã thiết lập. Tờ trình chuyển "Đã phê duyệt". Gửi email ET2 cho người trình. | High |
| TC-UC19-002 | Happy path: Lãnh đạo ký số — không có thiết lập vị trí ký | 1. Mở tờ trình chưa thiết lập vị trí ký. 2. Nhấn Ký số và xác nhận. | Không có thiết lập vị trí ký | Chữ ký chèn mặc định cuối trang cuối. Tờ trình chuyển "Đã phê duyệt". Email ET2 gửi người trình. | High |
| TC-UC19-003 | Happy path: Nội dung tài liệu sau ký số chứa đầy đủ thông tin ký | 1. Lãnh đạo ký số thành công. 2. Mở tài liệu sau ký. | Tài liệu PDF; Lãnh đạo có hình ảnh chữ ký trong hệ thống | Tài liệu chứa: Hình ảnh chữ ký + Tên người ký + thông tin ký bên phải hình ảnh. | High |
| TC-UC19-004 | Happy path: Lãnh đạo trả lại kèm ý kiến | 1. Đăng nhập Lãnh đạo. 2. Mở tờ trình bước ký duyệt. 3. Nhấn Trả lại. 4. Nhập lý do. 5. Xác nhận. | Lý do: "Cần bổ sung điều khoản thanh toán" | Tờ trình chuyển "Trả lại". Kết thúc quy trình. Gửi email ET3 cho người trình. | High |
| TC-UC19-005 | Negative: Người không phải Lãnh đạo có thẩm quyền không có nút Ký số | 1. Đăng nhập nhân viên hoặc Trưởng phòng. 2. Mở tờ trình ở bước ký duyệt. | Tài khoản không phải Lãnh đạo | Nút Ký số không hiển thị. Chỉ xem. | High |
| TC-UC19-006 | Negative: Xác nhận ký số với dữ liệu không hợp lệ | 1. Lãnh đạo nhấn Ký số. 2. Nhập thông tin không hợp lệ. 3. Nhấn Xác nhận. | Thông tin ký không hợp lệ hoặc trống | Hệ thống báo lỗi. Không thực hiện ký số. Người dùng cần cập nhật lại thông tin. | High |
| TC-UC19-007 | Edge case: Tài liệu đã ký không thể chỉnh sửa | 1. Lãnh đạo ký số thành công. 2. Bất kỳ ai cố chỉnh sửa nội dung tài liệu đã ký. | Tài liệu đã có chữ ký | Hệ thống không cho phép điều chỉnh tài liệu đã ký. | High |
| TC-UC19-008 | Edge case: Ý kiến Lãnh đạo lưu vào lịch sử xử lý | 1. Lãnh đạo ký số và nhập ý kiến. 2. Kiểm tra lịch sử. | Ý kiến: "Đã xem xét và ký duyệt" | Ý kiến Lãnh đạo xuất hiện trong lịch sử xử lý tờ trình. | Medium |
| TC-UC19-009 | Edge case: Trạng thái tờ trình sau ký số là "Đã phê duyệt" | 1. Lãnh đạo ký số thành công. 2. Kiểm tra trạng thái tại Tờ trình của tôi. | Ký số thành công | Trạng thái: "Đã phê duyệt". Không còn là "Chờ phê duyệt". | High |
| TC-UC19-010 | Edge case: Email ET2 gửi đúng người trình | 1. Lãnh đạo hoàn thành ký số. 2. Kiểm tra hộp thư người trình. | Người trình có email hợp lệ | Người trình nhận email ET2 với đầy đủ thông tin tờ trình. | High |
| TC-UC19-011 | Edge case: Ghi log khi ký số | 1. Lãnh đạo ký số thành công. 2. Kiểm tra log. | Ký số thành công | Log ghi: Hành động "Ký số", Ý kiến, Thời gian, Người xử lý. | Medium |
| TC-UC19-012 | Edge case: Người không có quyền không truy cập tờ trình đã ký | 1. Tờ trình trạng thái "Đã phê duyệt". 2. Đăng nhập tài khoản không liên quan. 3. Thử truy cập URL. | Tài khoản nhân viên không liên quan | Hệ thống từ chối truy cập. | Medium |

---

## Tổng hợp Open Questions cần PO xác nhận trước khi execute

| TC liên quan | OQ | Câu hỏi |
|---|---|---|
| TC-UC7-015 | OQ-01 | Ngày/Giờ họp "không bắt buộc" trong form nhưng logic kiểm tra trùng lịch cần thời gian — mâu thuẫn. BRD cần làm rõ. |
| TC-UC9-009, TC-UC9-010 | OQ-05 | Email hủy lịch: bảng trigger BRD ghi "người đặt", phần 6.7 ghi "người tham gia". Cần PO chọn 1 hành vi. |
| TC-UC10-009 | OQ-06 | Sửa nội dung không phải thời gian của phòng cần phê duyệt → có kích hoạt lại phê duyệt không? |
| TC-UC10-012 | OQ-10 | Email sửa lịch (BR-08): gửi cho người tham gia cũ, mới, hay cả hai? |
| TC-UC11-017 | OQ-08 | Xóa phòng có lịch ở trạng thái "Chờ phê duyệt" — có bị chặn không? |
| TC-UC18-006 | — | Tổ Thư ký 2 người — 1 Đồng ý + 1 Yêu cầu điều chỉnh: tờ trình xử lý như thế nào? Cần PO/Dev xác nhận. |
