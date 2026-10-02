# Clarification — Change Request: Mô hình Độ ưu tiên Campaign (Cross-campaign Priority)

**Nguồn:** Feedback PO/BA (Jun) sau demo nghiệm thu L2
**Phạm vi ảnh hưởng:** II.6.8 (Cross-campaign Priority), UC-CAM-01 (Danh sách), UC-CAM-02 (Tạo), UC-CAM-03 (Sửa Draft), UC-CAM-05 (Duyệt), UC-CAM-07 (Bật/Tắt), UC-PRIORITY-01 (Priority Matrix), Screen 2 (Campaign List), Screen 3 (Campaign Builder — Section 1 + Kênh & Lịch gửi), Screen Settings Tab 3
**Trạng thái:** ĐÃ CHỐT — sẵn sàng chuyển sang `ba-solution-agent`

---

## 1. Tóm tắt vấn đề (Problem Statement)

Cơ chế Cross-campaign Priority hiện tại (URD V4.17) có 3 hạn chế PO chỉ ra sau demo nghiệm thu L2:

1. Cảnh báo trùng ưu tiên chưa đủ — chỉ chặn/cảnh báo khi CẢ HAI campaign đều Active; Paused trùng số với Active thì không cảnh báo.
2. Mô hình nhập priority rời rạc (số nguyên 1–9999, không neo trigger) không còn phù hợp — nên dồn hết về Cài đặt, dựa trên nhóm campaign trùng trigger thay vì priority number tự do.
3. Thiếu khái niệm phân loại chiến dịch "không cần setup Độ ưu tiên / Lịch gửi cụ thể" — URD hiện chưa có khái niệm này.

Đây là thay đổi **kiến trúc nghiệp vụ** của cả cụm Priority — ảnh hưởng UI (bỏ field khỏi 2 màn, thêm nghiệp vụ mới ở Cài đặt), logic chặn trùng (đổi từ so toàn cục sang so trong nhóm trigger), và vòng đời campaign (UC-CAM-05/07).

---

## 2. Cơ chế HIỆN TẠI (tham chiếu, không đổi trừ khi ghi rõ ở mục 3)

- Priority là số nguyên toàn cục (1–9999), không gắn trigger; trùng số chỉ chặn khi cả hai đều Active.
- Chặn tại 3 nơi: Campaign List inline-edit (UC-CAM-01), Campaign Builder Section 1 STT 5 (UC-CAM-02), và tại thời điểm Duyệt (UC-CAM-05, patch V4.15).
- Draft/Pending/Ended không tham gia so trùng — chỉ Active giữ "vị trí xếp hạng thật".
- Priority Matrix (UC-PRIORITY-01 / Settings Tab 3) hiện là bảng phẳng liệt kê toàn bộ campaign Active, không nhóm theo trigger.
- Campaign mới mặc định priority = max score Active hiện có + 1 (tự xếp cuối).

---

## 3. QUYẾT ĐỊNH ĐÃ CHỐT

### 3.1. Tên nghiệp vụ phân loại chiến dịch (thay thế OQ-2)

Hai loại chiến dịch, phân biệt tường minh bằng lựa chọn khi Tạo campaign:

- **"Chiến dịch vận hành thường trực"** — chạy liên tục theo sự kiện, không giới hạn thời điểm, áp dụng cho mọi đối tượng thỏa điều kiện bất kỳ lúc nào. Ví dụ: campaign "Kích hoạt sim" — bất cứ khi nào có sim đủ điều kiện là gửi, không phân biệt thời gian.
- **"Chiến dịch có thời hạn"** — chạy trong 1 khung thời gian xác định, phục vụ mục đích marketing ngắn hạn/theo đợt. Ví dụ: campaign "Chào mừng đại lễ" — chỉ áp dụng trong vài ngày lễ, cần setup thời gian và độ ưu tiên.

Lựa chọn loại hình diễn ra **tường minh, chọn lúc Tạo** (không suy ra ngầm từ field khác).

### 3.2. Phạm vi ẩn/khóa field theo loại hình (thay thế phần còn lại của OQ-2)

Campaign Builder (trigger, phân khúc, ma trận kênh tin nhắn) **giữ nguyên cho cả 2 loại** — không ẩn.

Trong nhóm "Kênh & Lịch gửi", chỉ ẩn/khóa đúng 1 field:

| Field | Vận hành thường trực | Có thời hạn |
|---|---|---|
| Kênh gửi | Hiển thị, cấu hình bình thường | Hiển thị, cấu hình bình thường |
| Ngày bắt đầu | Hiển thị, bắt buộc | Hiển thị, bắt buộc |
| **Ngày kết thúc** | **Ẩn/khóa — tự động = Vô hạn** | **Hiển thị, bắt buộc nhập** |
| Khung giờ gửi trong ngày | Hiển thị, cấu hình bình thường | Hiển thị, cấu hình bình thường |
| Tần suất nhắc lại (Re-engagement) | Hiển thị, tùy chọn (không bắt buộc) | Hiển thị, cấu hình bình thường |

Field "Độ ưu tiên" bị bỏ khỏi Campaign Builder cho **cả 2 loại** (dồn hết về Cài đặt — xem mục 3.3), không phải riêng cho loại Vận hành thường trực.

Ràng buộc: "Độ ưu tiên" và "Ngày kết thúc" đi cùng nhau về mặt ý nghĩa nghiệp vụ — campaign Vận hành thường trực (không có Ngày kết thúc) sẽ không tham gia xếp hạng thủ công trong nhóm ưu tiên (xem mục 3.4).

### 3.3. Cơ chế nhóm theo trigger trùng (thay thế OQ-3)

- **Mức khớp**: 2 campaign chỉ cần **trùng ít nhất 1 trigger** là được xếp vào cùng 1 nhóm ưu tiên — không xét logic AND/OR nội bộ của từng campaign (vì AND/OR là cấu hình riêng để tự kích hoạt, không ảnh hưởng đến việc 2 campaign có khả năng cùng được đánh giá tại 1 thời điểm hay không).
- **Phạm vi tham gia nhóm**: chỉ campaign **Active** và thuộc loại **"Có thời hạn"** mới được liệt kê và cho sắp xếp thứ tự trong nhóm.
- **Cách hiển thị tại Cài đặt**: nhóm campaign theo trigger trùng nhau, mỗi nhóm hiển thị dạng bảng/danh sách các campaign đang dùng chung trigger đó; có filter lọc theo trigger. Admin sắp xếp thứ tự bằng kéo-thả.
- **Quyền sửa**: chỉ **Admin** được sắp xếp/quyết định thứ tự ưu tiên — không cho QTV Marketing chỉnh.
- **Đồng bộ khi cấu hình trigger thay đổi**: **tự động** — ngay khi 1 campaign (loại Có thời hạn) đổi trigger dẫn đến trùng mới hoặc hết trùng với campaign khác, hệ thống tự động thêm/gỡ khỏi nhóm tương ứng, không cần Admin xác nhận thủ công. Kèm 2 quy tắc bổ sung để đảm bảo dữ liệu ổn định:
  - Campaign vừa được thêm vào nhóm (do đổi trigger hoặc do mới Active) → luôn tự động xếp **cuối nhóm**, không chèn giữa.
  - Campaign bị gỡ khỏi nhóm (do đổi trigger không còn trùng) → các thành viên còn lại **giữ nguyên thứ tự tương đối**, không dồn số.
  - Đề xuất bổ sung UX (không bắt buộc, để BA quyết định khi thiết kế chi tiết): đánh dấu nhẹ (badge) cho campaign vừa được tự động thêm vào nhóm, để Admin biết cần rà soát nếu muốn — không chặn xử lý.
- **Campaign mới Active**: mặc định hiển thị ở **vị trí cuối cùng** trong nhóm trigger tương ứng. Admin phải chủ động vào Cài đặt thao tác nếu muốn thay đổi thứ tự.
- **Campaign loại "Vận hành thường trực"**: **không xuất hiện trong danh sách kéo-thả xếp hạng** dù có trùng trigger với nhóm nào — chỉ hiển thị dạng ghi chú/tham chiếu thông tin (minh bạch cho Admin biết "còn có campaign vận hành thường trực X cũng dùng trigger này"), không tham gia sắp xếp thủ công.

### 3.4. Áp dụng thứ tự khi xử lý event thực tế (bổ sung, liên quan 3.3)

- Nếu 1 khách hàng đồng thời thỏa điều kiện của campaign "Có thời hạn" và campaign "Vận hành thường trực" cùng trùng trigger: **campaign Có thời hạn luôn được ưu tiên xử lý trước**, mặc định, không cần Admin cấu hình gì thêm.
- Trong nhóm chỉ gồm các campaign "Có thời hạn": xử lý theo đúng thứ tự Admin đã sắp xếp tại Cài đặt.

### 3.5. Validate trùng tại bước Duyệt — UC-CAM-05 (thay thế OQ-4)

- Bước Duyệt (UC-CAM-05) **hoàn toàn tách rời** khỏi validate độ ưu tiên — không còn kiểm tra trùng priority tại đây.
- Sau khi Duyệt xong và campaign vào trạng thái Active: nếu nó rơi vào 1 nhóm trigger đã có campaign khác, hệ thống tự động xếp nó vào **cuối nhóm** (theo rule 3.3) — không cần bước xác nhận nào ở Duyệt.
- Trong lúc Admin chưa vào sắp xếp lại: campaign xử lý event bình thường theo đúng vị trí hiện tại nó đang đứng (vị trí cuối nhóm) — **không bị chặn**, không có trạng thái "chờ xếp hạng" đặc biệt nào.

### 3.6. Cảnh báo Tạm dừng × Active (thay thế OQ-1)

Feedback gốc ("Trùng độ ưu tiên, với camp đang tạm dừng - trùng với 1 th Đang hoạt động => show cảnh báo") được xác nhận là **ghi nhận bug của cơ chế cũ** (priority number rời rạc), không phải yêu cầu thiết kế mới cần giữ lại.

Với mô hình mới (mục 3.3): nhóm ưu tiên chỉ liệt kê campaign **Active** — campaign Tạm dừng không thuộc bất kỳ nhóm nào trong lúc tạm dừng, nên không tồn tại trạng thái "trùng" giữa Paused và Active để cảnh báo. Khi Paused được Kích hoạt lại (UC-CAM-07), nó quay lại nhóm và tự động xếp cuối theo đúng rule 3.3 — không phát sinh xung đột cần chặn ở bước Bật lại.

→ **Kết luận: không cần thiết kế cơ chế cảnh báo Tạm dừng × Active** — vấn đề tự nhiên không còn tồn tại trong mô hình mới. Không đưa vào scope solution.

---

## 4. Risk cần lưu ý khi thiết kế Solution (chuyển tiếp cho `ba-solution-agent`)

- **Risk breaking change**: thay đổi đồng thời 5+ vị trí URD (II.6.8, UC-CAM-01/02/03/05/07, UC-PRIORITY-01) — bắt buộc impact-analysis qua Traceability Map trước khi patch URD, tránh sót chỗ.
- **Risk UX chưa chốt — để `ba-solution-agent` chủ động đề xuất phương án, trình Jun duyệt ở bước solution**: QTV Marketing (người tạo/sửa campaign) có mất hoàn toàn khả năng xem thứ tự ưu tiên hiện tại của campaign mình quản lý không, hay cần 1 kênh xem read-only?
- **Risk dữ liệu cũ**: campaign Active hiện có đang mang priority number kiểu cũ (1–9999, không gắn trigger) — cần phương án migrate sang cơ chế nhóm mới (ví dụ: giữ nguyên thứ tự tương đối hiện tại, ánh xạ vào đúng nhóm trigger tương ứng làm điểm khởi đầu). Nêu trong Assumption/Migration Note của Solution doc.
- **Đã chốt**: 1 campaign "Có thời hạn" trùng đồng thời nhiều trigger khác nhau (thuộc nhiều nhóm khác nhau) → thứ tự của nó **độc lập theo từng nhóm** — mỗi nhóm là 1 bối cảnh tranh chấp riêng theo đúng trigger đó, không dùng chung 1 con số thứ tự cho tất cả các nhóm. `ba-solution-agent` thiết kế mô hình dữ liệu nghiệp vụ theo hướng: 1 campaign có thể có N vị trí ưu tiên, mỗi vị trí gắn với đúng 1 nhóm trigger.
- **Đã chốt**: nhiều campaign "Vận hành thường trực" cùng trùng 1 trigger (không có campaign Có thời hạn nào trong nhóm) → xử lý theo **`created_at` sớm hơn thắng** (tái dùng cơ chế tiebreak đã có ở II.6.8 cũ). Nhóm này **không** xuất hiện trong bàn kéo-thả ở Cài đặt — hoàn toàn tự động, Admin không thao tác.
- **Đã xác nhận không xung đột thuật ngữ**: grep toàn bộ URD V4.17 xác nhận cụm "thường trực/dài hạn/ngắn hạn" chưa được dùng làm thuật ngữ nghiệp vụ nào khác — tên "Chiến dịch vận hành thường trực" / "Chiến dịch có thời hạn" an toàn để dùng xuyên suốt.

---

## 5. Bước tiếp theo

Chuyển sang `ba-solution-agent` để thiết kế:
- Flow "Độ ưu tiên" mới tại Cài đặt (nhóm theo trigger, kéo-thả, filter theo trigger, xử lý tự động thêm/gỡ nhóm).
- Flow Campaign Builder rút gọn: bỏ field Độ ưu tiên; thêm lựa chọn loại hình chiến dịch (Vận hành thường trực / Có thời hạn) — ảnh hưởng đến field Ngày kết thúc.
- Flow Duyệt (UC-CAM-05) sau khi tách hoàn toàn validate ưu tiên ra khỏi bước này.
- Phương án hiển thị read-only cho QTV Marketing (risk UX ở mục 4).
- Phương án migrate dữ liệu priority cũ.
