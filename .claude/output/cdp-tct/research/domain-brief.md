# Domain Brief — CDP (Customer Data Platform) cho VNPost / TCT

**Ngày tạo:** 2026-05-28
**Người thực hiện:** ba-research-agent
**Dự án:** Hệ thống CDP cho TCT (Tổng Công Ty Bưu Điện Việt Nam — VNPost)
**Phiên bản:** v2.1
**Cập nhật:** 2026-06-05 — Bổ sung Mục 11 (đánh giá Apache Unomi và 4 CDP tương tự trên thị trường) và Mục 12 (phân tích 3 phương án triển khai Build/Buy/Partner, khuyến nghị BA, agenda meeting chốt phương án).

---

## 1. Tổng quan domain

### CDP là gì và tại sao VNPost cần

Customer Data Platform (Nền tảng Dữ liệu Khách hàng) là phần mềm thu thập, hợp nhất và kích hoạt dữ liệu khách hàng từ nhiều nguồn — tạo ra một hồ sơ khách hàng duy nhất, liên tục cập nhật, phục vụ Marketing, Sales, Operations và AI.

VNPost có lý do đặc biệt mạnh để xây CDP, xuất phát từ 3 thực tế:

- **Mạng lưới rộng nhưng dữ liệu phân mảnh:** VNPost có ~13.000 điểm phục vụ trên 63 tỉnh thành, xử lý hàng chục triệu bưu gửi/tháng qua 8+ hệ thống IT khác nhau — mỗi hệ thống lưu dữ liệu khách hàng theo cách riêng, không có hồ sơ thống nhất.
- **COD chiếm ~70–80% giao dịch thương mại điện tử tại Việt Nam:** Đây vừa là cơ hội (dữ liệu tài chính khách hàng phong phú) vừa là rủi ro (tỷ lệ hoàn hàng 10–15%, chi phí vận hành COD cao) — CDP giúp phân tích và giảm thiểu rủi ro này.
- **Cạnh tranh logistics ngày càng gay gắt:** J&T, GHN, GHTK, Ninja Van... đang cạnh tranh trực tiếp trên phân khúc e-commerce. Hiểu khách hàng tốt hơn = giữ chân khách hàng tốt hơn.

### Ba giai đoạn vận hành cốt lõi của CDP

1. **Thu thập (Data Collection):** Kéo dữ liệu từ mọi điểm chạm — web, app, hệ thống nội bộ, sàn TMĐT — theo thời gian thực hoặc theo lô.
2. **Hợp nhất (Data Unification / Identity Resolution):** Ghép nối định danh rời rạc (SĐT, email, CCCD, mã khách hàng) từ nhiều hệ thống thành một hồ sơ duy nhất.
3. **Kích hoạt (Data Activation):** Cung cấp hồ sơ và phân khúc cho các hệ thống downstream — gửi SMS, email, cá nhân hóa trải nghiệm, cung cấp cho AI.

### Đặc thù khi triển khai cho VNPost (tổng công ty lớn, đa nghiệp vụ)

- Dữ liệu phân tán qua hàng chục hệ thống IT, nhiều hệ thống legacy không có API chuẩn
- Cùng một khách hàng có thể là: người gửi hàng (qua quầy), shipper TMĐT (qua API), người nhận hàng (không có tài khoản) — 3 vai trò khác nhau trong cùng một giao dịch
- Identity Resolution phức tạp: một gói hàng có 2 chủ thể (người gửi + người nhận), cả hai đều là "khách hàng" theo nghĩa khác nhau
- Phải tuân thủ Nghị định 13/2023/NĐ-CP — đặc biệt dữ liệu vị trí địa lý được phân loại là **dữ liệu cá nhân nhạy cảm**

**Nguồn tham khảo:** [CDP.com — What is a CDP](https://cdp.com/basics/what-is-a-customer-data-platform-cdp/), [CDP Use Cases 2026](https://cdp.com/basics/cdp-use-cases/), [Last Mile Delivery Vietnam](https://fastforwardadvisors.com/last-mile-delivery-in-southeast-asia-and-vietnam/)

---

## 2. Hệ sinh thái hệ thống VNPost

### 2.1 Sơ đồ luồng nghiệp vụ bưu chính chuyển phát (8 giai đoạn)

```
[1] KHÁCH HÀNG
    Yêu cầu gửi hàng qua: MyVNPost App | Sàn TMĐT (Shopee, Lazada, TikTok Shop...)
         ↓
[2] THU GOM
    Tiếp nhận yêu cầu → Điều tin thu gom → Bưu tá đến lấy hàng
    Hệ thống: QLTG (Quản lý thu gom), PNS Thu Gom / DingDong (app bưu tá)
         ↓
[3] CHẤP NHẬN GỬI
    Giao nhận tại bưu cục → Nhập thông tin gửi → Chấp nhận gửi
    Hệ thống: Portal KHL (khách hàng lớn/doanh nghiệp), CAS (quầy bưu cục), MPITS
         ↓
[4] KHAI THÁC GỬI
    Chia chọn bưu gửi → Đóng chuyến thư → Lập bảng kê (BD6, BD10...)
    Hệ thống: BCCP (Phần mềm quản lý dịch vụ bưu chính)
         ↓
[5] GIAO NHẬN VẬN CHUYỂN
    Lập lệnh điều xe → Quét BD10 lên xe → Giám sát hành trình → Quét BD10 xuống
    Hệ thống: TMS (Transportation Management System)
         ↓
[6] KHAI THÁC NHẬN
    Xác nhận BD10 đến → Khai thác bưu gửi tại bưu cục nhận
    Hệ thống: BCCP
         ↓
[7] PHÁT
    Phân hướng → Lập bảng kê BD13 → Bưu tá phát → Phát lại / Chuyển hoàn / Nhập kho / Cắt hàng / Xuất hàng
    Hệ thống: PNS Phát / DingDong (app bưu tá), WMS (Warehouse Management System)
         ↓
[8] THU & TRẢ TIỀN COD
    Gạch nợ COD → Trả tiền người gửi
    Hệ thống: PayPost
```

### 2.2 Bản đồ phần mềm VNPost theo chức năng

| Hệ thống | Giai đoạn | Chức năng chính | Dữ liệu khách hàng tạo ra |
|---|---|---|---|
| **MyVNPost / My Vietnam Post Plus** | Đầu vào (KH) | Tạo đơn hàng, theo dõi bưu gửi, tài khoản, đối soát, hỗ trợ | Hành vi app, thông tin người gửi/nhận, lịch sử đơn hàng, tọa độ GPS (nếu bật) |
| **Sàn TMĐT** | Đầu vào (KH) | Marketplace order tạo đơn tự động qua API | Thông tin đơn hàng, người gửi, địa chỉ nhận, COD amount |
| **QLTG (Quản lý Thu gom)** | Thu gom | Quản lý lịch thu gom, phân công bưu tá | Địa chỉ lấy hàng, tần suất gửi, khu vực phục vụ |
| **PNS Thu Gom / DingDong** | Thu gom | App bưu tá: nhận tin, thu gom, cập nhật trạng thái | Thời gian thực tế lấy hàng, vị trí bưu tá (GPS), xác nhận thu gom |
| **Portal KHL (Khách Hàng Lớn)** | Chấp nhận gửi | Quản lý chấp nhận gửi hàng loạt cho doanh nghiệp TMĐT lớn; tính cước; in vận đơn | Thông tin doanh nghiệp gửi hàng, khối lượng, loại dịch vụ, cước phí |
| **CAS (Counter Acceptance System)** | Chấp nhận gửi | Quầy bưu cục — ~100 dịch vụ bưu chính + 60 dịch vụ tài chính trên 1 giao diện | Thông tin người gửi, người nhận, loại dịch vụ, cước phí, COD |
| **MPITS** | Xuyên suốt | Nền tảng tích hợp trung tâm — kết nối ~30 ứng dụng; xử lý tự động toàn bộ tác nghiệp | Hub tổng hợp: đơn hàng, tracking, khách hàng, tài chính |
| **BCCP** | Khai thác gửi + nhận | Quản lý chuyến thư, điều hướng bưu gửi, phân hướng tuyến phát, quản lý túi thư | Trạng thái bưu gửi theo từng điểm, bảng kê BD6/BD10 |
| **TMS (Transportation Mgmt)** | Giao nhận vận chuyển | Lập kế hoạch điều xe, xác nhận chứng từ, giám sát hành trình, báo cáo sản lượng | Tuyến vận chuyển, thời gian vận chuyển thực tế, GPS xe |
| **PNS Phát / DingDong** | Phát | App bưu tá: phát hàng, thu COD, xử lý chuyển hoàn, chăm sóc KH tại điểm phát | Trạng thái phát thực tế, kết quả COD, lý do không phát được, vị trí GPS bưu tá |
| **WMS (Warehouse Mgmt)** | Phát / Lưu kho | Quản lý nhập/xuất hàng, tồn kho bưu gửi | Hàng tồn kho, ngày nhập, lý do không phát |
| **PayPost** | Thu/Trả COD | Thu tiền COD, gạch nợ, trả tiền người gửi | Số tiền COD thực thu, thời gian thanh toán, lịch sử giao dịch tài chính |
| **CMS** | Hỗ trợ ngang | Quản lý nội dung website/portal | Nội dung marketing, thông tin sản phẩm |
| **PostID** | Hỗ trợ ngang | Xác thực danh tính người dùng | Định danh người dùng VNPost (identity anchor) |
| **Vmap (Bản đồ số)** | Hỗ trợ ngang | Bản đồ số Việt Nam, định tuyến | Hỗ trợ địa chỉ, tối ưu tuyến phát |
| **VPostCode (Địa chỉ số)** | Hỗ trợ ngang | Chuẩn hóa địa chỉ Việt Nam | Chuẩn hóa địa chỉ người gửi/nhận |
| **Care Đơn** | Hỗ trợ ngang | Chăm sóc khách hàng, xử lý sự vụ | Lịch sử khiếu nại, đánh giá dịch vụ, sự vụ phát sinh |

**Nguồn tham khảo:** [VNPost — BCCP phần mềm bưu chính](https://vnpost.vn/vi/tin-khac/quyet-dinh-trien-khai-phan-mem-ung-dung-quan-ly-cac-dich-vu-buu-chinh), [VNPost — DingDong nâng cấp hệ thống phát](https://vnpost.vn/en/hoat-dong-nganh/vietnam-post-nang-cap-he-thong-phan-mem-quan-li-cong-doan-phat-buu-gui-danh-cho-buu-ta), [VNPost — MPITS CNTT nền tảng tối ưu hóa](https://vnpost.vn/en/chuyen-phat-tmdt-logistics/cong-nghe-thong-tin-la-nen-tang-toi-uu-hoa-san-xuat-nang-cao-chat-luong-dich-vu)

---

## 3. Data Touchpoints — Dữ liệu khách hàng tại từng giai đoạn

### 3.1 Bản đồ dữ liệu theo luồng

| Giai đoạn | Hệ thống nguồn | Loại dữ liệu khách hàng | Định danh có trong dữ liệu |
|---|---|---|---|
| **Khách hàng tạo đơn** | MyVNPost, Sàn TMĐT | Tên, SĐT, địa chỉ (gửi + nhận), loại hàng, trọng lượng, COD amount, ghi chú | SĐT, email, user ID app |
| **Thu gom** | QLTG, PNS/DingDong | Địa chỉ lấy hàng, khoảng cách, thời gian thực lấy hàng, tần suất gửi theo địa chỉ | SĐT người gửi, địa chỉ |
| **Chấp nhận gửi** | CAS, Portal KHL, MPITS | Thông tin đầy đủ người gửi/nhận, loại dịch vụ chọn, cước phí tính, phương thức thanh toán | Mã khách hàng CAS, SĐT, mã vận đơn |
| **Khai thác** | BCCP | Trạng thái xử lý bưu gửi, tuyến đi, thời gian ở mỗi điểm khai thác | Mã vận đơn (tracking number) |
| **Vận chuyển** | TMS | Tuyến thực tế, thời gian vận chuyển, chậm trễ (nếu có) | Mã vận đơn, mã chuyến thư |
| **Phát hàng** | PNS Phát/DingDong | Kết quả phát (thành công/thất bại/chuyển hoàn), thời gian phát thực tế, lý do không phát, vị trí GPS | Mã vận đơn, SĐT người nhận |
| **COD** | PayPost | Số tiền COD thực thu, ngày thu, phương thức trả cho người gửi, lịch sử giao dịch | SĐT người gửi, mã vận đơn, số tài khoản (nếu chuyển khoản) |
| **Chăm sóc KH** | Care Đơn, CRM | Khiếu nại, sự vụ, đánh giá, lịch sử liên hệ | SĐT, email, mã vận đơn |

### 3.2 Phân loại dữ liệu theo mục đích CDP

**Dữ liệu nhân khẩu học (Demographic Data):**
- Họ tên, số điện thoại, địa chỉ thường xuyên gửi/nhận, email, CCCD (nếu có)
- Nguồn chính: CAS, Portal KHL, MyVNPost, PostID

**Dữ liệu hành vi giao dịch (Transactional Behavioral Data):**
- Tần suất gửi hàng (theo tuần/tháng), giá trị COD trung bình, loại dịch vụ ưa dùng (nhanh / thường / siêu tốc), khu vực gửi đi thường xuyên, tỷ lệ hàng hoàn
- Nguồn chính: BCCP, MPITS, PayPost, CAS

**Dữ liệu hành vi kênh (Channel Behavioral Data):**
- Thao tác trên app MyVNPost (xem tracking, tạo đơn, tìm bưu cục), tương tác website
- Nguồn chính: MyVNPost App

**Dữ liệu vị trí (Location Data — Nhạy cảm theo NĐ 13):**
- Địa chỉ người gửi, địa chỉ người nhận, GPS của bưu tá khi phát hàng
- `[CẢNH BÁO]` Dữ liệu vị trí được xác định qua dịch vụ định vị là **dữ liệu cá nhân nhạy cảm** theo Điều 2 Nghị định 13/2023/NĐ-CP — cần có quy trình xử lý đặc biệt và consent rõ ràng

**Dữ liệu tài chính (Financial Data — Nhạy cảm theo NĐ 13):**
- COD amount, lịch sử thanh toán, thông tin tài khoản ngân hàng (nếu chuyển khoản COD)
- `[CẢNH BÁO]` Dữ liệu tài chính của khách hàng (dữ liệu khách hàng của tổ chức tài chính) thuộc danh sách **dữ liệu nhạy cảm** theo NĐ 13

**Dữ liệu chất lượng dịch vụ (Service Quality Data):**
- Khiếu nại, sự vụ, thời gian giải quyết, đánh giá, lý do chuyển hoàn
- Nguồn chính: Care Đơn, PNS Phát

---

## 4. Actors và vai trò

### Người dùng vận hành hệ thống CDP

| Actor | Vai trò | Quan tâm chính |
|---|---|---|
| **Chuyên viên Marketing / CRM** | Tạo phân khúc khách hàng, thiết lập chiến dịch giữ chân/tái kích hoạt, xem báo cáo | Dễ dùng, tự phục vụ không cần IT, phân khúc chính xác |
| **Chuyên viên Phân tích Dữ liệu** | Phân tích hành vi, xây dashboard, đo hiệu quả chiến dịch, dự báo churn | Độ chính xác dữ liệu, khả năng truy vấn linh hoạt, kết nối BI tool |
| **Kỹ sư Dữ liệu (Data Engineer)** | Xây dựng pipeline tích hợp từ 8+ hệ thống nguồn, quản lý schema, xử lý lỗi | Tốc độ xử lý real-time, độ ổn định pipeline, dễ debug |
| **Quản trị viên Hệ thống** | Quản lý quyền truy cập, cấu hình, giám sát hiệu năng | Bảo mật, audit log, phân quyền chi tiết, tuân thủ NĐ 13 |

### Người ra quyết định / Hưởng lợi

| Actor | Vai trò | Quan tâm chính |
|---|---|---|
| **Ban lãnh đạo TCT / Giám đốc Marketing** | Phê duyệt chiến lược, đánh giá ROI | Hiệu quả chiến dịch, tỷ lệ giữ chân KH, doanh thu từ TMĐT |
| **Giám đốc IT / CTO** | Phê duyệt kiến trúc, đảm bảo an toàn, phê duyệt tích hợp | Không làm gián đoạn hệ thống hiện có, bảo mật, tuân thủ pháp lý |
| **Quản lý đơn vị tỉnh/thành** | Sử dụng insight để cải thiện vận hành địa phương | Báo cáo theo vùng, KPI phát hàng, tỷ lệ hoàn |
| **Khách hàng doanh nghiệp TMĐT (Shipper)** | Gửi hàng số lượng lớn, cần đối soát, theo dõi hiệu suất | Tỷ lệ phát thành công, COD được hoàn nhanh, báo cáo sản lượng |
| **Khách hàng cá nhân (Người gửi/Người nhận)** | Đối tượng dữ liệu được thu thập | Trải nghiệm cá nhân hóa, quyền riêng tư dữ liệu |

### Hệ thống nguồn (Source Systems) — Cập nhật từ sơ đồ

Xem Mục 2.2 cho danh sách đầy đủ. Ưu tiên tích hợp CDP:
1. **MPITS** — hub trung tâm, nếu lấy được từ đây là lấy được dữ liệu tổng hợp
2. **CAS / Portal KHL** — dữ liệu chấp nhận gửi, thông tin đầy đủ nhất về người gửi
3. **MyVNPost App** — dữ liệu hành vi real-time, người dùng đã xác thực
4. **PayPost** — dữ liệu tài chính COD, chỉ số quan trọng nhất với TMĐT
5. **PNS Phát / DingDong** — kết quả phát thực tế, tỷ lệ thành công/hoàn
6. **Care Đơn / CRM** — dữ liệu chất lượng dịch vụ, khiếu nại

---

## 5. Use Cases CDP thực tế cho VNPost

CDP không chỉ là "lưu dữ liệu" — 6 use case sau đây có giá trị kinh doanh trực tiếp và đo được:

### UC-01: Customer 360 — Hồ sơ khách hàng hợp nhất

**Vấn đề:** Hiện tại cùng một khách hàng doanh nghiệp TMĐT có thể tồn tại ở CAS (mã KHL), Portal KHL (mã riêng), MyVNPost (user ID), PayPost (mã tài chính) — 4 hồ sơ rời rạc, không biết là cùng một người.

**CDP làm gì:** Ghép nối qua SĐT/email/mã vận đơn → một hồ sơ duy nhất bao gồm toàn bộ lịch sử giao dịch, tần suất gửi, COD amount, tỷ lệ hoàn, khiếu nại.

**Outcome:** Đội bán hàng / CSKH có bức tranh đầy đủ về khách hàng trong 1 màn hình thay vì phải tra 4 hệ thống.

---

### UC-02: Phân khúc và giữ chân khách hàng TMĐT (Anti-Churn)

**Vấn đề:** VNPost không biết shipper TMĐT nào đang có xu hướng chuyển sang GHN/GHTK cho đến khi họ đã rời đi.

**CDP làm gì:** Phát hiện dấu hiệu sớm — sụt giảm sản lượng gửi hàng so với 4–8 tuần trước; tăng tỷ lệ khiếu nại; dừng tạo đơn trên Portal KHL. Tự động đưa vào phân khúc "at-risk" và kích hoạt chiến dịch giữ chân (ưu đãi cước, gọi điện chăm sóc).

**Outcome:** Nghiên cứu toàn cầu cho thấy tăng 5% tỷ lệ giữ chân có thể tăng 25–95% lợi nhuận. Với VNPost: mỗi shipper TMĐT lớn mang giá trị hàng trăm triệu VND/tháng.

---

### UC-03: Tái kích hoạt khách hàng không hoạt động (Win-Back)

**Vấn đề:** Khách hàng cá nhân và doanh nghiệp nhỏ gửi hàng theo mùa hoặc ngừng hẳn — VNPost không có cơ chế tự động nhận ra và tiếp cận lại.

**CDP làm gì:** Định nghĩa "inactive" = không có giao dịch trong X ngày (tùy phân khúc). Tự động trigger chiến dịch: SMS/zalo với ưu đãi theo lịch sử sử dụng dịch vụ trước đây.

**Outcome:** Win-back campaigns thường có tỷ lệ chuyển đổi 5–15% với chi phí thấp hơn nhiều so với acquisition.

---

### UC-04: Giảm tỷ lệ hoàn hàng và tối ưu COD

**Vấn đề:** COD chiếm ~70–80% giao dịch TMĐT Việt Nam; tỷ lệ hoàn hàng 10–15%. Mỗi lần hoàn tốn chi phí vận hành 2 chiều mà VNPost gánh.

**CDP làm gì:**
- Phân tích lịch sử người nhận: địa chỉ có hay xảy ra không phát được không? SĐT có thay đổi thường xuyên không?
- Phân tích người gửi: tỷ lệ hoàn của shipper cụ thể có cao bất thường không? (dấu hiệu hàng kém chất lượng hoặc fraud)
- Tạo "địa chỉ rủi ro cao" và "shipper rủi ro cao" để cảnh báo trước khi nhận đơn hoặc điều chỉnh điều khoản

**Outcome:** Giảm 1% tỷ lệ hoàn tương đương tiết kiệm hàng tỷ VND chi phí vận hành/năm ở quy mô VNPost.

---

### UC-05: Cá nhân hóa dịch vụ và Cross-sell

**Vấn đề:** Khách hàng cá nhân gửi hàng theo mùa (Tết, 11/11...) không nhận được ưu đãi phù hợp thời điểm; khách hàng doanh nghiệp chỉ dùng 1 dịch vụ không biết các dịch vụ khác phù hợp hơn.

**CDP làm gì:**
- Phân khúc theo hành vi mùa vụ → gửi thông báo đúng timing
- Phân tích loại hàng + trọng lượng + tuyến đường → đề xuất dịch vụ phù hợp hơn (siêu tốc, bảo hiểm hàng, lưu kho...)
- Tích hợp với CMS để hiển thị banner/nội dung cá nhân hóa trên MyVNPost App

**Outcome:** Tăng giá trị trung bình mỗi giao dịch; tăng sử dụng các dịch vụ giá trị cao.

---

### UC-06: Phát hiện gian lận (Fraud Detection)

**Vấn đề:** Địa chỉ giả, đơn hàng COD fraud (đặt hàng rồi từ chối nhận), tài khoản doanh nghiệp giả mạo để lấy ưu đãi.

**CDP làm gì:** Phát hiện pattern bất thường — địa chỉ người nhận thay đổi liên tục, số SĐT liên kết với nhiều tên khác nhau, tỷ lệ từ chối nhận COD bất thường cao.

**Outcome:** Giảm rủi ro tài chính và vận hành; theo nghiên cứu logistics tại Đông Nam Á, fraud chiếm 2–5% tổng giao dịch COD.

**Nguồn tham khảo:** [CDP Use Cases — churn prevention, cross-sell, win-back](https://cdp.com/basics/cdp-use-cases/), [CDP — Customer Loyalty & Retention](https://cdp.com/articles/customer-loyalty-retention-cdp/), [Last Mile Delivery Vietnam — COD challenges](https://fastforwardadvisors.com/last-mile-delivery-in-southeast-asia-and-vietnam/)

---

## 6. Đánh giá độ phức tạp tích hợp per-system

### Tiêu chí đánh giá

- **Độ quan trọng dữ liệu KH:** Mức độ ảnh hưởng đến chất lượng hồ sơ khách hàng trong CDP
- **Khả năng tích hợp:** Đánh giá dựa trên thông tin công khai và đặc thù hệ thống
- **Ưu tiên tích hợp:** Thứ tự nên làm trước trong MVP

| Hệ thống | Độ quan trọng KH | Khả năng tích hợp (ước tính) | Phương thức likely | Ưu tiên |
|---|---|---|---|---|
| **MPITS** | Rất cao — hub tổng hợp | Trung bình — hệ thống lõi, cần thẩm quyền IT cao | REST API (có kết nối sẵn với sàn TMĐT) hoặc DB link | P1 — nếu có API, đây là nguồn số 1 |
| **CAS / Portal KHL** | Cao — đầy đủ thông tin người gửi | Trung bình — web-based, likely có API hoặc DB export | REST API hoặc database link | P1 — dữ liệu chấp nhận gửi quan trọng |
| **MyVNPost App** | Cao — hành vi real-time người dùng xác thực | Cao — app hiện đại, likely có backend API | REST API / WebSocket / event streaming | P1 — duy nhất có dữ liệu hành vi real-time |
| **PayPost** | Cao — dữ liệu tài chính COD | Trung bình — hệ thống thanh toán, bảo mật cao | REST API hoặc batch export (file) | P2 — sau khi có dữ liệu giao dịch |
| **PNS Phát / DingDong** | Cao — kết quả phát thực tế | Trung bình cao — app mobile, likely sync với MPITS | REST API qua MPITS gateway hoặc trực tiếp | P1 — kết quả phát là KPI quan trọng nhất |
| **BCCP** | Trung bình — trạng thái vận hành nội bộ | Thấp đến trung bình — hệ thống legacy bưu chính | Database link hoặc batch file export | P2 — tracking data, ít cần real-time |
| **TMS** | Thấp với KH — cao với vận hành | Thấp — hệ thống vận tải chuyên biệt | Batch export hoặc database link | P3 — chỉ cần nếu phân tích thời gian giao hàng |
| **WMS** | Thấp với KH — dữ liệu kho | Thấp — hệ thống kho nội bộ | Batch export | P3 — chỉ cần nếu phân tích hàng tồn/hoàn |
| **Care Đơn / CRM** | Cao — dữ liệu chất lượng dịch vụ | Trung bình — CRM thường có API | REST API hoặc webhook | P2 — bổ sung chiều dữ liệu CSKH |
| **PostID** | Rất cao — định danh chuẩn | Cao — hệ thống identity, likely REST API | REST API | P1 — cần làm sớm để chuẩn hóa định danh |
| **Sàn TMĐT (Shopee, Lazada...)** | Cao — shipper lớn, dữ liệu đơn hàng | Cao — sàn có API chuẩn | REST API (webhook order event) | P1 — nguồn đơn hàng lớn nhất |

### Tóm tắt rủi ro tích hợp

- **Rủi ro cao nhất:** BCCP, TMS, WMS — likely là hệ thống legacy không có REST API; cần file export hoặc database direct connection; độ trễ dữ liệu cao (batch overnight thay vì real-time)
- **Cần xác nhận sớm nhất:** MPITS — nếu MPITS đã expose API cho sàn TMĐT thì khả năng lấy dữ liệu qua đây là cao nhất; nếu không thì cần tích hợp từng hệ thống con
- **Ẩn số lớn nhất:** PayPost — dữ liệu tài chính nhạy cảm, cần thẩm quyền phê duyệt cao hơn và quy trình bảo mật đặc biệt

`[Assumption]` Đánh giá "Khả năng tích hợp" ở trên dựa trên thông tin công khai và suy luận từ đặc thù từng loại hệ thống — chưa được xác nhận bởi IT VNPost. Cần clarify trực tiếp.

---

## 7. Compliance — Nghị định 13/2023/NĐ-CP áp dụng cho CDP

### Điểm quan trọng nhất với dự án CDP

Nghị định 13/2023/NĐ-CP (hiệu lực từ 01/07/2023) áp dụng cho **mọi tổ chức xử lý dữ liệu cá nhân của người dùng tại Việt Nam** — VNPost phải tuân thủ ngay khi thu thập và xử lý dữ liệu trong CDP.

### Phân loại dữ liệu VNPost sẽ xử lý trong CDP

| Loại dữ liệu | Phân loại theo NĐ 13 | Yêu cầu xử lý đặc biệt |
|---|---|---|
| Họ tên, địa chỉ, SĐT người gửi/nhận | Dữ liệu cá nhân thông thường | Cần consent; phải có mục đích rõ ràng |
| Vị trí GPS của bưu tá / địa chỉ phát hàng | **Dữ liệu nhạy cảm** (vị trí qua dịch vụ định vị) | Consent đặc biệt; bảo mật cao hơn |
| Dữ liệu tài chính COD, thông tin tài khoản | **Dữ liệu nhạy cảm** (dữ liệu KH tổ chức tài chính) | Consent đặc biệt; không chia sẻ cho bên thứ 3 |
| Hành vi sử dụng app MyVNPost | Dữ liệu cá nhân thông thường | Cần consent trong app |

### Nghĩa vụ bắt buộc của VNPost khi xây CDP

1. **Consent (Đồng ý):** Phải có sự đồng ý rõ ràng, tự nguyện trước khi xử lý dữ liệu. Với dữ liệu nhạy cảm, consent phải được thể hiện bằng văn bản (bao gồm văn bản điện tử).
2. **Quyền chủ thể dữ liệu:** Phải cho phép người dùng truy cập, chỉnh sửa, xóa dữ liệu của họ trong vòng 72 giờ kể từ khi yêu cầu.
3. **DPIA (Đánh giá tác động):** Phải thực hiện đánh giá tác động bảo vệ dữ liệu trước khi triển khai CDP xử lý dữ liệu quy mô lớn.
4. **Thông báo vi phạm:** Phải thông báo cho Cục An ninh mạng trong vòng 72 giờ khi phát hiện vi phạm dữ liệu.
5. **Mục đích rõ ràng:** Dữ liệu thu thập chỉ được dùng đúng mục đích đã khai báo khi lấy consent.

### Hàm ý thiết kế CDP

- CDP phải có module **Consent Management** tích hợp sẵn — không thể thêm vào sau
- Phải hỗ trợ **Right to be Forgotten** (xóa toàn bộ dữ liệu của một chủ thể khi có yêu cầu)
- Dữ liệu vị trí và tài chính phải được **mã hóa và phân quyền truy cập nghiêm ngặt** ngay từ thiết kế
- Cần thiết kế **audit trail** ghi lại ai truy cập dữ liệu nào, lúc nào

Apache Unomi (nếu được chọn làm nền tảng) có module consent management tích hợp sẵn — đây là điểm phù hợp.

**Nguồn tham khảo:** [Nghị định 13/2023/NĐ-CP văn bản gốc](https://vanban.chinhphu.vn/?pageid=27160&docid=207759), [VNG Cloud — Doanh nghiệp cần làm gì theo NĐ 13](https://vngcloud.vn/vi/blog/key-notes-for-businesses-under-decree-13-2023-nd-cp-on-personal-data-protection), [Tokio Marine — NĐ 13 chi tiết](https://tokiomarine.com.vn/nghi-dinh-so-13-2023-nd-cp-ve-bao-ve-du-lieu-ca-nhan.html)

---

## 8. Pain points phổ biến

### Khi xây dựng / triển khai CDP cho bưu chính

- **Dữ liệu phân mảnh trên 8+ hệ thống:** Schema, quy ước đặt tên và tần suất cập nhật khác nhau hoàn toàn — CAS dùng mã KHL, MyVNPost dùng user ID, PayPost dùng mã tài chính — không tự động khớp được
- **Identity resolution phức tạp đặc thù bưu chính:** Một giao dịch có 2 chủ thể (người gửi + người nhận), cả hai đều có dữ liệu nhưng chỉ người gửi là "khách hàng thực sự"; người nhận không có tài khoản VNPost nhưng xuất hiện trong hàng triệu giao dịch
- **Hệ thống legacy không có API:** BCCP, TMS, WMS nhiều khả năng không có REST API chuẩn — cần giải pháp batch/file-based làm chậm thời gian cập nhật dữ liệu
- **Đặc thù COD:** 70–80% giao dịch là COD — đây là dữ liệu tài chính nhạy cảm; pipeline tích hợp với PayPost sẽ cần quy trình phê duyệt bảo mật phức tạp hơn

### Khi vận hành CDP

- **Chất lượng địa chỉ kém:** Địa chỉ Việt Nam không chuẩn hóa, viết tắt, không có mã bưu điện đầy đủ → khó phân tích theo vùng địa lý; VPostCode giúp nhưng không giải quyết hoàn toàn
- **COD fraud gây nhiễu dữ liệu:** Địa chỉ giả, đơn hàng ảo làm sai lệch phân tích hành vi khách hàng
- **Seasonal spikes:** Dịp Tết, 11/11, 12/12 tạo ra lượng dữ liệu đột biến 5–10x bình thường — pipeline tích hợp cần có khả năng co giãn
- **Governance phức tạp:** Ai được xem dữ liệu khách hàng của tỉnh nào? Đơn vị tỉnh A có được xem dữ liệu KH của tỉnh B không?

---

## 9. Glossary — Thuật ngữ nghiệp vụ VNPost và CDP

| Thuật ngữ VNPost | Giải thích | Liên kết CDP |
|---|---|---|
| **Bưu gửi** | Vật phẩm được giao nhận qua hệ thống bưu chính (thư, gói hàng, kiện hàng) | Tương đương "đơn hàng" (order) — tạo ra event trong CDP |
| **Vận đơn / Mã vận đơn** | Chứng từ và mã theo dõi duy nhất của một bưu gửi | Tracking number — ID kết nối dữ liệu xuyên hệ thống |
| **Bưu tá** | Nhân viên VNPost thực hiện thu gom và phát bưu gửi | Internal actor — dữ liệu GPS bưu tá là dữ liệu nhạy cảm |
| **COD (Cash on Delivery)** | Thu tiền hộ người bán khi phát hàng đến tay người nhận | Nguồn dữ liệu tài chính quan trọng nhất trong CDP |
| **Chuyển hoàn** | Bưu gửi không phát được, trả lại người gửi | Chỉ số chất lượng — high return rate là dấu hiệu churn |
| **KHL** | Khách Hàng Lớn — doanh nghiệp TMĐT gửi khối lượng cao | Phân khúc khách hàng quan trọng nhất cần giữ chân |
| **Bưu cục** | Điểm giao dịch/tiếp nhận/phát của VNPost (~13.000 điểm) | Touchpoint vật lý — dữ liệu giao dịch tại quầy |
| **BD6, BD10, BD13** | Bảng kê giao nhận nội bộ (dùng trong khai thác và phát) | Dữ liệu vận hành nội bộ, ít liên quan trực tiếp đến CDP |
| **MPITS** | Modernization of Postal IT Systems — nền tảng IT tích hợp trung tâm của VNPost | Likely là hub dữ liệu quan trọng nhất để CDP kết nối |
| **PNS** | Pack and Send — hệ thống quản lý phát tuyến, điều phối bưu tá | Nguồn dữ liệu kết quả phát hàng real-time |
| **DingDong** | App smartphone cho bưu tá, đồng bộ với PNS | Mobile data source cho kết quả phát, GPS, COD |
| **CAS** | Counter Acceptance System — hệ thống chấp nhận gửi tại quầy | Nguồn dữ liệu chính về giao dịch tại bưu cục |
| **BCCP** | Phần mềm quản lý dịch vụ bưu chính — khai thác, chia chọn, chuyến thư | Legacy system — khó tích hợp real-time |
| **TMS** | Transportation Management System | Dữ liệu vận tải, ít liên quan trực tiếp đến KH |
| **WMS** | Warehouse Management System | Dữ liệu kho, liên quan khi phân tích hàng tồn/chuyển hoàn |
| **PayPost** | Hệ thống thanh toán COD | Nguồn dữ liệu tài chính nhạy cảm |
| **VPostCode** | Hệ thống địa chỉ số Việt Nam | Chuẩn hóa địa chỉ — quan trọng cho quality của dữ liệu CDP |
| **PostID** | Hệ thống định danh người dùng VNPost | Identity anchor — nền tảng cho Identity Resolution |

---

## 10. Câu hỏi clarify cần hỏi BA/PO

### CRITICAL — Phải có trước khi thiết kế solution

**Q1. TCT đã có Customer ID thống nhất toàn hệ thống chưa, hay mỗi hệ thống đang dùng định danh riêng (mã KHL trong CAS, user ID trong MyVNPost, mã tài chính trong PayPost)?**
> Lý do: Nền tảng của toàn bộ Identity Resolution. Nếu chưa có ID thống nhất, bài toán ghép nối định danh phải được giải quyết song song với CDP — ảnh hưởng trực tiếp đến kiến trúc và timeline.

**Q2. PostID hiện đang đóng vai trò gì? Toàn bộ người dùng VNPost có PostID không, hay chỉ người dùng đã đăng ký app MyVNPost?**
> Lý do: Nếu PostID là định danh chuẩn toàn hệ thống, đây là điểm neo (anchor) lý tưởng cho Identity Resolution. Nếu chỉ phủ app users, vẫn còn một phần lớn khách hàng giao dịch qua quầy không có PostID — cần xác định cách xử lý phần này.

**Q3. MPITS đã expose API ra ngoài cho sàn TMĐT và đối tác kết nối chưa? Nếu có, team IT có thể cung cấp danh sách endpoint và loại dữ liệu trả về không?**
> Lý do: Nếu MPITS đã có API gateway, CDP có thể kết nối qua đó thay vì tích hợp từng hệ thống con — giảm 70% công sức tích hợp. Đây là câu hỏi kỹ thuật có tác động lớn nhất đến kiến trúc CDP.

**Q4. Với dữ liệu nhạy cảm (vị trí GPS, dữ liệu COD/tài chính): VNPost đã có chính sách bảo vệ dữ liệu cá nhân theo Nghị định 13/2023/NĐ-CP chưa? Đã làm DPIA cho dự án CDP này chưa?**
> Lý do: Không thể thiết kế CDP mà không có phương án xử lý compliance NĐ 13 từ đầu. Consent Management, Right to be Forgotten, và audit trail phải là yêu cầu phi chức năng bắt buộc.

**Q5. Use case ưu tiên cho MVP là gì trong số: (a) Customer 360 — hồ sơ hợp nhất, (b) Anti-Churn — cảnh báo shipper TMĐT có nguy cơ rời đi, (c) COD fraud detection, (d) Cá nhân hóa trên MyVNPost App?**
> Lý do: Mỗi use case yêu cầu tập dữ liệu và kiến trúc khác nhau. Nếu không ưu tiên rõ ràng, MVP sẽ cố gắng làm tất cả và không làm được gì tốt.

---

### MAJOR — Ảnh hưởng lớn đến scope và thiết kế

**Q6. Ai là "khách hàng" trong CDP này — người gửi (shipper), người nhận (consignee), hay cả hai? VNPost muốn xây hồ sơ cho cả người nhận hay chỉ người gửi?**
> Lý do: Đây là quyết định phạm vi quan trọng đặc thù bưu chính. Người nhận xuất hiện trong hàng triệu giao dịch nhưng không có tài khoản VNPost — việc xây hồ sơ cho họ có giá trị không và có consent không?

**Q7. Mô hình đầu tư: tự xây trên Unomi, mua giải pháp đóng gói, hay thuê triển khai? Quyết định này đã được chốt chưa?**
> Lý do: Quyết định ảnh hưởng toàn bộ kiến trúc, chi phí, và timeline. Nếu chưa chốt, cần clarify trước khi thiết kế solution.

**Q8. Các hệ thống như BCCP, TMS, WMS có API không? Nếu không, team IT có thể cho phép read-only database access cho CDP không?**
> Lý do: Xác định phương thức tích hợp thực tế với các hệ thống legacy — ảnh hưởng trực tiếp đến tần suất cập nhật dữ liệu (real-time vs batch) và chất lượng hồ sơ khách hàng.

**Q9. CDP này phục vụ cấp nào: toàn TCT (tất cả các tỉnh/thành), hay pilot tại một số đơn vị trước? Có phân quyền dữ liệu theo tỉnh/vùng không?**
> Lý do: Ảnh hưởng đến kiến trúc multi-scope và mô hình phân quyền. Đơn vị tỉnh A có được xem dữ liệu KH của tỉnh B không?

**Q10. Tại sao dự án CDP được khởi động lúc này — có pain point kinh doanh cụ thể nào đang xảy ra không (sụt giảm thị phần TMĐT, tỷ lệ hoàn cao, không đo được ROI chiến dịch, mất khách hàng lớn)?**
> Lý do: Hiểu lý do thật sự giúp BA ưu tiên đúng scope và không đề xuất giải pháp rộng hơn mức cần thiết.

---

### MINOR — Có thể assume tạm, hỏi để xác nhận

**Q11. "Lịch sử định vị" trong yêu cầu ban đầu — là GPS từ app MyVNPost hay từ dữ liệu phát hàng của bưu tá? Ai sở hữu dữ liệu này — người dùng app hay bưu tá?**
> Lý do: Ảnh hưởng đến phân loại dữ liệu nhạy cảm theo NĐ 13 và thiết kế consent flow.

**Q12. VNPost đã có Data Team (Data Engineer, Data Analyst) chưa? Họ sẽ vận hành CDP sau khi xây xong hay cần đào tạo từ đầu?**
> Lý do: Ảnh hưởng đến yêu cầu self-service UX và kế hoạch đào tạo handover.

**Q13. Kỳ vọng go-live phase đầu là bao lâu? Có deadline cứng nào không (sự kiện, fiscal year, cam kết ban lãnh đạo)?**
> Lý do: Ảnh hưởng đến cách phân kỳ và phạm vi MVP.

---

## Assumptions được nêu rõ

- `[Assumption]` TCT ở đây là Tổng Công Ty Bưu Điện Việt Nam (VNPost) — toàn bộ phân tích trong file này dựa trên bối cảnh VNPost cụ thể.
- `[Assumption]` PostID là hệ thống định danh tập trung của VNPost — nhưng chưa rõ độ phủ (bao nhiêu % khách hàng đã có PostID). Cần xác nhận.
- `[Assumption]` MPITS đã kết nối API với sàn TMĐT (dựa trên thông tin công khai) — nhưng chưa rõ API này có thể dùng được cho CDP không. Cần xác nhận với IT VNPost.
- `[Assumption]` Đánh giá khả năng tích hợp trong Mục 6 là ước tính dựa trên thông tin công khai và đặc thù loại hệ thống — chưa được IT VNPost xác nhận.
- `[Assumption]` Apache Unomi được cân nhắc làm nền tảng kỹ thuật — chưa rõ đây là quyết định chốt hay chỉ là tham khảo. Cần xác nhận.
- `[Assumption]` Bối cảnh pháp lý áp dụng là Việt Nam — Nghị định 13/2023/NĐ-CP là khung pháp lý chính.

---

## 11. Đánh giá Apache Unomi và các CDP tương tự

### 11.1 Apache Unomi — Phân tích chuyên sâu

**Tổng quan và trạng thái dự án**

Apache Unomi là CDP mã nguồn mở đầu tiên được phát triển dưới sự quản lý của Apache Software Foundation. Phiên bản hiện tại là **v3.0.0** (phát hành tháng 11/2025), là bản nâng cấp lớn: nâng cấp lên Apache Karaf mới nhất và Elasticsearch v9 (quan trọng vì Elasticsearch 7 hết hỗ trợ tháng 01/2026). Bản 3.1 đang phát triển với multi-tenancy và OpenSearch 3. Dự án đang active, có cộng đồng đóng góp đều đặn. License: **Apache 2.0** — có thể dùng miễn phí cho mọi mục đích bao gồm thương mại, không cần mua license, không giới hạn số node.

**Ý nghĩa Apache 2.0 với VNPost:** Không tốn chi phí bản quyền, toàn quyền chỉnh sửa source code, dữ liệu nằm hoàn toàn trên hạ tầng nội bộ VNPost — đây là điểm phù hợp với yêu cầu data sovereignty của tổ chức nhà nước.

**Kiến trúc và tech stack**

| Thành phần | Chi tiết |
|---|---|
| Ngôn ngữ | Java (JDK 17 LTS) |
| Runtime | Apache Karaf (OSGi) |
| Database chính | Elasticsearch v9 (hoặc OpenSearch) |
| Action scripting | Groovy |
| Plugin system | OSGi bundles + JSON descriptors |
| Giao tiếp | REST API (JSON) — đầy đủ CRUD cho profiles, events, segments, rules |
| Triển khai | Docker (khuyến nghị) hoặc cài thủ công |

Hạ tầng tối thiểu cho production: ít nhất 3 node Elasticsearch (cluster), 1–3 node Unomi (tuỳ tải), RAM 8GB+/node. Elasticsearch là nút thắt cổ chai — hiệu năng Unomi phụ thuộc trực tiếp vào quy mô cluster Elasticsearch.

**Tính năng cốt lõi**

- **Profile Unification:** Thu thập dữ liệu từ web, mobile, CRM, IoT, bất kỳ hệ thống nào có REST API — ghép nối thành hồ sơ thống nhất, hỗ trợ Identity Resolution qua nhiều định danh
- **Segmentation & Scoring:** Xây dựng phân khúc khán giả động theo thời gian thực; tính điểm khách hàng dựa trên rule
- **Event Tracking:** Ghi nhận sự kiện từ mọi kênh digital; xử lý real-time (mỗi event kích hoạt rule ngay lập tức)
- **REST API:** Đầy đủ endpoint cho profiles, events, segments, rules, scoring, campaigns, goals — tích hợp được với bất kỳ ngôn ngữ nào
- **Plugin Architecture:** Mở rộng bằng Groovy actions, OSGi bundles, JSON descriptors — có thể custom sâu
- **AI Integration (v3.0):** Tích hợp OpenAI, Anthropic, và các mô hình AI khác qua API

**Consent Management — điểm đặc biệt quan trọng với NĐ 13**

Unomi có module Consent Management tích hợp sẵn, không cần mua thêm:
- **Consent API:** Ghi nhận consent đã chấp nhận/từ chối của từng visitor; tự động nhúng vào profile người dùng
- **Consent Expiration:** Đặt ngày hết hạn cho từng loại consent, tự động yêu cầu renew
- **Preference Centers:** Tạo trang quản lý consent cho người dùng tự điều chỉnh
- **Right to be Forgotten:** Xóa dữ liệu theo yêu cầu (GDPR-compliant — tương đương NĐ 13)
- **Data Anonymization:** Ẩn danh hóa dữ liệu khi cần
- **Audit Trail:** Ghi lại lịch sử consent thay đổi

Đây là CDP duy nhất trong danh sách có consent management ở cấp độ platform, không phải tính năng bổ sung tốn phí.

**Khả năng scale thực tế**

- Đã xử lý được hơn 20 triệu profiles trong các triển khai thực tế (theo phản hồi cộng đồng)
- Không có benchmark chính thức về events/giây — độ phức tạp CDP làm con số này rất khó đo chuẩn
- Scale chủ yếu qua việc mở rộng Elasticsearch cluster — phương ngang (horizontal scaling)
- `[Lưu ý]` Với quy mô VNPost (~13.000 điểm phục vụ, hàng chục triệu giao dịch/tháng), cần benchmark cụ thể trước khi chốt kiến trúc

**Tích hợp với hệ thống ngoài**

- **CMS/DXP:** Jahia jExperience, Drupal, Liferay, WordPress (qua plugin)
- **Kafka:** Không có connector chính thức sẵn có — cần custom (Kafka → Unomi qua REST API hoặc viết Kafka consumer riêng)
- **Elasticsearch:** Tích hợp native — đây là database chính của Unomi
- **OpenSearch:** Hỗ trợ từ v3.1 (đang phát triển)
- **Hệ thống tùy chỉnh:** Qua REST API — phù hợp với các hệ thống VNPost có sẵn (MPITS nếu đã có REST API)

**Điểm mạnh thực tế**

- Mã nguồn mở, Apache License — không tốn chi phí license, toàn quyền kiểm soát
- Consent management tích hợp sẵn — phù hợp NĐ 13
- Dữ liệu nằm hoàn toàn trên hạ tầng nội bộ — không lo data sovereignty
- REST API đầy đủ — tích hợp được với mọi hệ thống
- Cộng đồng Apache uy tín, dự án đang active, có roadmap rõ ràng
- Khả năng custom cao — phù hợp với đặc thù nghiệp vụ bưu chính không có sẵn trong các giải pháp đóng gói

**Điểm yếu thực tế**

- **Độ phức tạp triển khai: HARD** — yêu cầu đội IT có kinh nghiệm Java, Elasticsearch, OSGi/Karaf; không phải "cài xong dùng được ngay"
- **Cộng đồng nhỏ:** Ít tài liệu tiếng Việt/Đông Nam Á; support chủ yếu qua mailing list, phản hồi không nhanh
- **Không có giao diện marketing self-service hoàn chỉnh** — đội Marketing sẽ cần IT hỗ trợ để tạo campaign, segment; không "kéo thả" dễ dàng như Segment hay Antsomi
- **Không có connector sẵn cho hệ thống Việt Nam** (Zalo OA, MISA, các ERP nội địa) — phải custom toàn bộ
- **Kafka integration không có sẵn** — cần tự xây nếu VNPost muốn dùng event streaming
- **Ít reference customer ở APAC** — khó tìm case study tương tự để tham khảo kiến trúc
- **Không có managed service option** — VNPost phải tự vận hành toàn bộ (patching, backup, scaling)

**Nguồn:** [Apache Unomi Official](https://unomi.apache.org/), [Unomi v3.0 Release](https://unomi.apache.org/), [Consent Management Guide](https://www.tothenew.com/blog/navigating-trust-a-guide-to-consent-management-with-apache-unomi/), [Deployment Challenges](https://www.tothenew.com/blog/setting-the-stage-navigating-the-apache-unomi-setup-landscape-and-overcoming-challenges/), [G2 Reviews](https://www.g2.com/products/apache-unomi/reviews)

---

### 11.2 So sánh 4 CDP tương tự trên thị trường

#### A. Twilio Segment — CDP SaaS phổ biến nhất toàn cầu

| Tiêu chí | Chi tiết |
|---|---|
| Loại | SaaS Cloud — dữ liệu lưu trên hạ tầng Twilio (Mỹ) |
| Điểm nổi bật | 700+ tích hợp sẵn; Identity Resolution mạnh; 4 sản phẩm riêng: Connections, Protocols, Unify, Engage |
| Tính năng AI | AI audiences, churn/conversion prediction, natural language campaign creation |
| Pricing | Free (1.000 MTU), Team ~$120/tháng (10K MTU), Business/Enterprise custom ($25K–$500K+/năm) |
| Phù hợp VNPost? | **Không khuyến nghị làm lựa chọn chính** — dữ liệu ra ngoài lãnh thổ Việt Nam (vi phạm tinh thần NĐ 13 nếu có dữ liệu nhạy cảm), không hỗ trợ on-premise, vendor lock-in cao |
| Nguồn | [Segment Pricing](https://www.twilio.com/en-us/pricing/customer-data), [CDP.com Review](https://cdp.com/articles/what-is-twilio-segment/) |

#### B. mParticle (by Rokt) — CDP enterprise mạnh về mobile/app

| Tiêu chí | Chi tiết |
|---|---|
| Loại | SaaS Cloud — được Rokt mua lại tháng 01/2025 với giá $300 triệu |
| Điểm nổi bật | Mobile-first; IDSync (identity resolution mạnh); Data Planning (data quality enforcement); Cortex AI predictions; 300+ tích hợp |
| Pricing | Enterprise — $50K–$500K+/năm; dựa trên Monthly Active Users (MAU) |
| Phù hợp VNPost? | **Không phù hợp** — tập trung mobile-first trong khi VNPost có nhiều giao dịch qua quầy; giá quá cao với ngân sách tổng công ty nhà nước; dữ liệu ra ngoài |
| Nguồn | [mParticle CDP.com](https://cdp.com/articles/what-is-mparticle/), [Pricing Overview](https://www.mparticle.com/pricing/) |

#### C. RudderStack — Open Source CDP warehouse-first

| Tiêu chí | Chi tiết |
|---|---|
| Loại | Dual-license open source (AGPL-3.0 server / MIT SDKs) — warehouse-first approach |
| Điểm nổi bật | Chạy trên data warehouse sẵn có (Snowflake, BigQuery, Databricks, Redshift); không lưu dữ liệu tại RudderStack; 11 SDK; 60+ destinations; Reverse ETL |
| Pricing | Free (1M events/tháng), Starter $500/tháng (3M events), Growth custom |
| Phù hợp VNPost? | **Có tiềm năng nếu VNPost đã có data warehouse** — dữ liệu không rời khỏi hạ tầng VNPost; developer-friendly; nhưng không có Consent Management sẵn; ít phù hợp nếu chưa có data warehouse chuẩn |
| Nguồn | [RudderStack CDP.com](https://cdp.com/articles/what-is-rudderstack/), [RudderStack Pricing](https://www.rudderstack.com/pricing/) |

#### D. Antsomi CDP 365 — CDP nội địa Đông Nam Á

| Tiêu chí | Chi tiết |
|---|---|
| Loại | SaaS + On-premise (hybrid) — công ty Singapore, có văn phòng Việt Nam |
| Điểm nổi bật | "CDP AI đầu tiên native Đông Nam Á"; hỗ trợ cả SaaS và on-premise; tích hợp kênh Đông Nam Á (Zalo, Line, v.v.); ERFM model; giải thưởng VECOM 2021 |
| Pricing | Không công bố công khai — "Hybrid Pricing": phí cố định + phí theo hiệu suất |
| Phù hợp VNPost? | **Có tiềm năng cao nhất trong nhóm SaaS/thương mại** — hỗ trợ on-premise (giải quyết data sovereignty), am hiểu thị trường Đông Nam Á, có đội hỗ trợ Việt Nam. Tuy nhiên chưa có case study logistics/bưu chính rõ ràng, giá cần thương lượng trực tiếp |
| Nguồn | [Antsomi CDP 365](https://www.antsomi.com/antsomi-cdp-365/), [Vietnam MarTech Day 2024](https://www.antsomi.com/2024/11/01/vietnam-martech-day-2024-antsomi-champions-martech-innovation-in-southeast-asia/) |

#### E. Treasure AI (cũ: Treasure Data) — CDP enterprise APAC

| Tiêu chí | Chi tiết |
|---|---|
| Loại | SaaS Enterprise — đã đổi tên thành Treasure AI, định vị là "AI Marketing Cloud" từ 2025 |
| Điểm nổi bật | Mạnh APAC; partnership với ADA phủ 10 thị trường SEA; AI Agent Hub; Engagement AI Suite (email, SMS, mobile); dành cho doanh nghiệp lớn |
| Phù hợp VNPost? | **Không khuyến nghị** — giá enterprise rất cao (không public, nhưng nằm trong top tier); dữ liệu chủ yếu cloud nước ngoài; đang chuyển hướng sang AI Marketing Cloud, ít phù hợp với use case CDP thuần của VNPost |
| Nguồn | [Treasure AI APAC](https://www.treasuredata.com/the-cdp-industry-in-apac/), [CDP.com Review](https://cdp.com/articles/what-is-treasure-data/), [Mission Media APAC CDP 2026](https://missionmedia.asia/12-customer-data-platforms-asia-pacific-2026/) |

---

### 11.3 Bảng so sánh nhanh

| Tiêu chí | Apache Unomi | Twilio Segment | mParticle | RudderStack | Antsomi CDP 365 |
|---|---|---|---|---|---|
| License | Apache 2.0 (miễn phí) | SaaS trả phí | SaaS trả phí | AGPL/MIT | SaaS + On-prem |
| Dữ liệu nằm ở đâu | Hạ tầng VNPost | Cloud Mỹ | Cloud Mỹ | Warehouse VNPost hoặc Cloud | Cloud SEA hoặc On-prem |
| Consent Management | Sẵn có (tích hợp) | Cần cấu hình thêm | Cần cấu hình thêm | Không sẵn | Có (mức độ chưa rõ) |
| Phù hợp NĐ 13 | Cao (self-hosted) | Thấp (data ra ngoài) | Thấp (data ra ngoài) | Trung bình | Trung bình–Cao |
| Độ phức tạp triển khai | HARD | EASY–MEDIUM | MEDIUM | MEDIUM | EASY–MEDIUM |
| Tùy chỉnh nghiệp vụ | Rất cao | Trung bình | Trung bình | Trung bình | Trung bình |
| Chi phí năm 1 (ước tính) | Thấp (license miễn phí) nhưng cao về nhân lực | $100K–$400K/năm | $200K–$500K+/năm | $6K–$20K/năm (+ nhân lực) | Cần thương lượng |
| Cộng đồng Việt Nam | Rất ít | Không | Không | Ít | Có (văn phòng VN) |
| Reference APAC logistics | Không rõ | Có (nhưng ít) | Có (ít) | Không | Chưa rõ |

---

## 12. Phân tích phương án triển khai (Build / Buy / Partner)

### 12.1 Bối cảnh ra quyết định — VNPost cụ thể

Trước khi phân tích từng phương án, cần ghi nhận các ràng buộc đặc thù của VNPost ảnh hưởng trực tiếp đến quyết định:

- **Tổng công ty nhà nước:** Mua sắm phần mềm phải qua đấu thầu; ngân sách phê duyệt theo năm tài chính; quyết định lớn cần nhiều cấp phê duyệt
- **Data sovereignty bắt buộc:** Dữ liệu khách hàng (đặc biệt dữ liệu nhạy cảm theo NĐ 13) không thể lưu trên cloud nước ngoài — loại ngay các giải pháp SaaS cloud Mỹ
- **Hệ thống IT đặc thù:** MPITS, BCCP, CAS, PayPost là các hệ thống riêng của VNPost — không có connector sẵn trong bất kỳ CDP đóng gói nào; phần tích hợp luôn cần custom
- **Năng lực IT nội bộ:** Chưa rõ VNPost có đủ Data Engineer / Java Engineer để tự vận hành Unomi không (cần xác nhận)
- **Ưu tiên tuân thủ:** NĐ 13 không optional — Consent Management và audit trail phải là yêu cầu phi chức năng bắt buộc từ ngày 1

---

### 12.2 Option 1 — Build on Apache Unomi (Self-hosted open source)

**Mô tả:** VNPost (hoặc đơn vị CNTT trực thuộc) tự triển khai, cấu hình, và vận hành Unomi trên hạ tầng nội bộ VNPost. Toàn bộ code, data, hạ tầng thuộc VNPost.

**Chi phí ước tính:**

| Hạng mục | Ước tính |
|---|---|
| License Unomi | 0 đồng (Apache 2.0) |
| Hạ tầng (server/cloud nội bộ) | 200–500 triệu VND/năm |
| Nhân lực triển khai ban đầu (3–6 tháng, 2–3 kỹ sư) | 500 triệu – 1,2 tỷ VND |
| Nhân lực vận hành ongoing (1–2 kỹ sư) | 400–800 triệu VND/năm |
| Custom connector cho 5–8 hệ thống VNPost | 500 triệu – 1,5 tỷ VND (một lần) |
| Tổng năm đầu (ước tính) | **1,5–3,5 tỷ VND** |
| Tổng năm 2 trở đi | **600 triệu – 1,3 tỷ VND/năm** |

**Timeline ước tính:**
- MVP (3 hệ thống tích hợp, Customer 360 cơ bản): 6–9 tháng
- Full platform (8 hệ thống, tất cả use cases): 12–18 tháng

**Rủi ro chính:**
- Năng lực IT nội bộ: Nếu VNPost không có Java Engineer + Elasticsearch Engineer, phải tuyển hoặc đào tạo — mất thêm 3–6 tháng
- Maintenance dài hạn: Nếu đội IT bận với vận hành hệ thống hiện tại, CDP sẽ bị bỏ quên sau khi go-live
- Không có vendor support: Khi có sự cố nghiêm trọng, phải tự debug hoặc chờ cộng đồng phản hồi
- Version upgrade: Elasticsearch migration (e.g., v7 → v9) là công việc kỹ thuật phức tạp, đã từng gây gián đoạn ở nhiều tổ chức

**Phù hợp khi:**
- VNPost có đội IT mạnh với ít nhất 2–3 kỹ sư Java/Elasticsearch sẵn sàng
- Ưu tiên kiểm soát hoàn toàn dữ liệu và code
- Ngân sách eo hẹp nhưng timeline có thể kéo dài 12+ tháng
- Cần tùy chỉnh sâu theo nghiệp vụ bưu chính đặc thù

---

### 12.3 Option 2 — Buy SaaS CDP (Thuê ngoài hoàn toàn)

**Mô tả:** VNPost mua subscription CDP SaaS từ nhà cung cấp thương mại. Hạ tầng và vận hành do vendor lo; VNPost chỉ cần cấu hình và tích hợp API.

**Lưu ý quan trọng:** Hầu hết CDP SaaS phổ biến (Segment, mParticle, Treasure Data) lưu dữ liệu trên cloud quốc tế (AWS/GCP/Azure tại Mỹ hoặc châu Âu) — vi phạm tinh thần NĐ 13 với dữ liệu nhạy cảm. **Phương án SaaS cloud quốc tế gần như bị loại ngay từ yêu cầu data sovereignty.**

**Ngoại lệ khả thi:** Antsomi CDP 365 có option on-premise/private cloud — nếu deploy trên hạ tầng VNPost thì không vi phạm NĐ 13.

**Chi phí ước tính (Antsomi CDP 365 on-prem / SaaS khu vực):**

| Hạng mục | Ước tính |
|---|---|
| License/subscription năm đầu | 500 triệu – 2 tỷ VND (cần thương lượng) |
| Triển khai và tích hợp (vendor thực hiện) | 300–800 triệu VND |
| Đào tạo và onboarding | 50–150 triệu VND |
| Tổng năm đầu (ước tính) | **850 triệu – 3 tỷ VND** |
| Subscription ongoing | 500 triệu – 1,5 tỷ VND/năm |

**Timeline ước tính:**
- MVP: 3–5 tháng (nhanh hơn nhiều vì hạ tầng sẵn có)
- Triển khai đầy đủ: 6–9 tháng

**Rủi ro chính:**
- **Data sovereignty:** Phải yêu cầu cam kết rõ ràng bằng văn bản rằng toàn bộ dữ liệu ở trên hạ tầng VNPost hoặc hạ tầng đặt tại Việt Nam
- **Vendor lock-in:** Sau 3–5 năm, nếu muốn đổi vendor, toàn bộ logic segment, campaign, và tích hợp phải build lại từ đầu
- **Tùy chỉnh hạn chế:** Nghiệp vụ đặc thù của bưu chính (phân tích hành vi giao nhận theo tuyến, COD fraud detection) có thể không được hỗ trợ sẵn trong platform
- **Giá tăng theo thời gian:** Sau year 1, vendor thường tăng giá theo volume dữ liệu hoặc số lượng tính năng

**Phù hợp khi:**
- VNPost cần go-live nhanh (áp lực deadline từ ban lãnh đạo)
- Đội IT không đủ năng lực vận hành open source
- Ưu tiên "mua dịch vụ" hơn "xây hạ tầng" theo chiến lược outsourcing hiện tại
- Ngân sách hàng năm đủ lớn và ổn định

---

### 12.4 Option 3 — Partner với SI/Vendor (Thuê triển khai)

**Mô tả:** VNPost thuê System Integrator (SI) Việt Nam hoặc khu vực để triển khai CDP — có thể là Unomi, Antsomi, hoặc giải pháp custom. VNPost sở hữu kết quả cuối cùng; SI chịu trách nhiệm deliverable.

Đây là phương án phổ biến nhất với các tổng công ty nhà nước Việt Nam vì: phù hợp quy trình đấu thầu, rủi ro kỹ thuật chuyển sang SI, và VNPost không cần xây đội IT mới.

**Chi phí ước tính:**

| Hạng mục | Ước tính |
|---|---|
| Project fee (build + triển khai) | 2–6 tỷ VND (tùy scope) |
| License nền tảng (nếu dùng Unomi: 0đ; nếu dùng giải pháp khác: cộng thêm) | 0 – 1 tỷ VND |
| Annual support & maintenance | 300 triệu – 1 tỷ VND/năm |
| Tổng năm đầu (ước tính) | **2,3–7 tỷ VND** |
| Ongoing (năm 2+) | **300 triệu – 1 tỷ VND/năm** (nếu VNPost tự vận hành được) |

**Timeline ước tính:**
- Đấu thầu chọn SI: 2–3 tháng
- MVP: 6–9 tháng sau khi ký hợp đồng
- Full delivery: 12–15 tháng

**Rủi ro chính:**
- **Chất lượng SI:** SI Việt Nam chưa có nhiều kinh nghiệm triển khai CDP quy mô lớn — cần đánh giá kỹ reference projects
- **Knowledge transfer:** Sau khi SI bàn giao, đội IT VNPost có đủ năng lực vận hành không? Nếu không, sẽ tiếp tục phụ thuộc SI
- **Scope creep:** Hợp đồng cần định nghĩa scope rất rõ; với hệ thống phức tạp như VNPost, risk change request cao
- **Timeline bị kéo dài:** Dự án IT phức tạp tại Việt Nam thường trễ 30–50% so với kế hoạch ban đầu

**Phù hợp khi:**
- VNPost không có đội IT chuyên về CDP
- Muốn kiểm soát sở hữu code và dữ liệu (tốt hơn Option 2)
- Chấp nhận timeline dài hơn để đổi lấy độ an toàn cao hơn
- Có ngân sách project một lần (CAPEX) thay vì chi phí hàng năm liên tục (OPEX)

---

### 12.5 Khuyến nghị BA

**Đánh giá tổng hợp từ góc độ BA (dựa trên context VNPost):**

Dựa trên 4 ràng buộc cứng: (1) data sovereignty, (2) hệ thống legacy cần custom connector, (3) yêu cầu compliance NĐ 13 có consent management, và (4) quy trình mua sắm nhà nước — **Option 3 (Partner + Unomi) là phương án cân bằng nhất** cho VNPost, với lý do:

| Lý do | Giải thích |
|---|---|
| Unomi miễn phí license | Không tốn chi phí bản quyền — phù hợp với ngân sách nhà nước, không bị reject ở khâu phê duyệt license |
| Consent Management sẵn có | Đáp ứng NĐ 13 ngay từ platform, không cần build thêm |
| Dữ liệu tại VNPost | Self-hosted — giải quyết data sovereignty hoàn toàn |
| Custom connector khả thi | SI có thể viết connector cho MPITS, CAS, PayPost theo spec VNPost |
| VNPost không cần đội Unomi chuyên sâu ngay | SI làm trước, knowledge transfer sau — phù hợp năng lực hiện tại |
| Sở hữu sau khi bàn giao | VNPost sở hữu hoàn toàn, không vendor lock-in dài hạn |

**Điều kiện để Option 3 thành công:**
1. Chọn SI có thực chiến với Apache Unomi và Elasticsearch — phải yêu cầu demo POC (Proof of Concept) với 1–2 hệ thống nguồn của VNPost
2. Hợp đồng phải bao gồm Knowledge Transfer rõ ràng: đào tạo đội IT VNPost đến mức tự vận hành được
3. Định nghĩa rõ Acceptance Criteria trước khi ký: tốc độ xử lý, số profile, thời gian real-time, uptime SLA

**Khi nào nên xem xét Option 1 (Tự build):**
Nếu VNPost đã có hoặc sẵn sàng xây đội **Data Platform Engineer** (2–3 người Java/Elasticsearch) và timeline có thể kéo 12–18 tháng — Option 1 rẻ hơn về dài hạn.

**Khi nào nên xem xét Option 2 (Antsomi):**
Nếu áp lực deadline cứng (ví dụ: phải demo trước sự kiện lớn trong 6 tháng tới) và đội IT VNPost mỏng — Antsomi CDP 365 on-prem là phương án nhanh nhất.

`[Assumption]` Khuyến nghị trên dựa trên thông tin công khai và phân tích từ bối cảnh VNPost. Một số yếu tố có thể thay đổi kết luận: ngân sách thực tế được phê duyệt, năng lực IT nội bộ thực tế (chưa được xác nhận), và timeline go-live do ban lãnh đạo đặt ra.

---

### 12.6 Agenda gợi ý cho buổi meeting chốt phương án

**Mục tiêu buổi meeting:** Chốt 1 trong 3 phương án triển khai và xác định điều kiện tiên quyết để bắt đầu phase tiếp theo.

**Thời lượng đề xuất:** 90 phút

**Thành phần tham dự cần có:**
- Đại diện Ban lãnh đạo / Giám đốc Marketing (người ra quyết định cuối)
- Giám đốc IT / CTO (đánh giá năng lực kỹ thuật)
- Quản lý dự án (nếu có)
- BA (người dẫn dắt agenda)

**Agenda chi tiết:**

| Thứ tự | Nội dung | Thời gian | Người trình bày |
|---|---|---|---|
| 1 | Recap mục tiêu CDP và 5 use cases ưu tiên | 10 phút | BA |
| 2 | Trình bày 3 phương án: ưu/nhược/chi phí/timeline | 20 phút | BA |
| 3 | Q&A kỹ thuật — IT xác nhận năng lực nội bộ | 15 phút | IT Lead |
| 4 | Q&A tài chính — confirm budget range | 10 phút | Lãnh đạo |
| 5 | Thảo luận: ràng buộc nào là cứng, ràng buộc nào có thể linh hoạt? | 20 phút | Toàn bộ |
| 6 | Chốt phương án hoặc thu hẹp xuống 2 phương án còn xem xét | 10 phút | Lãnh đạo quyết định |
| 7 | Xác định next steps cụ thể (POC, RFQ, timeline tuyển SI...) | 5 phút | BA tổng hợp |

**5 câu hỏi BA cần có câu trả lời trước cuối buổi meeting:**

1. **Ngân sách:** VNPost chuẩn bị tối đa bao nhiêu cho dự án CDP (CAPEX một lần + OPEX hàng năm)?
2. **Timeline:** Có deadline cứng nào không (sự kiện, kỳ họp, cam kết ban lãnh đạo)?
3. **Năng lực IT:** Hiện có bao nhiêu kỹ sư Java/Data Engineer sẵn sàng làm CDP? Có kế hoạch tuyển thêm không?
4. **Data sovereignty:** IT xác nhận: toàn bộ dữ liệu CDP phải nằm trên hạ tầng VNPost — không ngoại lệ?
5. **Use case MVP:** Trong 5 use cases đã đề xuất, ban lãnh đạo muốn nhìn thấy kết quả gì đầu tiên trong 6 tháng?

---

## Nguồn tham khảo chính

- [VNPost — BCCP phần mềm quản lý bưu chính](https://vnpost.vn/vi/tin-khac/quyet-dinh-trien-khai-phan-mem-ung-dung-quan-ly-cac-dich-vu-buu-chinh)
- [VNPost — DingDong nâng cấp hệ thống phát](https://vnpost.vn/en/hoat-dong-nganh/vietnam-post-nang-cap-he-thong-phan-mem-quan-li-cong-doan-phat-buu-gui-danh-cho-buu-ta)
- [VNPost — MPITS nền tảng CNTT tối ưu hóa](https://vnpost.vn/en/chuyen-phat-tmdt-logistics/cong-nghe-thong-tin-la-nen-tang-toi-uu-hoa-san-xuat-nang-cao-chat-luong-dich-vu)
- [CDP.com — What is a CDP 2026](https://cdp.com/basics/what-is-a-customer-data-platform-cdp/)
- [CDP.com — Use Cases 2026](https://cdp.com/basics/cdp-use-cases/)
- [CDP.com — Customer Loyalty & Retention](https://cdp.com/articles/customer-loyalty-retention-cdp/)
- [CDP.com — Cross-sell & Upsell](https://www.cdpinstitute.org/resources/cross-sell-and-upsell/)
- [Nghị định 13/2023/NĐ-CP văn bản gốc](https://vanban.chinhphu.vn/?pageid=27160&docid=207759)
- [VNG Cloud — Doanh nghiệp cần làm gì theo NĐ 13](https://vngcloud.vn/vi/blog/key-notes-for-businesses-under-decree-13-2023-nd-cp-on-personal-data-protection)
- [Apache Unomi Official Website](https://unomi.apache.org)
- [Apache Unomi v3.0 Documentation](https://unomi.apache.org/documentation.html)
- [Apache Unomi Consent Management Guide](https://www.tothenew.com/blog/navigating-trust-a-guide-to-consent-management-with-apache-unomi/)
- [Apache Unomi Deployment Challenges](https://www.tothenew.com/blog/setting-the-stage-navigating-the-apache-unomi-setup-landscape-and-overcoming-challenges/)
- [Apache Unomi G2 Reviews 2026](https://www.g2.com/products/apache-unomi/reviews)
- [Twilio Segment CDP Overview](https://cdp.com/articles/what-is-twilio-segment/)
- [Twilio Segment Pricing](https://www.twilio.com/en-us/pricing/customer-data)
- [mParticle CDP Overview](https://cdp.com/articles/what-is-mparticle/)
- [RudderStack CDP Overview](https://cdp.com/articles/what-is-rudderstack/)
- [RudderStack Pricing](https://www.rudderstack.com/pricing/)
- [Antsomi CDP 365 Product Page](https://www.antsomi.com/antsomi-cdp-365/)
- [Treasure AI APAC CDP](https://www.treasuredata.com/the-cdp-industry-in-apac/)
- [12 CDPs APAC 2026 - Mission Media](https://missionmedia.asia/12-customer-data-platforms-asia-pacific-2026/)
- [CDP Build vs Buy Framework 2026](https://branch8.com/posts/cdp-build-vs-buy-decision-framework-2026)
- [CDP Cost Guide 2025](https://www.metacto.com/blogs/what-a-cdp-really-costs-in-2025-for-app-startups)
- [Last Mile Delivery Vietnam](https://fastforwardadvisors.com/last-mile-delivery-in-southeast-asia-and-vietnam/)
- [Vietnam COD statistics](https://www.statista.com/statistics/1275013/vietnam-share-of-online-shoppers-who-used-cod-shipping/)

---

**Bước tiếp theo:** Dùng file này làm input cho `ba-clarification-agent` — tiến hành buổi clarification với BA/PO của VNPost. Ưu tiên hỏi 5 câu CRITICAL (Q1–Q5) vì đây là những điểm ảnh hưởng trực tiếp đến kiến trúc và phạm vi hệ thống. Đặc biệt Q3 (API của MPITS) và Q2 (vai trò PostID) là 2 câu hỏi kỹ thuật mới — câu trả lời sẽ quyết định có cần làm từng tích hợp riêng lẻ hay có thể đi qua một gateway chung.
