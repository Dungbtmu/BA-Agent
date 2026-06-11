# Apache Unomi — Tài liệu tham khảo toàn diện

**Ngày:** 2026-06-11  
**Phương pháp:** Deep research — 103 agent, 21 nguồn, 95 claims, adversarial verify 25 claims → 6 xác nhận, 19 bác bỏ  
**Đối tượng đọc:** Phần 1 dành cho BA/PO; Phần 2 dành cho Dev/Tech Lead  
**Quy ước:** ✅ = đã xác minh từ primary source | ⚠️ = chưa xác minh đủ — cần research thêm

---

## Phần 1 — Nghiệp vụ (BA/PO)

### 1.1 Unomi là gì và giải bài toán gì

Apache Unomi là phần mềm CDP (Customer Data Platform) mã nguồn mở, do tổ chức Apache phát triển. Đây là **reference implementation của chuẩn OASIS CDP v1.0 (2019)** — tức là Unomi được xây dựng để minh hoạ cách một CDP nên hoạt động theo tiêu chuẩn quốc tế. ✅

**Bài toán cốt lõi Unomi giải:**  
Một tổ chức có nhiều hệ thống — website, app, CRM, hệ thống bán hàng — mỗi nơi lưu dữ liệu khách hàng riêng lẻ với mã định danh khác nhau. Unomi cung cấp lớp trung gian để gộp các mảnh dữ liệu đó về một hồ sơ duy nhất, phân nhóm khách hàng theo điều kiện, và tự động kích hoạt hành động khi có sự kiện xảy ra.

**Lưu ý quan trọng:** OASIS CXS Technical Committee đóng cửa tháng 12/2022 — chuẩn CDP v1.0 không còn được phát triển tiếp. Unomi vẫn đang phát triển độc lập (v2.x, v3.x) nhưng không còn gắn với một chuẩn quốc tế đang evolve. ✅

---

### 1.2 Sáu tính năng cốt lõi

#### Tính năng 1 — Gộp nhiều ID về một hồ sơ (Profile Unification)

**Bài toán giải:** Cùng một người có thể được nhận diện bằng nhiều mã khác nhau ở nhiều hệ thống — email ở website, user ID ở app, mã thành viên ở CRM. Unomi cho phép nối tất cả về một hồ sơ duy nhất mà không cần gộp database.

**Cách hoạt động:** Khi phát hiện hai profile có cùng một thuộc tính (ví dụ: cùng email), hệ thống sẽ copy toàn bộ dữ liệu vào profile chính (master), tạo một "cầu nối alias" giữ lại ID cũ để tra cứu, rồi xóa profile phụ. Ai dùng ID cũ tra cứu vẫn tìm ra đúng hồ sơ nhờ alias. ✅

**Quy tắc gộp phải khai báo rõ:** Không có AI hay machine learning tự đoán — phải định nghĩa rõ "nếu thuộc tính X trùng nhau thì gộp". Có thể cấu hình profile nào trở thành master (profile hiện tại hay profile trong event). ✅

**Giới hạn nghiệp vụ:**
- Không tự nhận diện cùng người nếu không có thuộc tính trùng khớp (ví dụ: cùng người dùng tên khác nhau ở hai nơi)
- Không phù hợp cho người dùng hoàn toàn anonymous (không có email, phone, hay ID nào trùng)

---

#### Tính năng 2 — Tự động kích hoạt hành động khi có sự kiện (Rule Engine)

**Bài toán giải:** Khi có sự kiện xảy ra (khách hàng đặt hàng, thất bại thanh toán, không hoạt động trong 30 ngày), hệ thống cần tự động phản ứng mà không cần người ngồi theo dõi thủ công.

**Cách hoạt động:** Mỗi Rule định nghĩa theo công thức: **Sự kiện xảy ra → Kiểm tra điều kiện → Thực hiện hành động**. Ví dụ: "khi sự kiện đặt hàng thất bại xảy ra, và đây là lần thứ 3 trong tháng → gắn nhãn 'rủi ro cao' vào hồ sơ khách hàng".

**Thiết kế quan trọng:** Unomi chỉ cập nhật hồ sơ và đẩy tín hiệu ra ngoài — không gọi trực tiếp SMS hay email. Hệ thống CRM hay gateway nhận tín hiệu đó và tự lo việc gửi. Tách biệt này giúp Unomi không bị phụ thuộc vào từng kênh output cụ thể.

**Giới hạn nghiệp vụ:** ⚠️ Chưa xác minh được Rule Engine xử lý đồng bộ (synchronous, tức thì) hay bất đồng bộ (asynchronous, có độ trễ). Đây là câu hỏi quan trọng cho các use case cần phản ứng trong vài giây.

---

#### Tính năng 3 — Phân nhóm khách hàng tự động (Segment Engine)

**Bài toán giải:** Marketing cần nhóm khách hàng theo điều kiện để chạy chiến dịch — ví dụ: "khách hàng mua trên 5 lần trong 3 tháng gần nhất", "khách hàng chưa tương tác trong 60 ngày".

**Cách hoạt động:** Segment là một tập điều kiện logic — profile nào thỏa mãn điều kiện thì thuộc nhóm đó. Segment không phải danh sách cứng, mà là truy vấn động: khi dữ liệu hồ sơ thay đổi, tư cách thành viên trong segment tự cập nhật theo.

**Giới hạn nghiệp vụ:** ⚠️ Chưa xác minh tần suất recalculate — là real-time (ngay khi có event) hay batch (chạy theo lịch mỗi đêm). Đây là điểm cần làm rõ trước khi thiết kế use case cần phân nhóm tức thì.

---

#### Tính năng 4 — Quản lý đồng ý sử dụng dữ liệu (Consent Management)

**Bài toán giải:** Các quy định về bảo vệ dữ liệu cá nhân (GDPR ở châu Âu, và tương đương ở các quốc gia khác) yêu cầu tổ chức phải xin phép người dùng trước khi dùng dữ liệu của họ cho từng mục đích khác nhau.

**Cách hoạt động:** Unomi có module quản lý đồng ý tích hợp sẵn, hỗ trợ khách hàng bật/tắt từng loại đồng ý. Khi người dùng yêu cầu xóa dữ liệu, Unomi xử lý trong hệ thống của mình.

**Giới hạn nghiệp vụ:** ⚠️ Chi tiết về các loại consent type, cơ chế Right to be Forgotten (xóa lan sang hệ thống khác), và audit trail chưa được xác minh đủ từ primary source. Không nên thiết kế compliance flow dựa trên giả định — cần test trực tiếp hoặc đọc source code.

**Điểm quan trọng:** Unomi chỉ xóa dữ liệu trong chính nó — không tự lan sang các hệ thống nguồn bên ngoài. Tổ chức cần tự thiết kế cơ chế lan truyền xóa nếu dữ liệu tồn tại ở nhiều chỗ.

---

#### Tính năng 5 — Tính điểm tín nhiệm tự động (Scoring)

**Bài toán giải:** Thay vì gắn nhãn nhị phân (có rủi ro / không có rủi ro), tổ chức cần xếp hạng khách hàng theo thang điểm để phân loại mức độ xử lý phù hợp.

**Cách hoạt động:** Mỗi hồ sơ khách hàng có thể được gán điểm số theo quy tắc — điểm tăng khi hành vi tốt, giảm khi hành vi xấu. Điểm tự cập nhật mỗi khi có sự kiện mới.

**Giá trị so với nhãn:** Khách hàng điểm 20 và điểm 80 đều có nhãn "rủi ro" — nhưng mức độ xử lý khác nhau. Điểm số cho phép tinh chỉnh ngưỡng phản ứng mà không cần tạo thêm nhãn mới.

**Giới hạn nghiệp vụ:** ⚠️ Cơ chế tính điểm cụ thể (công thức, tần suất cập nhật, có decay theo thời gian không) chưa được xác minh đủ.

---

#### Tính năng 6 — Thêm trường dữ liệu mới mà không cần IT (Schema Extension)

**Bài toán giải:** Khi nghiệp vụ cần thêm một chỉ số mới vào hồ sơ khách hàng — ví dụ "tỷ lệ hoàn hàng 30 ngày", "điểm tín nhiệm theo ngành" — thông thường phải nhờ IT sửa database schema và release hệ thống, mất vài tuần.

**Cách hoạt động:** Unomi cho phép khai báo property mới bằng file JSON, không cần viết code Java. Sau khi deploy file cấu hình, trường mới xuất hiện ngay trong hồ sơ của tất cả khách hàng. ✅

**Giới hạn nghiệp vụ:** Vẫn cần đóng gói file JSON vào một OSGi bundle và deploy — không phải đặt file raw bất kỳ đâu. Về quy trình, vẫn cần IT thực hiện deploy, nhưng không cần release code lõi hay thay đổi database schema.

---

### 1.3 Phù hợp và không phù hợp cho loại tổ chức nào

**Phù hợp:**
- Tổ chức có đội kỹ thuật Java mạnh, muốn kiểm soát hoàn toàn dữ liệu khách hàng
- Tổ chức có yêu cầu tuân thủ dữ liệu nghiêm ngặt (không muốn dữ liệu lên cloud bên thứ ba)
- Dự án muốn học pattern thiết kế CDP để tự xây — Unomi là tài liệu sống tốt nhất về cách CDP hoạt động
- Tổ chức đang dùng Jahia DXP (Unomi là engine tích hợp sẵn trong Jahia)

**Không phù hợp:**
- Tổ chức không có đội kỹ thuật Java/OSGi — Unomi đòi hỏi chuyên môn vận hành cao
- Tổ chức cần tính năng out-of-the-box cho marketing (không có giao diện kéo thả cho Marketing user)
- Use case cần hành vi e-commerce phức tạp (cart abandonment, product recommendation real-time) — Unomi không có sẵn
- Tổ chức cần time-to-value nhanh — setup và tùy chỉnh Unomi tốn nhiều thời gian hơn CDP thương mại

---

### 1.4 So sánh với CDP thương mại

| Tiêu chí | Unomi | Salesforce CDP | Segment | mParticle |
|---|---|---|---|---|
| **Chi phí** | Miễn phí (tự vận hành) | Rất cao (~$100k+/năm) | Trung bình–cao | Trung bình–cao |
| **Kiểm soát dữ liệu** | Hoàn toàn — self-hosted | Dữ liệu lên Salesforce cloud | Dữ liệu lên Segment cloud | Dữ liệu lên mParticle cloud |
| **Tích hợp sẵn** | Rất ít — tự viết connector | Rất nhiều (Salesforce ecosystem) | Rất nhiều (300+ destination) | Rất nhiều (mobile-first) |
| **Giao diện Marketing** | Không có — headless hoàn toàn | Đầy đủ drag-and-drop | Giới hạn | Giới hạn |
| **Đòi hỏi kỹ thuật** | Rất cao (Java/OSGi) | Thấp (SaaS) | Thấp–trung (SDK) | Trung (SDK + config) |
| **Tuân thủ dữ liệu** | Tự kiểm soát hoàn toàn | Phụ thuộc Salesforce policy | Phụ thuộc Segment policy | Phụ thuộc mParticle policy |
| **Khả năng tùy chỉnh** | Rất cao — mã nguồn mở | Thấp — closed platform | Trung | Trung |
| **Time-to-value** | Dài (vài tháng) | Ngắn (vài tuần) | Ngắn (vài ngày–tuần) | Trung |

> ⚠️ Bảng so sánh tổng hợp từ nguồn thứ cấp (PeerSpot, blog) — không qua xác minh adversarial. Dùng làm định hướng tham khảo, không dùng làm cơ sở quyết định mua/build.

---

## Phần 2 — Kỹ thuật (Dev/Tech Lead)

### 2.1 Kiến trúc tổng thể

**Runtime:** Apache Karaf — một OSGi container. Toàn bộ Unomi và các plugin chạy dưới dạng OSGi bundle trong Karaf. ✅

**Storage:** Elasticsearch làm backend lưu trữ duy nhất. Elasticsearch đóng vai trò:
- Lưu toàn bộ Profile, Session, Event, Segment definition, Rule definition
- Schema động (dynamically extensible) — thêm property mới vào document không cần migration
- Từ Unomi 2.x: có thêm JSON Schema validation pipeline cho incoming events — không hoàn toàn schema-free nữa ✅

**Lưu ý về Elasticsearch version:** ⚠️ Claim rằng Unomi 2.x yêu cầu chính xác Elasticsearch 7.17.5 đã bị bác bỏ. Unomi 3.0 chuyển từ deprecated rest-client sang official Elasticsearch Java client, có migration path sang Elasticsearch 9 — nhưng chi tiết version requirements cụ thể chưa được xác minh.

---

### 2.2 Data Model

```
Profile          ← hồ sơ tổng hợp của một người, cross-application
  └─ Session     ← một phiên tương tác cụ thể, có scope reference
       └─ Event  ← một hành động cụ thể trong session đó
  
Segment          ← định nghĩa điều kiện, không phải danh sách cứng
Rule             ← Event + Condition → Action
ProfileAlias     ← bảng tra cứu: ID cũ → Profile master hiện tại
```

**Profile:** Hồ sơ tổng hợp, tồn tại lâu dài, lưu mọi thuộc tính của một người. ⚠️ Quan hệ giữa Profile và Scope (có bị giới hạn theo application/tenant không) chưa được xác minh — claim rằng Profile là cross-application bị bác bỏ trong research này.

**Session:** Đại diện cho một phiên tương tác cụ thể (ví dụ: một lần truy cập website, một lần đặt hàng). Session có tham chiếu đến Scope (application nào tạo ra session này).

**Event:** Một hành động cụ thể xảy ra trong một Session — giao hàng thất bại, đăng nhập, tạo đơn. Event kích hoạt Rule Engine chạy.

**Segment:** Không phải danh sách khách hàng — là một tập điều kiện logic. Profile thỏa mãn điều kiện thì được tính là thành viên của segment đó. Tư cách thành viên thay đổi theo dữ liệu profile.

**Rule:** Định nghĩa phản ứng tự động: Event loại nào + Condition gì → Action gì.

---

### 2.3 Profile Merge — Cơ chế alias+delete (đã xác minh)

Từ Unomi v2.0+, merge hoạt động theo 3 bước: ✅

```
1. Copy properties    → toàn bộ dữ liệu của profile phụ copy sang profile master
2. Tạo ProfileAlias   → alias: {id: <ID profile phụ>, profileId: <ID profile master>}
3. Xóa profile phụ   → profile phụ bị delete khỏi Elasticsearch
```

**Tra cứu sau merge:** Bất kỳ request nào dùng ID cũ của profile phụ → Unomi tra bảng ProfileAlias → tìm ra profile master → trả về đúng hồ sơ. Many-to-one: nhiều alias có thể trỏ vào một master.

**Chọn master:** Configurable per Rule — parameter `forceEventProfileAsMaster`: ✅
- `false` (default): profile đầu tiên trong kết quả query trở thành master
- `true`: profile của incoming event trở thành master

**Field `mergedWith` trên Profile:** Đã deprecated từ v2.0.0 — vẫn còn trong API nhưng không được dùng nữa vì merged profiles bị xóa hoàn toàn (không còn tồn tại để `mergedWith` trỏ tới). ✅

**Giới hạn kỹ thuật cần lưu ý:** ⚠️ Có report về giới hạn 50 profile trong một lần merge query (TODO UNOMI-776) — chưa được xác minh đủ để khẳng định, nhưng đây là rủi ro cần kiểm tra nếu dự án có scenario gộp số lượng lớn.

---

### 2.4 Plugin Architecture — META-INF/cxs/ convention (đã xác minh)

Mọi extension cho Unomi đều là **OSGi bundle** và phải đặt định nghĩa tại thư mục `META-INF/cxs/` bên trong JAR: ✅

```
META-INF/cxs/
├── actions/          ← định nghĩa ActionType (JSON)
├── conditions/       ← định nghĩa ConditionType (JSON)
├── personas/         ← định nghĩa Persona (JSON)
├── properties/
│   ├── profiles/     ← định nghĩa ProfilePropertyType (JSON)
│   └── sessions/     ← định nghĩa SessionPropertyType (JSON)
├── rules/            ← định nghĩa Rule (JSON)
└── segments/         ← định nghĩa Segment (JSON)
```

Unomi đọc tất cả file JSON trong các thư mục này khi bundle được load và tự đăng ký vào runtime.

---

### 2.5 Schema Extension — thêm property không cần Java (đã xác minh)

Để thêm property mới vào Profile hoặc Session: ✅

1. Tạo file JSON PropertyType descriptor, ví dụ `deliveryFailureCount.json`:
```json
{
  "metadata": {
    "id": "deliveryFailureCount",
    "name": "Delivery Failure Count",
    "systemTags": ["profileProperties"]
  },
  "type": "integer",
  "defaultValue": 0,
  "rank": "100"
}
```

2. Đặt file tại `META-INF/cxs/properties/profiles/` trong OSGi bundle
3. Deploy bundle — property xuất hiện ngay trên tất cả Profile

**Quan trọng:** Bundle wrapper vẫn bắt buộc — không thể drop raw JSON file bất kỳ đâu. Tuy nhiên bundle này không cần Java class nào — chỉ cần file JSON đúng cấu trúc + OSGi manifest. ✅

---

### 2.6 Tính năng chưa xác minh đủ — cần research thêm

| Tính năng | Câu hỏi cụ thể | Cách xác minh |
|---|---|---|
| **Rule Engine execution** | Synchronous (tức thì tại request) hay asynchronous (có queue/delay)? | Test trực tiếp hoặc đọc ContextServlet.java |
| **Groovy Actions** | Từ v2.5+ có deploy Groovy action qua REST mà không cần bundle không? | Đọc GroovyActionService + test /cxs/groovyActions endpoint |
| **Segment recalculation** | Real-time per-event hay batch job? Tần suất nếu batch? | Đọc SegmentService + ScheduledTask definition |
| **Consent Management** | 4 consent types cụ thể là gì? Right to be Forgotten cascade hoạt động thế nào? | Đọc ConsentService + PrivacyService source code |
| **Scope/Multi-tenancy** | Profile có bị scope-bound không? Scope cô lập dữ liệu ở tầng nào? | Đọc ScopeService + test cross-scope query |
| **REST API** | Authentication mechanism cụ thể (JAAS, token, API key)? Rate limiting có không? | Đọc REST security config + test với Karaf console |
| **Elasticsearch version** | Unomi 2.x tương thích ES version nào? Unomi 3.x migration path cụ thể? | Đọc pom.xml dependencies + release notes |
| **Session/Event reassignment sau merge** | Có async delay sau merge không? Sessions của profile bị merge có reassign sang master không? | Đọc TaskExecutor configuration trong MergeProfilesOnPropertyAction |

---

## Phụ lục — Nguồn primary đáng tin cậy

| Nguồn | Loại | Dùng cho |
|---|---|---|
| [unomi.apache.org](https://unomi.apache.org/) | Official site | Tổng quan, use case tuyên bố |
| [unomi.apache.org/manual/latest/](https://unomi.apache.org/manual/latest/) | Official manual | Data model, tính năng, cấu hình |
| [github.com/apache/unomi — datamodel.adoc](https://github.com/apache/unomi/blob/master/manual/src/main/asciidoc/datamodel.adoc) | Source | Data model chi tiết |
| [github.com/apache/unomi — writing-plugins.adoc](https://github.com/apache/unomi/blob/master/manual/src/main/asciidoc/writing-plugins.adoc) | Source | Plugin architecture |
| [github.com/apache/unomi — recipes.adoc](https://github.com/apache/unomi/blob/master/manual/src/main/asciidoc/recipes.adoc) | Source | Profile merge mechanism |
| [MergeProfilesOnPropertyAction.java](https://github.com/apache/unomi/blob/master/plugins/baseplugin/src/main/java/org/apache/unomi/plugins/baseplugin/actions/MergeProfilesOnPropertyAction.java) | Source code | Merge implementation |
| [Profile.java](https://github.com/apache/unomi/blob/master/api/src/main/java/org/apache/unomi/api/Profile.java) | Source code | Profile API, deprecated fields |
| Apache Unomi mailing list (mail-archive.com) | Forum | Changelog, feature discussion, intent |

> Blog, CMSWire, Dremio wiki, third-party comparison site: **không đủ tin cậy** cho claims kỹ thuật. Toàn bộ claims từ các nguồn này trong research này đều bị bác bỏ.
