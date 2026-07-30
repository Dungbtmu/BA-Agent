---
type: brainstorm
feature: cdp
status: draft
updated: 2026-07-30
links:
  - .claude/input/CDP/CDP.md
  - .claude/input/CDP/cdp-system-design-brief.md
  - .claude/output/cdp-tct/wireframe/prototype-v3.html
  - .claude/output/cdp-tct/solution/identity-resolution-design.md
  - .claude/output/cdp-tct/solution/clarification.md
  - .claude/output/cdp-tct/research/customer-model/customer-model.md
---

# Brainstorm — Nền tảng cho URD/SRS hệ thống CDP VNPost

> Buổi làm việc chuẩn bị cho `/urd`. Nguồn chuẩn là `CDP.md`; giao diện lấy theo prototype v3 (bản chốt 24/07/2026). Khi tài liệu này khác `CDP.md`, tài liệu gốc là chuẩn.

---

## 1. Tổng quan

CDP là nền tảng dữ liệu khách hàng tập trung của Tổng công ty Bưu điện Việt Nam, làm ba việc: thu thập dữ liệu khách hàng từ toàn bộ hệ sinh thái, hợp nhất các định danh rời rạc thành một hồ sơ duy nhất, và đưa kết quả phân tích trở lại phục vụ kinh doanh, chăm sóc khách hàng và vận hành.

CDP **không thay thế** các hệ thống nghiệp vụ hiện có. Nó là lớp dữ liệu trung gian nằm giữa các hệ thống nguồn và các kênh sử dụng dữ liệu.

### Vấn đề đang giải

Dữ liệu khách hàng nằm rải trên hơn 15 hệ thống, mỗi hệ thống định danh khách hàng theo cách riêng. Cùng một khách hàng doanh nghiệp có thể tồn tại dưới bốn mã khác nhau — mã khách hàng lớn ở hệ thống chấp nhận gửi, mã người dùng trên ứng dụng, mã tài chính ở hệ thống thanh toán, và mã trên sàn thương mại điện tử. Không hệ thống nào biết đó là cùng một người.

Hậu quả đo được:

- Đội bán hàng và chăm sóc khách hàng phải tra 5–7 hệ thống mới dựng được bức tranh về một khách hàng
- Không nhận ra khách hàng lớn đang giảm sản lượng cho đến khi họ đã rời đi
- Không đo được hiệu quả của bất kỳ chiến dịch nào vì không ghép được hành vi trước và sau chiến dịch của cùng một người
- Tỷ lệ hoàn hàng 10–15% gây chi phí vận hành hai chiều mà không có cơ chế nhận diện rủi ro từ trước

### Quy mô

| Nội dung | Con số |
|---|---|
| Khách hàng đã ký hợp đồng (doanh nghiệp, khách hàng lớn) | 500.000 |
| Khách hàng mới đang nuôi dưỡng (cá nhân, chủ shop nhỏ) | 100.000 |
| Điểm phục vụ | 13.000 trên 34 tỉnh thành |
| Lượng bản ghi tiếp nhận | Khoảng 1,7 triệu bản ghi mỗi ngày qua 8 luồng dữ liệu |

Nút thắt về hiệu năng nằm ở lượng bản ghi tiếp nhận mỗi ngày, không phải số lượng hồ sơ khách hàng.

---

## 2. Phạm vi tài liệu URD/SRS

| Nội dung | Quyết định |
|---|---|
| Độ phủ | **Toàn bộ 7 phân hệ** theo `CDP.md`, khoảng 99 mã yêu cầu chức năng |
| Người đọc | Lập trình viên và kiểm thử viên triển khai hệ thống |
| Mức chi tiết | Đủ để lập trình và viết được ca kiểm thử, không cần hỏi lại người phân tích |
| Cách xử lý điểm chưa có câu trả lời từ khách hàng | Viết theo giả định có mã riêng, đánh dấu rõ chỗ nào phụ thuộc câu trả lời |

Bảy phân hệ: tiếp nhận dữ liệu · chuẩn hóa và xử lý dữ liệu · hợp nhất định danh · quản lý hồ sơ khách hàng 360 · phân khúc, phân tích và trí tuệ nhân tạo · kích hoạt dữ liệu · quản trị, bảo mật và quyền riêng tư.

> **Đề xuất cách làm khi sang `/urd`:** viết theo lô từng phân hệ, mỗi lô một vòng duyệt. Dựng một mạch cả 99 yêu cầu rồi mới đưa duyệt sẽ rất khó review.

---

## 3. Người dùng và quyền truy cập

### 3.1. Mười hai vai trò

Hợp nhất từ mục 8.3 (mô hình vai trò quản trị dữ liệu) và mục 8.7 (phân quyền truy cập) của `CDP.md`.

| # | Vai trò | Việc chính trên CDP | Đã có giao diện ở bản chốt |
|---|---|---|---|
| 1 | Chủ sở hữu dữ liệu | Phê duyệt mục đích sử dụng, phạm vi chia sẻ, quy tắc dữ liệu | Chưa |
| 2 | Người phụ trách dữ liệu | Đối soát định danh, xử lý dữ liệu lỗi, quản lý quy tắc | Có |
| 3 | Kỹ sư dữ liệu | Vận hành luồng tiếp nhận, xử lý lỗi đồng bộ | Chưa |
| 4 | Chuyên viên phân tích dữ liệu | Truy cập dữ liệu phân tích, dữ liệu đã che | Chưa |
| 5 | Quản trị hệ thống | Quản lý tài khoản, vai trò, cấu hình | Có |
| 6 | An toàn thông tin | Xem nhật ký, kiểm tra truy cập, điều tra sự cố | Chưa |
| 7 | Pháp chế và tuân thủ | Kiểm tra tuân thủ, quy trình đồng ý, quyền của khách hàng | Chưa |
| 8 | Tiếp thị và quản lý quan hệ khách hàng | Tạo phân khúc, chạy chiến dịch | Có |
| 9 | Chăm sóc khách hàng và tổng đài | Tra cứu hồ sơ, lịch sử giao dịch, khiếu nại | Có |
| 10 | Kinh doanh và khách hàng lớn | Xem khách hàng phụ trách, sản lượng, nguy cơ rời bỏ | Có |
| 11 | Vận hành và thu hộ | Xem vận đơn, trạng thái phát, thu hộ, hoàn hàng | Có |
| 12 | Lãnh đạo và quản lý đơn vị | Xem báo cáo tổng hợp theo phạm vi được phân quyền | Chưa |

Sáu vai trò đã có giao diện là tập con của bộ mười hai. Sáu vai trò còn lại cần được bổ sung màn hình khi làm phân hệ quản trị và phân hệ phân tích.

### 3.2. Nguyên tắc phân quyền

Áp dụng bảy nguyên tắc của mục 8.7: cấp quyền tối thiểu · chỉ người có nhu cầu nghiệp vụ hợp lệ · tách quyền cấu hình khỏi quyền xem dữ liệu · phân quyền theo đơn vị và địa bàn · phân quyền gắn với mục đích sử dụng · quyền đặc biệt có thời hạn · truy cập dữ liệu nhạy cảm cần phê duyệt.

Sáu cấp phạm vi theo mục 8.13: theo đơn vị và tỉnh thành · theo bưu cục và vùng phục vụ · theo khách hàng phụ trách · theo nhóm nghiệp vụ · theo mức độ chi tiết dữ liệu · theo mục đích sử dụng.

### 3.3. Ranh giới hệ thống

| Nội dung | Ranh giới |
|---|---|
| Đăng nhập | CDP **nhận danh tính** từ cổng đăng nhập chung của tổ chức (mã nhân sự đã cấp quyền hoặc đăng nhập một lần nội bộ). CDP **không tự quản lý** tài khoản và mật khẩu, không có màn đăng nhập riêng |
| Khách hàng cuối | **Không truy cập CDP.** Không có màn hình nào dành cho khách hàng VNPost |
| Trạng thái đồng ý dữ liệu | CDP **chỉ nhận** từ nguồn: ứng dụng MyVNPost, website, quầy giao dịch, hệ thống quan hệ khách hàng. CDP không tự thu đồng ý |
| Yêu cầu xem hoặc xóa dữ liệu của khách hàng | Đến qua **chăm sóc khách hàng tiếp nhận** rồi nhập vào CDP. Không có kênh tự phục vụ |
| Thiết bị truy cập | Trang web độc lập mở trên trình duyệt. Truy cập được bằng điện thoại qua đường dẫn |

---

## 4. Năng lực theo mức ưu tiên

Xếp theo ma trận bốn giai đoạn của mục 7.9, `CDP.md`.

**P0 — giai đoạn thử nghiệm**

Tiếp nhận dữ liệu thời gian thực và theo lô · chuẩn hóa số điện thoại, họ tên, địa chỉ · hợp nhất định danh với sơ đồ liên kết định danh · hồ sơ khách hàng 360 cơ bản · quản lý đồng ý · phân quyền theo vai trò · nhật ký thao tác · bảng theo dõi chất lượng dữ liệu.

**P1 — giai đoạn mở rộng nghiệp vụ**

Phân khúc động · đồng bộ sang hệ thống quan hệ khách hàng · kích hoạt kênh tiếp thị · lịch sử thu hộ và thanh toán · lịch sử khiếu nại · phân tích rủi ro thu hộ và hoàn hàng · danh sách loại trừ · xử lý yêu cầu của khách hàng.

**P2 — giai đoạn nâng cao**

Phân tích theo mô hình gần đây, tần suất, giá trị · giá trị vòng đời khách hàng · dự báo nguy cơ rời bỏ · chấm điểm khách hàng · phát hiện gian lận · gợi ý dịch vụ · danh mục dữ liệu và truy vết dòng dữ liệu · kiểm soát theo mục đích sử dụng · phân quyền theo đơn vị và địa bàn · báo cáo tuân thủ.

---

## 5. Tám luồng nghiệp vụ chính

### Luồng 1 — Tiếp nhận và chuẩn hóa dữ liệu

1. Hệ thống nguồn phát sinh dữ liệu: nhóm thời gian thực đẩy sự kiện ngay khi phát sinh; nhóm hệ thống cũ xuất theo lô trong khung 01:00–05:00.
2. CDP tiếp nhận và kiểm tra cấu trúc bản ghi: trường bắt buộc, kiểu dữ liệu, phiên bản cấu trúc.
3. Bản ghi sai cấu trúc đi thẳng vào hàng đợi lỗi, **không thử lại**. Bản ghi lỗi do mất kết nối hoặc nguồn quá tải được thử lại ba lần theo nhịp 1 phút, 5 phút, 15 phút.
4. CDP chuẩn hóa: số điện thoại về một dạng thống nhất, email về chữ thường, họ tên bỏ khoảng trắng thừa và xử lý dấu, địa chỉ bóc tách theo cấp hành chính và ánh xạ mã địa chỉ số, mã số thuế kiểm tra độ dài, mã vận đơn chuẩn hóa chữ hoa, trạng thái bưu gửi và trạng thái thu hộ ánh xạ về bộ trạng thái chuẩn.
5. Bản ghi đạt chuẩn chuyển sang bước so khớp định danh.
6. Kỹ sư dữ liệu và người phụ trách dữ liệu theo dõi tình trạng luồng trên bảng giám sát; có cảnh báo khi luồng chậm hoặc ngừng.

### Luồng 2 — Hợp nhất định danh

1. Hệ thống tính điểm tin cậy cho từng cặp bản ghi nghi trùng, dựa trên bộ luật đối sánh tuyệt đối và các tín hiệu hỗ trợ.
2. Điểm từ 95% trở lên: **tự động gộp**, không cần ai duyệt.
3. Điểm từ 85% đến dưới 95%: đưa vào **hàng đợi đối soát**.
4. Điểm từ 70% đến dưới 85%: **lưu quan hệ nghi vấn** trong sơ đồ liên kết định danh, không gộp, không đưa vào hàng đợi.
5. Điểm dưới 70%: **không gộp**.
6. Các trường hợp cấm gộp tự động dù điểm cao: chỉ trùng mã vận đơn, chỉ trùng địa chỉ, chỉ trùng địa chỉ mạng, chỉ trùng thiết bị, số điện thoại là số dùng chung hoặc tổng đài, người gửi và người nhận chỉ trùng một thông tin phụ, hoặc thiếu đồng ý cho mục đích kích hoạt.
7. Người phụ trách dữ liệu mở hồ sơ trong hàng đợi, xem bảng so sánh từng cột giữa các mã nguồn, tick chọn mã thuộc cùng khách hàng, xem trước hồ sơ chuẩn dự kiến, rồi xác nhận hợp nhất hoặc đánh dấu là các khách hàng khác nhau.
8. Hệ thống sinh mã định danh CDP, giữ lại toàn bộ mã nguồn cũ dưới dạng mã thay thế, tính lại điểm số, và ghi nhật ký gộp.

**Hình luồng — từ tiếp nhận đến hồ sơ chuẩn**

```
NGUỒN DỮ LIỆU
│  Thời gian thực : MyVNPost · CAS · MPITS · PNS/DingDong
│  Theo lô 01–05h : BCCP · TMS · WMS · PayPost
└──────────────┬──────────────────────────────────────────────────
               ▼
      ┌─────────────────────────┐
 [1]  │ TIẾP NHẬN               │
      │ Kiểm tra cấu trúc       │
      └────────┬────────────────┘
               │
       sai cấu trúc ──────────▶ HÀNG ĐỢI LỖI (không thử lại, giữ 30 ngày)
       mất kết nối ───────────▶ THỬ LẠI 3 lần: 1' → 5' → 15'
               │                        └─ vẫn lỗi ─▶ HÀNG ĐỢI LỖI
               │                                          │
               ▼                                          ▼
      ┌─────────────────────────┐              Người phụ trách dữ liệu
 [2]  │ CHUẨN HÓA               │              xem, sửa hoặc trả về nguồn
      │ SĐT · email · tên       │
      │ địa chỉ · MST · vận đơn │
      │ trạng thái phát/COD     │
      └────────┬────────────────┘
               ▼
      ┌─────────────────────────┐
 [3]  │ SO KHỚP ĐỊNH DANH       │
      │ Tính điểm tin cậy       │
      └────────┬────────────────┘
               │
     ┌─────────┼─────────┬──────────────┬──────────────┐
     ▼         ▼         ▼              ▼              ▼
  ≥ 95%    85–94%    70–84%         < 70%      cấm gộp tự động
     │         │         │              │       (chỉ trùng vận đơn/
     │         │         │              │        địa chỉ/IP/thiết bị,
     │         │         │              │        SĐT dùng chung,
     │         │         │              │        khác vai trò gửi/nhận)
     │         │         │              │              │
     ▼         ▼         ▼              ▼              ▼
  TỰ GỘP   HÀNG ĐỢI  LƯU QUAN HỆ    KHÔNG GỘP    ĐƯA VÀO HÀNG ĐỢI
     │      ĐỐI SOÁT  NGHI VẤN                       ĐỐI SOÁT
     │         │      (không hiện
     │         │       hàng đợi)
     │         ▼
     │   ┌──────────────────────────┐
     │   │ NGƯỜI ĐỐI SOÁT           │◀── nhắc sau 2 ngày
     │   │ So sánh từng cột         │    báo quản lý sau 5 ngày
     │   │ Tick chọn mã cần gộp     │
     │   │ Xem trước hồ sơ chuẩn    │
     │   └────────┬─────────────────┘
     │            │
     │      ┌─────┴─────┐
     │      ▼           ▼
     │  xác nhận   khác người
     │    gộp          │
     │      │          └──▶ gỡ cờ nghi trùng
     │      │
     │  người khác đã xử lý trước ──▶ hiện thông báo, làm mới danh sách
     │      │
     ▼      ▼
  ┌──────────────────────────┐
  │ HỒ SƠ CHUẨN              │──▶ GHI NHẬT KÝ GỘP (bất biến, lưu 5 năm)
  │ Sinh mã định danh CDP    │
  │ Giữ toàn bộ mã nguồn cũ  │──▶ Customer 360 · phân khúc · điểm số
  │ Tính lại điểm số         │
  └──────────────────────────┘
```

### Luồng 3 — Tra cứu hồ sơ khách hàng 360

1. Người dùng tìm khách hàng theo số điện thoại, email, mã khách hàng, mã khách hàng lớn, mã định danh VNPost, mã vận đơn hoặc mã số thuế.
2. Hệ thống trả danh sách kết quả, dữ liệu nhạy cảm đã che theo vai trò người tìm.
3. Người dùng mở một khách hàng để xem hồ sơ đầy đủ, gồm các nhóm: định danh, các mã liên kết, địa chỉ, thông tin doanh nghiệp nếu có, hoạt động theo mảng dịch vụ, hành vi số, chăm sóc khách hàng, điểm số và phân khúc, trạng thái đồng ý, nhật ký nguồn dữ liệu.
4. Mỗi nhóm dữ liệu hiển thị theo đúng quyền của vai trò; nhóm không được xem thì ẩn hoặc che, không hiện dữ liệu rỗng gây hiểu nhầm.
5. Người có quyền xem được nguồn phát sinh của từng nhóm dữ liệu và so sánh giá trị giữa các hệ thống nguồn.
6. Người có quyền ghi chú hoặc gắn nhãn khách hàng cần chăm sóc đặc biệt.

### Luồng 4 — Tạo và quản lý phân khúc

1. Người dùng tiếp thị mở danh sách phân khúc, xem các phân khúc đang có kèm số khách hàng khớp.
2. Tạo phân khúc mới: đặt tên, mô tả, rồi thêm điều kiện theo trường dữ liệu.
3. Điều kiện hỗ trợ lồng nhiều tầng với phép và, hoặc, phủ định — trên thuộc tính hồ sơ, hành vi, giao dịch, thu hộ, địa bàn, tỷ lệ hoàn, khiếu nại.
4. Hệ thống ước lượng số khách hàng khớp ngay khi người dùng sửa điều kiện.
5. Lưu phân khúc. Phân khúc động tự cập nhật khi dữ liệu khách hàng thay đổi.
6. Khi người dùng sửa điều kiện của phân khúc **đang được chiến dịch sử dụng**, hệ thống cảnh báo và liệt kê các chiến dịch bị ảnh hưởng. Người dùng xác nhận thì phân khúc cập nhật theo điều kiện mới.

### Luồng 5 — Chấm điểm và cảnh báo rủi ro

1. Hệ thống tính định kỳ các điểm số cho từng khách hàng: mức độ tương tác, giá trị vòng đời, nguy cơ rời bỏ, rủi ro thu hộ, rủi ro gian lận, chất lượng dịch vụ.
2. Kết quả được ghi vào hồ sơ khách hàng và hiển thị theo quyền của từng vai trò.
3. Khi điểm số vượt ngưỡng cảnh báo, hệ thống đưa khách hàng vào phân khúc tương ứng và phát cảnh báo.
4. Cảnh báo được gửi tới bộ phận liên quan qua thông báo trong ứng dụng và email.
5. Người nhận cảnh báo mở hồ sơ để xem căn cứ, và ghi nhận hành động xử lý.

### Luồng 6 — Kích hoạt dữ liệu

1. Người dùng chọn phân khúc cần kích hoạt và kênh gửi.
2. Hệ thống **kiểm tra đồng ý** cho từng khách hàng theo đúng mục đích và đúng kênh.
3. Khách hàng chưa đồng ý, đã từ chối, hoặc nằm trong danh sách loại trừ bị loại khỏi tệp gửi; hệ thống báo số lượng bị loại.
4. Hệ thống kiểm tra hạn mức tần suất gửi và khung giờ được phép gửi.
5. Tệp vượt ngưỡng cần phê duyệt thì chuyển sang bước phê duyệt trước khi gửi.
6. Tệp được đẩy sang kênh; hệ thống theo dõi trạng thái đồng bộ và ghi lịch sử kích hoạt.
7. Kết quả phản hồi từ kênh (gửi thành công, mở, phản hồi) được nhận về và cập nhật lại hồ sơ khách hàng.

**Hình luồng — kích hoạt có kiểm tra đồng ý**

```
Người dùng chọn phân khúc + kênh gửi
            │
            ▼
   ┌──────────────────────┐
   │ KIỂM TRA ĐỒNG Ý      │  theo mục đích + theo kênh
   └─────────┬────────────┘
             │
     ┌───────┴────────┐
     ▼                ▼
  đủ điều kiện    chưa đồng ý / đã từ chối / trong danh sách loại trừ
     │                │
     │                └──▶ LOẠI KHỎI TỆP ─▶ báo số lượng bị loại
     ▼
   ┌──────────────────────┐
   │ KIỂM TRA TẦN SUẤT    │  ≤3 tin/tuần · ≤1 tin/kênh/ngày
   │ VÀ KHUNG GIỜ         │  không gửi 21h–08h
   └─────────┬────────────┘
             │
     ┌───────┴────────┐
     ▼                ▼
   trong hạn      vượt hạn ─▶ giữ lại, gửi ở chu kỳ sau
     │
     ▼
   ┌──────────────────────┐
   │ KIỂM TRA NGƯỠNG      │  >1.000 bản ghi ─▶ chờ phê duyệt
   │ PHÊ DUYỆT            │  >100.000 ─▶ chặn, yêu cầu thu hẹp
   └─────────┬────────────┘
             ▼
   ĐẨY SANG KÊNH ──▶ theo dõi đồng bộ ──▶ ghi lịch sử kích hoạt
             │                                    │
        đồng bộ lỗi ──▶ cảnh báo + thử lại        ▼
                                          nhận phản hồi từ kênh
                                          cập nhật hồ sơ khách hàng
```

### Luồng 7 — Xử lý yêu cầu của khách hàng

1. Khách hàng gửi yêu cầu qua ứng dụng, website, bưu cục, tổng đài hoặc chăm sóc khách hàng.
2. Bộ phận tiếp nhận **xác thực danh tính** người yêu cầu để tránh trả dữ liệu cho sai người.
3. Phân loại yêu cầu: xem dữ liệu, chỉnh sửa, rút đồng ý, ngừng xử lý, xóa hoặc ẩn danh, yêu cầu giải thích.
4. Người phụ trách dữ liệu kiểm tra phạm vi dữ liệu trong CDP và các hệ thống nguồn liên quan.
5. Thực hiện xử lý trong CDP, hoặc chuyển yêu cầu sang hệ thống nguồn nếu dữ liệu gốc nằm ở đó.
6. Cập nhật trạng thái, ghi nhật ký, thông báo kết quả cho khách hàng.
7. Đồng bộ thay đổi sang các hệ thống nhận dữ liệu nếu ảnh hưởng tới đồng ý hoặc kích hoạt.

### Luồng 8 — Quản trị và tuân thủ

1. Quản trị hệ thống tạo tài khoản, gán vai trò và phạm vi dữ liệu theo đơn vị, địa bàn, nhóm khách hàng phụ trách.
2. Quyền đặc biệt được cấp có thời hạn và tự hết hạn.
3. Mọi thao tác quan trọng được ghi nhật ký không thể xóa: đăng nhập, tìm kiếm, xem hồ sơ, xem dữ liệu nhạy cảm, xuất dữ liệu, tạo và sửa phân khúc, kích hoạt chiến dịch, gộp và tách hồ sơ, thay đổi đồng ý, thay đổi phân quyền.
4. Xuất dữ liệu vượt ngưỡng phải qua phê duyệt; tệp xuất luôn che dữ liệu nhạy cảm trừ khi có quyền đặc biệt kèm lý do.
5. An toàn thông tin theo dõi truy cập bất thường: truy cập ngoài giờ, tải dữ liệu lớn, tra cứu nhiều lần dữ liệu định danh.
6. Bộ phận tuân thủ xem các báo cáo định kỳ về đồng ý, truy cập, xuất dữ liệu, xử lý yêu cầu khách hàng, chất lượng dữ liệu.

---

## 6. Đào sâu hành vi hệ thống

### 6.1. Điểm rẽ nhánh

| Mã | Luồng | Khi nào | Nhánh có | Nhánh không |
|---|---|---|---|---|
| DP-01 | 1 | Bản ghi đúng cấu trúc? | Chuyển sang chuẩn hóa | Vào hàng đợi lỗi, không thử lại |
| DP-02 | 1 | Lỗi có phải do mất kết nối hoặc nguồn quá tải? | Thử lại 3 lần theo nhịp 1–5–15 phút | Vào hàng đợi lỗi ngay |
| DP-03 | 1 | Địa chỉ chuẩn hóa được không? | Gắn mã địa chỉ số và vùng phục vụ | Đánh dấu chưa chuẩn hóa, đưa vào danh sách xử lý chất lượng dữ liệu |
| DP-04 | 2 | Điểm tin cậy thuộc vùng nào? | Xem bốn vùng ở Luồng 2 | — |
| DP-05 | 2 | Có thuộc trường hợp cấm gộp tự động? | Đưa vào hàng đợi đối soát dù điểm cao | Xử lý theo vùng điểm |
| DP-06 | 2 | Người đối soát kết luận cùng khách hàng? | Hợp nhất, sinh mã định danh CDP | Gỡ cờ nghi trùng, không đề xuất lại |
| DP-07 | 3 | Vai trò có quyền xem nhóm dữ liệu này? | Hiện đầy đủ | Ẩn hoặc che theo quy tắc |
| DP-08 | 4 | Phân khúc đang được chiến dịch nào dùng? | Cảnh báo, liệt kê chiến dịch, chờ xác nhận | Cập nhật ngay |
| DP-09 | 6 | Khách hàng đồng ý nhận kênh này cho mục đích này? | Giữ trong tệp gửi | Loại khỏi tệp, đếm vào số bị loại |
| DP-10 | 6 | Trong hạn mức tần suất và khung giờ? | Gửi | Giữ lại, chuyển sang chu kỳ sau |
| DP-11 | 6 | Số bản ghi vượt ngưỡng phê duyệt? | Chuyển sang bước phê duyệt | Xuất hoặc gửi trực tiếp |
| DP-12 | 7 | Xác thực được danh tính người yêu cầu? | Tiếp tục xử lý | Từ chối, ghi lý do |

### 6.2. Tình huống theo vai trò

Quy tắc chung của mục 8.8: cùng một màn hình, mỗi vai trò thấy mức chi tiết khác nhau — không chỉ ẩn hiện cả khối mà còn che nội dung bên trong từng trường.

| Nhóm dữ liệu | Chăm sóc KH | Tiếp thị | Kinh doanh | Vận hành/Thu hộ | Phụ trách dữ liệu | Quản trị |
|---|---|---|---|---|---|---|
| Họ tên, mã định danh | Đầy đủ | Đầy đủ | Đầy đủ | Đầy đủ | Đầy đủ | Đầy đủ |
| Số điện thoại, email | Che một phần | Che một phần | Đầy đủ | Che một phần | Đầy đủ | Đầy đủ |
| Số định danh cá nhân | Che | Không xem | Không xem | Không xem | Che | Đầy đủ theo quyền đặc biệt |
| Địa chỉ chi tiết | Đến phường/quận/tỉnh | Đến phường/quận/tỉnh | Đầy đủ | Đầy đủ | Đầy đủ | Đầy đủ |
| Lịch sử giao dịch | Đầy đủ | Tổng hợp | Đầy đủ | Đầy đủ | Đầy đủ | Đầy đủ |
| Thu hộ và tài khoản nhận tiền | Tổng hợp, tài khoản che | Không xem | Tổng hợp | Đầy đủ | Che | Đầy đủ theo quyền đặc biệt |
| Hành vi số | Đầy đủ | Đầy đủ | Tổng hợp | Không xem | Đầy đủ | Đầy đủ |
| Điểm gần đây/tần suất/giá trị, giá trị vòng đời, nguy cơ rời bỏ | Xem | Xem | Xem | Không xem | Xem | Xem |
| Điểm rủi ro thu hộ, rủi ro gian lận | **Không xem** | **Không xem** | Xem | Xem | Xem | Xem |
| Trạng thái đồng ý | Xem | Xem | Xem | Không xem | Xem | Xem |
| Nhật ký gộp hồ sơ | Tóm tắt của khách hàng đang mở | Không xem | Không xem | Không xem | Đầy đủ | Đầy đủ |

Sáu vai trò còn lại (chủ sở hữu dữ liệu, kỹ sư dữ liệu, phân tích dữ liệu, an toàn thông tin, pháp chế, lãnh đạo đơn vị) cần bổ sung dòng khi làm phân hệ quản trị — hiện chưa có giao diện nên chưa chốt được mức chi tiết.

### 6.3. Chuyển trạng thái

| Thực thể | Từ | Sang | Kích hoạt bởi | Quay lại được? |
|---|---|---|---|---|
| Hồ sơ khách hàng | Đang hoạt động | Ngừng hoạt động | Không phát sinh giao dịch quá thời hạn quy định | Có, khi phát sinh giao dịch mới |
| Hồ sơ khách hàng | Đang hoạt động | Bị khóa | Quản trị khóa theo yêu cầu nghiệp vụ hoặc rủi ro | Có, cần quyền quản trị |
| Hồ sơ khách hàng | Đang hoạt động | Đã hợp nhất | Hồ sơ được gộp vào hồ sơ chuẩn khác | Chỉ khi mở lại luồng tách hồ sơ |
| Quan hệ định danh | Chờ duyệt | Đã gộp | Người đối soát xác nhận, hoặc hệ thống tự gộp từ 95% | Chỉ khi mở lại luồng tách hồ sơ |
| Quan hệ định danh | Chờ duyệt | Bị từ chối | Người đối soát kết luận khác khách hàng | Không |
| Quan hệ định danh | Hoạt động | Hết hạn | Quan hệ không được xác nhận lại trong thời gian quy định | Có, khi phát sinh tín hiệu mới |
| Đồng ý dữ liệu | Chưa rõ | Đồng ý / Từ chối | Nguồn ghi nhận lựa chọn của khách hàng | Có |
| Đồng ý dữ liệu | Đồng ý | Đã rút | Khách hàng rút đồng ý | Có, nếu khách hàng đồng ý lại |
| Đồng ý dữ liệu | Đồng ý | Hết hạn | Quá thời hạn hiệu lực của đồng ý | Có, khi khách hàng gia hạn |
| Yêu cầu của khách hàng | Tiếp nhận | Đã xác thực | Bộ phận tiếp nhận xác thực danh tính | Không |
| Yêu cầu của khách hàng | Đã xác thực | Đang xử lý | Phân loại xong, chuyển người phụ trách | Không |
| Yêu cầu của khách hàng | Đang xử lý | Hoàn tất / Từ chối | Xử lý xong hoặc không đủ điều kiện | Không |
| Bản ghi lỗi đồng bộ | Chờ thử lại | Trong hàng đợi lỗi | Thử lại ba lần vẫn thất bại | Có, khi được sửa và nạp lại |
| Bản ghi lỗi đồng bộ | Trong hàng đợi lỗi | Đã xử lý / Bỏ qua | Người phụ trách dữ liệu quyết định | Không |
| Phân khúc | Đang hoạt động | Tạm dừng | Người dùng tạm dừng, hoặc điều kiện không còn hợp lệ | Có |

### 6.4. Khi luồng bị gián đoạn

| Tình huống | Hành vi hệ thống |
|---|---|
| Hai người cùng xử lý một hồ sơ nghi trùng | Ai bấm trước người đó thắng. Không khóa hồ sơ. Người sau thao tác nhận thông báo hiện ngay trên màn hình, danh sách được làm mới |
| Khách hàng rút đồng ý khi tệp đã đẩy sang kênh | CDP chặn ngay khi tạo tệp tiếp theo, đồng thời đẩy trạng thái rút đồng ý sang kênh trong vòng 24 giờ để kênh tự loại khỏi hàng chờ chưa gửi. Tin đã gửi đi thì ghi nhận vào lịch sử kích hoạt, không thu hồi |
| Điều kiện phân khúc bị sửa khi chiến dịch đang chạy | Hệ thống cảnh báo và liệt kê chiến dịch bị ảnh hưởng. Người dùng xác nhận thì phân khúc tự cập nhật theo điều kiện mới |
| Nguồn dữ liệu ngừng đẩy giữa chừng | Cảnh báo vàng khi tồn đọng cần hơn 15 phút xử lý; báo động đỏ khi nguồn ngừng quá 15 phút trong khung giờ hoạt động |
| Đồng bộ sang kênh nhận dữ liệu thất bại | Thử lại theo cơ chế của bước tiếp nhận; cảnh báo cho quản trị; ghi vào lịch sử đồng bộ |
| Người dùng mất kết nối khi đang đối soát | Thao tác chưa xác nhận không được lưu. Hồ sơ vẫn ở trạng thái chờ duyệt, xuất hiện lại trong danh sách |
| Hồ sơ chờ duyệt tồn đọng | Nhắc người phụ trách sau 2 ngày làm việc; báo quản lý sau 5 ngày; cảnh báo khi quá 200 hồ sơ chờ |
| Bản ghi trong hàng đợi lỗi không ai xử lý | Giữ 30 ngày, sau đó chuyển sang lưu trữ, không xóa |

### 6.5. Các tình huống đặc thù khác

Hai mươi tình huống của mục 6.9 `CDP.md` đều phải được xử lý, trong đó những cái ảnh hưởng nặng nhất tới thiết kế:

- Khách hàng đổi số điện thoại: giữ số cũ làm mã thay thế, không mất lịch sử giao dịch
- Một số điện thoại dùng cho nhiều người: đánh dấu là số dùng chung, không dùng làm khóa gộp tự động
- Người nhận không có tài khoản: tạo hồ sơ người nhận riêng, không kích hoạt nếu thiếu đồng ý
- Một người vừa là người gửi vừa là người nhận: một mã khách hàng có nhiều vai trò, từng giao dịch phải ghi rõ vai trò
- Doanh nghiệp có nhiều người liên hệ: quản lý theo mô hình tổ chức và người liên hệ, không gộp tất cả thành một cá nhân
- Chủ shop hoạt động trên nhiều kênh: hợp nhất bằng mã số thuế, mã khách hàng lớn, số điện thoại, email hoặc tài khoản nhận tiền
- Dữ liệu từ hệ thống cũ thiếu trường: liên kết qua mã vận đơn để lấy người gửi và người nhận
- Trạng thái thu hộ chênh lệch giữa các hệ thống: áp quy tắc nguồn ưu tiên, đưa chênh lệch vào hàng đợi đối soát
- Hồ sơ không đủ dữ liệu để gộp: tạo hồ sơ tạm độ tin cậy thấp, không kích hoạt, chờ bổ sung

---

## 7. Ràng buộc, giới hạn và câu chữ

### 7.1. Quy tắc nghiệp vụ

- Điểm tin cậy và bộ luật đối sánh áp dụng theo mục 6.6.1 và 6.6.2 của `CDP.md`, không dùng ngưỡng tự đặt
- Tên khách hàng **không được** dùng làm khóa gộp độc lập trong mọi trường hợp
- Số điện thoại đã đánh dấu dùng chung **không** được dùng làm khóa gộp tự động
- Không kích hoạt tiếp thị với khách hàng thiếu đồng ý hợp lệ; hồ sơ thiếu đồng ý **không bị xóa** khỏi CDP nhưng bị giới hạn mục đích sử dụng
- Đồng ý phải xét theo **từng mục đích** và **từng kênh**; đồng ý cho vận hành không tự động dùng được cho tiếp thị
- Nhật ký gộp hồ sơ và nhật ký đồng ý là **bất biến** — chỉ được ghi thêm, không sửa, không xóa
- Mã nguồn cũ **không bao giờ bị xóa** sau khi gộp, để truy vết và đồng bộ ngược
- Nguồn ưu tiên khi xung đột giá trị áp theo bảng 12 nhóm dữ liệu của mục 6.10

### 7.2. Giới hạn và con số

**Độ trễ dữ liệu**

| Nhóm | Giới hạn |
|---|---|
| Hành vi số, tạo đơn, tra cứu | ≤ 5 phút từ lúc phát sinh tới khi hiện trên hồ sơ |
| Trạng thái phát, thu hộ | ≤ 15 phút |
| Hệ thống cũ (khai thác, vận tải, kho) | 1 lần/ngày, chạy 01:00–05:00 |
| Đối soát thu hộ | 1 lần/ngày, sau khi hệ thống thanh toán chốt sổ |
| Mùa cao điểm | Cho phép trễ gấp 3, không quá 30 phút với nhóm thời gian thực |

**Xử lý lỗi đồng bộ**

| Nội dung | Giới hạn |
|---|---|
| Lỗi tạm thời | Thử lại 3 lần, giãn cách 1 phút, 5 phút, 15 phút |
| Lỗi cấu trúc dữ liệu | Không thử lại, vào hàng đợi lỗi ngay |
| Thời gian giữ bản ghi trong hàng đợi lỗi | 30 ngày, sau đó chuyển lưu trữ |

**Ngưỡng cảnh báo luồng dữ liệu**

| Mức | Điều kiện |
|---|---|
| Cảnh báo | Tồn đọng cần hơn 15 phút xử lý, hoặc tỷ lệ bản ghi lỗi vượt 1% trong 1 giờ |
| Báo động | Nguồn ngừng đẩy quá 15 phút trong khung giờ hoạt động, hoặc tỷ lệ lỗi vượt 5% trong 1 giờ, hoặc tồn đọng cần hơn 60 phút xử lý |

Dùng tỷ lệ và thời gian thay vì số bản ghi tuyệt đối, vì các luồng chênh nhau rất xa về lưu lượng.

**Hàng đợi đối soát định danh**

| Nội dung | Giới hạn |
|---|---|
| Thời hạn xử lý một hồ sơ chờ duyệt | 2 ngày làm việc |
| Nhắc nhở | Sau 2 ngày; báo quản lý sau 5 ngày |
| Hiển thị danh sách | 25 dòng/trang, sắp theo điểm tin cậy giảm dần |
| Cảnh báo tồn đọng | Quá 200 hồ sơ chờ, hoặc có hồ sơ chờ quá 5 ngày |

**Xuất dữ liệu**

| Số bản ghi | Yêu cầu |
|---|---|
| ≤ 1.000 | Xuất trực tiếp, ghi nhật ký |
| 1.001 – 10.000 | Phê duyệt của quản lý trực tiếp |
| Trên 10.000 | Phê duyệt của quản trị dữ liệu và bộ phận tuân thủ |
| Trần cứng một lần xuất | 100.000 bản ghi, không cho vượt kể cả khi có phê duyệt |

Tệp xuất luôn che dữ liệu nhạy cảm, trừ khi người xuất có quyền đặc biệt và ghi rõ lý do vào nhật ký.

**Tần suất gửi tới khách hàng**

| Quy tắc | Giá trị |
|---|---|
| Tối đa mọi kênh tiếp thị gộp lại | 3 tin/khách hàng/tuần |
| Tối đa trên cùng một kênh | 1 tin/khách hàng/ngày |
| Khoảng lặng | Không gửi tin tiếp thị từ 21:00 đến 08:00 |
| Tin vận hành | Không tính vào hạn mức trên |

Tin vận hành gồm thông báo bưu gửi đang tới, nhắc thu hộ, kết quả phát — đây là dịch vụ khách hàng đã mua, không phải tiếp thị.

**Thời hạn lưu nhật ký**

| Loại | Thời hạn |
|---|---|
| Gộp và tách hồ sơ, thay đổi trạng thái đồng ý | 5 năm |
| Kích hoạt chiến dịch | 3 năm |
| Thao tác thường: xem, tìm kiếm, xuất, đổi phân quyền | 2 năm |

**Thời hạn xử lý yêu cầu của khách hàng**

Đặt hai mốc: hạn nội bộ chặt hơn để có biên an toàn, trần theo luật là giới hạn tuyệt đối.

| Loại yêu cầu | Hạn nội bộ | Trần theo luật |
|---|---|---|
| Rút lại đồng ý | Trong ngày làm việc, mục tiêu 4 giờ làm việc | 2 ngày làm việc |
| Xem hoặc trích xuất dữ liệu | 7 ngày | 10–15 ngày |
| Chỉnh sửa dữ liệu | 7 ngày | 10–15 ngày |
| Ngừng xử lý dữ liệu | 10 ngày | 15–20 ngày |
| Xóa hoặc ẩn danh dữ liệu | 15 ngày | 20–30 ngày |

Cảnh báo khi còn một phần ba thời hạn nội bộ; báo lên quản lý ngay khi quá hạn nội bộ, tức vẫn còn biên trước khi chạm hạn luật.

**Mục tiêu chất lượng dữ liệu**

| Chỉ tiêu | Sau 6 tháng | Sau 12 tháng |
|---|---|---|
| Hồ sơ có số điện thoại hợp lệ | ≥ 90% | ≥ 95% |
| Địa chỉ chuẩn hóa được | ≥ 75% | ≥ 85% |
| Hồ sơ trùng còn sót sau hợp nhất | ≤ 5% | ≤ 2% |
| Hồ sơ khách hàng lớn có đủ mã số thuế và mã khách hàng lớn | ≥ 95% | ≥ 98% |
| Hồ sơ có trạng thái đồng ý rõ ràng | ≥ 60% | ≥ 80% |

Địa chỉ đặt thấp hơn vì địa chỉ Việt Nam vốn viết tắt và không chuẩn. Nhóm khách hàng lớn đặt cao nhất vì có hợp đồng nên dữ liệu bắt buộc phải đủ.

### 7.3. Câu chữ thông báo

**Không đủ quyền**

| Tình huống | Câu chữ |
|---|---|
| Không được xem một nhóm dữ liệu | "Bạn không có quyền xem thông tin này. Liên hệ quản trị hệ thống nếu công việc của bạn cần dùng đến." |
| Không được vào chức năng | "Bạn không có quyền truy cập chức năng này." |

**Chưa có dữ liệu**

| Tình huống | Câu chữ |
|---|---|
| Trong một ô | "Chưa có dữ liệu" |
| Cả màn hình | "Chưa có dữ liệu để hiển thị." |
| Sau khi lọc | "Không tìm thấy khách hàng nào khớp điều kiện lọc." |

**Bị người khác thao tác trước**

| Tình huống | Câu chữ |
|---|---|
| Đang ở danh sách | "Hồ sơ này vừa được {tên người} xử lý lúc {giờ}. Danh sách đã được cập nhật." |
| Đang mở màn đối chiếu | "Hồ sơ này vừa được {tên người} hợp nhất. Bạn không thể thao tác tiếp trên bản cũ." |

**Luồng dữ liệu có vấn đề**

| Tình huống | Câu chữ |
|---|---|
| Cảnh báo | "Luồng {tên nguồn} đang chậm — dữ liệu có thể trễ tới {N} phút." |
| Báo động | "Luồng {tên nguồn} đã ngừng nhận dữ liệu {N} phút. Dữ liệu khách hàng từ nguồn này chưa được cập nhật." |

**Hợp nhất hồ sơ**

| Tình huống | Câu chữ |
|---|---|
| Hợp nhất xong | "Đã hợp nhất {N} mã định danh vào hồ sơ {mã}. Lịch sử giao dịch và điểm số đã được tính lại." |
| Đánh dấu khác người | "Đã ghi nhận đây là các khách hàng khác nhau. Hệ thống sẽ không đề xuất hợp nhất các mã này nữa." |

**Cần phê duyệt**

| Tình huống | Câu chữ |
|---|---|
| Vượt ngưỡng xuất trực tiếp | "Tệp {N} bản ghi vượt mức được xuất trực tiếp. Yêu cầu đã gửi tới {người duyệt} chờ phê duyệt." |
| Vượt trần cứng | "Không thể xuất quá 100.000 bản ghi trong một lần. Vui lòng thu hẹp điều kiện lọc." |

**Vướng đồng ý dữ liệu**

| Tình huống | Câu chữ |
|---|---|
| Một phần bị loại | "{N} khách hàng trong phân khúc chưa đồng ý nhận {kênh}. Hệ thống đã loại khỏi tệp gửi." |
| Toàn bộ bị loại | "Không có khách hàng nào trong phân khúc này đủ điều kiện nhận {kênh}. Tệp gửi trống." |

---

## 8. Bối cảnh hệ thống

| Nội dung | Chi tiết |
|---|---|
| Thông tin nghiệp vụ cần lưu | Mười nhóm dữ liệu của hồ sơ hợp nhất (mục 6.2) · sơ đồ liên kết định danh · bảng ánh xạ mã nguồn về mã hợp nhất · trạng thái đồng ý · phân khúc và điểm số · nhật ký |
| Dịch vụ bên ngoài | Zalo OA · cổng tin nhắn · cổng thư điện tử · Facebook · sàn thương mại điện tử · ngân hàng · hệ thống địa chỉ số và bản đồ (chuẩn hóa địa chỉ) · hệ thống định danh người dùng VNPost |
| Kênh thông báo cho người dùng nội bộ | Thư điện tử hoặc thông báo đẩy |
| Xử lý chạy nền | Đồng bộ theo lô 01:00–05:00 · tính lại điểm khách hàng · cập nhật phân khúc động · dọn hàng đợi lỗi quá 30 ngày · rà soát đồng ý sắp hết hạn |
| Yêu cầu thời gian thực | Chỉ nhóm hành vi số và tạo đơn (≤ 5 phút); các nhóm khác chấp nhận trễ hơn |
| Ưu tiên tích hợp nguồn | Ưu tiên 1: nền tảng tích hợp trung tâm, hệ thống chấp nhận gửi và cổng khách hàng lớn, ứng dụng khách hàng, hệ thống định danh, ứng dụng bưu tá. Ưu tiên 2: hệ thống thanh toán, quan hệ khách hàng, khai thác bưu chính. Ưu tiên 3: vận tải, kho, kênh truyền thông, địa chỉ số |

---

## 9. Rủi ro nghiệp vụ

Ba rủi ro đầu là nặng nhất.

| Mã | Rủi ro | Khả năng | Hậu quả nghiệp vụ | Cách phòng |
|---|---|---|---|---|
| R1 | Đồng ý của 600.000 hồ sơ cũ không có bằng chứng lưu vết | Thường | Không kích hoạt được chiến dịch nào hợp pháp; hai use case giữ chân và kéo lại khách hàng mất phần lớn tệp đích ngay từ đầu | Rà soát ngay tỷ lệ hồ sơ có bằng chứng đồng ý; lên kế hoạch thu lại đồng ý qua ứng dụng và quầy trước khi chạy chiến dịch đầu tiên |
| R2 | Dữ liệu nguồn chất lượng kém, hợp nhất ra hồ sơ sai | Thường | Chăm sóc khách hàng tra cứu thấy sai, mất niềm tin, quay lại dùng hệ thống cũ — CDP thành kho dữ liệu chết | Đặt chỉ tiêu chất lượng dữ liệu ngay từ giai đoạn đầu; có bảng theo dõi và danh sách xử lý dữ liệu lỗi; công bố mức độ đầy đủ của từng hồ sơ để người dùng biết độ tin cậy |
| R3 | Không có người đối soát định danh chuyên trách | Thường | Hàng đợi tồn đọng, hồ sơ tiếp tục phân mảnh, hồ sơ khách hàng không bao giờ đầy đủ | Chốt vai trò chuyên trách và cam kết xử lý 2 ngày làm việc trước khi vận hành; theo dõi thời gian xử lý trung bình từ ngày đầu |
| R4 | Hệ thống cũ không cho kết nối, chỉ xuất tệp thưa | Thỉnh thoảng | Thiếu dữ liệu kết quả phát và lý do không phát được — không đủ dữ liệu chạy phân tích giảm hoàn hàng | Xác nhận sớm khả năng kết nối; nếu chỉ có tệp theo ngày thì điều chỉnh kỳ vọng về phân tích thời gian thực |
| R5 | Nền tảng tích hợp trung tâm không mở kết nối cho CDP | Thỉnh thoảng | Phải tích hợp riêng từng hệ thống, khối lượng và thời gian tăng nhiều lần | Đưa vào danh sách câu hỏi ưu tiên cao nhất với bộ phận công nghệ thông tin; chuẩn bị phương án tích hợp riêng lẻ |
| R6 | Phạm vi quá rộng, 99 yêu cầu chức năng làm dàn trải | Thỉnh thoảng | Sau 12 tháng không có phần nào dùng được trọn vẹn, khó thuyết phục đầu tư tiếp | Chia lô theo phân hệ, mỗi lô có kết quả dùng được; nghiệm thu theo lô thay vì chờ toàn bộ |
| R7 | Đơn vị tỉnh không đồng thuận chia sẻ dữ liệu khách hàng | Thỉnh thoảng | Dữ liệu khuyết theo vùng, báo cáo toàn quốc lệch, phát sinh tranh chấp quyền xem | Làm rõ phạm vi phân quyền theo đơn vị từ đầu; có người bảo trợ ở cấp tổng công ty |

---

## 10. Tiêu chí thành công

Bộ chỉ tiêu dưới đây lấy từ `clarification.md`, **chưa được VNPost xác nhận**.

| Chỉ tiêu | Mục tiêu |
|---|---|
| Tỷ lệ giữ chân khách hàng lớn | Tăng ít nhất 5% trong 12 tháng đầu |
| Thời gian phát hiện khách hàng có nguy cơ rời bỏ | Từ "không biết" xuống dưới 7 ngày |
| Tỷ lệ hoàn hàng | Giảm 1–2% ở nhóm người gửi được phân tích rủi ro |
| Tỷ lệ khách hàng lớn có hồ sơ hợp nhất | Ít nhất 80% sau giai đoạn 1 |

---

## 11. Đăng ký giả định

| Mã | Giả định | Nếu sai thì phải điều chỉnh gì |
|---|---|---|
| GD-01 | Quy mô người dùng nội bộ 200–500 tài khoản, 50–100 người dùng đồng thời lúc cao điểm | Yêu cầu phi chức năng về hiệu năng và quy mô hạ tầng |
| GD-02 | Toàn bộ con số giới hạn ở mục 7.2 do người phân tích đề xuất, chưa được VNPost duyệt | Rà lại các quy tắc nghiệp vụ và ca kiểm thử phụ thuộc con số |
| GD-03 | Hạn nội bộ xử lý yêu cầu khách hàng đặt chặt hơn trần luật | Cơ chế nhắc và báo vượt hạn |
| GD-04 | Nhật ký gộp hồ sơ và nhật ký đồng ý lưu 5 năm | Yêu cầu lưu trữ và chính sách xóa dữ liệu |
| GD-05 | Phân khúc có hai trạng thái đang hoạt động và tạm dừng, phân loại động và tĩnh | Bảng chuyển trạng thái của phân khúc và màn quản lý phân khúc |
| GD-06 | Chỉ tiêu chất lượng dữ liệu theo hai mốc 6 và 12 tháng | Cách đo và ngưỡng cảnh báo chất lượng dữ liệu |
| GD-07 | Tiêu chí thành công giữ theo bộ đang giả định | Toàn bộ phần mục tiêu và cách đo hiệu quả |
| GD-08 | CDP nhận danh tính từ cổng đăng nhập chung, không tự quản lý tài khoản | Phải bổ sung phân hệ quản lý tài khoản và xác thực |

---

## 12. Câu hỏi mở

| Mã | Câu hỏi | Người trả lời |
|---|---|---|
| OQ-01 | Use case nào ưu tiên cho giai đoạn đầu? | Chủ sản phẩm / VNPost |
| OQ-02 | "Khách hàng" trong CDP gồm người gửi, hay cả người nhận? Nếu có người nhận thì cơ chế đồng ý cho nhóm chưa từng đăng ký là gì? | Chủ sản phẩm / Pháp chế |
| OQ-03 | Hệ thống định danh người dùng VNPost hiện phủ bao nhiêu phần trăm khách hàng? Khách giao dịch tại quầy có mã định danh không? | Công nghệ thông tin VNPost |
| OQ-04 | Nền tảng tích hợp trung tâm có thể làm cổng dữ liệu cho CDP không? Nếu có thì cung cấp được những nhóm dữ liệu nào? | Công nghệ thông tin VNPost |
| OQ-05 | VNPost đã chuẩn bị đến đâu về tuân thủ bảo vệ dữ liệu cá nhân? Ai chịu trách nhiệm pháp lý? | Pháp chế / Tuân thủ |
| OQ-06 | Trong 600.000 hồ sơ hiện có, bao nhiêu phần trăm có bằng chứng đồng ý lưu vết được, và đồng ý đó có nêu rõ mục đích tiếp thị và phân tích không? | Pháp chế / Công nghệ thông tin |
| OQ-07 | Quy mô người dùng nội bộ thực tế — số tài khoản và số người dùng đồng thời? | VNPost |
| OQ-08 | Màn nào thực sự cần dùng được trên điện thoại? Màn tra cứu hồ sơ thì hợp lý, còn màn tạo phân khúc và bảng đối chiếu nhiều cột rất khó dùng trên màn hình nhỏ | Chủ sản phẩm |
| OQ-09 | Thời hạn lưu nhật ký 5 năm có đúng quy định nội bộ và quy định pháp luật không? | Pháp chế |
| OQ-10 | Bản giao diện đang chạy dùng ngưỡng 90% và 60%, tài liệu gốc quy định 95%, 85% và 70%. Sửa bản đang chạy theo tài liệu gốc, hay cập nhật tài liệu gốc theo bản đang chạy? | Chủ sản phẩm / VNPost |
| OQ-11 | Khi nào mở lại luồng tách hồ sơ? Đây là yêu cầu ưu tiên cao trong tài liệu gốc, hiện đang hoãn | Chủ sản phẩm |
| OQ-12 | VNPost đã có chính sách tần suất gửi tin cho khách hàng chưa? Nếu có thì lấy theo chính sách đó thay cho con số đề xuất | Tiếp thị VNPost |

**Đã chốt trong buổi này:** phạm vi tài liệu · người đọc · cách xử lý điểm chưa có câu trả lời · bộ vai trò · ranh giới đăng nhập · ranh giới với khách hàng cuối · cách xử lý khi hai người cùng thao tác · phạm vi hiệu lực của việc rút đồng ý · cách xử lý khi sửa phân khúc đang chạy chiến dịch · toàn bộ con số giới hạn · hai mốc thời hạn xử lý yêu cầu khách hàng · bộ câu chữ thông báo · ba rủi ro nặng nhất.

---

## 13. Bước tiếp theo

1. Trả lời OQ-01 đến OQ-05 với khách hàng — năm câu này ảnh hưởng phạm vi và kiến trúc, càng chậm càng rủi ro
2. Chốt OQ-10 trước khi viết phần hợp nhất định danh trong URD, vì con số ngưỡng đi thẳng vào quy tắc nghiệp vụ và ca kiểm thử
3. Chạy `/urd cdp` theo lô từng phân hệ, thứ tự đề xuất: hợp nhất định danh và hồ sơ khách hàng 360 trước (đã có giao diện và đã làm rõ nhất), rồi tiếp nhận và chuẩn hóa, sau đó phân khúc và phân tích, kích hoạt, cuối cùng là quản trị
