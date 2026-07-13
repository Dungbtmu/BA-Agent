# Điều Kiện Con Từng Trigger — CVM × BSS

> Mục đích: đặc tả điều kiện lọc thêm cho Campaign Builder UI và logic nghiệp vụ chi tiết cho từng trigger.
> Mỗi trigger có 2 phần: **(1) Bảng điều kiện lọc thêm** để Dev render UI filter động + **(2) Logic nghiệp vụ chi tiết** để BA/QA/Dev implement rule engine.
>
> Nguồn tổng hợp: `trigger-integration-summary.md`, `data-contract-template-lifecycle.md`
>
> Cập nhật: 2026-07-13 — đồng bộ toàn bộ bảng điều kiện lọc thêm với danh mục ảnh mới nhất (bổ sung đầy đủ các trường còn thiếu ở mọi trigger; thêm 2 trigger `E_DATA_100` và `E_APP_INACTIVE_X_DAYS`; mở rộng bộ toán tử `>`, `<`, `IS NULL`). Tổng 39 trigger.
> Cập nhật: 2026-07-08 — bổ sung trigger `E_CHURN_RISK` (nguy cơ rời mạng), đồng bộ cột Nguồn dữ liệu với timing NearRealtime đã chốt

---

## Quy ước

**Bảng điều kiện lọc thêm (Campaign Builder UI):**
- **Mức độ: Bắt buộc** — trường phải có giá trị trước khi lưu campaign; không điền → báo lỗi validate
- **Mức độ: Tùy chọn** — user có thể bỏ qua; không điền = không lọc theo điều kiện này
- **Logic mặc định: AND** — các điều kiện kết hợp AND; user đổi OR trong UI nếu cần
- **Toán tử:** `=`, `!=`, `>`, `<`, `>=`, `<=`, `BETWEEN`, `IN`, `NOT IN`, `CONTAINS`, `AFTER`, `BEFORE`, `IS NULL`, `IS NOT NULL`

**Logic nghiệp vụ chi tiết:**

| Ký hiệu | Ý nghĩa |
|---|---|
| **AND** | Tất cả điều kiện phải đúng đồng thời |
| **OR** | Ít nhất một điều kiện đúng |
| **NOT** | Điều kiện phủ định — không được thỏa mãn |
| `field` | Trường dữ liệu cụ thể trong payload hoặc BSS |
| ❓ | Cần xác nhận với PO/Tech trước khi implement |

---

## NHÓM 1 — Kích Hoạt

### E01 — SIM kích hoạt lần đầu (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E01 | lifecycle_number | Số lần kích hoạt | integer | =, !=, >, >= | 1 | Bắt buộc |  | AND | BSS/OCS API Event |
| E01 | activation_date | Ngày giờ kích hoạt | datetime | BEFORE, AFTER, BETWEEN | 2026-06-23 08:30:00 | Bắt buộc |  | AND | BSS/OCS API Event |
| E01 | sim_type | Loại SIM | enum | =, !=, IN, NOT IN | PHYSICAL, ESIM | Tùy chọn | SIM vật lý hay eSIM | AND | BSS/OCS API Event |
| E01 | package_code | Gói cước hiện tại | string | =, !=, IN, NOT IN, IS NULL | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| E01 | segment_age | Phân khúc tuổi | enum | =, IN, NOT IN | 19-24 | Tùy chọn | Chỉ có khi KH đăng ký qua eKYC | AND | BSS/OCS API Event |
| E01 | segment_job | Phân khúc nghề nghiệp | enum | IN | STUDENT, WORKER, OFFICE, DRIVER, OTHER | Tùy chọn | Chỉ có khi KH đăng ký qua eKYC | AND | BSS/OCS API Event |
| E01 | hours_since_activation | Số giờ từ khi kích hoạt | integer | >=, <= | 1899-12-30 00:00:00 | Bắt buộc | Thời gian từ mốc NGAY_0 | AND | BSS/OCS API Event |
| E01 | activation_source | Nguồn kích hoạt | enum | IN | AGENT, ONLINE, ESIM | Tùy chọn | Kênh KH dùng để kích hoạt SIM | AND | BSS/CRM |
| E01 | nationality | Quốc tịch | enum | IN, NOT IN | VN, ROW | Tùy chọn | Để gửi nội dung đúng ngôn ngữ | AND | BSS/CRM eKYC |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `resource.msisdn_status_history.to_state_id` = `ACTIVATED`
- COUNT bản ghi `to_state_id` = `ACTIVATED` theo `msisdn` = `1` (lần đầu tiên duy nhất)
- Trạng thái chuyển xảy ra trong vòng 30–60 phút tính từ thời điểm BSS gọi CVM

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- COUNT bản ghi `to_state_id` = `ACTIVATED` theo `msisdn` > `1` — là SIM tái kích hoạt, không phải KH mới

**Nhánh xử lý:**
- Không có nhánh — chỉ có 1 luồng duy nhất: ghi nhận NGAY_0 nội bộ và gửi USSD chào mừng
- Nội dung cá nhân hóa theo `segment_age`, `segment_job` nếu có — fallback về nội dung chung nếu thiếu

---

### E02 — Chưa cài app sau 24h kích hoạt SIM (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E02 | hours_since_activation | Số giờ từ khi kích hoạt | decimal | >, >=, <, BETWEEN | 24 | Bắt buộc | Đã đủ 24h từ mốc NGAY_0 | AND | BSS API Event |
| E02 | app_installed | Đã cài ứng dụng | boolean | = | FALSE | Bắt buộc | Điều kiện cốt lõi — phải = FALSE | AND | BSS API Event |
| E02 | app_install_time | Thời điểm cài app | datetime | IS NULL, BEFORE, AFTER | NULL | Khuyến nghị |  | AND | BSS/OCS API Event |
| E02 | sim_type | Loại SIM | enum | IN, NOT IN | PHYSICAL, ESIM | Tùy chọn | Lọc riêng cho eSIM vs SIM vật lý | AND | BSS API Event |
| E02 | segment_age | Phân khúc tuổi | enum | IN | 15-18, 19-24, 25-34 | Tùy chọn | Nhắm nhóm tuổi trẻ | AND | BSS API Event |
| E02 | device_type | Loại thiết bị | enum | IN | ANDROID, IOS, FEATURE | Tùy chọn | Gửi hướng dẫn cài app đúng store | AND | BSS API Event |
| E02 | sent_count | Số lần đã gửi trigger | integer | integer | 0 | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Khoảng thời gian từ `activation_date` (NGAY_0) đến thời điểm quét = 24 giờ trở lên
- Không có bản ghi trong `portal.access_app_logs` cho `user_profile_id` tương ứng trong 24h sau kích hoạt
- Không có bản ghi `app_install_log` theo `msisdn` tính đến 02:00 ngày N+1 (chưa cài app)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã có bản ghi `app_install_log` trước 02:00 ngày N+1 (đã cài app)
- Đã có bản ghi `portal.access_app_logs` trong 24h sau kích hoạt (đã mở app qua portal)
- Trigger E02 đã được gửi cho `msisdn` này rồi (không gửi lại)

**Thời điểm quét và gửi:**
- BSS quét lúc 01:00–02:00 ngày N+1
- BSS tạo file CSV và push vào SFTP lúc 02:00–04:00
- CVM gửi USSD lúc 09:00 ngày N+1

---

## NHÓM 2 — Sử Dụng

### E03 — Đăng nhập app lần đầu (Realtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E03 | open_count | Số lần mở app | integer | = | 1 | Bắt buộc |  | AND | BSS/OCS API Event |
| E03 | login_status | Trạng thái đăng nhập | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| E03 | app_version | Phiên bản ứng dụng | string | =, IN, NOT IN | 2.5.2000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E03 | device_os | Hệ điều hành | enum | =, IN | ANDROID, IOS | Tùy chọn |  | AND | BSS/OCS API Event |
| E03 | registration_age_hours | Số giờ từ đăng ký/kích hoạt | integer | <, <=, BETWEEN | 48 | Tùy chọn |  | AND | BSS/OCS API Event |
| E03 | hours_since_activation | Số giờ từ khi kích hoạt SIM | integer | >=, <=, BETWEEN | 24 | Tùy chọn | Lọc theo mốc thời gian từ kích hoạt | AND | SuperApp/Kafka API Event |
| E03 | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| E03 | balance | Số dư TKC tại thời điểm đăng nhập (đồng) | decimal | >=, <= | 50000 | Tùy chọn | KH có tiền → ưu tiên gợi ý mua gói | AND | BSS/OCS API Event |
| E03 | devide_type | Loại thiết bị | enum | IN | ANDROID/IOS | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- SuperApp ghi nhận sự kiện `FIRST_OPEN` cho `msisdn`
- Không có bản ghi `first_open` trước đó cho cùng `msisdn` (lần đầu tiên)
- `is_guest` = `false` — KH đã đăng nhập tài khoản, không phải khách vãng lai

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã có bản ghi `first_open` trước đó → bỏ qua hoàn toàn
- `is_guest` = `true` → CVM bỏ qua trigger, không render banner

**Nhánh xử lý:**
- Không có nhánh — chỉ có 1 luồng: hiển thị banner chào mừng cá nhân hóa tức thì trong phiên

**SLA:** CVM phải phản hồi trong 1–2 giây, nội dung lấy từ cache (đã chuẩn bị sẵn từ E01)

---

### E04 — Cài app nhưng chưa mở sau 24h (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E04 | app_installed | Đã cài app | boolean | = | TRUE | Bắt buộc |  | AND | BSS/OCS API Event |
| E04 | app_open_count | Số lần mở app | integer | =, <=, < | 0 | Bắt buộc |  | AND | BSS/OCS API Event |
| E04 | hours_since_install | Số giờ từ cài đặt | integer | >, >=, BETWEEN | 24 | Tùy chọn | Đã đủ 24h từ khi cài app | AND | SuperApp/BSS API Event |
| E04 | push_token_available | Có push token | boolean | = | TRUE | Khuyến nghị |  | AND | BSS/OCS API Event |
| E04 | sent_count | Số lần đã gửi | integer | = | 0 | Khuyến nghị |  | AND | BSS/OCS API Event |
| E04 | firebase_token | Firebase push token | string | IS NOT NULL | — | Bắt buộc | Phải có token mới gửi được Push Notification | AND | SuperApp/BSS API Event |
| E04 | device_type | Loại thiết bị | enum | IN | ANDROID, IOS | Tùy chọn | Hướng dẫn mở app đúng thiết bị | AND | SuperApp/BSS API Event |
| E04 | segment_age | Phân khúc tuổi | enum | IN | 15-18, 19-24, 25-34 | Tùy chọn | Nhắm nhóm tuổi trẻ | AND | SuperApp/BSS API Event |
| E04 | os_version | Phiên bản hệ điều hành | string | IN, CONTAINS | Android 12+, iOS 15+ | Tùy chọn | Lọc theo phiên bản OS hỗ trợ | AND | SuperApp/BSS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Có bản ghi `app_install_log` theo `msisdn` (đã cài app)
- Không có bản ghi `app_open_log` từ thời điểm `installed_at` đến 02:00 ngày N+1 (chưa mở app trong 24h)
- Khoảng thời gian từ `installed_at` đến thời điểm quét ≥ 24 giờ

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã có bản ghi `app_open_log` trước 02:00 ngày N+1
- Trigger E04 đã được gửi cho `msisdn` này rồi (tối đa 1 lần duy nhất)

**Thời điểm:**
- BSS quét lúc 01:00–02:00, push CSV lúc 02:00–04:00
- CVM gửi Push Notification lúc 08:00–09:00 theo giờ tối ưu phân khúc

---

### E05 — Chưa phát sinh cước sau 72h kích hoạt SIM (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E05 | hours_since_activation | Số giờ từ kích hoạt | integer | >, >=, BETWEEN | 72 | Bắt buộc |  | AND | BSS/OCS API Event |
| E05 | data_usage_mb | Tổng data đã dùng | decimal | =, <=, < | 0 | Bắt buộc | Phải = 0 để trigger | AND | OCS/BSS API Event |
| E05 | voice_usage_sec | Tổng giây thoại | integer | =, <=, < | 0 | Bắt buộc | Phải = 0 để trigger | AND | OCS/BSS API Event |
| E05 | sms_count | Số SMS | integer | =, <=, < | 0 | Bắt buộc | Phải = 0 để trigger | AND | OCS/BSS API Event |
| E05 | charge_amount | Tổng cước phát sinh | decimal | =, <=, < | 0 | Khuyến nghị |  | AND | BSS/OCS API Event |
| E05 | device_type | Loại thiết bị | enum | IN | ANDROID, IOS, FEATURE_PHONE | Tùy chọn | Hướng dẫn sử dụng đúng loại thiết bị | AND | OCS/BSS API Event |
| E05 | has_app | Đã cài app chưa | boolean | = | TRUE, FALSE | Tùy chọn | Có app → push; không có → USSD | AND | OCS/BSS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Khoảng thời gian từ `activation_date` (NGAY_0) đến thời điểm quét ≥ 72 giờ (đúng ngày N+3)
- `data_usage_d1_mb` + `data_usage_d2_mb` + `data_usage_d3_mb` = 0
- `voice_usage_d3_min` = 0
- `sms_count_d3` = 0
- Tổng lưu lượng 3 ngày = 0 (data + thoại + SMS đều bằng 0)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã phát sinh bất kỳ lưu lượng nào dù chỉ 1 SMS hoặc 1 MB data
- Trigger E05 đã được gửi cho `msisdn` này rồi

**Kênh gửi:** Tooltip trong app (chỉ hiển thị khi KH mở app trong ngày N+3)

---

### E06 — Cuộc gọi thất bại (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E06 | call_result | Kết quả cuộc gọi | enum | =, IN | FAILED, BUSY, NO_ANSWER | Bắt buộc |  | AND | BSS/OCS API Event |
| E06 | failure_reason | Mã nguyên nhân lỗi | enum | =, IN, NOT IN | INSUFFICIENT_BALANCE | Tùy chọn |  | AND | BSS/OCS API Event |
| E06 | call_direction | Chiều cuộc gọi | enum | =, IN | OUTGOING | Khuyến nghị |  | AND | BSS/OCS API Event |
| E06 | call_type | Loại cuộc gọi | enum | =, IN | ONNET, OFFNET, INTERNATIONAL | Tùy chọn | Để gợi ý đúng gói bổ sung | AND | BSS/OCS API Event |
| E06 | balance_amount | Số dư tài khoản chính | decimal | <, <=, =, > | 5000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E06 | fail_reason | Lý do thất bại | enum | IN, NOT IN | INSUFFICIENT_BALANCE, POOR_CONNECTION | Tùy chọn | Phân nhánh nội dung theo lý do | AND | BSS/OCS API Event |
| E06 | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- OCS ghi nhận `call_fail_reason` thuộc một trong hai giá trị sau:
  - `HET_TAI_KHOAN` — cuộc gọi bị ngắt do hết số dư
  - `SONG_YEU` — cuộc gọi bị ngắt do sóng yếu

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Lý do thất bại không thuộc `HET_TAI_KHOAN` hoặc `SONG_YEU` (các lý do khác không trigger)
- Đã gửi USSD cho cùng cuộc gọi này rồi (mỗi cuộc gọi chỉ gửi 1 lần)
- E_ZERO_BALANCE đã fire trong vòng 12 giờ — nếu nội dung tin nhắn nạp tiền trùng với E06, bỏ qua E06 ❓ (cần xác nhận window 12h hay 24h — Q17)

**Nhánh xử lý theo `fail_reason`:**
- `HET_TAI_KHOAN` → nội dung gợi ý nạp tiền hoặc tạm ứng; hiển thị `{{so_du_format}}`
- `SONG_YEU` → nội dung hướng dẫn đổi vị trí, kiểm tra SIM; không hiển thị số dư

**SLA:** CVM gửi USSD trong vòng 2 phút sau khi nhận event

---

### E07 — Milestone 7 ngày / Khảo sát ngắn (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E07 | days_since_activation | Số ngày từ kích hoạt | integer | =, BETWEEN | 7 | Bắt buộc | Đúng mốc 7 ngày từ NGAY_0 | AND | SuperApp/Kafka Batch CSV |
| E07 | engagement_score | Điểm gắn kết | decimal | <, <=, >, >=, BETWEEN | 50 | Tùy chọn |  | AND | BSS/OCS API Event |
| E07 | app_open_count_7d | Số lần mở app 7 ngày | integer | <, <=, >, >=, BETWEEN | 3 | Tùy chọn |  | AND | BSS/OCS API Event |
| E07 | usage_days_7d | Số ngày có sử dụng dịch vụ | integer | <, <=, >, >=, BETWEEN | 4 | Tùy chọn |  | AND | BSS/OCS API Event |
| E07 | survey_completed | Đã hoàn thành khảo sát | boolean | = | FALSE | Khuyến nghị | Chưa làm khảo sát → nhắc làm | AND | SuperApp/Kafka Batch CSV |
| E07 | package_code | Mã gói hiện tại | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn | Gợi ý đúng gói trong khảo sát | AND | BSS/OCS Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (OR — điều kiện nào đến trước thì trigger):**
- Điều kiện A: Đúng ngày NGAY_0 + 7 (BSS ghi `trigger_reason` = `DAY_7`)
- Điều kiện B: Số nhiệm vụ hoàn thành `completed_task_count` ≥ 3 (BSS ghi `trigger_reason` = `TASK_3`)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Trigger E07 đã được gửi cho `msisdn` này rồi

**Nhánh xử lý:**
- Gửi Banner khi KH mở app (kênh chính)
- Nếu KH không mở app trong 2 ngày → CVM chuyển kênh sang Push Notification
- Nếu điểm gắn kết ngày 7 < 7 → CVM gửi cảnh báo nội bộ đến CSKH trong 1 giờ

---

### E08 — Data ngày sắp hết (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E08 | quota_type | Loại quota data | enum | =, IN | DAILY_DATA | Bắt buộc | Gói data ngày hay tháng | AND | BSS/OCS API Event |
| E08 | used_percent | Tỷ lệ data đã sử dụng (%) | decimal | >, >=, =, <, BETWEEN | 80 | Bắt buộc |  | AND | BSS/OCS API Event |
| E08 | remaining_data_mb | Dung lượng data còn lại (MB) | decimal | <, <=, =, >, BETWEEN | 512 | Tùy chọn | Lọc theo lượng data còn lại tuyệt đối | AND | BSS/OCS API Event |
| E08 | total_quota_mb | Tổng dung lượng gói (MB) | decimal | >, >=, = | 5120 | Khuyến nghị |  | AND | BSS/OCS API Event |
| E08 | package_code | Mã gói data | string | =, !=, IN, NOT IN | D5 | Tùy chọn | Nhắm gói cụ thể | AND | BSS/OCS API Event |
| E08 | threshold_crossed | Ngưỡng vừa bị vượt | boolean | = | TRUE | Khuyến nghị |  | AND | BSS/OCS API Event |
| E08 | days_remaining | Số ngày còn lại trong chu kỳ | integer | >= | 1 | Tùy chọn | Ưu tiên gửi khi còn nhiều ngày dùng | AND | BSS/OCS API Event |
| E08 | addon_purchased_today | Đã mua gói bổ sung hôm nay | boolean | = | FALSE | Bắt buộc | Không gửi nếu đã mua thêm rồi | AND | BSS/OCS API Event |
| E08 | cycle_date | Ngày chu kỳ quota | date | =, BETWEEN | 2026-06-23 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `daily_data_used_mb / daily_data_quota_mb × 100 ≥ 80` (vượt ngưỡng do CVM cấu hình, mặc định 80%)
- `addon_purchased_today` = `false` (KH chưa mua gói bổ sung hôm nay)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã mua gói bổ sung trong ngày hôm đó (`addon_purchased_today` = `true`)
- Đã gửi USSD từ E08 cho `msisdn` này trong ngày (tối đa 1 lần/ngày)

**Nhánh xử lý:**
- Tất cả trường hợp → gửi USSD gợi ý mua thêm gói; CVM tính NBO

**Ghi chú:** Ngưỡng 80% do CVM cấu hình, không cứng trong OCS. Khi CVM đổi ngưỡng, OCS cập nhật ngưỡng fire event tương ứng mà không thay đổi logic xử lý.

**SLA:** CVM gửi USSD trong vòng 2 giờ sau khi nhận event

---


### E_DATA_100 — Hết 100% data (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_DATA_100 | quota_type | Loại quota | enum | =, IN | DATA | Bắt buộc |  | AND | OCS/BSS API Event |
| E_DATA_100 | remaining_data_mb | Data còn lại (MB) | decimal | =, <=, < | 0 | Bắt buộc |  | AND | OCS/BSS API Event |
| E_DATA_100 | used_percent | Tỷ lệ sử dụng (%) | decimal | =, >=, > | 100 | Khuyến nghị |  | AND | OCS/BSS API Event |
| E_DATA_100 | package_code | Mã gói | string | =, IN, NOT IN | D5 | Tùy chọn |  | AND | OCS/BSS API Event |
| E_DATA_100 | out_of_package_charge | Cho phép phát sinh cước ngoài gói | boolean | = | TRUE | Tùy chọn |  | AND | OCS/BSS API Event |

#### Logic nghiệp vụ chi tiết

_Chưa đặc tả logic chi tiết — bổ sung từ danh mục ảnh, cần PO/Tech hoàn thiện._

---

### E09 — Xem màn hình đổi gói nhưng chưa đăng ký (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E09 | screen_code | Mã màn hình đã xem | string | = | CHANGE_PACKAGE | Bắt buộc |  | AND | BSS/OCS API Event |
| E09 | time_on_screen_sec | Thời gian KH xem màn hình đổi gói (giây) | integer | >=, BETWEEN | 30 | Tùy chọn | Tối thiểu 30 giây mới tính là có intent | AND | SuperApp API Event |
| E09 | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| E09 | total_data_30d_mb | Tổng data dùng 30 ngày (MB) | float | >=, BETWEEN | 40000 | Tùy chọn | KH dùng nhiều → gợi ý gói lớn hơn | AND | BSS/OCS API Event |
| E09 | view_count | Số lần xem màn hình | integer | >, >=, BETWEEN | 1 | Tùy chọn |  | AND | BSS/OCS API Event |
| E09 | package_register_after_view | Có đăng ký sau khi xem | boolean | = | FALSE | Bắt buộc |  | AND | BSS/OCS API Event |
| E09 | hours_since_last_view | Số giờ từ lần xem cuối | integer | >, >=, BETWEEN | 1 | Tùy chọn |  | AND | BSS/OCS API Event |
| E09 | candidate_package_code | Mã gói đã xem | string | =, IN | D10 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- SuperApp ghi nhận `view_screen` = `ThayDoiGoi` cho `msisdn`
- Không có sự kiện `plan_register` trong vòng 10 phút sau `view_screen` (KH thoát không đăng ký)
- BSS ghi nhận vào `change_pkg_view_log`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã đăng ký gói trong session đó (có `plan_register` trong 10 phút)
- Popup E09 đã hiển thị cho `msisdn` này trong ngày hôm đó (tối đa 1 popup/ngày)

**Nhánh xử lý:**
- Tất cả trường hợp → hiển thị popup so sánh gói lần mở app kế tiếp
- Dự phòng: USSD nếu KH không mở app

---

### E11 — Ngày 30 — Tổng kết 1 tháng + Chuyển sang G4 (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E11 | days_since_activation | Số ngày từ kích hoạt | integer | =, BETWEEN | 30 | Bắt buộc | Đúng mốc 30 ngày từ NGAY_0 | AND | BSS/OCS Batch CSV |
| E11 | total_revenue_30d | Doanh thu 30 ngày | decimal | <, <=, >, >=, BETWEEN | 100000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E11 | active_days_30d | Số ngày hoạt động | integer | <, <=, >, >=, BETWEEN | 20 | Tùy chọn |  | AND | BSS/OCS API Event |
| E11 | engagement_score | Điểm gắn kết | decimal | <, <=, >, >=, BETWEEN | 60 | Tùy chọn |  | AND | BSS/OCS API Event |
| E11 | topup_after_score | Số lần nạp tiền trong 30 ngày | integer | >= | 1 | Tùy chọn |  | AND | BSS/OCS API Event |
| E11 | total_data_30d_mb | Tổng data dùng trong 30 ngày (MB) | decimal | >=, BETWEEN | 5120 | Tùy chọn | KH dùng nhiều → upsell gói lớn hơn | AND | BSS/OCS Batch CSV |
| E11 | target_group | Nhóm đích sau đánh giá | enum | =, IN | G4, RETENTION | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Đúng ngày NGAY_0 + 30

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Trigger E11 đã được gửi cho `msisdn` này rồi

**Nhánh xử lý — theo điểm gắn kết ngày 30:**
- Điểm gắn kết cao → phân nhánh sang G4 (upsell, cross-sell)
- Điểm gắn kết thấp → chiến dịch giữ chân

**Nhánh kênh gửi:**
- Chính: Banner in-app toàn màn hình + Email
- Dự phòng: Push Notification nếu không mở app

**Nhánh nội dung App:**
- Nếu có `app_tasks_completed`, `app_points_earned`, `referral_count` → hiển thị đầy đủ số liệu App + viễn thông
- Nếu thiếu dữ liệu App (KH chưa cài app) → fallback nội dung chỉ có số liệu viễn thông

---

### E13 — Tăng đột biến lưu lượng bất thường (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E13 | metric_type | Loại lưu lượng | enum | =, IN | DATA, VOICE, SMS | Bắt buộc |  | AND | BSS/OCS API Event |
| E13 | current_usage | Lưu lượng kỳ hiện tại | decimal | >, >=, BETWEEN | 10240 | Bắt buộc |  | AND | BSS/OCS API Event |
| E13 | baseline_avg_30d | Trung bình 30 ngày | decimal | >, >=, = | 2048 | Bắt buộc |  | AND | BSS/OCS API Event |
| E13 | traffic_spike_mb | Lưu lượng trong giờ đột biến (MB) | float | >=, BETWEEN | 512 | Bắt buộc | Ngưỡng tuyệt đối tối thiểu | AND | BSS/OCS Batch CSV |
| E13 | spike_hour | Giờ xảy ra đột biến (0–23) | integer | >=, <=, BETWEEN | 22 | Tùy chọn | Nhắm giờ cao điểm buổi tối | AND | BSS/OCS Batch CSV |
| E13 | spike_ratio | Tỷ lệ tăng so với baseline | decimal | >, >=, BETWEEN | 3 | Tùy chọn | Tối thiểu 3x mức nền mới trigger | AND | BSS/OCS Batch CSV |
| E13 | device_type | Loại thiết bị | enum | IN | ANDROID, IOS | Tùy chọn | Gợi ý gói phù hợp loại thiết bị | AND | SuperApp/Kafka Batch CSV |
| E13 | min_absolute_delta | Mức tăng tuyệt đối tối thiểu | decimal | >, >= | 1024 | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `traffic_spike_mb / baseline_mb > 3` (lưu lượng trong giờ đột biến > 3 lần mức nền)
- `baseline_mb` được tính từ CDR cùng khung giờ trong 30 ngày gần nhất

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã gửi cảnh báo E13 cho `msisdn` này trong vòng 24 giờ gần nhất
- Mức tăng `spike_ratio` nằm trong pattern bình thường đã được ghi nhận của KH ❓ (cần xác nhận CVM có tracking pattern cá nhân không)

---


### E_APP_INACTIVE_X_DAYS — X ngày KH không truy cập app (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_APP_INACTIVE_X_DAYS | days_since_last_open | Số ngày từ lần mở app cuối | integer | >, >=, =, BETWEEN | 7 | Bắt buộc |  | AND | SuperApp/Kafka Batch CSV |
| E_APP_INACTIVE_X_DAYS | last_open_time | Thời điểm mở app gần nhất | datetime | BEFORE, AFTER, BETWEEN | 2026-06-16 | Khuyến nghị |  | AND | SuperApp/Kafka Batch CSV |
| E_APP_INACTIVE_X_DAYS | app_installed | App còn được cài | boolean | = | TRUE | Tùy chọn |  | AND | SuperApp/Kafka Batch CSV |
| E_APP_INACTIVE_X_DAYS | push_token_available | Có push token | boolean | = | TRUE | Khuyến nghị |  | AND | SuperApp/Kafka Batch CSV |
| E_APP_INACTIVE_X_DAYS | inactive_campaign_sent_count | Số lần đã nhắc | integer | <, <= | 2 | Tùy chọn |  | AND | SuperApp/Kafka Batch CSV |

#### Logic nghiệp vụ chi tiết

_Chưa đặc tả logic chi tiết — bổ sung từ danh mục ảnh, cần PO/Tech hoàn thiện._

---

### E_VOICE_100_ONNET — Hết phút thoại nội mạng (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_VOICE_100_ONNET | quota_type | Loại quota thoại | enum | = | VOICE_ONNET | Bắt buộc | Phân biệt nội/ngoại mạng | AND | BSS/OCS API Event |
| E_VOICE_100_ONNET | remaining_minutes | Số phút còn lại | decimal | =, <=, < | 0 | Bắt buộc | Phải = 0 để trigger | AND | BSS/OCS API Event |
| E_VOICE_100_ONNET | package_code | Mã gói | string | =, IN, NOT IN | V90 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_VOICE_100_ONNET | days_remaining | Số ngày còn lại trong chu kỳ | integer | >= | 1 | Tùy chọn | Còn nhiều ngày → cấp bách hơn | AND | BSS/OCS API Event |
| E_VOICE_100_ONNET | balance | Số dư tài khoản (đồng) | decimal | >= | 10000 | Tùy chọn | KH có đủ tiền mua gói bổ sung | AND | BSS/OCS API Event |
| E_VOICE_100_ONNET | cycle_end_time | Thời điểm kết thúc chu kỳ | datetime | BEFORE, AFTER, BETWEEN | 2026-06-30 23:59:59 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Quota thoại nội mạng (`onnet_quota_min`) về 0 trong gói đang active
- OCS phân biệt được quota nội mạng riêng ❓ (cần xác nhận Q11, Q19)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Gói không tách quota nội/ngoại mạng riêng → dùng event chung với `call_type = BOTH` ❓
- Đã gửi USSD cùng loại trong session hiện tại

**Nhánh xử lý:**
- Nội dung: gợi ý mua gói bổ sung thoại nội mạng; hiển thị số dư tài khoản
- NBO đề xuất gói nội mạng riêng (nếu có)

**SLA:** CVM gửi USSD trong vòng 2 phút

---

### E_VOICE_100_OFFNET — Hết phút thoại ngoại mạng (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_VOICE_100_OFFNET | quota_type | Loại quota thoại | enum | = | VOICE_OFFNET | Bắt buộc | Phân biệt nội/ngoại mạng | AND | BSS/OCS API Event |
| E_VOICE_100_OFFNET | remaining_minutes | Số phút còn lại | decimal | =, <=, < | 0 | Bắt buộc | Phải = 0 để trigger | AND | BSS/OCS API Event |
| E_VOICE_100_OFFNET | package_code | Mã gói | string | =, IN, NOT IN | V90 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_VOICE_100_OFFNET | balance | Số dư tài khoản (đồng) | decimal | >= | 10000 | Tùy chọn | KH có đủ tiền mua gói bổ sung | AND | BSS/OCS API Event |
| E_VOICE_100_OFFNET | out_of_package_rate | Đơn giá ngoài gói | decimal | >, >=, BETWEEN | 1000 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Quota thoại ngoại mạng (`offnet_quota_min`) về 0 trong gói đang active
- OCS phân biệt được quota ngoại mạng riêng ❓ (cần xác nhận Q11, Q19)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Gói không tách quota nội/ngoại mạng riêng → dùng event chung với `call_type = BOTH` ❓
- Đã gửi USSD cùng loại trong session hiện tại

**Nhánh xử lý:**
- Nội dung: cảnh báo cước phát sinh ngoài gói; hiển thị `offnet_rate_per_min` nếu có
- Gợi ý mua gói bổ sung thoại ngoại mạng

**Ghi chú:** Thoại ngoại mạng hết quota nhạy cảm hơn về chi phí (cước ngoài gói cao hơn nội mạng)

**SLA:** CVM gửi USSD trong vòng 2 phút

---

### E_ZERO_BALANCE — Số dư TKC về 0 (Batch — proactive)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_ZERO_BALANCE | account_type | Loại tài khoản | enum | = | MAIN | Bắt buộc |  | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | balance_before | Số dư trước giao dịch | decimal | >, >= | 0 | Khuyến nghị |  | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | balance_after | Số dư sau giao dịch | decimal | =, <=, < | 0 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | transaction_type | Loại giao dịch làm giảm số dư | enum | =, IN | CALL, DATA, SMS, FEE | Tùy chọn |  | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | plan_expiry_date | Ngày hết hạn gói | date | BEFORE, AFTER | 2026-06-30 | Tùy chọn | Cảnh báo gói sắp hết hạn kèm theo | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | topup_count_30d | Số lần nạp trong 30 ngày gần nhất | integer | >=, <= | 2 | Tùy chọn | Phân biệt KH nạp thường xuyên vs hiếm | AND | BSS/OCS API Event |
| E_ZERO_BALANCE | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Số dư tài khoản (`balance`) về 0 do bất kỳ giao dịch nào (data, thoại, đăng ký gói)
- OCS phát hiện qua CDR giao dịch cuối làm cạn kiệt tài khoản
- Phát hiện qua batch nightly OCS → BSS

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- E_ZERO_BALANCE đã fire cho `msisdn` này trong 24 giờ gần nhất

**Phân biệt với E06:**
- E_ZERO_BALANCE: proactive — balance về 0 trước khi cuộc gọi thất bại
- E06: reactive — cuộc gọi đã thất bại rồi mới nhận USSD
- Rule chặn chéo: nếu E_ZERO_BALANCE đã fire trong vòng 12 giờ → E06 không gửi thêm tin nhắn nạp tiền ❓ (window 12h hay 24h cần xác nhận — Q17)

---

### E_CANCEL_PLAN — Hủy gói cước (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_CANCEL_PLAN | action_type | Loại hành động gói | enum | = | CANCEL | Bắt buộc |  | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | package_code | Mã gói bị hủy | string | =, IN, NOT IN | D5 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | cancelled_plan_type | Loại gói vừa hủy | enum | IN | DATA, VOICE, COMBO | Tùy chọn | Phân nhánh nội dung giữ chân | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | balance | Số dư tài khoản (đồng) | decimal | >=, <= | 50000 | Tùy chọn | KH còn tiền → ưu tiên giữ chân | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | subscriber_tenure_days | Số ngày KH đã dùng mạng | integer | >=, <= | 180 | Tùy chọn | Ưu tiên giữ KH lâu năm với ưu đãi tốt hơn | AND | BSS/CRM |
| E_CANCEL_PLAN | cancel_reason | Lý do hủy | enum | =, IN | CUSTOMER_REQUEST | Tùy chọn | Chỉ giữ chân khi KH chủ động hủy | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | auto_renew | Gói có tự động gia hạn | boolean | = | TRUE | Tùy chọn |  | AND | BSS/OCS API Event |
| E_CANCEL_PLAN | days_used | Số ngày đã sử dụng gói | integer | <, <=, >, >=, BETWEEN | 5 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- OCS xác nhận gói cước vừa bị hủy thành công (không phải hết hạn tự nhiên)
- `cancelled_plan_type` thuộc `DATA`, `VOICE`, hoặc `COMBO`

**Điều kiện chặn:** Không có điều kiện chặn mặc định — mọi trường hợp hủy gói thành công đều trigger

**Nhánh xử lý theo `cancelled_plan_type`:**
- `DATA` → nội dung giữ chân tập trung vào gói data; gợi ý gói data thay thế
- `VOICE` → nội dung giữ chân tập trung vào gói thoại; gợi ý gói thoại thay thế
- `COMBO` → nội dung giữ chân tổng hợp; gợi ý gói combo thay thế

**Nhánh phụ theo `cancel_reason` (nếu có):**
- `SWITCH_TO_OTHER` → nội dung giữ chân so sánh lợi ích
- Các lý do khác / không có lý do → nội dung giữ chân mặc định ❓ (cần xác nhận Q13: OCS có tách chủ động hủy vs hệ thống tự hủy không)

**SLA:** CVM gửi USSD/Push trong vòng 5 phút

---

### E_CHURN_RISK — Thuê bao có nguy cơ rời mạng (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_CHURN_RISK | no_usage_days | Số ngày liên tiếp không phát sinh lưu lượng | integer | >=, BETWEEN | 5 | Bắt buộc | Điều kiện cốt lõi — không data/thoại/SMS liên tiếp 5–7 ngày | AND | OCS/BSS API Event |
| E_CHURN_RISK | no_charge_days | Số ngày không phát sinh cước | integer | >=, BETWEEN | 30 | Tùy chọn | Không phát sinh cước trong 30 ngày | AND | OCS/BSS API Event |
| E_CHURN_RISK | revenue_drop_pct | % suy giảm doanh thu so với TB 2 tháng gần nhất | float | >=, BETWEEN | 80 | Tùy chọn | Ngưỡng đề xuất ≥80% suy giảm | AND | OCS/BSS API Event |
| E_CHURN_RISK | sim_on_off_ratio | Tỷ lệ bật/tắt sóng của SIM trong kỳ | float | >=, <= | 0.5 | Tùy chọn | Tỷ lệ cao → SIM ít cắm máy, dấu hiệu rời mạng | AND | HLR/BSS |
| E_CHURN_RISK | churn_risk_level | Mức độ nguy cơ | enum | =, IN | HIGH, MEDIUM | Tùy chọn | Ưu tiên xử lý theo mức độ | AND | BSS/CVM nội bộ |
| E_CHURN_RISK | current_plan | Gói đang dùng (nếu còn) | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn | Nhắm gói cụ thể khi giữ chân | AND | OCS |
| E_CHURN_RISK | subscriber_tenure_days | Tuổi thuê bao (số ngày đã dùng mạng) | integer | >=, <= | 540 | Tùy chọn | Ưu tiên giữ KH lâu năm với ưu đãi tốt hơn | AND | BSS/CRM |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND — tổ hợp tín hiệu, không phải 1 điều kiện đơn lẻ):**
- `no_usage_days` ≥ ngưỡng cấu hình (đề xuất 5–7 ngày) — không phát sinh lưu lượng liên tiếp
- `no_charge_days` ≥ ngưỡng cấu hình (đề xuất 30 ngày) — không phát sinh cước
- `revenue_drop_pct` ≥ ngưỡng cấu hình (đề xuất 80%) — so với doanh thu trung bình 2 tháng gần nhất
- `sim_on_off_ratio` bất thường (SIM tắt sóng nhiều hơn bình thường) — tín hiệu phụ, không bắt buộc phải có đủ dữ liệu này mới trigger

❓ Các ngưỡng cụ thể (số ngày, % suy giảm, tỷ lệ on/off) do CVM cấu hình, cần xác nhận PO/Tech trước khi implement (Q23)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Thuê bao đã ở trạng thái LOCK_1C hoặc LOCK_2C — đã có luồng xử lý riêng (xem nhóm Khóa 1c/2c), không trigger trùng
- Đã gửi chiến dịch giữ chân cho `msisdn` này trong X ngày gần đây (tránh spam liên tục)

**Nhánh xử lý theo `churn_risk_level`:**
- `HIGH` → chiến dịch giữ chân mạnh (ưu đãi lớn, đa kênh: USSD + Push + SMS)
- `MEDIUM` → chiến dịch giữ chân nhẹ (nhắc nhở, ưu đãi nhỏ hơn)

**Quan hệ với E_SEGMENT_UPDATE:** `E_CHURN_RISK` là trigger phát hiện dựa trên tiêu chí hành vi cụ thể ở trên; `E_SEGMENT_UPDATE` là sự kiện cập nhật phân khúc tổng hợp (bao gồm cả CHURN_RISK) sau khi hệ thống đã tính điểm. Có thể dùng `E_CHURN_RISK` làm 1 tín hiệu đầu vào cho `E_SEGMENT_UPDATE`. ❓ Cần xác nhận ai tính điểm churn — BSS/OCS hay CVM nội bộ (Q23)

**SLA:** CVM đưa vào luồng giữ chân (retention) ngay khi nhận event

---

### E_CONTENT_FAIL — Mua dịch vụ nội dung thất bại (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_CONTENT_FAIL | purchase_status | Trạng thái mua dịch vụ | enum | = | FAILED | Bắt buộc |  | AND | BSS/OCS API Event |
| E_CONTENT_FAIL | content_code | Mã dịch vụ nội dung | string | =, IN, NOT IN | MUSIC_PREMIUM | Bắt buộc |  | AND | BSS/OCS API Event |
| E_CONTENT_FAIL | failure_reason | Nguyên nhân thất bại | enum | =, IN | INSUFFICIENT_BALANCE, TIMEOUT | Tùy chọn | Phân nhánh nội dung theo lý do | AND | SuperApp/Kafka API Event |
| E_CONTENT_FAIL | purchase_amount | Giá trị giao dịch | decimal | >, >=, BETWEEN | 10000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_CONTENT_FAIL | balance | Số dư TKC tại thời điểm thất bại (đồng) | decimal | >=, <= | 5000 | Tùy chọn | KH hết tiền → gợi ý nạp thêm | AND | SuperApp/Kafka API Event |
| E_CONTENT_FAIL | content_type | Loại dịch vụ nội dung | enum | IN | MUSIC, VIDEO, GAME, NEWS | Tùy chọn | Gợi ý gói data phù hợp loại content | AND | SuperApp/Kafka API Event |
| E_CONTENT_FAIL | retry_count | Số lần thử lại | integer | >, >=, BETWEEN | 2 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- SuperApp ghi nhận giao dịch mua nội dung thất bại
- `fail_reason` thuộc `INSUFFICIENT_BALANCE` hoặc `NO_DATA_PLAN`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã gửi thông báo E_CONTENT_FAIL cho `msisdn` này trong phiên hiện tại ❓ (cần xác nhận Q15: SuperApp push trực tiếp vào CVM hay qua Kafka → BSS → CSV daily)

**Nhánh xử lý theo `fail_reason`:**
- `INSUFFICIENT_BALANCE` → gợi ý nạp tiền; hiển thị số tiền cần nạp để đủ mua
- `NO_DATA_PLAN` → gợi ý mua gói data; hiển thị gói phù hợp với loại nội dung

**Nhánh phụ theo `content_type`:**
- `GAME` → NBO gợi ý gói data tốc độ cao
- `MUSIC` / `VIDEO` → NBO gợi ý gói data tốc độ trung bình hoặc streaming
- Các loại khác → gợi ý gói data chung

**SLA:** CVM gửi trong vòng 2 phút khi KH còn trong phiên app

---

### E_APP_RATING — KH đánh giá app (NearRealtime — async)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_APP_RATING | rating_score | Điểm đánh giá | integer | =, <=, <, >, >=, BETWEEN | 3 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_APP_RATING | has_comment | Có bình luận | boolean | = | TRUE | Tùy chọn |  | AND | BSS/OCS API Event |
| E_APP_RATING | rating_topic | Chủ đề đánh giá | enum | =, IN | QUALITY, PRICE, UX | Tùy chọn |  | AND | BSS/OCS API Event |
| E_APP_RATING | app_version | Phiên bản ứng dụng | string | =, IN | 2.5.2000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_APP_RATING | rating_count | Số lần đã đánh giá | integer | =, <, > | 1 | Tùy chọn | Ít nhất 1 lần đánh giá | AND | SuperApp/Kafka API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- SuperApp ghi nhận KH submit đánh giá (`APP_RATING_SUBMITTED`)
- `rating_score` thuộc 1–5 (bắt buộc)

**Điều kiện chặn:** Không có điều kiện chặn — mọi submit đánh giá đều trigger (async, không cần phản hồi realtime)

**Nhánh xử lý theo `rating_score` (CVM tự phân loại thành `rating_category`):**
- `1–2` sao → `LOW`: trigger flow chăm sóc + hỏi vấn đề cụ thể; có thể cảnh báo CSKH nội bộ
- `3` sao → `MID`: cảm ơn + gợi ý tính năng chưa khám phá
- `4–5` sao → `HIGH`: cảm ơn + gợi ý review trên App Store / Play Store

---

### E_USAGE_NEED_ANALYSIS — Phân tích nhu cầu gói theo mức sử dụng (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_USAGE_NEED_ANALYSIS | analysis_metric | Chỉ tiêu phân tích | enum | =, IN | DATA, VOICE, SMS, ARPU | Bắt buộc |  | AND | BSS/OCS API Event |
| E_USAGE_NEED_ANALYSIS | depletion_count | Số lần hết data sớm trong 3 tháng | integer | >= | 2 | Tùy chọn | Pattern hết sớm liên tiếp | AND | BSS/OCS Batch CSV |
| E_USAGE_NEED_ANALYSIS | package_code | Gói hiện tại | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn | Nhắm gói cần nâng cấp | AND | BSS/OCS Batch CSV |
| E_USAGE_NEED_ANALYSIS | package_cycle | Chu kỳ gói | enum | IN | MONTHLY, DAILY, WEEKLY | Tùy chọn | Phân tích riêng theo chu kỳ gói | AND | BSS/OCS Batch CSV |
| E_USAGE_NEED_ANALYSIS | avg_usage_30d | Sử dụng trung bình 30 ngày | decimal | <, <=, >, >=, BETWEEN | 10240 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_USAGE_NEED_ANALYSIS | quota_utilization | Tỷ lệ sử dụng quota | decimal | <, <=, >, >=, BETWEEN | 0.85 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_USAGE_NEED_ANALYSIS | out_of_package_charge_30d | Cước ngoài gói 30 ngày | decimal | >, >=, BETWEEN | 50000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_USAGE_NEED_ANALYSIS | recommended_package_code | Mã gói được đề xuất | string | =, IN | D15 | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Đã có đủ dữ liệu tối thiểu: ít nhất một cặp (`avg_data_usage_mb` + `current_plan_data_quota_mb`) hoặc (`avg_voice_usage_min` + `current_plan_voice_quota_min`)
- `usage_to_quota_pct` vượt ngưỡng `HIGH_NEED` hoặc dưới ngưỡng `LOW_NEED` (ngưỡng cụ thể do PO/CVM cấu hình ❓ — Q22)
- `analysis_period_cycles` ≥ giá trị tối thiểu để tính trung bình đủ tin cậy (BSS/CVM cấu hình)

**Điều kiện chặn:** Không có điều kiện chặn mặc định ngoài ngưỡng `usage_to_quota_pct`

**Nhánh xử lý theo `usage_need_segment`:**
- `HIGH_NEED` → tư vấn nâng gói (`recommendation_direction` = `UPSIZE`)
- `LOW_NEED` → tư vấn gói nhỏ hơn / tiết kiệm hơn (`recommendation_direction` = `DOWNSIZE`)

**Nhánh phụ theo `primary_usage_resource`:**
- `DATA` → tập trung vào gói data
- `VOICE` → tập trung vào gói thoại
- `BOTH` → đề xuất gói combo

---

### E_NO_PLAN_X_DAYS — ACTIVE nhưng không có gói x ngày (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_NO_PLAN_X_DAYS | active_package_count | Số gói đang hoạt động | integer | =, <=, < | 0 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_NO_PLAN_X_DAYS | days_without_plan | Số ngày không có gói | integer | >, >=, =, BETWEEN | 7 | Bắt buộc | Số ngày tối thiểu mới gửi nhắc | AND | BSS/OCS Batch CSV |
| E_NO_PLAN_X_DAYS | last_package_code | Gói đăng ký gần nhất | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn | Gợi ý đăng ký lại đúng gói cũ | AND | BSS/OCS Batch CSV |
| E_NO_PLAN_X_DAYS | balance | Số dư tài khoản (đồng) | decimal | >= | 50000 | Tùy chọn | KH có tiền → ưu tiên nhắc đăng ký gói | AND | BSS/OCS Batch CSV |
| E_NO_PLAN_X_DAYS | last_package_end_date | Ngày gói gần nhất hết hạn | date | BEFORE, AFTER, BETWEEN | 2026-06-16 | Khuyến nghị |  | AND | BSS/OCS API Event |
| E_NO_PLAN_X_DAYS | subscriber_status | Trạng thái thuê bao | enum | = | ACTIVE | Bắt buộc | Chỉ trigger khi còn ACTIVE | AND | BSS/OCS Batch CSV |
| E_NO_PLAN_X_DAYS | recent_usage_amount | Mức sử dụng gần đây | decimal | >, >=, BETWEEN | 0 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `crm.subscribers.status` = `ACTIVE`
- Không có gói cước nào đang active (hết tất cả gói)
- Số ngày không có gói (`days_without_plan`) ≥ x (x do CVM cấu hình, mặc định 7 ngày)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã đăng ký gói mới trong ngày hôm nay
- Đã gửi E_NO_PLAN_X_DAYS cho `msisdn` này trong N ngày gần nhất ❓ (tần suất gửi nhắc cần xác nhận với PO)

**Nhánh xử lý:**
- Tất cả trường hợp → gợi ý đăng ký lại gói
- NBO dựa trên `balance` và `last_plan_name`: nếu số dư đủ mua gói cũ → gợi ý gia hạn lại gói cũ; nếu không đủ → gợi ý gói nhỏ hơn phù hợp số dư

---

### E_SEGMENT_UPDATE — Cập nhật phân khúc KH (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_SEGMENT_UPDATE | old_segment | Phân khúc cũ | string | =, IN, NOT IN | MASS | Bắt buộc |  | AND | BSS/OCS API Event |
| E_SEGMENT_UPDATE | new_segment | Phân khúc mới | string | =, IN, NOT IN | HIGH_VALUE | Bắt buộc |  | AND | BSS/OCS API Event |
| E_SEGMENT_UPDATE | segment_changed | Có thay đổi phân khúc | boolean | = | TRUE | Bắt buộc |  | AND | BSS/OCS API Event |
| E_SEGMENT_UPDATE | segment_score | Điểm phân khúc | decimal | >, >=, <, BETWEEN | 80 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_SEGMENT_UPDATE | effective_date | Ngày hiệu lực phân khúc | date | BEFORE, AFTER, BETWEEN | 2026-06-23 | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Phân khúc KH vừa thay đổi (`new_segment` ≠ `previous_segment`) hoặc được phân khúc lần đầu
- BSS hoặc CVM nội bộ hoàn thành tính toán phân khúc theo chu kỳ ❓ (cần xác nhận Q14: BSS tính hay CVM tự tính)

**Điều kiện chặn:** Không có điều kiện chặn nếu BSS push file — CVM xử lý mọi bản ghi

**Nhánh xử lý theo `new_segment`:**
- `CHURN_RISK` → chiến dịch giữ chân khẩn cấp; gợi ý ưu đãi đặc biệt
- `GAMING` → nội dung và gói data tốc độ cao cho gaming
- `ENTERTAINMENT` → nội dung và gói streaming/giải trí
- Các phân khúc khác → nội dung tùy chỉnh theo CVM cấu hình

**Ghi chú:** Nếu CVM tự tính phân khúc nội bộ → file `segment_update_{YYYYMMDD}.csv` không cần BSS push; CVM tự trigger mà không cần dữ liệu từ BSS.

---

### U01 — Nạp tiền thành công (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U01 | topup_status | Trạng thái nạp tiền | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| U01 | topup_count | Số lần nạp thành công | integer | =, >, >= | 2 | Bắt buộc | Tối thiểu lần thứ 2 mới trigger | AND | BSS/OCS API Event |
| U01 | topup_amount | Số tiền nạp | decimal | >, >=, =, BETWEEN | 50000 | Tùy chọn | Lọc theo mức tiền nạp | AND | BSS/OCS API Event |
| U01 | topup_channel | Kênh nạp | enum | =, IN, NOT IN | APP, BANK, RETAIL | Tùy chọn | Gợi ý kênh tiện lợi nhất | AND | BSS/OCS API Event |
| U01 | balance_after | Số dư sau nạp | decimal | >, >=, BETWEEN | 50000 | Tùy chọn | KH có số dư đủ để mua gói | AND | BSS/OCS API Event |
| U01 | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- OCS xác nhận giao dịch nạp tiền thành công (đã credit vào tài khoản)
- `topup_count ≥ 2` (lần nạp thứ 2 trở đi)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- `topup_count = 1` (lần nạp đầu tiên — không trigger gợi ý gói)
- KH đang có gói tháng hiệu lực phù hợp với mức dùng (không cần upsell) ❓ (cần xác nhận CVM có rule này không)

**Nhánh xử lý:**
- Tất cả trường hợp → CVM chạy NBO chọn gói gợi ý; gửi USSD trong vòng 3 phút
- Nội dung thay đổi theo `topup_count` để cá nhân hóa (lần 2, lần 3, lần 4+)

**SLA:** CVM gửi USSD trong vòng 3 phút sau khi nhận event

---

### U02 — Đăng ký / gia hạn gói thành công (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U02 | transaction_type | Loại giao dịch gói | enum | =, IN | REGISTER, RENEW | Bắt buộc | Phân biệt nâng cấp hay gia hạn | AND | BSS/OCS Batch CSV |
| U02 | transaction_status | Trạng thái giao dịch | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| U02 | success_count | Số lần thành công | integer | =, >, >= | 2 | Bắt buộc |  | AND | BSS/OCS API Event |
| U02 | package_code | Mã gói | string | =, IN, NOT IN | D5 | Tùy chọn |  | AND | BSS/OCS API Event |
| U02 | plan_name | Tên gói đã đăng ký | string | IN, NOT IN | GOI_DATA_120K | Tùy chọn | Nhắm gói cụ thể để cross-sell | AND | BSS/OCS Batch CSV |
| U02 | plan_type | Loại gói đã đăng ký | enum | IN | DATA, VOICE, COMBO | Tùy chọn | Phân nhánh nội dung gợi ý bổ sung | AND | BSS/OCS Batch CSV |
| U02 | plan_expiry_date | Ngày hết hạn gói | date | AFTER | 2026-06-30 | Tùy chọn | Gợi ý cross-sell trong thời hạn gói | AND | BSS/CRM |
| U02 | transaction_amount | Giá trị giao dịch | decimal | >, >=, BETWEEN | 50000 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- OCS xác nhận giao dịch đăng ký hoặc gia hạn gói thành công
- `plan_register_count ≥ 2` (lần đăng ký / gia hạn thứ 2 trở đi)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- `plan_register_count = 1` (lần đăng ký đầu tiên)
- KH đã nhận gợi ý cross-sell trong 30 ngày gần đây

**Nhánh xử lý:**
- Bước 1: USSD xác nhận đăng ký gói thành công (gửi lúc 09:00)
- Bước 2: Zalo OA gợi ý dịch vụ bổ sung phù hợp loại gói (gửi lúc 10:00–11:00)

**Nhánh theo `plan_type`:**
- `DATA` → gợi ý gói gia đình hoặc SIM thứ 2 có cùng quota data
- `VOICE` → gợi ý gói thoại bổ sung hoặc SIM thứ 2
- `COMBO` → gợi ý gói addon phù hợp

---

### U03 — Tra cứu số dư — gắn gợi ý inline (Realtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U03 | ussd_code | Mã USSD truy vấn | string | =, IN | *101# | Bắt buộc |  | AND | BSS/OCS API Event |
| U03 | balance_amount | Số dư hiện tại | decimal | <, <=, >, >=, BETWEEN | 10000 | Tùy chọn |  | AND | BSS/OCS API Event |
| U03 | query_count_period | Số lần tra cứu trong kỳ | integer | >, >=, BETWEEN | 3 | Tùy chọn |  | AND | BSS/OCS API Event |
| U03 | package_code | Gói đang dùng | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| U03 | plan_expiry_date | Ngày hết hạn gói | date | BEFORE | 2026-07-05 | Tùy chọn | Nhắc gia hạn khi tra số dư | AND | BSS/OCS API Event |
| U03 | data_used_pct | % data chu kỳ đã dùng | float | >=, BETWEEN | 60 | Tùy chọn | Cảnh báo data sắp hết kèm gợi ý | AND | BSS/OCS API Event |
| U03 | response_channel | Kênh phản hồi | enum | =, IN | USSD | Khuyến nghị |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- KH nhắn `*101#` để tra cứu số dư
- OCS nhận yêu cầu tra cứu

**Điều kiện chặn:** Không có điều kiện chặn — mọi tra cứu `*101#` đều trigger

**Nhánh xử lý:**
- Thành công: CVM phản hồi trong ≤ 2 giây → OCS gắn gợi ý inline vào kết quả USSD
- Timeout: CVM không phản hồi trong 2 giây → OCS trả kết quả tra cứu bình thường, không kèm gợi ý

**SLA nghiêm ngặt nhất:** 2 giây — nếu timeout, OCS KHÔNG được chờ thêm

---

### U04 — Nhận OTP từ app bên thứ 3 (Batch — từ HLR)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U04 | otp_detected | Có phát sinh OTP | boolean | = | TRUE | Bắt buộc |  | AND | BSS/OCS API Event |
| U04 | otp_count | Số lần nhận OTP trong ngày | integer | >= | 1 | Bắt buộc | Ít nhất 1 OTP trong ngày mới trigger | AND | HLR/SMS log Batch CSV |
| U04 | app_category | Danh mục ứng dụng gửi OTP | enum | IN | BANKING, ECOMMERCE, TRANSPORT, FINANCE, SOCIAL | Tùy chọn | Gợi ý gói data phù hợp với app đang dùng | AND | HLR/SMS log Batch CSV |
| U04 | package_code | Gói data hiện tại | string | IN, NOT IN | GOI_DATA_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| U04 | sender_brandname | Brandname gửi OTP | string | =, IN, NOT IN, CONTAINS | GRAB, SHOPEE | Tùy chọn | Nhắm đúng thương hiệu app | AND | HLR/SMS log Batch CSV |
| U04 | otp_count_period | Số OTP trong kỳ | integer | >, >=, BETWEEN | 2 | Tùy chọn | Pattern sử dụng app thường xuyên | AND | HLR/SMS log Batch CSV |
| U04 | period_days | Số ngày quan sát | integer | =, BETWEEN | 30 | Khuyến nghị |  | AND | BSS/OCS API Event |
| U04 | app_category | Nhóm ứng dụng | enum | =, IN | ECOMMERCE, TRANSPORT, FINANCE | Tùy chọn | Gợi ý gói data phù hợp với app đang dùng | AND | HLR/SMS log Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- HLR phát hiện KH nhận SMS chứa từ khóa OTP/mã xác thực
- Đầu số gửi SMS thuộc danh mục đầu số thương mại đã phân loại (banking, entertainment, social)
- `otp_count_today` ≥ 1 (ít nhất 1 lần nhận OTP trong ngày)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã gửi U04 cho `msisdn` này trong ngày (tối đa 1 lần/ngày/`msisdn`)
- KH đang có gói data phù hợp với loại app đã phát hiện

**Nhánh xử lý theo `app_category`:**
- `BANKING` → gợi ý gói data ổn định, tốc độ tốt cho giao dịch ngân hàng
- `ENTERTAINMENT` → gợi ý gói data có quota lớn hoặc gói streaming
- `SOCIAL` → gợi ý gói data ngày phù hợp mạng xã hội
- Các danh mục khác → gợi ý gói data chung

**Ghi chú:** HLR export CSV trực tiếp vào SFTP CVM, không đi qua BSS

---

### U05-A — Hết data tháng sớm 2 tháng liên tiếp (Batch — gói tháng)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U05-A | package_cycle | Chu kỳ gói | enum | = | MONTHLY | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-A | early_depletion_count | Số tháng hết data sớm liên tiếp | integer | >, >=, = | 2 | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-A | depletion_day | Ngày hết data trong chu kỳ | integer | <, <=, BETWEEN | 20 | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-A | remaining_cycle_days | Số ngày còn lại khi hết data | integer | >, >=, BETWEEN | 10 | Tùy chọn |  | AND | BSS/OCS API Event |
| U05-A | quota_type | Loại quota data | enum | IN | MONTHLY | Bắt buộc | Chỉ dành cho gói data tháng | AND | BSS/OCS Batch CSV |
| U05-A | package_code | Mã gói data tháng | string | IN, NOT IN | GOI_DATA_MONTHLY_70K | Tùy chọn |  | AND | BSS/OCS API Event |
| U05-A | avg_monthly_usage_mb | Data trung bình tháng | decimal | >, >=, BETWEEN | 20480 | Tùy chọn | KH dùng nhiều data mới cần nâng gói | AND | BSS/OCS Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND — áp dụng cho gói MONTHLY):**
- Gói hiện tại có hạn mức data cố định theo tháng (gói trả trước tháng, gói combo tháng)
- `month1_depleted_date` < ngày N_CUTOFF của tháng 1 (hết data trước ngày cut-off)
- `month2_depleted_date` < ngày N_CUTOFF của tháng 2 (hết data trước ngày cut-off tháng tiếp)
- N_CUTOFF do CVM cấu hình, mặc định là ngày 25 ❓ (cần xác nhận Q21d — cơ chế truyền ngưỡng từ CVM sang BSS)
- Export vào đầu tháng thứ 3

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Trigger U05-A đã kích hoạt cho `msisdn` này trong 3 tháng gần đây
- KH đã tự nâng gói lên rồi (gói hiện tại đã lớn hơn gói tháng 2)

**Áp dụng:** Chỉ KH gói tháng. Không áp dụng cho gói ngày — xem U05-B.

---

### U05-B — Pattern hết quota data ngày thường xuyên (Batch — gói ngày)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U05-B | package_cycle | Chu kỳ quota | enum | = | DAILY | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-B | depletion_days_count | Số ngày hết quota trong kỳ | integer | >, >=, BETWEEN | 10 | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-B | observation_days | Số ngày quan sát | integer | = | 30 | Bắt buộc |  | AND | BSS/OCS API Event |
| U05-B | depletion_ratio | Tỷ lệ ngày hết quota | decimal | >, >=, BETWEEN | 0.33 | Tùy chọn | Ít nhất 1/3 số ngày bị hết data | AND | BSS/OCS Batch CSV |
| U05-B | daily_quota_mb | Hạn mức data ngày của gói (MB) | integer | >=, <=, BETWEEN | 500 | Tùy chọn | Lọc gói ngày có quota thấp ≤ 500MB | AND | BSS/OCS Batch CSV |
| U05-B | package_code | Gói data ngày đang dùng | string | IN, NOT IN | GOI_DATA_NGAY_10K | Tùy chọn |  | AND | BSS/OCS API Event |
| U05-B | quota_type | Loại quota data | enum | IN | DAILY | Bắt buộc | Chỉ dành cho gói data ngày/quota reset ngày | AND | BSS/OCS Batch CSV |
| U05-B | avg_depletion_hour | Giờ trung bình hết quota | decimal | <, <=, BETWEEN | 18 | Tùy chọn | Hết data trước 14h → pattern rõ ràng | AND | BSS/OCS Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND — áp dụng cho gói DAILY):**
- Gói hiện tại có quota data reset hàng ngày
- `month1_depleted_days` ≥ N_DAYS_THRESHOLD (mặc định 15 ngày/tháng — ❓ cần PO xác nhận Q21b)
- `month2_depleted_days` ≥ N_DAYS_THRESHOLD
- 2 tháng liên tiếp đều thỏa M_MONTHS_THRESHOLD (mặc định 2 tháng — ❓ cần PO xác nhận Q21b)
- Export vào đầu tháng thứ (M+1)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Trigger U05-B đã kích hoạt cho `msisdn` này trong 3 tháng gần đây
- KH đã tự nâng gói lên rồi

**Nhánh xử lý theo `UPSELL_MODE` (CVM cấu hình — mặc định `BOTH` ❓ Q21c):**
- `DAILY_UPGRADE` → chỉ gợi ý gói ngày lớn hơn (`suggested_daily_plan`)
- `MONTHLY_UPSELL` → chỉ gợi ý chuyển sang gói tháng (`suggested_monthly_plan`)
- `BOTH` → đề xuất cả 2 lựa chọn; KH chọn

**Áp dụng:** Chỉ KH gói ngày. Không áp dụng cho gói tháng — xem U05-A.

**Ghi chú kỹ thuật ❓:** Cần xác nhận Q21a — OCS có event riêng khi quota ngày về 0 không? Nếu không thì BSS phải tính từ CDR (compute nặng).

---

### U06 — Chuyển đổi loại gói thành công (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U06 | change_status | Trạng thái chuyển gói | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| U06 | old_package_code | Mã gói cũ | string | =, IN, NOT IN | D5 | Bắt buộc | Để so sánh lợi ích với gói mới | AND | BSS/OCS Batch CSV |
| U06 | new_package_code | Mã gói mới | string | =, IN, NOT IN | D10 | Bắt buộc | Gợi ý addon phù hợp gói mới | AND | BSS/OCS Batch CSV |
| U06 | change_type | Loại chuyển đổi | enum | =, IN | UPGRADE, DOWNGRADE, LATERAL | Tùy chọn |  | AND | BSS/OCS API Event |
| U06 | price_delta | Chênh lệch giá gói | decimal | <, >, =, BETWEEN | 20000 | Tùy chọn | Nâng cấp nhiều → ưu đãi cao hơn | AND | BSS/OCS Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- OCS xác nhận giao dịch đăng ký gói mới thành công
- `old_plan_type` ≠ `new_plan_type` (đổi loại gói, không phải gia hạn cùng loại)
- Ví dụ: `VOICE` → `DATA`, `DATA` → `COMBO`, `COMBO` → `VOICE`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Chuyển gói cùng loại (vd: `DATA` → `DATA`) — không phải đổi loại
- KH đã nhận gợi ý dịch vụ bổ sung trong 7 ngày gần đây

**Nhánh xử lý:**
- Bước 1: USSD xác nhận chuyển gói + so sánh gói cũ/mới (09:00)
- Bước 2: Zalo OA gợi ý dịch vụ bổ sung phù hợp gói mới (10:00–12:00)

**Nhánh nội dung theo chiều chuyển đổi:**
- `VOICE → DATA` → hướng dẫn dùng data; gợi ý APN, ứng dụng tiết kiệm data
- `DATA → VOICE` → hướng dẫn gọi nội/ngoại mạng; gợi ý gói data addon nhỏ
- Các chiều khác → nội dung xác nhận + gợi ý addon liên quan

---

### U07 — Chuyển đổi SIM nội mạng (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U07 | sim_change_status | Trạng thái đổi SIM | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| U07 | old_sim_type | Loại SIM cũ | enum | =, IN | PHYSICAL, ESIM | Bắt buộc | Lọc hướng đổi: vật lý → eSIM hay ngược lại | AND | BSS/CRM Batch CSV |
| U07 | new_sim_type | Loại SIM mới | enum | =, IN | PHYSICAL, ESIM | Bắt buộc | Phân nhánh nội dung eSIM vs SIM vật lý | AND | BSS/CRM Batch CSV |
| U07 | sim_change_reason | Lý do đổi SIM | enum | =, IN | LOST, DAMAGED, ESIM_CONVERSION | Tùy chọn | Phân nhánh nội dung theo lý do đổi | AND | BSS/CRM Batch CSV |
| U07 | is_same_msisdn | Giữ nguyên số thuê bao | boolean | = | TRUE | Khuyến nghị | Giữ số cũ → gợi ý khác so với đổi số | AND | BSS/CRM Batch CSV |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- BSS ghi nhận giao dịch `sim_swap` thành công
- `old_sim_type` ≠ `new_sim_type` (đổi loại SIM)
- Ví dụ: `PHYSICAL` → `ESIM`, `ESIM` → `PHYSICAL`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đổi SIM cùng loại (vd: đổi SIM vật lý lấy SIM vật lý khác với cùng `sim_type`) ❓ (cần xác nhận: có trigger hay không)
- Sự kiện đổi SIM này đã được xử lý rồi (tránh gửi 2 lần cùng `new_iccid`)

**Nhánh xử lý theo `new_sim_type`:**
- `ESIM` → hướng dẫn kích hoạt eSIM; thông báo SIM vật lý cũ hết hiệu lực
- `PHYSICAL` → hướng dẫn lắp SIM vật lý mới; thông báo ngày kích hoạt

**Kênh gửi:**
- Chính: USSD + SMS
- Bổ sung: Push Notification nếu KH đã có app (`firebase_token` không rỗng)

---

### U08 — Gia hạn gói liên tiếp — Vinh danh trung thành (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U08 | renewal_status | Trạng thái gia hạn | enum | = | SUCCESS | Bắt buộc |  | AND | BSS/OCS API Event |
| U08 | consecutive_renewal_count | Số kỳ gia hạn liên tiếp | integer | >, >=, BETWEEN | 3 | Bắt buộc |  | AND | BSS/OCS API Event |
| U08 | package_code | Mã gói | string | =, IN, NOT IN | D10 | Tùy chọn |  | AND | BSS/OCS API Event |
| U08 | total_tenure_days | Số ngày gắn bó | integer | >, >=, BETWEEN | 180 | Tùy chọn |  | AND | BSS/OCS API Event |
| U08 | plan_cycle | Chu kỳ gói được gia hạn liên tiếp | enum | IN | DAILY, WEEKLY, MONTHLY | Tùy chọn | Lọc chuỗi gia hạn theo loại chu kỳ gói — mốc số lần vinh danh có thể khác nhau giữa gói ngày/tuần/tháng | AND | BSS/OCS Batch CSV |
| U08 | renewal_amount | Tổng tiền gia hạn trong chuỗi (đồng) | decimal | >=, BETWEEN | 210000 | Tùy chọn | Vinh danh theo mức chi tiêu | AND | BSS/OCS Batch CSV |
| U08 | subscriber_tenure_days | Tổng ngày gắn kết với mạng | integer | >=, BETWEEN | 90 | Tùy chọn | Kết hợp mốc thời gian gắn kết | AND | BSS/CRM |
| U08 | renewal_channel | Kênh gia hạn | enum | =, IN | AUTO, APP, USSD | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `consecutive_renewals` ≥ x với x là ngưỡng milestone theo `plan_cycle` (CVM cấu hình) ❓ (ngưỡng cụ thể theo từng chu kỳ cần xác nhận Q18)
- Chuỗi gia hạn không bị gián đoạn (dịch vụ phải liên tục, không có khoảng trống hết hạn)
- Tính cả gia hạn đúng hạn và gia hạn sớm trong cùng chuỗi đủ điều kiện
- Chỉ tính chuỗi gia hạn trong cùng 1 `plan_cycle` — không gộp chung số lần gia hạn giữa gói ngày và gói tháng vào cùng 1 chuỗi

**Ngưỡng đề xuất theo `plan_cycle` (BA đề xuất — CVM xác nhận):**
- `DAILY` → ngưỡng số lần cao hơn (VD: 30 lần liên tiếp ≈ 1 tháng dùng gói ngày)
- `WEEKLY` → ngưỡng trung bình (VD: 4 lần liên tiếp ≈ 1 tháng dùng gói tuần)
- `MONTHLY` → ngưỡng thấp hơn, dùng trực tiếp các mốc M3/M6/M12

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã tặng thưởng cùng milestone cho `msisdn` này trong chu kỳ gần đây
- KH đã ở hạng loyalty cao nhất (không có milestone tiếp theo để thăng)
- Chuỗi gia hạn bị gián đoạn (có khoảng trống hết hạn giữa các chu kỳ)

**Nhánh xử lý theo `renewal_pattern`:**
- `ON_TIME_ONLY` → nội dung vinh danh tính kỷ luật; khuyến khích tiếp tục
- `EARLY_ONLY` → nội dung vinh danh tính chủ động; khen ngợi gia hạn sớm
- `MIXED` → nội dung vinh danh chung; linh hoạt

**Nhánh theo `loyalty_milestone` (nếu có):**
- `M3` (3 tháng) → ưu đãi mức cơ bản
- `M6` (6 tháng) → ưu đãi mức trung
- `M12` (1 năm) → ưu đãi cao, có thể thăng hạng loyalty

**Kênh gửi:** Banner App + Email

---

### U09 — Sinh nhật / Kỷ niệm KH hoặc SIM (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U09 | anniversary_type | Loại ngày kỷ niệm | enum | =, IN | BIRTHDAY, SUBSCRIPTION_ANNIVERSARY | Bắt buộc | Phân nhánh nội dung chúc mừng | AND | BSS/CRM Batch CSV |
| U09 | days_to_anniversary | Số ngày đến dịp kỷ niệm | integer | =, BETWEEN | 0 | Bắt buộc |  | AND | BSS/OCS API Event |
| U09 | customer_tenure_days | Số ngày gắn bó | integer | >, >=, BETWEEN | 365 | Tùy chọn | Vinh danh KH lâu năm với ưu đãi đặc biệt | AND | BSS/CRM Batch CSV |
| U09 | date_of_birth | Ngày sinh | date | =, BETWEEN | 1995-06-23 | Tùy chọn | Lọc KH có sinh nhật đúng ngày hôm nay | AND | BSS/CRM eKYC |
| U09 | event_date | Ngày diễn ra sự kiện kỷ niệm | date | = | 2026-06-30 | Tùy chọn | Gửi đúng ngày kỷ niệm | AND | BSS/CRM |
| U09 | activation_date | Ngày kích hoạt | date | =, BETWEEN | 2025-06-23 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (OR — điều kiện nào đúng thì trigger):**
- Điều kiện A: Ngày hiện tại = ngày sinh nhật KH (`crm.customers.birthday`) với `event_type = BIRTHDAY` ❓ (cần xác nhận Q7 — trường `birthday` có tồn tại trong BSS không)
- Điều kiện B: Ngày hiện tại = ngày kỷ niệm N năm kể từ `resource.sims.activation_date` đầu tiên (N = 1, 2, 3...) với `event_type = SIM_ANNIVERSARY`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- Đã gửi lời chúc sinh nhật cho `msisdn` này trong năm nay (tránh gửi 2 lần nếu có lỗi batch)
- `crm.subscribers.status` ≠ `ACTIVE`

**Nhánh xử lý theo `event_type`:**
- `BIRTHDAY` → lời chúc sinh nhật; tặng quà data hoặc điểm thưởng
- `SIM_ANNIVERSARY` → lời chúc kỷ niệm dùng mạng N năm; tặng ưu đãi theo số năm

**Kênh:**
- Chính: Zalo OA (nếu có `zalo_oa_id`)
- Dự phòng: SMS (nếu không có `zalo_oa_id`)

---

### U10 — Ngày lễ / Sự kiện quốc gia (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U10 | holiday_code | Mã ngày lễ/sự kiện | string | =, IN | NATIONAL_DAY | Bắt buộc |  | AND | BSS/OCS API Event |
| U10 | event_date | Ngày diễn ra | date | =, BETWEEN | 2026-09-02 | Bắt buộc | Ngày lễ cụ thể đã cấu hình trong lịch CVM | AND | BSS/CRM Batch CSV |
| U10 | days_before_event | Số ngày gửi trước | integer | =, BETWEEN | 2 | Tùy chọn | Gửi sớm trước ngày lễ bao nhiêu ngày | AND | BSS/CRM Batch CSV |
| U10 | nationality | Quốc tịch | enum | =, IN, NOT IN | VN | Tùy chọn | Gửi nội dung đúng ngôn ngữ và phong tục | AND | BSS/CRM eKYC |
| U10 | region_code | Khu vực | string | =, IN, NOT IN | HN, HCM | Tùy chọn | Nhắm sự kiện địa phương nếu có | AND | BSS/CRM |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Ngày hiện tại nằm trong lịch ngày lễ cố định đã cấu hình trong CVM
- CVM gửi yêu cầu lấy danh sách KH cho BSS trước 2 ngày (BSS export trước ngày lễ D-2)
- `crm.subscribers.status` = `ACTIVE`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- `crm.subscribers.status` ≠ `ACTIVE`
- KH không thuộc phân khúc của ngày lễ đó (ví dụ: nam giới không nhận lời chúc 8/3, 20/10)

**Nhánh xử lý theo loại ngày lễ:**
- Ngày lễ quốc gia chung (30/4, 1/5, 2/9, Tết Nguyên Đán) → gửi toàn bộ KH ACTIVE
- 8/3, 20/10 → chỉ gửi KH nữ (`gender` = `FEMALE`) ❓ (phụ thuộc Q6 — có `gender` trong BSS không)
- Ngày Nhà giáo 20/11 → chỉ gửi KH có `job_segment` = `TEACHER` ❓ (phụ thuộc Q6 — có `job_segment` trong BSS không)
- Ngày Thầy thuốc 27/2 → chỉ gửi KH có `job_segment` = `DOCTOR` / `HEALTHCARE` ❓

**Kênh:**
- Chính: Zalo OA (gửi lúc 08:00 đúng ngày lễ)
- Dự phòng: SMS (nếu không có `zalo_oa_id`)

---

## NHÓM 3 — Gia Hạn

### U_PRE_EXPIRY — Trước khi gói hết hạn x ngày (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U_PRE_EXPIRY | days_to_expiry | Số ngày đến hết hạn | integer | =, <, <=, BETWEEN | 3 | Bắt buộc | Bao nhiêu ngày trước hết hạn → gửi nhắc | AND | BSS API Event |
| U_PRE_EXPIRY | auto_renew | Có tự động gia hạn | boolean | = | TRUE | Tùy chọn |  | AND | BSS/OCS API Event |
| U_PRE_EXPIRY | main_balance | Số dư tài khoản chính | decimal | <, <=, =, >, BETWEEN | 50000 | Tùy chọn |  | AND | BSS/OCS API Event |
| U_PRE_EXPIRY | renewal_fee | Phí gia hạn | decimal | >, >=, = | 50000 | Khuyến nghị |  | AND | BSS/OCS API Event |
| U_PRE_EXPIRY | plan_type | Loại gói | enum | IN | DATA, VOICE, COMBO | Tùy chọn | Phân nhánh nội dung nhắc gia hạn | AND | BSS API Event |
| U_PRE_EXPIRY | package_code | Mã gói sắp hết hạn | string | =, IN, NOT IN | D10 | Bắt buộc | Nhắm gói cụ thể cần gia hạn | AND | BSS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `days_to_expiry` = x với x do CVM cấu hình (thường 3 ngày và 7 ngày → 2 đợt nhắc)
- Vẫn còn gói đang active (`plan_expiry_date` > ngày hiện tại)
- `crm.subscribers.status` = `ACTIVE`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- `auto_renewal_enabled` = `true` (KH đã bật tự động gia hạn — không cần nhắc)
- KH đã gia hạn gói rồi trong ngày hôm nay

**Nhánh theo x ngày còn lại:**
- x = 7 ngày → nhắc nhẹ; nội dung thông tin
- x = 3 ngày → nhắc khẩn; nội dung tạo urgency; hiển thị số dư tài khoản để KH biết có đủ tiền gia hạn không

---

### U_POST_EXPIRY — Sau khi gói hết hạn x ngày chưa gia hạn (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| U_POST_EXPIRY | days_since_expiry | Số ngày từ khi hết hạn | integer | =, >, >=, BETWEEN | 1 | Bắt buộc |  | AND | BSS/OCS API Event |
| U_POST_EXPIRY | renewed_after_expiry | Đã gia hạn lại | boolean | = | FALSE | Bắt buộc |  | AND | BSS/OCS API Event |
| U_POST_EXPIRY | expired_package_code | Mã gói đã hết hạn | string | =, IN | D10 | Bắt buộc |  | AND | BSS/OCS API Event |
| U_POST_EXPIRY | current_active_package_count | Số gói hiện đang active | integer | =, <=, < | 0 | Tùy chọn |  | AND | BSS/OCS API Event |
| U_POST_EXPIRY | recent_usage_after_expiry | Sử dụng sau hết hạn | decimal | >, >=, BETWEEN | 0 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- `days_since_expiry` = x với x do CVM cấu hình (thường 1, 3, 7 ngày → nhiều đợt thúc)
- Chưa có gói mới active (chưa gia hạn)
- `subscriber_status` thuộc `ACTIVE` hoặc `GRACE`

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã gia hạn gói rồi kể từ khi hết hạn
- `renewal_attempts` ≥ ngưỡng tối đa CVM cấu hình (không gửi quá nhiều lần cho cùng 1 KH trong 1 đợt)

**Nhánh theo x ngày quá hạn:**
- x = 1 ngày → nhắc nhẹ; gợi ý gia hạn ngay
- x = 3 ngày → nhắc khẩn; tạo urgency; hiển thị số dư và gợi ý gói phù hợp mức tiền có
- x = 7 ngày → cảnh báo nguy cơ khóa 1 chiều; thúc hành động gấp

---

## NHÓM 4 — Khóa 1c / Khóa 2c

### E_LOCK_2C — Khóa 2 chiều (NearRealtime)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_LOCK_2C | old_status | Trạng thái cũ | enum | =, IN | LOCK_1C, ACTIVE | Bắt buộc |  | AND | BSS/OCS API Event |
| E_LOCK_2C | lock_direction | Chiều khóa | enum | IN | BOTH, INCOMING | Bắt buộc | Xác nhận khóa đúng 2 chiều | AND | BSS/CRM API Event |
| E_LOCK_2C | days_in_lock | Số ngày ở trạng thái khóa 2 chiều | integer | >=, <=, BETWEEN | 1 | Tùy chọn | Nhắc theo số ngày đã bị khóa | AND | BSS/CRM API Event |
| E_LOCK_2C | balance | Số dư TKC khi bị khóa (đồng) | decimal | >=, <= | 0 | Tùy chọn | Hướng dẫn nạp tiền để mở khóa | AND | BSS/OCS API Event |
| E_LOCK_2C | new_status | Trạng thái mới | enum | = | LOCK_2C | Bắt buộc |  | AND | BSS/OCS API Event |
| E_LOCK_2C | lock_reason | Lý do khóa | enum | =, IN | DEBT, KYC, CUSTOMER_REQUEST | Tùy chọn | Phân nhánh nội dung theo lý do khóa | AND | BSS/CRM API Event |
| E_LOCK_2C | lock_time | Thời điểm khóa | datetime | BEFORE, AFTER, BETWEEN | 2026-06-23 10:00:00 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_LOCK_2C | recoverable | Có thể tự khôi phục | boolean | = | TRUE | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- BSS/OCS xác nhận thuê bao chuyển trạng thái sang `LOCK_2C`
- `lock_reason` thuộc `EXPIRY` (hết hạn quá lâu) hoặc `ADMIN` (hệ thống can thiệp)

**Điều kiện chặn:** Không có điều kiện chặn — mọi trường hợp khóa 2 chiều đều trigger ngay

**Nhánh xử lý theo `lock_reason`:**
- `EXPIRY` → hướng dẫn nạp tiền và gia hạn gói để mở khóa; hiển thị số ngày đã hết hạn (`days_since_expiry`)
- `ADMIN` → hướng dẫn liên hệ CSKH hoặc đến điểm giao dịch; không hiển thị số ngày

**Kênh gửi:**
- USSD (bắt buộc — ngay cả khi bị khóa 2 chiều, SMS đến vẫn nhận được)
- Push App (bổ sung nếu có `firebase_token`)

**SLA:** Gửi ngay khi tài khoản bị khóa 2 chiều

---

### E_LOCK_1C — Khóa 1 chiều (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_LOCK_1C | lock_direction | Chiều khóa | enum | =, IN | OUTGOING, INCOMING | Bắt buộc |  | AND | BSS/OCS API Event |
| E_LOCK_1C | lock_reason | Lý do khóa | enum | =, IN | DEBT, INACTIVE, ADMIN | Bắt buộc | Phân nhánh theo nguyên nhân khóa | AND | BSS/OCS API Event |
| E_LOCK_1C | days_in_lock_1c | Số ngày đã khóa 1 chiều | integer | =, >, >=, BETWEEN | 1 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_LOCK_1C | outstanding_amount | Số tiền còn thiếu/nợ | decimal | >, >=, BETWEEN | 50000 | Tùy chọn |  | AND | BSS/OCS API Event |
| E_LOCK_1C | balance | Số dư TKC (đồng) | decimal | >=, <= | 0 | Tùy chọn | KH hết tiền → hướng dẫn nạp tiền | AND | BSS/OCS API Event |
| E_LOCK_1C | days_until_lock_2c | Số ngày còn lại đến khi bị khóa 2 chiều | integer | >=, <=, BETWEEN | 15 | Tùy chọn | Cảnh báo deadline trước khi khóa 2 chiều | AND | BSS/OCS API Event |
| E_LOCK_1C | scheduled_lock_2c_date | Ngày dự kiến khóa 2 chiều | date | BEFORE, AFTER, BETWEEN | 2026-06-30 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- BSS xác nhận thuê bao chuyển trạng thái sang `LOCK_1C` trong ngày
- `lock_scenario` thuộc `SYSTEM_ACTION` hoặc `INACTIVE`

**Điều kiện chặn:** Không có điều kiện chặn mặc định

**Nhánh xử lý theo `lock_scenario`:**
- `INACTIVE` → hướng dẫn kích hoạt lại bằng cách sử dụng dịch vụ; hiển thị `days_inactive` và `days_to_lock_2c`; tạo urgency trước deadline khóa 2 chiều
- `SYSTEM_ACTION` → hướng dẫn liên hệ CSKH để xác nhận; không tiết lộ `lock_reason_detail` cho KH

---

### E_PRE_LOCK_2C — Trước khi khóa 2 chiều x ngày (Batch)

#### Bảng điều kiện lọc thêm (Campaign Builder UI)

| Mã trigger | Tên trường kỹ thuật | Tên điều kiện nghiệp vụ | Kiểu dữ liệu | Toán tử hỗ trợ | Giá trị mẫu / mặc định | Mức độ | Ghi chú nghiệp vụ | Logic mặc định | Nguồn dữ liệu |
|---|---|---|---|---|---|---|---|---|---|
| E_PRE_LOCK_2C | current_status | Trạng thái hiện tại | enum | = | LOCK_1C | Bắt buộc |  | AND | BSS/OCS API Event |
| E_PRE_LOCK_2C | days_to_lock_2c | Số ngày đến khóa 2 chiều | integer | =, <, <=, BETWEEN | 3 | Bắt buộc |  | AND | BSS/OCS API Event |
| E_PRE_LOCK_2C | scheduled_lock_2c_date | Ngày dự kiến khóa 2 chiều | date | BEFORE, AFTER, BETWEEN | 2026-06-26 | Bắt buộc | Gửi trước ngày bị khóa 2 chiều | AND | BSS API Event |
| E_PRE_LOCK_2C | lock_reason | Lý do dự kiến khóa | enum | =, IN | DEBT, INACTIVE | Tùy chọn |  | AND | BSS/OCS API Event |
| E_PRE_LOCK_2C | balance | Số dư TKC (đồng) | decimal | >=, <= | 50000 | Tùy chọn | KH có tiền → hướng dẫn đóng nợ/nạp tiền | AND | BSS API Event |
| E_PRE_LOCK_2C | required_recovery_amount | Số tiền cần bổ sung/thanh toán | decimal | >, >=, BETWEEN | 50000 | Tùy chọn |  | AND | BSS/OCS API Event |

#### Logic nghiệp vụ chi tiết

**Điều kiện kích hoạt (AND):**
- Thuê bao đang ở trạng thái `LOCK_1C`
- `days_to_lock_2c` = x với x do CVM cấu hình (thường 7 ngày và 3 ngày → 2 đợt cảnh báo)
- `lock_2c_scheduled_date` đã được BSS tính (thường `lock_1c_date` + 30 ngày)

**Điều kiện chặn (bất kỳ một điều kiện dưới đây):**
- KH đã mở khóa 1 chiều thành công (trạng thái không còn `LOCK_1C`)
- Đã gửi E_PRE_LOCK_2C cho `msisdn` này cho cùng mốc x ngày rồi

**Nhánh theo x ngày còn lại:**
- x = 7 ngày → cảnh báo sớm; nội dung thông tin; hướng dẫn mở khóa
- x = 3 ngày → cảnh báo khẩn cấp; nội dung tạo urgency cực cao; hiển thị ngày deadline cụ thể

**Nhánh theo `lock_scenario`:**
- `INACTIVE` → hướng dẫn kích hoạt lại bằng cách sử dụng dịch vụ hoặc đăng ký gói
- `SYSTEM_ACTION` → hướng dẫn liên hệ CSKH trước deadline

---

## Tổng Hợp Rule Chặn Chéo Giữa Các Trigger

| Trigger A | Trigger B | Rule chặn chéo |
|---|---|---|
| E_ZERO_BALANCE | E06 | Nếu E_ZERO_BALANCE đã fire trong 12h → E06 không gửi thêm tin nhắn nạp tiền ❓ (window 12h hay 24h — Q17) |
| U_PRE_EXPIRY | U_POST_EXPIRY | Không chặn nhau — hai trigger độc lập (trước và sau hạn) |
| E_LOCK_1C | E_PRE_LOCK_2C | E_PRE_LOCK_2C chỉ fire khi KH đang ở `LOCK_1C` — bổ sung cho nhau, không chặn |
| E_PRE_LOCK_2C | E_LOCK_2C | Sau khi E_LOCK_2C fire → E_PRE_LOCK_2C không còn ý nghĩa; CVM tự dừng |
| E07 (Milestone D7) | E11 (Tổng kết D30) | Không chặn nhau — 2 mốc thời gian khác nhau |
| U05-A | U05-B | Loại trừ nhau theo loại gói: U05-A chỉ cho gói tháng; U05-B chỉ cho gói ngày |

---

## Open Questions Ảnh Hưởng Trực Tiếp Đến Điều Kiện Con

| Mã | Câu hỏi | Trigger bị ảnh hưởng | Mức độ |
|---|---|---|---|
| Q11 | OCS có push event riêng khi quota thoại về 0, hay chỉ ghi trong CDR? | E_VOICE_100_ONNET, E_VOICE_100_OFFNET | 🔴 Cần ngay |
| Q12 | E_ZERO_BALANCE: OCS push event realtime hay BSS phát hiện qua CDR đêm? | E_ZERO_BALANCE | 🔴 Cần ngay |
| Q13 | OCS có phân biệt KH chủ động hủy vs hệ thống tự hủy không? | E_CANCEL_PLAN | 🟡 Quan trọng |
| Q14 | Phân khúc CHURN_RISK, GAMING do BSS tính hay CVM tự tính? | E_SEGMENT_UPDATE | 🔴 Cần ngay |
| Q15 | SuperApp push E_CONTENT_FAIL trực tiếp vào CVM hay qua Kafka → BSS → CSV? | E_CONTENT_FAIL | 🔴 Cần ngay |
| Q17 | Rule chặn E_ZERO_BALANCE vs E06: window 12h hay 24h? Ai owns logic chặn? | E_ZERO_BALANCE, E06 | 🟡 Quan trọng |
| Q18 | Ngưỡng `consecutive_renewals` theo từng milestone và theo từng `plan_cycle` (ngày/tuần/tháng) là bao nhiêu; gia hạn sớm tính như thế nào? | U08 | 🟡 Quan trọng |
| Q19 | OCS có tách quota thoại nội/ngoại mạng riêng không? | E_VOICE_100_ONNET, E_VOICE_100_OFFNET | 🔴 Cần ngay |
| Q21a | OCS có event riêng khi quota data/ngày về 0 không? | U05-B | 🔴 Cần ngay |
| Q21b | Ngưỡng N_DAYS_THRESHOLD và M_MONTHS_THRESHOLD cho U05-B? | U05-B | 🟡 Quan trọng |
| Q21c | UPSELL_MODE mặc định (DAILY_UPGRADE / MONTHLY_UPSELL / BOTH)? | U05-B | 🟡 Quan trọng |
| Q22 | Ngưỡng HIGH_NEED / LOW_NEED cho E_USAGE_NEED_ANALYSIS? Owner tính phân khúc? | E_USAGE_NEED_ANALYSIS | 🔴 Cần ngay |
| Q23 | Tính điểm/tổ hợp tín hiệu nguy cơ rời mạng ở BSS/OCS hay CVM nội bộ? Ngưỡng cụ thể (số ngày không lưu lượng, % suy giảm doanh thu, tỷ lệ on/off sóng) là bao nhiêu? Quan hệ với E_SEGMENT_UPDATE thế nào? | E_CHURN_RISK, E_SEGMENT_UPDATE | 🔴 Cần ngay |
| Q6 | `gender`, `age_segment`, `job_segment` lấy từ đâu trong BSS? | U10 | 🔴 Cần ngay |
| Q7 | Trường `birthday` trong `crm.customers` có tồn tại không? | U09 | 🟡 Quan trọng |
