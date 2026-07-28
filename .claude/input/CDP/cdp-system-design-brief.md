# Design Brief — Hệ thống CDP VNPost (5 Chức năng)

**Hệ thống:** Customer Data Platform (CDP) — Tổng công ty Bưu điện Việt Nam (VNPost).
**Mục đích brief:** Dựng interactive prototype để review giao diện, gồm 5 chức năng trong một app liền mạch (chung navigation + design tokens).
**Yêu cầu:** Interactive prototype click được, data mẫu, self-contained. Đây là prototype mô phỏng — các nút lưu/gửi/merge chỉ hiển thị xác nhận, không gọi backend thật.

**5 chức năng:**
1. Kafka Integration — giám sát nhận dữ liệu
2. Customer Management — danh sách khách hàng *(nối liền sang Customer 360)*
3. Identity Resolution — phát hiện trùng & merge
4. Customer 360 — hồ sơ hợp nhất + timeline
5. Segment Management — tạo segment & xem KH thuộc segment

> Lưu ý luồng: Customer Management (CN2) và Customer 360 (CN4) **nối liền nhau** — từ danh sách KH click một dòng sẽ mở thẳng hồ sơ Customer 360. Giữ là 2 mục để đúng cấu trúc chức năng, nhưng về trải nghiệm là một luồng list → detail.

---

## A. Bối cảnh hệ thống

CDP hợp nhất dữ liệu khách hàng từ nhiều hệ thống nguồn của VNPost (CAS, CRM, MyVNPost, PayPost, PostID, PNS/DingDong, Portal KHL, MPITS, VPostCode) thành **một hồ sơ khách hàng duy nhất**.

Nguyên tắc dữ liệu: **"một Customer ID, nhiều vai trò, nhiều nguồn"**:
- Một khách hàng → một **Unified Customer ID (UID)** duy nhất.
- Một UID có nhiều vai trò: người gửi (SENDER), người nhận (RECEIVER), chủ shop (SHOP_OWNER), khách hàng lớn (KHL), SME.
- Hai loại khách hàng khác cấu trúc: **Cá nhân (INDIVIDUAL)** và **Doanh nghiệp (BUSINESS/KHL)**.

Quy mô: ~500.000 KH đã ký hợp đồng (B2B/KHL) + ~100.000 KH mới đang nuôi dưỡng (B2C/SME).

---

## B. Design system dùng chung (áp dụng cho cả 5 chức năng)

**Tính chất:** Operator console nội bộ — dense-data, ưu tiên quét nhanh & rõ ràng. KHÔNG phải landing page.

**Bảng màu:**
- Primary / brand: `#E30613` (đỏ VNPost — XÁC NHẬN LẠI mã chính thức; đây là giả định)
- Nền app: `#F7F8FA` / panel trắng `#FFFFFF`
- Text chính: `#1A1A1A` / phụ: `#6B7280`
- Trạng thái: success `#16A34A`, warning `#F59E0B`, danger `#DC2626`, info `#2563EB`
- Border/divider: `#E5E7EB`

**Typography:** Sans-serif rõ (Inter / IBM Plex Sans). Số liệu dùng tabular-figures.

**Khung layout chung:**
- **Sidebar trái** — 5 mục chức năng (icon + nhãn): Kafka Integration / Customer Management / Identity Resolution / Customer 360 / Segment Management.
- **Header trên** — logo CDP VNPost, tên màn hiện tại, ô tìm kiếm toàn cục (theo tên/SĐT/UID/MST), badge user, chuông cảnh báo.
- **Vùng nội dung** — thay đổi theo chức năng.

**Quy ước hiển thị chung:**
- Số tiền: định dạng VND có phân tách hàng nghìn (vd 458.200.000 ₫).
- Dữ liệu nhạy cảm masking: CCCD `012********`, tài khoản thanh toán masked.
- Badge trạng thái/điểm dùng màu nhất quán theo bảng màu trên.
- Trường chưa có dữ liệu: hiển thị "Chưa có dữ liệu" thay vì để trống.
- Responsive xuống mobile; focus bàn phím nhìn thấy.

---

## CN1. Kafka Integration (Giám sát Ingestion)

**Người dùng:** Admin / DevOps / Vận hành. **Loại UI:** Monitoring dashboard (không phải cấu hình connector).

CDP nhận dữ liệu real-time từ các hệ thống nguồn qua Kafka topic. Màn này theo dõi sức khỏe luồng nhận dữ liệu để phát hiện & xử lý sự cố nhanh.

**3.1. Dải KPI:** Tổng topic giám sát; số topic Healthy/Warning/Error; tổng message hôm nay; throughput hiện tại (msg/s); tổng consumer lag; message lỗi 24h.

**3.2. Biểu đồ throughput:** line/area chart message theo thời gian (tách thành công vs lỗi), lọc 1h/6h/24h/7 ngày.

**3.3. Bảng Topic (chính):** Cột — Trạng thái (badge HEALTHY/WARNING/ERROR/IDLE), Tên topic, Hệ thống nguồn, Loại dữ liệu, Message/phút, Tổng message 24h, Consumer lag (màu theo ngưỡng), Message lỗi 24h, Cập nhật cuối. Click dòng → panel chi tiết. Lọc theo trạng thái/nguồn/loại; sắp xếp theo lag/lỗi.

**3.4. Panel chi tiết Topic (slide-in):** partition, consumer group, offset, lag; throughput riêng; **danh sách message lỗi gần nhất** (thời gian, lý do, payload rút gọn); hành động mô phỏng (Retry, Tạm dừng consumer, Xem log).

**3.5. Alert feed:** cảnh báo gần đây (topic, mức độ, thời điểm, nội dung).

**Data mẫu topic:**
```
1. cdp.customer.profile    | CAS         | Khách hàng     | HEALTHY | 145 msg/ph | 208.000/24h | lag 12     | lỗi 0  | 5s trước
2. cdp.customer.profile.crm| CRM         | Khách hàng     | HEALTHY | 60 msg/ph  | 86.000/24h  | lag 4      | lỗi 2  | 8s trước
3. cdp.order.created       | MPITS       | Đơn hàng       | HEALTHY | 320 msg/ph | 461.000/24h | lag 30     | lỗi 5  | 2s trước
4. cdp.delivery.status     | PNS/DingDong| Trạng thái phát| WARNING | 210 msg/ph | 302.000/24h | lag 8.400  | lỗi 12 | 1m trước
5. cdp.cod.payment         | PayPost     | COD            | ERROR   | 0 msg/ph   | 54.000/24h  | lag 11.200 | lỗi 48 | 15m trước
6. cdp.behavior.event      | MyVNPost    | Hành vi        | HEALTHY | 480 msg/ph | 690.000/24h | lag 55     | lỗi 8  | 1s trước
7. cdp.customer.khl        | Portal KHL  | Khách hàng     | HEALTHY | 12 msg/ph  | 17.000/24h  | lag 0      | lỗi 0  | 20s trước
8. cdp.address.update      | VPostCode   | Địa chỉ        | IDLE    | 0 msg/ph   | 1.200/24h   | lag 0      | lỗi 0  | 2h trước
```
**Cảnh báo mẫu:**
```
[ERROR]   10:42  cdp.cod.payment — PayPost ngừng đẩy message 15 phút, lag 11.200
[WARNING] 10:30  cdp.delivery.status — Consumer lag vượt ngưỡng 8.000
[ERROR]   09:15  cdp.cod.payment — 48 message lỗi schema trong 1 giờ
```

---

## CN2. Customer Management (Danh sách khách hàng → nối sang Customer 360)

**Người dùng:** CSKH / Kinh doanh / Vận hành / Marketing.

**Màn danh sách:**
- Thanh tìm kiếm: tên / SĐT / UID / email / MST / mã KHL.
- Bộ lọc: customer_type (INDIVIDUAL/BUSINESS), customer_group (B2C/B2B/KHL/SME/ECOMMERCE), customer_status (ACTIVE/INACTIVE/BLOCKED/MERGED), segment, bưu cục quản lý.
- Bảng: UID, Tên/Công ty, Loại (badge), Nhóm (badge), SĐT, Tổng đơn, Doanh thu, Segment chính (chip), Trạng thái (badge), Cập nhật cuối.
- Click một dòng → mở **Customer 360** (CN4) của KH đó.
- Nút "Tạo Segment" → mở CN5.

**Data mẫu danh sách (tối thiểu 8–10 dòng, dùng 3 KH chi tiết bên dưới + bổ sung dòng hợp lý):**
```
UID-0000123456 | Nguyễn Thị Hương        | Cá nhân    | SME  | 0901234567 | 1.275  | 458.200.000 ₫    | Champions      | ACTIVE | hôm nay
UID-0000987654 | Công ty TNHH TM ABC     | Doanh nghiệp| KHL | 0987654321 | 18.450 | 6.820.000.000 ₫  | KHL giá trị cao| ACTIVE | hôm nay
UID-0000456789 | Phạm Minh Tuấn          | Cá nhân    | B2C  | 0978111222 | 86     | 9.400.000 ₫      | At Risk        | ACTIVE | 3 ngày trước
UID-0000334411 | Trần Quốc Bảo           | Cá nhân    | B2C  | 0905556677 | 412    | 78.500.000 ₫     | Loyal          | ACTIVE | hôm qua
UID-0000220088 | Cửa hàng Mẹ & Bé Sunny  | Doanh nghiệp| SME | 0933221100 | 2.310  | 612.000.000 ₫    | Chủ shop cao   | ACTIVE | hôm nay
UID-0000771122 | Lê Thị Hằng             | Cá nhân    | B2C  | 0918887766 | 24     | 1.800.000 ₫      | New            | ACTIVE | 1 tuần trước
UID-0000559900 | Công ty CP Logistics XYZ| Doanh nghiệp| KHL | 0966554433 | 9.870  | 3.410.000.000 ₫  | KHL giá trị cao| ACTIVE | hôm nay
UID-0000662200 | Đỗ Văn Khải             | Cá nhân    | B2C  | 0944332211 | 153    | 19.200.000 ₫     | At Risk        | INACTIVE| 2 tháng trước
```

---

## CN3. Identity Resolution (Phát hiện trùng & Merge)

**Người dùng:** Data Steward / Admin / CSKH cấp cao.

CDP tự phát hiện các hồ sơ có khả năng là **cùng một khách hàng** (trùng SĐT, email, CCCD, PostID...) và đề xuất gộp. Data Steward review từng cặp trước khi merge để tránh gộp nhầm.

**Luồng: list cặp nghi ngờ → mở so sánh side-by-side → merge.**

**3.1. Màn danh sách cặp nghi ngờ trùng:**
- Bảng các cặp ứng viên. Cột: Hồ sơ A (UID + tên + SĐT), Hồ sơ B (UID + tên + SĐT), **Độ tương đồng** (% — thanh/badge màu: ≥90 đỏ "rất giống", 70–89 vàng, <70 xám), Khóa trùng (PHONE/EMAIL/CCCD/POST_ID...), Nguồn phát hiện, Trạng thái (Chờ review / Đã merge / Đã bỏ qua).
- Lọc theo độ tương đồng, khóa trùng, trạng thái.
- Mỗi dòng: nút "So sánh" → mở màn so sánh; nút nhanh "Bỏ qua".

**3.2. Màn so sánh side-by-side (2 cột A | B):**
- Hiển thị song song các trường then chốt của 2 hồ sơ: tên, SĐT, email, CCCD/MST, PostID, địa chỉ, vai trò giao dịch, tổng đơn, nguồn dữ liệu.
- **Đánh dấu trực quan:** trường trùng khớp (xanh), trường khác nhau (vàng/đỏ).
- **Cảnh báo gộp nhầm:** nếu A là SENDER còn B là RECEIVER, hoặc SĐT là shared (dùng chung) → hiện cảnh báo "Kiểm tra kỹ trước khi merge".
- Khu vực quyết định: chọn hồ sơ nào làm **master** (UID giữ lại); với mỗi trường conflict cho phép chọn giá trị giữ lại (radio A/B), kèm hiển thị **reliability score** của từng giá trị để hỗ trợ quyết định.
- Hành động: "Xác nhận Merge" (mô phỏng — hiện modal xác nhận + ghi chú lý do), "Hai người khác nhau — Bỏ qua".

**Data mẫu cặp trùng:**
```
Cặp 1: UID-0000123456 (Nguyễn Thị Hương, 0901234567) ⟷ UID-0000123999 (Nguyen Thi Huong, 0901234567)
        Độ tương đồng 94% | Khóa: PHONE + tên gần đúng | Nguồn: CAS vs MyVNPost | Chờ review
Cặp 2: UID-0000987654 (Cty ABC, MST 0312345678) ⟷ UID-0000987001 (Cong ty ABC, MST 0312345678)
        Độ tương đồng 88% | Khóa: TAX_ID | Nguồn: Portal KHL vs CRM | Chờ review
Cặp 3: UID-0000456789 (Phạm Minh Tuấn, 0978111222) ⟷ UID-0000456120 (Pham M. Tuan, 0978111222)
        Độ tương đồng 76% | Khóa: PHONE (shared?) | CẢNH BÁO: A=SENDER, B=RECEIVER | Chờ review
```
> Cặp 3 cố ý có cảnh báo để minh họa case gộp nhầm người gửi/người nhận.

---

## CN4. Customer 360 (Hồ sơ hợp nhất + Timeline)

**Người dùng:** CSKH / Kinh doanh / Vận hành / Marketing / Admin. Đây là **màn trọng tâm** của hệ thống.

Hồ sơ tổng hợp toàn bộ dữ liệu một khách hàng từ mọi nguồn, tổ chức theo 13 block (A→M).

**Header:** avatar/initials (model không có ảnh — dùng initials) + tên + UID + badges (loại, nhóm, trạng thái, loyalty tier) + nút hành động (Sửa, Lịch sử merge, Yêu cầu xóa dữ liệu).

**Dải KPI:** Tổng đơn; Tổng doanh thu; Tỷ lệ phát thành công (%); Tỷ lệ hoàn (% — cao=đỏ); Độ đầy đủ hồ sơ (`data_completeness_score` — progress, ghi chú "demo").

**Dropdown chọn role người dùng** (CSKH / Marketing / Kinh doanh / Vận hành / Admin) → ẩn/hiện điểm số theo RBAC (xem bảng dưới).

**Tab/Section theo block:**
- **Tổng quan** — A (định danh) + E (vai trò: SENDER/RECEIVER count, primary role) + F (giao dịch) + segment hiện tại.
- **Định danh & Identity Graph** — D: danh sách alias (id_type, id_value, nguồn, confidence HIGH/MEDIUM/LOW, reliability score, primary). Có thể hiển thị dạng node trỏ về UID.
- **Địa chỉ** — C: nhiều địa chỉ, badge loại (SENDER/RECEIVER/CONTACT/BUSINESS), VPostCode, trạng thái chuẩn hóa.
- **Doanh nghiệp** — B: chỉ hiện nếu BUSINESS — MST, mã KHL, hợp đồng, danh sách contacts.
- **Giao dịch & COD** — F + G: KPI giao dịch, tuyến gửi thường xuyên; COD tổng/đã thu/chưa thu, trạng thái đối soát (MATCHED/DISCREPANCY/PENDING), tài khoản masked.
- **Hành vi số** — H: last login, device, sessions; **timeline sự kiện** (event_type + channel + thời gian).
- **CSKH** — I: tổng/đang mở khiếu nại, SLA, điểm hài lòng (0–5).
- **Điểm số & Phân khúc** — K: RFM (R/F/M 1–5 + nhóm), CLV, churn/engagement/cod_risk/fraud score (0–100, gauge màu), thời điểm tính + cảnh báo nếu cũ.
- **Consent** — L: trạng thái OPT_IN/OPT_OUT/UNKNOWN theo từng kênh (SMS/Email/Zalo/Push), nguồn & thời điểm; cảnh báo đỏ nếu `data_deletion_requested`. **Làm nổi bật về thị giác.**
- **Audit** — M: nguồn dữ liệu (chip), lịch sử merge, version, ghi chú data steward.

**Timeline tương tác (yêu cầu riêng của CN4):** một dòng thời gian hợp nhất xuyên block — gộp sự kiện hành vi (H), khiếu nại (I), merge (M), giao dịch lớn (F) — sắp theo thời gian, mỗi loại có icon/màu riêng. (Đây là phần compose từ nhiều block, không phải trường dữ liệu mới.)

**Bảng RBAC hiển thị điểm (Block K):**
| Điểm | CSKH | Marketing | Kinh doanh | Vận hành/COD | Admin |
|---|---|---|---|---|---|
| RFM | ✓ | ✓ | ✓ | ✗ | ✓ |
| CLV | ✓ | ✓ | ✓ | ✗ | ✓ |
| Churn | ✓ | ✓ | ✓ | ✗ | ✓ |
| Engagement | ✓ | ✓ | ✗ | ✗ | ✓ |
| COD Risk | ✗ | ✗ | ✓ | ✓ | ✓ |
| Fraud | ✗ | ✗ | ✓ | ✓ | ✓ |

**Data mẫu — 3 khách hàng:**

**KH1 — Cá nhân (chủ shop SME):**
```
UID-0000123456 | INDIVIDUAL | SME | ACTIVE | Nguyễn Thị Hương
SĐT 0901234567 | huong.nguyen@gmail.com | CCCD 012******** | PostID PID-889234 | BĐ Cầu Giấy
data_completeness 82 | roles [SENDER, SHOP_OWNER] primary SENDER | sender 1240 / receiver 35
total_orders 1.275 | revenue 458.200.000 ₫ | avg 106 đơn/tháng | dịch vụ EMS
tuyến: HN→HCM, HN→ĐN, HN→HP | phát thành công 94% | hoàn 8%
COD: tổng 1.250.000.000 ₫ / thu 1.180.000.000 ₫ / chưa thu 70.000.000 ₫ | MATCHED
loyalty GOLD 12.400 điểm | segments [Champions, Chủ shop hoạt động cao]
RFM R5 F5 M4 Champions | CLV 620.000.000 ₫ | churn 18 | engagement 88 | cod_risk 22 | fraud 9
consent SMS in / Email in / Zalo in / Push out
app login 24/06 09:12 Android, 47 phiên/30d | khiếu nại 3 (mở 0), hài lòng 4.5
nguồn: CAS, CRM, MyVNPost, PayPost, PostID, PNS
alias: PHONE 0901234567 (MyVNPost,HIGH,90,primary) | POST_ID PID-889234 (PostID,HIGH,95) | CRM_ID CRM-55120 (CRM,HIGH,85) | USER_APP_ID APP-99213 (MyVNPost,MEDIUM,70)
địa chỉ: SENDER primary — Số 5 ngõ 12 Trần Thái Tông, P.Dịch Vọng, Hà Nội — vpostcode 100123 — STANDARDIZED, NEW
```

**KH2 — Doanh nghiệp (KHL):**
```
UID-0000987654 | BUSINESS | KHL | ACTIVE | Công ty TNHH Thương mại ABC
MST 0312345678 | KHL-2024-0156 | đại diện Trần Văn Nam | hợp đồng ACTIVE 2024-01-15 → 2026-12-31 | ngành Bán lẻ TMĐT
contacts: Trần Văn Nam 0987654321 nam.tran@abc.vn Giám đốc (primary) | Lê Thị Mai 0911222333 mai.le@abc.vn Kế toán
data_completeness 91 | roles [SENDER, KHL] primary KHL
total_orders 18.450 | revenue 6.820.000.000 ₫ | avg 1.537 đơn/tháng | dịch vụ Bưu kiện thường
phát thành công 91% | hoàn 14%
COD: tổng 14.200.000.000 ₫ / thu 13.100.000.000 ₫ / chưa thu 1.100.000.000 ₫ | DISCREPANCY
segments [KHL giá trị cao, Tỷ lệ hoàn cần theo dõi]
RFM R5 F5 M5 Champions | CLV 9.500.000.000 ₫ | churn 35 | engagement 60 | cod_risk 58 | fraud 21
consent SMS in / Email in / Zalo unknown / Push unknown
khiếu nại 24 (mở 3), hài lòng 3.8, SLA 86%
nguồn: Portal KHL, CRM, CAS, MPITS, PayPost
```

**KH3 — Cá nhân rủi ro cao:**
```
UID-0000456789 | INDIVIDUAL | B2C | ACTIVE | Phạm Minh Tuấn
SĐT 0978111222 | PostID PID-220145 | data_completeness 54
roles [SENDER, RECEIVER] primary RECEIVER
total_orders 86 | revenue 9.400.000 ₫ | phát thành công 72% | hoàn 31%
COD: tổng 24.000.000 ₫ / chưa thu 8.500.000 ₫ | PENDING
loyalty: Chưa có dữ liệu | segments [At Risk, Tỷ lệ hoàn cao]
RFM R2 F2 M2 At Risk | churn 78 | engagement 25 | cod_risk 71 | fraud 44
consent SMS out / Email unknown | data_deletion_requested false
khiếu nại 7 (mở 2), hài lòng 2.9
nguồn: CAS, PNS, PayPost
```

---

## CN5. Segment Management (Tạo segment & xem KH thuộc segment)

**Người dùng:** Marketing / Kinh doanh / CSKH.

**5.1. Màn danh sách segment:** bảng các segment đã tạo — Tên, Mô tả, Số KH khớp, Loại (động/tĩnh), Cập nhật, Trạng thái. Nút "Tạo segment mới".

**5.2. Rule builder (tạo/sửa segment):**
- Đặt tên + mô tả segment.
- Khu vực thêm điều kiện theo trường, hỗ trợ **AND/OR** và nhóm điều kiện. Trường có thể lọc: customer_type, customer_group, province, loyalty_tier, churn_score, cod_risk_score, return_rate, total_orders, total_revenue, rfm_group, last_transaction_date, roles.
- Mỗi điều kiện: chọn trường → toán tử (=, >, <, ≥, ≤, chứa, thuộc) → giá trị.
- **Preview:** số khách hàng khớp (cập nhật khi sửa rule) + bảng mẫu vài KH khớp.

**Data mẫu segment:**
```
Danh sách:
- "KHL giá trị cao"          | customer_group=KHL AND total_revenue>1.000.000.000 | 1.240 KH | Động | hôm nay
- "Nguy cơ rời bỏ"           | churn_score>=70                                     | 8.560 KH | Động | hôm qua
- "Tỷ lệ hoàn cao cần xử lý" | return_rate>0.2 AND total_orders>50                 | 3.120 KH | Động | 3 ngày trước
- "Chủ shop hoạt động cao"   | roles chứa SHOP_OWNER AND avg_monthly_orders>50     | 920 KH   | Động | 1 tuần trước

Ví dụ rule builder cho "Nguy cơ rời bỏ":
  ( churn_score >= 70 )  AND  ( last_transaction_date < 90 ngày trước )
  → Preview: 8.560 khách hàng khớp
  → bảng mẫu: Phạm Minh Tuấn (churn 78), Đỗ Văn Khải (churn 72)...
```

---

## C. Yêu cầu chất lượng (toàn bộ)
- Một app liền mạch, sidebar 5 chức năng, chuyển qua lại được.
- Responsive xuống mobile (bảng chuyển dạng card khi hẹp).
- Focus bàn phím nhìn thấy; tôn trọng reduced-motion.
- Masking dữ liệu nhạy cảm đúng spec (CCCD, tài khoản thanh toán).
- Badge & màu trạng thái nhất quán theo mục B.
- Tiền tệ VND có phân tách hàng nghìn.
- Khối Consent (CN4-L) và cảnh báo ERROR (CN1), cảnh báo gộp nhầm (CN3) làm nổi bật thị giác.
- Prototype mô phỏng — nút lưu/gửi/merge/retry chỉ hiện xác nhận, không gọi backend.

---

## D. Prompt gợi ý cho Claude Design
> "Dựng interactive prototype theo design brief đính kèm — hệ thống CDP nội bộ của VNPost, một app liền mạch với sidebar 5 chức năng (Kafka Integration, Customer Management, Identity Resolution, Customer 360, Segment Management). Dùng design tokens và data mẫu trong brief. Customer Management click một KH sẽ mở Customer 360. Ưu tiên độ rõ ràng kiểu operator console, không phải landing page."

> Gợi ý nếu Claude Design dựng nặng: làm lần lượt từng chức năng theo thứ tự CN1→CN5, mỗi lần một màn, rồi ghép vào chung sidebar.
