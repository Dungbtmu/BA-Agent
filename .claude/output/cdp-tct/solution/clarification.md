# Clarification — CDP VNPost

**Ngày tạo:** 2026-06-05
**Người thực hiện:** ba-clarification-agent
**Dự án:** Customer Data Platform cho VNPost / TCT
**Input:** Domain Brief v2.1 + Context BA đã biết (chưa có buổi họp chính thức với VNPost)
**Trạng thái:** Sẵn sàng cho buổi clarification đầu tiên với PO/IT VNPost

---

## Problem Statement

### Problem Statement chính

VNPost đang vận hành mạng lưới ~13.000 điểm phục vụ, xử lý hàng chục triệu bưu gửi/tháng qua 8+ hệ thống IT riêng biệt — nhưng không có hồ sơ khách hàng thống nhất nào tồn tại trên toàn hệ thống.

Hậu quả trực tiếp:

- **Đội Sales/CSKH không biết khách hàng nào đang có nguy cơ rời đi** vì không thể nhìn thấy xu hướng sụt giảm sản lượng xuyên hệ thống — mỗi hệ thống chỉ thấy một mảnh dữ liệu, không ai thấy bức tranh tổng.
- **Không đo được ROI của bất kỳ chiến dịch marketing nào** vì không ghép được hành vi trước và sau chiến dịch của cùng một khách hàng.
- **Tỷ lệ hoàn hàng 10–15% gây chi phí vận hành 2 chiều** mà không có cơ chế phát hiện pattern rủi ro từ trước.
- **Cùng một khách hàng doanh nghiệp TMĐT có thể tồn tại dưới 4 định danh khác nhau** (mã KHL trong CAS, user ID trong MyVNPost, mã tài chính trong PayPost, ID trên sàn TMĐT) — không hệ thống nào biết đây là cùng một người.

### Business Objective

Xây dựng CDP làm nền tảng để VNPost có thể:

1. **Ưu tiên MVP:** Giảm churn khách hàng TMĐT lớn (KHL) — phát hiện sớm dấu hiệu rời đi và kích hoạt chiến dịch giữ chân trước khi mất.
2. **Nền tảng dài hạn:** Một hồ sơ khách hàng hợp nhất (Customer 360) phục vụ toàn bộ các use case phân tích, marketing, và vận hành.

`[Assumption: Business Objective #1 (Anti-Churn KHL) được giả định là ưu tiên dựa trên phân tích Domain Brief — chưa được VNPost xác nhận. Câu hỏi Q5 trong phần clarify sẽ làm rõ điều này.]`

### Success Metrics (sơ bộ — cần xác nhận với VNPost)

| Metric | Mục tiêu |
|---|---|
| Tỷ lệ giữ chân KHL (doanh nghiệp TMĐT) | Tăng ít nhất 5% trong 12 tháng đầu sau khi Anti-Churn use case đi vào vận hành |
| Thời gian phát hiện KHL at-risk | Từ "không biết" xuống dưới 7 ngày kể từ khi có dấu hiệu sụt giảm |
| Tỷ lệ hoàn hàng | Giảm ít nhất 1–2% trong phân khúc người gửi được phân tích rủi ro |
| Hồ sơ khách hàng được hợp nhất | Ít nhất 80% KHL (khách hàng doanh nghiệp) có hồ sơ hợp nhất sau phase 1 |

`[Assumption: Tất cả metric trên đều là giả định — chưa có số mục tiêu cụ thể từ VNPost. Cần xác nhận trong buổi họp.]`

### Scope sơ bộ (cần confirm)

**Khả năng trong scope:**
- Thu thập và hợp nhất dữ liệu từ các hệ thống ưu tiên (MPITS, CAS/Portal KHL, MyVNPost, PayPost, PNS Phát)
- Hồ sơ khách hàng 360 cho đối tượng người gửi (cá nhân và doanh nghiệp TMĐT/KHL)
- Phân khúc động và cảnh báo Anti-Churn
- Consent Management và audit trail tuân thủ NĐ 13

**Có thể ngoài scope (cần xác nhận):**
- Hồ sơ cho người nhận (consignee) — chưa có tài khoản VNPost, câu hỏi consent phức tạp
- Tích hợp BCCP/TMS/WMS trong MVP (hệ thống legacy, khó API)
- Fraud detection trong MVP (cần đủ dữ liệu lịch sử trước)
- Phân quyền dữ liệu theo tỉnh/vùng trong giai đoạn đầu

---

## Những gì đã rõ

### Confirmed từ Domain Brief v2.1 (nghiên cứu công khai, chưa được VNPost xác nhận trực tiếp)

**Về hệ sinh thái IT:**
- VNPost có ít nhất 8 hệ thống IT chính: MPITS, CAS, MyVNPost, PNS/DingDong, BCCP, TMS, WMS, PayPost, PostID — mỗi hệ thống có định danh khách hàng riêng
- MPITS được xác định là hub tích hợp trung tâm, đã kết nối API với sàn TMĐT (Shopee, Lazada, TikTok Shop)
- BCCP, TMS, WMS nhiều khả năng là hệ thống legacy, ít có REST API chuẩn
- PostID tồn tại như một hệ thống định danh, nhưng chưa rõ độ phủ

**Về nghiệp vụ:**
- Luồng nghiệp vụ 8 giai đoạn từ tạo đơn đến thu/trả tiền COD đã được map rõ
- COD chiếm ~70–80% giao dịch TMĐT tại Việt Nam — đây là đặc thù quan trọng nhất
- Tỷ lệ hoàn hàng 10–15% là pain point đo được
- Mạng lưới ~13.000 điểm phục vụ trên 63 tỉnh thành

**Về pháp lý:**
- Nghị định 13/2023/NĐ-CP áp dụng bắt buộc — dữ liệu vị trí GPS và dữ liệu COD/tài chính là dữ liệu nhạy cảm
- CDP phải có Consent Management, Right to be Forgotten, và audit trail từ ngày 1

**Về phương án triển khai (phân tích, chưa chốt):**
- 3 phương án đã được phân tích: Build Unomi / Buy SaaS (Antsomi) / Partner + Unomi
- Domain Brief khuyến nghị Option 3 (Partner + Unomi) là cân bằng nhất
- SaaS cloud quốc tế (Segment, mParticle) gần như bị loại do yêu cầu data sovereignty

**Về 6 use cases:**
- UC-01 Customer 360, UC-02 Anti-Churn, UC-03 Win-Back, UC-04 Giảm hoàn, UC-05 Cross-sell, UC-06 Fraud Detection đều đã được phân tích giá trị kinh doanh

### Chưa có xác nhận từ VNPost

- Toàn bộ thông tin trên đến từ nguồn công khai và nghiên cứu — chưa được IT/PO VNPost xác nhận
- Chưa có buổi họp chính thức nào với VNPost
- Chưa có PRD hay tài liệu yêu cầu từ phía VNPost
- Chưa biết ngân sách được phê duyệt là bao nhiêu
- Chưa biết timeline go-live mà ban lãnh đạo kỳ vọng

---

## Domain Gap Analysis

**Domain:** Customer Data Platform (CDP)
**Client:** VNPost — Tổng Công Ty Bưu Điện Việt Nam (tổng công ty nhà nước, đa nghiệp vụ, hệ thống legacy)
**Domain Brief:** `.claude/output/cdp-tct/research/domain-brief.md` (v2.1)

### Checklist so sánh Domain điển hình vs VNPost

| Hạng mục | Mô hình CDP điển hình | VNPost — Đã biết | Trạng thái |
|---|---|---|---|
| **Identity Resolution** | Ghép nối qua email/SĐT/cookie — 1 khách hàng = 1 hồ sơ | Mỗi hệ thống dùng định danh riêng; cùng 1 KH có thể có 4+ ID khác nhau | VARIANT — phức tạp hơn điển hình vì tính 2 chiều (người gửi + người nhận) |
| **Actor chính — Marketing User** | Chuyên viên marketing tự tạo segment, campaign — không cần IT | Chưa biết VNPost có đội Marketing/CRM chuyên biệt không | MISSING — cần xác nhận |
| **Actor chính — Data Engineer** | Vận hành pipeline tích hợp thường xuyên | Chưa biết VNPost có Data Engineer chuyên CDP không | MISSING — cần xác nhận |
| **Nguồn dữ liệu** | Web, app, CRM — thường có API chuẩn | 8+ hệ thống, nhiều hệ thống legacy (BCCP, TMS, WMS) không có REST API | VARIANT — nguy cơ batch-only cho nhiều nguồn |
| **Consent Management** | Tích hợp vào platform hoặc mua thêm | Chưa có — cần build hoặc dùng Unomi | MISSING — nhưng đã xác định giải pháp |
| **Customer Profile** | Profile cho người dùng đã đăng ký | VNPost có cả người gửi (có tài khoản) và người nhận (không có tài khoản) | VARIANT — đặc thù bưu chính, cần quyết định scope |
| **Activation Channels** | Email, SMS, push notification, personalization | Chưa biết VNPost đang dùng kênh nào (Zalo OA? SMS? Email?) | MISSING |
| **Governance / Phân quyền** | Phân quyền theo team/phòng ban | VNPost có 63 tỉnh thành — phân quyền theo đơn vị địa phương phức tạp hơn | VARIANT |
| **Data Sovereignty** | Linh hoạt (SaaS OK) | Tổng công ty nhà nước — dữ liệu phải nằm trong hạ tầng VNPost | NEW — ràng buộc cứng không có trong CDP điển hình |
| **Procurement** | Mua phần mềm trực tiếp | Quy trình đấu thầu nhà nước — ảnh hưởng toàn bộ timeline và cách chọn vendor | NEW — ràng buộc cứng đặc thù nhà nước |
| **COD as Data Source** | Không có trong CDP điển hình | COD chiếm 70–80% giao dịch — nguồn dữ liệu tài chính quan trọng nhất | NEW — đặc thù bưu chính Việt Nam |
| **MVP Use Case** | Tuỳ domain — thường là Customer 360 hoặc segmentation | Chưa được VNPost xác nhận use case ưu tiên | MISSING — CRITICAL |
| **Budget / Ngân sách** | Thường xác định từ đầu | Chưa biết ngân sách được phê duyệt | MISSING — CRITICAL |
| **Real-time vs Batch** | CDP điển hình ưu tiên real-time | Nhiều hệ thống VNPost legacy chỉ có thể batch — trade-off phải quyết định | VARIANT |

---

### CRITICAL Gaps (phải hỏi trước khi clarify solution)

| Gap | CDP điển hình | VNPost — Thực tế | Câu hỏi cần đặt |
|---|---|---|---|
| **GAP-C1: Identity Resolution — Người nhận không có tài khoản** | CDP điển hình chỉ track người dùng đã đăng ký; identity resolution qua email/SĐT đã xác thực | VNPost có 2 chủ thể trong mỗi giao dịch: người gửi (có tài khoản) và người nhận (không có tài khoản, xuất hiện trong hàng triệu giao dịch) | VNPost muốn xây hồ sơ cho người nhận (consignee) hay chỉ người gửi (shipper)? Nếu có cả người nhận: cơ chế consent cho người nhận chưa từng đăng ký là gì? |
| **GAP-C2: MVP Use Case chưa xác định** | CDP có giá trị khác nhau tuỳ use case đầu tiên — Customer 360 cần tất cả tích hợp, Anti-Churn chỉ cần subset | 6 use cases đã phân tích nhưng chưa có xác nhận ưu tiên từ VNPost | Trong 6 use cases (Customer 360, Anti-Churn, Win-Back, Giảm hoàn, Cross-sell, Fraud Detection): VNPost muốn thấy kết quả gì đầu tiên trong 6 tháng? Điều gì đang gây đau nhất cho business hiện tại? |
| **GAP-C3: Customer ID thống nhất — Có hay không?** | CDP điển hình thường có một ID chuẩn để bắt đầu identity resolution; nếu không có thì khâu ghép nối phức tạp và tốn thời gian | VNPost có PostID nhưng chưa biết PostID có phủ toàn bộ hệ thống hay chỉ app users | PostID đang là định danh chuẩn cho bao nhiêu % khách hàng toàn hệ thống? Khách hàng giao dịch qua quầy (không có app) có PostID không? |

### MAJOR Gaps (ảnh hưởng tính năng/tích hợp)

| Gap | CDP điển hình | VNPost — Thực tế | Câu hỏi cần đặt |
|---|---|---|---|
| **GAP-M1: Năng lực Data Team nội bộ** | CDP vận hành tốt khi có Data Engineer (pipeline) + Marketing Analyst (segment) + Admin (governance) | Chưa biết VNPost có đội này chưa hay phải build từ đầu | VNPost hiện có Data Engineer, Data Analyst chuyên CDP chưa? Hay đang plan tuyển? Điều này ảnh hưởng trực tiếp đến lựa chọn Build vs Buy vs Partner. |
| **GAP-M2: Kênh kích hoạt (Activation Channel)** | CDP tạo ra phân khúc và segment xong phải "activate" qua kênh giao tiếp cụ thể | Chưa biết VNPost đang dùng kênh nào để liên hệ KH: SMS? Zalo OA? Email? Push notification qua MyVNPost? | Hiện tại VNPost đang dùng kênh giao tiếp nào với KH (SMS, Zalo, email, push app)? Có tích hợp sẵn Marketing Automation tool nào không? |
| **GAP-M3: MPITS API — Gateway hay tích hợp riêng lẻ?** | CDP điển hình kết nối qua 1 gateway dữ liệu nếu có — giảm số lượng tích hợp | MPITS đã kết nối với sàn TMĐT qua API, nhưng chưa biết API này có expose cho CDP không | MPITS đã expose API cho sàn TMĐT. API đó có thể cho CDP kết nối vào không? Nếu có, CDP có thể lấy được dữ liệu tổng hợp từ MPITS thay vì tích hợp từng hệ thống con? |
| **GAP-M4: Phương án triển khai chưa chốt** | CDP điển hình không có ràng buộc nhà nước — chọn theo tech và budget | VNPost là tổng công ty nhà nước: mua sắm qua đấu thầu, ngân sách theo năm tài chính, quyết định cần nhiều cấp phê duyệt | Phương án Build/Buy/Partner đã được chốt ở cấp nào chưa? Hay đang ở giai đoạn khảo sát? Ngân sách được phê duyệt (range) là bao nhiêu? |
| **GAP-M5: Governance theo tỉnh/vùng** | CDP điển hình phân quyền theo team/phòng ban | VNPost có 63 tỉnh thành với đơn vị tỉnh có thể hoạt động tương đối độc lập | Đơn vị tỉnh có được xem dữ liệu KH của tỉnh khác không? CDP phục vụ cấp trung ương (TCT), cấp vùng, hay cả hai? |

### MINOR Gaps (cần xác nhận, không block)

| Gap | Ghi chú | Câu hỏi cần đặt |
|---|---|---|
| **GAP-N1: Thuật ngữ "khách hàng" không nhất quán** | Cần phân biệt rõ: **người gửi** (khách hàng gửi hàng — cá nhân hoặc doanh nghiệp TMĐT/KHL) vs **người nhận** (đầu kia của bưu gửi) vs **bưu tá/shipper** (nhân viên VNPost phát hàng — internal actor, không phải khách hàng). Trong các buổi họp nội bộ VNPost, "khách hàng" có thể chỉ một trong các nhóm này | Khi VNPost nói "khách hàng" trong bối cảnh CDP, họ muốn nói đến đối tượng nào chủ yếu? Và "shipper" trong nội bộ VNPost được dùng để chỉ bưu tá hay chỉ doanh nghiệp gửi hàng? |
| **GAP-N2: "Lịch sử định vị" trong yêu cầu ban đầu** | Nếu đây là GPS từ app MyVNPost (do người gửi bật) → khác về consent so với GPS từ app DingDong của bưu tá | Dữ liệu định vị mà VNPost muốn thu thập trong CDP là định vị của ai: người dùng app MyVNPost, hay bưu tá (DingDong)? |
| **GAP-N3: Seasonal spike handling** | Dịp Tết, 11/11 tạo spike 5–10x — CDP điển hình không nhất thiết cần thiết kế cho spike lớn như vậy, nhưng với VNPost thì cần | VNPost có kỳ vọng gì về hiệu năng CDP trong mùa cao điểm (Tết, 11/11)? Hay sẽ xử lý trong giai đoạn sau? |

### Sub-domain cần research thêm (nếu có)

- **Quy trình đấu thầu IT của tổng công ty nhà nước Việt Nam** — nếu phương án Partner được chọn, timeline đấu thầu có thể mất 2–4 tháng trước khi bắt đầu build. Domain Brief đề cập nhưng chưa phân tích timeline đấu thầu cụ thể.
- **Zalo OA Integration** — nếu VNPost dùng Zalo OA làm kênh kích hoạt (rất phổ biến tại Việt Nam), cần research khả năng tích hợp CDP → Zalo OA. Domain Brief chưa cover kênh này.

---

## Danh sách câu hỏi clarify

*Phương pháp: Các câu hỏi sau đây tổng hợp từ 3 nguồn: (1) Q1–Q13 trong Domain Brief v2.1, (2) Gap analysis ở trên, (3) yêu cầu về Business Objective và Success Metrics. Đã loại bỏ trùng lặp, gộp câu liên quan, ưu tiên lại theo mức độ ảnh hưởng thực tế.*

---

### CRITICAL — Phải có câu trả lời trước khi thiết kế solution

**C1. Use case ưu tiên cho MVP là gì?**

> Trong số 6 use cases đã phân tích (Customer 360, Anti-Churn, Win-Back, Giảm hoàn, Cross-sell, Fraud Detection):
> - VNPost muốn thấy kết quả gì cụ thể trong **6–9 tháng đầu**?
> - Pain point nào đang gây ra thiệt hại kinh doanh lớn nhất ngay lúc này?
>
> *Lý do CRITICAL: Mỗi use case yêu cầu tập dữ liệu, tích hợp, và kiến trúc khác nhau hoàn toàn. Anti-Churn chỉ cần dữ liệu sản lượng theo thời gian từ MPITS/CAS. Customer 360 yêu cầu tất cả 5–6 tích hợp hoạt động đồng thời. Fraud Detection cần 6–12 tháng dữ liệu lịch sử trước khi có giá trị. Nếu không chốt use case MVP, sẽ không thể xác định scope tích hợp, timeline, hay kiến trúc.*

---

**C2. "Khách hàng" trong CDP này bao gồm những ai?**

> - CDP sẽ xây hồ sơ cho **người gửi** (cá nhân + doanh nghiệp TMĐT/KHL) chỉ, hay cả **người nhận** nữa?
> - Nếu bao gồm người nhận: họ chưa bao giờ tạo tài khoản VNPost — cơ chế **consent** cho nhóm này là gì? VNPost có quyền xây hồ sơ cho người nhận hay không (pháp lý)?
> - Với khách hàng doanh nghiệp (KHL): CDP sẽ track ở cấp **công ty** (mã KHL) hay cấp **cá nhân phụ trách** (người ký hợp đồng)?
>
> *Lý do CRITICAL: Đây là quyết định phạm vi quan trọng nhất, đặc thù của bưu chính mà CDP điển hình không gặp. Nếu bao gồm người nhận — số lượng profile tăng gấp 2–3 lần, yêu cầu consent phức tạp hơn nhiều, và có thể vi phạm NĐ 13 nếu không có cơ chế consent phù hợp. Nếu chỉ người gửi — scope gọn hơn nhiều nhưng bỏ lỡ dữ liệu về hành vi nhận hàng.*

---

**C3. Customer ID thống nhất và vai trò của PostID**

> - Hiện tại mỗi hệ thống (CAS, MyVNPost, PayPost, Portal KHL) đang dùng định danh riêng — **đã có mapping thủ công hoặc tự động nào giữa các ID này chưa?**
> - PostID đang phủ bao nhiêu % khách hàng VNPost? Khách hàng giao dịch qua quầy (không có app) có PostID không?
> - Nếu PostID chưa phủ đủ: **SĐT có thể dùng làm ID ghép nối chính không?** Có trường hợp nào cùng SĐT nhưng là 2 khách hàng khác nhau không?
>
> *Lý do CRITICAL: Nền tảng của toàn bộ Identity Resolution. Nếu không có ID thống nhất (hoặc PostID chưa phủ đủ), bài toán ghép nối định danh là công việc kỹ thuật lớn nhất và rủi ro nhất trong toàn dự án — ảnh hưởng trực tiếp đến kiến trúc và timeline ít nhất 3–6 tháng. Không thể phân tích Anti-Churn hay Customer 360 nếu không biết A ở CAS và B ở MyVNPost là cùng 1 người.*

---

**C4. MPITS có thể là gateway dữ liệu cho CDP không?**

> - MPITS đã kết nối API với sàn TMĐT. **API đó có thể dùng cho CDP không?**
> - Nếu CDP kết nối được với MPITS — dữ liệu từ các hệ thống con (CAS, PNS, PayPost) có được MPITS tổng hợp sẵn không, hay vẫn phải tích hợp từng hệ thống riêng lẻ?
> - IT VNPost có thể cung cấp danh sách endpoint MPITS và loại dữ liệu trả về không?
>
> *Lý do CRITICAL: Nếu MPITS đã là hub tổng hợp và có API — CDP có thể lấy dữ liệu từ 1 nguồn thay vì 8 nguồn. Câu trả lời ảnh hưởng đến: số lượng tích hợp cần làm (1 vs 8), timeline MVP, và tổng chi phí dự án. Đây là câu hỏi kỹ thuật quan trọng nhất quyết định kiến trúc.*

---

**C5. Compliance NĐ 13 — VNPost đã chuẩn bị đến đâu?**

> - VNPost đã có **chính sách bảo vệ dữ liệu cá nhân** theo NĐ 13/2023 chưa?
> - Đã thực hiện **DPIA (Đánh giá tác động xử lý dữ liệu)** cho dự án CDP này chưa?
> - Hiện tại VNPost đang **thu consent** từ khách hàng như thế nào khi họ giao dịch (tại quầy, qua app, qua sàn TMĐT)? Consent đó đã cover việc dùng dữ liệu cho phân tích CDP chưa?
> - Đơn vị nào trong VNPost chịu trách nhiệm pháp lý về bảo vệ dữ liệu cá nhân (DPO — Data Protection Officer)?
>
> *Lý do CRITICAL: CDP xử lý dữ liệu cá nhân quy mô lớn — bao gồm dữ liệu nhạy cảm (vị trí GPS, dữ liệu COD/tài chính). Không thể thiết kế CDP mà không biết VNPost đang ở đâu trong hành trình tuân thủ NĐ 13. Nếu VNPost chưa có consent mechanism → Consent Management phải là tính năng #1 của CDP, không phải tính năng phụ trợ.*

---

### MAJOR — Ảnh hưởng lớn đến scope, tính năng, và timeline

**M1. Phương án triển khai và ngân sách**

> - Quyết định Build/Buy/Partner đã được chốt ở cấp lãnh đạo nào chưa, hay đang ở giai đoạn khảo sát?
> - **Ngân sách tổng dự án** (CAPEX và OPEX hàng năm) đã được phê duyệt range là bao nhiêu?
> - Nếu phương án Partner được chọn — **quy trình đấu thầu** sẽ mất bao lâu? Có vendor nào VNPost đang xem xét cụ thể chưa?
>
> *Lý do MAJOR: Phương án triển khai và ngân sách ảnh hưởng toàn bộ kiến trúc, timeline, và cách phân kỳ dự án. Không có số ngân sách — không thể đề xuất giải pháp phù hợp.*

---

**M2. Năng lực đội IT/Data nội bộ VNPost**

> - VNPost hiện có bao nhiêu **Data Engineer / Data Analyst / Java Engineer** có thể tham gia dự án CDP?
> - Nếu chọn Option 3 (Partner + Unomi): **sau khi SI bàn giao, đội IT VNPost có năng lực tự vận hành không?** Hay sẽ cần duy trì hợp đồng support với SI?
> - Có kế hoạch tuyển thêm nhân lực cho CDP không?
>
> *Lý do MAJOR: Năng lực IT nội bộ quyết định khả thi của Option 1 (tự build) và mức độ phụ thuộc vào SI trong Option 3. Nếu VNPost không có Data Engineer — cần thiết kế UI self-service cho Marketing team, không thể yêu cầu họ viết query SQL để tạo segment.*

---

**M3. Kênh kích hoạt (Activation Channel)**

> - VNPost đang dùng kênh nào để liên hệ với khách hàng: **SMS, Zalo OA, Email, Push notification qua MyVNPost app**?
> - Có hệ thống **Marketing Automation** nào đang vận hành không (ví dụ: Haravan, HubSpot, hay hệ thống tự xây)?
> - Khách hàng doanh nghiệp (KHL) thường được liên hệ qua kênh nào — Account Manager gọi điện trực tiếp hay qua kênh tự động?
>
> *Lý do MAJOR: CDP tạo ra phân khúc nhưng giá trị thực đến từ "activation" — gửi đúng thông điệp đến đúng người qua đúng kênh. Nếu VNPost không có kênh tự động hoặc Marketing Automation tool → CDP chỉ là công cụ phân tích, không kích hoạt được chiến dịch Anti-Churn tự động. Cần biết để thiết kế integration phù hợp.*

---

**M4. Hệ thống legacy (BCCP, TMS, WMS) — có thể tích hợp không?**

> - BCCP, TMS, WMS có REST API không? Nếu không — **IT có cho phép read-only database access** để CDP lấy dữ liệu không?
> - Nếu chỉ có file export: **tần suất export là bao nhiêu** (realtime, hourly, daily, weekly)?
> - Dữ liệu từ 3 hệ thống này có cần thiết cho MVP không, hay có thể đưa vào phase sau?
>
> *Lý do MAJOR: Nếu BCCP/TMS/WMS chỉ có thể batch export daily → một số use case như Anti-Churn real-time sẽ bị ảnh hưởng. Cần biết để thiết kế kiến trúc phù hợp (real-time streaming vs batch processing) và ưu tiên tích hợp trong MVP.*

---

**M5. Phạm vi địa lý và phân quyền theo đơn vị tỉnh/vùng**

> - CDP giai đoạn đầu phục vụ **cấp TCT (toàn quốc)** hay sẽ pilot tại **một số tỉnh/vùng cụ thể** trước?
> - Quản lý đơn vị tỉnh có quyền xem dữ liệu khách hàng trong tỉnh của mình không? Có được xem dữ liệu tỉnh khác không?
> - Nếu pilot tỉnh — tỉnh/vùng nào được ưu tiên?
>
> *Lý do MAJOR: Ảnh hưởng đến kiến trúc phân quyền dữ liệu và quy mô hạ tầng trong MVP. CDP toàn quốc từ đầu vs CDP pilot một vùng là 2 bài toán khác nhau về độ phức tạp kỹ thuật và chi phí.*

---

### MINOR — Có thể assume tạm, hỏi để xác nhận

**N0. UI Spec — Banner ngang trong Template Email**

> - Field `Image (optional) banner ngang` trong form soạn Email template có ép ảnh về tỷ lệ cụ thể không (ví dụ: 16:9, 3:1, 600×200px)?
> - Nếu có crop/resize tự động — hệ thống xử lý phía server hay phía client trước khi upload?
> - Kích thước tối đa file ảnh được phép upload là bao nhiêu (MB)?
>
> *Có thể assume tạm: Banner ngang tỷ lệ 3:1 (600×200px), kích thước tối đa 2MB, hệ thống chỉ hiển thị preview theo tỷ lệ gốc không ép crop. Nếu sai → cần thêm validation và hướng dẫn kích thước trong UI.*

---

**N1. Timeline go-live và deadline cứng**

> - Ban lãnh đạo VNPost có **deadline cứng** nào cho dự án CDP không (cam kết Hội đồng thành viên, sự kiện ra mắt, kỳ họp ngành)?
> - MVP cần demo được sau bao nhiêu tháng kể từ khi bắt đầu dự án?
>
> *Có thể assume tạm: Không có deadline cứng trong 6 tháng tới. Nếu sai → cần điều chỉnh scope MVP và phương án triển khai.*

---

**N2. Dữ liệu định vị thuộc về ai**

> - "Lịch sử định vị" trong yêu cầu ban đầu — là **GPS từ app MyVNPost** (của người gửi) hay **GPS từ DingDong** (của bưu tá khi phát hàng)?
> - Ai là chủ thể dữ liệu: người dùng app hay bưu tá?
>
> *Có thể assume tạm: Dữ liệu định vị trong CDP là địa chỉ gửi/nhận (text), không phải GPS real-time. Nếu sai → cần thêm yêu cầu consent đặc biệt cho dữ liệu nhạy cảm.*

---

**N3. Dữ liệu người nhận từ sàn TMĐT — VNPost có quyền gì?**

> - Khi đơn hàng được tạo từ Shopee/Lazada — thông tin người nhận (tên, SĐT, địa chỉ) do sàn TMĐT gửi sang. **VNPost có quyền dùng thông tin này cho mục đích phân tích CDP không?**
> - Điều khoản hợp tác với các sàn TMĐT có giới hạn mục đích sử dụng dữ liệu không?
>
> *Có thể assume tạm: VNPost chỉ dùng dữ liệu người nhận từ sàn TMĐT cho mục đích vận hành giao vận, không dùng cho phân tích CDP. Nếu sai → mở ra nguồn dữ liệu người nhận phong phú nhưng cần xem lại điều khoản pháp lý.*

---

**N4. Tần suất cập nhật hồ sơ khách hàng**

> - Marketing team kỳ vọng dữ liệu trong CDP được cập nhật với tần suất nào: **real-time (dưới 5 phút), gần real-time (dưới 1 giờ), hay daily**?
> - Use case Anti-Churn có cần phát hiện dấu hiệu sụt giảm trong ngày, hay theo tuần là đủ?
>
> *Có thể assume tạm: Cập nhật daily là đủ cho giai đoạn MVP. Nếu sai → cần streaming architecture ngay từ đầu, tăng phức tạp và chi phí.*

---

## Tổng hợp Assumption đang áp dụng

| Mã | Assumption | Nếu sai — cần điều chỉnh |
|---|---|---|
| A1 | MVP use case là Anti-Churn KHL (doanh nghiệp TMĐT) | Toàn bộ tập tích hợp ưu tiên và timeline thay đổi |
| A2 | CDP chỉ xây hồ sơ cho người gửi (cá nhân + doanh nghiệp TMĐT/KHL), không bao gồm người nhận | Tăng gấp đôi số lượng profile, thêm yêu cầu consent phức tạp |
| A3 | SĐT là định danh ghép nối chính khi PostID chưa phủ đủ | Cần giải pháp identity resolution phức tạp hơn |
| A4 | BCCP/TMS/WMS không có REST API — dùng batch export daily | Kiến trúc và timeline tích hợp thay đổi đáng kể |
| A5 | Không có deadline cứng trong 6 tháng tới | Cần re-scope MVP nếu có deadline |
| A6 | Phương án triển khai là Partner + Unomi (Option 3) | Toàn bộ kiến trúc và cách phân kỳ thay đổi |
| A7 | Dữ liệu định vị trong CDP là địa chỉ text, không phải GPS real-time | Cần thêm consent mechanism cho dữ liệu nhạy cảm |
| A8 | CDP pilot ở cấp TCT toàn quốc, không giới hạn theo tỉnh trong MVP | Quy mô hạ tầng và phân quyền thay đổi nếu chọn pilot tỉnh |

---

## Agenda gợi ý cho buổi clarification với VNPost

**Mục tiêu buổi họp:** Trả lời đủ 5 câu CRITICAL để BA có thể thiết kế solution phase tiếp theo.

**Thời lượng đề xuất:** 60 phút (có thể kéo dài 90 phút nếu cần thảo luận phương án triển khai)

**Thành phần tham dự cần có:**
- PO hoặc đại diện nghiệp vụ (Marketing/CRM) — trả lời câu C1, C2
- IT Lead hoặc đại diện kỹ thuật VNPost — trả lời câu C3, C4, M3, M4
- Người có thẩm quyền về ngân sách/phương án — trả lời câu M1
- Đại diện pháp lý/compliance (nếu có) — trả lời câu C5

---

### Phần 1 — Warm-up (5 phút)

BA trình bày nhanh: mục tiêu buổi họp, những gì đã nghiên cứu (Domain Brief), và những gì cần xác nhận từ VNPost.

> *Thông điệp mở đầu:* "Chúng tôi đã nghiên cứu hệ sinh thái IT và nghiệp vụ của VNPost. Hôm nay chúng ta cần chốt 5 điểm để có thể bắt đầu thiết kế giải pháp phù hợp — nếu thiếu bất kỳ điểm nào trong số này, rủi ro thiết kế sai scope rất cao."

---

### Phần 2 — Use case và Business Goal (15 phút)

**Câu hỏi dẫn dắt:**

1. "VNPost đang gặp pain point nào lớn nhất với khách hàng TMĐT hiện tại?" *(mở, để VNPost tự nói trước)*
2. "Trong 6 use cases chúng tôi đã phân tích, use case nào VNPost muốn thấy kết quả trong 6–9 tháng đầu?"
3. "Khi nói 'khách hàng' trong context CDP này — VNPost muốn xây hồ sơ cho người gửi, người nhận, hay cả hai?"

*Mục tiêu phần này:* Chốt C1 và C2.

---

### Phần 3 — Dữ liệu và tích hợp (20 phút)

**Câu hỏi dẫn dắt:**

4. "PostID hiện tại phủ bao nhiêu % khách hàng toàn hệ thống? Khách hàng giao dịch qua quầy có PostID không?"
5. "MPITS đã có API gateway kết nối với sàn TMĐT. API đó có thể cho CDP kết nối vào không? Nếu có, CDP sẽ lấy được những dữ liệu gì từ MPITS?"
6. "BCCP, TMS, WMS có REST API không? Nếu không, IT có thể cho read-only database access hoặc file export định kỳ không?"

*Mục tiêu phần này:* Chốt C3 và C4, xác nhận M4.

---

### Phần 4 — Compliance và Dữ liệu nhạy cảm (10 phút)

**Câu hỏi dẫn dắt:**

7. "VNPost đã có chính sách bảo vệ dữ liệu cá nhân theo NĐ 13 chưa? Ai là người chịu trách nhiệm pháp lý về dữ liệu cá nhân trong tổ chức?"
8. "Hiện tại khi khách hàng giao dịch — VNPost thu consent như thế nào? Consent đó có cover việc dùng dữ liệu cho phân tích CDP không?"

*Mục tiêu phần này:* Chốt C5.

---

### Phần 5 — Phương án triển khai và nguồn lực (10 phút)

**Câu hỏi dẫn dắt:**

9. "VNPost đã có định hướng về phương án Build/Buy/Partner chưa? Hay đang ở giai đoạn khảo sát? Ngân sách range dự kiến là bao nhiêu?"
10. "VNPost hiện có Data Engineer hay Java Engineer có thể tham gia dự án CDP không?"

*Mục tiêu phần này:* Chốt M1 và M2.

---

### Phần 6 — Wrap-up và Next Steps (5–10 phút)

BA tổng kết:
- Những điểm đã chốt trong buổi họp
- Những điểm cần VNPost xác nhận thêm bằng văn bản hoặc buổi họp kỹ thuật riêng
- Bước tiếp theo: BA sẽ output Solution Design trong vòng [X] ngày sau khi có đủ 5 câu CRITICAL

---

### Câu hỏi dự phòng (nếu còn thời gian)

- "CDP này phục vụ toàn quốc từ đầu hay pilot tại một số tỉnh?" *(M5)*
- "VNPost đang dùng kênh gì để liên hệ với khách hàng — SMS, Zalo, email?" *(M3)*
- "Có deadline cứng nào cho dự án này không?" *(N1)*

---

## Kết luận

Buổi clarification này sẽ cho phép BA chốt **5 điểm CRITICAL** — sau đó có thể tiến hành `ba-solution-agent` để thiết kế solution, xác định phạm vi MVP và kiến trúc tích hợp phù hợp với thực tế VNPost.

Nếu **không có câu trả lời cho C1 (MVP Use Case) và C4 (MPITS API)** sau buổi họp — rủi ro thiết kế solution sai scope là rất cao và không nên tiến vào phase tiếp theo.
