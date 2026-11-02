# Tổng hợp khả năng trigger CVM theo CDR BSS mẫu 30/09/2026

> Danh sách lấy từ sheet "Điều kiện con (Trigger)" của `Trigger-CVM (4).xlsx`: 39 trigger. Mỗi trigger được chấm theo các field của sheet đó, ưu tiên field Bắt buộc. Bản Excel đi kèm (có chi tiết từng field): `cdr-trigger-feasibility.xlsx`.

## Tóm tắt

| Mức | Số trigger | Định nghĩa |
|---|---|---|
| A. Làm được | 6 | Mọi field bắt buộc có sẵn trong CDR, hoặc chỉ cần tính toán thuần (đếm, cộng, trừ ngày) từ cột có nghĩa rõ ràng. |
| B. Làm được nhờ suy luận | 14 | Có ít nhất một field bắt buộc phải suy luận: dựa vào cách hiểu cột chưa được CNTT xác nhận, ghép nhiều file, hoặc bỏ một điều kiện không có dữ liệu. Cột "Suy luận như thế nào" ghi rõ từng bước. |
| C. Chưa làm được | 11 | Có field bắt buộc không có nguồn trong bộ CDR và không suy ra được. |
| D. Ngoài phạm vi CDR | 8 | Field bắt buộc thuộc SuperApp hoặc HLR, bộ CDR này không đánh giá được. |
| Tổng | 39 | |

Thay đổi so với bản 1: danh sách chỉ lấy từ sheet "Điều kiện con (Trigger)" (4 tiêu chí Rời mạng được thay bằng E_CHURN_RISK). Định nghĩa mức A được siết nên E05, E13, E_DATA_100, U02, E_NO_PLAN_X_DAYS, E_CHURN_RISK chuyển từ A sang B.

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

| Mã | Tên | Kết luận | Rủi ro và giới hạn |
|---|---|---|---|
| E01 | SIM kích hoạt lần đầu | Đủ dữ liệu: mốc kích hoạt lấy trực tiếp từ PTM, số lần và số giờ tự tính. | PTM chỉ có ngày, không có giờ, nên SLA 30–60 phút cần timestamp từ message Kafka. 3 thuê bao PTM mẫu chưa có SMS tới 900 nên điều kiện "gọi 900" chưa kiểm chứng được. |
| E_CANCEL_PLAN | Hủy gói cước | Cấu trúc file đủ cho hai field bắt buộc (SERVICE_CODE, TYPE). File mẫu rỗng nên chưa kiểm chứng được. | Chưa có dòng dữ liệu nào, và ý nghĩa giá trị TYPE chưa rõ. OCS báo không có CDR hủy gói nhưng nguồn BSS này có thể bù. |
| U10 | Ngày lễ / sự kiện quốc gia | Đủ: lịch ngày lễ là cấu hình trong CVM, tập nhận là thuê bao đang hoạt động. | Không có phân khúc nên gửi chung, không cá thể hóa. |
| E_USAGE_NEED_ANALYSIS | Phân tích nhu cầu gói theo mức sử dụng | Đủ cho phần tính mức sử dụng trung bình. | Cần 30 ngày lịch sử. Tỷ lệ dùng trên quota và gói đề xuất cần danh mục gói, đây là cấu hình ngoài CDR. |
| E07 | Milestone 7 ngày / khảo sát ngắn | Đủ cho nhánh theo ngày (field bắt buộc chỉ là days_since_activation). | Nhánh "hoàn thành nhiệm vụ" và điểm gắn kết thuộc SuperApp, không làm được từ CDR. |
| E11 | Tổng kết ngày 30 | Đủ cho field bắt buộc (days_since_activation); tổng data 30 ngày và số ngày hoạt động tự tính từ DATA. | Doanh thu, số lần nạp, điểm gắn kết và nhóm đích cần nạp tiền và SuperApp, bỏ qua trong bản này. |

## B. Làm được nhờ suy luận

### E08 — Data ngày sắp hết

- **Kết luận:** Làm được nếu Thuebao_GOI_MOI phát các mức dùng giữa chừng.
- **Field bắt buộc:** quota_type (Suy luận); used_percent (Suy luận); addon_purchased_today (Tự tính)
- **Suy luận như thế nào:** 1) Coi Thuebao_GOI_MOI là bảng theo dõi mức dùng gói data: SERVICE_GROUP là gói, PERCENT_USAGE là used_percent. 2) Fire khi PERCENT_USAGE đạt ngưỡng cấu hình (ví dụ 90%). 3) quota_type gán là data. Phân biệt gói ngày/tháng lấy từ danh mục gói theo SERVICE_GROUP. 4) addon_purchased_today = có dòng DK_GOI của MSISDN trong ngày.
- **Rủi ro:** Mẫu chỉ có mức 100%. Nếu file chỉ phát khi hết gói thì không bắt được mức 80–90% và E08 mất ý nghĩa, chỉ còn E_DATA_100. OCS cũng báo ngưỡng theo % không có.
- **Cần xác nhận:** GOI_MOI có phát các mức giữa không? Có danh mục gói không?

### E_DATA_100 — Hết 100% data

- **Kết luận:** Làm được nếu Thuebao_GOI_MOI đúng là bản ghi mức dùng gói.
- **Field bắt buộc:** quota_type (Suy luận); remaining_data_mb (Suy luận)
- **Suy luận như thế nào:** 1) Dòng GOI_MOI có PERCENT_USAGE=100% nghĩa là gói data đã dùng hết. 2) remaining_data_mb = hạn mức trừ ACCUMULATED_USAGE; hạn mức suy ra bằng chính ACCUMULATED_USAGE (1024) khi đạt 100%, nên còn lại bằng 0. 3) quota_type gán là data. Căn cứ ủng hộ: tên gói POTTEL_BIG_1MB khớp 1024 (KB bằng 1 MB), và cả 7 thuê bao đều dùng từ 108 MB đến 1,8 GB trong ngày, vượt xa 1 MB.
- **Rủi ro:** Nếu 1024 là MB thì thuê bao dùng chỉ 108 MB không thể đạt 100%, tức cách hiểu sai. Cần xác nhận đơn vị và việc file có phát lại theo từng chu kỳ không.
- **Cần xác nhận:** ACCUMULATED_USAGE có đơn vị gì? Thuebao_GOI_MOI là snapshot mức dùng hay danh sách gói mới đăng ký?

### E05 — Chưa phát sinh cước sau 72h

- **Kết luận:** Làm được theo rule rút gọn "data bằng 0 và SMS bằng 0".
- **Field bắt buộc:** hours_since_activation (Tự tính); data_usage_mb (Tự tính); voice_usage_sec (Không có); sms_count (Có sẵn)
- **Suy luận như thế nào:** 1) NGAY_0 là ngày của dòng PTM; D1–D3 là 3 ngày kế tiếp. 2) data_usage_mb = tổng up + down của MSISDN trong DATA ở D1–D3. 3) sms_count = số dòng SMS của MSISDN. 4) voice_usage_sec không có nên bỏ điều kiện thoại, coi thoại bằng 0. 5) Fire khi data và SMS cùng bằng 0.
- **Rủi ro:** Thuê bao chỉ gọi thoại sẽ bị đưa nhầm vào tệp. File SMS mẫu chỉ có tin tới 900 nên sms_count có thể thiếu. Cả 3 thuê bao PTM mẫu đều có data (49 MB, 62 MB, 281 MB) nên chưa có ca dương tính.
- **Cần xác nhận:** Có CDR thoại không? Đơn vị DATA_VOLUME? File SMS đầy đủ có tin thường không?

### E13 — Tăng đột biến lưu lượng bất thường

- **Kết luận:** Làm được theo ngày sau khi tích lũy lịch sử.
- **Field bắt buộc:** metric_type (Tự tính); current_usage (Tự tính); baseline_avg_30d (Tự tính); traffic_spike_mb (Suy luận); spike_ratio (Tự tính)
- **Suy luận như thế nào:** 1) current_usage = tổng up + down của MSISDN trong ngày từ DATA. 2) baseline_avg_30d = trung bình các ngày trước đó. 3) spike_ratio = current_usage chia baseline. 4) traffic_spike_mb gốc là lưu lượng trong giờ đột biến; CDR không có giờ nên thay bằng lưu lượng cả ngày. 5) spike_hour (tùy chọn) bỏ.
- **Rủi ro:** Đột biến ngắn trong một giờ bị loãng khi gộp theo ngày. Cần 30 ngày lịch sử, hoặc rút baseline còn 7 ngày để chạy sớm. Phải cộng gộp các dòng bị cắt DURATION=3600 và kiểm tra dòng trùng LOCAL_SEQUENCE_NUMBER.
- **Cần xác nhận:** Message Kafka của DATA có giờ không? Khóa chống trùng là gì?

### U02 — Đăng ký/gia hạn gói thành công lần >= 2

- **Kết luận:** Làm được cho phần đăng ký gói, phần gia hạn chưa thấy trong mẫu.
- **Field bắt buộc:** transaction_type (Suy luận); transaction_status (Suy luận); success_count (Tự tính)
- **Suy luận như thế nào:** 1) Mỗi dòng Thuebao_DK_GOI là một giao dịch đăng ký gói (TYPE=1). 2) File không có cột trạng thái, nên coi dòng có mặt là giao dịch thành công. 3) success_count = đếm số dòng của MSISDN trong khoảng lọc, fire khi bằng hoặc vượt N.
- **Rủi ro:** Nếu TYPE không phải "thành công" thì đếm sai. Mẫu 3 dòng, mỗi thuê bao 1 lần nên chưa có ca đủ N=2.
- **Cần xác nhận:** Giá trị TYPE nghĩa là gì? DK_GOI có chứa gia hạn không?

### U05-A — Hết data tháng sớm 2 tháng liên tiếp

- **Kết luận:** Làm được sau khi E_DATA_100 chạy và tích lũy 2 chu kỳ tháng.
- **Field bắt buộc:** package_cycle (Suy luận); early_depletion_count (Tự tính); depletion_day (Tự tính); quota_type (Suy luận)
- **Suy luận như thế nào:** 1) Mỗi lần Thuebao_GOI_MOI báo PERCENT_USAGE=100% của gói tháng là một lần hết data. 2) depletion_day = ngày của sự kiện tính từ ngày bắt đầu chu kỳ. 3) early_depletion_count = số tháng liên tiếp hết trước ngày cut-off (mặc định 25). 4) package_cycle và quota_type lấy từ danh mục gói.
- **Rủi ro:** Gói trong mẫu là 1MB, không phải gói tháng. Kế thừa mọi giả định của E_DATA_100. Cần ít nhất 2 tháng dữ liệu.
- **Cần xác nhận:** Như E_DATA_100 và có danh mục gói.

### U05-B — Pattern hết quota data ngày thường xuyên

- **Kết luận:** Làm được sau khi E_DATA_100 chạy và tích lũy lịch sử ngày.
- **Field bắt buộc:** package_cycle (Suy luận); depletion_days_count (Tự tính); observation_days (Cấu hình CVM); quota_type (Suy luận)
- **Suy luận như thế nào:** 1) Mỗi sự kiện PERCENT_USAGE=100% của gói ngày là một ngày hết quota. 2) depletion_days_count = số ngày trong kỳ quan sát có sự kiện. 3) observation_days là tham số CVM cấu hình. 4) avg_depletion_hour (tùy chọn) không có vì chỉ có ngày.
- **Rủi ro:** Phụ thuộc file GOI_MOI phát theo từng chu kỳ ngày. Kế thừa giả định của E_DATA_100.
- **Cần xác nhận:** Như E_DATA_100.

### U06 — Chuyển đổi loại gói thành công

- **Kết luận:** Làm được gần đúng bằng cách ghép hủy gói và đăng ký gói cùng ngày.
- **Field bắt buộc:** change_status (Suy luận); old_package_code (Suy luận); new_package_code (Suy luận)
- **Suy luận như thế nào:** 1) Lấy dòng HUY_GOI (gói A) và dòng DK_GOI (gói B) của cùng MSISDN trong cùng ngày. 2) Nếu A khác B thì coi là chuyển gói thành công: old_package_code = A, new_package_code = B, change_status = thành công.
- **Rủi ro:** Không bắt được chuyển gói mà hủy và đăng ký lệch ngày. Nhầm trường hợp khách tự hủy rồi đăng ký gói khác không liên quan. HUY_GOI mẫu rỗng. OCS có CDR ChangeBrand chính xác hơn.
- **Cần xác nhận:** Khi đổi gói, hệ thống ghi HUY_GOI và DK_GOI cùng lúc không?

### U07 — Chuyển đổi SIM nội mạng

- **Kết luận:** Phát hiện được việc đổi SIM, không phân loại được loại SIM.
- **Field bắt buộc:** sim_change_status (Suy luận); old_sim_type (Không có); new_sim_type (Không có)
- **Suy luận như thế nào:** 1) Lưu IMSI gần nhất của từng MSISDN từ DATA. 2) Khi cùng MSISDN xuất hiện IMSI khác thì coi là đã đổi SIM, và is_same_msisdn = đúng. 3) old_sim_type và new_sim_type không có nên nội dung phân nhánh theo loại SIM không làm được.
- **Rủi ro:** Chỉ phát hiện khi khách có dùng data sau khi đổi. Mẫu 78 MSISDN đều chỉ có 1 IMSI nên chưa có ca đổi SIM.
- **Cần xác nhận:** Có nguồn sim_swap từ BSS để biết loại SIM không?

### E_NO_PLAN_X_DAYS — ACTIVE nhưng không có gói x ngày

- **Kết luận:** Làm được chính xác cho SIM mới; thuê bao cũ cần snapshot gói hiện tại.
- **Field bắt buộc:** active_package_count (Suy luận); days_without_plan (Suy luận); subscriber_status (Có sẵn)
- **Suy luận như thế nào:** 1) Trạng thái gói là tổng cộng dồn sự kiện: cộng 1 khi có DK_GOI hoặc GOI_MOI, trừ 1 khi có HUY_GOI, ra active_package_count. 2) days_without_plan = hôm nay trừ ngày kích hoạt (PTM) nếu chưa từng đăng ký gói, hoặc trừ ngày HUY_GOI gần nhất nếu số gói về 0. 3) subscriber_status lấy từ PTM (Active), loại MSISDN có trong Thuebao_HUY.
- **Rủi ro:** Không có hạn gói nên gói hết hạn tự nhiên không bị trừ, dẫn tới bỏ sót khách hết gói. Thuê bao kích hoạt trước khi bắt đầu thu thập có số gói không biết.
- **Cần xác nhận:** Có snapshot gói đang hoạt động (vasoffer) cho thuê bao cũ không?

### E_CHURN_RISK — Thuê bao có nguy cơ rời mạng

- **Kết luận:** Làm được tiêu chí cốt lõi "không phát sinh lưu lượng liên tiếp".
- **Field bắt buộc:** no_usage_days (Suy luận)
- **Suy luận như thế nào:** 1) Dựng danh sách thuê bao nền từ PTM, DATA và SMS đã từng xuất hiện. 2) no_usage_days = số ngày liên tiếp gần nhất MSISDN không có dòng nào trong DATA và SMS. 3) CDR thoại không có nên coi thoại bằng 0. 4) Loại MSISDN có trong Thuebao_HUY. 5) Fire khi no_usage_days đạt 5–7 ngày.
- **Rủi ro:** Thuê bao chỉ gọi thoại bị nhầm là không dùng. Cần danh sách thuê bao nền đầy đủ, nếu không thuê bao chưa từng xuất hiện sẽ bị bỏ sót. Các tiêu chí cước 30 ngày, doanh thu giảm 80% và bật/tắt sóng không có dữ liệu.
- **Cần xác nhận:** Có danh sách thuê bao nền và CDR thoại không?

### U08 — Gia hạn gói liên tiếp / vinh danh trung thành

- **Kết luận:** Làm được ở mức đếm số kỳ gia hạn; chưa phân loại đúng hạn hay sớm hạn.
- **Field bắt buộc:** renewal_status (Suy luận); consecutive_renewal_count (Suy luận)
- **Suy luận như thế nào:** 1) Coi mỗi dòng Thuebao_DK_GOI là một kỳ gia hạn thành công (renewal_status). 2) consecutive_renewal_count = số dòng DK_GOI liên tiếp của cùng MSISDN và cùng SERVICE_CODE, mỗi dòng cách nhau xấp xỉ một chu kỳ gói trong danh mục. 3) Fire khi đạt mốc M3, M6, M12.
- **Rủi ro:** Không phân biệt gia hạn tự động với đăng ký thủ công. Cần tích lũy ít nhất 3 chu kỳ. Không có hạn gói nên không phân loại ON_TIME, EARLY.
- **Cần xác nhận:** TYPE có phân biệt gia hạn không? Có danh mục chu kỳ gói không?

### U_PRE_EXPIRY — Trước khi gói hết hạn x ngày

- **Kết luận:** Làm được nếu có danh mục chu kỳ gói; có rủi ro nhắc nhầm gói tự gia hạn.
- **Field bắt buộc:** days_to_expiry (Suy luận); package_code (Có sẵn)
- **Suy luận như thế nào:** 1) Hạn gói = ngày đăng ký (DK_GOI.LOGDATE hoặc GOI_MOI.INSERT_TIME) cộng chu kỳ gói tra từ danh mục theo SERVICE_CODE. 2) days_to_expiry = hạn trừ hôm nay. 3) Fire khi bằng X (7 hoặc 3 ngày). 4) package_code lấy từ SERVICE_CODE.
- **Rủi ro:** Gói tự gia hạn không có CDR gia hạn trong mẫu, nên CVM không biết hạn đã dời và nhắc nhầm. Không có danh mục chu kỳ gói.
- **Cần xác nhận:** Có CDR gia hạn không? Chu kỳ từng gói?

### U_POST_EXPIRY — Sau khi gói hết hạn x ngày chưa gia hạn

- **Kết luận:** Như U_PRE_EXPIRY.
- **Field bắt buộc:** days_since_expiry (Suy luận); renewed_after_expiry (Tự tính); expired_package_code (Có sẵn)
- **Suy luận như thế nào:** 1) Hạn gói suy ra như U_PRE_EXPIRY. 2) days_since_expiry = hôm nay trừ hạn. 3) renewed_after_expiry = có dòng DK_GOI cùng MSISDN và cùng gói sau ngày hạn. 4) expired_package_code = SERVICE_CODE.
- **Rủi ro:** Như U_PRE_EXPIRY. Khách đã gia hạn tự động nhưng CDR không ghi sẽ bị nhắc nhầm.
- **Cần xác nhận:** Như U_PRE_EXPIRY.

## C. Chưa làm được

| Mã | Tên | Lý do | Ghi chú |
|---|---|---|---|
| E06 | Cuộc gọi thất bại | Không có CDR cuộc gọi, không có cuộc gọi lỗi. | OCS cũng báo chưa có loại CDR này. |
| E_VOICE_100_ONNET | Hết phút thoại nội mạng | Không có CDR thoại và quota phút. | Có thể bù từ OCS Lowbalance nếu OCS tách account phút nội mạng. |
| E_VOICE_100_OFFNET | Hết phút thoại ngoại mạng | Không có CDR thoại và quota phút. | Như nội mạng. |
| E_ZERO_BALANCE | Số dư TKC về 0 | Không có số dư hay tài khoản. | Có thể bù từ OCS Lowbalance ngưỡng 0. |
| U01 | Nạp tiền thành công lần >= 2 | Không có CDR nạp tiền trong mẫu. | OCS xác nhận có CDR_Recharge, nên làm được khi luồng này được mở qua Kafka. |
| U03 | Tra cứu số dư / gắn gợi ý inline | Không có tín hiệu tra cứu *101#. | OCS không có CDR loại này. |
| U09 | Sinh nhật / kỷ niệm KH hoặc SIM | Thiếu ngày sinh; kỷ niệm tròn năm cần ít nhất 1 năm lịch sử PTM. | Sau 1 năm thu thập PTM thì phần kỷ niệm SIM mới tính được. |
| E_SEGMENT_UPDATE | Cập nhật phân khúc KH | Không có dữ liệu phân khúc. | Cần dữ liệu CRM hoặc quy tắc phân khúc. |
| E_LOCK_2C | Khóa 2 chiều | Không có CDR đổi trạng thái khóa. | Thuebao_HUY là ngắt thuê bao. Nếu CNTT cấp log đổi trạng thái có cấu trúc giống PTM (BEFORE_STATUS, AFTER_STATUS) thì làm được. OCS có CDR_Change_State nhưng chỉ cho khóa do hết hạn. |
| E_LOCK_1C | Khóa 1 chiều | Không có CDR đổi trạng thái khóa. | Như E_LOCK_2C. |
| E_PRE_LOCK_2C | Trước khi khóa 2 chiều | Phụ thuộc E_LOCK_1C và quy định số ngày từ khóa 1 chiều sang 2 chiều. | Phải có ngày khóa 1 chiều trước. |

## D. Ngoài phạm vi CDR

| Mã | Tên | Ghi chú |
|---|---|---|
| E02 | Chưa cài app sau 24h kích hoạt SIM | Cần log cài app của SuperApp (app_installed bắt buộc). CDR chỉ cho mốc kích hoạt. Phụ thuộc SuperApp. |
| U04 | Nhận OTP từ app bên thứ 3 | Cần log tin nhắn đến (OTP) từ HLR hoặc SMS log. File SMS mẫu chỉ có tin gửi đi tới 900. Nguồn là HLR export. |
| E03 | Đăng nhập app lần đầu | Thuộc SuperApp.  |
| E_CONTENT_FAIL | Mua dịch vụ nội dung thất bại | Thuộc SuperApp.  |
| E_APP_RATING | KH đánh giá app | Thuộc SuperApp.  |
| E04 | Cài app nhưng chưa mở sau 24h | Thuộc SuperApp.  |
| E09 | Xem màn hình đổi gói nhưng chưa đăng ký | Thuộc SuperApp.  |
| E_APP_INACTIVE_X_DAYS | X ngày KH không truy cập app | Thuộc SuperApp.  |

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

- Sheet "Đánh giá trigger" và "Assessment OCS" chấm theo CDR của OCS (CDR_Recharge, ChangeBrand, Change_State, Lowbalance, vasoffer, PCRF, Voice). Bộ CDR mẫu này là loại khác (Thuebao_* của BSS, PGW data, SMS) nên hai nguồn bổ sung cho nhau.
- E_CANCEL_PLAN và E_DATA_100 có thể làm được nhờ file BSS dù OCS báo không đủ. U01, E_ZERO_BALANCE, E_VOICE_100 và khóa 1 chiều, 2 chiều làm được khi OCS mở luồng tương ứng qua Kafka.
- Mô hình kết nối đã thống nhất: các nguồn đẩy CDR hoặc message qua Kafka của Cloud-CNTT, CVM là consumer.
