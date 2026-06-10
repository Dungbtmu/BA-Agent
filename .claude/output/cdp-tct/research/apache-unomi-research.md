# Apache Unomi — Nghiên cứu tham khảo cho dự án CDP VNPost/TCT

**Ngày nghiên cứu:** 2026-06-10  
**Phương pháp:** Deep research — 6 góc tìm kiếm, 22 nguồn, 81 claims, verify adversarial 25 claims (9 confirmed / 16 bị bác)  
**Mục đích:** Đánh giá Apache Unomi làm tham khảo kiến trúc và tính năng cho CDP VNPost/TCT

---

## Verdict tổng thể

Apache Unomi là CDP open-source trưởng thành về tính năng cốt lõi — profile management, event-driven rule engine, segment engine, và GDPR privacy tools — được xây dựng trên kiến trúc OSGi có thể mở rộng qua plugin.

**Kết luận:** Unomi khả thi làm core engine CDP nhưng đòi hỏi đầu tư đáng kể vào custom integration. Không phải lựa chọn low-effort. Phù hợp nếu team có năng lực Java/OSGi.

---

## Tính năng được xác nhận (confidence: high)

### 1. Profile Schema mở rộng qua OSGi Plugin

Custom profile và session properties được định nghĩa qua JSON files trong plugin OSGi:
- Đặt tại `META-INF/cxs/properties/profiles` và `META-INF/cxs/properties/sessions`
- Deploy tự động khi bundle khởi động — không cần sửa core Unomi
- Extension types hỗ trợ: ActionTypes, ConditionTypes, Personas, PropertyTypes, Rules, Scorings, Segments, ValueTypes

**Nguồn:** GitHub master branch (maintained đến 2024), unomi.apache.org/manual/latest/

### 2. Rule Engine — Conditional Action trên Event

- Cho phép thực thi actions phản ứng với incoming events
- Conditions có thể test event data, profile, hoặc session
- Groovy Actions: hot-deploy logic không cần restart server — compile tại runtime bởi GroovyActionsService

**Giới hạn quan trọng:** External API integration bị hạn chế bởi OSGi class loader — chỉ services accessible bởi GroovyActionDispatcher mới dùng được từ Groovy script.

**Nguồn:** writing-plugins.adoc (GitHub), unomi.apache.org/integrations.html

### 3. Java OSGi Plugin — Deep Core Integration

- Cung cấp ConditionEvaluator và ConditionESQueryBuilder cho custom conditions
- PersistenceService interface cho custom persistence types
- Truy cập toàn bộ Unomi API
- Tài liệu định vị đây là "last resort" khi Groovy Actions không đủ

**Nguồn:** writing-plugins.adoc, unomi.apache.org/integrations.html

### 4. Identity Resolution qua ProfileAlias

- Nhiều identifier từ các touchpoint khác nhau tham chiếu một profile duy nhất
- ProfileAlias là feature của Unomi 2.0.0+ (thay thế legacy 'mergedWith')
- Merge là rule-triggered (MergeProfileOnPropertyAction + shared identifier như email)
- **Không phải probabilistic stitching tự động**

**Nguồn:** unomi.apache.org/manual/latest/, deepwiki.com/apache/unomi, GitHub (ProfileServiceImpl, MergeProfilesOnPropertyAction.java)

### 5. GDPR / Privacy Compliance — Built-in

Ba endpoint privacy sẵn có:
1. `POST /cxs/privacy/profiles/{profileID}/anonymize` — ẩn danh hóa
2. `DELETE /cxs/privacy/profiles/{profileID}?withData=false` — xóa dữ liệu
3. `GET /cxs/client/myprofile.[json,csv,yaml,text]` — download dữ liệu cá nhân

Consent management system: tracking consents, consent type definitions (Section 10 & 11 trong manual).

**Lưu ý bảo mật:** Docs khuyến nghị proxy các endpoints này để tránh unauthorized access.

**Nguồn:** unomi-manual-2_7_x.pdf, privacy.adoc (GitHub)

### 6. Apache Unomi V3 — Release cuối 2025

Tính năng mới trong V3:
- **Multi-tenancy:** Complete Tenant Isolation, Dual API Key System (V3.1)
- **Enhanced clustering:** Elasticsearch-based distributed task scheduling — thay thế Hazelcast và Karaf Cellar
- **V2 API compatibility mode**

**Nguồn:** Apache mailing list (Aug–Dec 2025): msg09295, msg09302, msg09693

### 7. Real-time Data Processing (medium confidence)

Capabilities được mô tả: real-time decisioning, unified profiles, segmentation/scoring, privacy built-in.

**Lưu ý:** "Real-time" chưa được xác nhận qua independent performance benchmark. Cần validate với production load.

---

## Điểm yếu và giới hạn

| Điểm yếu | Mức độ ảnh hưởng |
|---|---|
| Physical channel (quầy, giao hàng) không có native connector — cần custom integration layer | **Cao** với VNPost/TCT |
| Identity stitching chỉ rule-triggered — không tự động với khách vãng lai chưa định danh | **Cao** |
| Groovy Actions bị giới hạn class loader — không gọi external API tùy ý | **Trung bình** — dùng Java plugin thay thế |
| Rất ít connector out-of-box — mọi integration phải build thủ công | **Cao** |
| Community nhỏ — risk long-term maintainability | **Trung bình** |

---

## Claims bị bác (quan trọng cần biết)

- ❌ Profile schema **không** mở rộng qua REST API thuần — phải qua OSGi plugin
- ❌ Segment membership **không** confirm là real-time tự động sau mỗi property change
- ❌ MVEL scripting đã deprecated — chỉ còn Groovy (với class loader constraint)
- ❌ Unomi **không** chính thức implement OASIS CXS Standard
- ❌ Event types **không** hoàn toàn open-schema — có validation mechanism
- ❌ OSGi hot-deploy **không** thực sự zero-downtime theo cách được claim

---

## Tính năng Unomi có thể tham khảo thiết kế cho CDP TCT

> Phần này không đánh giá "có dùng Unomi không" — mà trả lời câu hỏi: **nếu tự thiết kế CDP cho VNPost, nên học pattern nào từ Unomi?**

---

### T1. ProfileAlias → Pattern thiết kế Identity Layer cho bưu chính

**Pattern Unomi:** Một profile có thể có vô số alias từ nhiều nguồn — không merge ngay mà dùng alias làm lookup bridge. Merge chỉ xảy ra khi có shared anchor (email/SĐT) được xác nhận qua rule.

**Áp dụng cho VNPost:**

VNPost có đúng bài toán này: cùng một người gửi xuất hiện ở CAS (mã KHL), MyVNPost (user ID), PayPost (mã tài chính), PostID — 4 định danh rời rạc.

Nên thiết kế theo pattern tương tự:
- **PostID** = master profile ID (anchor)
- **SĐT / email** = merge key — khi trùng nhau thì alias về cùng profile
- **Mã KHL (CAS), User ID (MyVNPost), Mã tài chính (PayPost)** = alias — lookup bridge, không cần tồn tại trước trong profile

Merge không cần xảy ra ngay — có thể lazy-merge khi có event xác nhận (lần đầu đăng nhập app sau khi giao dịch tại quầy với cùng SĐT).

**Điểm cần giải quyết thêm (không có trong Unomi):** Người nhận xuất hiện trong hàng triệu giao dịch nhưng không có tài khoản VNPost. Unomi không có sẵn pattern cho "anonymous recipient" — VNPost cần thiết kế riêng: tạo lightweight profile cho người nhận dựa trên SĐT, không đòi consent đầy đủ, chỉ dùng cho phân tích địa chỉ rủi ro cao và COD fraud (UC-04, UC-06).

---

### T2. Rule Engine (Event → Condition → Action) → Pattern thiết kế Trigger Engine

**Pattern Unomi:** `Event nhận vào → kiểm tra Condition trên profile/session → thực thi Action (cập nhật profile, gọi downstream)`. Mỗi Rule là một cấu hình JSON, không cần code deploy lại.

**Áp dụng trực tiếp cho VNPost:**

| Use case | Event trigger | Condition | Action |
|---|---|---|---|
| **UC-02 Anti-Churn** | Cuối tuần: tổng hợp sản lượng 4 tuần | Sản lượng tuần này < 70% trung bình 4 tuần trước + là KHL | Gắn tag `at-risk`, đưa vào segment `churn-alert`, trigger chiến dịch CSKH |
| **UC-03 Win-Back** | Scheduler hàng ngày | Không có giao dịch trong 60 ngày + từng là khách hoạt động | Đưa vào segment `inactive`, trigger SMS win-back với ưu đãi theo lịch sử dịch vụ |
| **UC-04 COD Risk** | Event giao hàng thất bại | Lần thất bại thứ 3 tại cùng địa chỉ trong 30 ngày | Gắn tag `high-risk-address`, cảnh báo trước khi nhận đơn tiếp theo |
| **UC-06 Fraud** | Event tạo đơn COD | SĐT người nhận liên kết với >5 tên khác nhau trong 30 ngày | Flag `fraud-suspect`, tạo alert cho team vận hành |

**Insight thiết kế quan trọng từ Unomi:** Ngưỡng nghiệp vụ (70%, 60 ngày, 3 lần) phải là tham số cấu hình được — Marketing tự điều chỉnh mà không cần IT. Unomi làm được điều này qua JSON rule config. CDP VNPost nên có "Rule Config Layer" tương tự.

**Điều cần tránh lại từ Unomi:** Groovy Actions bị giới hạn class loader khi gọi external API. Action "trigger chiến dịch CSKH" thực chất là gọi CRM, gọi SMS gateway, gọi Zalo OA — không được nhúng vào rule script. Nên thiết kế: Rule Engine chỉ publish event vào message queue (Kafka) → downstream systems tự consume và thực thi.

**Cơ chế timing:** Với UC-02 và UC-03, batch daily là đủ — không cần real-time segment. Chỉ UC-06 (fraud detection) mới cần xử lý gần real-time khi tạo đơn. Unomi xử lý tất cả theo event real-time — VNPost không cần làm phức tạp đến vậy, nên thiết kế hai tầng: near-real-time cho fraud/alert, batch daily cho anti-churn/win-back.

---

### T3. Groovy Actions Hot-deploy → Pattern thiết kế Business Logic Layer tách khỏi core

**Pattern Unomi:** Logic nghiệp vụ (tính điểm, cập nhật thuộc tính, phân loại khách hàng) tách ra khỏi core engine — thay đổi mà không cần build lại toàn bộ hệ thống.

**Áp dụng cho VNPost:** Quy tắc phân loại khách hàng thay đổi thường xuyên theo chiến lược kinh doanh:
- Ngưỡng "at-risk" hôm nay: giảm 30% sản lượng → tháng sau có thể đổi thành 20%
- Điểm số loyalty tier: công thức thay đổi theo từng đợt chính sách
- Định nghĩa "inactive": 60 ngày với KHL, 90 ngày với khách hàng cá nhân

Nếu logic này nằm trong core code, mỗi lần đổi phải release. Nên thiết kế theo pattern:
```
Tầng 1: Core Engine (bất biến) — xử lý event, lưu profile, tính toán
Tầng 2: Rule Config Layer (thay đổi được) — Marketing/BA cấu hình ngưỡng, điều kiện, action
Tầng 3: Downstream Connector (thay đổi được) — cấu hình kênh output: Zalo, SMS, CRM
```

**Giới hạn của Unomi cần học để tránh:** Groovy bị giới hạn class loader khi ra ngoài sandbox OSGi. Tương đương trong CDP VNPost: không để logic gọi MPITS, PayPost, BSS nằm trong rule script — tách thành connector service riêng.

---

### T4. Consent Management → Tính năng nên tham khảo gần như nguyên xi

**Pattern Unomi đã giải quyết đúng:**
- Consent per profile, per consent type (marketing, analytics, location...)
- Consent Expiration — tự động yêu cầu renew sau thời hạn
- Preference Center — người dùng tự quản lý consent
- Right to be Forgotten — xóa toàn bộ dữ liệu trong 72 giờ
- Audit Trail — ghi lịch sử thay đổi consent

**Mapping sang yêu cầu NĐ 13 cho VNPost:**

| Yêu cầu NĐ 13 | Pattern Unomi tương ứng | Ghi chú |
|---|---|---|
| Consent rõ ràng trước khi xử lý | Consent API per visitor | Người gửi consent khi tạo tài khoản MyVNPost hoặc giao dịch tại quầy |
| Consent bằng văn bản điện tử với dữ liệu nhạy cảm | Consent Type riêng cho location + financial | Cần thêm field: timestamp, IP/device, phiên bản điều khoản |
| Quyền xóa dữ liệu trong 72 giờ | DELETE endpoint | Phải cascade xóa qua tất cả hệ thống nguồn — không chỉ xóa trong CDP |
| Mục đích rõ ràng | Consent Type = mục đích cụ thể | Mỗi use case cần consent riêng: "phân tích hành vi", "gửi marketing", "chia sẻ với đối tác" |
| Thông báo vi phạm 72 giờ | Audit Trail | Cần thêm: alert system khi phát hiện truy cập bất thường |

**Điểm cần thiết kế thêm cho VNPost (không có trong Unomi):**
- Consent theo vai trò: người gửi và người nhận có consent scope khác nhau — người nhận không có tài khoản, VNPost chỉ xử lý thông tin họ trong giới hạn thực hiện dịch vụ
- Consent cho dữ liệu GPS bưu tá: đây là dữ liệu nhân viên nội bộ, không phải khách hàng — cần policy riêng, không phải consent flow của khách hàng
- Consent inheritance: khi khách hàng đồng ý trên MyVNPost, điều đó có apply cho giao dịch tại quầy CAS không? Cần định nghĩa rõ scope

---

### T5. Scope / Multi-tenancy (V3) → Pattern thiết kế phân quyền theo tỉnh/vùng

**Pattern Unomi V3:** "Complete Tenant Isolation" — mỗi scope có dữ liệu, API key, và user riêng biệt. Không có dữ liệu leak giữa scope.

**Áp dụng cho VNPost (câu hỏi Q9 trong Domain Brief):** Đơn vị tỉnh A có được xem dữ liệu KH của tỉnh B không? Đây là bài toán governance quan trọng chưa được chốt.

Gợi ý thiết kế theo pattern Unomi Scope:
- **Scope = Vùng địa lý** (Bắc, Trung, Nam) hoặc **Scope = Tỉnh/TP** — tùy mức độ phân quyền
- Data Engineer TCT có thể xem toàn bộ (cross-scope)
- Marketing tỉnh A chỉ xem được scope tỉnh A
- Ban lãnh đạo TCT xem được aggregated view (không phải raw profile)
- KHL lớn (Shopee Seller gửi toàn quốc) cần cross-scope view — cần role đặc biệt

**Lưu ý thực tế:** Unomi V3.1 multi-tenancy chưa confirm GA tính đến Jun 2026. Nếu triển khai Unomi thực tế, cần kiểm tra version hiện tại tại unomi.apache.org/downloads. Nếu thiết kế CDP riêng (không dùng Unomi), pattern Scope isolation vẫn là tham chiếu kiến trúc tốt.

---

### Tổng hợp: Nên học gì, nên tránh gì

| Tính năng Unomi | Nên học | Nên tránh / Điều chỉnh |
|---|---|---|
| ProfileAlias pattern | Học nguyên xi cho digital channels | Bổ sung thêm "anonymous recipient" profile cho bưu chính |
| Rule Engine (Event→Condition→Action) | Học pattern, áp dụng cho 4 use cases | Tách action gọi external ra message queue — không nhúng vào rule script |
| Groovy hot-deploy | Học ý tưởng "business logic layer tách khỏi core" | Không dùng Groovy vì class loader constraint — dùng config-driven approach thay thế |
| Consent Management (GDPR tooling) | Học gần như nguyên xi | Thêm: consent theo vai trò gửi/nhận, consent GPS bưu tá, consent scope per touchpoint |
| Scope / Multi-tenancy V3 | Học pattern isolation | Định nghĩa scope theo nhu cầu governance VNPost, không nhất thiết theo cách Unomi định nghĩa |
| Segment Engine | Học khái niệm dynamic segment | Thiết kế 2 tầng: near-real-time (fraud) + batch daily (anti-churn) — không cần real-time cho tất cả |

---

## Câu hỏi mở chưa có câu trả lời

1. **Segment recalculation có real-time không?** Critical gap: nếu event "giao hàng thành công" không ngay lập tức cập nhật segment, toàn bộ real-time personalization use case bị ảnh hưởng
2. **V3.1 multi-tenancy đã GA chưa** (tính đến Jun 2026)? Breaking changes V2→V3 ảnh hưởng migration thế nào?
3. **Scale với workload VNPost** (hàng triệu event giao nhận/ngày)? Chưa có benchmark cho high-volume logistics
4. **TCO so với alternatives** (RudderStack open-source, Jitsu)? Khi tính cả effort build custom plugin

---

## Nguồn tham khảo chính

| Nguồn | Loại | Góc nghiên cứu |
|---|---|---|
| github.com/apache/unomi/blob/master/manual/src/main/asciidoc/writing-plugins.adoc | Primary | Khả năng mở rộng |
| unomi.apache.org/manual/latest/ | Primary | Tổng quan tính năng |
| downloads.apache.org/unomi/2.7.0/unomi-manual-2_7_x.pdf | Primary | Privacy & compliance |
| unomi.apache.org/integrations.html | Primary | Tích hợp & hệ sinh thái |
| mail-archive.com/dev@unomi.apache.org (msg09295, 09302, 09693) | Primary | V3 roadmap & release |
| deepwiki.com/apache/unomi/1-overview | Secondary | Kiến trúc tổng quan |
