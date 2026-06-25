# Customer 360 — Bản chốt thông tin hiển thị

**Dự án:** CDP cho VNPost/TCT  
**Nguồn tham chiếu chính:** `.claude/input/CDP/CDP.md`  
**Ngày tạo:** 2026-06-24  
**Mục đích:** Tập hợp toàn bộ thông tin liên quan đến Customer 360 từ nhiều chỗ rải rác trong tài liệu thành một bản duy nhất — làm cơ sở chốt trước khi vào wireframe

---

## 1. Customer 360 là gì trong hệ thống này?

Theo tài liệu (mục 5.3, dòng 307):

> "Hệ thống tạo hồ sơ khách hàng gồm thông tin định danh, giao dịch, hành vi số, COD, khiếu nại, lịch sử CSKH, phân khúc và điểm số."

Theo mục 4.1 (dòng 239):

> "Hợp nhất dữ liệu khách hàng từ nhiều hệ thống để tạo hồ sơ khách hàng duy nhất — phục vụ: Kinh doanh, CSKH, Marketing, đơn vị tỉnh/thành."

**Tóm lại:** Customer 360 là màn hình tra cứu hồ sơ khách hàng duy nhất — thay thế việc CSKH phải mở 5–7 hệ thống khác nhau để tìm thông tin cùng một khách.

---

## 2. Thông tin Customer 360 sẽ hiển thị — gom từ tài liệu

Tài liệu đề cập Customer 360 ở ít nhất **4 chỗ khác nhau** với cách gọi và mức độ chi tiết khác nhau. Bảng dưới gom lại và đối chiếu:

| Nhóm thông tin | Mục 3.5 (dòng 200) | Mục 6.2 (dòng 334) | FR-C360 (dòng 672) | **Sẽ hiển thị** | Ghi chú |
|---|---|---|---|---|---|
| **Thông tin định danh** | Họ tên, SĐT, email, PostID, mã KHL, MST, CCCD, User ID | Họ tên, SĐT, email, CCCD, mã KHL, MST, PostID, User ID | FR-C360-01 + FR-C360-02 | Họ tên · SĐT (masked) · Email (masked) · PostID · Mã KH · Mã KHL · MST · CCCD (masked) · User ID App · CRM ID · Trạng thái hoạt động · Bưu cục quản lý · Phân nhóm KH | Đồng nhất cả 3 chỗ |
| **Thông tin doanh nghiệp** | Loại KH, nhóm KH, trạng thái hợp đồng | Tên DN, MST, mã KHL, người đại diện, hợp đồng | Không có FR riêng | Tên doanh nghiệp · MST · Mã KHL · Người đại diện · Trạng thái hợp đồng · Nhóm KH | Chỉ hiển thị khi KH là doanh nghiệp; cần chốt layout riêng hay dùng chung template |
| **Địa chỉ** | Địa chỉ gửi/nhận, khu vực | Địa chỉ liên hệ, địa chỉ gửi, địa chỉ nhận, địa chỉ thường dùng, vùng phục vụ | Không có FR riêng | Địa chỉ liên hệ chính · Địa chỉ gửi thường dùng · Địa chỉ nhận thường dùng · Vùng phục vụ | Cần chốt: hiển thị tối đa bao nhiêu địa chỉ mỗi loại |
| **Lịch sử giao dịch** | Đơn hàng, mã vận đơn, loại dịch vụ, cước phí, sản lượng | Đơn hàng, vận đơn, loại dịch vụ, cước phí, tuyến gửi, doanh thu | FR-C360-03 + FR-C360-04 | **Danh sách đơn (FR-C360-03):** Mã vận đơn · Loại dịch vụ · Tuyến gửi · Cước phí · Trạng thái · Ngày tạo đơn — **Chi tiết một đơn (FR-C360-04, drill-down):** Timeline 7 bước từ tạo đơn → chuyển hoàn | FR-C360-04 là drill-down của FR-C360-03, không phải tab riêng; cần chốt số ngày/số đơn tối đa |
| **Lịch sử CSKH / Khiếu nại** | Kết quả phát, khiếu nại, đánh giá | Khiếu nại, phản ánh, đánh giá, lịch sử liên hệ, kết quả xử lý | FR-C360-07 | Loại khiếu nại · Kênh tiếp nhận · Ngày tạo · Trạng thái xử lý · Kết quả · SLA (đúng hạn / trễ hạn) | Cần chốt cách hiển thị SLA |
| **Loyalty** | Không đề cập | Hạng KH, điểm tích lũy, ưu đãi, lịch sử quyền lợi | FR-C360-08 — Medium | Hạng KH · Điểm tích lũy · Ưu đãi hiện có · Lịch sử sử dụng quyền lợi | Chỉ hiển thị nếu có dữ liệu loyalty; có thể chưa có data thực ở MVP |
| **Hành vi số / Timeline tương tác** | Mở app, đăng nhập, tạo đơn, tra cứu cước, click banner | Đăng nhập, mở app, tạo đơn, tra cứu cước, tracking, click banner | FR-C360-06 | Đăng nhập app · Tra cứu cước · Tìm bưu cục · Click banner/chiến dịch · Nhận SMS/Email/Zalo · Tương tác tại quầy — **Không gộp** sự kiện tạo đơn hoàn chỉnh (đã có ở FR-C360-03) | Cần liệt kê đủ loại sự kiện trước khi wireframe; tránh trùng với FR-C360-03 |
| **COD và thanh toán** | COD amount, trạng thái thu/trả, lịch sử thanh toán, đối soát | Số tiền COD, trạng thái thu/trả, đối soát, lịch sử TT, tài khoản nhận | FR-C360-05 | Tổng COD phát sinh · Tổng COD đã thu · Tổng COD chưa thu · Trạng thái đối soát · Lịch sử thanh toán theo thời gian | FR-C360-05 là **tổng hợp COD toàn bộ** — không phải chi tiết từng đơn (đã có trong FR-C360-03) |
| **Phân khúc và điểm số** | Không đề cập | Segment, RFM, CLV, churn score, COD risk score, fraud score | FR-C360-09 | Phân khúc hiện tại · RFM (R/F/M riêng) · CLV · Churn score · Engagement score · COD risk score · Fraud score | FR-C360-15 là layer tính toán backend, kết quả đẩy vào đây hiển thị — không phải section UI riêng; cần chốt scale điểm và phân quyền xem |
| **Consent** | Opt-in/opt-out | Opt-in/opt-out, mục đích, kênh, thời điểm, trạng thái kích hoạt | FR-C360-10 | Trạng thái theo kênh: SMS · Email · Zalo · Push Notification — mỗi kênh ghi rõ: opt-in/opt-out · Thời điểm ghi nhận · Nguồn consent | Quan trọng về pháp lý (Luật 91/2025/QH15); cần chốt layout bảng hay toggle |

---

## 3. Danh sách đầy đủ 15 chức năng FR-C360

Nguồn: mục 7.5, dòng 672–688

| Mã | Tên chức năng | Độ ưu tiên | Mô tả tóm tắt |
|---|---|---|---|
| FR-C360-01 | Unified Profile Dashboard | **High** | Header hồ sơ: thông tin cá nhân, liên hệ, bưu cục quản lý, phân nhóm, trạng thái hoạt động |
| FR-C360-02 | Customer Identity Panel | **High** | Các định danh liên quan: SĐT, email, PostID, CRM ID, mã KHL, mã KH, User ID app, MST |
| FR-C360-03 | Transaction History | **High** | Lịch sử đơn hàng, vận đơn, dịch vụ, cước phí, sản lượng, doanh thu, tuyến gửi |
| FR-C360-04 | Shipment Timeline | **High** | Timeline bưu gửi: tạo đơn → thu gom → chấp nhận gửi → khai thác → vận chuyển → phát → chuyển hoàn |
| FR-C360-05 | COD & Payment History | **High** | COD amount, trạng thái thu/trả, đối soát, lịch sử thanh toán — theo phân quyền |
| FR-C360-06 | Multi-channel Timeline Stream | **High** | Timeline sự kiện theo thời gian thực: online (tra cứu cước, tạo đơn trên app) + trực tiếp tại quầy |
| FR-C360-07 | Complaint & Case History | **High** | Lịch sử khiếu nại, phản ánh, đánh giá dịch vụ, kết quả xử lý, SLA |
| FR-C360-08 | Loyalty Information | Medium | Hạng KH, điểm tích lũy, ưu đãi, lịch sử quyền lợi — nếu có dữ liệu loyalty |
| FR-C360-09 | Segment & Score Display | **High** | Phân khúc, RFM, CLV, churn score, engagement score, COD risk score, fraud score — theo phân quyền |
| FR-C360-10 | Consent Status Display | **High** | Trạng thái opt-in/opt-out theo từng mục đích/kênh, thời điểm, nguồn consent |
| FR-C360-11 | Role-based Masking | **High** | Che dữ liệu nhạy cảm (SĐT, email, CCCD, tài khoản TT) theo vai trò — xuyên suốt tất cả nhóm |
| FR-C360-12 | Customer Search | **High** | Tìm theo: SĐT, email, mã KH, mã KHL, PostID, mã vận đơn, MST |
| FR-C360-13 | Data Source Traceability | Medium | Xem nguồn phát sinh của từng nhóm dữ liệu trong hồ sơ |
| FR-C360-14 | Customer Note & Tagging | Medium | Ghi chú, gắn tag, đánh dấu KH cần chăm sóc đặc biệt |
| FR-C360-15 | Computed Attributes | Medium | Tự động tính: điểm tương tác số hóa, CLV, tỷ lệ sụt giảm sản lượng, tỷ lệ hoàn |

---

## 4. Nguồn dữ liệu cho từng nhóm thông tin

Nguồn: mục 6.2 (dòng 334), mục 6.10 (dòng 540), mục 3.5 (dòng 200)

| Nhóm thông tin | Hệ thống nguồn chính | Nguồn ưu tiên khi xung đột |
|---|---|---|
| Thông tin định danh | CRM, CAS, Portal KHL, MyVNPost, PostID | PostID/MyVNPost (đã xác thực) > CRM > CAS |
| Số điện thoại | PostID/MyVNPost, CRM, CAS, Portal KHL | PostID/MyVNPost (xác thực) → lưu SĐT khác làm alias |
| Email | CRM, Portal KHL, MyVNPost | Email xác thực hoặc email hợp đồng/DN |
| Tên cá nhân | PostID, CRM, MyVNPost, CAS | PostID → CRM → nguồn có xác thực hoặc cập nhật gần nhất |
| Thông tin doanh nghiệp | Portal KHL, CRM, hợp đồng | Tên theo MST/hợp đồng |
| Địa chỉ | VPostCode/Vmap (sau chuẩn hóa), CAS, Portal KHL, MyVNPost | Địa chỉ đã chuẩn hóa và giao dịch gần nhất |
| Lịch sử giao dịch | CAS, MPITS, Portal KHL, BCCP, MyVNPost | *(nhiều nguồn, mapping về bộ trạng thái chuẩn)* |
| Trạng thái phát | PNS/DingDong, BCCP, MPITS | PNS/DingDong (cập nhật kết quả phát trực tiếp) |
| COD và thanh toán | PayPost, ngân hàng | **PayPost** (nguồn ưu tiên tuyệt đối cho trạng thái COD) |
| Khiếu nại / CSKH | CRM, Care Đơn, Call Center | CRM/Care Đơn (hệ thống CSKH chính thức) |
| Phân khúc / Điểm số | CDP Analytics & AI | CDP tự tính — không lấy từ hệ thống nguồn nào |
| Consent | CDP Consent Store, CRM, app/web | Trạng thái consent **mới nhất** có bằng chứng ghi nhận |

---

## 5. Quy tắc chuẩn hóa dữ liệu trước khi hiển thị

Nguồn: mục 6.5 (dòng 383)

| Trường dữ liệu | Quy tắc chuẩn hóa | Ví dụ |
|---|---|---|
| Số điện thoại | Loại bỏ khoảng trắng, dấu chấm, gạch ngang; chuẩn về format 10 số | `+84988xxxxxx` → `0988xxxxxx` |
| Email | Chuyển về chữ thường, loại khoảng trắng | `KHACHHANG@EMAIL.COM` → `khachhang@email.com` |
| Tên khách hàng | Xóa khoảng trắng thừa, chuẩn hóa viết hoa, xử lý dấu tiếng Việt | `"nguyen van a"` → `"Nguyễn Văn A"` |
| Địa chỉ | Bóc tách 4 cấp hành chính, ánh xạ VPostCode/Vmap | `"P. Dịch Vọng, CG, HN"` → `"Phường Dịch Vọng, Quận Cầu Giấy, Hà Nội"` |
| Mã số thuế | Kiểm tra cấu trúc 10 hoặc 13 số | Loại bỏ ký tự không hợp lệ |
| CCCD | Masking khi hiển thị, mã hóa khi lưu trữ | `0123**********` |
| Mã vận đơn | Chuẩn hóa chữ hoa, loại ký tự thừa | `" vn123456 "` → `"VN123456"` |
| Trạng thái giao dịch | Mapping từ nhiều bộ trạng thái về bộ chuẩn | `"Delivered"`, `"Phát TC"`, `"Thành công"` → `"Phát thành công"` |
| Trạng thái COD | Chuẩn hóa về bộ trạng thái đối soát | `"Đã thu"`, `"Thu COD thành công"` → `"COD đã thu"` |

---

## 6. Phân quyền hiển thị (Role-based Masking — FR-C360-11)

Nguồn: mục 6.11 (dòng 555), FR-C360-11 (dòng 684), FR-GOV-01 (dòng 745)

| Nhóm dữ liệu nhạy cảm | Rủi ro nếu lộ | Cách kiểm soát |
|---|---|---|
| SĐT, email | Lộ thông tin liên hệ | Masking theo vai trò: `0912***444` |
| CCCD | Lộ dữ liệu định danh nhạy cảm | Mã hóa + masking + audit log bắt buộc |
| Địa chỉ / GPS | Lộ vị trí cá nhân | Phân quyền theo mục đích; kiểm soát export |
| COD / tài khoản thanh toán | Lộ dữ liệu tài chính | Mã hóa + phân quyền nghiêm ngặt + audit log |
| Hành vi app/web | Dùng sai mục đích | Gắn consent theo mục đích; chỉ activation khi được phép |
| Phân khúc / Điểm số | Lạm dụng điểm rủi ro | Kiểm soát quyền xem theo vai trò (CSKH ≠ Marketing ≠ Kinh doanh) |

**Nguyên tắc:** Cùng một màn hình Customer 360, nhưng mỗi vai trò thấy mức độ chi tiết khác nhau — không chỉ ẩn/hiện tab mà còn ẩn nội dung bên trong từng trường.

---

## 7. Điểm còn thiếu — BA cần chốt trước khi wireframe

| # | Điểm cần chốt | Nhóm ảnh hưởng | Mức độ chặn wireframe |
|---|---|---|---|
| 1 | **Layout KH cá nhân vs doanh nghiệp khác nhau không?** Hay dùng chung một template rồi ẩn/hiện section? | Nhóm 1 + 2 | **Chặn** — quyết định cấu trúc layout chính |
| 2 | **Địa chỉ nào hiển thị?** Địa chỉ liên hệ / gửi thường dùng / nhận thường dùng — hiển thị bao nhiêu cái, ưu tiên cái nào | Nhóm 3 | **Chặn** — không biết cần mấy dòng địa chỉ trong header |
| 3 | **Lịch sử giao dịch:** hiển thị bao nhiêu ngày gần nhất? Bao nhiêu đơn tối đa trên màn hình? Có filter không? | Nhóm 4 | Không chặn ngay, nhưng cần chốt trước khi spec bảng |
| 4 | **Timeline tương tác:** liệt kê đủ loại sự kiện cần gộp vào — tài liệu chỉ nói "tạo đơn trên app + hoạt động tại quầy" | Nhóm 5 | **Chặn** — không biết timeline gồm những sự kiện gì |
| 5 | **Scale điểm số hiển thị:** 0–100 hay thấp/trung/cao? Dạng số hay badge màu? | Nhóm 8 | Không chặn ngay nhưng ảnh hưởng component design |
| 6 | **Consent hiển thị dạng nào:** bảng theo kênh (SMS/Email/Zalo/Push) hay toggle? Ai được xem? | Nhóm 9 | Không chặn ngay |
| 7 | **SLA khiếu nại hiển thị thế nào:** số giờ xử lý, hay màu cảnh báo (đúng hạn/trễ hạn)? | Nhóm 7 | Không chặn ngay |
| 8 | **Trường bắt buộc vs tùy chọn trong header:** khi KH không có email hoặc không có PostID thì hiển thị gì? Để trống hay ẩn dòng? | Nhóm 1 | **Chặn** — ảnh hưởng xử lý trạng thái rỗng của mỗi trường |

---

## 8. Khuyến nghị thứ tự chốt

**Chốt ngay (chặn wireframe):**
1. Layout cá nhân vs doanh nghiệp → cùng hay khác template
2. Địa chỉ hiển thị loại nào, bao nhiêu cái
3. Loại sự kiện trong multi-channel timeline
4. Cách xử lý trường rỗng trong header

**Chốt sau (không chặn ngay nhưng cần trước khi spec component):**
5. Lịch sử giao dịch: số ngày, số đơn tối đa
6. Scale điểm số
7. Layout consent theo kênh
8. Hiển thị SLA khiếu nại

**Đã đủ để wireframe ngay (không cần chốt thêm):**
- Nhóm định danh (header, identity panel)
- Nhóm COD/thanh toán
- Nhóm khiếu nại (ngoại trừ SLA)
- Nhóm consent (ngoại trừ layout chi tiết)
