# CVM Data Requirements — Bảng tổng hợp dữ liệu CVM cần

> Bảng này tổng hợp **toàn bộ dữ liệu CVM cần**, gom theo nhóm thông tin — không phân theo trigger.
> Mục đích: nhìn nhanh CVM cần gì, lấy từ đâu, bảng nào, và có sẵn chưa.
>
> Cập nhật: 2026-05-14

**Chú thích trạng thái:**
- ✅ **Có sẵn** — BSS đã có, lấy được ngay
- ⚠️ **Cần nhận** — dữ liệu tồn tại ở nguồn khác, BSS/CVM phải thiết lập cơ chế nhận
- ❌ **Chưa có** — chưa rõ nguồn hoặc chưa có schema, cần clarify với PO/Tech

**Chú thích cơ chế:**
- **Push Event (API)** — nguồn gọi API CVM ngay khi sự kiện xảy ra (NearRealtime)
- **Push CSV (SFTP)** — nguồn tạo file CSV và đẩy vào SFTP CVM đọc (Offline/Batch)
- **Direct** — BSS query trực tiếp khi cần (không cần push)

---

## Nhóm 1 — Định danh & Trạng thái Thuê bao

> Dữ liệu nền tảng. Trigger nào cũng cần. CVM dùng để xác định "KH này là ai, SIM này trạng thái gì."

| Thông tin | Nguồn | Bảng BSS | Trường BSS | Cơ chế CVM nhận | Trạng thái | Trigger dùng |
|---|---|---|---|---|---|---|
| Số điện thoại (MSISDN) | 🟡 BSS | `crm.subscribers` | `msisdn` | Push Event / Push CSV | ✅ Có sẵn | Tất cả |
| Mã khách hàng | 🟡 BSS | `crm.subscribers` | `customer_id` | Push Event / Push CSV | ✅ Có sẵn | E01, U01–U10 |
| Trạng thái thuê bao | 🟡 BSS | `crm.subscribers` | `status` | Push CSV | ✅ Có sẵn | U10 (lọc ACTIVE) |
| Ngày kích hoạt SIM (NGAY_0) | 🟡 BSS | `resource.sims` | `activation_date` | Push Event (E01) | ✅ Có sẵn | E01, E02, E05, E07, E11, U09 |
| Loại SIM | 🟡 BSS | `resource.sims` | `sim_type` | Push Event / Push CSV | ✅ Có sẵn | E01, E02, U07 |
| Mã ICCID | 🟡 BSS | `resource.sims` | `iccid` | Push Event | ✅ Có sẵn | E01, U07 |
| Số lần kích hoạt SIM | 🟡 BSS | `crm.subscriber_lifecycles` | `lifecycle_number` | Push Event | ✅ Có sẵn | E01 (điều kiện chặn: lifecycle > 1) |
| Ngày hết hạn gói (BSS side) | 🟡 BSS | `resource.msisdns` | `expiry_date` | Push CSV | ✅ Có sẵn | U02, U03, U06 |
| Đã chuyển mạng vào (port-in) | 🟡 BSS | `resource.msisdns` | `is_ported`, `port_in_date` | Push CSV | ✅ Có sẵn | Trigger PORT_IN |

---

## Nhóm 2 — Thông tin Cá nhân Khách hàng

> Dùng để cá nhân hóa nội dung (tên, email) và phân khúc thông điệp (giới tính, nghề nghiệp, tuổi). Một phần BSS đã có, một phần chưa có schema.

| Thông tin | Nguồn | Bảng BSS | Trường BSS | Cơ chế CVM nhận | Trạng thái | Trigger dùng |
|---|---|---|---|---|---|---|
| Họ tên KH | 🟡 BSS | `crm.customers` | `full_name` | Push Event / Push CSV | ✅ Có sẵn | E01, E03, E07, E11, U08, U09, U10 |
| Email KH | 🟡 BSS | `crm.customers` | `contact_email` | Push CSV | ✅ Có sẵn | E11, U08, U09 |
| Zalo OA ID | 🟡 BSS | `crm.customers`? | `zalo_oa_id`? | Push CSV | ⚠️ Cần xác nhận trường tồn tại | U09, U10 |
| Ngày sinh | ❓ Chưa rõ | `crm.customers`? | `birthday`? | Push CSV | ❌ Chưa có — không thấy trong schema | U09 |
| Giới tính | ❓ Chưa rõ | `crm.customers`? | `gender`? | Push CSV | ❌ Chưa có — cần cho 8/3, 20/10 | U10 |
| Nghề nghiệp | ❓ Chưa rõ | — | — | Push CSV | ❌ Chưa có — cần cho ngày Nhà giáo, Thầy thuốc | E01–E04, U10 |
| Phân khúc tuổi | ❓ Chưa rõ | — | — | Push CSV | ❌ Chưa có — cần để chọn giờ gửi tối ưu | E02, E04 |

---

## Nhóm 3 — Dữ liệu Tài khoản & Gói cước

> Toàn bộ nhóm này thuộc OCS. BSS không có realtime. CVM nhận qua 2 cơ chế: Push Event (NearRealtime khi sự kiện xảy ra) hoặc Push CSV (batch nightly cho dữ liệu tổng hợp).

| Thông tin | Nguồn | Bảng OCS | Trường | Cơ chế CVM nhận | Trạng thái | Trigger dùng |
|---|---|---|---|---|---|---|
| Số dư tài khoản hiện tại | 🔴 OCS | — | `balance` | Push Event (E06, U01, U03) | ⚠️ Cần OCS push | E06, U01, U03 |
| Tên gói đang dùng | 🔴 OCS | — | `current_plan` | Push Event / Push CSV | ⚠️ Cần OCS push | E01, E03, E08, E09, E11, U01–U06, U09 |
| Loại gói (DATA / VOICE / COMBO) | 🔴 OCS | — | `plan_type` | Push CSV | ⚠️ Cần OCS push | U02, U06 |
| Ngày hết hạn chu kỳ gói (OCS side) | 🔴 OCS | — | `plan_expiry_date` | Push CSV | ⚠️ Cần OCS push | U02 |
| Hạn mức data ngày (quota) | 🔴 OCS | — | `daily_data_quota_mb` | Push Event | ⚠️ Cần OCS push | E08 |
| Data đã dùng trong ngày | 🔴 OCS | — | `daily_data_used_mb` | Push Event | ⚠️ Cần OCS push | E08 |
| % data ngày đã dùng | 🔴 OCS | — | Tính: used/quota | Push Event | ⚠️ OCS tính hoặc CVM tự tính | E08 |
| Số ngày còn lại trong chu kỳ | 🔴 OCS | — | `days_remaining` | Push Event | ⚠️ Cần OCS push | E08 |
| % data chu kỳ đã dùng | 🔴 OCS | — | `data_used_pct` | Push Event | ⚠️ Cần OCS push | U03 |
| Đã mua gói bổ sung hôm nay | 🔴 OCS | — | `addon_purchased_today` | Push Event | ⚠️ Cần OCS push | E08 (điều kiện chặn) |
| Lý do cuộc gọi thất bại | 🔴 OCS | — | `fail_reason` | Push Event | ⚠️ Cần OCS push | E06 |
| Số tiền nạp | 🔴 OCS | — | `topup_amount` | Push Event | ⚠️ Cần OCS push | U01 |
| Số dư sau khi nạp | 🔴 OCS | — | `balance_after` | Push Event | ⚠️ Cần OCS push | U01 |
| Tổng số lần nạp tiền | 🔴 OCS | — | `topup_count` | Push Event / Push CSV | ⚠️ Cần OCS push | U01, U02 |
| Tổng số lần đăng ký gói | 🔴 OCS | — | `plan_register_count` | Push CSV | ⚠️ Cần OCS push | U02 |
| Gói cũ (trước khi chuyển) | 🔴 OCS | — | `old_plan`, `old_plan_type` | Push CSV | ⚠️ Cần OCS push | U06 |
| Gói mới (sau khi chuyển) | 🔴 OCS | — | `new_plan`, `new_plan_type` | Push CSV | ⚠️ Cần OCS push | U06 |
| Ngày hết data tháng 1 & 2 | 🔴 OCS | — | `data_depleted_date` | Push CSV (đầu tháng 3) | ⚠️ Cần OCS push | U05 |
| Gói đề xuất nâng lên | 🔴 OCS + CVM | — | `suggested_plan` | Push CSV hoặc CVM tự tính | ⚠️ Cần confirm ai tính | U05 |
| Số lần gia hạn đúng hạn liên tiếp | 🟡 BSS + 🔴 OCS | `resource.msisdns` | `expiry_date` (lịch sử) | Push CSV | ⚠️ BSS tính từ lịch sử OCS | U08 |
| Gói được giữ nguyên sau đổi SIM | 🔴 OCS | — | `current_plan` | Push CSV | ⚠️ Cần OCS push | U07 |

---

## Nhóm 4 — Hành vi Ứng dụng (SuperApp)

> Toàn bộ nhóm này từ SuperApp. Cơ chế: SuperApp push event vào Kafka → BSS consume và lưu vào các bảng log → BSS tổng hợp và push CSV cho CVM theo batch nightly. **Chưa xác nhận BSS có consume Kafka từ SuperApp không.**

| Thông tin | Nguồn | Bảng BSS (dự kiến) | Trường | Cơ chế CVM nhận | Trạng thái | Trigger dùng |
|---|---|---|---|---|---|---|
| Đã cài app chưa | 🔵 SuperApp | `app_install_log` | — | Push CSV (kiểm tra bản ghi) | ⚠️ Cần BSS consume Kafka | E02 |
| Thời điểm cài app | 🔵 SuperApp | `app_install_log` | `installed_at` | Push CSV | ⚠️ Cần BSS consume Kafka | E04 |
| Đã mở app chưa | 🔵 SuperApp | `app_open_log` | — | Push CSV (kiểm tra bản ghi) | ⚠️ Cần BSS consume Kafka | E04 |
| Lần mở app đầu tiên (first_open) | 🔵 SuperApp | — | `first_open` event | Push Event (API trực tiếp vào CVM) | ⚠️ Cần SuperApp push trực tiếp | E03 |
| Loại thiết bị (Android/iOS) | 🔵 SuperApp | `app_install_log` | `device_type` | Push CSV | ⚠️ Cần BSS consume Kafka | E01, E04, E05, E13 |
| Tên thiết bị | 🔵 SuperApp | `app_install_log` | `device_name` | Push CSV | ⚠️ Cần BSS consume Kafka | E01, E02 |
| Phiên bản hệ điều hành | 🔵 SuperApp | `app_install_log` | `os_version` | Push CSV | ⚠️ Cần BSS consume Kafka | E03, E04 |
| Firebase token | 🔵 SuperApp | `app_install_log` | `firebase_token` | Push CSV | ⚠️ Cần BSS consume Kafka | E04, E07, U07 |
| Hành vi xem màn hình đổi gói | 🔵 SuperApp | `change_pkg_view_log` | `view_timestamp`, `time_on_screen`, `view_count` | Push CSV (qua Kafka → BSS) | ⚠️ Cần BSS consume Kafka | E09 |
| Số nhiệm vụ hoàn thành | 🔵 SuperApp | `task_log` | `completed_task_count` | Push CSV (qua Kafka → BSS) | ⚠️ Cần BSS consume Kafka | E07 |

---

## Nhóm 5 — Dữ liệu Lịch sử Tổng hợp

> Không có sẵn — BSS/OCS phải tính toán, tổng hợp rồi mới push cho CVM. CVM dùng để chạy NBO model, ARPU model, phân nhánh G4, tính điểm gắn kết.

| Thông tin | Nguồn gốc | Ai tổng hợp | Cơ chế CVM nhận | Trạng thái | Trigger dùng |
|---|---|---|---|---|---|
| Tổng data D1–D3 từ NGAY_0 (MB) | 🔴 OCS | BSS tính từ batch OCS | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E05 |
| Tổng data + thoại + SMS D1–D7 | 🔴 OCS | BSS tổng hợp | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E07 |
| Tổng data D1–D30 (GB) | 🔴 OCS | BSS tổng hợp | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E11 |
| Tổng thoại D1–D30 (phút) | 🔴 OCS | BSS tổng hợp | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E11 |
| Số lần đổi gói trong 30 ngày | 🔴 OCS | BSS đếm từ log OCS | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E11 |
| Lịch sử lưu lượng 30 ngày (MB) | 🔴 OCS | BSS tổng hợp | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E09 |
| Tổng data 2 tháng liên tiếp (GB) | 🔴 OCS | BSS tổng hợp theo chu kỳ | Push CSV (đầu tháng 3) | ⚠️ Cần OCS → BSS batch nightly | U05 |
| Mức nền traffic (baseline) theo khung giờ | 🔴 OCS + 🟡 CDR | BSS tính từ CDR 30 ngày | Push CSV | ⚠️ BSS có CDR, cần tính baseline | E13 |
| Traffic giờ đột biến (spike) | 🔴 OCS | BSS so với baseline | Push CSV | ⚠️ Cần OCS → BSS batch nightly | E13 |
| Điểm gắn kết ngày 7 / ngày 30 | 🔴 OCS + 🔵 SuperApp | CVM tự tính từ nhiều nguồn | — | ⚠️ CVM tự tính — cần định nghĩa công thức | E07, E11 |
| Điểm trung thành / hạng KH | ❓ Chưa rõ | — | — | ❌ Chưa có schema | U08, U09 |
| Loại ứng dụng phát OTP (phân loại) | ⚪ HLR | HLR export trực tiếp | Push CSV vào CVM (không qua BSS) | ✅ Cơ chế đã rõ — HLR → SFTP CVM | U04 |
| Tần suất nhận OTP từng app | ⚪ HLR | HLR tổng hợp | Push CSV vào CVM | ✅ Cơ chế đã rõ | U04 |

---

## Tổng hợp nhanh theo trạng thái

| Trạng thái | Số lượng trường | Hành động cần làm |
|---|---|---|
| ✅ **Có sẵn trong BSS** | 9 trường (Nhóm 1) + 3 trường (Nhóm 2) = **~12 trường** | Confirm schema, lập Data Contract |
| ⚠️ **Cần thiết lập cơ chế nhận** | ~35 trường (OCS + SuperApp) | Confirm với Tech Lead: Push Event API hoặc Push CSV/Kafka |
| ❌ **Chưa có nguồn — cần clarify** | 6 trường | Hỏi PO: `birthday`, `gender`, `job`, `age_segment`, `loyalty_point`, `loyalty_tier` |
