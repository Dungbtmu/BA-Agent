# Đánh giá trigger CVM theo CDR BSS mẫu 30/09/2026

> Nguồn: `Trigger-CVM (4).xlsx` và 7 file CDR trong `.claude/input/DB BSS/CDR_30092026/`. Nguyên tắc chấm: field CVM tự tính được từ CDR thì không tính là thiếu. Bản Excel đi kèm: `cdr-trigger-feasibility.xlsx`.

## Tóm tắt

| Mức | Số mục |
|---|---|
| A. Làm được | 12 |
| B. Làm được nhờ suy luận | 8 |
| C. Chưa làm được | 14 |
| D. Ngoài phạm vi CDR | 8 |
| Tổng | 42 (38 trigger và 4 tiêu chí Thuê bao rời mạng) |

## Giả định và lưu ý

- Mẫu CDR chỉ là 1 ngày (30/09/2026), 78 thuê bao Pottel, tên file có chữ THU_NGHIEM. Chưa có ca dương tính cho hầu hết trigger.
- Kết nối thực tế qua Kafka của Cloud-CNTT, CVM là consumer. File CSV chỉ là mẫu về nội dung, không phải cách truyền. Tên trường và độ trễ trong message Kafka có thể khác.
- Ý nghĩa các file Thuebao_* và các trường TYPE, DISCONNECTIONCODE, ACCUMULATED_USAGE, PERCENT_USAGE là suy luận của BA, chưa được CNTT xác nhận.
- Field "tự tính" nghĩa là CVM tính từ dữ liệu tích lũy trong CVM, không yêu cầu nguồn cấp thêm.

## Mô tả bộ CDR mẫu

| File | Số dòng | Nội dung và nhận xét |
|---|---|---|
| `Thuebao_PTM_VNPOST_20260930.csv` | 3 | Kích hoạt thuê bao mới (Idle sang Active). SUBSCRIBER_ID = MSISDN bỏ "84". Chỉ có ngày. |
| `Thuebao_DK_GOI_VNPOST_20260930.csv` | 3 | Đăng ký gói (PT_DL200_0K). Trùng hoàn toàn 3 thuê bao PTM. TYPE=1 chưa rõ nghĩa. |
| `Thuebao_GOI_MOI_VNPOST_20260930.csv` | 7 | Gói POTTEL_BIG_1MB, ACCUMULATED_USAGE 1024–1025, PERCENT_USAGE 100%. Cả 7 thuê bao có phát sinh data. |
| `Thuebao_HUY_GOI_VNPOST_20260930.csv` | 0 | Hủy gói. Chỉ có header, không có dữ liệu. |
| `Thuebao_HUY_VNPOST_20260930.csv` | 9 | Ngắt thuê bao (DISCONNECTIONCODE=3). Không có thuê bao nào phát sinh data trong ngày. |
| `THU_NGHIEM_VNPOST_DATA_20260930.csv` | 4516 | 4143 eG-CDR và 373 S-CDR, 78 thuê bao. 7 cột rỗng hoàn toàn (ID, CHANGE_TIME, REC_ENTITY_ID, ERR_ID, UPLOAD_TIME, CHARGING_ID, RECORD_SEQUENCE_NUMBER). Chỉ có ngày, không có giờ. 825 dòng có DURATION=3600. |
| `THU_NGHIEM_VNPOST_SMS_20260930.csv` | 16 | Toàn bộ SMS gửi tới đầu số 900 (REC_TYPE=SMT), 9 thuê bao. Duy nhất file có giờ-phút-giây. |

## A. Làm được

| Mã | Tên | File CDR dùng | CVM tự tính thế nào | Giả định và rủi ro | Điều kiện tiên quyết |
|---|---|---|---|---|---|
| E01 | SIM kích hoạt lần đầu | Thuebao_PTM | lifecycle_number = số lần MSISDN chuyển Idle sang Active trong lịch sử PTM. hours_since_activation = giờ nhận event trừ ngày kích hoạt. | PTM chỉ ghi kích hoạt mới. PTM chỉ có ngày, không có giờ, nên SLA 30–60 phút cần timestamp từ message Kafka. 3 thuê bao PTM mẫu chưa có SMS tới 900 để kiểm chứng điều kiện "gọi 900". | Khai báo luồng PTM qua Kafka kèm timestamp. |
| E_DATA_100 | Hết 100% data | Thuebao_GOI_MOI, Thuebao_DK_GOI | Fire khi PERCENT_USAGE = 100%. remaining_data_mb = hạn mức trừ ACCUMULATED_USAGE. addon_purchased_today lấy từ DK_GOI. | Giả định GOI_MOI là snapshot mức dùng gói. 7 thuê bao mẫu đã dùng 108 MB đến 1,8 GB trong ngày nên cần xác nhận đơn vị và ý nghĩa 1024. | CNTT xác nhận ý nghĩa GOI_MOI. |
| E_CANCEL_PLAN | Hủy gói cước | Thuebao_HUY_GOI | package_code = SERVICE_CODE. action_type = TYPE. Thời điểm = LOGDATE. Số dư và lý do hủy là tùy chọn. | File mẫu rỗng nên chưa kiểm chứng được. Chưa rõ giá trị TYPE. | Xin mẫu HUY_GOI có dữ liệu và bảng giải nghĩa TYPE. |
| E05 | Chưa phát sinh cước sau 72h tính từ thời điểm kích hoạt thành công | Thuebao_PTM, THU_NGHIEM_..._DATA, THU_NGHIEM_..._SMS | NGAY_0 từ PTM. data_usage_mb = tổng DATA trong 3 ngày. sms_count từ file SMS. | Dùng rule rút gọn "data + SMS = 0". Thiếu CDR thoại nên báo nhầm thuê bao chỉ gọi. File SMS mẫu chỉ có tin tới 900 nên cần bản đầy đủ. | Bổ sung CDR thoại để dùng rule gốc. |
| E13 | Tăng đột biến lưu lượng bất thường | THU_NGHIEM_..._DATA | baseline_avg_30d = trung bình lưu lượng theo ngày (up + down). spike_ratio = hôm nay chia baseline. | Chỉ so theo ngày vì CDR không có giờ. Cần 30 ngày lịch sử, hoặc rút baseline còn 7 ngày để chạy sớm. Phải cộng gộp các dòng bị cắt 3600 giây và kiểm tra dòng trùng LOCAL_SEQUENCE_NUMBER. | Tích lũy lịch sử; chốt khóa chống trùng. |
| U02 | Đăng ký/gia hạn gói thành công lần >= 2 | Thuebao_DK_GOI | success_count = số dòng DK_GOI của MSISDN trong ngày. Mã gói = SERVICE_CODE. | Giả định TYPE=1 là đăng ký thành công. Chưa rõ file có chứa gia hạn không. Mẫu mỗi thuê bao chỉ 1 dòng nên chưa có ca ≥2. | Xác nhận ý nghĩa TYPE. |
| U10 | Ngày lễ / sự kiện quốc gia | PTM, DATA, SMS, HUY (để lập danh sách thuê bao đang hoạt động) | holiday_code và event_date là lịch cấu hình trong CVM. Tập nhận là các thuê bao đang hoạt động, loại thuê bao đã ngắt. | Không có phân khúc nên gửi chung, không cá thể hóa. | Lịch ngày lễ cấu hình trong CVM. |
| E_USAGE_NEED_ANALYSIS | Phân tích nhu cầu gói theo mức sử dụng | THU_NGHIEM_..._DATA | avg_usage_30d = trung bình lưu lượng theo ngày. Tỷ lệ dùng trên quota tính khi có danh mục gói. | Cần danh mục gói (quota, giá). Thông tin khách hàng (họ tên, tuổi, giới tính) lấy từ CRM. | Danh mục gói; dữ liệu CRM. |
| E_NO_PLAN_X_DAYS | Thuê bao trong x ngày nhưng không có gói | Thuebao_PTM, Thuebao_DK_GOI, Thuebao_HUY_GOI | Với SIM mới: có PTM mà 10 ngày không có DK_GOI thì fire. Số gói đang hoạt động = đăng ký trừ hủy. | Với thuê bao cũ phải có snapshot gói hiện tại, nếu không không biết họ đang có gói hay không. Chưa có hạn gói nên gói hết hạn tự nhiên không bắt được. | Snapshot gói hiện tại (vasoffer) cho thuê bao cũ. |
| E07 | Milestone 7 ngày / khảo sát ngắn | Thuebao_PTM, THU_NGHIEM_..._DATA | Nhánh theo ngày: days_since_activation tính từ PTM. usage_days_7d = số ngày có DATA. | Nhánh "hoàn thành ≥3 nhiệm vụ" và điểm gắn kết thuộc SuperApp, không làm được từ CDR. | SuperApp cấp task_log cho nhánh nhiệm vụ. |
| E11 | Tổng kết ngày 30 | Thuebao_PTM, THU_NGHIEM_..._DATA | days_since_activation từ PTM. total_data_30d_mb và active_days_30d từ DATA. | Số lần nạp, doanh thu và dữ liệu app là tùy chọn nên bỏ qua. Điểm gắn kết và nhóm đích cần bổ sung sau. | Nạp tiền và SuperApp nếu muốn đầy đủ. |
| CHURN-1 | Rời mạng: không phát sinh lưu lượng liên tiếp 5–7 ngày | THU_NGHIEM_..._DATA, THU_NGHIEM_..._SMS, Thuebao_HUY | Đếm số ngày liên tiếp MSISDN không xuất hiện trong DATA và SMS. Loại MSISDN có trong HUY. | Cần 5–7 ngày lịch sử. Thiếu CDR thoại nên báo nhầm thuê bao chỉ gọi. | Tích lũy lịch sử; bổ sung CDR thoại. |

## B. Làm được nhờ suy luận

| Mã | Tên | File CDR dùng | CVM tự tính thế nào | Giả định và rủi ro | Điều kiện tiên quyết |
|---|---|---|---|---|---|
| E08 | Data ngày sắp hết | Thuebao_GOI_MOI, Thuebao_DK_GOI | used_percent = PERCENT_USAGE. remaining_data_mb = hạn mức trừ ACCUMULATED_USAGE. addon_purchased_today lấy từ DK_GOI. | Mẫu GOI_MOI chỉ có mức 100%, chưa biết file có phát mức 80–90% hay không. Nếu không thì phải đổi sang ngưỡng tuyệt đối. | CNTT xác nhận GOI_MOI phát theo ngưỡng; có danh mục gói. |
| U05-A | Hết data tháng sớm 2 tháng liên tiếp | Thuebao_GOI_MOI | Đếm số tháng liên tiếp có sự kiện hết data (PERCENT_USAGE = 100%) trước ngày cut-off. | Phụ thuộc E_DATA_100. Cần 2 tháng lịch sử và danh mục gói để lọc gói tháng. | E_DATA_100 chạy ổn định; danh mục gói. |
| U05-B | Pattern hết quota data ngày thường xuyên | Thuebao_GOI_MOI | Đếm số ngày hết quota trong kỳ quan sát từ sự kiện 100% của gói ngày. | Phụ thuộc E_DATA_100 và việc GOI_MOI phát theo từng chu kỳ ngày. Cần ít nhất 2 tháng lịch sử. | E_DATA_100 chạy ổn định. |
| U06 | Chuyển đổi loại gói thành công | Thuebao_HUY_GOI, Thuebao_DK_GOI | Hủy gói A rồi đăng ký gói B trong cùng ngày thì coi là chuyển gói: old_package_code = gói hủy, new_package_code = gói đăng ký. | Có thể nhầm trường hợp hủy rồi tự đăng ký lại cùng gói. OCS có CDR ChangeBrand nên là nguồn chính xác hơn. | Xác nhận cách ghi nhận; hoặc dùng ChangeBrand từ OCS. |
| U07 | Chuyển đổi SIM nội mạng | THU_NGHIEM_..._DATA | sim_change_status suy từ việc IMSI của cùng MSISDN thay đổi giữa các ngày. | Không biết loại SIM cũ và mới (vật lý hay eSIM) trong khi hai field này bắt buộc. Chỉ phát hiện khi khách có dùng data sau khi đổi. | Cần nguồn sim_swap từ BSS để biết loại SIM. |
| U08 | Gia hạn gói liên tiếp / vinh danh trung thành | Thuebao_DK_GOI | Đếm chuỗi dòng DK_GOI liên tiếp của cùng MSISDN và cùng gói theo chu kỳ. | Phụ thuộc TYPE có phân biệt gia hạn với đăng ký mới hay không. Không có hạn gói nên chưa phân loại đúng hạn hay sớm hạn. | Xác nhận TYPE; lịch sử vài chu kỳ. |
| U_PRE_EXPIRY | Trước khi gói hết hạn x ngày | Thuebao_DK_GOI, Thuebao_GOI_MOI | Hạn gói = ngày đăng ký cộng chu kỳ gói trong danh mục. days_to_expiry = hạn trừ hôm nay. | Gói tự gia hạn không có CDR gia hạn nên sẽ nhắc nhầm khách đã gia hạn. | Danh mục gói (chu kỳ); CDR gia hạn. |
| U_POST_EXPIRY | Sau khi gói hết hạn x ngày chưa gia hạn | Thuebao_DK_GOI, Thuebao_GOI_MOI | days_since_expiry = hôm nay trừ hạn suy ra. renewed_after_expiry = có DK_GOI mới sau hạn. | Như U_PRE_EXPIRY. | Danh mục gói; CDR gia hạn. |

## C. Chưa làm được

| Mã | Tên | File CDR dùng | CVM tự tính thế nào | Giả định và rủi ro | Điều kiện tiên quyết |
|---|---|---|---|---|---|
| E06 | Cuộc gọi thất bại | Không có | Không suy ra được cuộc gọi thất bại và nguyên nhân từ bộ CDR hiện có. | Cần CDR cuộc gọi thất bại kèm lý do. | OCS hoặc CNTT tạo CDR call-fail. |
| E_VOICE_100_ONNET | Hết phút thoại nội mạng | Không có | Không có CDR thoại và quota phút. | Có thể bù từ OCS Lowbalance nếu tách account nội mạng. | OCS mở luồng Lowbalance và CDR thoại. |
| E_VOICE_100_OFFNET | Hết phút thoại ngoại mạng | Không có | Không có CDR thoại và quota phút. | Như nội mạng. | OCS mở luồng Lowbalance và CDR thoại. |
| E_ZERO_BALANCE | Số dư TKC về 0 | Không có | Không có số dư, nạp tiền hay trừ tiền trong bộ CDR. | Có thể bù từ OCS Lowbalance. Cần chốt TKC tương ứng account nào. | OCS mở luồng Lowbalance. |
| U01 | Nạp tiền thành công lần >= 2 | Không có trong mẫu | Bộ CDR mẫu không có nạp tiền. | OCS xác nhận có CDR_Recharge nên làm được khi luồng này được mở. | OCS mở luồng CDR_Recharge qua Kafka. |
| U03 | Tra cứu số dư / gắn gợi ý inline | Không có | Không có tín hiệu tra cứu *101#. | Cần OCS mở API đồng bộ hoặc bỏ trigger. | Quyết định của CNTT. |
| U09 | Sinh nhật / kỷ niệm KH hoặc SIM | Thuebao_PTM (chỉ cho kỷ niệm kích hoạt) | Kỷ niệm tròn năm tính từ ngày kích hoạt trong PTM. | Chưa có ngày sinh. Kỷ niệm tròn năm cần ít nhất 1 năm lịch sử PTM. | Ngày sinh từ CRM; lịch sử PTM. |
| E_SEGMENT_UPDATE | Cập nhật phân khúc KH | Không có | Phân khúc HSSV, tài xế, cán bộ, hưu trí không suy ra được từ CDR. | Cần dữ liệu khách hàng từ CRM hoặc quy tắc phân khúc từ nghiệp vụ. | Dữ liệu CRM. |
| E_LOCK_2C | Khóa 2 chiều | Không có | Không có CDR đổi trạng thái khóa. File HUY là ngắt thuê bao, không phải khóa. | Có thể bù từ OCS CDR_Change_State. | OCS mở luồng Change_State. |
| E_LOCK_1C | Khóa 1 chiều | Không có | Như khóa 2 chiều. | Như khóa 2 chiều. | OCS mở luồng Change_State. |
| E_PRE_LOCK_2C | Trước khi khóa 2 chiều | Không có | Phụ thuộc E_LOCK_1C và số ngày từ khóa 1 chiều sang 2 chiều. | Phải có ngày khóa 1 chiều trước. | E_LOCK_1C chạy được; chốt số ngày quy định. |
| CHURN-2 | Rời mạng: không phát sinh cước trong 30 ngày | Không có | Không có dữ liệu cước hay nạp tiền. Có thể tạm dùng "không có data và SMS 30 ngày" làm xấp xỉ nhưng không phải cước thật. | Xấp xỉ sẽ lệch với khách chỉ nhận cuộc gọi. | CDR cước hoặc nạp tiền. |
| CHURN-3 | Rời mạng: doanh thu suy giảm 80% so với trung bình 2 tháng | Không có | Không có doanh thu. | Cần dữ liệu doanh thu từ Billing hoặc nạp tiền. | CDR nạp tiền hoặc doanh thu. |
| CHURN-4 | Rời mạng: tỷ lệ bật tắt sóng của SIM | THU_NGHIEM_..._DATA | Chưa có định nghĩa tỷ lệ bật tắt sóng. Chỉ có phiên data, không có sự kiện mất sóng. | Cần định nghĩa nghiệp vụ trước. | Định nghĩa của PO. |

## D. Ngoài phạm vi CDR

| Mã | Tên | Nguồn cần |
|---|---|---|
| E02 | Chưa cài app sau 24h kích hoạt SIM | Cần log cài app và mở app từ SuperApp. |
| U04 | Nhận OTP từ app bên thứ 3 | Cần file otp_detection từ HLR. |
| E03 | Đăng nhập app lần đầu | Cần event FIRST_OPEN. |
| E_CONTENT_FAIL | Mua dịch vụ nội dung thất bại | Cần event mua nội dung thất bại. |
| E_APP_RATING | KH đánh giá app | Cần event đánh giá app. |
| E04 | Cài app nhưng chưa mở sau 24h | Cần log cài và mở app. |
| E09 | Xem màn hình đổi gói nhưng chưa đăng ký (hành trình mua cả sim, cả gói => Tính từ x phút từ lúc bắt đầu hành trình mua sim/mua gói | Cần event xem màn hình đổi gói. |
| E_APP_INACTIVE_X_DAYS | X ngày KH không truy cập app | Cần log truy cập app. |

## Câu hỏi cần CNTT xác nhận

| STT | Câu hỏi | Trigger bị ảnh hưởng |
|---|---|---|
| 1 | GOI_MOI có phát các mức giữa (80%, 90%) hay chỉ phát khi hết 100%? ACCUMULATED_USAGE=1024 có đơn vị gì? | E_DATA_100, E08, U05-A, U05-B |
| 2 | HUY_GOI có dữ liệu thật không? Xin mẫu một ngày có bản ghi. | E_CANCEL_PLAN, U06, E_NO_PLAN_X_DAYS |
| 3 | Giá trị TYPE trong DK_GOI và HUY_GOI nghĩa là gì (đăng ký, gia hạn, tự gia hạn, hủy)? | U02, U06, U08 |
| 4 | DISCONNECTIONCODE=3 trong Thuebao_HUY nghĩa là gì? | Danh sách loại trừ, rời mạng |
| 5 | Đơn vị DATA_VOLUME_UPLINK và DATA_VOLUME_DOWNLINK là byte hay KB? | E13, E05, E11, E_USAGE_NEED_ANALYSIS |
| 6 | Message Kafka của DATA, PTM, DK_GOI có giờ-phút-giây không (hiện CSV chỉ có ngày)? | E01, E13 và mọi trigger Near Realtime |
| 7 | File SMS có chứa SMS thường (không chỉ tin tới 900) không? 900 là đầu số gì? | E05, E01 |
| 8 | Sẽ có thêm CDR thoại, nạp tiền, đổi gói, đổi trạng thái, Lowbalance từ OCS qua Kafka không? | U01, U06, E_ZERO_BALANCE, E_VOICE_100, E_LOCK_1C/2C |
| 9 | Danh mục gói (mã gói, tên, chu kỳ, quota, giá) lấy từ đâu cho CVM? | E_USAGE_NEED_ANALYSIS, U_PRE/POST_EXPIRY, E08 |
| 10 | LOCAL_SEQUENCE_NUMBER lặp ở một số dòng: có phải bản ghi trùng không, khóa chống trùng là gì? | E13, E05, E11 |

## Đối chiếu với đánh giá OCS của CNTT

- Sheet "Đánh giá trigger" và "Assessment OCS" trong Excel chấm theo CDR của OCS (CDR_Recharge, ChangeBrand, Change_State, Lowbalance, vasoffer, PCRF, Voice). Bộ CDR mẫu này là loại khác (Thuebao_* của BSS, PGW data, SMS).
- Hai nguồn bổ sung cho nhau: E_CANCEL_PLAN, E_DATA_100 và E01 có thể làm được nhờ file BSS dù OCS báo không đủ.
- Các trigger U01, E_ZERO_BALANCE, E_VOICE_100 và khóa 1 chiều, 2 chiều làm được khi OCS mở luồng tương ứng qua Kafka.
- Mô hình kết nối đã thống nhất: các nguồn đẩy CDR hoặc message qua Kafka của Cloud-CNTT, CVM là consumer. Các file `data-contract-template*.md` hiện còn viết theo kiểu CSV hoặc API, cần rà lại sau khi chốt schema Kafka.
