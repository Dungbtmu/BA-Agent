# Phân tích sâu Apache Unomi — Dự án CDP VNPost/TCT

**Ngày nghiên cứu:** 2026-06-10  
**Phương pháp:** Deep research — 101 agent, 19 nguồn, 76 claims, verify adversarial 25 claims  
**Mục đích:** Trả lời 4 câu hỏi: Unomi là gì → có tính năng gì → VNPost đáp ứng được gì → cần học thêm gì

> Bài này viết cho BA/PO đọc — không dùng jargon kỹ thuật. Khi cần tra lại bằng chứng kỹ thuật, xem `apache-unomi-research.md`.

---

## Câu hỏi 1 — Apache Unomi là gì?

### Một câu tóm gọn

Unomi là phần mềm mã nguồn mở dùng để **gộp dữ liệu khách hàng từ nhiều nơi về một chỗ**, tự động phân loại khách hàng theo nhóm, và kích hoạt hành động (gửi SMS, gắn nhãn, cảnh báo) khi có sự kiện xảy ra.

Nó do tổ chức Apache (cùng tổ chức tạo ra Apache Kafka, Apache Tomcat) phát triển — mã nguồn mở, miễn phí, có thể chỉnh sửa tùy ý.

### Bộ máy bên trong (không cần hiểu chi tiết, biết để hình dung)

Unomi gồm 4 khối chính:

| Khối | Vai trò | Tương tự với VNPost |
|---|---|---|
| **Hồ sơ khách hàng (Profile)** | Nơi lưu toàn bộ thông tin của 1 người: tên, lịch sử giao dịch, nhãn phân loại, điểm tín nhiệm | Hồ sơ KHL trong CAS — nhưng gộp thêm từ 7 hệ thống khác |
| **Phiên làm việc (Session)** | Ghi lại một lần tương tác cụ thể — ví dụ: lần đặt hàng này, lần vào app này | Một vận đơn cụ thể |
| **Sự kiện (Event)** | Bất kỳ hành động nào xảy ra: giao hàng thất bại, đăng nhập app, tạo đơn | Trạng thái vận đơn thay đổi trong BCCP/PNS |
| **Quy tắc (Rule)** | Định nghĩa: "nếu sự kiện X xảy ra và điều kiện Y đúng → làm Z" | Quy tắc nghiệp vụ của Marketing |

---

## Câu hỏi 2 — Unomi có những tính năng gì?

### Tính năng 1: Gộp nhiều ID về 1 hồ sơ (đã xác minh — độ tin cậy cao)

**Unomi làm thế nào:** Khi một khách hàng xuất hiện ở 2 hệ thống khác nhau (ví dụ: cùng số điện thoại nhưng mã khác nhau), Unomi tạo một "cầu nối" (gọi là **alias**) nối hai mã đó về cùng một hồ sơ chính. Hồ sơ bị trùng sẽ được hợp nhất — dữ liệu gộp vào hồ sơ chính, hồ sơ cũ xóa đi, cầu nối giữ lại để tra cứu sau.

**Quan trọng:** Việc gộp không tự động xảy ra — phải có **quy tắc kích hoạt** (ví dụ: "nếu email trùng nhau thì gộp"). Unomi không tự đoán, không tự ghép ngẫu nhiên.

**Áp dụng cho VNPost:** Đây chính xác là bài toán VNPost đang có — cùng 1 người gửi nhưng có 4 mã khác nhau (CAS mã KHL, MyVNPost user ID, PayPost mã tài chính, PostID). Pattern gộp alias của Unomi là pattern đúng để học.

---

### Tính năng 2: Tự động làm gì đó khi có sự kiện (đã xác minh — độ tin cậy cao)

**Unomi làm thế nào:** Mỗi khi có sự kiện xảy ra (giao hàng thất bại, tạo đơn, đăng nhập), Unomi kiểm tra tất cả quy tắc đang có:

```
Sự kiện xảy ra
   → Kiểm tra điều kiện (có đúng không?)
   → Nếu đúng: thực hiện hành động
   → Trả về hồ sơ khách hàng đã cập nhật
```

Các điều kiện Unomi hỗ trợ: loại sự kiện là gì, thuộc tính sự kiện có thỏa mãn không, hồ sơ khách hàng hiện tại có điều kiện gì không.

Các hành động Unomi hỗ trợ: tăng một con số trong hồ sơ (ví dụ: đếm số lần thất bại), gán giá trị mới vào hồ sơ (ví dụ: gán nhãn "at-risk").

**Điểm mạnh:** Quy tắc được cấu hình bằng file JSON — không cần viết code mới, không cần release hệ thống khi thay đổi ngưỡng.

**Áp dụng cho VNPost:** Bốn use case (UC-02 anti-churn, UC-03 win-back, UC-04 COD risk, UC-06 fraud) đều phù hợp với pattern này. Chi tiết xem bảng trong `apache-unomi-research.md` (Mục T2).

---

### Tính năng 3: Quản lý đồng ý dữ liệu (đã được mô tả trong tài liệu chính thức)

Unomi có sẵn module quản lý đồng ý — người dùng có thể bật/tắt từng loại đồng ý (marketing, phân tích, vị trí, tài chính). Khi yêu cầu xóa dữ liệu, Unomi xóa được trong hệ thống của mình.

**Điểm cần lưu ý với VNPost:**
- Unomi xóa dữ liệu trong chính nó — không tự lan sang CAS, PayPost, BCCP
- VNPost cần thiết kế thêm "bộ lan truyền xóa" để đảm bảo xóa đúng trong 72 giờ theo luật bảo vệ dữ liệu cá nhân mới

---

### Tính năng chưa được xác minh (cần nghiên cứu thêm)

Những tính năng sau **bị bác bỏ trong kiểm tra độ tin cậy** — không phải không tồn tại, mà là chưa có bằng chứng đủ mạnh từ nguồn chính thức:

| Tính năng | Câu hỏi còn mở |
|---|---|
| Phân vùng dữ liệu theo tỉnh/vùng | Unomi Scope có thực sự ngăn dữ liệu tỉnh A rò sang tỉnh B không? Hay chỉ là nhãn phân loại? |
| Tính lại phân khúc tức thì | Khi KHL giảm sản lượng, Unomi có cập nhật nhóm "at-risk" ngay lập tức không, hay chờ đến hôm sau? |
| Danh sách endpoint REST API | Các endpoint cụ thể và cách xác thực chưa được verify đủ tin cậy |

---

## Câu hỏi 3 — Unomi đáp ứng được gì cho VNPost?

### 3.1 Bản đồ dữ liệu VNPost theo hệ thống

> Nguồn: tài liệu gốc từ VNPost (sơ đồ kiến trúc MPITS, chức năng phần mềm, quy trình tổng quan). Đây là dữ liệu thực tế — không phải assumption.

#### Nhóm 1 — Điểm tiếp xúc khách hàng (dữ liệu giá trị nhất)

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **MyVietnam Post** (app) | Đơn hàng, tài khoản, người dùng, đối soát, hỗ trợ, hành vi app | User ID, SĐT, email | T1 alias, T2 event hành vi |
| **vnpost.vn** (web) | Hành vi tìm kiếm, theo dõi đơn, tạo đơn online | Cookie/session, SĐT | T1, T2 |
| **SmartLocker** ⚠️ | Lịch sử lấy hàng tại tủ khóa, thời điểm lấy | SĐT người nhận | T1 alias bổ sung — **chưa rõ có ID riêng hay chỉ dùng SĐT** |
| **PostPay / E-Money** ⚠️ | Lịch sử thanh toán, ví điện tử, COD — **dữ liệu tài chính nhạy cảm** | Mã tài chính, SĐT | T1 alias, T4 consent tài chính bắt buộc |

> ⚠️ SmartLocker và E-Money là 2 hệ thống **phát hiện từ tài liệu gốc VNPost** — chưa có trong Domain Brief v2.1. Cần xác nhận với IT VNPost.

#### Nhóm 2 — Chấp nhận gửi (dữ liệu đầy đủ nhất về người gửi)

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **CAS (Counter Acceptance)** | Họ tên, SĐT, địa chỉ người gửi + nhận, loại dịch vụ, cước phí, COD amount, tra cứu, sự vụ | Mã KHL — **alias chính trong T1** | T1 anchor, T2 event tạo đơn |
| **Portal KHL** | Chấp nhận gửi hàng loạt cho doanh nghiệp, tính cước, in vận đơn | Mã KHL doanh nghiệp | T1, T2 sản lượng KHL |
| **MPITS** | Hub tổng hợp — nhận dữ liệu từ tất cả hệ thống, kết nối sàn TMĐT, ~30 ứng dụng | Hub — không tạo ID riêng | **T3 tầng lõi** — cổng vào dữ liệu quan trọng nhất nếu có API |

#### Nhóm 3 — Thu gom (tần suất + địa chỉ lấy hàng)

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **PNS Thu Gom / DingDong** | Tin thu gom, điều tin, chuyển tuyến, thống kê, báo cáo | SĐT người gửi, địa chỉ | T2 event lấy hàng thực tế |
| **QLTG** | Lịch thu gom, phân công bưu tá | Địa chỉ lấy hàng | Phân tích tần suất gửi |

#### Nhóm 4 — Phát hàng (kết quả — quan trọng nhất cho UC-04, UC-06)

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **PNS Phát / DingDong** | Kết quả phát (thành công/thất bại/hoàn), gạch nợ, vận đơn, sự vụ, chăm sóc KH, tuyến đường thư | SĐT người nhận, mã vận đơn | **T2 trigger chính** (event phát thất bại), T1 hồ sơ người nhận |
| **WMS** | Nhập/xuất hàng, tồn kho bưu gửi, lý do không phát | Mã vận đơn | Phân tích hàng tồn — ít liên quan KH trực tiếp |
| **TMS** | Kế hoạch điều xe, xác nhận chứng từ, giám sát hành trình, báo cáo sản lượng | Mã chuyến thư | Thời gian vận chuyển thực tế — chất lượng dịch vụ |

#### Nhóm 5 — Tài chính COD (nhạy cảm nhất)

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **PayPost** | Thu COD, gạch nợ, trả tiền người gửi, lịch sử tài chính | Mã tài chính, SĐT, số TK ngân hàng | T1 alias tài chính, T4 consent bắt buộc, **UC-04 COD risk** |
| **E-Money** ⚠️ | Ví điện tử, giao dịch thanh toán | Mã ví, SĐT | Bổ sung T4 — **tách biệt PayPost, dữ liệu ở 2 chỗ khác nhau** |

> ⚠️ PayPost và E-Money là 2 hệ thống tài chính **riêng biệt** — dữ liệu COD không nằm ở một chỗ. Cần xác nhận phạm vi từng hệ thống với IT VNPost.

#### Nhóm 6 — Quản lý quan hệ khách hàng ⚠️

| Hệ thống | Dữ liệu KH có | Định danh | Liên quan pattern |
|---|---|---|---|
| **Care Đơn** | Khiếu nại, sự vụ, chăm sóc KH | SĐT, mã vận đơn | T2 tín hiệu churn, input UC-02 |
| **CCP / SalesForce / PostSale** ⚠️ | Quan hệ KH doanh nghiệp, sales pipeline, hồ sơ KHL | Mã KHL doanh nghiệp | Hồ sơ KHL đầy đủ nhất — cần kết nối T1 |
| **Cas-HTKH** ⚠️ | Hỗ trợ KH tại quầy — **tách biệt Care Đơn** | SĐT, mã KHL | Dữ liệu khiếu nại có thể bị phân mảnh giữa 2 hệ thống này |

> ⚠️ CCP/SalesForce/PostSale và Cas-HTKH là 2 hệ thống **phát hiện từ tài liệu gốc** — chưa có trong Domain Brief v2.1. Đặc biệt, có 2 hệ thống hỗ trợ KH riêng biệt → dữ liệu khiếu nại có thể bị phân mảnh.

#### Nhóm 7 — Hệ thống nền (định danh + địa chỉ)

| Hệ thống | Dữ liệu có | Liên quan pattern |
|---|---|---|
| **PostID** | Định danh người dùng VNPost | **T1 master profile ID — điểm neo trung tâm** |
| **VPostCode / Địa chỉ số** | Chuẩn hóa địa chỉ Việt Nam | T1 chuẩn hóa địa chỉ người nhận anonymous |
| **Vmap / Bản đồ số** | Tọa độ, tuyến đường | T2 phân tích địa chỉ rủi ro theo vùng |

#### Nhóm 8 — Báo cáo & Phân tích (đã có sẵn)

| Hệ thống | Dữ liệu có | Ghi chú |
|---|---|---|
| **CAS-ReportingDB** | Báo cáo dữ liệu từ CAS | Có thể dùng làm nguồn batch export cho CDP ban đầu |
| **BI** | Báo cáo phân tích nội bộ | CDP bổ sung chiều dữ liệu KH vào đây |

---

### 3.2 Đáp ứng được ngay (học pattern trực tiếp)

| VNPost cần | Hệ thống nguồn | Unomi có | Ghi chú |
|---|---|---|---|
| Gộp nhiều mã định danh về 1 hồ sơ | CAS, MyVNPost, PayPost, PostID, SmartLocker | **Có** — cơ chế alias đã xác minh | Cần định nghĩa quy tắc gộp: "nếu SĐT trùng thì gộp" |
| Tự động gắn nhãn KHL sắp rời đi | Portal KHL / CAS (sản lượng) qua MPITS | **Có** — Rule Engine đã xác minh | Cấu hình ngưỡng: giảm 30% trong 4 tuần |
| Phát hiện địa chỉ giao hàng thất bại lặp lại | PNS Phát / DingDong | **Có** — đếm số lần thất bại qua Rule Engine | Ngưỡng: 3 lần/30 ngày |
| Quản lý đồng ý dữ liệu | MyVNPost App, CAS quầy, PayPost | **Có** — module consent sẵn có | Cần thêm bộ lan truyền xóa sang hệ thống nguồn |
| Cấu hình ngưỡng không cần IT release | — | **Có** — quy tắc là file cấu hình | Marketing tự điều chỉnh, không đợi IT |

### 3.3 Cần tùy chỉnh thêm

| VNPost cần | Hệ thống liên quan | Unomi chưa có | Cần làm thêm gì |
|---|---|---|---|
| Hồ sơ người nhận (không có tài khoản) | PNS Phát — SĐT + địa chỉ người nhận | Không có pattern này | Thiết kế riêng: hồ sơ "nhẹ" chỉ có SĐT + địa chỉ |
| Kết nối với từng hệ thống nguồn | CAS, PayPost, MPITS, PNS, Care Đơn... | Không có connector sẵn | Tự viết cầu nối cho từng hệ thống |
| Gửi cảnh báo qua Zalo OA, SMS | Care Đơn, CRM | Không hỗ trợ trực tiếp | Unomi chỉ đẩy tín hiệu ra — Care Đơn/SMS gateway lo gửi |
| Phân quyền tỉnh A không xem được tỉnh B | Toàn bộ hệ thống | Chưa xác minh đủ | Cần kiểm tra thêm trước khi quyết định |

### 3.4 Unomi không giải quyết được

| VNPost cần | Hệ thống liên quan | Lý do Unomi không giải quyết |
|---|---|---|
| Thu dữ liệu từ bưu tá gặp mặt (offline) | PNS Phát / DingDong | Unomi chỉ biết dữ liệu kỹ thuật số — không có khái niệm "cuộc gặp vật lý" |
| KHL doanh nghiệp có nhiều chi nhánh | CCP / SalesForce / Portal KHL | Unomi không có cấu trúc B2B tự nhiên (công ty → nhiều tài khoản con) |
| Phát hiện fraud theo mạng lưới địa chỉ | PNS Phát + VPostCode | Unomi nhìn từng profile riêng lẻ — không phân tích "cụm địa chỉ cạnh nhau cùng từ chối hàng" |
| Gộp dữ liệu khiếu nại từ 2 hệ thống riêng | Care Đơn + Cas-HTKH | Unomi không tự biết 2 hệ thống này chứa cùng loại dữ liệu — cần thiết kế pipeline hợp nhất trước |

---

## Câu hỏi 4 — Cần tham khảo thêm gì ngoài Unomi?

### 4.1 Vấn đề pháp lý — CẬP NHẬT QUAN TRỌNG

> **Nghị định 13/2023/NĐ-CP đã bị thay thế.** Văn bản pháp lý hiện hành là:
> - **Luật Bảo vệ Dữ liệu Cá nhân số 91/2025/QH15** — thông qua 26/6/2025
> - **Nghị định 356/2025/NĐ-CP** — ban hành 31/12/2025
> - **Cả hai có hiệu lực từ ngày 1/1/2026**

Thay đổi quan trọng nhất so với NĐ 13 cũ:

| Nội dung | NĐ 13/2023 (cũ) | Luật BVDLCN 2025 (mới) |
|---|---|---|
| Mức phạt vi phạm thông thường | Thấp hơn | Tối đa 3 tỷ VND (~115.000 USD) |
| Mức phạt chuyển dữ liệu ra nước ngoài | Thấp hơn | **Tối đa 5% doanh thu năm trước** |
| Phạm vi áp dụng | Cả gián tiếp lẫn trực tiếp | Chỉ tổ chức **trực tiếp** xử lý dữ liệu |

**Ý nghĩa với VNPost:** Luật mới nghiêm hơn và mức phạt cao hơn nhiều. CDP VNPost phải tuân thủ Luật BVDLCN 2025 — không còn là NĐ 13 nữa. Cần tư vấn pháp lý chính thức trước khi chốt thiết kế Consent Management.

**Câu hỏi pháp lý chưa có câu trả lời (cần hỏi luật sư):** Luật mới có ngoại lệ gì cho ngành bưu chính khi xử lý thông tin người nhận không có tài khoản không?

---

### 4.2 Pattern gộp ID cho người nhận anonymous

Đây là bài toán **Unomi không có sẵn** và **nghiên cứu này cũng chưa xác minh được pattern chuẩn**.

Bài toán: Người nhận bưu kiện không có tài khoản VNPost. VNPost biết họ qua SĐT hoặc địa chỉ. Cùng một người nhận có thể nhận hàng hàng chục lần từ nhiều người gửi khác nhau — nhưng VNPost không có ID thống nhất cho họ.

Pattern đề xuất (dựa trên suy luận logic, chưa được xác minh):
- Tạo hồ sơ "nhẹ" cho người nhận: chỉ có SĐT + địa chỉ + lịch sử giao dịch
- Không yêu cầu consent đầy đủ — chỉ dùng để phân tích rủi ro địa chỉ (UC-04) và phát hiện fraud (UC-06)
- Không dùng dữ liệu người nhận cho marketing — đây là ranh giới pháp lý

**Cần nghiên cứu thêm:** Pattern này có precedent (tiền lệ) ở các công ty logistics lớn nào? FedEx, DHL, J&T Express Việt Nam đang làm thế nào?

---

### 4.3 Phát hiện gian lận COD — Pattern từ thị trường

Từ nghiên cứu, các hệ thống phát hiện gian lận COD tại logistics Đông Nam Á thường dùng 3 tín hiệu:

| Tín hiệu | Mô tả | Áp dụng cho VNPost |
|---|---|---|
| **Tỷ lệ từ chối nhận** của địa chỉ | Địa chỉ này đã có bao nhiêu đơn bị từ chối trong 30 ngày? | Unomi làm được nếu thiết kế đúng |
| **Tần suất đổi tên** người nhận | SĐT này liên kết với bao nhiêu tên khác nhau? | Cần xây thêm ngoài Unomi |
| **Mạng lưới địa chỉ** nghi ngờ | Một cụm địa chỉ cạnh nhau có tỷ lệ từ chối đồng thời tăng? | Unomi không làm được — cần công cụ phân tích mạng lưới riêng |

Tín hiệu thứ 3 (mạng lưới địa chỉ) là phức tạp nhất và có giá trị nhất — không phải Unomi, mà cần một công cụ phân tích đồ thị (graph analysis). Đây là phạm vi của giai đoạn sau, không phải MVP.

---

### 4.4 Phân quyền tỉnh/vùng — Chưa có pattern đã xác minh

Câu hỏi "tỉnh A không xem được dữ liệu tỉnh B" vẫn chưa có câu trả lời rõ ràng từ Unomi (bị bác bỏ trong kiểm tra độ tin cậy). Cần:

1. Kiểm tra trực tiếp source code Unomi về ScopeService
2. Hoặc thiết kế riêng: tầng phân quyền ở lớp ngoài Unomi, Unomi chỉ là kho lưu trữ

---

## Tóm tắt — Làm gì tiếp theo

### Đã xác nhận — có thể thiết kế ngay

- **Cơ chế gộp ID** theo pattern ProfileAlias → dùng cho T1 (Identity Layer VNPost)
- **Rule Engine** Event→Condition→Action → dùng cho T2 (Trigger Engine, 4 use case)
- **Cấu hình ngưỡng không cần IT** → dùng cho T3 (Business Logic Layer tách biệt)
- **Luật bảo vệ dữ liệu mới** là Luật 91/2025 + NĐ 356/2025 (không còn là NĐ 13/2023)

### Cần làm rõ thêm trước khi thiết kế

| Câu hỏi | Tại sao quan trọng | Cách làm rõ |
|---|---|---|
| Scope của Unomi có thực sự phân vùng dữ liệu tỉnh/vùng không? | Ảnh hưởng trực tiếp đến T5 và Q9 | Đọc source code ScopeService trên GitHub |
| Segment có tính lại tức thì hay cần chờ? | UC-06 fraud cần phát hiện trong vài phút — nếu batch thì không đủ | Test trực tiếp trên môi trường Unomi |
| Luật BVDLCN 2025 có ngoại lệ cho bưu chính không? | Người nhận không có tài khoản — VNPost có được lưu thông tin họ không? | Hỏi luật sư/chuyên gia pháp lý |
| Pattern hồ sơ người nhận anonymous — logistics lớn làm thế nào? | VNPost cần thiết kế riêng phần này | Nghiên cứu bổ sung FedEx/DHL/J&T |
| SmartLocker có ID định danh riêng không, hay chỉ dùng SĐT? | Nếu có ID riêng → thêm 1 alias nữa vào T1; nếu chỉ SĐT → gộp được ngay | Hỏi IT VNPost về SmartLocker system |
| PayPost và E-Money có phải 2 hệ thống độc lập không? | Nếu độc lập → dữ liệu tài chính bị phân mảnh thêm, cần 2 connector riêng | Hỏi IT VNPost về phạm vi từng hệ thống |
| Care Đơn và Cas-HTKH có dữ liệu khiếu nại trùng nhau không? | Nếu trùng → phải dedup trước khi đưa vào CDP; nếu tách biệt → cần hợp nhất | Hỏi đội CSKH về quy trình xử lý sự vụ |

### Vẫn cần câu trả lời từ PO/client

Năm câu CRITICAL trong [Domain Brief](domain-brief.md) vẫn còn nguyên — đặc biệt Q9 (phân quyền tỉnh/TCT) và Q4 (chính sách bảo vệ dữ liệu theo luật mới).

---

## Nguồn đã kiểm tra

| Nguồn | Loại | Kết quả |
|---|---|---|
| unomi.apache.org/manual/latest/ | Chính thức | Xác nhận cơ chế alias + Rule Engine |
| github.com/apache/unomi/blob/master/manual/src/main/asciidoc/datamodel.adoc | Chính thức | Xác nhận cấu trúc ProfileAlias |
| downloads.apache.org/unomi/2.7.0/unomi-manual-2_7_x.pdf | Chính thức | Nguồn gốc, nhiều claims bị bác |
| tilleke.com — luật BVDLCN 2025 | Công ty luật | Xác nhận thay thế NĐ 13, mức phạt mới |
| dfdl.com — luật BVDLCN 2025 | Công ty luật | Xác nhận độc lập — Luật 91/2025 + NĐ 356/2025 |

*Tổng cộng: 101 agent, 19 nguồn, 76 claims trích xuất, 25 claims kiểm tra, 9 xác nhận, 16 bị bác bỏ.*
