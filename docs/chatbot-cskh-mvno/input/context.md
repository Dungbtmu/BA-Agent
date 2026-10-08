# Context dự án — Trợ lý ảo AI all-in-one (chatbot-cskh-mvno)

> Tài liệu ghi nhận bối cảnh đầu vào của dự án, tổng hợp từ yêu cầu của Ban và meeting note buổi họp đầu tiên với Ban.
> Đây là tài liệu **input gốc** — ghi lại nguyên văn nội dung đã nhận, không phải kết quả phân tích của BA.

---

## 1. Yêu cầu của Ban — 7 nhóm nghiệp vụ / lĩnh vực

| STT | Lĩnh vực | Nội dung, nghiệp vụ cần xử lý |
|---|---|---|
| 1 | Viễn thông | - Tra cứu thuê bao, cước, data, gói cước bằng các cách thức khác nhau như hình ảnh, giọng nói, văn bản,...<br>- Tư vấn và đề xuất gói phù hợp theo nhu cầu sử dụng và thói quen của khách hàng.<br>- Đăng ký/gia hạn/hủy dịch vụ; xử lý sự cố.<br>- Quản lý các dịch vụ đang sử dụng. |
| 2 | Cá nhân và công việc | - Quản lý lịch, lịch họp, nhắc việc, ghi chú; tạo và quản lý task.<br>- Tổng hợp công việc trong ngày; nhắc việc theo deadline đặt trước.<br>- Soạn Email, văn bản và đề xuất kế hoạch theo chủ đề có sẵn.<br>- Chuyển giọng nói thành văn bản.<br>- Tạo Slide báo cáo.<br>- Phân tích hình ảnh và tài liệu. |
| 3 | Tài chính & thanh toán | - Tra cứu giao dịch, số dư tài khoản.<br>- Nhắc lịch thanh toán và thanh toán hóa đơn cước di động, điện nước,...<br>- Phân tích chi tiêu; cảnh báo giao dịch bất thường; hỗ trợ các sản phẩm tài chính. |
| 4 | Giải trí | - Gợi ý phim, nhạc, game, truyền hình theo sở thích hoặc tâm trạng,...<br>- Tìm kiếm nội dung; tạo playlist cá nhân hóa nội dung theo sở thích.<br>- Gợi ý đăng ký gói tiện ích, giải trí phù hợp. |
| 5 | Sức khỏe và đời sống cá nhân | - Tổng hợp dữ liệu sức khỏe được cấp quyền; theo dõi vận động/giấc ngủ; nhắc lịch tập luyện thể thao.<br>- Hỗ trợ xây dựng mục tiêu cá nhân; đưa ra khuyến nghị phù hợp theo mục tiêu. |
| 6 | Du lịch và tiện ích đời sống | - Tìm/đặt vé máy bay, tàu, khách sạn theo địa điểm và thời gian khách hàng yêu cầu.<br>- Xây dựng lịch trình ăn uống, vui chơi theo địa điểm. Đặt thêm các dịch vụ tiện ích. |
| 7 | Khác | - Hỗ trợ tra cứu pháp luật, xử lý các nghiệp vụ thủ tục hành chính công tích hợp trên App nhà mạng.<br>- Hỏi đáp các nội dung thường ngày. |

---

## 2. Meeting note — buổi họp đầu tiên với Ban

### 2.1. Mục tiêu

Xây dựng trợ lý AI all-in-one theo hướng GenAI, thông minh hơn và hỗ trợ nhiều lĩnh vực để tạo khác biệt với chatbot của các nhà mạng khác.

### 2.2. Phạm vi

- Tích hợp vào **Super App** trước mắt.
- Mỗi lĩnh vực được tổ chức thành **một module riêng**, có **dữ liệu riêng để training**.
- Phần chia module là việc của bên AI / dữ liệu / luồng xử lý. Về mặt trải nghiệm, người dùng chỉ thao tác trên **một giao diện / một đoạn chat duy nhất**.

### 2.3. Người dùng

Người dùng cuối (end user) — thuê bao sử dụng dịch vụ nhà mạng.

### 2.4. Ngôn ngữ hỗ trợ

Tiếng Việt, tiếng Anh và tiếng Trung.

### 2.5. Dữ liệu và hiện trạng

- Dữ liệu lấy từ nhiều nguồn, không chỉ Super App:
  - **Super App:** các nhóm nghiệp vụ đã có sẵn chức năng/nội dung trên App (ví dụ giải trí, đặt vé máy bay).
  - **Hệ thống BSS:** dữ liệu nhóm Viễn thông. BSS đang trong quá trình triển khai, nhưng đã có thông tin BSS **có thể trả về API** cho phần Viễn thông.
- App chưa lên store.
- Cần họp với các đầu mối để làm rõ thông tin, dữ liệu hiện có và dữ liệu có thể cung cấp.

### 2.6. Phạm vi MVP — theo từng nhóm nghiệp vụ

Tập trung vào các nhóm nghiệp vụ khả thi:

| Nhóm nghiệp vụ | Hiện trạng / hướng xử lý đã thống nhất | Việc cần làm rõ (theo meeting note) |
|---|---|---|
| Viễn thông | Trước mắt khả thi. Lấy dữ liệu qua API bên BSS (BSS đang triển khai nhưng có thể trả về API cho phần Viễn thông). Cần hỗ trợ người dùng tương tác bằng ngôn ngữ tự nhiên. | Dữ liệu và API từ BSS (chờ cung cấp); tiến độ triển khai BSS. |
| Tài chính & thanh toán | Chưa chắc khả thi. Hiện chưa biết Super App đã có chức năng tài chính & thanh toán hay chưa. | Super App có chức năng này hay chưa; bên Dịch vụ số có thể cung cấp được chưa; cung cấp những thông tin nào. |
| Giải trí | Super App đã có các nội dung này. | Trải nghiệm và trao đổi thêm: những luồng nào cần thông tin gì. |
| Du lịch và tiện ích | Super App đã có chức năng đặt vé máy bay. Lịch trình ăn uống theo địa điểm: trước mắt có thể **hard code các địa điểm cố định** để AI đọc và đưa ra gợi ý. | Phân rã để biết bên App cần cung cấp thông tin gì. |
| Cá nhân và công việc | Phạm vi MVP là **Email** (trên Outlook): lấy dữ liệu, tóm tắt, soạn và gửi email. Bên AI đã phát triển được phần này. | — |
| Các nhóm nghiệp vụ khác | Chưa xác định. | Phân rã và tìm hiểu đầu mối, tình trạng hiện tại. BA bổ sung chi tiết tại mục Đầu ra cần thực hiện. |

> **Lưu ý:** Thông tin trong bảng trên mới là hướng xử lý ở mức định hướng, chưa có chi tiết cụ thể cho nhóm nào. **Tất cả các nhóm nghiệp vụ** (không chỉ nhóm "Các nhóm nghiệp vụ khác") đều cần BA phân rã, tìm hiểu đầu mối và tình trạng hiện tại, rồi bổ sung chi tiết tại mục 2.8 (Đầu ra BA cần thực hiện).

### 2.7. Tích hợp bổ sung

Có nhu cầu kết nối **Google Calendar** và **Apple Calendar** cho nghiệp vụ quản lý lịch họp, nhắc nhở,...

### 2.8. Đầu ra BA cần thực hiện

- Phân rã nghiệp vụ, tìm hiểu đầu mối và tình trạng hiện tại cho **tất cả** các nhóm nghiệp vụ (hiện chưa nhóm nào có chi tiết cụ thể).
- Xác định các nội dung cần **TTVT** làm rõ.
- Xây dựng mô tả nghiệp vụ chi tiết để TTVT triển khai UI/UX.
- Với từng nhóm nghiệp vụ, BA phải phân tích và chỉ rõ:
  - Cần thông tin gì, lấy từ hệ thống nào.
  - Bên hệ thống nguồn cần cung cấp thông tin gì.
  - Thông tin / luồng nào do BA phân tích; luồng nào do bên AI xử lý.
  - Data flow đi như thế nào.
- Mục tiêu: BA define rõ phần nghiệp vụ của các bên để từng bên biết mình cần làm gì.

---

## 3. Ghi chú của BA

- BA đang chưa rõ phạm vi công việc của BA khi triển khai một hệ thống chatbot kết hợp cả luồng nghiệp vụ và AI → cần làm rõ (xem mục 4).

---

## 4. Điểm cần làm rõ từ input

| # | Nội dung | Nguồn |
|---|---|---|
| 1 | Nhóm 5 (Sức khỏe và đời sống cá nhân) và nhóm 7 (Khác) không được nêu tên riêng trong danh sách MVP — giả định tạm thời thuộc dòng "Các nhóm nghiệp vụ khác", chưa vào MVP. | Mục 1 và 2.6 |
| 2 | Nhóm Cá nhân và công việc: MVP chỉ nêu Email. Các nghiệp vụ còn lại (lịch, task, Slide, chuyển giọng nói thành văn bản, phân tích hình ảnh/tài liệu) chưa rõ có vào MVP hay không; lịch có nhu cầu tích hợp bổ sung (Mục 2.7). | Mục 1 và 2.6 |
| 3 | Tên viết tắt **TTVT** xuất hiện trong meeting note — cần xác nhận đơn vị này là đầu mối triển khai UI/UX. | Mục 2.8 |
