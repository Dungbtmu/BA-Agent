# Customer 360 — Bản đánh giá đầy đủ

**Dự án:** CDP cho VNPost/TCT  
**Nguồn tham chiếu:** `.claude/input/CDP/CDP.md`  
**Ngày tạo:** 2026-06-24  
**Mục đích:** Tổng hợp 9 nhóm thông tin Customer 360 — vị trí trong tài liệu, mức độ đầy đủ và việc BA cần làm trước khi wireframe

---

## Phần 1 — 9 nhóm thông tin và vị trí trong tài liệu

| # | Nhóm thông tin | Xuất hiện ở | Có spec FR-C360 | Mức đầy đủ |
|---|---|---|---|---|
| 1 | **Thông tin định danh** | Mục 3.5, 6.2, 6.3, 6.4, 6.5, 7.5 — 6 chỗ | FR-C360-01, FR-C360-02 | Đầy đủ nhất |
| 2 | **Thông tin doanh nghiệp** | Mục 3.5, 6.2, 6.3, 6.9, 7.11 — 5 chỗ | Không có | Thiếu spec màn hình |
| 3 | **Địa chỉ** | Mục 3.4, 3.5, 6.2, 6.3, 6.5, 6.9, UC-11, FR-DPS-04 — 8 chỗ | Không có trong FR-C360 | Đủ logic, thiếu UI |
| 4 | **Lịch sử giao dịch** | Mục 3.4, 3.5, 6.2, 6.3, 6.5, 6.10 — 6 chỗ | FR-C360-03, FR-C360-04 | Tương đối đủ |
| 5 | **Timeline tương tác đa kênh** | Mục 3.5 (x2), 6.2, 6.3, 7.5 — 5 chỗ | FR-C360-06 | Mỏng, thiếu chi tiết |
| 6 | **COD và thanh toán** | Mục 3.4, 3.5, 6.2, 6.5, 6.9, 6.10 — 6 chỗ | FR-C360-05 | Đủ nhất trong nhóm |
| 7 | **Khiếu nại / CSKH** | Mục 3.4, 3.5, 6.2, UC-12 — 4 chỗ | FR-C360-07 | Có spec nhưng ngắn |
| 8 | **Phân khúc và điểm số** | Mục 6.2, 6.3, UC-02/08/09, FR-ANA-01–15 | FR-C360-09 (1 dòng) | Logic đủ, UI rất ngắn |
| 9 | **Consent** | Mục 6.2, 6.3, 6.10, UC-10, Chương 8 — 7 chỗ | FR-C360-10 | Đủ về pháp lý, UI ngắn |

---

## Phần 2 — 15 chức năng FR-C360 trong tài liệu

| Mã | Tên chức năng | Độ ưu tiên | Nhóm thông tin |
|---|---|---|---|
| FR-C360-01 | Bảng hồ sơ hợp nhất (Unified Profile Dashboard) | High | Định danh |
| FR-C360-02 | Khung định danh (Customer Identity Panel) | High | Định danh |
| FR-C360-03 | Lịch sử giao dịch (Transaction History) | High | Giao dịch |
| FR-C360-04 | Dòng thời gian bưu gửi (Shipment Timeline) | High | Giao dịch |
| FR-C360-05 | Lịch sử COD và thanh toán (COD & Payment History) | High | COD/Thanh toán |
| FR-C360-06 | Timeline tương tác đa kênh (Multi-channel Timeline Stream) | High | Tương tác |
| FR-C360-07 | Lịch sử khiếu nại (Complaint & Case History) | High | Khiếu nại/CSKH |
| FR-C360-08 | Thông tin loyalty (Loyalty Information) | Medium | *(chưa có nhóm riêng)* |
| FR-C360-09 | Phân khúc và điểm số (Segment & Score Display) | High | Phân khúc/Điểm số |
| FR-C360-10 | Trạng thái consent (Consent Status Display) | High | Consent |
| FR-C360-11 | Che dữ liệu theo vai trò (Role-based Masking) | High | *(xuyên suốt mọi nhóm)* |
| FR-C360-12 | Tìm kiếm khách hàng (Customer Search) | High | *(entrypoint)* |
| FR-C360-13 | Truy vết nguồn dữ liệu (Data Source Traceability) | Medium | *(xuyên suốt)* |
| FR-C360-14 | Ghi chú và gắn nhãn (Customer Note & Tagging) | Medium | *(thao tác nghiệp vụ)* |
| FR-C360-15 | Tính toán thuộc tính phái sinh (Computed Attributes) | Medium | Phân khúc/Điểm số |

---

## Phần 3 — Đánh giá từng nhóm: đủ hay BA cần bổ sung

### Nhóm 1 — Thông tin định danh

- **Tài liệu có:** họ tên, SĐT, email, PostID, CRM ID, mã KHL, MST, CCCD, User ID App — nguồn từ 12 loại định danh (mục 6.4)
- **Chuẩn hóa:** SĐT format 0988xxxxxx, email lowercase, tên bỏ dấu cách thừa
- **Spec chức năng:** FR-C360-01 (dashboard tổng) + FR-C360-02 (identity panel) đã có
- **BA cần làm:** Chốt trường nào bắt buộc hiển thị vs trường nào chỉ hiển thị khi có dữ liệu

### Nhóm 2 — Thông tin doanh nghiệp

- **Tài liệu có:** tên DN, MST, mã KHL, người đại diện, hợp đồng, mô hình Organization–Contact (mục 6.9 case 9)
- **Thiếu:** không có FR-C360 nào xử lý riêng cho KH doanh nghiệp
- **BA cần làm:** Xác định KH cá nhân và KH doanh nghiệp hiển thị tab/section khác nhau hay dùng chung layout

### Nhóm 3 — Địa chỉ

- **Tài liệu có:** chuẩn hóa 4 cấp hành chính, ánh xạ VPostCode/Vmap, xử lý địa chỉ sai (mục 6.5, 6.9 case 12)
- **Nguồn:** PNS/DingDong, VPostCode, Vmap, TMS, CAS
- **Thiếu:** không có FR-C360 nào quy định hiển thị địa chỉ trên màn hình C360
- **BA cần làm:** Xác định loại địa chỉ nào hiển thị (địa chỉ liên hệ / địa chỉ gửi thường dùng / địa chỉ nhận thường dùng) và số lượng tối đa hiển thị

### Nhóm 4 — Lịch sử giao dịch

- **Tài liệu có:** luồng 9 bước hành trình từ tạo đơn đến CSKH (mục 3.4 rất chi tiết), FR-C360-03 (transaction history) + FR-C360-04 (shipment timeline)
- **Chuẩn hóa:** mapping trạng thái giao dịch về bộ chuẩn
- **BA cần làm:** Chốt hiển thị bao nhiêu ngày gần nhất, bao nhiêu đơn tối đa, có phân trang không

### Nhóm 5 — Timeline tương tác đa kênh

- **Tài liệu có:** FR-C360-06 ("gộp hoạt động online và trực tiếp tại quầy theo thời gian thực") — chỉ 1 dòng mô tả
- **BA cần làm:** Liệt kê đủ loại sự kiện cần gộp (tạo đơn trên app, tra cứu cước, click banner, nhận SMS, vào quầy, gọi CSKH...) và thứ tự sắp xếp timeline

### Nhóm 6 — COD và thanh toán

- **Tài liệu có:** FR-C360-05 (COD amount, trạng thái thu/trả, đối soát), nguồn ưu tiên PayPost, xử lý chênh lệch PayPost vs PNS/DingDong (mục 6.9 case 15, 6.10)
- **BA cần làm:** Xác nhận phạm vi hiển thị — chỉ COD hay cả thanh toán ví/online

### Nhóm 7 — Khiếu nại / CSKH

- **Tài liệu có:** FR-C360-07 (khiếu nại, phản ánh, đánh giá, kết quả, SLA), UC-12 (phân tích chất lượng dịch vụ)
- **BA cần làm:** Định nghĩa "SLA xử lý" hiển thị dưới dạng nào (số giờ, màu cảnh báo, đúng/trễ hạn)

### Nhóm 8 — Phân khúc và điểm số

- **Tài liệu có:** 15 chức năng phân tích FR-ANA-01 đến FR-ANA-15, nhưng phần hiển thị trên C360 chỉ có FR-C360-09 (1 dòng: "RFM, CLV, churn score, engagement score, COD risk score, fraud score")
- **BA cần làm:** Xác định scale điểm (0–100 hay thấp/trung/cao), cách hiển thị (badge, thanh tiến trình, số), ai được xem điểm nào (CSKH vs Marketing vs Kinh doanh)

### Nhóm 9 — Consent

- **Tài liệu có:** Chương 8 mục 8.10 + 8.12, FR-C360-10, FR-GOV-02, FR-GOV-16 — đầy đủ nhất về pháp lý (Luật Bảo vệ Dữ liệu Cá nhân số 91/2025/QH15, có hiệu lực từ 1/1/2026)
- **BA cần làm:** Xác định layout hiển thị consent theo từng kênh (SMS / Email / Zalo / Push) — dạng bảng hay dạng toggle

---

## Phần 4 — Tóm tắt việc BA cần làm để chốt C360

| Việc cần chốt | Nhóm liên quan | Mức độ tác động |
|---|---|---|
| Trường bắt buộc vs tùy chọn trong header C360 | Nhóm 1 | Cao — ảnh hưởng layout chính |
| Layout riêng cho KH cá nhân vs doanh nghiệp | Nhóm 2 | Cao — hai loại KH rất khác nhau |
| Địa chỉ nào hiển thị, tối đa bao nhiêu | Nhóm 3 | Trung bình |
| Giao dịch hiển thị bao nhiêu ngày, bao nhiêu đơn | Nhóm 4 | Trung bình |
| Liệt kê đủ loại sự kiện trong timeline | Nhóm 5 | Cao — hiện tài liệu chưa có |
| Phạm vi COD vs thanh toán online | Nhóm 6 | Thấp |
| Cách hiển thị SLA khiếu nại | Nhóm 7 | Thấp |
| Scale điểm số + phân quyền xem | Nhóm 8 | Cao — hiện tài liệu rất thiếu |
| Layout consent theo kênh | Nhóm 9 | Trung bình |

**Ưu tiên cho MVP:**
- Nhóm 1 (định danh), Nhóm 4 (giao dịch), Nhóm 6 (COD), Nhóm 9 (consent) — đã đủ nền để làm wireframe
- Nhóm 2, Nhóm 5, Nhóm 8 — BA cần bổ sung trước khi wireframe
