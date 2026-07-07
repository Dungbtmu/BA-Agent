# Data Contract Template — BSS → CVM — Theo nhóm vòng đời

> Tài liệu đặc tả schema các file CSV và API event mà BSS/OCS/SuperApp cung cấp cho CVM.
> Mục đích: làm tài liệu thống nhất giữa BSS team và CVM team trước khi implement.
>
> Cập nhật: 2026-06-03 — Tái cấu trúc theo sơ đồ vòng đời CVM: Nhóm vòng đời → Nhánh nghiệp vụ VIỄN THÔNG/APP → Kỹ thuật NearRealtime/Batch/CSV; giữ nguyên payload/schema từ bản gốc.
> Trạng thái: **DRAFT — cần xác nhận với Tech Lead / SA**

**Quy ước chung:**
- Encoding: UTF-8
- Delimiter CSV: dấu phẩy (`,`)
- Header row: có (dòng đầu tiên là tên cột)
- Datetime format: `YYYY-MM-DD HH:MM:SS` (UTC+7)
- Tên file: `{tên_file}_{YYYYMMDD}.csv`
- Cơ chế delivery: BSS push vào SFTP/thư mục CVM đọc
- Giá trị rỗng: để trống (không dùng NULL hay 0 thay thế)

---

## PHẦN 1 — Tổng quan nhóm vòng đời, nhánh nghiệp vụ và kênh kỹ thuật

Tài liệu được tổ chức lại theo sơ đồ vòng đời CVM. Mỗi nhóm vòng đời được chia theo nhánh nghiệp vụ trước (`VIỄN THÔNG`, `APP`, hoặc nhóm nghiệp vụ tương ứng), sau đó mới tách theo kênh kỹ thuật `NearRealtime` và `Batch/CSV` để đội Dev/BSS/OCS/SuperApp dễ xác định cơ chế tích hợp.

| Nhóm vòng đời | Nhánh nghiệp vụ | NearRealtime | Batch/CSV |
|---|---|---|---|
| NHÓM 1 — Kích hoạt | VIỄN THÔNG | E01 | Không có |
| NHÓM 1 — Kích hoạt | APP | E02 | Không có |
| NHÓM 2 — Sử dụng | VIỄN THÔNG | U01, U03, E08, E_DATA_100, E_VOICE_100, E06, E_ZERO_BALANCE, E_CANCEL_PLAN, E05, E_CHURN_RISK, U05-B-RT | U04, E13, E_USAGE_NEED_ANALYSIS, U09, U10, E_SEGMENT_UPDATE, E_NO_PLAN_X_DAYS, E11, U02, U05-A, U05-B, U06, U07 |
| NHÓM 2 — Sử dụng | APP | E03, E_CONTENT_FAIL, E_APP_RATING, E04, E09 | E07, E_APP_INACTIVE_X_DAYS |
| NHÓM 3 — Gia hạn gói/dịch vụ | GIA HẠN | U_PRE_EXPIRY, U_POST_EXPIRY | U08 |
| NHÓM 4 — Khóa 1c/Khóa 2c | KHÓA 1C/2C | E_LOCK_2C, E_LOCK_1C, E_PRE_LOCK_2C | Không có |

> **Bổ sung theo xác nhận BA:** `E_DATA_100` được thêm cho trigger hết 100% data theo NearRealtime; `E_APP_INACTIVE_X_DAYS` được thêm cho trigger X ngày KH không truy cập APP theo Batch/CSV. `E_ZERO_BALANCE` được chuẩn hóa thành API NearRealtime.
>
> **Cập nhật 2026-07-03 — chốt theo bảng trigger đầy đủ (cột "Loại xử lý"):**
> - Chuyển sang **NearRealtime**: `E02` (chưa cài app 24h), `E04` (chưa mở app 24h), `E05` (chưa phát sinh cước 72h), `E09` (hành trình mua SIM/gói bỏ dở), `U_PRE_EXPIRY`, `U_POST_EXPIRY`, `E_LOCK_1C`, `E_PRE_LOCK_2C`.
> - Thêm trigger **`E_CHURN_RISK`** — cảnh báo thuê bao có nguy cơ rời mạng (không lưu lượng 5–7 ngày, không cước 30 ngày, doanh thu giảm ≥80% so với trung bình 2 tháng, tỷ lệ bật/tắt sóng SIM).
> - Tách **`U05`** thành **`U05-A`** (gói data tháng) và **`U05-B`** (pattern hết quota data ngày, Batch) để đồng bộ với `data-contract-template.md`. Bổ sung thêm **`U05-B-RT`** — trigger NearRealtime riêng biệt khi KH hết quota ngày/tuần 3 lần liên tiếp (không gộp chung schema với U05-B batch).

---

## PHẦN 2 — NHÓM 1 — Kích hoạt

### Nhánh nghiệp vụ — VIỄN THÔNG

**Đối chiếu mục trong ảnh:**
- Kích hoạt SIM/SIM Bundle đăng ký gói thành công

#### Kỹ thuật — NearRealtime

##### 1.1 Event: SIM_KÍCH_HOẠT (E01)

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

#### Kỹ thuật — Batch/CSV

Không có file Batch/CSV trong phạm vi hiện tại.

---

### Nhánh nghiệp vụ — APP

**Đối chiếu mục trong ảnh:**
- Mời cài đặt APP

> **Ghi chú:** `E02` là luồng mời cài APP sau 24h chưa cài, đang được đặt ở nhóm Kích hoạt để bám mapping đã chốt trước đó.

#### Kỹ thuật — NearRealtime

##### 2.1 Event: CHƯA_CÀI_APP_SAU_24H (E02)

**Mô tả:** KH kích hoạt SIM nhưng chưa cài app sau 24h tính từ thời điểm E01
**Trigger bởi:** BSS
**Yêu cầu kỹ thuật:** BSS gọi API CVM ngay khi phát hiện đủ 24h từ E01 mà chưa có bản ghi cài app. CVM gửi USSD/tin nhắn nhắc cài app trong vòng vài phút sau khi nhận event.
**Timing:** Ngay khi đủ 24h tính từ E01 mà chưa cài app

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV sang **NearRealtime** theo bảng trigger đã chốt. Payload dưới đây thay cho file CSV `no_app_install_D1` trước đây.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `NO_APP_INSTALL_24H` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện đủ 24h chưa cài app | BSS — thời điểm quét đủ điều kiện | `2026-05-14 08:30:00` |
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

## PHẦN 3 — NHÓM 2 — Sử dụng

### Nhánh nghiệp vụ — VIỄN THÔNG

**Đối chiếu mục trong ảnh:**
- Nạp tiền thành công
- *101#
- Hết x% Data
- Hết 100% Data
- Hết phút thoại trong gói
- Nhận OTP bên thứ 3
- Cuộc gọi thất bại
- Lưu lượng tăng đột biến x lần so với cùng kỳ
- Nhu cầu KH lớn hơn gói đăng ký
- Hết tiền TKC
- Sinh nhật/kỷ niệm/ngày lễ
- Phân loại tập thuê bao
- Hủy gói cước
- Thuê bao x ngày không có gói sử dụng

#### Kỹ thuật — NearRealtime

##### 1.5 Event: NẠP_TIỀN_THÀNH_CÔNG (U01)

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

##### 1.6 Event: TRA_CỨU_SỐ_DƯ (U03)

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

##### 1.3 Event: DATA_NGÀY_SẮP_HẾT (E08)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi lưu lượng data trong ngày vượt ngưỡng x% hạn mức (đề xuất **90%**). CVM phải xử lý và gửi USSD trong vòng 2 giờ sau khi nhận event.
**Timing:** Ngay khi vượt ngưỡng x% (đề xuất 90%)

> **Rule tần suất gửi (2026-07-03):** Mỗi ngày CVM chỉ gửi **1 lần** khi KH chạm ngưỡng x%. Nếu KH lặp lại pattern chạm ngưỡng nhiều ngày, CVM đếm số lần: **đến lần thứ 3** (bắt trigger lần 3) mới chuyển sang đề xuất gói khác (nâng gói/đổi gói) thay vì chỉ nhắc mua thêm data ngày. Ngưỡng x% (90%) và số lần (3) do CVM cấu hình.

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
| `threshold_pct` | integer | ✅ | Ngưỡng cảnh báo áp dụng (do CVM cấu hình, đề xuất 90) | CVM cấu hình | `90` |
| `depleted_trigger_count` | integer | ✅ | Số ngày liên tiếp KH chạm ngưỡng — đến lần thứ 3 CVM chuyển sang đề xuất đổi/nâng gói | OCS/CVM đếm theo chu kỳ | `1` |
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

##### Event: HẾT_100_PHẦN_TRĂM_DATA (E_DATA_100)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi quota data trong chu kỳ/gói về 0. CVM phải xử lý NearRealtime để gửi thông báo/gợi ý mua gói bổ sung trong vòng 2 phút sau khi nhận event.
**Timing:** Ngay khi data còn lại của gói về 0

> **Phân biệt với E08 và E_USAGE_NEED_ANALYSIS:** `E08` là cảnh báo khi dùng tới ngưỡng x%/80% data. `E_DATA_100` là sự kiện hết 100% data tại thời điểm hiện tại. `E_USAGE_NEED_ANALYSIS` là batch phân tích nhu cầu gói theo mức sử dụng qua nhiều chu kỳ để tư vấn gói lớn hơn hoặc nhỏ hơn.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `DATA_QUOTA_100` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm data về 0 | OCS — thời điểm quota data còn lại bằng 0 | `2026-06-03 15:30:00` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_DATA_70K` |
| `data_quota_mb` | integer | ✅ | Tổng quota data của gói (MB) | OCS — hạn mức data theo gói | `10240` |
| `data_used_mb` | integer | ✅ | Data đã dùng trong chu kỳ/gói (MB) | OCS — lưu lượng sử dụng thực tế | `10240` |
| `data_remaining_mb` | integer | ✅ | Data còn lại (MB), phải bằng 0 tại thời điểm trigger | OCS — quota data còn lại | `0` |
| `days_remaining` | integer | ✅ | Số ngày còn lại trong chu kỳ gói | OCS — số ngày còn lại | `8` |
| `balance` | integer | ❌ | Số dư tài khoản hiện tại (đồng) | OCS — số dư realtime | `25000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao hết 100% data | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh gợi ý mua thêm data | Văn bản | Payload `current_plan` | — | `GOI_DATA_70K` |
| `{{quota_data_mb}}` | ✅ | Tổng quota data của gói để KH hiểu đã dùng hết bao nhiêu | Số | Payload `data_quota_mb` | — | `10240 MB` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trong chu kỳ — tạo cảm giác cấp bách | Số | Payload `days_remaining` | — | `8 ngày` |
| `{{so_du_tai_khoan}}` | ❌ | Số dư tài khoản để KH biết có đủ tiền mua gói bổ sung không | Tiền (VND) | Payload `balance` | không hiển thị số dư | `25.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_data_bo_sung}}` | ❌ | Tên gói bổ sung data NBO đề xuất | Văn bản | CVM NBO | không hiện gợi ý | `GOI_DATA_NGAY_5K` |

---

##### 1.7 Event: HẾT_PHÚT_THOẠI (E_VOICE_100)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS gọi API CVM ngay khi quota thoại trong gói về 0. CVM phải gửi USSD trong vòng 2 phút để KH thấy ngay sau cuộc gọi cuối.
**Timing:** Ngay khi hết 100% quota thoại trong gói

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `VOICE_QUOTA_100` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm hết quota thoại | OCS — thời điểm quota về 0 | `2026-06-03 14:22:00` |
| `current_plan` | string | ✅ | Tên gói đang dùng | OCS — tên gói đang active | `GOI_THOAI_50K` |
| `voice_quota_min` | integer | ✅ | Tổng quota thoại của gói (phút) | OCS — theo định nghĩa gói | `300` |
| `days_remaining` | integer | ✅ | Số ngày còn lại trong chu kỳ gói | OCS — số ngày còn lại | `8` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS — số dư realtime | `25000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại thuê bao hết quota thoại | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi}}` | ✅ | Tên gói đang dùng để làm ngữ cảnh gợi ý gói mới | Văn bản | Payload `current_plan` | — | `GOI_THOAI_50K` |
| `{{quota_thoai_phut}}` | ✅ | Tổng quota thoại của gói — để KH hiểu đã dùng hết bao nhiêu | Số | Payload `voice_quota_min` | — | `300 phút` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn lại trong chu kỳ — tạo cảm giác cấp bách | Số | Payload `days_remaining` | — | `8 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để KH biết có đủ tiền mua gói bổ sung không | Tiền (VND) | Payload `balance` | — | `25.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_thoai_bo_sung}}` | ❌ | Tên gói bổ sung thoại NBO đề xuất | Văn bản | CVM NBO | không hiện gợi ý | `GOI_THOAI_NGAY_5K` |

---

##### 1.4 Event: CUỘC_GỌI_THẤT_BẠI (E06)

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

##### Event: SỐ_DƯ_TKC_VỀ_0 (E_ZERO_BALANCE)

**Trigger bởi:** OCS
**Yêu cầu kỹ thuật:** OCS push API NearRealtime sang CVM ngay khi số dư TKC về 0 do bất kỳ giao dịch nào. CVM xử lý proactive để gửi thông báo/gợi ý nạp tiền trước khi KH phát sinh lỗi cuộc gọi hoặc lỗi mua dịch vụ.
**Timing:** Ngay khi số dư TKC về 0

> **Phân biệt với E06:** E06 trigger khi cuộc gọi đã thất bại (reactive); `E_ZERO_BALANCE` trigger khi balance về 0 do bất kỳ giao dịch nào (proactive). Hai trigger có thể fire gần nhau — cần rule chặn: nếu `E_ZERO_BALANCE` đã fire trong vòng 12h thì E06 không gửi thêm tin nhắn nạp tiền.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `ZERO_BALANCE` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm số dư TKC về 0 | OCS — thời điểm giao dịch làm cạn tài khoản | `2026-06-03 23:45:00` |
| `last_transaction_type` | string | ✅ | Loại giao dịch cuối làm hết tiền | OCS — phân loại giao dịch | `VOICE_CALL` hoặc `DATA_USAGE` hoặc `PLAN_REGISTER` |
| `balance` | integer | ✅ | Số dư hiện tại, phải bằng 0 tại thời điểm trigger | OCS — số dư realtime | `0` |
| `current_plan` | string | ❌ | Gói đang dùng | OCS — tên gói đang active | `GOI_DATA_70K` |
| `plan_expiry_date` | date | ❌ | Ngày hết hạn gói | `resource.msisdns.expiry_date` (BSS) hoặc OCS | `2026-06-14` |
| `topup_count_30d` | integer | ❌ | Số lần nạp trong 30 ngày gần nhất | OCS — lịch sử giao dịch nạp tiền | `2` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa hết tiền | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{ten_goi_hien_tai}}` | ❌ | Gói đang dùng để kèm gợi ý nạp đúng mức | Văn bản | Payload `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất kèm mức nạp tối thiểu để dùng tiếp | Văn bản | CVM NBO | không hiện gợi ý | `GOI_DATA_70K` |

---

##### 1.8 Event: HỦY_GÓI_CƯỚC (E_CANCEL_PLAN)

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

##### Event: THUÊ_BAO_NGUY_CƠ_RỜI_MẠNG (E_CHURN_RISK)

**Mô tả:** KH có dấu hiệu nguy cơ rời mạng, xác định qua tổ hợp tín hiệu hành vi: không phát sinh lưu lượng liên tiếp 5–7 ngày, không phát sinh cước trong 30 ngày, doanh thu suy giảm ≥80% so với trung bình 2 tháng gần nhất, tỷ lệ bật/tắt sóng SIM bất thường.
**Trigger bởi:** OCS/BSS gọi API CVM (phân tích tổ hợp tín hiệu nhiều ngày)
**Yêu cầu kỹ thuật:** BSS/OCS phân tích tổ hợp tín hiệu theo chu kỳ (nightly), khi KH vượt ngưỡng nguy cơ rời mạng thì gọi API CVM. CVM đưa vào luồng giữ chân (retention). Các ngưỡng (số ngày không lưu lượng, % suy giảm doanh thu) do CVM cấu hình, BSS/OCS không hardcode.
**Timing:** Ngay khi phát hiện KH vượt ngưỡng nguy cơ rời mạng (theo chu kỳ phân tích)

> **Phân biệt với E_SEGMENT_UPDATE:** `E_CHURN_RISK` là trigger phát hiện nguy cơ rời mạng dựa trên các tiêu chí hành vi cụ thể ở đây. `E_SEGMENT_UPDATE` là sự kiện cập nhật phân khúc tổng hợp (bao gồm cả CHURN_RISK) sau khi hệ thống đã tính điểm phân khúc — có thể dùng E_CHURN_RISK làm một tín hiệu đầu vào cho phân khúc. Cần xác nhận với Tech Lead xem tính churn ở BSS/OCS hay CVM nội bộ (Q23).

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS/OCS — event name cố định | `CHURN_RISK_DETECTED` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện nguy cơ rời mạng | BSS/OCS — thời điểm phân tích đủ điều kiện | `2026-06-03 03:00:00` |
| `no_usage_days` | integer | ✅ | Số ngày liên tiếp không phát sinh lưu lượng (data/thoại/SMS) | OCS/BSS tính từ CDR | `6` |
| `no_charge_days` | integer | ✅ | Số ngày không phát sinh cước | OCS/BSS tính từ CDR/giao dịch | `30` |
| `revenue_current` | integer | ❌ | Doanh thu kỳ hiện tại (đồng) | OCS/BSS tổng hợp | `10000` |
| `revenue_avg_2months` | integer | ❌ | Doanh thu trung bình 2 tháng gần nhất (đồng) | OCS/BSS tổng hợp | `80000` |
| `revenue_drop_pct` | float | ✅ | % suy giảm doanh thu so với trung bình 2 tháng gần nhất | BSS tính: `(revenue_avg_2months - revenue_current) / revenue_avg_2months × 100` | `87.5` |
| `sim_on_off_ratio` | float | ❌ | Tỷ lệ bật/tắt sóng của SIM trong kỳ (dấu hiệu SIM ít cắm máy) | HLR/BSS — tỷ lệ thời gian tắt sóng | `0.65` |
| `churn_risk_level` | string | ❌ | Mức độ nguy cơ để CVM ưu tiên xử lý | BSS/CVM phân loại theo tổ hợp tín hiệu | `HIGH` hoặc `MEDIUM` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `age_segment` | string | ❌ | Tuổi/phân khúc tuổi KH | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `25-34` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `540` |
| `current_plan` | string | ❌ | Gói đang dùng (nếu còn) | OCS — tên gói đang active | `GOI_DATA_70K` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi giữ chân áp dụng (nếu có) | OCS/CVM — chương trình KM đang chạy | `KM_GIUCHAN_30PCT` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH có nguy cơ rời mạng | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn giữ chân | Văn bản | Payload `full_name` hoặc CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_ngay_khong_dung}}` | ❌ | Số ngày không phát sinh lưu lượng — làm ngữ cảnh nhắc | Số | Payload `no_usage_days` | không hiển thị | `6 ngày` |
| `{{muc_do_nguy_co}}` | ❌ | Mức độ nguy cơ để CVM chọn cường độ chiến dịch giữ chân | Văn bản | Payload `churn_risk_level` | dùng mức mặc định | `HIGH` |
| `{{chuong_trinh_km}}` | ❌ | Chương trình khuyến mãi giữ chân để tăng động lực ở lại | Văn bản | Payload `promotion_code` | không hiện KM | `Giảm 30% khi gia hạn` |
| `{{goi_giu_chan_de_xuat}}` | ❌ | Tên gói/ưu đãi CVM đề xuất để giữ chân KH | Văn bản | CVM NBO dựa trên hành vi | không hiện gợi ý | `GOI_GIAM_GIA_30PCT` |

---

##### 2.3 Event: CHƯA_PHÁT_SINH_CƯỚC_72H (E05)

**Mô tả:** KH chưa phát sinh cước sau 72 giờ tính từ thời điểm kích hoạt thành công, tức tổng data + thoại + SMS = 0 trong 3 ngày. Phân ra 2 trường hợp: (1) **chưa mua gói** nào, (2) **đã mua gói nhưng chưa phát sinh lưu lượng**.
**Trigger bởi:** OCS → BSS gọi API CVM
**Yêu cầu kỹ thuật:** BSS/OCS gọi API CVM ngay khi phát hiện đủ 72h từ kích hoạt mà lưu lượng vẫn = 0. CVM gửi gợi ý (tooltip trong app hoặc tin nhắn ngắn) để tránh gây phiền.
**Timing:** Ngay khi đủ 72h tính từ thời điểm kích hoạt mà chưa phát sinh cước

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `zero_usage_D3` sang **NearRealtime** theo bảng trigger đã chốt. Bổ sung `usage_scenario` để phân biệt 2 trường hợp chưa mua gói / đã mua chưa dùng.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS/OCS — event name cố định | `NO_USAGE_72H` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện đủ 72h chưa phát sinh cước | BSS/OCS — thời điểm đủ điều kiện | `2026-05-14 09:00:00` |
| `activation_date` | datetime | ✅ | NGAY_0 | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-05-11 09:00:00` |
| `usage_scenario` | string | ✅ | Phân loại: chưa mua gói hay đã mua gói nhưng chưa dùng | BSS/OCS — kiểm tra tồn tại gói active | `NO_PLAN` hoặc `HAS_PLAN_NO_USAGE` |
| `current_plan` | string | ❌ | Gói đang có (nếu `usage_scenario = HAS_PLAN_NO_USAGE`) | OCS — tên gói đang active | `GOI_DATA_70K` |
| `data_usage_d1_mb` | float | ✅ | Lưu lượng data ngày 1 (MB) | OCS — lưu lượng realtime | `0` |
| `data_usage_d2_mb` | float | ✅ | Lưu lượng data ngày 2 (MB) | OCS — lưu lượng realtime | `0` |
| `data_usage_d3_mb` | float | ✅ | Lưu lượng data ngày 3 (MB) | OCS — lưu lượng realtime | `0` |
| `voice_usage_d3_min` | float | ✅ | Lưu lượng thoại ngày 3 (phút) | OCS — lưu lượng realtime | `0` |
| `sms_count_d3` | integer | ✅ | Số SMS ngày 3 | OCS — lưu lượng realtime | `0` |
| `device_type` | string | ❌ | Loại thiết bị | `app_install_log.device_type` (SuperApp → Kafka → BSS) | `ANDROID` |
| `has_app` | boolean | ❌ | Đã cài app chưa | Kiểm tra tồn tại bản ghi trong `app_install_log` theo `msisdn` | `true` |
| `apn_configured` | boolean | ❌ | Cài đặt điểm truy cập APN nếu có | OCS/BSS hoặc nguồn cấu hình thiết bị — cần xác nhận | `true` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn kích hoạt dùng mạng | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH chưa phát sinh cước sau 72h | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{truong_hop_su_dung}}` | ✅ | Phân nhánh nội dung: chưa mua gói (gợi ý mua gói) hoặc đã mua chưa dùng (hướng dẫn dùng) | Văn bản | Payload `usage_scenario` | — | `NO_PLAN` |
| `{{so_ngay_chua_dung}}` | ✅ | Số ngày từ khi kích hoạt đến nay KH chưa dùng data/thoại/SMS | Số | CVM tính = `ngay_hien_tai - activation_date` | — | `3 ngày` |
| `{{loai_thiet_bi}}` | ❌ | Loại thiết bị để gợi ý hướng dẫn cài đặt phù hợp | Văn bản | Payload `device_type` | không đề cập thiết bị | `IOS` |
| `{{huong_dan_su_dung}}` | ❌ | Hướng dẫn ngắn để KH bắt đầu dùng dịch vụ hoặc kiểm tra APN | Văn bản | CVM cấu hình tĩnh theo `device_type`/`apn_configured` | nội dung hướng dẫn mặc định | `Mở dữ liệu di động và kiểm tra APN` |

---

#### Kỹ thuật — Batch/CSV

##### 2.9 File: `otp_detection_{YYYYMMDD}.csv` (U04)

**Mô tả:** Danh sách KH nhận SMS OTP từ app bên thứ 3 (ngân hàng, giải trí, mạng xã hội)
**Trigger bởi:** HLR (Home Location Register — Hệ thống Đăng ký vị trí thuê bao)
**Cơ chế:** HLR phát hiện SMS OTP → đẩy qua Kafka → export CSV trực tiếp vào SFTP CVM (không đi qua BSS)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Phạm vi áp dụng (2026-07-03):** Chỉ áp dụng cho KH đang dùng **gói data tháng (MONTHLY)** — CVM lọc theo `current_plan` có chu kỳ tháng trước khi gửi gợi ý. KH dùng gói ngày/tuần hoặc không có gói data tháng thì bỏ qua.

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

##### 2.6 File: `traffic_spike_{YYYYMMDD}.csv` (E13)

**Mô tả:** Danh sách KH có lưu lượng tăng đột biến x lần so với cùng kỳ (đề xuất ngưỡng **3 lần**)
**Trigger bởi:** OCS → BSS (nightly) + CDR (baseline)
**Thời điểm push:** 02:00–04:00 hàng ngày (hoặc cuối tuần nếu đánh giá theo tuần)

> **Chu kỳ đánh giá (2026-07-03):** Ngưỡng đột biến đề xuất **x = 3 lần** so với cùng kỳ. Đề xuất cho phép chọn **chu kỳ đánh giá theo ngày hoặc theo tuần** (`spike_period`): so sánh lưu lượng ngày với baseline ngày cùng khung giờ, hoặc so sánh tổng lưu lượng tuần với baseline tuần. Ngưỡng x và chu kỳ do CVM cấu hình.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `spike_timestamp` | datetime | ✅ | Thời điểm phát hiện tăng đột biến | OCS → BSS nightly batch | `2026-05-13 23:00:00` |
| `spike_period` | string | ✅ | Chu kỳ đánh giá đột biến | CVM cấu hình | `DAILY` hoặc `WEEKLY` |
| `spike_hour` | integer | ❌ | Giờ xảy ra (0–23) — chỉ áp dụng khi `spike_period = DAILY` | OCS → BSS nightly batch | `22` |
| `traffic_spike_mb` | float | ✅ | Lưu lượng kỳ đột biến (MB) | OCS → BSS nightly batch | `512` |
| `baseline_mb` | float | ✅ | Mức nền cùng kỳ (MB) | BSS tính từ `cdr.*` lịch sử cùng kỳ | `150` |
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

##### 2.16 File: `usage_need_analysis_{YYYYMMDD}.csv` (E_USAGE_NEED_ANALYSIS)

**Mô tả:** Danh sách KH cần tư vấn gói cước theo nhu cầu sử dụng thực tế. CVM/BSS phân tích tập thuê bao trong **2 tháng liên tiếp** để xác định **ai cần nâng gói** (nhu cầu lớn hơn gói đang dùng) và **ai cần hạ gói** (nhu cầu nhỏ hơn, cần gói tiết kiệm hơn).
**Trigger bởi:** OCS → BSS (nightly batch, phân tích mức sử dụng 2 tháng liên tiếp)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Chốt cửa sổ phân tích (2026-07-03):** Phân tích tập thuê bao trong **2 tháng liên tiếp** (nhất quán với U05-A). Bổ sung trường thông tin KH và kênh/hình thức đăng ký theo cột Mô tả trong bảng trigger.
>
> **Rà soát bổ sung (2026-07-07):** Đối chiếu đủ 13 trường theo yêu cầu: Họ tên, Giới tính, Tuổi KH (qua `date_of_birth`, CVM tự tính tuổi), SĐT, Tuổi thuê bao, Tên gói, Giá gói, Chu kỳ gói, **Ngày hết hạn gói + số ngày tính từ lúc hết hạn** (mới bổ sung — trước đó thiếu), Chương trình KM, Gói gợi ý, Kênh đăng ký, Hình thức đăng ký.

> **Rule phân loại nhu cầu:** `usage_need_segment = HIGH_NEED` khi mức sử dụng thực tế cao hơn đáng kể so với hạn mức/giá trị gói hiện tại; CVM tư vấn gói lớn hơn. `usage_need_segment = LOW_NEED` khi mức sử dụng thực tế thấp hơn đáng kể so với gói hiện tại; CVM tư vấn gói nhỏ hơn hoặc gói tiết kiệm hơn. Ngưỡng cụ thể do PO/CVM cấu hình theo từng loại tài nguyên và từng nhóm gói.
>
> **Điều kiện dữ liệu tối thiểu:** Mỗi bản ghi phải có ít nhất một cặp dữ liệu đủ để tính nhu cầu: `avg_data_usage_mb` + `current_plan_data_quota_mb`, hoặc `avg_voice_usage_min` + `current_plan_voice_quota_min`.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `current_plan_price` | integer | ❌ | Giá gói hiện tại (đồng) | OCS — bảng giá gói | `70000` |
| `plan_cycle` | string | ❌ | Chu kỳ gói hiện tại | OCS — định nghĩa gói | `MONTHLY` |
| `analysis_period_cycles` | integer | ✅ | Số chu kỳ gần nhất dùng để phân tích nhu cầu — chốt **2** (2 tháng liên tiếp) | BSS/CVM cấu hình | `2` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `date_of_birth` | date | ❌ | Ngày sinh KH — CVM tự tính tuổi cụ thể từ trường này | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `1998-03-15` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `365` |
| `plan_expiry_date` | date | ❌ | Ngày hết hạn gói hiện tại | `resource.msisdns.expiry_date` (BSS) | `2026-07-10` |
| `days_since_expiry` | integer | ❌ | Số ngày tính từ lúc hết hạn gói (âm nếu gói còn hạn, dương nếu đã hết hạn) | BSS tính: `ngay_hien_tai - plan_expiry_date` | `-5` |
| `register_channel` | string | ❌ | Kênh đăng ký gói của KH | OCS/BSS — kênh giao dịch đăng ký | `APP` hoặc `USSD` hoặc `AGENCY` |
| `register_method` | string | ❌ | Hình thức đăng ký | OCS/BSS — hình thức giao dịch | `MANUAL` hoặc `AUTO` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi đang áp dụng (nếu có) | OCS/CVM — chương trình KM | `KM_NANGGOI_15PCT` |
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
| `{{tuoi_kh}}` | ❌ | Tuổi KH — CVM tính từ `date_of_birth` để cá nhân hóa nội dung theo độ tuổi | Số | CSV `date_of_birth` → CVM tính tuổi | không cá nhân hóa theo tuổi | `28` |
| `{{ngay_het_han_goi}}` | ❌ | Ngày hết hạn gói hiện tại — để KH cân nhắc thời điểm đổi gói | Ngày (DD/MM/YYYY) | CSV `plan_expiry_date` | không hiển thị ngày hết hạn | `10/07/2026` |
| `{{so_ngay_tinh_tu_het_han}}` | ❌ | Số ngày tính từ lúc hết hạn (âm = còn hạn, dương = đã hết hạn) — làm ngữ cảnh cấp bách khi tư vấn đổi gói | Số | CSV `days_since_expiry` | không hiển thị | `-5 ngày` |
| `{{phan_khuc_nhu_cau}}` | ✅ | Phân khúc nhu cầu theo mức sử dụng thực tế để CVM phân nhánh nội dung | Văn bản | CSV `usage_need_segment` | — | `HIGH_NEED` |
| `{{loai_nhu_cau_chinh}}` | ✅ | Loại tài nguyên chính khiến KH cần đổi gói | Văn bản | CSV `primary_usage_resource` | — | `DATA` |
| `{{ty_le_su_dung_so_voi_goi}}` | ✅ | Tỷ lệ sử dụng so với gói hiện tại — bằng chứng để giải thích vì sao tư vấn đổi gói | Số | CSV `usage_to_quota_pct` | — | `175.0%` |
| `{{huong_tu_van_goi}}` | ✅ | Hướng tư vấn gói: gói lớn hơn cho nhu cầu lớn, gói nhỏ/tiết kiệm hơn cho nhu cầu nhỏ | Văn bản | CSV `recommendation_direction` | — | `UPSIZE` |
| `{{goi_de_xuat_theo_nhu_cau}}` | ❌ | Tên gói NBO đề xuất phù hợp với phân khúc nhu cầu | Văn bản | CSV `suggested_plan` hoặc CVM NBO | không hiện tên gói cụ thể | `GOI_DATA_120K` |

---

##### 2.14 File: `birthday_list_{YYYYMMDD}.csv` (U09)

**Mô tả:** Danh sách KH có sinh nhật, kỷ niệm tròn x năm kích hoạt SIM, hoặc sinh nhật nhà mạng trong ngày
**Trigger bởi:** BSS (quét hàng ngày)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Bổ sung sự kiện (2026-07-03):** Ngoài `BIRTHDAY` (sinh nhật KH) và `SIM_ANNIVERSARY` (tròn x năm kích hoạt SIM), thêm `OPERATOR_ANNIVERSARY` — sinh nhật nhà mạng, gửi cho toàn bộ tập KH vào ngày cố định (CVM cấu hình).

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `event_type` | string | ✅ | Loại sự kiện | BSS tổng hợp: `BIRTHDAY` nếu khớp `birthday`, `SIM_ANNIVERSARY` nếu khớp `activation_date`, `OPERATOR_ANNIVERSARY` nếu trùng ngày sinh nhật nhà mạng (CVM cấu hình) | `BIRTHDAY` hoặc `SIM_ANNIVERSARY` hoặc `OPERATOR_ANNIVERSARY` |
| `event_year` | integer | ❌ | Số năm (tuổi hoặc năm thứ mấy dùng SIM); không áp dụng cho `OPERATOR_ANNIVERSARY` | BSS tính từ `crm.customers.birthday` (tuổi) hoặc `resource.sims.activation_date` (năm dùng SIM) | `25` |
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

##### 2.15 File: `subscriber_demographic_{YYYYMMDD}.csv` (U10)

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

##### 2.18 File: `segment_update_{YYYYMMDD}.csv` (E_SEGMENT_UPDATE)

**Mô tả:** Danh sách KH vừa được phân loại hoặc thay đổi phân khúc thuê bao (CHURN_RISK, GAMING, ENTERTAINMENT...)
**Trigger bởi:** BSS hoặc CVM nội bộ (batch, chạy định kỳ theo chu kỳ phân khúc)
**Thời điểm push:** 02:00–04:00 hàng ngày (hoặc theo chu kỳ tuần/tháng tùy cấu hình)

> **Lưu ý:** Nếu phân khúc được CVM tự tính nội bộ thì file này không cần BSS push — CVM tự trigger. Nếu BSS tính và push thì dùng schema dưới đây. Cần xác nhận với Tech Lead.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `new_segment` | string | ✅ | Phân khúc mới — gồm phân khúc hành vi (CHURN_RISK, GAMING...) và phân khúc nhân khẩu/nghề nghiệp | BSS hoặc CVM nội bộ | `HSSV` / `FREELANCER_DRIVER` / `OFFICER_CNVVP` / `KOL_KOC_REVIEWER` / `RETIRED` / `CHURN_RISK` / `GAMING` |
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

##### 2.19 File: `no_plan_x_days_{YYYYMMDD}.csv` (E_NO_PLAN_X_DAYS)

**Mô tả:** Danh sách KH trạng thái ACTIVE nhưng không có gói cước nào đang active trong x ngày liên tiếp. Đề xuất ngưỡng **x = 10 ngày** (do CVM cấu hình).
**Trigger bởi:** BSS (quét hàng ngày từ `resource.msisdns`)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Chốt ngưỡng (2026-07-07):** Bỏ mốc nhắc nhiều tầng đã đề xuất trước đó, chốt **1 ngưỡng duy nhất x = 10 ngày** không có gói cước liên tiếp thì trigger. Rà soát đủ 10 trường theo yêu cầu: Họ tên, Giới tính, Tuổi KH (qua `date_of_birth`), SĐT, Tuổi thuê bao, Tên gói chính gần nhất, Chương trình KM, Gói gợi ý, Kênh đăng ký, Hình thức đăng ký.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `days_without_plan` | integer | ✅ | Số ngày không có gói cước (ngưỡng trigger: 10) | BSS tính từ ngày gói cuối hết hạn | `10` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `date_of_birth` | date | ❌ | Ngày sinh KH — CVM tự tính tuổi cụ thể từ trường này | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `1998-03-15` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày KH đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `365` |
| `last_plan_name` | string | ❌ | Tên gói chính gần nhất KH từng dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `last_plan_expiry_date` | date | ✅ | Ngày hết hạn gói cuối | `resource.msisdns.expiry_date` (BSS) | `2026-05-26` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi đăng ký lại (nếu có) | OCS/CVM — chương trình KM | `KM_QUAYLAI_20PCT` |
| `suggested_plan` | string | ❌ | Gói gợi ý NBO đề xuất phù hợp với số dư hiện có | CVM NBO dựa trên `balance` và `last_plan_name` | `GOI_DATA_30K` |
| `register_channel` | string | ❌ | Kênh đăng ký gói gợi ý | CVM cấu hình | `APP` hoặc `USSD` |
| `register_method` | string | ❌ | Hình thức đăng ký | CVM cấu hình | `MANUAL` hoặc `AUTO` |
| `balance` | integer | ✅ | Số dư tài khoản hiện tại (đồng) | OCS → BSS nightly batch | `15000` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH không có gói cước | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_ngay_khong_goi}}` | ✅ | Số ngày không có gói — tạo cảm giác cấp bách kích hoạt lại | Số | CSV `days_without_plan` | — | `10 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để CVM gợi ý gói phù hợp với mức tiền có sẵn | Tiền (VND) | CSV `balance` | — | `15.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn nhắc đăng ký gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{tuoi_kh}}` | ❌ | Tuổi KH — CVM tính từ `date_of_birth` để cá nhân hóa nội dung theo độ tuổi | Số | CSV `date_of_birth` → CVM tính tuổi | không cá nhân hóa theo tuổi | `28` |
| `{{ten_goi_cu}}` | ❌ | Gói cước chính gần nhất KH từng dùng — gợi ý gia hạn lại | Văn bản | CSV `last_plan_name` | không đề cập gói cũ | `GOI_DATA_70K` |
| `{{chuong_trinh_km}}` | ❌ | Chương trình khuyến mãi đăng ký lại để tăng động lực | Văn bản | CSV `promotion_code` | không hiện KM | `Giảm 20% khi đăng ký lại` |
| `{{kenh_dang_ky}}` | ❌ | Kênh đăng ký gợi ý để hướng dẫn KH thao tác | Văn bản | CSV `register_channel` | dùng kênh mặc định | `APP` |
| `{{goi_phu_hop_de_xuat}}` | ❌ | Tên gói NBO đề xuất phù hợp với số dư hiện có | Văn bản | CSV `suggested_plan` hoặc CVM NBO dựa trên `balance` và `last_plan_name` | không hiện gợi ý | `GOI_DATA_30K` |

---

#### Bổ sung từ template gốc — Giữ trong nhóm Sử dụng / Viễn thông

Các schema dưới đây có trong template gốc và được giữ lại theo xác nhận BA. Các trigger này thuộc nhóm Sử dụng / Viễn thông nhưng không thể hiện rõ thành từng ý trên ảnh, nên được tách riêng để không làm nhiễu mapping chính.

##### 2.7 File: `ngay_30_summary_{YYYYMMDD}.csv` (E11)

**Mô tả:** Tổng kết hành trình 30 ngày đầu của KH
**Trigger bởi:** OCS → BSS (nightly), theo sự kiện NGAY_0 + 30
**Thời điểm push:** 02:00–04:00 của ngày NGAY_0 + 30

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `full_name` | string(64) | ✅ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `contact_email` | string | ❌ | Email KH | `crm.customers.contact_email` | `nguyenvana@gmail.com` |
| `activation_date` | datetime | ✅ | NGAY_0 | `resource.msisdn_status_history.change_date` (lần ACTIVATED đầu tiên) | `2026-04-14 08:00:00` |
| `total_data_gb` | float | ✅ | Tổng data N1–N30 (GB) | OCS → BSS nightly batch (tổng hợp N1–N30) | `18.5` |
| `total_voice_min` | float | ✅ | Tổng thoại N1–N30 (phút) | OCS → BSS nightly batch (tổng hợp N1–N30) | `210` |
| `topup_count` | integer | ✅ | Số lần nạp tiền | OCS → BSS nightly batch | `3` |
| `plan_change_count` | integer | ✅ | Số lần đổi gói | OCS → BSS nightly batch | `1` |
| `firebase_token` | string | ❌ | Firebase token | `app_install_log.firebase_token` (SuperApp → Kafka → BSS) | `fMnR8x...` |
| `current_plan` | string | ✅ | Gói đang dùng hiện tại | OCS → BSS nightly batch | `GOI_DATA_120K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin tổng kết hành trình 30 ngày | Văn bản | CSV `full_name` | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH đến mốc 30 ngày | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{tong_data_30_ngay_gb}}` | ✅ | Tổng data KH tiêu thụ 30 ngày đầu — điểm nhấn trong tổng kết | Số | CSV `total_data_gb` | — | `18.5 GB` |
| `{{tong_thoai_30_ngay_phut}}` | ✅ | Tổng phút thoại 30 ngày đầu — điểm nhấn trong tổng kết | Số | CSV `total_voice_min` | — | `210 phút` |
| `{{so_lan_nap_tien}}` | ✅ | Số lần nạp tiền — thể hiện mức độ gắn kết với mạng | Số | CSV `topup_count` | — | `3` |
| `{{so_lan_doi_goi}}` | ✅ | Số lần đổi gói — thể hiện KH chủ động tìm gói phù hợp | Số | CSV `plan_change_count` | — | `1` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói cước KH đang dùng tại thời điểm tổng kết | Văn bản | CSV `current_plan` | — | `GOI_DATA_120K` |
| `{{email_kh}}` | ❌ | Email KH để gửi báo cáo tổng kết nếu kênh Email được bật | Văn bản | CSV `contact_email` | không gửi Email, chỉ Banner + Push | `nguyenvana@gmail.com` |

---

##### 2.8 File: `plan_register_{YYYYMMDD}.csv` (U02)

**Mô tả:** Danh sách KH đăng ký/gia hạn gói lần ≥ 2 trong ngày
**Trigger bởi:** OCS → BSS (nightly)
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `plan_name` | string | ✅ | Tên gói đã đăng ký | OCS → BSS nightly batch | `GOI_DATA_120K` |
| `plan_type` | string | ✅ | Loại gói | OCS → BSS nightly batch | `DATA` hoặc `VOICE` hoặc `COMBO` |
| `plan_cycle` | string | ✅ | Chu kỳ gói — dùng làm bộ lọc gói theo chu kỳ | OCS → BSS nightly batch | `DAILY` hoặc `WEEKLY` hoặc `MONTHLY` |
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

##### 2.10 File: `billing_2month_{YYYYMM}.csv` (U05-A)

**Mô tả:** Danh sách KH dùng **gói data tháng (MONTHLY)** hết data sớm (trước ngày 25) trong 2 tháng liên tiếp
**Trigger bởi:** OCS → BSS (nightly), export đầu tháng thứ 3
**Thời điểm push:** 02:00–04:00 ngày 1 của tháng thứ 3

> **Phạm vi áp dụng:** Chỉ KH đang dùng gói có hạn mức data cố định theo tháng. Không áp dụng cho gói data/ngày — xem file `daily_quota_pattern_{YYYYMM}.csv` (U05-B).

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `month1_depleted_date` | date | ✅ | Ngày hết data tháng 1 | OCS → BSS nightly batch (ngày data về 0 tháng 1) | `2026-03-18` |
| `month2_depleted_date` | date | ✅ | Ngày hết data tháng 2 | OCS → BSS nightly batch (ngày data về 0 tháng 2) | `2026-04-21` |
| `month1_total_data_gb` | float | ✅ | Tổng data dùng tháng 1 (GB) | OCS → BSS nightly batch | `15.2` |
| `month2_total_data_gb` | float | ✅ | Tổng data dùng tháng 2 (GB) | OCS → BSS nightly batch | `16.8` |
| `suggested_plan` | string | ❌ | Gợi ý gói nâng lên (nếu BSS có thể cung cấp) | OCS hoặc CVM tự tính dựa trên mức dùng trung bình | `GOI_DATA_120K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin gợi ý nâng gói | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH hết data sớm 2 tháng liên tiếp | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói đang dùng — làm cơ sở so sánh khi đề xuất nâng gói | Văn bản | CSV `current_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_data_thang_1}}` | ✅ | Ngày hết data tháng trước — minh chứng pattern hết sớm | Ngày (DD/MM/YYYY) | CSV `month1_depleted_date` | — | `18/03/2026` |
| `{{ngay_het_data_thang_2}}` | ✅ | Ngày hết data tháng gần nhất — xác nhận pattern lặp lại | Ngày (DD/MM/YYYY) | CSV `month2_depleted_date` | — | `21/04/2026` |
| `{{tong_data_thang_1_gb}}` | ✅ | Tổng data tháng trước KH đã dùng trước khi hết | Số | CSV `month1_total_data_gb` | — | `15.2 GB` |
| `{{tong_data_thang_2_gb}}` | ✅ | Tổng data tháng gần nhất KH đã dùng trước khi hết | Số | CSV `month2_total_data_gb` | — | `16.8 GB` |
| `{{goi_nang_de_xuat}}` | ❌ | Tên gói nâng NBO đề xuất dựa trên mức dùng thực tế 2 tháng | Văn bản | CSV `suggested_plan` (optional) hoặc CVM NBO | không hiện tên gói cụ thể | `GOI_DATA_120K` |

---

##### 2.10b File: `daily_quota_pattern_{YYYYMM}.csv` (U05-B)

**Mô tả:** Danh sách KH dùng **gói data ngày/tuần (DAILY/WEEKLY)** có pattern hết quota data thường xuyên. Số ngày hết ≥ N_DAYS_THRESHOLD trong M_MONTHS_THRESHOLD tháng liên tiếp; các ngưỡng do CVM cấu hình.
**Trigger bởi:** OCS → BSS (nightly), export đầu tháng thứ (M+1)
**Thời điểm push:** 02:00–04:00 ngày 1 của tháng thứ (M+1)

> **Phạm vi áp dụng:** Chỉ KH đang dùng gói có quota data reset hàng ngày (gói ngày, gói tuần chia data/ngày). Không áp dụng cho gói tháng — xem `billing_2month_{YYYYMM}.csv` (U05-A).
>
> **Phân biệt với U05-B-RT:** File batch này dùng để phân tích **pattern nhiều tháng** (tư vấn nâng/hạ gói định kỳ, không cấp bách). Trường hợp KH hết quota **3 lần liên tiếp trong thời gian ngắn** (cấp bách hơn, cần đề xuất ngay) dùng trigger riêng **`U05-B-RT`** bên dưới — không gộp chung schema.
>
> **⚠️ Vấn đề nguồn dữ liệu (Q21):** Để biết "ngày X KH có hết quota/ngày không", OCS cần event quota/ngày về 0 hoặc BSS tính từ CDR. Nếu OCS không có event này thì compute nặng — cần xác nhận với BSS/Tech.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `current_plan` | string | ✅ | Gói ngày/tuần đang dùng | OCS → BSS nightly batch | `GOI_DATA_NGAY_10K` |
| `plan_cycle` | string | ✅ | Chu kỳ gói | OCS — định nghĩa gói | `DAILY` hoặc `WEEKLY` |
| `daily_quota_mb` | integer | ✅ | Hạn mức data/ngày của gói (MB) | OCS — theo định nghĩa gói | `500` |
| `month1_depleted_days` | integer | ✅ | Số ngày trong tháng 1 hết quota | OCS/BSS — cần xác nhận (Q21) | `18` |
| `month2_depleted_days` | integer | ✅ | Số ngày trong tháng 2 hết quota | OCS/BSS — cần xác nhận (Q21) | `20` |
| `month1_total_data_gb` | float | ❌ | Tổng data tháng 1 (GB) | OCS → BSS nightly batch | `14.2` |
| `month2_total_data_gb` | float | ❌ | Tổng data tháng 2 (GB) | OCS → BSS nightly batch | `15.0` |
| `suggested_daily_plan` | string | ❌ | Gói ngày lớn hơn đề xuất | OCS hoặc CVM NBO | `GOI_DATA_NGAY_15K` |
| `suggested_monthly_plan` | string | ❌ | Gói tháng đề xuất nếu upsell | CVM NBO tự tính | `GOI_DATA_70K` |

**Param template:**

> CVM phân nhánh nội dung theo `UPSELL_MODE` (cấu hình nội bộ CVM): `DAILY_UPGRADE` gợi ý gói ngày lớn hơn; `MONTHLY_UPSELL` gợi ý gói tháng; `BOTH` đề xuất cả 2.

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin gợi ý | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH hết quota gói ngày/tuần thường xuyên | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói ngày/tuần đang dùng — làm ngữ cảnh gợi ý | Văn bản | CSV `current_plan` | — | `GOI_DATA_NGAY_10K` |
| `{{han_muc_data_ngay_mb}}` | ✅ | Hạn mức data/ngày của gói hiện tại | Số | CSV `daily_quota_mb` | — | `500 MB` |
| `{{goi_ngay_nang_de_xuat}}` | ❌ | Gói ngày lớn hơn NBO đề xuất | Văn bản | CSV `suggested_daily_plan` hoặc CVM NBO | không hiện | `GOI_DATA_NGAY_15K` |
| `{{goi_thang_upsell_de_xuat}}` | ❌ | Gói tháng NBO đề xuất thay thế | Văn bản | CSV `suggested_monthly_plan` hoặc CVM NBO | không hiện | `GOI_DATA_70K` |

---

##### Event: HẾT_QUOTA_NGÀY_LIÊN_TIẾP (U05-B-RT)

**Mô tả:** KH hết quota gói ngày/tuần **3 lần liên tiếp** (3 ngày hết gói ngày, hoặc 3 chu kỳ hết gói tuần) — tín hiệu cấp bách hơn pattern nhiều tháng của U05-B, cần đề xuất nâng gói ngay, không đợi tổng hợp cuối tháng.
**Trigger bởi:** OCS gọi API CVM
**Yêu cầu kỹ thuật:** OCS đếm số lần liên tiếp KH hết quota gói ngày/tuần; ngay khi chạm ngưỡng (đề xuất 3 lần, do CVM cấu hình) thì gọi API CVM. CVM gửi đề xuất nâng gói trong vòng vài phút.
**Timing:** Ngay khi đủ N lần hết quota liên tiếp (đề xuất N = 3)

> **Phân biệt với U05-B:** Đây là trigger **NearRealtime riêng biệt**, không phải cùng file/schema với `U05-B` batch. `U05-B` phân tích pattern nhiều tháng để tư vấn định kỳ; `U05-B-RT` phản ứng ngay khi phát hiện chuỗi hết quota liên tiếp trong thời gian ngắn.
>
> **⚠️ Vấn đề nguồn dữ liệu (Q21):** Cần OCS có event quota/ngày về 0 theo thời gian thực để đếm được `consecutive_depleted_count` ngay lập tức. Nếu OCS chỉ ghi CDR (không có event realtime) thì không thể làm nhánh này NearRealtime — cần xác nhận với BSS/Tech trước khi implement.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | OCS — event name cố định | `DAILY_QUOTA_DEPLETED_STREAK` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm chạm đủ N lần liên tiếp | OCS — thời điểm đủ điều kiện | `2026-05-15 23:50:00` |
| `current_plan` | string | ✅ | Gói ngày/tuần đang dùng | OCS — tên gói đang active | `GOI_DATA_NGAY_10K` |
| `plan_cycle` | string | ✅ | Chu kỳ gói | OCS — định nghĩa gói | `DAILY` hoặc `WEEKLY` |
| `daily_quota_mb` | integer | ✅ | Hạn mức data/ngày của gói (MB) | OCS — theo định nghĩa gói | `500` |
| `consecutive_depleted_count` | integer | ✅ | Số lần hết quota liên tiếp tính đến thời điểm trigger | OCS (event quota về 0) — cần xác nhận (Q21) | `3` |
| `balance` | integer | ❌ | Số dư tài khoản hiện tại (đồng) | OCS — số dư realtime | `25000` |
| `suggested_daily_plan` | string | ❌ | Gói ngày lớn hơn đề xuất | OCS hoặc CVM NBO | `GOI_DATA_NGAY_15K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH vừa hết quota liên tiếp N lần | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{so_lan_het_lien_tiep}}` | ✅ | Số lần hết quota liên tiếp — bằng chứng cho đề xuất nâng gói ngay | Số | Payload `consecutive_depleted_count` | — | `3 lần` |
| `{{ten_goi_hien_tai}}` | ✅ | Gói ngày/tuần đang dùng — làm ngữ cảnh đề xuất | Văn bản | Payload `current_plan` | — | `GOI_DATA_NGAY_10K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin đề xuất | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{goi_ngay_nang_de_xuat}}` | ❌ | Gói ngày lớn hơn NBO đề xuất ngay | Văn bản | Payload `suggested_daily_plan` hoặc CVM NBO | không hiện | `GOI_DATA_NGAY_15K` |

---

##### 2.11 File: `plan_change_{YYYYMMDD}.csv` (U06)

**Mô tả:** Danh sách KH chuyển đổi **gói tháng (MONTHLY)** thành công trong ngày
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

##### 2.12 File: `sim_swap_{YYYYMMDD}.csv` (U07)

**Mô tả:** Danh sách KH chuyển đổi SIM/đổi phôi SIM nội mạng thành công trong ngày. Bao gồm **4 tổ hợp chuyển đổi**: vật lý→vật lý, vật lý→eSIM, eSIM→vật lý, eSIM→eSIM. Trường hợp cùng loại (vật lý→vật lý, eSIM→eSIM) chính là **đổi phôi SIM**.
**Trigger bởi:** BSS (sim_swap event)
**Thời điểm push:** 02:00–04:00 hàng ngày

> **Phân loại swap (2026-07-03):** `swap_type` xác định rõ tổ hợp: `PHYS_TO_PHYS` và `ESIM_TO_ESIM` là đổi phôi (cùng loại); `PHYS_TO_ESIM` và `ESIM_TO_PHYS` là chuyển đổi loại SIM. CVM phân nhánh nội dung hướng dẫn theo `swap_type`.

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `old_sim_type` | string | ✅ | Loại SIM cũ | `resource.sims.sim_type` (bản ghi SIM cũ) | `PHYSICAL` hoặc `ESIM` |
| `new_sim_type` | string | ✅ | Loại SIM mới | `resource.sims.sim_type` (bản ghi SIM mới) | `PHYSICAL` hoặc `ESIM` |
| `swap_type` | string | ✅ | Tổ hợp chuyển đổi (4 case) | BSS phân loại từ `old_sim_type` + `new_sim_type` | `PHYS_TO_PHYS`, `PHYS_TO_ESIM`, `ESIM_TO_PHYS`, `ESIM_TO_ESIM` |
| `is_reissue` | boolean | ✅ | Có phải đổi phôi (cùng loại SIM) không | BSS: `true` nếu `old_sim_type = new_sim_type` | `true` |
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
| `{{loai_chuyen_doi}}` | ✅ | Tổ hợp chuyển đổi/đổi phôi để CVM phân nhánh nội dung hướng dẫn | Văn bản | CSV `swap_type` | — | `ESIM_TO_ESIM` |
| `{{ma_iccid_moi}}` | ✅ | Mã ICCID SIM mới để KH lưu làm bằng chứng kích hoạt | Văn bản | CSV `new_iccid` | — | `8984012345678901235` |
| `{{ten_goi_giu_nguyen}}` | ✅ | Tên gói cước được giữ nguyên sau khi đổi SIM — reassurance KH | Văn bản | CSV `current_plan` | — | `GOI_DATA_120K` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin xác nhận đổi SIM | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |

---

---

### Nhánh nghiệp vụ — APP

**Đối chiếu mục trong ảnh:**
- Sau 24h chưa cài đặt APP
- Sau 24h chưa truy cập APP
- Hoàn thành nhiệm vụ trên APP
- Đăng nhập APP lần đầu
- Chuyển đổi gói cước trên APP nhưng không thực hiện
- X ngày KH không truy cập APP
- Mua dịch vụ nội dung không thành công
- KH đánh giá APP

> **Ghi chú mapping:** “Sau 24h chưa cài đặt APP” đã được đặc tả bằng `E02` tại `NHÓM 1 — Kích hoạt / APP`. “Sau 24h chưa truy cập APP” được đặc tả bằng `E04`. “X ngày KH không truy cập APP” được bổ sung mới bằng `E_APP_INACTIVE_X_DAYS`.

#### Kỹ thuật — NearRealtime

##### 1.2 Event: ĐĂNG_NHẬP_APP_LẦN_ĐẦU (E03)

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

##### 1.10 Event: MUA_DỊCH_VỤ_NỘI_DUNG_THẤT_BẠI (E_CONTENT_FAIL)

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

##### 1.11 Event: ĐÁNH_GIÁ_APP (E_APP_RATING)

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

#### Kỹ thuật — Batch/CSV

##### 2.2 Event: CHƯA_MỞ_APP_SAU_24H (E04)

**Mô tả:** KH đã cài app nhưng chưa mở trong 24h
**Trigger bởi:** SuperApp → BSS gọi API CVM
**Yêu cầu kỹ thuật:** BSS/SuperApp gọi API CVM ngay khi phát hiện đủ 24h từ lúc cài mà chưa mở app. CVM gửi Push/tin nhắn nhắc mở app.
**Timing:** Ngay khi đủ 24h tính từ lúc cài app mà chưa mở

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `app_installed_no_open` sang **NearRealtime** theo bảng trigger đã chốt.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | SuperApp/BSS — event name cố định | `NO_APP_OPEN_24H` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện đủ 24h chưa mở app | SuperApp/BSS — thời điểm đủ điều kiện | `2026-05-14 10:15:00` |
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

##### 2.4 File: `milestone_D7_{YYYYMMDD}.csv` (E07)

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

##### 2.5 Event: HÀNH_TRÌNH_MUA_BỎ_DỞ (E09)

**Mô tả:** KH bắt đầu hành trình mua nhưng chưa hoàn tất đăng ký, gồm 2 luồng: (1) **xem màn hình đổi gói nhưng chưa đăng ký**; (2) **hành trình mua cả SIM cả gói** — tính từ X phút kể từ lúc bắt đầu hành trình mua SIM/mua gói, với **điều kiện có để lại SĐT**.
**Trigger bởi:** SuperApp → gọi API CVM
**Yêu cầu kỹ thuật:** SuperApp gọi API CVM khi phát hiện KH đã qua X phút (X do CVM cấu hình) kể từ lúc bắt đầu hành trình mua mà chưa hoàn tất. Chỉ trigger nếu KH đã để lại SĐT trong hành trình. CVM gửi nhắc hoàn tất mua trong vòng vài phút.
**Timing:** Ngay khi đủ X phút kể từ lúc bắt đầu hành trình mua mà chưa hoàn tất (điều kiện: có SĐT)

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `change_pkg_view` sang **NearRealtime** và mở rộng phạm vi bao gồm hành trình mua SIM + gói bỏ dở, theo bảng trigger đã chốt.
>
> **Điều kiện chặn:** Nếu KH chưa để lại SĐT trong hành trình (`msisdn` rỗng) → CVM không trigger (không có kênh liên hệ).

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | SuperApp — event name cố định | `PURCHASE_JOURNEY_ABANDONED` |
| `msisdn` | string(15) | ✅ | Số điện thoại KH để lại trong hành trình | SuperApp — SĐT KH nhập | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm phát hiện bỏ dở (đủ X phút) | SuperApp — thời điểm đủ điều kiện | `2026-05-13 21:15:00` |
| `journey_type` | string | ✅ | Loại hành trình bỏ dở | SuperApp — phân loại hành trình | `CHANGE_PLAN` hoặc `BUY_SIM_AND_PLAN` |
| `journey_start_at` | datetime | ✅ | Thời điểm bắt đầu hành trình mua SIM/mua gói | SuperApp — mốc bắt đầu hành trình | `2026-05-13 21:05:00` |
| `minutes_since_start` | integer | ✅ | Số phút kể từ lúc bắt đầu hành trình | SuperApp/CVM tính | `10` |
| `last_step` | string | ❌ | Bước cuối cùng KH dừng lại trong hành trình | SuperApp — bước hành trình | `SELECT_PLAN` hoặc `PAYMENT` |
| `current_plan` | string | ❌ | Gói đang dùng (nếu là luồng đổi gói của KH hiện hữu) | OCS — tên gói đang active | `GOI_DATA_70K` |
| `selected_plan` | string | ❌ | Gói KH đang chọn dở trong hành trình | SuperApp — gói đang chọn | `GOI_DATA_120K` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{ten_kh}}` | ✅ | Họ tên KH để cá nhân hóa tin nhắn nhắc hoàn tất mua | Văn bản | CVM cache từ E01 (nếu là KH hiện hữu) | `"Quý khách"` | `Nguyễn Văn A` |
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH để lại trong hành trình | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{loai_hanh_trinh}}` | ✅ | Loại hành trình để CVM phân nhánh nội dung nhắc | Văn bản | Payload `journey_type` | — | `BUY_SIM_AND_PLAN` |
| `{{ten_goi_hien_tai}}` | ❌ | Tên gói đang dùng (nếu là luồng đổi gói) — ngữ cảnh gợi ý | Văn bản | Payload `current_plan` | không hiện gói hiện tại | `GOI_DATA_70K` |
| `{{goi_dang_chon}}` | ❌ | Gói KH đang chọn dở — nhắc hoàn tất đúng gói đó | Văn bản | Payload `selected_plan` | không hiện gói đang chọn | `GOI_DATA_120K` |
| `{{goi_de_xuat}}` | ❌ | Tên gói NBO đề xuất phù hợp với hành vi | Văn bản | CVM NBO — không lấy từ BSS/OCS | không hiện gợi ý | `GOI_DATA_120K` |

---

##### File: `app_inactive_x_days_{YYYYMMDD}.csv` (E_APP_INACTIVE_X_DAYS)

**Mô tả:** Danh sách KH đã từng sử dụng/đăng nhập APP nhưng không truy cập APP trong x ngày liên tiếp. Giá trị x do CVM cấu hình theo campaign, ví dụ 7, 14 hoặc 30 ngày.
**Trigger bởi:** SuperApp → Kafka/BSS tổng hợp batch hoặc CVM tự tính từ log truy cập APP
**Thời điểm push:** 02:00–04:00 hàng ngày

| Cột | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` hoặc SuperApp account mapping | `0901234567` |
| `last_open_at` | datetime | ✅ | Lần gần nhất KH mở APP | SuperApp `app_open_log.last_open_at` | `2026-05-20 08:30:00` |
| `days_since_last_open` | integer | ✅ | Số ngày không truy cập APP | BSS/CVM tính từ `last_open_at` đến ngày hiện tại | `14` |
| `app_version` | string | ❌ | Phiên bản APP lần gần nhất KH sử dụng | SuperApp log | `2.1.0` |
| `device_type` | string | ❌ | Loại thiết bị | SuperApp log | `ANDROID` |
| `firebase_token` | string | ❌ | Firebase token để gửi Push nếu còn hiệu lực | `app_install_log.firebase_token` | `fMnR8x...` |
| `current_plan` | string | ❌ | Gói đang dùng | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `last_campaign_sent_at` | datetime | ❌ | Lần gần nhất CVM đã nhắc KH quay lại APP | CVM campaign history | `2026-05-28 09:00:00` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH không truy cập APP trong x ngày | Văn bản | CSV `msisdn` | — | `0901234567` |
| `{{so_ngay_khong_mo_app}}` | ✅ | Số ngày KH chưa mở APP | Số | CSV `days_since_last_open` | — | `14 ngày` |
| `{{ngay_mo_app_gan_nhat}}` | ✅ | Ngày gần nhất KH mở APP | Ngày (DD/MM/YYYY) | CSV `last_open_at` | — | `20/05/2026` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn quay lại APP | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{ten_goi_hien_tai}}` | ❌ | Gói đang dùng để gợi ý thao tác phù hợp trong APP | Văn bản | CSV `current_plan` | `"gói hiện tại"` | `GOI_DATA_70K` |
| `{{uu_dai_quay_lai_app}}` | ❌ | Ưu đãi hoặc nội dung NBO để kéo KH quay lại APP | Văn bản | CVM campaign/NBO | không hiện ưu đãi cụ thể | `Nhận 1GB khi mở APP hôm nay` |

---

---

## PHẦN 4 — NHÓM 3 — Gia hạn gói/dịch vụ

### Nhánh nghiệp vụ — GIA HẠN

**Đối chiếu mục trong ảnh:**
- Gia hạn thành công liên tiếp gói tháng x chu kỳ
- x ngày trước khi hết hạn gói
- x ngày sau khi hết hạn mà KH không gia hạn gói

> **Ghi chú hợp nhất:** Trigger chuỗi gia hạn `U_RENEWAL_STREAK` đã được gộp vào `U08`. Nhánh GIA HẠN chỉ dùng file `renewal_loyalty_{YYYYMMDD}.csv` để xử lý cả gia hạn đúng hạn và gia hạn sớm theo mốc cấu hình của CVM.

> **Cập nhật timing (2026-07-03):** `U_PRE_EXPIRY` và `U_POST_EXPIRY` chuyển từ Batch/CSV sang **NearRealtime** theo bảng trigger đã chốt. `U08` giữ Batch/CSV.

#### Kỹ thuật — Batch/CSV

##### 2.13 File: `renewal_loyalty_{YYYYMMDD}.csv` (U08)

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
| `plan_cycle` | string | ✅ | Chu kỳ gói được gia hạn liên tiếp — dùng làm bộ lọc chuỗi gia hạn theo loại chu kỳ | OCS → BSS nightly batch | `DAILY` hoặc `WEEKLY` hoặc `MONTHLY` |
| `renewal_count_threshold` | integer | ❌ | Ngưỡng số lần gia hạn (x lần) theo từng chu kỳ để tính đủ điều kiện vinh danh | CVM cấu hình theo `plan_cycle` | `3` |
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

##### 2.20 Event: TRƯỚC_KHI_HẾT_HẠN_GÓI (U_PRE_EXPIRY)

**Mô tả:** KH còn x ngày trước khi gói hết hạn (x do CVM cấu hình, thường 3 và 7 ngày)
**Trigger bởi:** BSS gọi API CVM
**Yêu cầu kỹ thuật:** BSS gọi API CVM ngay khi KH chạm mốc còn x ngày trước hết hạn. CVM gửi nhắc gia hạn.
**Timing:** Ngay khi KH còn đúng x ngày trước khi gói hết hạn

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `pre_expiry` sang **NearRealtime** theo bảng trigger đã chốt.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `PRE_EXPIRY` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm chạm mốc còn x ngày | BSS — thời điểm đủ điều kiện | `2026-06-07 08:00:00` |
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

##### 2.21 Event: SAU_KHI_HẾT_HẠN_GÓI (U_POST_EXPIRY)

**Mô tả:** KH đã hết hạn gói x ngày mà chưa gia hạn (x do CVM cấu hình, thường 1, 3, 7 ngày)
**Trigger bởi:** BSS gọi API CVM
**Yêu cầu kỹ thuật:** BSS gọi API CVM ngay khi KH chạm mốc đã quá hạn x ngày mà chưa gia hạn. CVM gửi thúc gia hạn.
**Timing:** Ngay khi KH đã quá hạn đúng x ngày mà chưa gia hạn

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `post_expiry` sang **NearRealtime** theo bảng trigger đã chốt. Bổ sung trường thông tin KH và kênh/hình thức gia hạn theo cột Mô tả trong bảng trigger.
>
> **Rà soát bổ sung (2026-07-07):** Đối chiếu đủ 13 trường theo yêu cầu: Họ tên, Giới tính, Tuổi KH (qua `date_of_birth`, CVM tự tính tuổi), SĐT, Tuổi thuê bao, Tên gói, Giá gói, Chu kỳ gói, Ngày hết hạn gói + số ngày tính từ lúc hết hạn, Chương trình KM, Gói gợi ý (bổ sung cột `suggested_plan` — trước đó chỉ có ở param), Kênh gia hạn, Hình thức gia hạn.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `POST_EXPIRY` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm chạm mốc quá hạn x ngày | BSS — thời điểm đủ điều kiện | `2026-06-03 08:00:00` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `crm.request_register_info.ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `date_of_birth` | date | ❌ | Ngày sinh KH — CVM tự tính tuổi cụ thể từ trường này | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `1998-03-15` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `365` |
| `last_plan` | string | ✅ | Tên gói cuối cùng đã hết hạn | OCS → BSS nightly batch | `GOI_DATA_70K` |
| `last_plan_price` | integer | ❌ | Giá gói vừa hết hạn (đồng) | OCS — bảng giá gói | `70000` |
| `plan_cycle` | string | ❌ | Chu kỳ gói vừa hết hạn | OCS — định nghĩa gói | `MONTHLY` |
| `expiry_date` | date | ✅ | Ngày gói đã hết hạn | `resource.msisdns.expiry_date` (BSS) | `2026-05-31` |
| `days_since_expiry` | integer | ✅ | Số ngày đã qua hạn chưa gia hạn (tính từ lúc hết hạn) | BSS tính: `ngay_hien_tai - expiry_date` | `3` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi áp dụng (nếu có) | OCS/CVM — chương trình KM đang chạy | `KM_GIAHAN_20PCT` |
| `suggested_plan` | string | ❌ | Gói gợi ý NBO đề xuất phù hợp số dư hiện có (nếu có) | CVM NBO dựa trên `balance` và `last_plan` | `GOI_DATA_50K` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS → BSS nightly batch | `5000` |
| `subscriber_status` | string | ✅ | Trạng thái thuê bao hiện tại | `crm.subscribers.status` | `ACTIVE` hoặc `GRACE` |
| `renewal_channel` | string | ❌ | Kênh gia hạn gợi ý cho KH | CVM cấu hình — kênh gia hạn ưu tiên | `APP` hoặc `USSD` hoặc `WEB` |
| `renewal_method` | string | ❌ | Hình thức gia hạn (thủ công / tự động) | CVM cấu hình theo `auto_renewal_enabled` | `MANUAL` hoặc `AUTO` |
| `renewal_attempts` | integer | ❌ | Số lần CVM đã nhắc gia hạn trong đợt này | CVM nội bộ — đếm số lần trigger đã fire | `1` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại KH chưa gia hạn sau khi hết hạn | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{ten_goi_cu}}` | ✅ | Tên gói vừa hết hạn — để KH nhận ra và cân nhắc gia hạn lại | Văn bản | Payload `last_plan` | — | `GOI_DATA_70K` |
| `{{ngay_het_han}}` | ✅ | Ngày gói đã hết hạn — xác nhận mốc cần hành động | Ngày (DD/MM/YYYY) | Payload `expiry_date` | — | `31/05/2026` |
| `{{so_ngay_qua_han}}` | ✅ | Số ngày đã quá hạn — tạo cảm giác cấp bách | Số | Payload `days_since_expiry` | — | `3 ngày` |
| `{{so_du_tai_khoan}}` | ✅ | Số dư tài khoản — để CVM gợi ý gói phù hợp mức tiền có sẵn | Tiền (VND) | Payload `balance` | — | `5.000 VNĐ` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn thúc gia hạn | Văn bản | Payload `full_name` hoặc CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{tuoi_kh}}` | ❌ | Tuổi KH — CVM tính từ `date_of_birth` để cá nhân hóa nội dung theo độ tuổi | Số | Payload `date_of_birth` → CVM tính tuổi | không cá nhân hóa theo tuổi | `28` |
| `{{gia_goi_cu}}` | ❌ | Giá gói vừa hết hạn — để KH so sánh khi cân nhắc gia hạn | Tiền (VND) | Payload `last_plan_price` | không hiển thị giá | `70.000 VNĐ` |
| `{{chuong_trinh_km}}` | ❌ | Chương trình khuyến mãi gia hạn để tăng động lực | Văn bản | Payload `promotion_code` | không hiện KM | `Giảm 20% khi gia hạn` |
| `{{kenh_gia_han}}` | ❌ | Kênh gia hạn gợi ý để hướng dẫn KH thao tác | Văn bản | Payload `renewal_channel` | dùng kênh mặc định | `APP` |
| `{{goi_gia_han_de_xuat}}` | ❌ | Tên gói gia hạn NBO đề xuất phù hợp số dư hiện có | Văn bản | CVM NBO dựa trên `balance` và `last_plan` | không hiện gợi ý | `GOI_DATA_50K` |

---

---

## PHẦN 5 — NHÓM 4 — Khóa 1c/Khóa 2c

### Nhánh nghiệp vụ — KHÓA 1C/2C

**Đối chiếu mục trong ảnh:**
- Nguyên nhân khóa 1c
- x ngày trước khi khóa 2c
- Khóa 2 chiều

#### Kỹ thuật — NearRealtime

##### 1.9 Event: KHÓA_2_CHIỀU (E_LOCK_2C)

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

##### 1.12 Event: KHÓA_1_CHIỀU (E_LOCK_1C)

**Mô tả:** KH bị khóa 1 chiều — tách 2 kịch bản: (A) hệ thống tác động (admin lock), (B) không sử dụng quá lâu
**Trigger bởi:** BSS/OCS gọi API CVM
**Yêu cầu kỹ thuật:** BSS/OCS gọi API CVM ngay khi thuê bao chuyển trạng thái LOCK_1C. CVM gửi USSD/Push hướng dẫn khôi phục.
**Timing:** Ngay khi tài khoản bị khóa 1 chiều

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `lock_1c` sang **NearRealtime** theo bảng trigger đã chốt. Bổ sung trường thông tin KH + nguyên nhân khóa + CTKM theo cột Mô tả trong bảng.
>
> **Rà soát bổ sung (2026-07-07):** Đối chiếu đủ 7 trường theo yêu cầu: Họ tên, Giới tính, Tuổi KH (qua `date_of_birth`, CVM tự tính tuổi), SĐT, Tuổi thuê bao, Nguyên nhân khóa 1 chiều, Chương trình KM.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `LOCK_1C` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm khóa 1 chiều | BSS — thời điểm chuyển trạng thái LOCK_1C | `2026-06-02 00:00:00` |
| `full_name` | string(64) | ❌ | Họ tên KH | `crm.customers.full_name` | `Nguyễn Văn A` |
| `gender` | string | ❌ | Giới tính KH | `ekyc_data` → `gender` — chưa xác nhận nguồn | `MALE` |
| `date_of_birth` | date | ❌ | Ngày sinh KH — CVM tự tính tuổi cụ thể từ trường này | `ekyc_data` → `date_of_birth` — chưa xác nhận nguồn | `1998-03-15` |
| `subscriber_tenure_days` | integer | ❌ | Tuổi thuê bao (số ngày đã dùng mạng) | BSS tính từ `resource.msisdn_status_history` | `365` |
| `lock_scenario` | string | ✅ | Kịch bản khóa (nguyên nhân khóa 1 chiều) | BSS phân loại: `SYSTEM_ACTION` hoặc `INACTIVE` | `INACTIVE` |
| `days_inactive` | integer | ❌ | Số ngày không sử dụng (nếu kịch bản INACTIVE) | BSS tính từ CDR | `90` |
| `lock_reason_detail` | string | ❌ | Mô tả chi tiết lý do (nếu kịch bản SYSTEM_ACTION) | BSS — lý do admin lock | `FRAUD_SUSPECTED` |
| `promotion_code` | string | ❌ | Chương trình khuyến mãi mở khóa/quay lại (nếu có) | OCS/CVM — chương trình KM đang chạy | `KM_MOKHOA_10PCT` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS — số dư realtime | `0` |
| `days_to_lock_2c` | integer | ❌ | Số ngày còn lại trước khi chuyển khóa 2 chiều | BSS tính theo quy định | `30` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại bị khóa 1 chiều | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{nguyen_nhan_khoa}}` | ✅ | Nguyên nhân khóa 1 chiều để CVM phân nhánh nội dung hướng dẫn khôi phục | Văn bản | Payload `lock_scenario` | — | `INACTIVE` |
| `{{so_ngay_con_den_khoa_2c}}` | ❌ | Số ngày còn lại trước khi khóa 2 chiều — tạo urgency | Số | Payload `days_to_lock_2c` | không hiển thị đếm ngược | `30 ngày` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn khóa 1 chiều | Văn bản | Payload `full_name` hoặc CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{tuoi_kh}}` | ❌ | Tuổi KH — CVM tính từ `date_of_birth` để cá nhân hóa nội dung theo độ tuổi | Số | Payload `date_of_birth` → CVM tính tuổi | không cá nhân hóa theo tuổi | `28` |
| `{{chuong_trinh_km}}` | ❌ | Chương trình khuyến mãi mở khóa để tăng động lực quay lại | Văn bản | Payload `promotion_code` | không hiện KM | `Giảm 10% khi mở khóa` |
| `{{huong_dan_mo_khoa}}` | ✅ | Hướng dẫn cụ thể để KH tự mở khóa 1 chiều | Văn bản | CVM cấu hình tĩnh theo `lock_scenario` | — | `Sử dụng dịch vụ hoặc liên hệ 1800xxx` |

---

##### 1.13 Event: TRƯỚC_KHI_KHÓA_2_CHIỀU (E_PRE_LOCK_2C)

**Mô tả:** KH đang ở trạng thái khóa 1 chiều và còn x ngày trước khi bị khóa 2 chiều (x do CVM cấu hình, thường 7 và 3 ngày)
**Trigger bởi:** BSS gọi API CVM
**Yêu cầu kỹ thuật:** BSS gọi API CVM ngay khi KH chạm mốc còn x ngày trước khóa 2 chiều. CVM gửi cảnh báo khẩn.
**Timing:** Ngay khi KH còn đúng x ngày trước khi khóa 2 chiều

> **Cập nhật timing (2026-07-03):** Chuyển từ Batch/CSV `pre_lock_2c` sang **NearRealtime** theo bảng trigger đã chốt.

| Trường | Kiểu | Bắt buộc | Mô tả | Nguồn tham chiếu | Ví dụ |
|---|---|---|---|---|---|
| `event_type` | string | ✅ | Loại sự kiện | BSS — event name cố định | `PRE_LOCK_2C` |
| `msisdn` | string(15) | ✅ | Số điện thoại | `crm.subscribers.msisdn` | `0901234567` |
| `event_timestamp` | datetime | ✅ | Thời điểm chạm mốc còn x ngày | BSS — thời điểm đủ điều kiện | `2026-06-25 08:00:00` |
| `lock_1c_date` | date | ✅ | Ngày bắt đầu khóa 1 chiều | BSS — ngày chuyển trạng thái LOCK_1C | `2026-05-03` |
| `lock_2c_scheduled_date` | date | ✅ | Ngày dự kiến khóa 2 chiều | BSS tính từ quy định (thường lock_1c_date + 30 ngày) | `2026-07-02` |
| `days_to_lock_2c` | integer | ✅ | Số ngày còn lại trước khóa 2 chiều | BSS tính: `lock_2c_scheduled_date - ngay_hien_tai` | `7` |
| `lock_scenario` | string | ✅ | Kịch bản khóa 1c ban đầu | BSS — `SYSTEM_ACTION` hoặc `INACTIVE` | `INACTIVE` |
| `balance` | integer | ✅ | Số dư tài khoản (đồng) | OCS — số dư realtime | `0` |

**Param template:**

| Param | Bắt buộc | Mô tả | Định dạng | Nguồn dữ liệu | Fallback | Ví dụ |
|---|---|---|---|---|---|---|
| `{{so_dien_thoai}}` | ✅ | Số điện thoại sắp bị khóa 2 chiều | Văn bản | Payload `msisdn` | — | `0901234567` |
| `{{so_ngay_con_lai}}` | ✅ | Số ngày còn trước khi khóa 2 chiều — điểm nhấn urgency | Số | Payload `days_to_lock_2c` | — | `7 ngày` |
| `{{ngay_khoa_2c_du_kien}}` | ✅ | Ngày dự kiến khóa 2 chiều — để KH nắm deadline | Ngày (DD/MM/YYYY) | Payload `lock_2c_scheduled_date` | — | `02/07/2026` |
| `{{ten_kh}}` | ❌ | Họ tên KH để cá nhân hóa tin nhắn cảnh báo khẩn | Văn bản | CVM cache từ E01 | `"Quý khách"` | `Nguyễn Văn A` |
| `{{huong_dan_mo_khoa}}` | ✅ | Hướng dẫn cụ thể để mở khóa trước deadline | Văn bản | CVM cấu hình tĩnh theo `lock_scenario` | — | `Đăng ký gói cước tại *101# hoặc ứng dụng` |

---

---

## PHẦN 6 — Open Questions cần xác nhận trước khi implement
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
| Q19 | Các trường gửi OB (outbound): VTDĐ vừa gửi danh sách trường thông tin outbound mới — cần đối chiếu với payload/param hiện tại và bổ sung/chỉnh cho khớp. **[Chờ tài liệu VTDĐ]** | Tất cả trigger có gửi OB | 🔴 Cần ngay |
| Q20 | Gói gợi ý (`goi_de_xuat`, `suggested_plan`...): hiện để "CVM NBO tự tính". VTDĐ có nguyên tắc riêng cho gói gợi ý — cần áp nguyên tắc VTDĐ thay vì để CVM tự quyết. **[Chờ nguyên tắc VTDĐ]** | Tất cả trigger có gói gợi ý | 🔴 Cần ngay |
| Q21b | Nguyên tắc nâng gói (`recommendation_direction = UPSIZE`, `suggested_plan`): VTDĐ có nguyên tắc riêng xác định khi nào nâng và nâng lên gói nào — cần áp nguyên tắc VTDĐ. **[Chờ nguyên tắc VTDĐ]** | E_USAGE_NEED_ANALYSIS, U05-A, U05-B | 🔴 Cần ngay |
| Q22 | U05-B-RT: OCS có event realtime khi quota/ngày về 0 để đếm `consecutive_depleted_count` ngay lập tức không? Nếu chỉ có CDR (không có event realtime) thì không thể làm trigger này NearRealtime — phải gộp lại thành báo cáo batch | U05-B-RT | 🔴 Cần ngay |
| Q23 | E_CHURN_RISK: tính điểm/tổ hợp tín hiệu nguy cơ rời mạng ở BSS/OCS hay CVM nội bộ? Ngưỡng cụ thể (số ngày không lưu lượng, % suy giảm doanh thu, tỷ lệ on/off sóng) là bao nhiêu và ai owns? Quan hệ với E_SEGMENT_UPDATE thế nào? | E_CHURN_RISK, E_SEGMENT_UPDATE | 🔴 Cần ngay |
| Q24 | Các trigger vừa chuyển sang NearRealtime (E02, E04, E05, E09, U_PRE_EXPIRY, U_POST_EXPIRY, E_LOCK_1C, E_PRE_LOCK_2C): BSS/OCS/SuperApp có khả năng push realtime theo đúng điều kiện (đủ 24h/72h/x ngày/x phút) không, hay chỉ quét batch được? Cần xác nhận tính khả thi | Các trigger chuyển RT | 🔴 Cần ngay |
| Q25 | E09: X phút (ngưỡng bỏ dở hành trình mua) và cơ chế SuperApp phát hiện "để lại SĐT" là gì? SuperApp track được `journey_start_at` và phân biệt `journey_type` không? | E09 | 🟡 Quan trọng |
| Q26 | U07: BSS phân loại được `swap_type` (4 case) và `is_reissue` (đổi phôi cùng loại) trong sự kiện sim_swap không? | U07 | 🟡 Quan trọng |
| Q27 | E_SEGMENT_UPDATE: phân khúc nghề nghiệp/nhân khẩu (HSSV, tài xế/tự do, CBCNVVP, KOL/KOC, hưu trí) lấy từ nguồn nào — eKYC có đủ để phân loại không? (liên quan Q6) | E_SEGMENT_UPDATE | 🔴 Cần ngay |
