# SÁNG KIẾN

## Số hóa toàn diện quy trình thiết lập, kiểm duyệt và vận hành an toàn chiến dịch chăm sóc khách hàng đa kênh

---

### 1. Thông tin chung

| Mục | Nội dung |
|---|---|
| Tên sáng kiến | Số hóa toàn diện quy trình thiết lập, kiểm duyệt và vận hành an toàn chiến dịch chăm sóc khách hàng đa kênh |
| Lĩnh vực áp dụng | Phân tích nghiệp vụ và thiết kế hệ thống Customer Value Management (CVM) — quản lý chiến dịch marketing tự động cho doanh nghiệp viễn thông |
| Phạm vi | Toàn bộ vòng đời một chiến dịch: thiết lập → kiểm duyệt → phát hành → vận hành → dừng/khôi phục |
| Đối tượng hưởng lợi | Quản trị viên Marketing (người tạo và vận hành chiến dịch), Quản trị viên Hệ thống (người duyệt và kiểm soát rủi ro), gián tiếp là khách hàng cuối (không bị làm phiền bởi tin nhắn cấu hình sai) |

---

### 2. Hiện trạng và lý do đề xuất sáng kiến

#### 2.1. Bối cảnh nghiệp vụ

Hệ thống CVM cho phép đội ngũ Marketing tự cấu hình chiến dịch gửi tin tự động (Push, Zalo OA, SMS, USSD, Banner, Email) khi khách hàng phát sinh sự kiện phù hợp — không cần Dev can thiệp cho từng chiến dịch. Đây là mô hình tự phục vụ (self-service), giúp Marketing chủ động về tốc độ ra chiến dịch. Tuy nhiên, mô hình tự phục vụ luôn đi kèm rủi ro: người cấu hình không phải Dev nên rất dễ để lọt cấu hình sai (thiếu nội dung, tham số không hợp lệ, quên chặn khách hàng đã từ chối nhận tin) xuống môi trường vận hành thật — hậu quả là gửi nhầm, gửi sai, hoặc gửi cho người không được phép nhận, gây thiệt hại uy tín và rủi ro tuân thủ.

#### 2.2. Vấn đề cụ thể trước khi có giải pháp

Trong quá trình phân tích và đặc tả nghiệp vụ hệ thống CVM, các vấn đề sau được nhận diện là **lỗ hổng kiểm soát chất lượng** nếu không được thiết kế thành cơ chế bắt buộc trên hệ thống mà chỉ trông chờ vào tính cẩn thận của người thao tác:

1. **Rủi ro cấu hình nội dung sai nhưng vẫn được lưu và gửi duyệt**: người tạo chiến dịch có thể gõ nhầm tham số cá nhân hóa (ví dụ `{{ten_kh}}`) không thuộc sự kiện kích hoạt (trigger) đã chọn, hoặc dùng ảnh Banner sai tỉ lệ, nội dung SMS vượt ngưỡng ký tự mà không biết — nếu không được chặn ngay tại thời điểm soạn thảo, các lỗi này chỉ được phát hiện khi chiến dịch đã chạy, hoặc tệ hơn là khi khách hàng đã nhận được tin nhắn lỗi.
2. **Rủi ro "gãy" cấu hình khi dữ liệu nền thay đổi mà không có cảnh báo**: hệ thống cho phép Quản trị viên Hệ thống điều chỉnh hoặc vô hiệu hóa các tham số/điều kiện lọc gắn với sự kiện kích hoạt (trigger) để phục vụ nhu cầu nghiệp vụ mới. Nếu không có cơ chế phát hiện, các chiến dịch đang chạy dựa trên tham số/điều kiện đó sẽ tiếp tục gửi tin với nội dung sai hoặc lọc sai đối tượng nhận — mà không ai biết cho đến khi có khiếu nại.
3. **Rủi ro bỏ sót lớp an toàn (chặn số điện thoại từ chối nhận tin) khi vận hành nhiều chiến dịch song song**: danh sách chặn phục vụ nhiều mục đích khác nhau (theo từng chiến dịch cụ thể, hoặc chặn toàn diện mọi chiến dịch) — nếu không được thiết kế thành 2 lớp rõ ràng và đồng bộ với chính màn hình cấu hình chiến dịch, người vận hành dễ nhầm lẫn phạm vi chặn, dẫn đến chặn thiếu (khách hàng đã khiếu nại vẫn nhận được tin) hoặc chặn thừa (khách hàng hợp lệ bị chặn nhầm).
4. **Rủi ro khôi phục chiến dịch sai thời điểm sau khi tạm dừng**: khi một chiến dịch đang tạm dừng cần kích hoạt lại, nếu hệ thống không phân biệt được "dữ liệu nền có thay đổi trong lúc tạm dừng hay không", việc cho phép khôi phục ngay lập tức có thể khiến chiến dịch chạy tiếp với cấu hình đã lỗi thời mà không qua bất kỳ bước xác nhận lại nào.

#### 2.3. Hệ quả nếu không xử lý

- Gửi tin nhắn có nội dung lỗi (tham số rỗng, sai ngữ cảnh) đến số lượng lớn khách hàng cùng lúc — vì bản chất chiến dịch là gửi hàng loạt, một lỗi nhỏ ở khâu cấu hình sẽ bị nhân rộng ngay lập tức.
- Vi phạm cam kết không gửi tin cho khách hàng đã từ chối nhận tin (DNC) hoặc đã yêu cầu chặn (Blacklist) — rủi ro về uy tín thương hiệu và tuân thủ quy định viễn thông.
- Chi phí xử lý sự cố sau phát hành cao hơn nhiều lần so với chi phí ngăn chặn tại thời điểm cấu hình — phải rà soát thủ công từng chiến dịch, từng khách hàng đã nhận tin để khắc phục hậu quả.
- Người vận hành mất niềm tin vào tính tự phục vụ của hệ thống, có xu hướng quay lại xin xác nhận thủ công từ Dev/BA trước mỗi lần phát hành — làm mất chính lợi ích tốc độ mà mô hình tự phục vụ hướng tới.

---

### 3. Nội dung giải pháp (điểm mới của sáng kiến)

Xuất phát từ việc phân tích sâu các kịch bản rủi ro nêu trên trong quá trình đặc tả yêu cầu, giải pháp được đề xuất và thiết kế thành một **chuỗi kiểm soát khép kín xuyên suốt vòng đời chiến dịch**, gồm 4 lớp bảo vệ hoạt động tại đúng thời điểm rủi ro phát sinh — thay vì dồn việc kiểm tra vào một bước duy nhất trước khi phát hành.

#### 3.1. Lớp 1 — Chặn lỗi cấu hình ngay tại thời điểm soạn thảo (Validate-at-source)

Thay vì để người soạn thảo tự chịu trách nhiệm phát hiện lỗi, hệ thống chủ động:
- Kiểm tra thời gian thực (real-time) nội dung tin nhắn ngay khi đang gõ: phát hiện tham số cá nhân hóa không thuộc sự kiện kích hoạt đã chọn, cảnh báo ngay tại vị trí lỗi thay vì báo chung chung; tự động tính và cảnh báo số đoạn tin nhắn SMS vượt ngưỡng ký tự (phân biệt có dấu/không dấu tiếng Việt) để người soạn kiểm soát được chi phí gửi tin trước khi phát hành.
- Chặn cứng việc lưu cấu hình (không chỉ cảnh báo) đối với các lỗi có khả năng gây gửi sai ngay từ gốc — ví dụ tham số cá nhân hóa không tồn tại — đảm bảo lỗi này không thể tồn tại ở bất kỳ trạng thái nào của chiến dịch, kể cả bản nháp.

**Điểm mới**: việc kiểm tra được đẩy về đúng thời điểm phát sinh lỗi (khi gõ) thay vì thời điểm phát hành — giảm số vòng lặp sửa đi sửa lại giữa các bước.

#### 3.2. Lớp 2 — Cơ chế "vô hiệu hóa mềm" thay cho xóa cứng dữ liệu nền dùng chung (Soft-lock thay vì Hard-delete)

Tham số cá nhân hóa và điều kiện lọc phân khúc gắn với mỗi sự kiện kích hoạt là dữ liệu nền được nhiều chiến dịch tham chiếu cùng lúc. Giải pháp thiết kế cơ chế **Khóa/Mở khóa** thay cho xóa cứng:
- Khi cần ngừng sử dụng một tham số hoặc điều kiện lọc, người quản trị **Khóa** thay vì xóa — dữ liệu và các liên kết với chiến dịch đang dùng vẫn được giữ nguyên vẹn, tránh gãy cấu trúc dữ liệu.
- Trước khi khóa, hệ thống chủ động liệt kê đầy đủ danh sách chiến dịch sẽ bị ảnh hưởng, buộc người quản trị xác nhận có ý thức về phạm vi tác động — không thể khóa "vô tình".
- Tự động phát hiện và gắn cờ cảnh báo trên đúng những chiến dịch thực sự bị ảnh hưởng (có tham chiếu trực tiếp đến phần vừa thay đổi) — không cảnh báo tràn lan gây nhiễu cho các chiến dịch không liên quan.

**Điểm mới**: đây là cách tiếp cận đảo ngược tư duy thông thường (xóa dữ liệu không dùng nữa) — chuyển sang tư duy bảo toàn dữ liệu và chủ động truy vết tác động trước khi thay đổi, phù hợp với đặc thù dữ liệu dùng chung nhiều nơi.

#### 3.3. Lớp 3 — Kiểm duyệt hai tầng trước khi phát hành (Dual-layer Approval)

Chiến dịch chỉ được phép hoạt động thật khi vượt qua đồng thời hai tầng kiểm soát độc lập:
- **Tầng tự động**: hệ thống tự chặn nút phát hành nếu còn bất kỳ vấn đề nào chưa xử lý (thiếu nội dung bắt buộc, tham số lỗi, thiếu danh sách chặn khi đã bật, ảnh minh họa sai định dạng...) — hiển thị rõ số lượng vấn đề còn tồn và điều hướng trực tiếp đến đúng vị trí cần sửa.
- **Tầng con người**: sau khi vượt qua tầng tự động, chiến dịch vẫn phải qua một người kiểm duyệt độc lập (không phải người tạo) xem xét toàn bộ nội dung, đối tượng nhận, và cấu hình an toàn trước khi cho phép vận hành thật.

**Điểm mới**: tách bạch rõ trách nhiệm — máy chịu trách nhiệm bắt lỗi kỹ thuật/cấu hình, con người chịu trách nhiệm phán đoán về mặt nội dung và chiến lược — không dồn toàn bộ gánh nặng kiểm tra cho một phía.

#### 3.4. Lớp 4 — Lưới an toàn hai phạm vi tích hợp trực tiếp vào luồng gửi tin (Layered Suppression)

Danh sách chặn số điện thoại được thiết kế thành hai phạm vi độc lập nhưng cùng vận hành trong một luồng kiểm tra duy nhất trước khi gửi tin:
- **Phạm vi hẹp**: chặn theo từng cặp chiến dịch — kênh cụ thể, phục vụ nhu cầu vận hành hàng ngày của người tạo chiến dịch.
- **Phạm vi rộng**: chặn một số điện thoại khỏi toàn bộ chiến dịch, toàn bộ kênh — dành riêng cho quản trị viên xử lý các trường hợp cần chặn dứt điểm (ví dụ khách hàng đã khiếu nại nghiêm trọng).
- Hai phạm vi này được đồng bộ hai chiều với chính màn hình cấu hình chiến dịch: chọn số chặn ngay trong lúc soạn chiến dịch sẽ tự động xuất hiện ở màn hình quản lý tập trung và ngược lại — người vận hành luôn nhìn thấy một nguồn dữ liệu duy nhất, không phải đối chiếu thủ công giữa nhiều nơi.
- Khi kích hoạt lại một chiến dịch đã tạm dừng, hệ thống tự động phân biệt trường hợp cấu hình nền có thay đổi trong lúc tạm dừng hay không, để quyết định cho khôi phục ngay hay bắt buộc quay lại bước duyệt.

**Điểm mới**: gộp hai nhu cầu tưởng chừng mâu thuẫn (chặn linh hoạt theo từng chiến dịch và chặn triệt để toàn hệ thống) vào một mô hình dữ liệu thống nhất, tránh tình trạng tồn tại nhiều "phiên bản sự thật" khác nhau về việc một số điện thoại có đang bị chặn hay không.

---

### 4. Tính mới và tính sáng tạo

So với cách làm phổ biến (thường chỉ có một bước kiểm duyệt cuối cùng trước khi phát hành, các thay đổi dữ liệu nền không được đối chiếu ngược với các chiến dịch đang dùng), sáng kiến này có các điểm mới:

1. **Chuyển từ kiểm soát một điểm sang kiểm soát nhiều lớp tại đúng thời điểm rủi ro phát sinh** — mỗi loại rủi ro được chặn ngay tại nơi nó sinh ra (lúc soạn, lúc thay đổi dữ liệu nền, lúc duyệt, lúc gửi), thay vì dồn tất cả trách nhiệm vào bước duyệt cuối.
2. **Tư duy bảo toàn dữ liệu (Khóa thay vì Xóa) áp dụng nhất quán cho dữ liệu dùng chung** — giải quyết triệt để bài toán kinh điển trong hệ thống có tham chiếu chéo: sửa/xóa dữ liệu gốc làm hỏng dữ liệu phái sinh mà không ai biết.
3. **Cơ chế truy vết tác động chủ động (Impact Preview) trước khi thực hiện thay đổi** — không chờ sự cố xảy ra rồi mới điều tra, mà bắt buộc nhìn thấy hậu quả trước khi hành động.
4. **Đồng bộ hai chiều giữa cấu hình phân tán (trong từng chiến dịch) và quản lý tập trung (danh sách chặn)** — loại bỏ tình trạng dữ liệu trùng lặp, không nhất quán vốn rất phổ biến khi một khái nghiệp vụ (số bị chặn) được thao tác từ nhiều màn hình khác nhau.

---

### 5. Hiệu quả áp dụng (ước tính định lượng)

| Tiêu chí | Trước khi áp dụng giải pháp | Sau khi áp dụng giải pháp | Ghi chú |
|---|---|---|---|
| Thời điểm phát hiện lỗi cấu hình tham số | Sau khi chiến dịch đã gửi tin (phát hiện qua khiếu nại/báo cáo) | Ngay khi soạn thảo, trước khi lưu | Giảm 100% khả năng lỗi tham số lọt xuống môi trường vận hành thật |
| Số bước xử lý khi dữ liệu nền (trigger) thay đổi | Rà soát thủ công toàn bộ chiến dịch đang chạy để tìm chiến dịch bị ảnh hưởng | Hệ thống tự động xác định và gắn cờ đúng chiến dịch bị ảnh hưởng | Ước tính giảm phần lớn thời gian rà soát thủ công, tỉ lệ thuận với số lượng chiến dịch đang vận hành đồng thời |
| Rủi ro gửi tin cho số đã yêu cầu chặn | Phụ thuộc vào việc người vận hành nhớ đối chiếu nhiều danh sách rời rạc | Tự động kiểm tra tập trung, đồng bộ hai chiều, không phụ thuộc trí nhớ người thao tác | Giảm thiểu triệt để rủi ro bỏ sót thủ công |
| Số lần chiến dịch bị dừng đột ngột do lỗi phát hiện muộn | Có khả năng xảy ra bất kỳ lúc nào trong quá trình vận hành | Được chặn từ giai đoạn soạn thảo và duyệt | Giảm gián đoạn vận hành, tăng độ tin cậy của lịch chạy chiến dịch |
| Mức độ tin cậy của mô hình tự phục vụ (self-service) | Người vận hành có xu hướng cần xác nhận thêm từ BA/Dev trước khi an tâm phát hành | Người vận hành có thể tự tin phát hành khi hệ thống báo không còn vấn đề nào tồn đọng | Rút ngắn thời gian từ ý tưởng chiến dịch đến khi vận hành thật |

*Ghi chú: các số liệu định lượng chính xác (số giờ tiết kiệm, số sự cố giảm) cần được đo đạc thực tế sau khi hệ thống được Dev triển khai và đưa vào vận hành theo đúng đặc tả — số liệu trong bảng trên là ước tính định tính dựa trên phân tích quy trình.*

---

### 6. Khả năng áp dụng và nhân rộng

- **Áp dụng trực tiếp**: toàn bộ chức năng Quản lý Chiến dịch, Quản lý Sự kiện kích hoạt và Quản lý Danh sách chặn của hệ thống CVM.
- **Khả năng nhân rộng**: nguyên tắc "Khóa thay vì Xóa cho dữ liệu dùng chung" và "kiểm duyệt hai tầng tự động — con người" là các nguyên lý thiết kế tổng quát, có thể áp dụng cho bất kỳ hệ thống nào có đặc điểm tương tự — nhiều người dùng tự cấu hình dựa trên dữ liệu nền dùng chung, và có yêu cầu kiểm soát rủi ro trước khi phát hành ra diện rộng (ví dụ các hệ thống quản lý khách hàng, quản lý nội dung khác đang được triển khai song song).

---

*Sáng kiến được đúc kết trong quá trình phân tích và đặc tả yêu cầu nghiệp vụ cho hệ thống CVM, dựa trên việc nhận diện các kịch bản rủi ro thực tế qua nhiều vòng làm việc với các bên liên quan.*
