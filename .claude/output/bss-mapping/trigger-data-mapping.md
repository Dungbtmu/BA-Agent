# Trigger Data Mapping — CVM × BSS

> Cập nhật: 2026-05-25 — Đồng bộ tên field theo data-contract-template.md v2026-05-14

**Chú thích nguồn dữ liệu:**
- 🟡 BSS — dữ liệu có sẵn trong DB BSS (bảng/trường cụ thể)
- 🔴 OCS — dữ liệu sử dụng realtime, BSS không có, cần OCS cung cấp
- 🔵 SuperApp — hành vi trong ứng dụng, BSS không có, cần SuperApp push
- ⚪ HLR — Home Location Register (Hệ thống Đăng ký vị trí thuê bao), export CSV trực tiếp vào CVM, không qua BSS
- ❓ Chưa rõ nguồn — cần clarify với PO/Tech

**Chú thích cơ chế:**
- **Push Event (API)** — nguồn chủ động gửi event vào CVM ngay khi sự kiện xảy ra, không delay
- **Push Event (Kafka)** — nguồn đẩy event vào message queue Kafka, CVM consume từ topic tương ứng
- **Push CSV** — BSS chủ động tạo file CSV và đẩy vào thư mục/SFTP mà CVM đọc theo lịch

---

## G3 — THÍCH NGHI & HÌNH THÀNH THÓI QUEN (Ngày 0–30)

### G3S1 — Giai đoạn con 1: Ngày 0–7

| Mã | Tên trigger | Mục đích | Loại | Điều kiện kích hoạt | Cơ chế & Timing | Điều kiện chặn | Luồng dữ liệu | Kênh gửi |
|---|---|---|---|---|---|---|---|---|
| G3S1.E01 | Kích hoạt SIM lần đầu → Mốc NGAY_0 | Chào mừng KH ngay khi SIM hoạt động lần đầu. Ghi nhận NGAY_0 — điểm gốc T=0 của toàn bộ hành trình G3, khởi động lịch trigger tiếp theo. | NearRealtime — Push Event (API) | `resource.msisdn_status_history` ghi nhận `to_state_id` = ACTIVATED lần đầu tiên (COUNT = 1) — đây là thời điểm số thực sự được kích hoạt cho KH, phân biệt với `resource.sims.activation_date` có thể là ngày đại lý nhập kho | **Bước 1:** BSS phát hiện MSISDN chuyển trạng thái → ACTIVATED (ghi vào `resource.msisdn_status_history`). **Bước 2:** BSS kiểm tra COUNT bản ghi `to_state_id` = ACTIVATED của msisdn này = 1 → xác nhận lần đầu. **Bước 3:** BSS gọi API CVM với payload event `SIM_KÍCH_HOẠT` trong vòng 30–60 phút, kèm `change_date` làm mốc NGAY_0. **Bước 4:** CVM nhận event, ghi nhận NGAY_0 nội bộ. **Bước 5:** CVM render nội dung USSD theo phân khúc KH. **Bước 6:** CVM gửi USSD trong vòng 30 phút – 4 giờ sau khi nhận event. | COUNT bản ghi ACTIVATED trong `msisdn_status_history` > 1 → bỏ qua (tái kích hoạt, không phải KH mới) | `BSS → API push → CVM → USSD` | USSD |
| G3S1.E02 | Chưa cài app sau 24 giờ kích hoạt SIM | Nhắc KH cài ứng dụng Pottel nếu sau 24h kích hoạt vẫn chưa cài. Tăng tỉ lệ cài app trong ngày đầu. | Offline — Push CSV | NGAY_0 + 24h VÀ không có bản ghi trong `portal.access_app_logs` cho `user_profile_id` tương ứng trong 24h — xác nhận KH chưa mở app. Cơ chế phát hiện: CVM Scheduled Task quét `resource.msisdns` (lọc `activation_date` trong khoảng 24h trước) → join với `portal.user_profiles` lấy `user_profile_id` → kiểm tra `portal.access_app_logs` | **Bước 0 (Song song, mỗi giờ):** CVM Scheduled Task quét `resource.msisdns` tìm thuê bao có `activation_date` trong khoảng N-24h → join `portal.user_profiles` → kiểm tra `portal.access_app_logs`. Nếu không có bản ghi → đưa vào danh sách chờ push. **Bước 1:** Lúc 01:00–02:00, BSS join danh sách SIM kích hoạt ngày N với `app_install_log` (SuperApp Kafka) → lọc ra danh sách chưa cài app. BSS đối soát thêm với `portal.access_app_logs` để loại KH đã mở app qua portal. **Bước 2:** BSS tạo file CSV `no_app_install_D1_YYYYMMDD.csv` và push vào SFTP lúc 02:00–04:00. **Bước 3:** CVM đọc file lúc 04:00–08:00, xử lý danh sách. **Bước 4:** CVM gửi USSD lúc 09:00 ngày N+1. | KH đã có bản ghi trong `portal.access_app_logs` trong 24h sau kích hoạt; hoặc đã cài app (có trong `app_install_log`) trước 02:00 ngày N+1 → không có trong file CSV | `BSS → Push CSV (SFTP) → CVM → USSD` | USSD |
| G3S1.E03 | Đăng nhập app lần đầu – Màn hình chào mừng | Hiển thị banner chào mừng cá nhân hóa tức thì khi KH mở app lần đầu. Tạo ấn tượng đầu, hướng dẫn 5 bước onboarding. | NearRealtime — Push Event (API) | Lần đăng nhập app đầu tiên sau khi SIM đã kích hoạt (`first_open` chưa từng xảy ra) | **Bước 1:** KH mở app lần đầu, SuperApp ghi nhận sự kiện `first_open`. **Bước 2:** SuperApp gọi API CVM với payload `{msisdn, event: first_open, timestamp}` ngay lập tức. **Bước 3:** CVM kiểm tra msisdn chưa có bản ghi `first_open` → xác nhận lần đầu. **Bước 4:** CVM lấy nội dung đã chuẩn bị sẵn theo phân khúc từ cache. **Bước 5:** CVM trả về nội dung banner cho SuperApp hiển thị tức thì trong phiên. | Đã có bản ghi `first_open` trước đó → bỏ qua | `SuperApp → API push → CVM → Banner (từ cache)` | Banner App/Web |
| G3S1.E04 | Chưa mở app sau 24 giờ cài đặt | Nhắc KH mở app nếu đã cài nhưng chưa mở trong 24h. Chỉ gửi 1 lần duy nhất qua Push miễn phí (Firebase). | Offline — Push CSV (qua Kafka) | Có bản ghi `app_install_log` nhưng KHÔNG có bản ghi `app_open_log` sau 24h kể từ `installed_at` | **Bước 1:** SuperApp ghi sự kiện cài app và mở app vào Kafka topic `app_events`. **Bước 2:** BSS consume từ Kafka, lưu vào `app_install_log` và `app_open_log`. **Bước 3:** Lúc 01:00–02:00, BSS join 2 bảng → lọc danh sách đã cài nhưng chưa mở trong 24h. **Bước 4:** BSS tạo file CSV `app_installed_no_open_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 5:** CVM đọc file 04:00–08:00, gửi Push Notification lúc 08:00–09:00 theo giờ tối ưu của từng phân khúc. | KH đã mở app trước 02:00 ngày N+1; hoặc đã gửi trigger này 1 lần rồi | `SuperApp → Kafka → BSS → Push CSV → CVM → Push Notification (Firebase)` | Push Notification (Firebase) |
| G3S1.E05 | Chưa phát sinh cước sau 72 giờ kích hoạt SIM | Phát hiện KH có SIM nhưng chưa dùng data/thoại/SMS sau 3 ngày. Có thể do lỗi APN hoặc sóng yếu — cần hỗ trợ chủ động. | Offline — Push CSV | NGAY_0 + 72h và tổng `data_usage + voice_usage + sms_count = 0` trong 3 ngày liên tiếp | **Bước 1:** OCS tổng hợp lưu lượng hàng ngày theo từng msisdn, gửi sang BSS qua batch (OCS → BSS nightly). **Bước 2:** Lúc 01:00 ngày N+3, BSS tính tổng lưu lượng D1–D3 từ NGAY_0 → lọc danh sách có tổng = 0. **Bước 3:** BSS tạo file CSV `zero_usage_D3_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–10:00. **Bước 5:** Tooltip hiển thị khi KH mở app trong ngày N+3. | KH đã phát sinh bất kỳ lưu lượng nào (dù chỉ 1 SMS); hoặc đã gửi trigger này rồi | `OCS → BSS (nightly batch) → Push CSV → CVM → Tooltip trong app` | Banner App/Web (tooltip — chỉ hiện khi KH mở app) |
| G3S1.E06 | Cuộc gọi thất bại – hết tiền hoặc sóng yếu | Phản hồi tức thì khi cuộc gọi bị ngắt. Gợi ý nạp tiền hoặc tạm ứng ngay trên USSD. | NearRealtime — Push Event (API) | OCS ghi nhận `call_fail_reason IN ('HẾT_TÀI_KHOẢN', 'SÓNG_YẾU')` | **Bước 1:** KH thực hiện cuộc gọi, cuộc gọi bị ngắt. **Bước 2:** OCS ghi nhận sự kiện thất bại và phân loại lý do (`HẾT_TÀI_KHOẢN` hoặc `SÓNG_YẾU`). **Bước 3:** OCS gọi API CVM với payload `{msisdn, fail_reason, balance, timestamp}` ngay lập tức. **Bước 4:** CVM phân loại lý do → chọn nội dung USSD phù hợp. **Bước 5:** CVM gửi USSD trong vòng 2 phút sau khi nhận event. | Lý do thất bại không thuộc 2 loại trên; hoặc đã gửi USSD cho cùng cuộc gọi này rồi | `OCS → API push → CVM → Phân loại → USSD` | USSD |
| G3S1.E07 | Milestone + Khảo sát ngắn sau 7 ngày | Chúc mừng milestone 7 ngày. Thu thập feedback qua khảo sát 2 câu. Nếu điểm gắn kết < 7 → cảnh báo CSKH trong 1h. | Offline — Push CSV | Đúng ngày NGAY_0 + 7 HOẶC số nhiệm vụ hoàn thành ≥ 3 (điều kiện nào đến trước) | **Bước 1:** SuperApp ghi nhật ký nhiệm vụ (`task_log`) qua Kafka → BSS collect. **Bước 2:** OCS cung cấp tổng lưu lượng D1–D7 qua batch nightly → BSS lưu. **Bước 3:** Lúc 01:00 ngày N+7, BSS kiểm tra điều kiện kép (N+7 hoặc tasks ≥ 3) → lọc danh sách. **Bước 4:** BSS tạo file CSV `milestone_D7_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 5:** CVM đọc file 04:00–10:00, tính điểm gắn kết, gửi Banner khi KH mở app. **Bước 6:** Nếu điểm < 7 → CVM gửi cảnh báo nội bộ đến CSKH trong 1h. Nếu KH không mở app trong 2 ngày → chuyển sang Push. | Đã gửi milestone D7 cho KH này rồi | `SuperApp → Kafka → BSS + OCS (nightly) → Push CSV → CVM → Banner + Khảo sát` | Banner App/Web → Push (dự phòng nếu không mở app 2 ngày) |

**Dữ liệu đầu vào — G3S1:**

| Mã trigger | Trường dữ liệu cần | Nguồn | Bảng BSS | Trường BSS | Ghi chú |
|---|---|---|---|---|---|
| E01 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E01 | Thời điểm kích hoạt (NGAY_0) | 🟡 BSS | `resource.msisdn_status_history` | `change_date` (khi `to_state_id` = ACTIVATED, COUNT = 1) | `resource.sims.activation_date` có thể là ngày đại lý nhập kho — không phải lúc KH thực sự lắp SIM. Dùng `change_date` từ `msisdn_status_history` mới là mốc T=0 chính xác |
| E01 | Loại SIM | 🟡 BSS | `resource.sims` | `sim_type` | PHYSICAL / ESIM |
| E01 | Tên & loại thiết bị kích hoạt | 🔵 SuperApp | app_install_log | device_name, device_type | ❓ BSS có collect không? |
| E01 | Loại gói KH đã chọn | 🔴 OCS | — | — | Tên gói, loại gói |
| E01 | Phân khúc KH (tuổi / nghề nghiệp) | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB — bóc tách từ CCCD qua OCR eKYC) | `date_of_birth` → tính `age_segment`; `gender` tùy dữ liệu OCR. **Coverage:** chỉ KH đăng ký qua luồng eKYC (video call). KH kênh khác vẫn thiếu. Cần confirm với Dev: `ekyc_data` JSON có trường nào, format ra sao |
| E02 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E02 | Họ tên KH | 🟡 BSS | `crm.customers` | `full_name` | Bắt buộc trong CSV |
| E02 | Thời điểm kích hoạt SIM (NGAY_0) | 🟡 BSS | `resource.msisdn_status_history` | `change_date` (khi `to_state_id` = ACTIVATED, COUNT = 1) | Nhất quán với E01 — `resource.sims.activation_date` có thể là ngày đại lý nhập kho |
| E02 | Tên thiết bị kích hoạt SIM | 🔵 SuperApp | app_install_log | device_name | ❓ BSS có collect không? |
| E02 | Trạng thái cài app (chưa cài) | 🔵 SuperApp | app_install_log | — | Kiểm tra: không có bản ghi |
| E02 | Phân khúc tuổi | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB) → `segment_age` | Chỉ KH đăng ký qua eKYC |
| E02 | Phân khúc nghề nghiệp | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB) → `segment_job` | Chỉ KH đăng ký qua eKYC |
| E03 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E03 | Tên KH | 🟡 BSS | `crm.customers` | `full_name` | |
| E03 | Thời điểm đăng nhập app lần đầu | 🔵 SuperApp | — | `FIRST_OPEN` event | Từ SuperApp |
| E03 | Loại gói đang dùng | 🔴 OCS | — | `current_plan` | Để render banner cá nhân hóa |
| E03 | Dung lượng data còn lại | 🔴 OCS | — | `data_remaining` | Dung lượng data còn lại |
| E03 | Phân khúc KH | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB) | Như E01 — chỉ KH đăng ký qua eKYC |
| E04 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E04 | Thời điểm cài ứng dụng | 🔵 SuperApp | app_install_log | installed_at | |
| E04 | Loại thiết bị (Android/iOS) | 🔵 SuperApp | app_install_log | device_type, os | |
| E04 | Mã thông báo Firebase | 🔵 SuperApp | app_install_log | firebase_token | Cần để gửi Push |
| E04 | Phân khúc tuổi | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB) → `segment_age` | Để chọn giờ gửi tối ưu. Chỉ KH đăng ký qua eKYC |
| E04 | Phân khúc nghề nghiệp | ⚠️ BSS — coverage một phần | `crm.request_register_info` | `ekyc_data` (JSONB) → `segment_job` | Chỉ KH đăng ký qua eKYC |
| E05 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E05 | Lưu lượng data ngày 1 (MB) | 🔴 OCS | — | `data_usage_d1_mb` | Lưu lượng data ngày 1 |
| E05 | Lưu lượng data ngày 2 (MB) | 🔴 OCS | — | `data_usage_d2_mb` | Lưu lượng data ngày 2 |
| E05 | Lưu lượng data ngày 3 (MB) | 🔴 OCS | — | `data_usage_d3_mb` | OCS → BSS qua batch |
| E05 | Lưu lượng thoại ngày 3 (phút) | 🔴 OCS | — | `voice_usage_d3_min` | |
| E05 | Lưu lượng SMS ngày 3 | 🔴 OCS | — | `sms_count_d3` | |
| E05 | Loại thiết bị | 🔵 SuperApp | app_install_log | device_type | |
| E05 | Cài đặt APN | 🔵 SuperApp | — | apn_config | Nếu có |
| E06 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E06 | Nguyên nhân cuộc gọi thất bại | 🔴 OCS | — | `fail_reason` | `HET_TAI_KHOAN` / `SONG_YEU` (không dấu, nhất quán với contract) |
| E06 | Số dư tài khoản hiện tại | 🔴 OCS | — | `balance` | |
| E06 | Thời điểm cuộc gọi thất bại | 🔴 OCS | — | `event_timestamp` | |
| E07 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E07 | Tên KH | 🟡 BSS | `crm.customers` | `full_name` | |
| E07 | Số nhiệm vụ hướng dẫn đã hoàn thành | 🔵 SuperApp | task_log | completed_task_count | |
| E07 | Lý do kích hoạt trigger | 🟡 BSS | — | `trigger_reason` | `DAY_7` hoặc `TASK_3` — BSS tổng hợp khi tạo CSV |
| E07 | Tổng data D1–D7 (MB) | 🔴 OCS | — | `total_data_d1_d7_mb` | OCS → BSS batch |
| E07 | Tổng thoại D1–D7 (phút) | 🔴 OCS | — | `total_voice_d1_d7_min` | OCS → BSS batch |
| E07 | Điểm gắn kết ngày 7 | 🔴 OCS + 🔵 SuperApp | — | — | CVM tự tính từ nhiều nguồn |

---

### G3S2 — Giai đoạn con 2: Ngày 8–30

| Mã | Tên trigger | Mục đích | Loại | Điều kiện kích hoạt | Cơ chế & Timing | Điều kiện chặn | Luồng dữ liệu | Kênh gửi |
|---|---|---|---|---|---|---|---|---|
| G3S2.E08 | Data ngày sắp hết – Gợi ý mua thêm gói | Cảnh báo và gợi ý mua thêm gói data khi KH đã dùng ≥80% hạn mức ngày. Tránh mất kết nối giữa chừng. | NearRealtime — Push Event (API) | `daily_data_used / daily_data_quota ≥ 80%` trong ngày | **Bước 1:** OCS theo dõi lưu lượng data realtime theo từng msisdn. **Bước 2:** Khi ngưỡng 80% bị vượt, OCS gọi API CVM với payload `{msisdn, data_used, data_quota, days_remaining, timestamp}`. **Bước 3:** CVM kiểm tra điều kiện chặn (đã mua addon hôm nay chưa, đã gửi USSD hôm nay chưa). **Bước 4:** CVM tính toán gợi ý gói bổ sung phù hợp (NBO). **Bước 5:** CVM gửi USSD trong vòng 2 giờ sau khi nhận event. | KH đã mua gói bổ sung trong ngày hôm đó; hoặc đã nhận USSD từ trigger này trong ngày (tối đa 1 lần/ngày) | `OCS → API push (threshold event) → CVM → NBO → USSD / Zalo OA` | USSD → Zalo OA |
| G3S2.E09 | Hành vi xem màn hình đổi gói – chưa đăng ký | Phát hiện KH đang cân nhắc đổi gói (vào màn hình nhưng thoát không đăng ký). Hiển thị popup so sánh gói thông minh lần mở app kế tiếp. | Offline — Push CSV (qua Kafka) | SuperApp ghi `view_screen: ThayDoiGoi` và không có sự kiện `plan_register` trong 10 phút tiếp theo | **Bước 1:** KH vào màn hình đổi gói trong app. SuperApp ghi sự kiện `change_pkg_view` vào Kafka topic `app_events`. **Bước 2:** KH thoát mà không đăng ký. SuperApp ghi `exit_without_register` sau 10 phút timeout. **Bước 3:** BSS consume Kafka, lưu vào `change_pkg_view_log`. **Bước 4:** Lúc 01:00–02:00, BSS tổng hợp log trong ngày → lọc KH đủ điều kiện → tạo file CSV `change_pkg_view_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 5:** CVM đọc file 04:00–10:00, tính gợi ý gói tốt nhất dựa trên lịch sử lưu lượng 30 ngày. **Bước 6:** Popup xuất hiện lần mở app kế tiếp của KH. | KH đã đăng ký gói trong session đó; popup đã hiển thị hôm nay rồi | `SuperApp → Kafka → BSS → Push CSV → CVM → Popup trong app` | Banner App/Web (popup) → USSD (dự phòng) |
| G3S2.E13 | Tăng đột biến lưu lượng bất thường | Cảnh báo KH khi data tăng đột biến >3x mức nền — có thể do app chạy nền, cập nhật tự động, hoặc bị dùng trái phép. | Offline — Push CSV | `traffic_current_hour > 3 × baseline_same_hour` trong 1 giờ | **Bước 1:** OCS đo lưu lượng theo khung giờ cho từng msisdn, cung cấp cho BSS qua batch nightly. **Bước 2:** BSS tính mức nền (baseline) từ lịch sử CDR theo cùng khung giờ trong 30 ngày. **Bước 3:** Lúc 01:00–02:00, BSS so sánh lưu lượng hôm qua với baseline → lọc KH có traffic > 3x. **Bước 4:** BSS tạo file CSV `traffic_spike_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 5:** CVM đọc file 04:00–10:00, gửi USSD lúc 09:00–10:00. | KH đã nhận cảnh báo này trong vòng 24h; mức tăng nằm trong pattern bình thường của KH | `OCS → BSS (nightly) + CDR (baseline) → Push CSV → CVM → USSD / Zalo OA` | USSD → Zalo OA |
| G3S2.E11 | Ngày 30 – Tổng kết 1 tháng + chuyển sang G4 | Tổng kết hành trình 30 ngày đầu. Tính điểm gắn kết để phân nhánh: KH tích cực → G4 (upsell), KH ít tương tác → chiến dịch giữ chân. | Offline — Push CSV | Đúng ngày NGAY_0 + 30 | **Bước 1:** OCS cung cấp tổng lưu lượng N1–N30, số lần nạp tiền, số lần đổi gói qua batch nightly. **Bước 2:** BSS tổng hợp toàn bộ dữ liệu 30 ngày: lưu lượng, hành vi app, lịch sử giao dịch. **Bước 3:** Lúc 01:00 ngày N+30, BSS lọc danh sách KH đến đúng ngày → tạo file CSV `ngay_30_summary_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–08:00, tính điểm gắn kết, phân nhánh G4 hoặc giữ chân. **Bước 5:** CVM gửi Banner + Email lúc 08:00–09:00. Nếu không mở app → Push. | Đã gửi tổng kết D30 cho KH này rồi | `OCS (nightly) → BSS → Push CSV → CVM → Điểm gắn kết → Phân nhánh → App + Email` | Banner App/Web (toàn màn hình) + Email → Push (dự phòng) |

**Dữ liệu đầu vào — G3S2:**

| Mã trigger | Trường dữ liệu cần | Nguồn | Bảng BSS | Trường BSS | Ghi chú |
|---|---|---|---|---|---|
| E08 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E08 | Tên & loại gói đang dùng | 🔴 OCS | — | plan_name, plan_type | |
| E08 | Hạn mức data ngày (quota) | 🔴 OCS | — | daily_data_quota | |
| E08 | Data đã dùng trong ngày | 🔴 OCS | — | daily_data_used | |
| E08 | % data ngày đã dùng | 🔴 OCS | — | Tính: used/quota | |
| E08 | Số ngày còn lại trong chu kỳ gói | 🔴 OCS | — | days_remaining | |
| E08 | Lịch sử mua gói bổ sung hôm nay | 🔴 OCS | — | `addon_purchased_today` | Để tránh gửi trùng |
| E08 | Đề xuất gói bổ sung phù hợp | 🔴 OCS + CVM | — | — | CVM tính NBO |
| E09 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E09 | Gói đang dùng hiện tại | 🔴 OCS | — | `current_plan` | |
| E09 | Thời điểm vào màn hình | 🔵 SuperApp | `change_pkg_view_log` | `view_timestamp` | Thời điểm vào màn hình |
| E09 | Số lần xem màn hình đổi gói | 🔵 SuperApp | `change_pkg_view_log` | `view_count_today` | |
| E09 | Thời gian trên màn hình đổi gói (giây) | 🔵 SuperApp | `change_pkg_view_log` | `time_on_screen_sec` | |
| E09 | Tổng data 30 ngày (MB) | 🔴 OCS | — | `total_data_30d_mb` | Để gợi ý gói phù hợp |
| E13 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E13 | Lưu lượng giờ đột biến (MB) | 🔴 OCS | — | `traffic_spike_mb` | |
| E13 | Thời điểm phát hiện tăng đột biến | 🔴 OCS | — | `spike_timestamp` | |
| E13 | Giờ xảy ra (0–23) | 🔴 OCS | — | `spike_hour` | Giờ xảy ra (0–23) |
| E13 | Tỉ lệ tăng (spike/baseline) | 🔴 OCS | — | `spike_ratio` | Tỉ lệ tăng (spike/baseline) |
| E13 | Mức nền cùng khung giờ (MB) | 🔴 OCS / CDR | `cdr.*` | `baseline_mb` | Mức nền cùng khung giờ (tính từ lịch sử CDR 30 ngày) |
| E13 | Danh sách ứng dụng đang kết nối | 🔵 SuperApp | — | active_apps | Nếu có |
| E13 | Loại thiết bị | 🔵 SuperApp | app_install_log | device_type | |
| E11 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| E11 | Tên KH | 🟡 BSS | `crm.customers` | `full_name` | |
| E11 | Email KH | 🟡 BSS | `crm.customers` | `contact_email` | Để gửi Email tổng kết |
| E11 | Tổng lưu lượng data N1–N30 | 🔴 OCS | — | total_data_gb | OCS → BSS batch |
| E11 | Tổng lưu lượng thoại N1–N30 | 🔴 OCS | — | total_voice_min | |
| E11 | Số lần nạp tiền | 🔴 OCS | — | topup_count | |
| E11 | Số lần đổi gói | 🔴 OCS | — | plan_change_count | |
| E11 | Điểm gắn kết ngày 30 | 🔴 OCS + 🔵 SuperApp | — | — | CVM tự tính |

---

## G4 — MỞ RỘNG & TĂNG TRƯỞNG (Tháng 2+)

| Mã | Tên trigger | Mục đích | Loại | Điều kiện kích hoạt | Cơ chế & Timing | Điều kiện chặn | Luồng dữ liệu | Kênh gửi |
|---|---|---|---|---|---|---|---|---|
| G4.U01 | Nạp tiền thành công (lần ≥ 2) | Gắn gợi ý đăng ký gói cước ngay sau khi KH nạp tiền lần 2+. Tận dụng thời điểm KH vừa có số dư để upsell gói tháng. | NearRealtime — Push Event (API) | Giao dịch nạp tiền thành công và `topup_count ≥ 2` | **Bước 1:** KH nạp tiền thành công qua bất kỳ kênh nào. **Bước 2:** OCS ghi nhận giao dịch, đếm `topup_count`. **Bước 3:** Nếu `topup_count ≥ 2`, OCS gọi API CVM với payload `{msisdn, topup_amount, balance_after, topup_count, current_plan}`. **Bước 4:** CVM chạy NBO model → chọn gợi ý gói phù hợp nhất. **Bước 5:** CVM gửi USSD trong vòng 3 phút sau khi nhận event. | `topup_count = 1` (lần nạp đầu tiên); KH đang có gói tháng hiệu lực phù hợp | `OCS → API push (topup_success) → CVM → NBO → USSD` | USSD |
| G4.U02 | Đăng ký / gia hạn gói cước thành công (lần ≥ 2) | Xác nhận đăng ký gói thành công và gợi ý dịch vụ bổ sung (gói gia đình, SIM thứ 2) sau 24h. Cross-sell cho KH trung thành. | Offline — Push CSV | Đăng ký hoặc gia hạn gói thành công và `plan_register_count ≥ 2` | **Bước 1:** OCS ghi nhận giao dịch đăng ký/gia hạn gói thành công. **Bước 2:** OCS cung cấp log giao dịch cho BSS qua batch nightly. **Bước 3:** Lúc 01:00–02:00, BSS lọc giao dịch `plan_register_count ≥ 2` trong ngày → tạo file CSV `plan_register_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–10:00. **Bước 5:** CVM gửi USSD xác nhận lúc 09:00; gửi Zalo OA gợi ý bổ sung lúc 10:00–11:00. | `plan_register_count = 1`; KH đã nhận gợi ý cross-sell trong 30 ngày gần đây | `OCS (nightly) → BSS → Push CSV → CVM → USSD + Zalo OA` | USSD → Zalo OA |
| G4.U03 | Tra cứu số dư – gắn gợi ý thông minh | Bổ sung gợi ý gia hạn gói ngay inline trong kết quả tra cứu *101#. Không gửi thêm tin — gắn luôn vào màn hình USSD KH đang xem. | NearRealtime — Push Event (API) | KH nhắn `*101#` để tra cứu số dư | **Bước 1:** KH nhắn `*101#`. OCS nhận yêu cầu tra cứu. **Bước 2:** OCS gọi API CVM đồng thời với việc xử lý tra cứu, payload `{msisdn, balance, current_plan, expiry_date, data_used_pct}`. **Bước 3:** CVM lấy gợi ý gói tốt nhất từ cache (đã tính sẵn), phản hồi trong 2 giây. **Bước 4:** OCS nhận gợi ý từ CVM, gắn inline vào kết quả USSD trước khi trả về màn hình KH. | CVM không phản hồi trong 2 giây → OCS trả kết quả tra cứu bình thường, không kèm gợi ý | `OCS → API push (balance_check) → CVM → Gợi ý (cache) → OCS → Inline USSD` | USSD (inline trong kết quả tra cứu) |
| G4.U04 | Nhận OTP từ app bên thứ 3 – gợi ý gói data | Phát hiện KH đang dùng app ngân hàng/giải trí/mạng xã hội (qua OTP nhận được). Gợi ý gói data phù hợp với hành vi thực tế. | Offline — Push CSV (từ HLR) | KH nhận SMS chứa từ khóa OTP/mã xác thực từ đầu số thương mại đã phân loại | **Bước 1:** KH nhận SMS OTP từ app bên thứ 3 (ngân hàng, giải trí, mạng xã hội). **Bước 2:** HLR (Home Location Register — Hệ thống Đăng ký vị trí thuê bao) phát hiện SMS chứa từ khóa OTP từ đầu số thương mại, ghi nhận vào log. **Bước 3:** HLR đẩy log qua Kafka. **Bước 4:** Lúc 02:00–04:00, HLR export file CSV `otp_detection_YYYYMMDD.csv` trực tiếp vào SFTP CVM (không đi qua BSS). **Bước 5:** CVM đọc file 04:00–10:00, đối chiếu đầu số với danh sách phân loại app → phân loại loại app → chọn gợi ý gói phù hợp. **Bước 6:** CVM gửi USSD lúc 09:00–10:00. **Giới hạn:** 1 lần/ngày. | KH đã nhận USSD từ trigger này trong ngày; KH đang có gói data phù hợp rồi | `HLR → Kafka → CSV (otp_detection) → CVM → Phân loại app → USSD` | USSD |
| G4.U05 | Hết data sớm liên tục 2 tháng → Đề xuất nâng gói | Phát hiện xu hướng KH liên tục hết data trước cuối tháng trong 2 kỳ liên tiếp. Đề xuất nâng gói phù hợp nhu cầu thực tế. | Offline — Push CSV | `data_depleted_before_day_25 = True` trong 2 chu kỳ thanh toán liên tiếp | **Bước 1:** OCS ghi nhận ngày hết data mỗi chu kỳ cho từng msisdn, cung cấp cho BSS qua batch nightly. **Bước 2:** Đầu tháng thứ 3, BSS tổng hợp lịch sử 2 tháng → kiểm tra điều kiện hết data trước ngày 25 trong cả 2 kỳ. **Bước 3:** BSS tạo file CSV `billing_2month_YYYYMM.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–10:00, chạy ARPU model, thiết lập A/B test. **Bước 5:** CVM gửi Zalo OA + Banner lúc 09:00–11:00. | Trigger này đã kích hoạt cho KH trong 3 tháng gần đây; KH đã tự nâng gói rồi | `OCS (nightly) → BSS → Push CSV → CVM → ARPU model + A/B Test → Zalo OA + App` | Zalo OA → Banner App/Web |
| G4.U06 | Chuyển đổi loại gói (thoại ↔ data) thành công | Xác nhận chuyển gói thành công và gợi ý dịch vụ bổ sung phù hợp với loại gói mới sau 24h. | Offline — Push CSV | Giao dịch chuyển gói thành công và `loại gói cũ ≠ loại gói mới` | **Bước 1:** KH đăng ký gói mới khác loại với gói cũ. OCS ghi nhận giao dịch. **Bước 2:** OCS cung cấp log `plan_change` cho BSS qua batch nightly. **Bước 3:** Lúc 01:00–02:00, BSS lọc giao dịch đổi loại gói trong ngày → tạo file CSV `plan_change_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–10:00, so sánh gói cũ/mới, chọn gợi ý bổ sung phù hợp. **Bước 5:** CVM gửi USSD xác nhận lúc 09:00; gửi Zalo OA gợi ý lúc 10:00–12:00. | Chuyển gói cùng loại (VD: data → data); KH đã nhận gợi ý bổ sung trong 7 ngày gần đây | `OCS (nightly) → BSS → Push CSV → CVM → So sánh gói → USSD + Zalo OA` | USSD → Zalo OA |
| G4.U07 | Chuyển đổi SIM nội mạng (SIM thường → SIM số đẹp / eSIM) | Xác nhận đổi SIM thành công, hướng dẫn kích hoạt SIM mới và thông báo số cũ hết hiệu lực. | Offline — Push CSV | Giao dịch đổi SIM nội mạng thành công và `loại SIM cũ ≠ loại SIM mới` | **Bước 1:** KH yêu cầu đổi SIM tại cửa hàng hoặc qua app. BSS xử lý giao dịch `sim_swap`. **Bước 2:** BSS ghi nhận sự kiện đổi SIM thành công vào `sim_swap_log`. **Bước 3:** Lúc 01:00–02:00, BSS lọc giao dịch `sim_swap` trong ngày → tạo file CSV `sim_swap_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–10:00. **Bước 5:** CVM gửi USSD + SMS hướng dẫn kích hoạt lúc 09:00–10:00. Nếu KH có app → gửi thêm Push. | Đổi SIM cùng loại; sự kiện đổi SIM đã được xử lý rồi | `BSS (sim_swap event) → Push CSV → CVM → USSD + SMS + Push` | USSD + SMS → Push (nếu có app) |
| G4.U08 | Gia hạn gói đúng hạn 3 tháng liên tiếp – Khóa trung thành | Tôn vinh KH trung thành sau 3 tháng gia hạn đúng hạn liên tiếp. Nâng hạng và trao quyền lợi mới. | Offline — Push CSV | `số lần gia hạn đúng hạn liên tiếp ≥ 3` (gia hạn trước ngày hết hạn hoặc trong ngày hết hạn) | **Bước 1:** OCS ghi nhận giao dịch gia hạn gói. BSS nhận qua batch nightly, lưu lịch sử gia hạn. **Bước 2:** Sau mỗi lần gia hạn, BSS tính số lần gia hạn đúng hạn liên tiếp (không bị gián đoạn dịch vụ). **Bước 3:** Khi đủ điều kiện ≥ 3 lần, BSS tạo file CSV `renewal_loyalty_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 4:** CVM đọc file 04:00–08:00, tính điểm trung thành, xác định hạng mới. **Bước 5:** CVM gửi Banner + Email lúc 08:00–09:00. | Đã tặng thưởng trung thành trong 3 tháng gần đây; KH đã ở hạng cao nhất rồi | `OCS (nightly) → BSS → Push CSV → CVM → Điểm trung thành → App + Email` | Banner App/Web + Email |
| G4.U09 | Sinh nhật & Ngày kỷ niệm KH / SIM | Gửi lời chúc và tặng quà data đúng ngày sinh nhật KH hoặc ngày kỷ niệm mua SIM. Tăng gắn kết cảm xúc. | Offline — Push CSV | Ngày hiện tại = ngày sinh nhật KH HOẶC ngày kỷ niệm = N năm kể từ `activation_date` đầu tiên | **Bước 1:** Mỗi ngày lúc 01:00, BSS quét `crm.customers` theo `birthday` và `resource.sims` theo `activation_date` → lọc danh sách KH có ngày đặc biệt hôm nay. **Bước 2:** BSS tạo file CSV `birthday_list_YYYYMMDD.csv` push vào SFTP lúc 02:00–04:00. **Bước 3:** CVM đọc file 04:00–08:00, soạn nội dung cá nhân hóa theo tên và gói đang dùng. **Bước 4:** CVM gửi Zalo OA lúc 08:00–09:00. Nếu KH chưa có Zalo → SMS dự phòng. | Đã gửi lời chúc sinh nhật năm nay rồi; KH không có trạng thái ACTIVE | `BSS → Push CSV → CVM → Nội dung cá nhân hóa → Zalo OA / SMS` | Zalo OA → SMS (dự phòng) |
| G4.U10 | Các ngày lễ & sự kiện quốc gia | Gửi lời chúc và ưu đãi vào các ngày lễ lớn. Nội dung cá nhân hóa theo giới tính (8/3, 20/10) và nghề nghiệp (ngày Nhà giáo, Thầy thuốc). Chuẩn bị trước 2 ngày. | Offline — Push CSV | Ngày hiện tại nằm trong lịch ngày lễ cố định đã cấu hình trong CVM | **Bước 1:** CVM có lịch cố định (Tết, 30/4, 1/5, 2/9, 8/3, 20/10...). Trước mỗi ngày lễ 2 ngày, CVM gửi yêu cầu lấy danh sách KH cho BSS. **Bước 2:** BSS export file CSV `subscriber_demographic_YYYYMMDD.csv` (toàn bộ KH ACTIVE + thông tin nhân khẩu nếu có) push vào SFTP lúc 02:00–04:00 của ngày chuẩn bị. **Bước 3:** CVM đọc file 04:00–08:00, phân khúc theo dịp lễ (giới tính cho 8/3, 20/10; nghề nghiệp cho ngày Nhà giáo...). **Bước 4:** CVM soạn nội dung theo phân khúc, lên lịch gửi. **Bước 5:** Gửi Zalo OA lúc 08:00 đúng ngày lễ. | KH không có trạng thái ACTIVE; ngày lễ không áp dụng cho phân khúc của KH (VD: 8/3 chỉ gửi cho nữ) | `BSS → Push CSV (chuẩn bị trước 2 ngày) → CVM → Phân khúc → Zalo OA / SMS` | Zalo OA → SMS (dự phòng) |

**Dữ liệu đầu vào — G4:**

| Mã trigger | Trường dữ liệu cần | Nguồn | Bảng BSS | Trường BSS | Ghi chú |
|---|---|---|---|---|---|
| U01 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U01 | Số tiền nạp | 🔴 OCS | — | topup_amount | |
| U01 | Số dư sau nạp | 🔴 OCS | — | balance_after | |
| U01 | Lần nạp thứ mấy | 🔴 OCS | — | topup_count | |
| U01 | Gói hiện tại | 🔴 OCS | — | current_plan | |
| U01 | Đề xuất gói phù hợp (NBO) | 🔴 OCS + CVM | — | — | CVM tính toán |
| U02 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U02 | Tên gói đã đăng ký | 🔴 OCS | — | `plan_name` | |
| U02 | Loại gói | 🔴 OCS | — | `plan_type` | `DATA` / `VOICE` / `COMBO` |
| U02 | Ngày hết hạn gói | 🟡 BSS | `resource.msisdns` | `plan_expiry_date` | |
| U02 | Thời điểm đăng ký | 🔴 OCS | — | `register_timestamp` | Thời điểm đăng ký |
| U02 | Lần đăng ký thứ mấy | 🔴 OCS | — | `plan_register_count` | |
| U02 | Gợi ý dịch vụ bổ sung | 🔴 OCS + CVM | — | — | CVM tính toán |
| U03 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U03 | Số dư tài khoản | 🔴 OCS | — | balance | |
| U03 | Tên gói hiện tại | 🔴 OCS | — | current_plan | |
| U03 | Ngày hết hạn gói | 🟡 BSS | `resource.msisdns` | `expiry_date` | |
| U03 | % data đã dùng | 🔴 OCS | — | data_used_pct | |
| U03 | Đề xuất gói tốt nhất | 🔴 OCS + CVM | — | — | CVM tính toán |
| U04 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U04 | Đầu số gửi OTP | ⚪ HLR | `otp_detection_log` | `sender_number` | Đầu số gửi OTP |
| U04 | Phân loại ứng dụng phát OTP | ⚪ HLR | `otp_detection_log` | `app_category` | Phân loại từ đầu số gửi SMS — HLR export CSV trực tiếp, không qua BSS |
| U04 | Tên ứng dụng cụ thể | ⚪ HLR | `otp_detection_log` | `app_name` | Tên ứng dụng cụ thể (optional) |
| U04 | Số lần nhận OTP trong ngày | ⚪ HLR | `otp_detection_log` | `otp_count_today` | |
| U04 | Gói data hiện tại | 🔴 OCS | — | `current_plan` | |
| U04 | Đề xuất gói phù hợp | 🔴 OCS + CVM | — | — | Dựa trên loại app |
| U05 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U05 | Ngày hết data tháng 1 | 🔴 OCS | — | `month1_depleted_date` | |
| U05 | Ngày hết data tháng 2 | 🔴 OCS | — | `month2_depleted_date` | |
| U05 | Tổng data dùng tháng 1 (GB) | 🔴 OCS | — | `month1_total_data_gb` | |
| U05 | Tổng data dùng tháng 2 (GB) | 🔴 OCS | — | `month2_total_data_gb` | |
| U05 | Gói hiện tại | 🔴 OCS | — | `current_plan` | |
| U05 | Gợi ý gói nâng lên | 🔴 OCS + CVM | — | `suggested_plan` | Gợi ý gói nâng lên (optional) |
| U05 | Đề xuất gói nâng lên & chênh lệch giá | 🔴 OCS + CVM | — | — | CVM tính toán |
| U06 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U06 | Gói cũ | 🔴 OCS | — | `old_plan` | |
| U06 | Gói mới | 🔴 OCS | — | `new_plan` | |
| U06 | Chênh lệch data (GB) | 🔴 OCS | — | `data_diff_gb` | Chênh lệch data (GB) |
| U06 | Chênh lệch thoại (phút) | 🔴 OCS | — | `voice_diff_min` | Chênh lệch thoại (phút) |
| U06 | Ngày hết hạn gói mới | 🟡 BSS | `resource.msisdns` | `expiry_date` | |
| U06 | Đề xuất dịch vụ bổ sung | 🔴 OCS + CVM | — | — | |
| U07 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U07 | Loại SIM cũ | 🟡 BSS | `resource.sims` | `sim_type` (bản ghi cũ) | PHYSICAL / ESIM |
| U07 | Loại SIM mới | 🟡 BSS | `resource.sims` | `sim_type` (bản ghi mới) | |
| U07 | Số SIM / MSISDN mới | 🟡 BSS | `resource.sims` + `resource.msisdns` | `new_iccid`, `msisdn` | |
| U07 | Thời điểm đổi SIM | 🟡 BSS | `resource.sims` | `activation_date` | |
| U07 | Gói cước được giữ nguyên | 🔴 OCS | — | current_plan | |
| U08 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U08 | Họ tên KH | 🟡 BSS | `crm.customers` | `full_name` | Bắt buộc trong CSV |
| U08 | Email KH | 🟡 BSS | `crm.customers` | `contact_email` | Để gửi Email |
| U08 | Số lần gia hạn đúng hạn liên tiếp | 🟡 BSS + 🔴 OCS | `resource.msisdns` | `expiry_date` (lịch sử) | CVM tính từ lịch sử |
| U08 | Ngày gia hạn lần cuối | 🟡 BSS | — | `last_renewal_date` | Ngày gia hạn lần cuối |
| U08 | Điểm trung thành | ❓ Chưa rõ | — | — | Chưa có schema BSS |
| U08 | Hạng hiện tại & quyền lợi hạng mới | ❓ Chưa rõ | — | — | Chưa có schema BSS |
| U09 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U09 | Tên KH | 🟡 BSS | `crm.customers` | `full_name` | |
| U09 | Ngày sinh nhật | ❓ Chưa rõ | `crm.customers`? | `birthday`? | Không thấy trong schema hiện tại |
| U09 | Ngày kỷ niệm mua SIM | 🟡 BSS | `resource.sims` | `activation_date` | Ngày kích hoạt đầu tiên |
| U09 | Gói đang dùng | 🔴 OCS | — | `current_plan` | |
| U09 | Zalo OA ID | ❓ Chưa rõ | `crm.customers`? | `zalo_oa_id`? | Cần cho kênh Zalo OA |
| U09 | Điểm trung thành hiện tại | ❓ Chưa rõ | — | — | Chưa có schema |
| U10 | Số điện thoại | 🟡 BSS | `crm.subscribers` | `msisdn` | |
| U10 | Tên KH | 🟡 BSS | `crm.customers` | `full_name` | |
| U10 | Giới tính | ❓ Chưa rõ | `crm.customers`? | `gender`? | Cần cho 8/3, 20/10 — không thấy trong schema |
| U10 | Phân khúc nghề nghiệp | ❓ Chưa rõ | — | `job_segment`? | Cần cho ngày Nhà giáo, Thầy thuốc — không có trong BSS |
| U10 | Phân khúc tuổi | ❓ Chưa rõ | — | `age_segment`? | Không có trong BSS |
| U10 | Gói đang dùng | 🔴 OCS | — | `current_plan` | |
| U10 | Trạng thái thuê bao | 🟡 BSS | `crm.subscribers` | `status` | Chỉ gửi cho KH ACTIVE |

---

## Tổng hợp khoảng trống dữ liệu cần clarify

| Dữ liệu thiếu | Trigger bị ảnh hưởng | Nguồn kỳ vọng | Mức độ |
|---|---|---|---|
| Ngày sinh / tuổi KH | E01, E02, E04, E07, U09, U10 | Hồ sơ đăng ký KH — BSS chưa có trường này | 🔴 Critical |
| Giới tính KH | U10 (8/3, 20/10) | Hồ sơ đăng ký KH — BSS chưa có | 🟡 Important |
| Nghề nghiệp KH | E01, E02, E03, E04, U10 | Hồ sơ đăng ký KH — BSS chưa có | 🟡 Important |
| Điểm trung thành / hạng KH | U08, U09 | Module loyalty — chưa có schema nào trong BSS | 🟡 Important |
| App install log (device, OS, Firebase token) | E02, E04 | SuperApp → Kafka → BSS — cần xác nhận BSS có consume và lưu không | 🔴 Critical |
| App open log | E04 | SuperApp → Kafka → BSS | 🔴 Critical |
| OCS: dữ liệu sử dụng realtime (data, thoại, số dư, gói) | E03, E05, E06, E08, U01, U02, U03, U05, U06 | OCS — cần API push (NearRealtime) và batch nightly (Offline) | 🔴 Critical |
| SuperApp events (first_open, change_pkg_view) | E03, E09 | SuperApp → API push hoặc Kafka — chưa rõ cơ chế tích hợp | 🔴 Critical |
| HLR: OTP detection log | U04 | HLR export CSV trực tiếp vào CVM qua Kafka — không đi qua BSS | ✅ Đã rõ |
