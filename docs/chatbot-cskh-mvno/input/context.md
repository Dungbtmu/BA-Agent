# Context dự án — Trợ lý ảo AI all-in-one (chatbot-cskh-mvno)

> Tài liệu ghi nhận bối cảnh đầu vào của dự án, tổng hợp từ yêu cầu của Ban và meeting note buổi họp đầu tiên với Ban.
> Đây là tài liệu **input gốc**: ghi lại nội dung đã nhận, không phải kết quả phân tích của BA.

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

Xây dựng trợ lý AI all-in-one theo hướng GenAI, thông minh hơn và hỗ trợ nhiều lĩnh vực, nhằm tạo khác biệt với chatbot của các nhà mạng khác.

### 2.2. Phạm vi

- Trước mắt tích hợp vào **Super App**.
- Mỗi lĩnh vực được tổ chức thành **một module riêng**, có **dữ liệu riêng để training**.
- Việc chia module thuộc về bên AI / dữ liệu / luồng xử lý. Về trải nghiệm, người dùng chỉ thao tác trên **một giao diện, một đoạn chat duy nhất**.

### 2.3. Người dùng

Người dùng cuối (end user): thuê bao sử dụng dịch vụ nhà mạng.

### 2.4. Ngôn ngữ hỗ trợ

Tiếng Việt, tiếng Anh và tiếng Trung.

### 2.5. Dữ liệu và hiện trạng

Dữ liệu lấy từ nhiều nguồn, không chỉ Super App:

- **Super App:** các nhóm nghiệp vụ mà App đã có sẵn chức năng hoặc nội dung (ví dụ giải trí, đặt vé máy bay).
- **Hệ thống BSS:** dữ liệu nhóm Viễn thông. BSS đang trong quá trình triển khai, nhưng đã xác nhận BSS **có thể trả về API** cho phần Viễn thông.

Hiện trạng khác: App chưa lên store. Cần họp với các đầu mối để làm rõ thông tin, dữ liệu hiện có và dữ liệu có thể cung cấp.

### 2.6. Phạm vi MVP — đánh giá theo từng nhóm nghiệp vụ

MVP tập trung vào các nhóm nghiệp vụ khả thi:

| Nhóm nghiệp vụ | Hiện trạng / hướng xử lý | Việc cần làm rõ |
|---|---|---|
| Viễn thông | Trước mắt khả thi. Lấy dữ liệu qua API của BSS (BSS đang triển khai nhưng có thể trả về API cho phần Viễn thông). Cần hỗ trợ người dùng tương tác bằng ngôn ngữ tự nhiên. | Dữ liệu và API từ BSS (chờ cung cấp); tiến độ triển khai BSS. |
| Tài chính & thanh toán | Chưa chắc khả thi. Hiện chưa biết Super App đã có chức năng tài chính & thanh toán hay chưa. | Super App có chức năng này hay chưa; bên Dịch vụ số có thể cung cấp được chưa và cung cấp những thông tin nào. |
| Giải trí | Super App đã có các nội dung này. | Trao đổi thêm về trải nghiệm: những luồng nào cần thông tin gì. |
| Du lịch và tiện ích | Super App đã có chức năng đặt vé máy bay. Lịch trình ăn uống theo địa điểm: trước mắt có thể **hard code các địa điểm cố định** để AI đọc và đưa ra gợi ý. | Phân rã để biết bên App cần cung cấp thông tin gì. |
| Cá nhân và công việc | Phạm vi MVP là **Email** (trên Outlook): lấy dữ liệu, tóm tắt, soạn và gửi email. Bên AI đã phát triển được phần này. | — |
| Các nhóm nghiệp vụ khác | Chưa xác định. | Phân rã, tìm hiểu đầu mối và tình trạng hiện tại. |

> **Lưu ý:** Bảng trên mới là định hướng, chưa nhóm nào có chi tiết cụ thể. **Tất cả các nhóm nghiệp vụ**, kể cả nhóm đang được đánh giá là chưa khả thi, đều cần BA phân rã, tìm hiểu đầu mối và tình trạng hiện tại, sau đó bổ sung chi tiết theo mục 2.8.

### 2.7. Tích hợp bổ sung

Có nhu cầu kết nối **Google Calendar** và **Apple Calendar** cho nghiệp vụ quản lý lịch họp, nhắc nhở,...

### 2.8. Đầu ra BA cần thực hiện

**Sản phẩm bàn giao:**

- Phân rã nghiệp vụ của **tất cả** các nhóm, kể cả nhóm hoặc nghiệp vụ đang được đánh giá là chưa khả thi.
- Danh sách nội dung cần **TTVT** làm rõ.
- Mô tả nghiệp vụ chi tiết để TTVT triển khai UI/UX.

**Với từng nghiệp vụ sau phân rã, BA phân tích và chỉ rõ:**

- Cần thông tin gì, lấy từ hệ thống nào.
- Hệ thống nguồn cần cung cấp thông tin gì.
- Thông tin, luồng nào do BA phân tích; luồng nào do bên AI xử lý.
- Data flow đi như thế nào.
- Nghiệp vụ cần gì, đang có gì, để giải thích được vì sao khả thi hoặc chưa khả thi, làm căn cứ đánh giá timeline và lập kế hoạch triển khai.

**Mục tiêu:** BA định nghĩa rõ phần nghiệp vụ của các bên để mỗi bên biết mình cần làm gì.

---

## 3. Ghi chú của BA

BA chưa rõ phạm vi công việc của mình khi triển khai một hệ thống chatbot kết hợp cả luồng nghiệp vụ và AI. Nội dung này cần được làm rõ.

---

## 4. Điểm cần làm rõ từ input

| # | Nội dung | Nguồn |
|---|---|---|
| 1 | Nhóm 5 (Sức khỏe và đời sống cá nhân) và nhóm 7 (Khác) không được nêu tên riêng trong danh sách MVP, hiện được đánh giá là **chưa khả thi** (thuộc dòng "Các nhóm nghiệp vụ khác"). Vẫn phải phân rã chi tiết từng nghiệp vụ như các nhóm khác, để biết lý do chưa khả thi và điều kiện để khả thi. | Mục 1 và 2.6 |
| 2 | Nhóm Cá nhân và công việc: MVP chỉ nêu Email. Các nghiệp vụ còn lại (lịch, task, Slide, chuyển giọng nói thành văn bản, phân tích hình ảnh/tài liệu) hiện được đánh giá là chưa khả thi trong MVP. Riêng lịch có nhu cầu tích hợp bổ sung (mục 2.7). Vẫn phải phân rã chi tiết để đánh giá khả thi. | Mục 1, 2.6 và 2.7 |
| 3 | Từ viết tắt **TTVT** xuất hiện trong meeting note. Cần xác nhận đây là đơn vị triển khai UI/UX. | Mục 2.8 |
