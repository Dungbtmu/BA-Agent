# Domain Brief — CDP (Customer Data Platform) cho TCT

**Ngày tạo:** 2026-05-28
**Người thực hiện:** ba-research-agent
**Dự án:** Hệ thống CDP cho TCT (Tổ chức/Tập đoàn lớn tại Việt Nam)
**Phiên bản:** v1.0

---

## 1. Tổng quan domain

### CDP là gì

Customer Data Platform (Nền tảng Dữ liệu Khách hàng) là phần mềm thu thập, hợp nhất và kích hoạt dữ liệu khách hàng từ nhiều nguồn khác nhau — tạo ra một hồ sơ khách hàng duy nhất, liên tục cập nhật, có thể dùng được bởi các nhóm Marketing, Sales, Product và AI.

### Ba giai đoạn vận hành cốt lõi

1. **Thu thập dữ liệu (Data Collection):** Tích hợp với web, mobile app, CRM, POS, email, hệ thống nội bộ, IoT — kéo dữ liệu hành vi và giao dịch từ mọi điểm chạm của khách hàng theo thời gian thực.

2. **Hợp nhất dữ liệu (Data Unification / Identity Resolution):** Ghép nối các định danh rời rạc (cookie trình duyệt, device ID, email, số điện thoại, mã khách hàng CRM) từ nhiều nguồn và thiết bị thành một hồ sơ duy nhất (Unified Customer Profile).

3. **Kích hoạt dữ liệu (Data Activation):** Cung cấp hồ sơ và phân khúc đã xử lý cho các hệ thống downstream — công cụ email, quảng cáo, personalization engine, AI agent — để triển khai trải nghiệm cá nhân hóa.

### Đặc thù khi triển khai cho tập đoàn lớn (phù hợp bối cảnh TCT)

- Dữ liệu phân tán qua nhiều công ty con, nhiều ngành nghề, nhiều hệ thống IT khác nhau
- Identity resolution phức tạp hơn vì cùng một khách hàng có thể tương tác qua nhiều kênh của nhiều đơn vị
- Cần phân tách dữ liệu theo đơn vị nghiệp vụ (Scope) nhưng vẫn có hồ sơ hợp nhất ở cấp tập đoàn
- Triển khai doanh nghiệp lớn thường kéo dài 16–24 tuần, có thể phân kỳ 3–6 tháng
- Phải tuân thủ quy định bảo vệ dữ liệu cá nhân (tại VN: Nghị định 13/2023/NĐ-CP về PDPD)

**Nguồn tham khảo:** [CDP.com — What is a CDP](https://cdp.com/basics/what-is-a-customer-data-platform-cdp/), [Hightouch — What is a CDP](https://hightouch.com/blog/what-is-a-customer-data-platform-cdp)

---

## 2. Các actor chính

### Người dùng vận hành hệ thống

| Actor | Vai trò | Quan tâm chính |
|---|---|---|
| **Marketer / Chuyên viên Marketing** | Tạo phân khúc khách hàng, thiết lập chiến dịch, xem báo cáo hành vi | Dễ dùng, không cần IT hỗ trợ, phân khúc chính xác |
| **Chuyên viên Phân tích Dữ liệu (Data Analyst)** | Truy vấn hồ sơ, xây dựng dashboard, phân tích hành vi, đo lường chiến dịch | Độ chính xác dữ liệu, khả năng truy vấn linh hoạt |
| **Kỹ sư Dữ liệu (Data Engineer)** | Thiết lập pipeline tích hợp, quản lý schema, xử lý lỗi dữ liệu | Tốc độ xử lý real-time, độ ổn định pipeline, khả năng mở rộng |
| **Quản trị viên Hệ thống (System Admin)** | Quản lý quyền truy cập, cấu hình hệ thống, theo dõi hiệu năng | Bảo mật, audit log, phân quyền chi tiết |

### Người hưởng lợi / Người ra quyết định

| Actor | Vai trò | Quan tâm chính |
|---|---|---|
| **Giám đốc Marketing (CMO/Marketing Director)** | Phê duyệt chiến lược sử dụng CDP, đánh giá ROI | Hiệu quả chiến dịch, tỷ lệ chuyển đổi, chi phí |
| **Giám đốc IT / CTO** | Phê duyệt kiến trúc kỹ thuật, đảm bảo an toàn hệ thống | Tích hợp hạ tầng hiện có, bảo mật, tuân thủ |
| **Giám đốc Kinh doanh các đơn vị** | Sử dụng insight từ CDP cho quyết định kinh doanh | Báo cáo tổng quan, phân tích phân khúc khách hàng |
| **Khách hàng cuối** | Đối tượng dữ liệu được thu thập và xử lý | Trải nghiệm cá nhân hóa, quyền riêng tư dữ liệu |

### Hệ thống ngoài tích hợp (Source Systems)

- Hệ thống CRM (quản lý quan hệ khách hàng)
- Mobile App / Website (tracking hành vi)
- POS / Hệ thống bán lẻ (giao dịch offline)
- Email Marketing Platform
- Hệ thống Call Center / Support
- ERP / Core Banking (nếu có)
- Kênh quảng cáo (Google Ads, Facebook Ads...)

---

## 3. Quy trình nghiệp vụ cốt lõi

### 3.1 Thu thập và nạp dữ liệu (Data Ingestion)

```
Nguồn dữ liệu → SDK/API/Connector → Event Queue → CDP Processing Engine → Profile Store
```

- Real-time: SDK web/mobile gửi event khi người dùng tương tác (click, view, purchase...)
- Batch: Import dữ liệu lịch sử từ CRM, ERP qua file hoặc API
- Streaming: Nhận log hệ thống, dữ liệu định vị theo luồng liên tục

**Tự động hóa bằng phần mềm:** Toàn bộ — không cần nhập tay

### 3.2 Hợp nhất hồ sơ khách hàng (Identity Resolution & Profile Merging)

```
Nhiều định danh rời rạc → Matching Engine (email, phone, device ID...) → Unified Profile
```

- Xác định cùng một khách hàng qua các kênh khác nhau (cross-device, cross-channel)
- Gộp hồ sơ trùng lặp (duplicate merging)
- Xây dựng "Golden Record" — hồ sơ chính xác nhất

**Tự động hóa bằng phần mềm:** Một phần tự động, có thể cần rule thủ công cho edge case

### 3.3 Phân khúc khách hàng (Segmentation)

```
Unified Profile + Behavioral Data → Rule-based / ML-based → Segment → Activation
```

- Tạo phân khúc tĩnh (tập cố định) hoặc động (tự cập nhật theo thời gian thực)
- Điều kiện phân khúc: nhân khẩu học, hành vi, lịch sử giao dịch, vị trí địa lý...
- Phân khúc tự động bằng AI/ML cho các mô hình phức tạp (churn prediction, LTV scoring)

**Tự động hóa bằng phần mềm:** Cao — chạy liên tục, cập nhật tự động

### 3.4 Kích hoạt dữ liệu (Data Activation & Personalization)

```
Segment → Trigger Rule → Downstream System (Email/Ads/Push notification/Chatbot)
```

- Gửi phân khúc sang các kênh marketing để nhắm mục tiêu
- Kích hoạt thông báo, offer cá nhân hóa theo hành vi real-time
- Cung cấp dữ liệu khách hàng cho AI agent / recommendation engine

**Tự động hóa bằng phần mềm:** Cao

### 3.5 Phân tích và báo cáo (Analytics & Reporting)

```
Profile Store + Event History → Analytics Engine → Dashboard / Report
```

- Phân tích hành vi theo nhóm khách hàng
- Đo lường hiệu quả chiến dịch (campaign attribution)
- Theo dõi KPI: tỷ lệ chuyển đổi, doanh thu theo phân khúc, CLV (Customer Lifetime Value)

**Tự động hóa bằng phần mềm:** Một phần — cần cấu hình dashboard và metric

---

## 4. Pain point phổ biến

### Khi xây dựng / triển khai CDP

- **Dữ liệu phân mảnh:** Mỗi hệ thống nguồn có schema, quy ước đặt tên và tần suất cập nhật khác nhau — tích hợp tốn nhiều thời gian và dễ phát sinh lỗi
- **Identity resolution không hoàn chỉnh:** Khách hàng dùng nhiều email, thiết bị, mua qua nhiều kênh → hồ sơ bị tách rời, dữ liệu không phản ánh đúng thực tế
- **Hiểu lầm về phạm vi:** Tổ chức thường nhầm rằng mua CDP là xong, bỏ qua giai đoạn triển khai phức tạp (thường mất 16–24 tuần cho enterprise)
- **Chi phí cao và phức tạp khi onboarding:** Các giải pháp enterprise CDP thường đắt và cần chuyên gia tư vấn hỗ trợ triển khai

### Khi vận hành CDP

- **Chất lượng dữ liệu kém:** Dữ liệu đầu vào không chuẩn hóa → hồ sơ sai, phân khúc không chính xác
- **Thiếu governance dữ liệu:** Không có quy trình quản lý data schema thống nhất → entropy theo thời gian
- **Khó đo ROI:** Tác động CDP khó quy ra số liệu cụ thể — đặc biệt trong B2B hoặc các hành trình khách hàng dài
- **Quyền riêng tư và tuân thủ:** Thu thập dữ liệu hành vi real-time cần tuân thủ quy định — tại VN là Nghị định 13/2023/NĐ-CP, cần consent management rõ ràng

### Đặc thù cho tập đoàn lớn (bối cảnh TCT)

- Dữ liệu phân tán qua nhiều đơn vị thành viên, hệ thống IT khác nhau, thậm chí legacy system không có API
- Xung đột về ownership dữ liệu giữa các đơn vị — ai sở hữu hồ sơ khách hàng?
- Governance phức tạp: quyền truy cập dữ liệu của đơn vị A có được xem dữ liệu của đơn vị B không?

**Nguồn tham khảo:** [CDP.com — CDP Challenges](https://cdp.com/articles/common-cdp-challenges/), [Treasure Data — CDP Challenges](https://blog.treasuredata.com/blog/2022/06/10/biggest-customer-data-platform-challenges-and-how-to-solve-them-tda/)

---

## 5. Thuật ngữ quan trọng

| Tiếng Việt | Tiếng Anh | Giải thích |
|---|---|---|
| Hồ sơ khách hàng hợp nhất | Unified Customer Profile | Bản ghi duy nhất của một khách hàng, tổng hợp từ mọi nguồn dữ liệu |
| Phân giải danh tính | Identity Resolution | Quá trình ghép nối các định danh rời rạc (email, device ID, số ĐT) về cùng một người |
| Hồ sơ vàng | Golden Record | Phiên bản hồ sơ khách hàng chính xác nhất sau khi đã hợp nhất và làm sạch |
| Phân khúc khách hàng | Segment | Nhóm khách hàng có chung đặc điểm hoặc hành vi, dùng để nhắm mục tiêu |
| Sự kiện | Event | Một hành động cụ thể của người dùng được ghi lại (click, mua hàng, đăng nhập...) |
| Kích hoạt dữ liệu | Data Activation | Đưa dữ liệu/phân khúc từ CDP vào các hệ thống để tạo trải nghiệm cá nhân hóa |
| Điểm tiếp xúc | Touchpoint | Mỗi kênh/nơi khách hàng tương tác với thương hiệu (web, app, cửa hàng, call center...) |
| Dữ liệu bên thứ nhất | First-party Data | Dữ liệu trực tiếp từ khách hàng của doanh nghiệp, không qua trung gian |
| Luồng dữ liệu thời gian thực | Real-time Streaming | Dữ liệu được xử lý ngay khi phát sinh, không đợi theo lô (batch) |
| Quy tắc | Rule | Logic tự động: khi Điều kiện X thỏa mãn → thực hiện Hành động Y |
| Phạm vi | Scope | Đơn vị tổ chức logic trong CDP — mỗi website, app hoặc đơn vị kinh doanh có thể là một Scope riêng |
| Persona | Persona | Hồ sơ mẫu (archetype) đại diện cho một nhóm khách hàng mục tiêu, dùng để định hướng chiến lược |
| Mục tiêu | Goal | Chỉ tiêu chuyển đổi cần đo (ví dụ: hoàn thành mua hàng, đăng ký tài khoản) |
| Chiến dịch | Campaign | Tập hợp các rule và tracking được thiết lập cho một sáng kiến marketing |
| Hành động | Action | Phản hồi tự động được kích hoạt khi Rule thỏa điều kiện (gửi email, cập nhật hồ sơ...) |
| Dữ liệu có cấu trúc | Structured Data | Dữ liệu có schema cố định (bảng, JSON schema...) — ví dụ: thông tin đơn hàng |
| Dữ liệu phi cấu trúc | Unstructured Data | Dữ liệu không có schema cố định — ví dụ: log hệ thống, lịch sử định vị, click stream |
| CLV / LTV | Customer Lifetime Value | Giá trị doanh thu ước tính của một khách hàng trong toàn bộ vòng đời mối quan hệ |
| Quản lý đồng ý | Consent Management | Cơ chế ghi nhận và thực thi sự đồng ý của khách hàng về thu thập/xử lý dữ liệu |

---

## 6. Benchmark / Tham chiếu

### 6.1 Apache Unomi — Tham chiếu bắt buộc của dự án

**Website:** [https://unomi.apache.org](https://unomi.apache.org)

Apache Unomi là một **CDP mã nguồn mở** do Apache Software Foundation phát triển, được thiết kế như một **implementation chuẩn của OASIS CDP Specification**. Đây là lựa chọn phù hợp cho tổ chức muốn kiểm soát hoàn toàn dữ liệu, không bị phụ thuộc vendor.

**Các khái niệm core của Unomi:**

| Khái niệm | Mô tả |
|---|---|
| **Profile** | Hồ sơ khách hàng — đơn vị trung tâm, tích lũy thuộc tính, phân khúc, lịch sử hành vi theo thời gian |
| **Event** | Sự kiện người dùng (page view, click, form submit...) — kích hoạt Rule và cập nhật Profile |
| **Segment** | Phân khúc động — tự động phân loại Profile theo điều kiện cấu hình sẵn |
| **Rule** | Logic tự động: Condition → Action — xử lý khi Event đến |
| **Scope** | Container tổ chức logic — mỗi website, app, đơn vị kinh doanh là một Scope riêng |
| **Persona** | Hồ sơ archetype — đại diện cho một nhóm khách hàng mục tiêu để định hướng nội dung |
| **Goal** | Mục tiêu chuyển đổi cần theo dõi trong một chiến dịch |
| **Campaign** | Tập hợp Goal và Rule cho một sáng kiến marketing cụ thể |
| **Condition** | Điều kiện logic đánh giá thuộc tính Profile hoặc Event để quyết định có kích hoạt Rule không |
| **Action** | Hành động thực thi khi Rule khớp — cập nhật Profile, gọi API ngoài, thay đổi UX |
| **Source** | Nguồn gốc của Event — xác định kênh nào gửi sự kiện |

**Kiến trúc kỹ thuật Unomi:**
- Backend: **Elasticsearch** (lưu trữ Profile, Segment, Event)
- Runtime: **Apache Karaf** (OSGi container — hỗ trợ plugin mở rộng theo module)
- API: **REST/JSON** — đầy đủ endpoint cho Profile, Event, Segment, Rule, Campaign, Goal
- Tích hợp: Horizontal scalability, plugin architecture với JSON descriptor
- Privacy: Tích hợp sẵn **GDPR consent management** và data portability
- AI-ready: Cung cấp ngữ cảnh khách hàng sạch, hợp nhất, real-time cho LLM, chatbot, recommendation engine

**Phù hợp với yêu cầu TCT:**
- Không phụ thuộc vendor, không bị lock-in
- Hỗ trợ multi-scope (phù hợp tập đoàn nhiều đơn vị)
- Xử lý real-time event qua REST API
- Privacy-first (cần thiết theo Nghị định 13/2023/NĐ-CP)

---

### 6.2 Các CDP thương mại phổ biến (để tham chiếu so sánh)

| Sản phẩm | Điểm nổi bật | Phân khúc | Tham khảo |
|---|---|---|---|
| **Segment (Twilio)** | Pipeline dữ liệu sạch, 700+ tích hợp, phù hợp developer-first | Mid-market đến Enterprise | [segment.com](https://segment.com) |
| **Tealium** | Giải pháp toàn diện (CDP + Tag Management + ML), 1,300+ tích hợp, privacy mạnh | Enterprise lớn | [tealium.com](https://tealium.com) |
| **mParticle** | Chuyên sâu mobile-first, real-time data orchestration, dùng bởi HBO Max, JetBlue | Enterprise mobile-heavy | [mparticle.com](https://mparticle.com) |
| **Adobe Real-Time CDP** | Tích hợp sâu với Adobe ecosystem, real-time profile, phù hợp marketing-heavy | Enterprise lớn có Adobe stack | [adobe.com](https://business.adobe.com) |
| **RudderStack** | Mã nguồn mở, triển khai nhanh (8–12 tuần), data warehouse native | Mid-market, engineering-led | [rudderstack.com](https://rudderstack.com) |
| **Amperity** | Identity Resolution chuyên sâu cho dữ liệu phức tạp, messy, legacy | Enterprise có dữ liệu lịch sử phức tạp | [amperity.com](https://amperity.com) |

**Nguồn tham khảo:** [houseofmartech.com — CDP Comparison 2025](https://houseofmartech.com/blog/mparticle-vs-segment-vs-tealium-2025-cdp-comparison), [CDP.com — Best Vendors](https://cdp.com/basics/cdp-vendors/), [Hightouch — Best Enterprise CDPs](https://hightouch.com/blog/best-enterprise-cdps)

---

## 7. Câu hỏi clarify cần hỏi BA/PO

Dựa trên những gì đã research, dưới đây là các câu hỏi tập trung vào phần còn chưa rõ của dự án TCT. BA cần clarify những điểm này trước khi thiết kế solution.

### Về phạm vi và bối cảnh tổ chức

**Q1. TCT có bao nhiêu đơn vị thành viên và bao nhiêu hệ thống IT cần tích hợp vào CDP?**
> Lý do hỏi: Quy mô tích hợp quyết định độ phức tạp của pipeline và thời gian triển khai. Một tập đoàn có 5–10 đơn vị với hệ thống khác nhau sẽ khác hoàn toàn so với 50+ đơn vị.

**Q2. CDP này phục vụ toàn bộ tập đoàn (cấp group) hay chỉ một hoặc vài đơn vị thành viên trước?**
> Lý do hỏi: Cần xác định có cần kiến trúc multi-tenant (mỗi đơn vị là một Scope riêng) hay bắt đầu từ một đơn vị cụ thể để pilot.

**Q3. Cấu trúc IT hiện tại của TCT như thế nào — IT tập trung (centralized) hay phân tán tại từng đơn vị?**
> Lý do hỏi: Ảnh hưởng trực tiếp đến cách thiết kế quyền truy cập dữ liệu, governance, và ai là chủ sở hữu dữ liệu khách hàng ở cấp nào.

---

### Về dữ liệu và nguồn tích hợp

**Q4. Những hệ thống nguồn nào đã tồn tại và sẵn sàng tích hợp? Hệ thống nào có API, hệ thống nào chỉ có file export (CSV, Excel)?**
> Lý do hỏi: Xác định phương thức ingestion phù hợp (real-time API vs batch file) và ưu tiên thứ tự tích hợp.

**Q5. Khi nói "lịch sử định vị" — đây là dữ liệu GPS từ app di động của TCT, hay từ bên thứ ba (telco, đối tác)? Có thỏa thuận pháp lý để sử dụng không?**
> Lý do hỏi: Dữ liệu định vị thuộc nhóm nhạy cảm cao, cần xác nhận nguồn gốc và tính hợp pháp trước khi thiết kế pipeline thu thập.

**Q6. Hiện tại dữ liệu khách hàng đang được lưu ở đâu — Data Warehouse, Data Lake, hay phân tán trong từng ứng dụng? TCT đã có hạ tầng dữ liệu nào sẵn chưa (Hadoop, Kafka, Spark...)?**
> Lý do hỏi: CDP sẽ bổ sung hay thay thế hạ tầng hiện có? Cần hiểu để tránh duplicate và thiết kế đúng điểm tích hợp.

---

### Về định danh và hợp nhất hồ sơ

**Q7. Một khách hàng của TCT được nhận diện bằng gì — mã khách hàng duy nhất (Customer ID) toàn tập đoàn, hay mỗi đơn vị có ID riêng? Hiện có hệ thống Master Data Management (MDM) chưa?**
> Lý do hỏi: Đây là yếu tố then chốt quyết định độ phức tạp của identity resolution. Nếu chưa có Customer ID thống nhất, đây phải là bài toán giải quyết trước hoặc cùng lúc với CDP.

---

### Về mục đích sử dụng và người dùng

**Q8. Ai là người dùng chính của CDP sau khi xây xong — đội Marketing, đội Data/Analytics, hay cả hai? Họ có kỹ năng kỹ thuật ở mức nào (self-service hay cần IT hỗ trợ mỗi lần)?**
> Lý do hỏi: Ảnh hưởng lớn đến thiết kế UX của giao diện quản trị — CDP kiểu "marketer-friendly" khác hoàn toàn CDP kiểu "engineer-first".

**Q9. Use case ưu tiên số một khi ra mắt CDP là gì — hợp nhất hồ sơ (có dữ liệu đẹp để nhìn), phân khúc cho chiến dịch marketing, hay real-time personalization trên app/web?**
> Lý do hỏi: Tránh scope creep và xác định MVP. Mỗi use case có yêu cầu kỹ thuật và thời gian xây dựng khác nhau.

---

### Về tuân thủ và bảo mật

**Q10. TCT đã có chính sách thu thập và xử lý dữ liệu cá nhân (theo Nghị định 13/2023/NĐ-CP) chưa? Hệ thống CDP có cần tích hợp cơ chế xin đồng ý (consent) của khách hàng không?**
> Lý do hỏi: Consent management là yêu cầu bắt buộc theo pháp luật VN. Nếu chưa có, cần xây dựng cùng với CDP — không thể thu thập dữ liệu hành vi mà không có cơ sở pháp lý.

---

### Câu hỏi bổ sung (nếu còn thời gian)

**Q11. TCT có ưu tiên giải pháp mã nguồn mở (như Apache Unomi) hay thương mại? Đã có vendor nào được cân nhắc chưa?**

**Q12. Kỳ vọng về thời gian go-live của phase đầu tiên là bao lâu? Có deadline cứng không (ra mắt sản phẩm, sự kiện, fiscal year...)?**

---

## Assumption được nêu rõ

- `[Assumption]` TCT là một tập đoàn/tổng công ty có nhiều đơn vị thành viên, hoạt động đa ngành. Nếu TCT là một doanh nghiệp đơn lẻ, một số câu hỏi về multi-tenant và governance có thể không áp dụng.
- `[Assumption]` "Cấu trúc công nghệ thông tin hiện tại" trong yêu cầu PO ngụ ý TCT đã có IT infrastructure sẵn — mức độ trưởng thành của hạ tầng này chưa rõ, cần clarify.
- `[Assumption]` Apache Unomi được đề xuất là nền tảng kỹ thuật — tuy nhiên chưa rõ đây là quyết định chốt hay chỉ là tham khảo. Cần xác nhận.
- `[Assumption]` Bối cảnh pháp lý áp dụng là Việt Nam — dẫn chiếu Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân.

---

## Nguồn tham khảo chính

- [Apache Unomi Official Website](https://unomi.apache.org)
- [CDP.com — What is a CDP Guide 2026](https://cdp.com/basics/what-is-a-customer-data-platform-cdp/)
- [CDP.com — Common CDP Challenges](https://cdp.com/articles/common-cdp-challenges/)
- [CDP.com — Best CDP Vendors](https://cdp.com/basics/cdp-vendors/)
- [Hightouch — What is a CDP](https://hightouch.com/blog/what-is-a-customer-data-platform-cdp)
- [Hightouch — Best Enterprise CDPs](https://hightouch.com/blog/best-enterprise-cdps)
- [RudderStack — Why Engineering needs to own CDP](https://www.rudderstack.com/blog/why-engineering-and-it-need-to-own-the-cdp/)
- [houseofmartech.com — CDP Comparison 2025](https://houseofmartech.com/blog/mparticle-vs-segment-vs-tealium-2025-cdp-comparison)
- [LiveRamp — Identity Resolution vs CDP](https://liveramp.com/blog/identity-resolution-vs-cdp-where-cdps-miss-the-mark)
- [Adobe — What is a CDP](https://business.adobe.com/blog/basics/what-is-a-customer-data-platform)

---

**Bước tiếp theo:** Dùng file này làm input cho `ba-clarification-agent` — tiến hành buổi clarification với BA/PO của TCT, ưu tiên các câu hỏi Q1, Q4, Q7, Q9, Q10 vì đây là những điểm ảnh hưởng lớn nhất đến kiến trúc và phạm vi hệ thống.
