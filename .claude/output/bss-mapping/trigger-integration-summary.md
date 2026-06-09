# Bảng Tổng Hợp Trigger/Event Theo Cơ Chế Tích Hợp

> Mục đích: giúp BA/PO/Dev/Tester theo dõi nhanh trigger/event theo **nhóm vòng đời**, **nhánh nghiệp vụ** và loại xử lý `Realtime` / `Near Realtime` / `Offline/Batch`.
>
> Nguồn đối chiếu chính: `data-contract-template.md`.

| Nhóm vòng đời | Nhánh nghiệp vụ | Mã | Tên trigger/event | Loại xử lý | Cơ chế tích hợp | Timing/SLA chính | Ghi chú |
|---|---|---|---|---|---|---|---|
| NHÓM 1 — Kích hoạt | VIỄN THÔNG | `E01` | SIM kích hoạt lần đầu | `Near Realtime` | Push Event API từ BSS sang CVM | 30–60 phút sau khi SIM ACTIVE lần đầu | Mốc `NGAY_0`; dùng `msisdn_status_history.change_date` |
| NHÓM 1 — Kích hoạt | APP | `E02` | Chưa cài app sau 24h kích hoạt SIM | `Offline/Batch` | Push CSV từ BSS sang CVM | BSS push 02:00–04:00; CVM gửi khoảng 09:00 | Batch D+1 |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E05` | Chưa phát sinh cước sau 72h | `Offline/Batch` | OCS nightly sang BSS, BSS push CSV | Push CSV 02:00–04:00 | Tổng data/voice/SMS = 0 trong 3 ngày |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E06` | Cuộc gọi thất bại | `Near Realtime` | OCS gọi API CVM | CVM gửi USSD trong vòng 2 phút | Reactive sau lỗi cuộc gọi |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E08` | Data ngày sắp hết | `Near Realtime` | OCS gọi API CVM khi vượt ngưỡng | CVM xử lý và gửi trong vòng 2 giờ | Ngưỡng do CVM cấu hình |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_VOICE_100_ONNET` | Hết phút thoại nội mạng | `Near Realtime` | OCS gọi API CVM khi quota nội mạng về 0 | CVM gửi trong vòng 2 phút | Tách riêng nếu quota nội/ngoại mạng riêng |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_VOICE_100_OFFNET` | Hết phút thoại ngoại mạng | `Near Realtime` | OCS gọi API CVM khi quota ngoại mạng về 0 | CVM gửi trong vòng 2 phút | Nhạy cảm về cước ngoài gói |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_ZERO_BALANCE` | Số dư TKC về 0 | `Offline/Batch` | OCS → BSS nightly batch, phát hiện qua CDR giao dịch cuối làm cạn kiệt tài khoản | BSS push 02:00–04:00 hàng ngày | File gốc có Q12 hỏi có chuyển được sang realtime không |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_CANCEL_PLAN` | Hủy gói cước | `Near Realtime` | OCS gọi API CVM | CVM gửi trong vòng 5 phút | Giữ chân ngay sau khi hủy |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E04` | Cài app nhưng chưa mở sau 24h | `Offline/Batch` | SuperApp event qua Kafka, BSS tổng hợp CSV | Push CSV 02:00–04:00; gửi Push 08:00–09:00 | Theo file gốc nằm tại mục 2.2 Viễn thông — Batch/CSV |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E13` | Tăng đột biến lưu lượng bất thường | `Offline/Batch` | OCS/CDR batch sang BSS, BSS push CSV | Push CSV 02:00–04:00; gửi 09:00–10:00 | So sánh với baseline 30 ngày |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E09` | Xem màn hình đổi gói nhưng chưa đăng ký | `Offline/Batch` | SuperApp Kafka sang BSS, BSS push CSV | Push CSV 02:00–04:00; popup lần mở app kế tiếp | Theo file gốc nằm tại mục 2.2 Viễn thông — Batch/CSV |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_USAGE_NEED_ANALYSIS` | Phân tích nhu cầu gói theo mức sử dụng | `Offline/Batch` | OCS/BSS phân tích usage theo chu kỳ, push CSV | Push CSV 02:00–04:00 | Đã đổi tên từ trigger early depleted để đúng bản chất nghiệp vụ |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_NO_PLAN_X_DAYS` | ACTIVE nhưng không có gói x ngày | `Offline/Batch` | BSS quét hàng ngày từ trạng thái/gói | Push CSV 02:00–04:00 | Gợi ý đăng ký lại gói |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `E_SEGMENT_UPDATE` | Cập nhật phân khúc KH | `Offline/Batch` | BSS hoặc CVM batch theo chu kỳ phân khúc | Theo lịch batch phân khúc | Nếu CVM tự tính thì không cần BSS push |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U01` | Nạp tiền thành công lần >= 2 | `Near Realtime` | OCS gọi API CVM | CVM gửi trong vòng 3 phút | Tận dụng lúc KH vừa có số dư |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U02` | Đăng ký/gia hạn gói thành công lần >= 2 | `Offline/Batch` | OCS nightly sang BSS, BSS push CSV | Push CSV 02:00–04:00 | Cross-sell sau giao dịch |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U03` | Tra cứu số dư / gắn gợi ý inline | `Realtime` | OCS gọi CVM đồng thời khi xử lý `*101#` | CVM phản hồi trong 2 giây | Nếu timeout, OCS trả kết quả bình thường |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U04` | Nhận OTP từ app bên thứ 3 | `Offline/Batch` | HLR export CSV trực tiếp vào CVM | Push CSV 02:00–04:00 | Không đi qua BSS |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U05-A` | Hết data tháng sớm 2 tháng liên tiếp | `Offline/Batch` | OCS/BSS tổng hợp đầu tháng thứ 3 | Push CSV đầu tháng | Dành cho gói data tháng |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U05-B` | Pattern hết quota data ngày thường xuyên | `Offline/Batch` | OCS/BSS tổng hợp theo tháng | Push CSV đầu tháng | Dành cho gói data ngày/quota reset ngày |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U06` | Chuyển đổi loại gói thành công | `Offline/Batch` | OCS nightly sang BSS, BSS push CSV | Push CSV 02:00–04:00 | Gửi xác nhận và gợi ý bổ sung |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U07` | Chuyển đổi SIM nội mạng | `Offline/Batch` | BSS push CSV từ `sim_swap` | Push CSV 02:00–04:00 | Đổi SIM thường/eSIM/số đẹp |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U09` | Sinh nhật / kỷ niệm KH hoặc SIM | `Offline/Batch` | BSS quét hàng ngày và push CSV | Push CSV 02:00–04:00 | Chúc mừng, tặng quà |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | `U10` | Ngày lễ / sự kiện quốc gia | `Offline/Batch` | BSS chuẩn bị danh sách trước ngày lễ | Push CSV trước ngày lễ 2 ngày | Theo lịch ngày lễ của CVM |
| NHÓM 2 — Sử dụng | APP | `E03` | Đăng nhập app lần đầu | `Realtime` | SuperApp gọi API CVM, CVM trả banner ngay | CVM phản hồi trong 1–2 giây | Hiển thị tức thì trong phiên app |
| NHÓM 2 — Sử dụng | APP | `E_CONTENT_FAIL` | Mua dịch vụ nội dung thất bại | `Near Realtime` | SuperApp/Kafka push event vào CVM | CVM gửi trong vòng 2 phút | Khi KH còn trong phiên |
| NHÓM 2 — Sử dụng | APP | `E_APP_RATING` | KH đánh giá app | `Near Realtime` | SuperApp push event, CVM xử lý async | Ngay khi submit; không cần response realtime | Event-driven nhưng async |
| NHÓM 2 — Sử dụng | APP | `E07` | Milestone 7 ngày / khảo sát ngắn | `Offline/Batch` | SuperApp + OCS batch sang BSS, BSS push CSV | Push CSV 02:00–04:00 | Có thể cảnh báo CSKH nếu điểm gắn kết thấp |
| NHÓM 2 — Sử dụng | APP | `E11` | Tổng kết ngày 30 | `Offline/Batch` | OCS + SuperApp + BSS tổng hợp CSV | Push CSV 02:00–04:00 | File đa nguồn; theo file gốc nằm tại mục 2.4 APP — Batch/CSV |
| NHÓM 3 — Gia hạn gói/dịch vụ | GIA HẠN | `U08` | Gia hạn gói liên tiếp / vinh danh trung thành | `Offline/Batch` | OCS/BSS tổng hợp chuỗi gia hạn, push CSV | Push CSV 02:00–04:00 | Đã gộp trigger renewal streak cũ vào U08 |
| NHÓM 3 — Gia hạn gói/dịch vụ | GIA HẠN | `U_PRE_EXPIRY` | Trước khi gói hết hạn x ngày | `Offline/Batch` | BSS quét `expiry_date` hàng ngày | Push CSV 02:00–04:00 | Nhắc gia hạn trước hạn |
| NHÓM 3 — Gia hạn gói/dịch vụ | GIA HẠN | `U_POST_EXPIRY` | Sau khi gói hết hạn x ngày chưa gia hạn | `Offline/Batch` | BSS quét `expiry_date` hàng ngày | Push CSV 02:00–04:00 | Thúc gia hạn sau hạn |
| NHÓM 4 — Khóa 1c/Khóa 2c | KHÓA 1C/2C | `E_LOCK_2C` | Khóa 2 chiều | `Near Realtime` | BSS/OCS gọi API CVM khi trạng thái chuyển LOCK_2C | Ngay khi tài khoản bị khóa 2 chiều | Gửi hướng dẫn khôi phục |
| NHÓM 4 — Khóa 1c/Khóa 2c | KHÓA 1C/2C | `E_LOCK_1C` | Khóa 1 chiều | `Offline/Batch` | BSS quét danh sách khóa 1 chiều | Push CSV 02:00–04:00 | Tách ADMIN/INACTIVE |
| NHÓM 4 — Khóa 1c/Khóa 2c | KHÓA 1C/2C | `E_PRE_LOCK_2C` | Trước khi khóa 2 chiều | `Offline/Batch` | BSS quét trạng thái khóa 1 chiều + ngày khóa | Push CSV 02:00–04:00 | Cảnh báo deadline khóa 2 chiều |
