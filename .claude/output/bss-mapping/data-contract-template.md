# Data Contract Template — BSS → CVM

> Tài liệu đặc tả schema các file CSV và API event mà BSS/OCS/SuperApp cung cấp cho CVM.
> Mục đích: làm tài liệu thống nhất giữa BSS team và CVM team trước khi implement.
>
> Cập nhật: 2026-06-03 — Tái cấu trúc toàn bộ theo 4 nhóm vòng đời (Kích hoạt / Sử dụng / Gia hạn / Khóa); bổ sung 14 trigger mới; chốt E_ZERO_BALANCE giữ lại (ranh giới rõ với E06)
> Trạng thái: **DRAFT — cần xác nhận với Tech Lead / SA**
>
> **⚠️ Cập nhật 2026-07-03 — Bản chuẩn mới nhất là `data-contract-template-lifecycle.md`.**
> File này được đồng bộ các thay đổi cốt lõi (thêm trigger `E_CHURN_RISK`, rule E08 90%/lần 3). Các thay đổi chi tiết theo bảng trigger đầy đủ — đổi timing sang NearRealtime (E02, E04, E05, E09, U_PRE_EXPIRY, U_POST_EXPIRY, E_LOCK_1C, E_PRE_LOCK_2C), thêm trigger **`U05-B-RT`** (NearRealtime, riêng biệt với `U05-B` batch — KHÔNG gộp chung schema) khi KH hết quota ngày/tuần 3 lần liên tiếp, bổ sung trường KH/kênh/chu kỳ gói (U02 `plan_cycle`, U04/U06 chỉ gói tháng, U07 4 case + đổi phôi, U09 sinh nhật nhà mạng, E_USAGE_NEED_ANALYSIS chốt 2 tháng, E_NO_PLAN_X_DAYS mốc x=10, chuẩn hóa phân khúc HSSV/tài xế/CBCNVVP/hưu trí) — **được đặc tả đầy đủ trong `data-contract-template-lifecycle.md`**. Khi có khác biệt giữa 2 file, lấy bản `-lifecycle.md` làm chuẩn.
> Nhóm Open Question chờ tài liệu VTDĐ (trường OB, nguyên tắc gói gợi ý, nguyên tắc nâng gói): xem Phần Open Questions của bản `-lifecycle.md`.

**Quy ước chung:**
- Encoding: UTF-8
- Delimiter CSV: dấu phẩy (`,`)
- Header row: có (dòng đầu tiên là tên cột)
- Datetime format: `YYYY-MM-DD HH:MM:SS` (UTC+7)
- Tên file: `{tên_file}_{YYYYMMDD}.csv`
- Cơ chế delivery: BSS push vào SFTP/thư mục CVM đọc
- Giá trị rỗng: để trống (không dùng NULL hay 0 thay thế)

---

---

## NHÓM 1 — Kích hoạt

### 1.1 NearRealtime

> **Lưu ý chung NearRealtime:** Endpoint cụ thể (`POST /api/...`) sẽ do Dev định nghĩa khi implement. Tài liệu này chỉ đặc tả **payload (các trường dữ liệu cần thiết)** và **yêu cầu kỹ thuật** để Dev và BSS/OCS team thống nhất.

#### Event: SIM_KÍCH_HOẠT (E01)

**Trigger bởi:** BSS
**Yêu cầu kỹ thuật:** BSS gọi API CVM ngay khi SIM chuyển trạng thái ACTIVE lần đầu. Phải đến CVM trong vòng 30–60 phút sau sự kiện.
**Timing:** Trong vòng 30–60 phút sau khi SIM chuyển trạng thái ACTIVE

> **Lưu ý nguồn NGAY_0:** Trường `activation_date` phải lấy từ `resource.msisdn_status_history.change_date` (khi `to_state_id` = ACTIVATED, COUNT = 1) — không dùng `resource.sims.activation_date` vì đây có thể là ngày đại lý quét mã nhập kho, chưa phải lúc KH thực sự lắp SIM.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | CVM nội bộ — không lấy từ nguồn ngoài | `SIM_KICH_HOAT` |
| `msisdn` | string(15) | ✅ | Số điện thoại thuê bao | `crm.subscribers.msisdn` | `0901234567` |
| `iccid` | string(20) | ✅ | Mã ICCID của SIM | `resource.sims.iccid` | `8984012345678901234` |
| `sim_type` | string | ✅ | Loại SIM | `resource.sims.sim_type` | `PHYSICAL` hoặc `ESIM` |
| `activation_date` | datetime | ✅ | Mốc NGAY_0 — lấy từ `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `resource.msisdn_status_history.change_date` (khi `to_state_id` = ACTIVATED, COUNT = 1) | `2026-05-14 08:30:00` |
| `lifecycle_number` | integer | ✅ | Số lần kích hoạt (phải = 1) | COUNT bản ghi ACTIVATED trong `resource.msisdn_status_history` theo `msisdn` | `1` |
| `customer_id` | string | ✅ | Mã khách hàng trong BSS | `crm.subscribers.customer_id` | `CUS-00012345` |
| `full_name` | string(64) | ✅ | Họ tên khách hàng | `crm.customers.full_name` | `Nguyễn Văn A` |
| `current_plan` | string | ❌ | Tên gói đang dùng (từ OCS) | OCS — không có bảng BSS | `GOI_DATA_70K` |
| `device_name` | string | ❌ | Tên thiết bị kích hoạt | `app_install_log.device_name` (SuperApp → Kafka → BSS) | `Samsung Galaxy A55` |
| `device_type` | string | ❌ | Loại thiết bị | `app_install_log.device_type` (SuperApp → Kafka → BSS) | `ANDROID` hoặc `IOS` |
| `segment_age` | string | ❌ | Phân khúc tuổi — bóc tách từ `ekyc_data.date_of_birth` trong `crm.request_register_info` hoặc `esim_agency.individual_info`; chỉ có với KH đăng ký qua luồng eKYC | `crm.request_register_info.ekyc_data` → `date_of_birth` hoặc `esim_agency.individual_info` | `19-24` |
| `segment_job` | string | ❌ | Phân khúc nghề nghiệp — nguồn tương tự `segment_age`; chỉ có với KH đăng ký qua luồng eKYC | `crm.request_register_info.ekyc_data` → nghề nghiệp hoặc `esim_agency.individual_info` | `STUDENT` |
| `nationality` | string | ❌ | Quốc tịch KH — để CVM chọn ngôn ngữ gửi tin chào mừng phù hợp | `crm.request_register_info.ekyc_data` → loại giấy tờ (CCCD = VN, hộ chiếu = nước ngoài); hoặc `esim_agency.individual_info` — cần xác nhận BSS có trường này không | `VN` hoặc `US` |
| `accept_language` | string | ❌ | Ngôn ngữ ưu tiên của KH trong app — để CVM gửi tin đúng ngôn ngữ khi `nationality` không đủ | SuperApp — ngôn ngữ KH cài đặt trong ứng dụng | `vi` hoặc `en` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên khách hàng để cá nhân hóa lời chào | Văn bản | `crm.customers.full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao vừa kích hoạt | Văn bản | `crm.subscribers.msisdn` | — | `0901234567` |
| `{{loai_sim}}` | ✅ | Loại SIM KH đang dùng (SIM vật lý hay eSIM) | Văn bản | `resource.sims.sim_type` | — | `ESIM` |
| `{{ten_goi}}` | ❌ | Tên gói cước đang active để gợi ý nâng gói | Văn bản | OCS — `current_plan` (optional trong payload) | `"gói đang dùng"` | `GOI_DATA_70K` |
| `{{ten_thiet_bi}}` | ❌ | Tên thiết bị KH đang dùng để cá nhân hóa nội dung | Văn bản | SuperApp → `app_install_log.device_name` | bỏ qua câu đề cập thiết bị | `Samsung Galaxy A55` |
| `{{phan_khuc_tuoi}}` | ❌ | Phân khúc độ tuổi để gửi nội dung phù hợp nhóm | Văn bản | `ekyc_data` → `segment_age` | không hiển thị nội dung theo phân khúc | `19-24` |
| `{{quoc_tich}}` | ❌ | Quốc tịch KH để CVM chọn ngôn ngữ gửi tin | Văn bản | `ekyc_data` → `nationality` | `"VN"` (mặc định tiếng Việt) | `VN` |

---

---

### 1.2 Batch/CSV

#### File: `no_app_install_D1_{YYYYMMDD}.csv` (E02)

**Mô tả:** Danh sách KH kích hoạt SIM ngày N-1 nhưng chưa cài app sau 24h
**Trigger bởi:** BSS
**Thời điểm push:** 02:00–04:00 hàng ngày
**CVM đọc:** 04:00–08:00, gửi USSD lúc 09:00

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `activation_date` | datetime | ✅ | Thời điểm kích hoạt SIM (NGAY_0) | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-05-13 08:30:00` |
| `sim_type` | string | ✅ | Loại SIM | `resource.sims.sim_type` | `PHYSICAL` |
| `device_name` | string | ❌ | Tên thiết bị kích hoạt | `app_install_log.device_name` (SuperApp → Kafka → BSS) | `Samsung Galaxy A55` |
| `segment_age` | string | ❌ | Phân khúc tuổi | `crm.request_register_info.ekyc_data` → `date_of_birth` hoặc `esim_agency.individual_info` | `19-24` |
| `segment_job` | string | ❌ | Phân khúc nghề nghiệp | `crm.request_register_info.ekyc_data` → nghề nghiệp hoặc `esim_agency.individual_info` | `DRIVER` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn nhắc cài app | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao chưa cài app sau D+1 | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_thiet_bi}}` | ❌ | Tên thiết bị KH dùng để cá nhân hóa hướng dẫn cài đặt | Văn bản | CSV `device_name` (optional) | bỏ qua câu đề cập thiết bị | `Samsung Galaxy A55` |
| `{{link_cai_app}}` | ✅ | Đường dẫn tải app tương ứng với hệ điều hành KH | Văn bản | Cấu hình tĩnh CVM (App Store / Google Play) | — | `https://play.google.com/...` |

---

---

## NHÓM 2 — Sử dụng

### 2.1 Viễn thông — NearRealtime

> **Lưu ý chung NearRealtime:** Endpoint cụ thể (`POST /api/...`) sẽ do Dev định nghĩa khi implement. Tài liệu này chỉ đặc tả **payload (các trường dữ liệu cần thiết)** và **yêu cầu kỹ thuật** để Dev và BSS/OCS team thống nhất.

#### Event: CUỘC_GỌI_THẤT_BẠI (E06)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay sau khi cuộc gọi bị ngắt. CVM phải gửi USSD trong vòng 2 phút để KH thấy ngay trên màn hình.
**Timing:** Trong vòng vài giây sau khi cuộc gọi bị ngắt

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `CALL_FAIL` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm thất bại | OCS — thời điểm cuộc gọi thất bại | `2026-05-14 16:05:00` |
| `fail_reason` | string | ✅ | Lý do thất bại | OCS — phân loại lý do thất bại | `HET_TAI_KHOAN` hoặc `SONG_YEU` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS — số dư tài khoản realtime | `3500` |

**Param template:**

> Template chia 2 nhánh theo `fail_reason`: `HET_TAI_KHOAN` và `SONG_YEU`.

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao bị thất bại cuộc gọi | Văn bản | Payload E06 `msisdn` | — | `0901234567` |
| `{{ly_do_that_bai}}` | ✅ | Lý do cuộc gọi thất bại để CVM phân nhánh nội dung phù hợp | Văn bản | Payload E06 `fail_reason` | — | `HET_TAI_KHOAN` |
| `{{so_du_hien_tai}}` | ✅ | Số dư tài khoản (dạng số nguyên để tính toán) | Tiền (VND) | Payload E06 `balance` (đồng) | — | `3500` |
| `{{so_du_format}}` | ✅ | Số dư tài khoản đã định dạng để hiển thị cho KH dễ đọc | Tiền (VND) | CVM format từ `balance` | — | `3.500 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn thất bại cuộc gọi | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### Event: DATA_NGÀY_80_PHẦN_TRĂM (E08)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi lưu lượng data trong ngày vượt ngưỡng 80% hạn mức. CVM phải xử lý và gửi USSD trong vòng 2 giờ sau khi nhận event.
**Timing:** Ngay khi vượt ngưỡng 80%

> **Lưu ý ngưỡng cảnh báo:** Tỷ lệ 80% (hoặc x%) là ngưỡng do **CVM cấu hình**, không cứng trong OCS. OCS chỉ cần expose API nhận ngưỡng từ CVM và fire event khi vượt ngưỡng đó. Khi CVM muốn thay đổi ngưỡng cảnh báo (ví dụ: từ 80% xuống 70%), chỉ cần cập nhật cấu hình CVM mà không cần BSS/OCS thay đổi logic.
>
> **Rule tần suất gửi (2026-07-03):** Đề xuất ngưỡng **90%**. Mỗi ngày CVM chỉ gửi **1 lần** khi KH chạm ngưỡng. Nếu KH lặp lại pattern nhiều ngày, đến **lần thứ 3** (bắt trigger lần 3) mới chuyển sang đề xuất nâng/đổi gói thay vì chỉ nhắc mua thêm data ngày. Ngưỡng % và số lần do CVM cấu hình. Dùng trường `depleted_trigger_count`.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `DATA_THRESHOLD_80` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm vượt ngưỡng | OCS — thời điểm vượt ngưỡng | `2026-05-14 14:22:00` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_DATA_70K` |
| `daily_data_quota_mb` | integer | ✅ | Hạn mức data ngày (MB) | OCS — hạn mức data ngày theo gói | `2048` |
| `daily_data_used_mb` | integer | ✅ | Data đã dùng trong ngày (MB) | OCS — lưu lượng realtime | `1638` |
| `daily_data_pct` | float | ✅ | % data đã dùng | OCS — tính: `daily_data_used_mb / daily_data_quota_mb × 100` | `80.0` |
| `days_remaining` | integer | ✅ | Số ngày còn lại trong chu kỳ gói | OCS — số ngày còn lại trong chu kỳ gói | `12` |
| `addon_purchased_today` | boolean | ✅ | Đã mua gói bổ sung hôm nay chưa | OCS — lịch sử giao dịch trong ngày | `false` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao để xác nhận đúng KH | Văn bản | Payload E08 `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ✅ | Tên gói cước đang active để hiển thị ngữ cảnh | Văn bản | Payload E08 `current_plan` | — | `GOI_DATA_70K` |
| `{{phan_tram_da_dung}}` | ✅ | % data đã tiêu thụ trong ngày để cảnh báo KH | Số | Payload E08 `daily_data_pct` | — | `80.0%` |
| `{{data_da_dung_mb}}` | ✅ | Dung lượng data đã dùng để KH nắm con số cụ thể | Số | Payload E08 `daily_data_used_mb` | — | `1638 MB` |
| `{{han_muc_ngay_mb}}` | ✅ | Hạn mức data ngày theo gói để so sánh với mức đã dùng | Số | Payload E08 `daily_data_quota_mb` | — | `2048 MB` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trong chu kỳ gói để tạo cảm giác cấp bách | Số | Payload E08 `days_remaining` | — | `12 ngày` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_bo_sung_de_xuat}}` | ❌ | Tên gói bổ sung data được NBO đề xuất phù hợp với hành vi | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý cụ thể | `GOI_DATA_NGAY_5K` |

---

---

#### Event: NẠP_TIỀN_THÀNH_CÔNG (U01)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay sau khi giao dịch nạp tiền được xác nhận thành công. CVM phải gửi USSD trong vòng 3 phút để gợi ý khi KH còn đang trên màn hình nạp.
**Timing:** Trong vòng vài giây sau khi nạp thành công

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `TOPUP_SUCCESS` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm nạp | OCS — thời điểm giao dịch xác nhận | `2026-05-14 20:10:00` |
| `topup_amount` | integer | ✅ | Số tiền nạp (đồng) | OCS — số tiền giao dịch | `100000` |
| `balance_after` | integer | ✅ | Số dư sau khi nạp (đồng) | OCS — số dư sau giao dịch | `103500` |
| `topup_count` | integer | ✅ | Tổng số lần nạp (bao gồm lần này) | OCS — tổng số lần nạp tích lũy | `3` |
| `current_plan` | string | ❌ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_DATA_70K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao vừa nạp tiền thành công | Văn bản | Payload U01 `msisdn` | — | `0901234567` |
| `{{so_tien_nap}}` | ✅ | Số tiền vừa nạp để xác nhận giao dịch cho KH | Tiền (VND) | Payload U01 `topup_amount` (đồng) | — | `100.000 VNĐ` |
| `{{so_du_sau_nap}}` | ✅ | Số dư tài khoản sau khi nạp để KH biết còn bao nhiêu | Tiền (VND) | Payload U01 `balance_after` (đồng) | — | `103.500 VNĐ` |
| `{{lan_nap_thu_may}}` | ✅ | Lần nạp thứ mấy — dùng để cá nhân hóa nội dung theo độ trung thành | Số | Payload U01 `topup_count` | — | `3` |
| `{{ten_goi}}` | ❌ | Tên gói đang dùng để gợi ý đăng ký gói kèm nạp tiền | Văn bản | Payload U01 `current_plan` (optional) | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất phù hợp với hành vi nạp tiền | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý | `GOI_DATA_120K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn xác nhận nạp tiền | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### Event: TRA_CỨU_SỐ_DƯ (U03)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM đồng thời với việc xử lý tra cứu *101#. **CVM phải phản hồi trong vòng 2 giây** — nếu timeout, OCS trả kết quả tra cứu bình thường không kèm gợi ý. Đây là SLA nghiêm ngặt nhất trong toàn bộ hệ thống.
**Timing:** Ngay khi KH nhắn *101#

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `BALANCE_CHECK` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm tra cứu | OCS — thời điểm nhắn *101# | `2026-05-14 11:30:00` |
| `balance` | integer | ✅ | Số dư hiện tại (đồng) | OCS — số dư tài khoản realtime | `45000` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_DATA_70K` |
| `plan_expiry_date` | date | ✅ | Ngày hết hạn gói | `resource.msisdns.expiry_date` (BSS) | `2026-05-28` |
| `data_used_pct` | float | ❌ | % data chu kỳ đã dùng | OCS — tính từ lưu lượng chu kỳ | `62.5` |

**Param template:**

> SLA: CVM phải phản hồi trong 2 giây — nếu timeout, OCS trả kết quả bình thường không kèm gợi ý.

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao đang tra cứu số dư | Văn bản | Payload U03 `msisdn` | — | `0901234567` |
| `{{so_du_hien_tai}}` | ✅ | Số dư tài khoản hiện tại — hiển thị trong kết quả tra cứu | Tiền (VND) | Payload U03 `balance` (đồng) | — | `45.000 VNĐ` |
| `{{ten_goi}}` | ✅ | Tên gói cước đang active để hiển thị kèm số dư | Văn bản | Payload U03 `current_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_han_goi}}` | ✅ | Ngày hết hạn gói để KH cân nhắc gia hạn sớm | Ngày (DD/MM/YYYY) | Payload U03 `plan_expiry_date` | — | `28/05/2026` |
| `{{phan_tram_data_da_dung}}` | ❌ | % data chu kỳ đã tiêu thụ để KH có bức tranh tổng | Số | Payload U03 `data_used_pct` | không hiển thị % data | `62.5%` |
| `{{goi_gia_han_de_xuat}}` | ❌ | Tên gói gia hạn được NBO đề xuất phù hợp với mức dùng | Văn bản | CVM NBO cache (đã tính sẵn) | không gắn gợi ý | `GOI_DATA_120K` |

---

---

#### Event: HẾT_PHÚT_THOẠI_NỘI_MẠNG (E_VOICE_100_ONNET)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi quota thoại **nội mạng** trong gói về 0. CVM phải gửi USSD trong vòng 2 phút để KH thấy ngay sau cuộc gọi cuối.
**Timing:** Ngay khi hết 100% quota thoại nội mạng trong gói

> **Phân biệt với E_VOICE_100_OFFNET:** Thoại nội mạng và ngoại mạng thường có quota riêng trong gói. OCS cần tách 2 event để CVM gợi ý đúng loại gói bổ sung (gói nội mạng vs gói ngoại mạng). Nếu gói không tách quota nội/ngoại mà gộp chung thì dùng 1 event với trường `call_type = BOTH`.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `VOICE_ONNET_QUOTA_100` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm hết quota thoại nội mạng | OCS — thời điểm quota về 0 | `2026-06-03 14:22:00` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_THOAI_50K` |
| `onnet_quota_min` | integer | ✅ | Tổng quota thoại nội mạng của gói (phút) | OCS — theo định nghĩa gói | `300` |
| `days_remaining` | integer | ✅ | Số ngày còn lại trong chu kỳ gói | OCS — số ngày còn lại | `8` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS — số dư realtime | `25000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao hết quota thoại nội mạng | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh gợi ý gói mới | Văn bản | Payload `current_plan` | — | `GOI_THOAI_50K` |
| `{{quota_thoai_noi_mang_phut}}` | ✅ | Tổng quota thoại nội mạng của gói — để KH hiểu đã dùng hết bao nhiêu | Số | Payload `onnet_quota_min` | — | `300 phút` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trong chu kỳ — tạo cảm giác cấp bách | Số | Payload `days_remaining` | — | `8 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để KH biết có đủ tiền mua gói bổ sung không | Tiền (VND) | Payload `balance` | — | `25.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_thoai_noi_mang_bo_sung}}` | ❌ | Tên gói bổ sung thoại nội mạng NBO đề xuất | Văn bản | CVM NBO | không hiện gợi ý | `GOI_THOAI_NOIM_NGAY_5K` |

---

---

#### Event: HẾT_PHÚT_THOẠI_NGOẠI_MẠNG (E_VOICE_100_OFFNET)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi quota thoại **ngoại mạng** trong gói về 0. CVM phải gửi USSD trong vòng 2 phút để KH thấy ngay sau cuộc gọi cuối.
**Timing:** Ngay khi hết 100% quota thoại ngoại mạng trong gói

> **Lưu ý:** Thoại ngoại mạng hết quota thường nhạy cảm hơn về chi phí (cước tính ngoài gói cao hơn nội mạng). CVM nên gợi ý gói bổ sung ngoại mạng riêng hoặc cảnh báo mức phí phát sinh nếu KH tiếp tục gọi. Cần xác nhận với OCS xem quota nội/ngoại mạng có tách riêng trong tất cả các gói không.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `VOICE_OFFNET_QUOTA_100` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm hết quota thoại ngoại mạng | OCS — thời điểm quota về 0 | `2026-06-03 16:45:00` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_THOAI_50K` |
| `offnet_quota_min` | integer | ✅ | Tổng quota thoại ngoại mạng của gói (phút) | OCS — theo định nghĩa gói | `100` |
| `days_remaining` | integer | ✅ | Số ngày còn lại trong chu kỳ gói | OCS — số ngày còn lại | `8` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS — số dư realtime | `25000` |
| `offnet_rate_per_min` | integer | ❌ | Cước thoại ngoại mạng ngoài gói (đồng/phút) — để CVM hiển thị cảnh báo phí phát sinh | OCS — bảng giá cước | `1500` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao hết quota thoại ngoại mạng | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh gợi ý gói mới | Văn bản | Payload `current_plan` | — | `GOI_THOAI_50K` |
| `{{quota_thoai_ngoai_mang_phut}}` | ✅ | Tổng quota thoại ngoại mạng của gói — để KH biết đã dùng hết bao nhiêu | Số | Payload `offnet_quota_min` | — | `100 phút` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trong chu kỳ — tạo cảm giác cấp bách | Số | Payload `days_remaining` | — | `8 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để KH biết có đủ tiền mua gói bổ sung không | Tiền (VND) | Payload `balance` | — | `25.000 VNĐ` |
| `{{cuoc_ngoai_goi_dong_phut}}` | ❌ | Cước thoại ngoại mạng ngoài gói — cảnh báo KH về chi phí phát sinh nếu tiếp tục gọi | Tiền (VND) | Payload `offnet_rate_per_min` | không hiển thị cước | `1.500 VNĐ/phút` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_thoai_ngoai_mang_bo_sung}}` | ❌ | Tên gói bổ sung thoại ngoại mạng NBO đề xuất | Văn bản | CVM NBO | không hiện gợi ý | `GOI_THOAI_NGOAIM_NGAY_5K` |

---

---

#### Event: HẾT_TIỀN_TKC (E_ZERO_BALANCE)

**Mô tả:** Danh sách KH có số dư TKC về 0 (proactive — trước khi có cuộc gọi thất bại xảy ra)
**Trigger bởi:** OCS → BSS (nightly batch; phát hiện qua CDR giao dịch cuối làm cạn kiệt tài khoản)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Phân biệt với E06:** E06 trigger khi cuộc gọi đã thất bại (reactive); E_ZERO_BALANCE trigger khi balance về 0 do bất kỳ giao dịch nào (proactive). Hai trigger có thể fire đồng thời — cần rule chặn: nếu E_ZERO_BALANCE đã fire trong vòng 12h thì E06 không gửi thêm tin nhắn nạp tiền.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `zero_balance_timestamp` | datetime | ✅ | Thời điểm balance về 0 | OCS — thời điểm CDR làm cạn kiệt tài khoản | `2026-06-02 23:45:00` |
| `last_transaction_type` | string | ✅ | Loại giao dịch cuối làm hết tiền | OCS — phân loại CDR | `VOICE_CALL` hoặc `DATA_USAGE` hoặc `PLAN_REGISTER` |
| `current_plan` | string | ❌ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `plan_expiry_date` | date | ❌ | Ngày hết hạn gói | `resource.msisdns.expiry_date` (BSS) | `2026-06-14` |
| `topup_count_30d` | integer | ❌ | Số lần nạp trong 30 ngày gần nhất | OCS → BSS nightly batch | `2` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa hết tiền | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{ten_goi_hien_tai}}` | ❌ | Gói đang dùng để kèm gợi ý nạp đúng mức | Văn bản | CSV `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất kèm mức nạp tối thiểu để dùng tiếp | Văn bản | CVM NBO | không hiện gợi ý | `GOI_DATA_70K` |

---

---

#### Event: HỦY_GÓI_CƯỚC (E_CANCEL_PLAN)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi KH xác nhận hủy gói thành công. CVM phải gửi USSD/Push trong vòng 5 phút để giữ chân trước khi KH rời đi.
**Timing:** Ngay khi gói bị hủy thành công

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `PLAN_CANCELLED` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm hủy gói | OCS — thời điểm hủy thành công | `2026-06-03 15:10:00` |
| `cancelled_plan` | string | ✅ | Tên gói vừa bị hủy | OCS — tên gói vừa cancel | `GOI_DATA_70K` |
| `cancelled_plan_type` | string | ✅ | Loại gói vừa hủy | OCS | `DATA` hoặc `VOICE` hoặc `COMBO` |
| `cancel_reason` | string | ❌ | Lý do hủy (nếu OCS ghi nhận) | OCS — lý do KH chọn khi hủy | `SWITCH_TO_OTHER` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS — số dư realtime sau hủy | `50000` |
| `subscriber_tenure_days` | integer | ❌ | Số ngày KH đã dùng mạng | BSS tính từ `resource.msisdn_status_history.change_date` lần đầu tiên | `180` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa hủy gói | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi_da_huy}}` | ✅ | Tên gói vừa hủy để xác nhận giao dịch | Văn bản | Payload `cancelled_plan` | — | `GOI_DATA_70K` |
| `{{loai_goi_da_huy}}` | ✅ | Loại gói để phân nhánh nội dung giữ chân phù hợp | Văn bản | Payload `cancelled_plan_type` | — | `DATA` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn giữ chân | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_thay_the_de_xuat}}` | ❌ | Tên gói thay thế NBO đề xuất để KH cân nhắc | Văn bản | CVM NBO | không hiện gợi ý | `GOI_DATA_50K` |

---

---

#### Event: THUÊ_BAO_NGUY_CƠ_RỜI_MẠNG (E_CHURN_RISK)

**Trigger bởi:** OCS/BSS gọi API CVM (phân tích tổ hợp tín hiệu nhiều ngày)
**Yêu cầu kỹ thuật:** BSS/OCS phân tích tổ hợp tín hiệu theo chu kỳ (nightly), khi KH vượt ngưỡng nguy cơ rời mạng thì gọi API CVM để đưa vào luồng giữ chân. Các ngưỡng do CVM cấu hình, BSS/OCS không hardcode.
**Timing:** Ngay khi phát hiện KH vượt ngưỡng nguy cơ rời mạng (theo chu kỳ phân tích)

> **Mô tả:** KH có dấu hiệu nguy cơ rời mạng, xác định qua tổ hợp tín hiệu: không phát sinh lưu lượng liên tiếp 5–7 ngày, không phát sinh cước trong 30 ngày, doanh thu suy giảm ≥80% so với trung bình 2 tháng gần nhất, tỷ lệ bật/tắt sóng SIM bất thường. Phân biệt với `E_SEGMENT_UPDATE` (CHURN_RISK): E_CHURN_RISK là tín hiệu phát hiện dựa trên tiêu chí cụ thể; E_SEGMENT_UPDATE là kết quả cập nhật phân khúc tổng hợp. Xem thêm bản `data-contract-template-lifecycle.md`.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS/OCS — event name cố định | `CHURN_RISK_DETECTED` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện nguy cơ rời mạng | BSS/OCS — thời điểm phân tích đủ điều kiện | `2026-06-03 03:00:00` |
| `no_usage_days` | integer | ✅ | Số ngày liên tiếp không phát sinh lưu lượng | OCS/BSS tính từ CDR | `6` |
| `no_charge_days` | integer | ✅ | Số ngày không phát sinh cước | OCS/BSS tính từ CDR/giao dịch | `30` |
| `revenue_avg_2months` | integer | ❌ | Doanh thu trung bình 2 tháng gần nhất (đồng) | OCS/BSS tổng hợp | `80000` |
| `revenue_drop_pct` | float | ✅ | % suy giảm doanh thu so với trung bình 2 tháng gần nhất | BSS tính | `87.5` |
| `sim_on_off_ratio` | float | ❌ | Tỷ lệ bật/tắt sóng của SIM trong kỳ | HLR/BSS | `0.65` |
| `churn_risk_level` | string | ❌ | Mức độ nguy cơ | BSS/CVM phân loại | `HIGH` hoặc `MEDIUM` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `age_segment` | string | ❌ | Tuổi/phân khúc tuổi KH | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `25-34` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `540` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi giữ chân (nếu có) | OCS/CVM | `KM_GIUCHAN_30PCT` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có nguy cơ rời mạng | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin giữ chân | Văn bản | Payload `full_name` hoặc CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{muc_do_nguy_co}}` | ❌ | Mức độ nguy cơ để CVM chọn cường độ chiến dịch | Văn bản | Payload `churn_risk_level` | mức mặc định | `HIGH` |
| `{{goi_giu_chan_de_xuat}}` | ❌ | Tên gói/ưu đãi CVM đề xuất để giữ chân | Văn bản | CVM NBO | không hiện gợi ý | `GOI_GIAM_GIA_30PCT` |

---

---

### 2.2 Viễn thông — Batch/CSV

#### File: `app_installed_no_open_{YYYYMMDD}.csv` (E04)

**Mô tả:** Danh sách KH đã cài app nhưng chưa mở trong 24h
**Trigger bởi:** BSS (consume từ Kafka SuperApp events)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `installed_at` | datetime | ✅ | Thời điểm cài app | `app_install_log.installed_at` (SuperApp → Kafka → BSS) | `2026-05-13 10:15:00` |
| `device_type` | string | ✅ | Loại thiết bị | `app_install_log.device_type` (SuperApp → Kafka → BSS) | `ANDROID` |
| `os_version` | string | ❌ | Phiên bản hệ điều hành | `app_install_log.os_version` (SuperApp → Kafka → BSS) | `Android 14` |
| `firebase_token` | string | ✅ | Firebase device token để gửi Push | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |
| `segment_age` | string | ❌ | Phân khúc tuổi | `crm.request_register_info.ekyc_data` → `date_of_birth` hoặc `esim_agency.individual_info` | `15-18` |
| `segment_job` | string | ❌ | Phân khúc nghề nghiệp | `crm.request_register_info.ekyc_data` → nghề nghiệp hoặc `esim_agency.individual_info` | `STUDENT` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn nhắc mở app | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH đã cài app nhưng chưa mở | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{loai_thiet_bi}}` | ❌ | Loại thiết bị để cá nhân hóa nội dung hướng dẫn | Văn bản | CSV `device_type` | không đề cập thiết bị | `ANDROID` |

---

---

#### File: `zero_usage_D3_{YYYYMMDD}.csv` (E05)

**Mô tả:** Danh sách KH có tổng lưu lượng = 0 sau 72h kích hoạt SIM
**Trigger bởi:** OCS → BSS (nightly batch)
**Thời điểm push:** 02:00–04:00 ngày N+3

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `activation_date` | datetime | ✅ | NGAY_0 | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-05-11 09:00:00` |
| `data_usage_d1_mb` | float | ✅ | Lưu lượng data ngày 1 (MB) | OCS → BSS nightly batch | `0` |
| `data_usage_d2_mb` | float | ✅ | Lưu lượng data ngày 2 (MB) | OCS → BSS nightly batch | `0` |
| `data_usage_d3_mb` | float | ✅ | Lưu lượng data ngày 3 (MB) | OCS → BSS nightly batch | `0` |
| `voice_usage_d3_min` | float | ✅ | Lưu lượng thoại ngày 3 (phút) | OCS → BSS nightly batch | `0` |
| `sms_count_d3` | integer | ✅ | Số SMS ngày 3 | OCS → BSS nightly batch | `0` |
| `device_type` | string | ❌ | Loại thiết bị | `app_install_log.device_type` (SuperApp → Kafka → BSS) | `ANDROID` |
| `has_app` | boolean | ❌ | Đã cài app chưa | Kiểm tra tồn tại bản ghi trong `app_install_log` theo `msisdn` | `true` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn kích hoạt dùng mạng | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có lưu lượng = 0 sau D+3 | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_ngay_chua_dung}}` | ✅ | Số ngày từ khi kích hoạt đến nay KH chưa dùng mạng | Số | CVM tính = `ngay_hien_tai - activation_date` | — | `3 ngày` |
| `{{loai_thiet_bi}}` | ❌ | Loại thiết bị để gợi ý hướng dẫn cài đặt phù hợp | Văn bản | CSV `device_type` | không đề cập thiết bị | `IOS` |

---

---

#### File: `change_pkg_view_{YYYYMMDD}.csv` (E09)

**Mô tả:** Danh sách KH vào màn hình đổi gói nhưng thoát không đăng ký
**Trigger bởi:** SuperApp → Kafka → BSS
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `view_timestamp` | datetime | ✅ | Thời điểm vào màn hình | `change_pkg_view_log.view_timestamp` (SuperApp → Kafka → BSS) | `2026-05-13 21:10:00` |
| `time_on_screen_sec` | integer | ✅ | Thời gian trên màn hình (giây) | `change_pkg_view_log.time_on_screen` (SuperApp → Kafka → BSS) | `47` |
| `view_count_today` | integer | ✅ | Số lần xem trong ngày | `change_pkg_view_log.view_count` (SuperApp → Kafka → BSS) | `2` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `total_data_30d_mb` | float | ❌ | Tổng data 30 ngày (MB) — từ OCS | OCS → BSS nightly batch (tổng hợp 30 ngày) | `45000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn nhắc đổi gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa thoát màn hình đổi gói | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh cho gợi ý đổi gói | Văn bản | CSV `current_plan` | — | `GOI_DATA_70K` |
| `{{tong_data_30_ngay_gb}}` | ❌ | Tổng data 30 ngày gần nhất — so sánh với hạn mức gói hiện tại | Số | CSV `total_data_30d_mb` → CVM convert sang GB | không hiện so sánh với mức dùng thực | `45.0 GB` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất phù hợp với mức dùng data thực tế | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý | `GOI_DATA_120K` |

---

---

#### File: `traffic_spike_{YYYYMMDD}.csv` (E13)

**Mô tả:** Danh sách KH có lưu lượng tăng đột biến > 3x mức nền
**Trigger bởi:** OCS → BSS (nightly) + CDR (baseline)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `spike_timestamp` | datetime | ✅ | Thời điểm phát hiện tăng đột biến | OCS → BSS nightly batch | `2026-05-13 23:00:00` |
| `spike_hour` | integer | ✅ | Giờ xảy ra (0–23) | OCS → BSS nightly batch | `22` |
| `traffic_spike_mb` | float | ✅ | Lưu lượng giờ đột biến (MB) | OCS → BSS nightly batch | `512` |
| `baseline_mb` | float | ✅ | Mức nền cùng khung giờ (MB) | BSS tính từ `cdr.*` lịch sử 30 ngày cùng khung giờ | `150` |
| `spike_ratio` | float | ✅ | Tỉ lệ tăng (spike/baseline) | BSS tính: `traffic_spike_mb / baseline_mb` | `3.41` |
| `device_type` | string | ❌ | Loại thiết bị | `app_install_log.device_type` (SuperApp → Kafka → BSS) | `ANDROID` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có lưu lượng tăng đột biến | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{luong_dot_bien_mb}}` | ✅ | Lưu lượng data trong giờ đột biến để minh hoạ mức tiêu thụ | Số | CSV `traffic_spike_mb` | — | `512 MB` |
| `{{muc_nen_mb}}` | ✅ | Mức nền cùng khung giờ 30 ngày trước — làm cơ sở so sánh | Số | CSV `baseline_mb` | — | `150 MB` |
| `{{ti_le_tang}}` | ✅ | Tỉ lệ tăng so với mức nền — tạo cảm giác cấp bách cho KH | Số | CSV `spike_ratio` | — | `3.4x` |
| `{{gio_xay_ra}}` | ✅ | Giờ xảy ra đột biến để KH nhận ra đúng thời điểm của họ | Văn bản | CSV `spike_hour` | — | `22:00` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa cảnh báo lưu lượng đột biến | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### File: `plan_register_{YYYYMMDD}.csv` (U02)

**Mô tả:** Danh sách KH đăng ký/gia hạn gói lần ≥ 2 trong ngày
**Trigger bởi:** OCS → BSS (nightly)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `plan_name` | string | ✅ | Tên gói đã đăng ký | OCS → BSS nightly batch | `GOI_DATA_120K` |
| `plan_type` | string | ✅ | Loại gói | OCS → BSS nightly batch | `DATA` hoặc `VOICE` hoặc `COMBO` |
| `plan_expiry_date` | date | ✅ | Ngày hết hạn gói | `resource.msisdns.expiry_date` (BSS) | `2026-06-14` |
| `plan_register_count` | integer | ✅ | Tổng số lần đăng ký gói | OCS → BSS nightly batch (tổng tích lũy) | `3` |
| `register_timestamp` | datetime | ✅ | Thời điểm đăng ký | OCS → BSS nightly batch | `2026-05-13 18:30:00` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa đăng ký/gia hạn gói | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_da_dang_ky}}` | ✅ | Tên gói KH vừa đăng ký để xác nhận giao dịch | Văn bản | CSV `plan_name` | — | `GOI_DATA_120K` |
| `{{loai_goi}}` | ✅ | Loại gói để phân nhánh nội dung tin nhắn phù hợp | Văn bản | CSV `plan_type` | — | `DATA` |
| `{{ngay_het_han_goi}}` | ✅ | Ngày hết hạn gói để KH nắm thời hạn sử dụng | Ngày (DD/MM/YYYY) | CSV `plan_expiry_date` | — | `14/06/2026` |
| `{{lan_dang_ky_thu_may}}` | ✅ | Lần đăng ký/gia hạn thứ mấy — dùng để cá nhân hóa theo mức trung thành | Số | CSV `plan_register_count` | — | `3` |
| `{{goi_bo_sung_de_xuat}}` | ❌ | Tên gói bổ sung NBO đề xuất kèm gói chính vừa đăng ký | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý bổ sung | `GOI_DATA_NGAY_5K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin xác nhận đăng ký gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### File: `otp_detection_{YYYYMMDD}.csv` (U04)

**Mô tả:** Danh sách KH nhận SMS OTP từ app bên thứ 3 (ngân hàng, giải trí, mạng xã hội)
**Trigger bởi:** HLR (Home Location Register — Hệ thống Đăng ký vị trí thuê bao)
**Cơ chế:** HLR phát hiện SMS OTP → đẩy qua Kafka → export CSV trực tiếp vào SFTP CVM (không đi qua BSS)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | HLR log | `0901234567` |
| `sender_number` | string | ✅ | Đầu số gửi OTP | HLR log — đầu số gửi SMS | `VPBANK` |
| `app_category` | string | ✅ | Phân loại ứng dụng | HLR — phân loại từ danh sách đầu số thương mại | `BANKING` hoặc `ENTERTAINMENT` hoặc `SOCIAL` |
| `app_name` | string | ❌ | Tên ứng dụng cụ thể | HLR — tra cứu từ danh mục đầu số | `VPBank NEO` |
| `otp_count_today` | integer | ✅ | Số lần nhận OTP từ app này trong ngày | HLR log — đếm theo ngày | `2` |
| `current_plan` | string | ❌ | Gói data hiện tại (từ OCS) | OCS (nếu HLR có thể lấy) hoặc để trống | `GOI_DATA_70K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH được phát hiện nhận OTP từ app bên thứ 3 | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{loai_ung_dung}}` | ✅ | Phân loại app gửi OTP — để CVM chọn nội dung gợi ý phù hợp | Văn bản | CSV `app_category` | — | `BANKING` |
| `{{ten_ung_dung}}` | ❌ | Tên app cụ thể gửi OTP — để cá nhân hóa tin nhắn hơn | Văn bản | CSV `app_name` (optional) | dùng `{{loai_ung_dung}}` thay thế | `VPBank NEO` |
| `{{goi_data_de_xuat}}` | ❌ | Tên gói data NBO đề xuất dựa trên loại app KH đang dùng | Văn bản | CVM NBO dựa trên `app_category` | không hiện gợi ý | `GOI_DATA_70K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn gợi ý dựa trên OTP | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### File: `billing_2month_{YYYYMM}.csv` (U05-A)

**Mô tả:** Danh sách KH dùng **gói data tháng (MONTHLY)** bị hết data trước ngày N_CUTOFF trong 2 tháng liên tiếp — N_CUTOFF do CVM cấu hình (mặc định: ngày 25)
**Trigger bởi:** OCS → BSS (nightly), export đầu tháng thứ 3
**Thời điểm push:** 02:00–04:00 ngày 1 của tháng thứ 3

> **Phạm vi áp dụng:** Chỉ KH đang dùng gói có hạn mức data cố định theo tháng (gói trả trước tháng, gói combo tháng...). Không áp dụng cho gói data/ngày — xem file `daily_quota_pattern_{YYYYMM}.csv` (U05-B).
>
> **Ngưỡng cấu hình:** Ngày cut-off (mặc định 25) do CVM cấu hình — BSS không hardcode. Khi CVM thay đổi ngưỡng thì BSS cần được notify để export lại đúng điều kiện. Cần thống nhất cơ chế truyền ngưỡng từ CVM sang BSS (config file, API, hay email thông báo thủ công).

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `month1_depleted_date` | date | ✅ | Ngày hết data tháng 1 (trước ngày N_CUTOFF) | OCS → BSS nightly batch (ngày data về 0 tháng 1) | `2026-03-18` |
| `month2_depleted_date` | date | ✅ | Ngày hết data tháng 2 (trước ngày N_CUTOFF) | OCS → BSS nightly batch (ngày data về 0 tháng 2) | `2026-04-21` |
| `month1_total_data_gb` | float | ✅ | Tổng data KH đã dùng trong tháng 1 (GB) | OCS → BSS nightly batch | `15.2` |
| `month2_total_data_gb` | float | ✅ | Tổng data KH đã dùng trong tháng 2 (GB) | OCS → BSS nightly batch | `16.8` |
| `suggested_plan` | string | ❌ | Gợi ý gói tháng nâng lên | OCS hoặc CVM NBO tự tính dựa trên mức dùng trung bình | `GOI_DATA_120K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin gợi ý nâng gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH hết data sớm 2 tháng liên tiếp | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói tháng đang dùng — làm cơ sở so sánh khi đề xuất nâng | Văn bản | CSV `current_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_data_thang_1}}` | ✅ | Ngày hết data tháng trước — minh chứng pattern hết sớm | Ngày (DD/MM/YYYY) | CSV `month1_depleted_date` | — | `18/03/2026` |
| `{{ngay_het_data_thang_2}}` | ✅ | Ngày hết data tháng gần nhất — xác nhận pattern lặp lại | Ngày (DD/MM/YYYY) | CSV `month2_depleted_date` | — | `21/04/2026` |
| `{{tong_data_thang_1_gb}}` | ✅ | Tổng data tháng trước KH đã dùng trước khi hết | Số | CSV `month1_total_data_gb` | — | `15.2 GB` |
| `{{tong_data_thang_2_gb}}` | ✅ | Tổng data tháng gần nhất KH đã dùng trước khi hết | Số | CSV `month2_total_data_gb` | — | `16.8 GB` |
| `{{goi_nang_de_xuat}}` | ❌ | Tên gói tháng NBO đề xuất dựa trên mức dùng thực tế | Văn bản | CSV `suggested_plan` hoặc CVM NBO | không hiện tên gói cụ thể | `GOI_DATA_120K` |

---

---

#### File: `daily_quota_pattern_{YYYYMM}.csv` (U05-B)

**Mô tả:** Danh sách KH dùng **gói data ngày (DAILY)** có pattern hết quota data/ngày thường xuyên — số ngày hết ≥ N_DAYS_THRESHOLD trong M_MONTHS_THRESHOLD tháng liên tiếp; N_DAYS_THRESHOLD và M_MONTHS_THRESHOLD do CVM cấu hình
**Trigger bởi:** OCS → BSS (nightly), export đầu tháng thứ (M+1)
**Thời điểm push:** 02:00–04:00 ngày 1 của tháng thứ (M+1)

> **Phạm vi áp dụng:** Chỉ KH đang dùng gói có quota data reset hàng ngày (gói ngày, gói tuần có chia data/ngày...). Không áp dụng cho gói tháng — xem file `billing_2month_{YYYYMM}.csv` (U05-A).
>
> **Ngưỡng cấu hình — toàn bộ do CVM cấu hình, BSS không hardcode:**
> - `N_DAYS_THRESHOLD`: số ngày hết quota/tháng để tính là "thường xuyên" — **[A1] Assumption mặc định: 15 ngày/tháng (≥50%). Cần PO xác nhận.**
> - `M_MONTHS_THRESHOLD`: số tháng liên tiếp có pattern — **[A2] Assumption mặc định: 2 tháng liên tiếp (nhất quán với U05-A). Cần PO xác nhận.**
> - `UPSELL_MODE`: chiến lược gợi ý — `DAILY_UPGRADE` (nâng gói ngày lớn hơn), `MONTHLY_UPSELL` (chuyển sang gói tháng), hoặc `BOTH` (đề xuất cả 2, CVM chọn theo NBO) — **[A3] Assumption mặc định: `BOTH`. Cần PO xác nhận.**
>
> **⚠️ Vấn đề nguồn dữ liệu (Q21):** Để biết "ngày X KH có hết quota/ngày không", OCS cần có event quota/ngày về 0 hoặc BSS phải tính từ CDR. Nếu OCS không có event này thì compute nặng — cần xác nhận với BSS/Tech trước khi commit schema.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói ngày đang dùng | OCS → BSS nightly batch | `GOI_DATA_NGAY_10K` |
| `daily_quota_mb` | integer | ✅ | Hạn mức data/ngày của gói (MB) | OCS — theo định nghĩa gói | `500` |
| `month1_depleted_days` | integer | ✅ | Số ngày trong tháng 1 mà quota/ngày bị hết | OCS (event quota/ngày = 0) hoặc BSS tính từ CDR — **cần xác nhận nguồn (Q21)** | `18` |
| `month2_depleted_days` | integer | ✅ | Số ngày trong tháng 2 mà quota/ngày bị hết | OCS (event quota/ngày = 0) hoặc BSS tính từ CDR — **cần xác nhận nguồn (Q21)** | `20` |
| `month1_avg_depleted_hour` | float | ❌ | Giờ trung bình hết quota trong tháng 1 (0–24) — để CVM hiểu KH hết vào buổi nào | OCS hoặc BSS từ CDR — có thể không có nếu nguồn là event đơn giản | `14.5` |
| `month2_avg_depleted_hour` | float | ❌ | Giờ trung bình hết quota trong tháng 2 (0–24) | OCS hoặc BSS từ CDR | `13.8` |
| `month1_total_data_gb` | float | ✅ | Tổng data KH đã dùng trong tháng 1 (GB) | OCS → BSS nightly batch | `14.2` |
| `month2_total_data_gb` | float | ✅ | Tổng data KH đã dùng trong tháng 2 (GB) | OCS → BSS nightly batch | `15.0` |
| `suggested_daily_plan` | string | ❌ | Gói ngày lớn hơn được đề xuất (nâng quota/ngày) | OCS hoặc CVM NBO | `GOI_DATA_NGAY_15K` |
| `suggested_monthly_plan` | string | ❌ | Gói tháng được đề xuất nếu upsell (tính từ avg data/ngày × 30) | CVM NBO tự tính — không lấy từ BSS/OCS | `GOI_DATA_70K` |

**Param template:**

> CVM phân nhánh nội dung theo `UPSELL_MODE` (cấu hình nội bộ CVM): `DAILY_UPGRADE` chỉ gợi ý gói ngày lớn hơn; `MONTHLY_UPSELL` chỉ gợi ý gói tháng; `BOTH` đề xuất cả 2 để KH chọn.

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin gợi ý | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có pattern hết data/ngày thường xuyên | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói ngày đang dùng — làm ngữ cảnh cho gợi ý | Văn bản | CSV `current_plan` | — | `GOI_DATA_NGAY_10K` |
| `{{han_muc_data_ngay_mb}}` | ✅ | Hạn mức data/ngày của gói hiện tại — để KH hiểu tại sao hay hết | Số | CSV `daily_quota_mb` | — | `500 MB` |
| `{{so_ngay_het_thang_1}}` | ✅ | Số ngày tháng trước KH bị hết quota data/ngày — minh chứng pattern | Số | CSV `month1_depleted_days` | — | `18 ngày` |
| `{{so_ngay_het_thang_2}}` | ✅ | Số ngày tháng gần nhất KH bị hết quota data/ngày — xác nhận lặp lại | Số | CSV `month2_depleted_days` | — | `20 ngày` |
| `{{tong_data_thang_1_gb}}` | ✅ | Tổng data tháng trước — để so sánh với gói tháng nếu upsell | Số | CSV `month1_total_data_gb` | — | `14.2 GB` |
| `{{tong_data_thang_2_gb}}` | ✅ | Tổng data tháng gần nhất — xác nhận mức dùng ổn định | Số | CSV `month2_total_data_gb` | — | `15.0 GB` |
| `{{goi_ngay_nang_de_xuat}}` | ❌ | Gói ngày lớn hơn NBO đề xuất *(hiển thị khi UPSELL_MODE = DAILY_UPGRADE hoặc BOTH)* | Văn bản | CSV `suggested_daily_plan` hoặc CVM NBO | không hiện nếu không có | `GOI_DATA_NGAY_15K` |
| `{{goi_thang_upsell_de_xuat}}` | ❌ | Gói tháng NBO đề xuất thay thế *(hiển thị khi UPSELL_MODE = MONTHLY_UPSELL hoặc BOTH)* | Văn bản | CSV `suggested_monthly_plan` hoặc CVM NBO | không hiện nếu không có | `GOI_DATA_70K` |

---

---

#### File: `plan_change_{YYYYMMDD}.csv` (U06)

**Mô tả:** Danh sách KH chuyển đổi loại gói thành công trong ngày
**Trigger bởi:** OCS → BSS (nightly)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `old_plan` | string | ✅ | Tên gói cũ | OCS → BSS nightly batch | `GOI_THOAI_50K` |
| `old_plan_type` | string | ✅ | Loại gói cũ | OCS → BSS nightly batch | `VOICE` |
| `new_plan` | string | ✅ | Tên gói mới | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `new_plan_type` | string | ✅ | Loại gói mới | OCS → BSS nightly batch | `DATA` |
| `new_plan_expiry_date` | date | ✅ | Ngày hết hạn gói mới | `resource.msisdns.expiry_date` (BSS, sau khi cập nhật) | `2026-06-14` |
| `change_timestamp` | datetime | ✅ | Thời điểm chuyển gói | OCS → BSS nightly batch | `2026-05-13 15:45:00` |
| `data_diff_gb` | float | ❌ | Chênh lệch data giữa gói mới và cũ (GB) | BSS tính: `new_plan.data_quota - old_plan.data_quota` | `+5.0` |
| `voice_diff_min` | float | ❌ | Chênh lệch thoại giữa gói mới và cũ (phút) | BSS tính: `new_plan.voice_quota - old_plan.voice_quota` | `-300` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa chuyển đổi gói thành công | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_cu}}` | ✅ | Tên gói cũ để KH xác nhận đúng giao dịch chuyển đổi | Văn bản | CSV `old_plan` | — | `GOI_THOAI_50K` |
| `{{loai_goi_cu}}` | ✅ | Loại gói cũ để phân nhánh nội dung so sánh | Văn bản | CSV `old_plan_type` | — | `VOICE` |
| `{{ten_goi_moi}}` | ✅ | Tên gói mới để xác nhận giao dịch và khuyến khích dùng thử | Văn bản | CSV `new_plan` | — | `GOI_DATA_70K` |
| `{{loai_goi_moi}}` | ✅ | Loại gói mới để phân nhánh nội dung hướng dẫn | Văn bản | CSV `new_plan_type` | — | `DATA` |
| `{{ngay_het_han_goi_moi}}` | ✅ | Ngày hết hạn gói mới để KH nắm thời hạn sử dụng | Ngày (DD/MM/YYYY) | CSV `new_plan_expiry_date` | — | `14/06/2026` |
| `{{chenh_lech_data_gb}}` | ❌ | Chênh lệch data giữa gói mới và cũ — thể hiện lợi ích khi nâng | Số | CSV `data_diff_gb` | không hiển thị so sánh data | `+5.0 GB` |
| `{{chenh_lech_thoai_phut}}` | ❌ | Chênh lệch thoại giữa gói mới và cũ — thể hiện đánh đổi khi chuyển | Số | CSV `voice_diff_min` | không hiển thị so sánh thoại | `-300 phút` |
| `{{goi_bo_sung_de_xuat}}` | ❌ | Tên gói bổ sung NBO đề xuất kèm gói mới vừa chuyển | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý | `GOI_DATA_NGAY_5K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin xác nhận chuyển gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### File: `sim_swap_{YYYYMMDD}.csv` (U07)

**Mô tả:** Danh sách KH đổi SIM nội mạng thành công trong ngày
**Trigger bởi:** BSS (sim_swap event)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `old_sim_type` | string | ✅ | Loại SIM cũ | `resource.sims.sim_type` (bản ghi SIM cũ) | `PHYSICAL` |
| `new_sim_type` | string | ✅ | Loại SIM mới | `resource.sims.sim_type` (bản ghi SIM mới) | `ESIM` |
| `new_iccid` | string(20) | ✅ | ICCID SIM mới | `resource.sims.iccid` (bản ghi SIM mới) | `8984012345678901235` |
| `swap_timestamp` | datetime | ✅ | Thời điểm đổi SIM | `resource.sims.activation_date` (SIM mới) | `2026-05-13 11:00:00` |
| `current_plan` | string | ✅ | Gói cước được giữ nguyên | OCS → BSS nightly batch | `GOI_DATA_120K` |
| `firebase_token` | string | ❌ | Firebase token (nếu có app) | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa đổi SIM thành công | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{loai_sim_cu}}` | ✅ | Loại SIM cũ để xác nhận đúng giao dịch đổi SIM | Văn bản | CSV `old_sim_type` | — | `PHYSICAL` |
| `{{loai_sim_moi}}` | ✅ | Loại SIM mới KH vừa chuyển sang | Văn bản | CSV `new_sim_type` | — | `ESIM` |
| `{{ma_iccid_moi}}` | ✅ | Mã ICCID SIM mới để KH lưu làm bằng chứng kích hoạt | Văn bản | CSV `new_iccid` | — | `8984012345678901235` |
| `{{ten_goi_giu_nguyen}}` | ✅ | Tên gói cước được giữ nguyên sau khi đổi SIM — reassurance KH | Văn bản | CSV `current_plan` | — | `GOI_DATA_120K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin xác nhận đổi SIM | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

#### File: `birthday_list_{YYYYMMDD}.csv` (U09)

**Mô tả:** Danh sách KH có sinh nhật hoặc kỷ niệm mua SIM trong ngày
**Trigger bởi:** BSS (quét hàng ngày)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `event_type` | string | ✅ | Loại sự kiện | BSS tổng hợp: `BIRTHDAY` nếu khớp `birthday`, `SIM_ANNIVERSARY` nếu khớp `activation_date` | `BIRTHDAY` hoặc `SIM_ANNIVERSARY` |
| `event_year` | integer | ✅ | Số năm (tuổi hoặc năm thứ mấy dùng SIM) | BSS tính từ `crm.customers.birthday` (tuổi) hoặc `resource.sims.activation_date` (năm dùng SIM) | `25` |
| `zalo_oa_id` | string | ❌ | Zalo OA ID nếu KH đã liên kết | `crm.customers.zalo_oa_id` (nếu có trường này) | `84901234567` |
| `current_plan` | string | ❌ | Gói đang dùng (từ OCS) | OCS → BSS nightly batch | `GOI_DATA_70K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa lời chúc sinh nhật/kỷ niệm | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có sự kiện sinh nhật hoặc kỷ niệm SIM | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{loai_su_kien}}` | ✅ | Loại sự kiện để CVM phân nhánh nội dung lời chúc phù hợp | Văn bản | CSV `event_type` | — | `BIRTHDAY` |
| `{{so_tuoi_hoac_so_nam}}` | ✅ | Số tuổi (sinh nhật) hoặc số năm dùng SIM (kỷ niệm) — điểm nhấn cá nhân hóa | Số | CSV `event_year` | — | `25` |
| `{{ten_goi_hien_tai}}` | ❌ | Gói đang dùng — để kèm ưu đãi sinh nhật phù hợp gói | Văn bản | CSV `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{zalo_oa_id}}` | ❌ ⚠️ | Zalo OA ID để gửi tin qua Zalo — kênh ưu tiên cho sự kiện đặc biệt | Văn bản | CSV `zalo_oa_id` — chưa xác nhận BSS có trường này | chuyển sang SMS dự phòng | `84901234567` |

---

---

#### File: `subscriber_demographic_{YYYYMMDD}.csv` (U10)

**Mô tả:** Danh sách toàn bộ KH ACTIVE kèm thông tin nhân khẩu học (cho ngày lễ)
**Trigger bởi:** BSS (chuẩn bị trước ngày lễ 2 ngày)
**Thời điểm push:** 02:00–04:00 của ngày chuẩn bị (D-2 trước ngày lễ)

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `status` | string | ✅ | Trạng thái thuê bao | `crm.subscribers.status` | `ACTIVE` |
| `gender` | string | ❌ | Giới tính | `crm.request_register_info.ekyc_data` → `gender` hoặc `esim_agency.individual_info` — chưa xác nhận | `MALE` / `FEMALE` / `OTHER` |
| `age_segment` | string | ❌ | Phân khúc tuổi | `crm.request_register_info.ekyc_data` → `date_of_birth` hoặc `esim_agency.individual_info` — chưa xác nhận | `19-24` |
| `job_segment` | string | ❌ | Phân khúc nghề nghiệp | Chưa có nguồn trong BSS — cần clarify với PO/Tech | `TEACHER` |
| `zalo_oa_id` | string | ❌ | Zalo OA ID | `crm.customers.zalo_oa_id` (nếu có trường này) — chưa xác nhận | `84901234567` |
| `current_plan` | string | ❌ | Gói đang dùng (từ OCS) | OCS → BSS nightly batch | `GOI_DATA_70K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa lời chúc ngày lễ | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH nhận lời chúc ngày lễ | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_ngay_le}}` | ✅ | Tên ngày lễ cụ thể để cá nhân hóa nội dung lời chúc | Văn bản | CVM cấu hình tĩnh (lịch ngày lễ cố định) | — | `Quốc tế Phụ nữ 8/3` |
| `{{gioi_tinh}}` | ❌ ⚠️ | Giới tính KH để phân nhánh nội dung lời chúc theo giới | Văn bản | CSV `gender` — chưa xác nhận nguồn BSS | không phân nhánh theo giới tính | `FEMALE` |
| `{{phan_khuc_nghe_nghiep}}` | ❌ ⚠️ | Nghề nghiệp KH để gửi nội dung phù hợp ngày lễ nghề nghiệp | Văn bản | CSV `job_segment` — chưa có nguồn trong BSS | không gửi nội dung nghề nghiệp | `TEACHER` |
| `{{ten_goi_hien_tai}}` | ❌ | Gói đang dùng — để kèm ưu đãi ngày lễ phù hợp gói | Văn bản | CSV `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |

---

---

#### File: `usage_need_analysis_{YYYYMMDD}.csv` (E_USAGE_NEED_ANALYSIS)

**Mô tả:** Danh sách KH cần tư vấn gói cước theo nhu cầu sử dụng thực tế. CVM/BSS phân tích mức sử dụng data/thoại so với gói hiện tại để xác định KH có **nhu cầu lớn** cần tư vấn gói lớn hơn, hoặc **nhu cầu nhỏ** cần tư vấn gói nhỏ/tiết kiệm hơn.
**Trigger bởi:** OCS → BSS (nightly batch, phân tích mức sử dụng theo chu kỳ)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **⚠️ Cập nhật 2026-07-07 — bản chuẩn ở `data-contract-template-lifecycle.md`:** đã bổ sung đủ 13 trường theo rà soát (Họ tên, Giới tính, Tuổi KH qua `date_of_birth`, SĐT, Tuổi thuê bao, Tên gói, Giá gói, Chu kỳ gói, **Ngày hết hạn gói + số ngày tính từ lúc hết hạn**, Chương trình KM, Gói gợi ý, Kênh đăng ký, Hình thức đăng ký). File này (bản gốc) chưa cập nhật các trường đó — xem bản `-lifecycle.md` để lấy schema đầy đủ.

> **Rule phân loại nhu cầu:** `usage_need_segment = HIGH_NEED` khi mức sử dụng thực tế cao hơn đáng kể so với hạn mức/giá trị gói hiện tại; CVM tư vấn gói lớn hơn. `usage_need_segment = LOW_NEED` khi mức sử dụng thực tế thấp hơn đáng kể so với gói hiện tại; CVM tư vấn gói nhỏ hơn hoặc gói tiết kiệm hơn. Ngưỡng cụ thể do PO/CVM cấu hình theo từng loại tài nguyên và từng nhóm gói, không hard-code trong contract này.
>
> **Điều kiện dữ liệu tối thiểu:** Mỗi bản ghi phải có ít nhất một cặp dữ liệu đủ để tính nhu cầu: `avg_data_usage_mb` + `current_plan_data_quota_mb`, hoặc `avg_voice_usage_min` + `current_plan_voice_quota_min`.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `analysis_period_cycles` | integer | ✅ | Số chu kỳ gần nhất dùng để phân tích nhu cầu | BSS/CVM cấu hình | `3` |
| `avg_data_usage_mb` | float | ❌ | Data trung bình KH đã dùng trong mỗi chu kỳ phân tích | OCS → BSS nightly batch | `35840` |
| `avg_voice_usage_min` | float | ❌ | Số phút thoại trung bình KH đã dùng trong mỗi chu kỳ phân tích | OCS → BSS nightly batch | `420` |
| `current_plan_data_quota_mb` | float | ❌ | Hạn mức data của gói hiện tại trong mỗi chu kỳ | OCS — định nghĩa gói | `20480` |
| `current_plan_voice_quota_min` | float | ❌ | Hạn mức thoại của gói hiện tại trong mỗi chu kỳ | OCS — định nghĩa gói | `300` |
| `usage_to_quota_pct` | float | ✅ | Tỷ lệ sử dụng thực tế so với hạn mức/giá trị gói hiện tại, dùng làm cơ sở phân loại nhu cầu | BSS tính từ usage và quota theo tài nguyên chính | `175.0` |
| `primary_usage_resource` | string | ✅ | Tài nguyên chính dẫn tới phân loại nhu cầu | OCS/BSS phân loại theo mức sử dụng nổi bật nhất | `DATA` hoặc `VOICE` hoặc `BOTH` |
| `usage_need_segment` | string | ✅ | Phân khúc nhu cầu theo mức sử dụng thực tế | BSS tính theo ngưỡng PO/CVM cấu hình, hoặc CVM tự tính nếu chỉ nhận dữ liệu thô | `HIGH_NEED` hoặc `LOW_NEED` |
| `recommendation_direction` | string | ✅ | Hướng tư vấn gói cho KH | CVM/BSS mapping từ `usage_need_segment` | `UPSIZE` hoặc `DOWNSIZE` |
| `suggested_plan` | string | ❌ | Gói đề xuất phù hợp với nhu cầu: gói lớn hơn nếu `HIGH_NEED`, gói nhỏ/tiết kiệm hơn nếu `LOW_NEED` | CVM NBO ưu tiên tự tính; BSS chỉ cung cấp nếu đã có rule thống nhất | `GOI_DATA_120K` hoặc `GOI_DATA_50K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn tư vấn gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH được phân tích nhu cầu | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói đang dùng — cơ sở so sánh khi tư vấn gói phù hợp hơn | Văn bản | CSV `current_plan` | — | `GOI_DATA_70K` |
| `{{phan_khuc_nhu_cau}}` | ✅ | Phân khúc nhu cầu theo mức sử dụng thực tế để CVM phân nhánh nội dung | Văn bản | CSV `usage_need_segment` | — | `HIGH_NEED` |
| `{{loai_nhu_cau_chinh}}` | ✅ | Loại tài nguyên chính khiến KH cần đổi gói | Văn bản | CSV `primary_usage_resource` | — | `DATA` |
| `{{ty_le_su_dung_so_voi_goi}}` | ✅ | Tỷ lệ sử dụng so với gói hiện tại — bằng chứng để giải thích vì sao tư vấn đổi gói | Số | CSV `usage_to_quota_pct` | — | `175.0%` |
| `{{huong_tu_van_goi}}` | ✅ | Hướng tư vấn gói: gói lớn hơn cho nhu cầu lớn, gói nhỏ/tiết kiệm hơn cho nhu cầu nhỏ | Văn bản | CSV `recommendation_direction` | — | `UPSIZE` |
| `{{goi_de_xuat_theo_nhu_cau}}` | ❌ | Tên gói NBO đề xuất phù hợp với phân khúc nhu cầu | Văn bản | CSV `suggested_plan` hoặc CVM NBO | không hiện tên gói cụ thể | `GOI_DATA_120K` |

---

---

#### File: `no_plan_x_days_{YYYYMMDD}.csv` (E_NO_PLAN_X_DAYS)

**Mô tả:** Danh sách KH trạng thái ACTIVE nhưng không có gói cước nào đang active trong x ngày liên tiếp (x do CVM cấu hình, mặc định 7 ngày)
**Trigger bởi:** BSS (quét hàng ngày từ `resource.msisdns`)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `days_without_plan` | integer | ✅ | Số ngày không có gói cước | BSS tính từ ngày gói cuối hết hạn | `7` |
| `last_plan_name` | string | ❌ | Tên gói cuối cùng KH từng dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `last_plan_expiry_date` | date | ✅ | Ngày hết hạn gói cuối | `resource.msisdns.expiry_date` (BSS) | `2026-05-26` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS → BSS nightly batch | `15000` |
| `subscriber_tenure_days` | integer | ❌ | Số ngày KH đã dùng mạng | BSS tính từ `resource.msisdn_status_history` | `365` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH không có gói cước | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_ngay_khong_goi}}` | ✅ | Số ngày không có gói — tạo cảm giác cấp bách kích hoạt lại | Số | CSV `days_without_plan` | — | `7 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để CVM gợi ý gói phù hợp với mức tiền có sẵn | Tiền (VND) | CSV `balance` | — | `15.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn nhắc đăng ký gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{ten_goi_cu}}` | ❌ | Gói cước cuối KH từng dùng — gợi ý gia hạn lại | Văn bản | CSV `last_plan_name` | không đề cập gói cũ | `GOI_DATA_70K` |
| `{{goi_phu_hop_de_xuat}}` | ❌ | Tên gói NBO đề xuất phù hợp với số dư hiện có | Văn bản | CVM NBO dựa trên `balance` và `last_plan_name` | không hiện gợi ý | `GOI_DATA_30K` |

---

---

#### File: `segment_update_{YYYYMMDD}.csv` (E_SEGMENT_UPDATE)

**Mô tả:** Danh sách KH vừa được phân loại hoặc thay đổi phân khúc thuê bao (CHURN_RISK, GAMING, ENTERTAINMENT...)
**Trigger bởi:** BSS hoặc CVM nội bộ (batch, chạy định kỳ theo chu kỳ phân khúc)
**Thời điểm push:** 02:00–04:00 hàng ngày (hoặc theo chu kỳ tuần/tháng tùy cấu hình)

> **Lưu ý:** Nếu phân khúc được CVM tự tính nội bộ thì file này không cần BSS push — CVM tự trigger. Nếu BSS tính và push thì dùng schema dưới đây. Cần xác nhận với Tech Lead.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `new_segment` | string | ✅ | Phân khúc mới | BSS hoặc CVM nội bộ | `CHURN_RISK` hoặc `GAMING` hoặc `ENTERTAINMENT` |
| `previous_segment` | string | ❌ | Phân khúc cũ (nếu có) | BSS/CVM — lịch sử phân khúc | `NORMAL` |
| `segment_score` | float | ❌ | Điểm churn/engagement để CVM ưu tiên xử lý | BSS hoặc CVM nội bộ | `0.82` |
| `segment_date` | date | ✅ | Ngày phân khúc được cập nhật | BSS/CVM nội bộ | `2026-06-02` |
| `trigger_factors` | string | ❌ | Các yếu tố chính dẫn đến phân khúc này | BSS/CVM nội bộ — danh sách cách nhau dấu `;` | `NO_APP_30D;PLAN_CANCEL;LOW_USAGE` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa được cập nhật phân khúc | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn chăm sóc theo segment | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{phan_khuc_moi}}` | ✅ | Phân khúc mới để CVM chọn chiến lược tiếp cận phù hợp | Văn bản | CSV `new_segment` | — | `CHURN_RISK` |
| `{{goi_giu_chan_de_xuat}}` | ❌ | Tên gói/ưu đãi CVM đề xuất để giữ chân KH nguy cơ rời mạng | Văn bản | CVM NBO dựa trên `new_segment` | không hiện gợi ý | `GOI_GIAM_GIA_30PCT` |

---

---

### 2.3 APP — NearRealtime

> **Lưu ý chung NearRealtime:** Endpoint cụ thể (`POST /api/...`) sẽ do Dev định nghĩa khi implement. Tài liệu này chỉ đặc tả **payload (các trường dữ liệu cần thiết)** và **yêu cầu kỹ thuật** để Dev và BSS/OCS team thống nhất.

#### Event: ĐĂNG_NHẬP_APP_LẦN_ĐẦU (E03)

**Trigger bởi:** SuperApp
**Yêu cầu kỹ thuật:** SuperApp gọi API CVM ngay lập tức khi KH mở app lần đầu. CVM phải phản hồi nội dung banner trong vòng 1–2 giây để hiển thị tức thì trong phiên.
**Timing:** Ngay lập tức khi KH mở app lần đầu

> **Lưu ý:** Payload E03 chỉ chứa dữ liệu SuperApp có thể cung cấp tại thời điểm `first_open`. Các trường `full_name`, `current_plan`, `data_remaining` để render banner cá nhân hóa — CVM tự lấy từ cache nội bộ (đã được nạp khi E01 kích hoạt trước đó) sau khi nhận `msisdn`, không yêu cầu SuperApp push kèm.
>
> **Điều kiện chặn:** Nếu `is_guest = true` → CVM bỏ qua trigger, không gửi banner (KH chưa định danh, không có đủ dữ liệu để cá nhân hóa).

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | SuperApp — event name cố định | `FIRST_OPEN` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm mở app lần đầu | SuperApp — thời điểm `first_open` | `2026-05-14 09:15:00` |
| `device_type` | string | ✅ | Loại thiết bị | SuperApp — thiết bị KH đang dùng | `ANDROID` |
| `is_guest` | boolean | ✅ | KH là khách vãng lai (chưa đăng nhập tài khoản) — nếu `true`, CVM chặn không gửi banner | SuperApp — trạng thái đăng nhập phiên hiện tại | `false` |
| `full_name` | string(64) | ❌ | Tên KH để cá nhân hóa banner — SuperApp có thể push kèm nếu đã lấy được; nếu không, CVM tự lấy từ cache | SuperApp (nếu đã đăng nhập) hoặc CVM cache từ E01 | `Nguyễn Văn A` |
| `agency_id` | integer | ❌ | Mã đại lý KH mua SIM — để CVM hiển thị đúng thương hiệu/nhận diện đại lý trong banner | `esim_agency.*` hoặc `crm.subscribers` — cần xác nhận SuperApp có biết `agency_id` không | `1023` |
| `country_code` | string | ❌ | Mã quốc gia theo ngôn ngữ cài đặt app — để CVM chọn ngôn ngữ gửi tin | SuperApp — ngôn ngữ/quốc gia KH cài đặt trong ứng dụng | `vi` hoặc `en` |
| `os_version` | string | ❌ | Phiên bản hệ điều hành | SuperApp — hệ điều hành thiết bị | `Android 14` |
| `app_version` | string | ❌ | Phiên bản ứng dụng | SuperApp — phiên bản ứng dụng | `2.1.0` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa banner chào mừng | Văn bản | CVM cache từ E01 (hoặc SuperApp push `full_name` kèm) | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại để xác nhận đúng tài khoản trong banner | Văn bản | Payload E03 `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ❌ | Tên gói cước hiện tại để hiển thị trong banner cá nhân hóa | Văn bản | CVM cache từ E01 `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{data_con_lai}}` | ❌ | Lượng data còn lại để gợi ý nạp thêm khi thấp | Số | CVM cache từ OCS (`data_remaining`) | bỏ qua hiển thị data còn lại | `1.2 GB` |
| `{{ma_dai_ly}}` | ❌ | Mã đại lý để hiển thị logo/thương hiệu đại lý đúng | Số | Payload E03 `agency_id` | không hiện logo đại lý | `1023` |

> **Lưu ý:** `is_guest = true` → CVM chặn, không render param nào.

---

---

#### Event: MUA_DỊCH_VỤ_NỘI_DUNG_THẤT_BẠI (E_CONTENT_FAIL)

**Trigger bởi:** SuperApp → Kafka → CVM
**Yêu cầu kỹ thuật:** SuperApp push event ngay khi giao dịch mua nội dung thất bại. CVM phải gửi thông báo trong vòng 2 phút khi KH còn đang trong phiên.
**Timing:** Ngay khi giao dịch mua nội dung thất bại

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | SuperApp — event name cố định | `CONTENT_PURCHASE_FAIL` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm thất bại | SuperApp — thời điểm giao dịch thất bại | `2026-06-03 20:30:00` |
| `content_type` | string | ✅ | Loại dịch vụ nội dung | SuperApp — phân loại nội dung | `GAME` hoặc `MUSIC` hoặc `VIDEO` |
| `content_name` | string | ❌ | Tên dịch vụ/gói nội dung cụ thể | SuperApp — tên sản phẩm | `Game Premium Monthly` |
| `fail_reason` | string | ✅ | Lý do thất bại | SuperApp/OCS — phân loại lý do | `INSUFFICIENT_BALANCE` hoặc `NO_DATA_PLAN` |
| `balance` | integer | ❌ | Số dư tài khoản tại thời điểm thất bại (đồng) | OCS — qua SuperApp hoặc CVM tự look up | `3000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH mua nội dung thất bại | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{loai_noi_dung}}` | ✅ | Loại dịch vụ nội dung để CVM phân nhánh gợi ý phù hợp | Văn bản | Payload `content_type` | — | `GAME` |
| `{{ly_do_that_bai}}` | ✅ | Lý do thất bại để CVM phân nhánh nội dung giải pháp | Văn bản | Payload `fail_reason` | — | `INSUFFICIENT_BALANCE` |
| `{{ten_noi_dung}}` | ❌ | Tên dịch vụ cụ thể KH muốn mua | Văn bản | Payload `content_name` | dùng `{{loai_noi_dung}}` thay thế | `Game Premium Monthly` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất để KH có đủ điều kiện mua nội dung | Văn bản | CVM NBO dựa trên `content_type` và `fail_reason` | không hiện gợi ý | `GOI_DATA_70K` |

---

---

#### Event: ĐÁNH_GIÁ_APP (E_APP_RATING)

**Trigger bởi:** SuperApp
**Yêu cầu kỹ thuật:** SuperApp push event khi KH submit đánh giá. CVM xử lý async — không cần phản hồi realtime.
**Timing:** Ngay khi KH submit đánh giá trong app

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | SuperApp — event name cố định | `APP_RATING_SUBMITTED` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm submit đánh giá | SuperApp | `2026-06-03 21:00:00` |
| `rating_score` | integer | ✅ | Điểm đánh giá (1–5) | SuperApp — điểm KH chọn | `2` |
| `rating_category` | string | ❌ | Hạng đánh giá phân loại bởi CVM | CVM tự phân loại: `LOW`(1–2), `MID`(3), `HIGH`(4–5) | `LOW` |
| `feedback_text` | string | ❌ | Nội dung phản hồi tự do (nếu KH điền) | SuperApp — text input KH nhập | `App lag, hay bị lỗi` |
| `app_version` | string | ❌ | Phiên bản app KH đang dùng | SuperApp | `2.1.0` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa đánh giá app | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{diem_danh_gia}}` | ✅ | Điểm KH vừa chọn — để CVM phân nhánh nội dung phản hồi | Số | Payload `rating_score` | — | `2` |
| `{{hang_danh_gia}}` | ✅ | Hạng phân loại (LOW/MID/HIGH) để điều hướng nội dung CVM | Văn bản | CVM tự phân loại từ `rating_score` | — | `LOW` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa phản hồi | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

> **Lưu ý phân nhánh:** `LOW` (1–2 sao) → trigger flow chăm sóc + hỏi vấn đề cụ thể; `MID` (3 sao) → cảm ơn + gợi ý tính năng; `HIGH` (4–5 sao) → cảm ơn + gợi ý review trên Store.

---

### 2.4 APP — Batch/CSV

#### File: `milestone_D7_{YYYYMMDD}.csv` (E07)

**Mô tả:** Danh sách KH đến ngày 7 hoặc đã hoàn thành ≥ 3 nhiệm vụ
**Trigger bởi:** BSS (kết hợp task_log từ SuperApp và usage từ OCS)
**Thời điểm push:** 02:00–04:00 ngày N+7

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `activation_date` | datetime | ✅ | NGAY_0 | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-05-07 08:00:00` |
| `completed_tasks` | integer | ✅ | Số nhiệm vụ đã hoàn thành | `task_log.completed_task_count` (SuperApp → Kafka → BSS) | `4` |
| `trigger_reason` | string | ✅ | Lý do kích hoạt | BSS tổng hợp: `DAY_7` nếu đúng N+7, `TASK_3` nếu tasks ≥ 3 | `DAY_7` hoặc `TASK_3` |
| `total_data_d1_d7_mb` | float | ✅ | Tổng data D1–D7 (MB) | OCS → BSS nightly batch | `3840` |
| `total_voice_d1_d7_min` | float | ✅ | Tổng thoại D1–D7 (phút) | OCS → BSS nightly batch | `45.5` |
| `firebase_token` | string | ❌ | Firebase token (dự phòng nếu không mở app) | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |
| `contact_email` | string | ❌ | Email KH | `crm.customers.contact_email` | `nguyenvana@gmail.com` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn chúc mừng D7 | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao đến mốc D7 | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_nhiem_vu_hoan_thanh}}` | ✅ | Số nhiệm vụ KH đã hoàn thành — để vinh danh và tạo động lực | Số | CSV `completed_tasks` | — | `4` |
| `{{tong_data_7_ngay_gb}}` | ✅ | Tổng data KH đã dùng trong 7 ngày đầu để tóm tắt hành trình | Số | CSV `total_data_d1_d7_mb` → CVM convert sang GB | — | `3.8 GB` |
| `{{tong_thoai_7_ngay_phut}}` | ✅ | Tổng phút thoại KH đã dùng trong 7 ngày đầu | Số | CSV `total_voice_d1_d7_min` | — | `45.5 phút` |
| `{{ly_do_kich_hoat}}` | ❌ | Lý do trigger để CVM phân nhánh nội dung tin nhắn phù hợp | Văn bản | CSV `trigger_reason` | `"DAY_7"` | `DAY_7` |

---

---

#### File: `ngay_30_summary_{YYYYMMDD}.csv` (E11)

**Mô tả:** Tổng kết hành trình 30 ngày đầu của KH — bao gồm số liệu viễn thông (data, thoại, nạp tiền, đổi gói) và hành trình trải nghiệm App (nhiệm vụ hoàn thành, điểm tích lũy, giới thiệu bạn bè)
**Trigger bởi:** BSS (kết hợp OCS + task_log/referral_log từ SuperApp → Kafka → BSS), theo sự kiện NGAY_0 + 30
**Thời điểm push:** 02:00–04:00 của ngày NGAY_0 + 30
**Kênh gửi chính:** Banner in-app + Push notification (không phải USSD)

> **Lưu ý nguồn dữ liệu tổng hợp:** E11 là file đa nguồn — BSS phải aggregate cả dữ liệu OCS (viễn thông) lẫn SuperApp (hành trình App) vào cùng 1 file. Cơ chế tương tự E07. Nếu KH chưa cài app thì các trường App (`app_tasks_completed`, `app_points_earned`, `referral_count`) để trống — CVM fallback sang nội dung chỉ có số liệu viễn thông.
>
> **⚠️ Cần xác nhận (Q20):** BSS có aggregate được `app_tasks_completed`, `app_points_earned`, `referral_count` lũy kế 30 ngày trong batch nightly không? Hay phải SuperApp cung cấp riêng qua API? Và "giới thiệu thành công" (`referral_count`) được định nghĩa thế nào — bạn bè chỉ cần kích hoạt SIM, hay phải dùng đủ N ngày?

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `activation_date` | datetime | ✅ | NGAY_0 | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-04-14 08:00:00` |
| `total_data_gb` | float | ✅ | Tổng data N1–N30 (GB) | OCS → BSS nightly batch (tổng hợp N1–N30) | `18.5` |
| `total_voice_min` | float | ✅ | Tổng thoại N1–N30 (phút) | OCS → BSS nightly batch (tổng hợp N1–N30) | `210` |
| `topup_count` | integer | ✅ | Số lần nạp tiền N1–N30 | OCS → BSS nightly batch | `3` |
| `plan_change_count` | integer | ✅ | Số lần đổi gói N1–N30 | OCS → BSS nightly batch | `1` |
| `current_plan` | string | ✅ | Gói đang dùng tại ngày N30 | OCS → BSS nightly batch | `GOI_DATA_120K` |
| `app_tasks_completed` | integer | ❌ ⚠️ | Số nhiệm vụ App KH đã hoàn thành trong 30 ngày | `task_log.completed_task_count` (SuperApp → Kafka → BSS) — cần xác nhận BSS aggregate lũy kế được không | `5` |
| `app_points_earned` | integer | ❌ ⚠️ | Tổng điểm tích lũy trong App 30 ngày đầu | SuperApp → Kafka → BSS — cần xác nhận nguồn | `350` |
| `referral_count` | integer | ❌ ⚠️ | Số thuê bao KH đã giới thiệu thành công trong 30 ngày | SuperApp hoặc BSS referral_log — cần xác nhận định nghĩa "thành công" và có tracking không | `2` |
| `firebase_token` | string | ❌ | Firebase token để gửi Push | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |
| `contact_email` | string | ❌ | Email KH để gửi báo cáo tổng kết | `crm.customers.contact_email` | `nguyenvana@gmail.com` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin tổng kết hành trình 30 ngày | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH đến mốc 30 ngày | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{tong_data_30_ngay_gb}}` | ✅ | Tổng data tiêu thụ 30 ngày đầu — điểm nhấn số liệu viễn thông | Số | CSV `total_data_gb` | — | `18.5 GB` |
| `{{tong_thoai_30_ngay_phut}}` | ✅ | Tổng phút thoại 30 ngày đầu | Số | CSV `total_voice_min` | — | `210 phút` |
| `{{so_lan_nap_tien}}` | ✅ | Số lần nạp tiền — thể hiện mức độ gắn kết với mạng | Số | CSV `topup_count` | — | `3` |
| `{{so_lan_doi_goi}}` | ✅ | Số lần đổi gói — thể hiện KH chủ động tìm gói phù hợp | Số | CSV `plan_change_count` | — | `1` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói cước KH đang dùng tại thời điểm tổng kết | Văn bản | CSV `current_plan` | — | `GOI_DATA_120K` |
| `{{so_nhiem_vu_hoan_thanh}}` | ❌ ⚠️ | Số nhiệm vụ App đã hoàn thành — điểm nhấn hành trình trải nghiệm App | Số | CSV `app_tasks_completed` — chưa xác nhận nguồn | bỏ phần App nếu trống | `5` |
| `{{diem_tich_luy}}` | ❌ ⚠️ | Tổng điểm tích lũy trong App 30 ngày — tạo cảm giác thành tựu | Số | CSV `app_points_earned` — chưa xác nhận nguồn | bỏ phần điểm nếu trống | `350 điểm` |
| `{{so_thuê_bao_gioi_thieu}}` | ❌ ⚠️ | Số bạn bè KH đã giới thiệu dùng mạng — củng cố hành vi advocacy | Số | CSV `referral_count` — chưa xác nhận tracking | bỏ phần giới thiệu nếu trống | `2 bạn` |
| `{{email_kh}}` | ❌ | Email KH để gửi báo cáo tổng kết nếu kênh Email được bật | Văn bản | CSV `contact_email` | không gửi Email, chỉ Banner + Push | `nguyenvana@gmail.com` |

---

---

## NHÓM 3 — Gia hạn gói/dịch vụ

### 3.1 Batch/CSV

#### File: `renewal_loyalty_{YYYYMMDD}.csv` (U08)

**Mô tả:** Danh sách KH đạt mốc gia hạn liên tiếp để vinh danh trung thành, bao gồm gia hạn đúng hạn và gia hạn sớm theo rule cấu hình của CVM.
**Trigger bởi:** OCS → BSS (nightly), theo sự kiện đủ điều kiện gia hạn liên tiếp
**Thời điểm push:** 02:00–04:00 khi phát hiện KH đạt mốc gia hạn liên tiếp

> **Ghi chú hợp nhất:** `U_RENEWAL_STREAK` đã được gộp vào U08. Không tạo file `renewal_streak_{YYYYMMDD}.csv` riêng; mọi case gia hạn liên tiếp, kể cả gia hạn sớm, đi qua file `renewal_loyalty_{YYYYMMDD}.csv`.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `contact_email` | string | ❌ | Email KH | `crm.customers.contact_email` | `nguyenvana@gmail.com` |
| `consecutive_renewals` | integer | ✅ | Số chu kỳ gia hạn liên tiếp đủ điều kiện, bao gồm đúng hạn và sớm hạn theo rule CVM | BSS tính từ lịch sử `resource.msisdns.expiry_date` + OCS renewal log | `5` |
| `includes_early_renewal` | boolean | ✅ | Có ít nhất 1 lần gia hạn sớm trong chuỗi không | OCS → BSS nightly batch | `true` |
| `renewal_pattern` | string | ✅ | Kiểu chuỗi gia hạn để CVM phân nhánh nội dung | BSS phân loại từ lịch sử gia hạn | `ON_TIME_ONLY` hoặc `EARLY_ONLY` hoặc `MIXED` |
| `last_renewal_date` | date | ✅ | Ngày gia hạn lần cuối | OCS → BSS nightly batch | `2026-05-13` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_120K` |
| `plan_expiry_date` | date | ✅ | Ngày hết hạn gói hiện tại sau lần gia hạn gần nhất | `resource.msisdns.expiry_date` (BSS) | `2026-07-01` |
| `loyalty_milestone` | string | ❌ | Mốc trung thành KH vừa đạt được | CVM cấu hình theo `consecutive_renewals` | `M3` hoặc `M6` hoặc `M12` |
| `reward_type` | string | ❌ | Loại quyền lợi/ưu đãi áp dụng cho mốc gia hạn | CVM cấu hình theo milestone | `LOYALTY_POINTS` hoặc `DATA_BONUS` hoặc `RANK_UP` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin vinh danh gia hạn liên tiếp | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH đạt mốc gia hạn liên tiếp | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_lan_gia_han_lien_tiep}}` | ✅ | Số chu kỳ gia hạn liên tiếp — điểm nhấn để vinh danh | Số | CSV `consecutive_renewals` | — | `5 tháng` |
| `{{kieu_chuoi_gia_han}}` | ✅ | Kiểu chuỗi gia hạn để phân nhánh copy: đúng hạn, sớm hạn hoặc hỗn hợp | Văn bản | CSV `renewal_pattern` | — | `EARLY_ONLY` |
| `{{ngay_gia_han_cuoi}}` | ✅ | Ngày gia hạn lần cuối — xác nhận mốc trung thành gần nhất | Ngày (DD/MM/YYYY) | CSV `last_renewal_date` | — | `13/05/2026` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói cước KH đang dùng khi đạt mốc trung thành | Văn bản | CSV `current_plan` | — | `GOI_DATA_120K` |
| `{{moc_trung_thanh}}` | ❌ | Mốc trung thành KH vừa đạt để cá nhân hóa nội dung vinh danh | Văn bản | CSV `loyalty_milestone` hoặc CVM tính nội bộ | không hiển thị mốc cụ thể | `M6` |
| `{{uu_dai_trung_thanh}}` | ❌ | Ưu đãi đặc biệt dành cho KH gia hạn liên tiếp | Văn bản | CVM cấu hình theo `consecutive_renewals` / `loyalty_milestone` | không hiện ưu đãi | `Tặng 5GB data tháng sau` |
| `{{diem_trung_thanh}}` | ❌ ⚠️ | Điểm tích lũy trong chương trình loyalty — chưa xác nhận schema | Số | CVM tính nội bộ — chưa xác nhận schema module loyalty | không hiển thị điểm | `350` |
| `{{hang_trung_thanh_moi}}` | ❌ ⚠️ | Hạng loyalty KH vừa đạt được — để tạo cảm giác thành tựu | Văn bản | CVM tính nội bộ — chưa xác nhận schema | không hiển thị hạng | `Silver` |
| `{{email_kh}}` | ❌ | Email KH để gửi giấy chứng nhận trung thành qua Email | Văn bản | CSV `contact_email` | không gửi Email | `nguyenvana@gmail.com` |

---

---

#### File: `pre_expiry_{YYYYMMDD}.csv` (U_PRE_EXPIRY)

**Mô tả:** Danh sách KH còn x ngày trước khi gói hết hạn (x do CVM cấu hình, thường 3 và 7 ngày)
**Trigger bởi:** BSS (quét hàng ngày từ `resource.msisdns.expiry_date`)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `plan_expiry_date` | date | ✅ | Ngày hết hạn gói | `resource.msisdns.expiry_date` (BSS) | `2026-06-10` |
| `days_to_expiry` | integer | ✅ | Số ngày còn đến khi hết hạn | BSS tính: `expiry_date - ngay_hien_tai` | `3` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS → BSS nightly batch | `80000` |
| `data_remaining_mb` | float | ❌ | Data còn lại trong chu kỳ (MB) | OCS → BSS nightly batch | `512` |
| `auto_renewal_enabled` | boolean | ❌ | KH đã bật tự động gia hạn chưa | OCS/BSS — cài đặt tự động gia hạn | `false` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH sắp hết hạn gói | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh nhắc gia hạn | Văn bản | CSV `current_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_han}}` | ✅ | Ngày hết hạn gói — mốc KH cần hành động trước | Ngày (DD/MM/YYYY) | CSV `plan_expiry_date` | — | `10/06/2026` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trước khi hết hạn — tạo cảm giác cấp bách | Số | CSV `days_to_expiry` | — | `3 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để KH biết có đủ tiền gia hạn không | Tiền (VND) | CSV `balance` | — | `80.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn nhắc gia hạn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{data_con_lai_gb}}` | ❌ | Data còn lại để KH quyết định gia hạn ngay hay đợi | Số | CSV `data_remaining_mb` → CVM convert sang GB | không hiển thị data còn lại | `0.5 GB` |
| `{{goi_gia_han_de_xuat}}` | ❌ | Tên gói gia hạn NBO đề xuất | Văn bản | CVM NBO | không hiện gợi ý | `GOI_DATA_120K` |

---

---

#### File: `post_expiry_{YYYYMMDD}.csv` (U_POST_EXPIRY)

**Mô tả:** Danh sách KH đã hết hạn gói x ngày mà chưa gia hạn (x do CVM cấu hình, thường 1, 3, 7 ngày)
**Trigger bởi:** BSS (quét hàng ngày từ `resource.msisdns.expiry_date`)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `last_plan` | string | ✅ | Tên gói cuối cùng đã hết hạn | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `expiry_date` | date | ✅ | Ngày gói đã hết hạn | `resource.msisdns.expiry_date` (BSS) | `2026-05-31` |
| `days_since_expiry` | integer | ✅ | Số ngày đã qua hạn chưa gia hạn | BSS tính: `ngay_hien_tai - expiry_date` | `3` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS → BSS nightly batch | `5000` |
| `subscriber_status` | string | ✅ | Trạng thái thuê bao hiện tại | `crm.subscribers.status` | `ACTIVE` hoặc `GRACE` |
| `renewal_attempts` | integer | ❌ | Số lần CVM đã nhắc gia hạn trong đợt này | CVM nội bộ — đếm số lần trigger đã fire | `1` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH chưa gia hạn sau khi hết hạn | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_cu}}` | ✅ | Tên gói vừa hết hạn — để KH nhận ra và cân nhắc gia hạn lại | Văn bản | CSV `last_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_han}}` | ✅ | Ngày gói đã hết hạn — xác nhận mốc cần hành động | Ngày (DD/MM/YYYY) | CSV `expiry_date` | — | `31/05/2026` |
| `{{so_ngay_qua_han}}` | ✅ | Số ngày đã quá hạn — tạo cảm giác cấp bách | Số | CSV `days_since_expiry` | — | `3 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để CVM gợi ý gói phù hợp mức tiền có sẵn | Tiền (VND) | CSV `balance` | — | `5.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn thúc gia hạn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_gia_han_de_xuat}}` | ❌ | Tên gói gia hạn NBO đề xuất phù hợp số dư hiện có | Văn bản | CVM NBO dựa trên `balance` và `last_plan` | không hiện gợi ý | `GOI_DATA_50K` |

---

---

## NHÓM 4 — Khóa 1c/Khóa 2c

### 4.1 NearRealtime

> **Lưu ý chung NearRealtime:** Endpoint cụ thể (`POST /api/...`) sẽ do Dev định nghĩa khi implement. Tài liệu này chỉ đặc tả **payload (các trường dữ liệu cần thiết)** và **yêu cầu kỹ thuật** để Dev và BSS/OCS team thống nhất.

#### Event: KHÓA_2_CHIỀU (E_LOCK_2C)

**Trigger bởi:** OCS/BSS
**Yêu cầu kỹ thuật:** BSS/OCS gọi API CVM ngay khi thuê bao chuyển trạng thái LOCK_2C. CVM gửi USSD và Push APP để KH biết cách khôi phục.
**Timing:** Ngay khi tài khoản bị khóa 2 chiều

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `LOCK_2C` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm khóa 2 chiều | BSS — thời điểm chuyển trạng thái | `2026-06-03 00:00:00` |
| `lock_reason` | string | ✅ | Lý do khóa 2 chiều | BSS — phân loại lý do | `EXPIRY` hoặc `ADMIN` |
| `days_since_expiry` | integer | ❌ | Số ngày kể từ khi hết hạn gói | BSS tính từ `resource.msisdns.expiry_date` | `30` |
| `firebase_token` | string | ❌ | Firebase token để gửi Push | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại bị khóa 2 chiều | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ly_do_khoa}}` | ✅ | Lý do khóa để CVM phân nhánh nội dung hướng dẫn khôi phục | Văn bản | Payload `lock_reason` | — | `EXPIRY` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn khóa 2 chiều | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_ngay_qua_han}}` | ❌ | Số ngày đã quá hạn — tạo cảm giác cấp bách | Số | Payload `days_since_expiry` | không hiển thị số ngày | `30 ngày` |
| `{{huong_dan_mo_khoa}}` | ✅ | Hướng dẫn cụ thể để KH tự mở khóa | Văn bản | CVM cấu hình tĩnh theo `lock_reason` | — | `Nạp tiền và gia hạn gói tại *101#` |

---

---

### 4.2 Batch/CSV

#### File: `lock_1c_{YYYYMMDD}.csv` (E_LOCK_1C)

**Mô tả:** Danh sách KH bị khóa 1 chiều trong ngày — tách 2 kịch bản: (A) hệ thống tác động (admin lock), (B) không sử dụng quá lâu
**Trigger bởi:** BSS (quét hàng ngày)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `lock_1c_date` | date | ✅ | Ngày bị khóa 1 chiều | BSS — ngày chuyển trạng thái LOCK_1C | `2026-06-02` |
| `lock_scenario` | string | ✅ | Kịch bản khóa | BSS phân loại: `SYSTEM_ACTION` hoặc `INACTIVE` | `INACTIVE` |
| `days_inactive` | integer | ❌ | Số ngày không sử dụng (nếu kịch bản INACTIVE) | BSS tính từ CDR | `90` |
| `lock_reason_detail` | string | ❌ | Mô tả chi tiết lý do (nếu kịch bản SYSTEM_ACTION) | BSS — lý do admin lock | `FRAUD_SUSPECTED` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS → BSS nightly batch | `0` |
| `days_to_lock_2c` | integer | ❌ | Số ngày còn lại trước khi chuyển khóa 2 chiều | BSS tính theo quy định | `30` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại bị khóa 1 chiều | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{kich_ban_khoa}}` | ✅ | Kịch bản khóa để CVM phân nhánh nội dung hướng dẫn khôi phục phù hợp | Văn bản | CSV `lock_scenario` | — | `INACTIVE` |
| `{{so_ngay_con_den_khoa_2c}}` | ❌ | Số ngày còn lại trước khi khóa 2 chiều — tạo urgency | Số | CSV `days_to_lock_2c` | không hiển thị đếm ngược | `30 ngày` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn khóa 1 chiều | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{huong_dan_mo_khoa}}` | ✅ | Hướng dẫn cụ thể để KH tự mở khóa 1 chiều | Văn bản | CVM cấu hình tĩnh theo `lock_scenario` | — | `Sử dụng dịch vụ hoặc liên hệ 1800xxx` |

---

---

#### File: `pre_lock_2c_{YYYYMMDD}.csv` (E_PRE_LOCK_2C)

**Mô tả:** Danh sách KH đang ở trạng thái khóa 1 chiều và còn x ngày trước khi bị khóa 2 chiều (x do CVM cấu hình, thường 7 và 3 ngày)
**Trigger bởi:** BSS (quét hàng ngày từ trạng thái LOCK_1C + ngày khóa)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `lock_1c_date` | date | ✅ | Ngày bắt đầu khóa 1 chiều | BSS — ngày chuyển trạng thái LOCK_1C | `2026-05-03` |
| `lock_2c_scheduled_date` | date | ✅ | Ngày dự kiến khóa 2 chiều | BSS tính từ quy định (thường lock_1c_date + 30 ngày) | `2026-07-02` |
| `days_to_lock_2c` | integer | ✅ | Số ngày còn lại trước khóa 2 chiều | BSS tính: `lock_2c_scheduled_date - ngay_hien_tai` | `7` |
| `lock_scenario` | string | ✅ | Kịch bản khóa 1c ban đầu | BSS — `SYSTEM_ACTION` hoặc `INACTIVE` | `INACTIVE` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS → BSS nightly batch | `0` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại sắp bị khóa 2 chiều | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn trước khi khóa 2 chiều — điểm nhấn urgency | Số | CSV `days_to_lock_2c` | — | `7 ngày` |
| `{{ngay_khoa_2c_du_kien}}` | ✅ | Ngày dự kiến khóa 2 chiều — để KH nắm deadline | Ngày (DD/MM/YYYY) | CSV `lock_2c_scheduled_date` | — | `02/07/2026` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo khẩn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{huong_dan_mo_khoa}}` | ✅ | Hướng dẫn cụ thể để mở khóa trước deadline | Văn bản | CVM cấu hình tĩnh theo `lock_scenario` | — | `Đăng ký gói cước tại *101# hoặc ứng dụng` |

---

## PHẦN 3 — Open Questions cần xác nhận trước khi implement

| # | Câu hỏi | Liên quan | Mức độ |
|---|---|---|---|
| Q1 | SFTP path và cấu trúc thư mục cụ thể mà CVM sẽ đọc file? | Tất cả file CSV | 🔴 Cần ngay |
| Q2 | CVM pull file theo lịch hay BSS notify sau khi push xong? | Tất cả file CSV | 🔴 Cần ngay |
| Q3 | Xử lý lỗi: file CSV thiếu cột bắt buộc → CVM làm gì? | Tất cả file CSV | 🔴 Cần ngay |
| Q4 | API endpoint của CVM để BSS/OCS gọi push event — ai define? | Tất cả Push Event | 🔴 Cần ngay |
| Q5 | BSS có ghi `inbound_sms_log` (SMS đến từ bên ngoài) không? | U04 | 🔴 Cần ngay |
| Q6 | `gender`, `age_segment`, `job_segment` KH lấy từ đâu trong BSS? | E01–E04, U09, U10 | 🔴 Cần ngay |
| Q7 | Trường `birthday` trong `crm.customers` có tồn tại không? | U09 | 🟡 Quan trọng |
| Q8 | OCS cung cấp dữ liệu lưu lượng cho BSS theo cơ chế nào (API hay file)? Tần suất? | E05, E07, E11, U05 | 🟡 Quan trọng |
| Q9 | SuperApp push sự kiện `first_open`, `change_pkg_view` vào BSS qua Kafka hay API trực tiếp? | E03, E09 | 🟡 Quan trọng |
| Q10 | File CSV có cần nén (gzip) để tối ưu dung lượng không? | Tất cả file CSV lớn (U10) | 🟢 Nice-to-have |
| Q11 | OCS có push event riêng khi quota thoại về 0, hay chỉ ghi trong CDR? Nếu chỉ có CDR thì cần BSS xử lý batch thay vì NearRealtime | E_VOICE_100 | 🔴 Cần ngay |
| Q12 | E_ZERO_BALANCE: OCS push event khi balance về 0 realtime hay BSS phát hiện qua CDR hàng đêm? Nếu batch thì SLA "proactive" giảm — cần cân nhắc | E_ZERO_BALANCE | 🔴 Cần ngay |
| Q13 | E_CANCEL_PLAN: OCS có phân biệt được KH chủ động hủy vs hệ thống tự hủy (hết hạn) không? Cần tách 2 trigger hay dùng `cancel_reason` để phân nhánh? | E_CANCEL_PLAN | 🟡 Quan trọng |
| Q14 | E_SEGMENT_UPDATE: Phân khúc (CHURN_RISK, GAMING, ENTERTAINMENT...) do BSS tính hay CVM tính nội bộ? Nếu CVM tính thì file 2.18 không cần — CVM tự trigger | E_SEGMENT_UPDATE | 🔴 Cần ngay |
| Q15 | E_CONTENT_FAIL: SuperApp có thể push event mua nội dung thất bại trực tiếp vào CVM không, hay phải qua Kafka → BSS → CSV daily? Nếu qua BSS thì mất tính realtime | E_CONTENT_FAIL | 🔴 Cần ngay |
| Q16 | E_LOCK_1C / E_PRE_LOCK_2C / E_LOCK_2C: BSS có sẵn trường ghi thời gian dự kiến chuyển trạng thái không? Hay CVM phải tự tính từ quy định nghiệp vụ? | E_LOCK_1C, E_PRE_LOCK_2C, E_LOCK_2C | 🟡 Quan trọng |
| Q17 | Rule chặn E_ZERO_BALANCE vs E06: window 12h có đủ không, hay nên dùng window 24h? Ai owns logic chặn — CVM hay BSS? | E_ZERO_BALANCE, E06 | 🟡 Quan trọng |
| Q18 | U08: Sau khi gộp `U_RENEWAL_STREAK` vào U08, ngưỡng `consecutive_renewals` theo từng milestone là bao nhiêu và gia hạn sớm được tính như thế nào trong chuỗi đủ điều kiện? | U08 | 🟡 Quan trọng |
| Q19 | E_VOICE_100: OCS có tách quota thoại nội mạng và ngoại mạng riêng không? Nếu gói gộp chung 1 quota thì 2 event ONNET/OFFNET dùng chung 1 event với trường `call_type = BOTH` | E_VOICE_100_ONNET, E_VOICE_100_OFFNET | 🔴 Cần ngay |
| Q20 | E11 — hành trình App: BSS có aggregate `app_tasks_completed`, `app_points_earned`, `referral_count` trong batch nightly 30 ngày không, hay phải SuperApp cung cấp riêng theo API? | E11 | 🟡 Quan trọng |
| Q21a | U05-B: OCS có event riêng khi quota data/ngày về 0 không (giống E08 nhưng 100%)? Nếu có thì BSS đếm được số ngày hết/tháng ngay. Nếu không thì BSS phải tính từ CDR — cần xác nhận compute có khả thi không | U05-B | 🔴 Cần ngay |
| Q21b | U05-B: Ngưỡng N_DAYS_THRESHOLD (số ngày hết/tháng) và M_MONTHS_THRESHOLD (số tháng liên tiếp) là bao nhiêu? Assumption hiện tại: 15 ngày và 2 tháng | U05-B | 🟡 Quan trọng |
| Q21c | U05-B: UPSELL_MODE mặc định là gì — chỉ gợi ý nâng gói ngày (DAILY_UPGRADE), chuyển hẳn sang gói tháng (MONTHLY_UPSELL), hay đề xuất cả 2 (BOTH)? Assumption hiện tại: BOTH | U05-B | 🟡 Quan trọng |
| Q21d | U05-A + U05-B: Ngưỡng cut-off ngày 25 (U05-A) và N_DAYS_THRESHOLD (U05-B) do CVM cấu hình — cơ chế truyền ngưỡng từ CVM sang BSS là gì? Config file, API notify, hay thủ công? | U05-A, U05-B | 🟡 Quan trọng |
| Q22 | E_USAGE_NEED_ANALYSIS: Ngưỡng phân loại `HIGH_NEED` / `LOW_NEED` là bao nhiêu theo từng loại gói và ai là owner tính phân khúc nhu cầu — BSS tính sẵn trong file hay CVM tự tính từ dữ liệu usage/quota? | E_USAGE_NEED_ANALYSIS | 🔴 Cần ngay |
