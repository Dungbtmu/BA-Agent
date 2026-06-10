# Apache Unomi — CDP VNPost có thể học gì?

**Ngày:** 2026-06-10  
**Mục đích:** Tìm hiểu Unomi để đánh giá CDP VNPost nên học hỏi và phát triển tính năng gì

---

## 1. Unomi là gì

Unomi là phần mềm CDP mã nguồn mở của tổ chức Apache — giúp gộp dữ liệu khách hàng từ nhiều hệ thống về một hồ sơ duy nhất, tự động phân loại khách hàng theo nhóm, và kích hoạt hành động tự động khi có sự kiện xảy ra. VNPost **không cần dùng Unomi trực tiếp** — nhưng cách Unomi thiết kế các tính năng là nguồn tham khảo giá trị để xây CDP riêng.

---

## 2. Tính năng Unomi có giá trị tham khảo cho CDP VNPost

### 2.1 Gộp nhiều ID của cùng một khách hàng về một hồ sơ

**Bài toán Unomi giải:** Khi cùng một người xuất hiện ở nhiều hệ thống với mã định danh khác nhau, Unomi tạo "cầu nối alias" nối các mã đó về một hồ sơ chính. Hồ sơ trùng được hợp nhất, dữ liệu gộp vào hồ sơ chính, cầu nối giữ lại để tra cứu sau.

**Quy tắc gộp phải khai báo rõ** — ví dụ: "nếu số điện thoại trùng nhau thì gộp". Hệ thống không tự đoán, không tự ghép ngẫu nhiên.

**CDP VNPost học được gì:** Đây chính xác là bài toán VNPost đang có — cùng 1 người gửi nhưng có 4 mã khác nhau ở 4 hệ thống (CAS mã KHL, MyVNPost user ID, PayPost mã tài chính, PostID). Pattern alias của Unomi là cách đúng để giải quyết mà không cần phải hợp nhất database ngay từ đầu.

---

### 2.2 Tự động kích hoạt hành động khi có sự kiện

**Bài toán Unomi giải:** Mỗi khi có sự kiện xảy ra (giao hàng thất bại, tạo đơn, đăng nhập), hệ thống kiểm tra các quy tắc đang có rồi tự động thực hiện hành động tương ứng. Công thức: **Sự kiện xảy ra → Kiểm tra điều kiện → Thực hiện hành động**.

Các hành động hỗ trợ: gắn nhãn khách hàng, cập nhật chỉ số, đẩy tín hiệu ra hàng đợi để hệ thống khác xử lý tiếp. Hệ thống **không gọi trực tiếp SMS hay Zalo** — chỉ phát tín hiệu ra, CRM hay SMS gateway tự lo việc gửi.

**CDP VNPost học được gì:** Bốn use case ưu tiên của VNPost đều phù hợp với pattern này:

| Use case | Sự kiện | Điều kiện | Hành động |
|---|---|---|---|
| UC-02 Anti-Churn | Sản lượng KHL giảm | Giảm 30% so với 4 tuần trước | Gắn nhãn "cần chăm sóc" |
| UC-03 Win-Back | Không có giao dịch mới | Quá 60 ngày im lặng | Đưa vào danh sách win-back |
| UC-04 COD Risk | Đơn COD thất bại | Thất bại lần thứ 3 | Gắn cờ rủi ro địa chỉ |
| UC-06 Fraud | Số điện thoại xuất hiện với nhiều tên | Liên kết trên 5 tên khác nhau | Tạo cảnh báo gian lận |

---

### 2.3 Marketing tự thay đổi ngưỡng mà không cần IT

**Bài toán Unomi giải:** Tách hệ thống thành 3 tầng rõ ràng:
- **Tầng lõi** — IT xây một lần, không đụng vào nữa
- **Tầng cấu hình** — Marketing tự điều chỉnh ngưỡng (60 ngày, 30%, 3 lần thất bại...) bằng file cấu hình, không cần viết code
- **Tầng kết nối** — cấu hình kênh đầu ra: Zalo, SMS, CRM

**CDP VNPost học được gì:** Khi Marketing muốn đổi định nghĩa "inactive" từ 60 ngày sang 45 ngày, không cần nhờ IT sửa code và chờ release. Tự điều chỉnh trong ngày. Đây là yếu tố quyết định tốc độ phản ứng của đội Marketing.

---

### 2.4 Thêm chỉ số phân tích khách hàng mà không cần IT

**Bài toán Unomi giải:** Muốn thêm trường dữ liệu mới vào hồ sơ khách hàng (ví dụ: "tỷ lệ hoàn hàng 30 ngày", "số lần giao thất bại tích lũy"), chỉ cần khai báo bằng file cấu hình — không cần sửa code lõi, không cần release hệ thống.

**CDP VNPost học được gì:** Mỗi khi nghiệp vụ cần thêm chỉ số phân tích mới — ví dụ "điểm tín nhiệm COD", "số tháng hoạt động liên tiếp", "tỷ lệ thành công theo tỉnh" — không cần chờ IT. BA hoặc Data Analyst tự định nghĩa và triển khai trong ngày.

---

### 2.5 Tính điểm tín nhiệm tự động theo hành vi

**Bài toán Unomi giải:** Mỗi hồ sơ khách hàng có thể được gán điểm số tự động — điểm tăng khi có hành vi tốt, giảm khi có hành vi xấu, tự cập nhật mỗi khi có sự kiện mới.

**CDP VNPost học được gì:** Điểm số cho phép phân loại chính xác hơn nhãn nhị phân (có/không). Khách hàng điểm 20 và điểm 80 đều có nhãn "rủi ro" nhưng mức độ xử lý khác nhau.

| Hành vi | Tác động điểm | Dùng cho |
|---|---|---|
| Giao hàng thành công | Tăng điểm | Hồ sơ tín nhiệm tích cực |
| Từ chối nhận COD | Giảm điểm | UC-04 COD Risk |
| Địa chỉ giao thất bại lần 2, lần 3 | Giảm điểm mạnh hơn | UC-04 cảnh báo trước khi nhận đơn mới |
| Số điện thoại liên kết nhiều tên khác nhau | Giảm điểm | UC-06 Fraud Detection |
| Thanh toán COD đúng hạn nhiều lần | Tăng điểm | KHL uy tín — ưu tiên phục vụ |

---

### 2.6 Quản lý đồng ý sử dụng dữ liệu

**Bài toán Unomi giải:** Người dùng có thể bật/tắt từng loại đồng ý riêng (marketing, phân tích, vị trí, tài chính). Khi yêu cầu xóa, Unomi xóa dữ liệu trong hệ thống của mình.

**CDP VNPost học được gì:** Phân tách đồng ý theo từng mục đích thay vì một nút đồng ý chung. Khách hàng đồng ý cho phân tích vận chuyển nhưng có thể không đồng ý cho marketing — CDP phải tôn trọng sự khác biệt này.

**Điểm VNPost cần tự bổ sung:** Unomi chỉ xóa trong hệ thống của nó — không tự lan sang CAS, PayPost, BCCP. VNPost cần thiết kế thêm cơ chế lan truyền xóa để tuân thủ thời hạn 72 giờ theo Luật BVDLCN 2025.

---

## 3. Tính năng Unomi có nhưng VNPost không cần học

| Tính năng | Lý do VNPost không cần |
|---|---|
| Theo dõi hành vi duyệt web theo phiên | VNPost là logistics — khách hàng không "duyệt" như e-commerce |
| Cá nhân hóa nội dung website theo thời gian thực | Không phải use case ưu tiên hiện tại |
| Phân tích hành vi trong ứng dụng (tracking) | MyVNPost App đã có sẵn, không cần Unomi làm thêm |

---

## 4. Bài toán Unomi không giải được — VNPost phải tự xây

| Bài toán VNPost cần | Tại sao Unomi không có |
|---|---|
| Hồ sơ người nhận không có tài khoản | Unomi được thiết kế cho người dùng đã đăng nhập — không có cấu trúc cho "người nhận bưu kiện không có tài khoản" |
| Phát hiện gian lận theo mạng lưới địa chỉ | Unomi phân tích từng hồ sơ riêng lẻ — không nhìn được "một cụm địa chỉ cạnh nhau đồng loạt từ chối hàng" |
| Khách hàng doanh nghiệp có nhiều chi nhánh | Unomi không có cấu trúc B2B tự nhiên (công ty → nhiều tài khoản con) |
| Dữ liệu từ tương tác vật lý của bưu tá | Unomi chỉ xử lý dữ liệu kỹ thuật số — không có khái niệm "bưu tá gặp mặt trực tiếp" |
| Gộp dữ liệu khiếu nại từ hai hệ thống riêng | Unomi không tự nhận biết Care Đơn và Cas-HTKH chứa cùng loại dữ liệu — cần pipeline hợp nhất trước |

---

## 5. Bản đồ hệ thống VNPost và dữ liệu liên quan

> Nguồn: tài liệu kiến trúc gốc từ VNPost (sơ đồ MPITS, chức năng phần mềm, quy trình tổng quan).

### Nhóm 1 — Điểm tiếp xúc khách hàng (dữ liệu hành vi)

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **MyVietnam Post** (app) | Đơn hàng, tài khoản, hành vi app, đối soát, hỗ trợ | Gộp ID, kích hoạt theo sự kiện hành vi |
| **vnpost.vn** (web) | Hành vi tìm kiếm, theo dõi đơn, tạo đơn online | Gộp ID |
| **SmartLocker** ⚠️ | Lịch sử lấy hàng tại tủ khóa, thời điểm lấy | Gộp ID — chưa rõ có mã định danh riêng hay chỉ dùng số điện thoại |
| **PostPay / E-Money** ⚠️ | Lịch sử thanh toán, ví điện tử, COD | Gộp ID, quản lý đồng ý tài chính bắt buộc |

> ⚠️ SmartLocker và E-Money phát hiện từ tài liệu gốc VNPost — chưa có trong Domain Brief v2.1. Cần xác nhận với IT VNPost.

### Nhóm 2 — Chấp nhận gửi (dữ liệu đầy đủ nhất về người gửi)

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **CAS (Counter Acceptance)** | Họ tên, số điện thoại, địa chỉ người gửi + nhận, loại dịch vụ, COD amount, sự vụ | Mã KHL là điểm neo trung tâm trong gộp ID; nguồn sự kiện tạo đơn |
| **Portal KHL** | Chấp nhận gửi hàng loạt cho doanh nghiệp, tính cước | Theo dõi sản lượng KHL — phục vụ UC-02 Anti-Churn |
| **MPITS** | Hub tổng hợp — nhận từ tất cả hệ thống, kết nối sàn thương mại điện tử | Cổng dữ liệu trung tâm nếu có API — không tạo ID riêng |

### Nhóm 3 — Thu gom (tần suất và địa chỉ lấy hàng)

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **PNS Thu Gom / DingDong** | Tin thu gom, điều tin, chuyển tuyến | Sự kiện lấy hàng thực tế — tần suất gửi của khách |
| **QLTG** | Lịch thu gom, phân công bưu tá | Phân tích tần suất gửi theo địa điểm |

### Nhóm 4 — Phát hàng (kết quả — quan trọng nhất cho UC-04, UC-06)

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **PNS Phát / DingDong** | Kết quả phát (thành công/thất bại/hoàn), gạch nợ, sự vụ, tuyến đường thư | Nguồn sự kiện chính cho UC-04 và UC-06; hồ sơ người nhận anonymous |
| **WMS** | Nhập/xuất hàng, tồn kho bưu gửi, lý do không phát | Phân tích hàng tồn — ít liên quan trực tiếp hồ sơ khách hàng |
| **TMS** | Kế hoạch điều xe, giám sát hành trình, báo cáo sản lượng | Thời gian vận chuyển thực tế — chỉ số chất lượng dịch vụ |

### Nhóm 5 — Tài chính COD (nhạy cảm nhất)

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **PayPost** | Thu COD, gạch nợ, trả tiền người gửi, lịch sử tài chính | Gộp ID phía tài chính; quản lý đồng ý bắt buộc; UC-04 COD Risk |
| **E-Money** ⚠️ | Ví điện tử, giao dịch thanh toán | Tách biệt PayPost — dữ liệu tài chính nằm ở hai chỗ khác nhau |

> ⚠️ PayPost và E-Money là hai hệ thống tài chính riêng biệt — cần xác nhận phạm vi từng hệ thống với IT VNPost.

### Nhóm 6 — Quản lý quan hệ khách hàng ⚠️

| Hệ thống | Dữ liệu khách hàng có | Liên quan đến tính năng nào |
|---|---|---|
| **Care Đơn** | Khiếu nại, sự vụ, chăm sóc khách hàng | Tín hiệu churn sớm — đầu vào cho UC-02 |
| **CCP / SalesForce / PostSale** ⚠️ | Quan hệ KHL doanh nghiệp, sales pipeline | Hồ sơ KHL đầy đủ nhất — cần kết nối vào gộp ID |
| **Cas-HTKH** ⚠️ | Hỗ trợ khách hàng tại quầy — tách biệt Care Đơn | Dữ liệu khiếu nại có thể bị phân mảnh giữa hai hệ thống này |

> ⚠️ CCP/SalesForce/PostSale và Cas-HTKH phát hiện từ tài liệu gốc — chưa có trong Domain Brief v2.1. Đặc biệt cần làm rõ: hai hệ thống hỗ trợ khách hàng có dữ liệu trùng nhau không?

### Nhóm 7 — Hệ thống nền (định danh và địa chỉ)

| Hệ thống | Dữ liệu có | Vai trò trong CDP |
|---|---|---|
| **PostID** | Định danh người dùng VNPost | Điểm neo trung tâm trong gộp ID |
| **VPostCode / Địa chỉ số** | Chuẩn hóa địa chỉ Việt Nam | Chuẩn hóa địa chỉ người nhận — phục vụ hồ sơ anonymous |
| **Vmap / Bản đồ số** | Tọa độ, tuyến đường | Phân tích địa chỉ rủi ro theo vùng địa lý |

### Nhóm 8 — Báo cáo và phân tích (đã có sẵn)

| Hệ thống | Dữ liệu có | Ghi chú |
|---|---|---|
| **CAS-ReportingDB** | Báo cáo từ CAS | Có thể dùng làm nguồn xuất dữ liệu hàng ngày cho CDP trong giai đoạn đầu |
| **BI** | Báo cáo phân tích nội bộ | CDP bổ sung chiều dữ liệu khách hàng vào đây |

---

## 6. Câu hỏi còn mở — cần xác nhận với PO và IT VNPost

| Câu hỏi | Tại sao cần biết |
|---|---|
| Luật BVDLCN 2025 có ngoại lệ gì cho ngành bưu chính khi lưu thông tin người nhận không có tài khoản? | Quyết định phạm vi hồ sơ người nhận anonymous — được phép lưu gì, không được lưu gì |
| SmartLocker có mã định danh riêng không, hay chỉ dùng số điện thoại? | Nếu có mã riêng → thêm một alias nữa vào cơ chế gộp ID; nếu chỉ số điện thoại → gộp được ngay |
| PayPost và E-Money là hai hệ thống độc lập hay liên thông? | Nếu độc lập → dữ liệu tài chính bị phân mảnh, cần hai cầu nối riêng; ảnh hưởng thiết kế UC-04 |
| Care Đơn và Cas-HTKH có lưu cùng loại dữ liệu khiếu nại không, hay hoàn toàn tách biệt? | Nếu trùng → phải hợp nhất trước khi đưa vào CDP; nếu tách biệt → cần gộp thủ công |
| CCP/SalesForce/PostSale là hệ thống nội bộ VNPost hay phần mềm bên thứ ba? | Nếu bên thứ ba → cần thỏa thuận tích hợp riêng; ảnh hưởng lộ trình kết nối dữ liệu |
| Phân quyền tỉnh/vùng trong CDP: tỉnh A có được xem dữ liệu khách hàng của tỉnh B không? | Quyết định kiến trúc phân quyền — cần chốt với lãnh đạo trước khi thiết kế |
