# Identity Resolution Design — CDP VNPost/TCT

**Ngày tạo:** 2026-06-26
**Người thực hiện:** ba-solution-agent
**Phiên bản:** v1.0
**Mục đích:** Giải quyết 3 BLOCK issues trước khi tiếp tục wireframe và prototype Identity Resolution
**Phụ thuộc vào:** clarification.md, cdp-system-design-brief.md

---

## Bối cảnh và Quyết định đã chốt (Không thay đổi)

| Quyết định đã chốt | Nội dung |
|---|---|
| D-01 | Identity Resolution do BE xử lý tự động theo confidence score — càng ít thủ công càng tốt |
| D-02 | Màn hình "Đối soát hồ sơ" giữ trong sidebar nhưng read-only — đã xóa nút Merge/Dismiss |
| D-03 | Tab "Hồ sơ liên kết" thêm vào Customer 360 — hiển thị alias IDs, nguồn, ngày merge, confidence score |
| D-04 | Kafka ẩn tạm khỏi sidebar |

---

## Solution Overview

**Bài toán cần giải:** 3 lỗ hổng trong thiết kế Identity Resolution khiến Dev không biết code gì (BL-01), hệ thống không có khả năng giải trình khi bị kiểm tra (BL-02), và không có cơ chế phục hồi khi merge nhầm (BL-03).

**Giải pháp đề xuất:** Bổ sung ba lớp còn thiếu vào nền Identity Resolution hiện có — (1) State machine đầy đủ với định nghĩa hành vi rõ ràng cho từng vùng confidence score, (2) Audit log bất biến tự động ghi sau mỗi merge, (3) Flow "Báo cáo merge sai" cho phép Data Steward yêu cầu unmerge có kiểm soát. Tất cả ba lớp tương thích với nguyên tắc read-only UI và BE-driven đã chốt.

**Lý do chọn hướng này:** Thay vì trao quyền Merge/Dismiss cho người dùng UI (đơn giản nhưng vi phạm D-01 và D-02), giải pháp duy trì BE là người ra quyết định cuối cùng, UI chỉ là cửa sổ quan sát và kênh báo cáo. Điều này giữ nguyên kiến trúc đã chốt và phù hợp với tinh thần "ít thủ công nhất có thể".

---

## BL-01 — State Machine Confidence Score

### Vấn đề

Vùng 60–89% chưa có định nghĩa hành vi. Dev không biết code gì với nhóm này, wireframe không thể vẽ màn "Đối soát hồ sơ" vì không rõ dữ liệu sẽ hiển thị gì và có luồng tương tác nào không.

### State Machine đề xuất

```
Confidence Score
      │
      ├─ ≥ 90%  ──────────────────────────────────────────────────────────────────►  AUTO_MERGE
      │          BE tự động merge ngay                                               (Trạng thái: MERGED)
      │          Không cần review                                                    Ghi audit log tự động
      │
      ├─ 60–89%  ─────────────────────────────────────────────────────────────────►  PENDING_REVIEW
      │          BE đưa vào hàng chờ review                                          (Trạng thái: PENDING)
      │          Hiển thị tại màn "Đối soát hồ sơ"                                 Data Steward xem và quyết định
      │                                                                              (Xem chi tiết Flow 2 bên dưới)
      │
      └─ < 60%  ──────────────────────────────────────────────────────────────────►  AUTO_DISMISS
                 BE tự động bỏ qua                                                   (Trạng thái: DISMISSED)
                 Không hiển thị cho người dùng                                       Ghi log để audit nếu cần
```

### Định nghĩa trạng thái

| Trạng thái | Ai tạo | Hành vi UI | Ghi log |
|---|---|---|---|
| `MERGED` | BE tự động (≥90%) hoặc Data Steward xác nhận (60–89%) | Hiển thị trong tab "Hồ sơ liên kết" của Customer 360 | Bắt buộc — xem BL-02 |
| `PENDING` | BE tự động (60–89%) | Hiển thị trong màn "Đối soát hồ sơ" — read-only, Data Steward chọn quyết định | Ghi thời điểm phát hiện + điểm số |
| `DISMISSED` | BE tự động (<60%) hoặc Data Steward xác nhận là "Hai người khác nhau" | Không hiển thị trong UI thường. Admin có thể xem trong lịch sử | Ghi lý do dismiss |
| `UNMERGE_REQUESTED` | Data Steward báo cáo merge sai | Hiển thị trong hàng chờ xử lý của Admin | Ghi người báo cáo + lý do |
| `UNMERGED` | Admin/BE xử lý sau khi xác nhận request | Hiển thị trong audit trail | Ghi đầy đủ — xem BL-02 |

### Xử lý vùng PENDING (60–89%) — Hai phương án

#### Phương án A — Data Steward quyết định thủ công (Đề xuất)

Sau khi BE đưa vào `PENDING`:
- Màn "Đối soát hồ sơ" hiển thị cặp hồ sơ so sánh side-by-side (read-only)
- Data Steward xem thông tin và chọn một trong hai:
  - **"Xác nhận merge"** → BE thực hiện merge → trạng thái thành `MERGED`
  - **"Hai người khác nhau"** → BE ghi nhận → trạng thái thành `DISMISSED`
- Quyết định của Data Steward được ghi vào audit log cùng tên người, thời điểm, lý do tùy chọn

Ưu điểm: Giảm rủi ro merge nhầm ở vùng nhạy cảm (60–89% là vùng không chắc chắn). Phù hợp với COD risk requirement.

Nhược điểm: Tăng khối lượng công việc thủ công cho Data Steward nếu số lượng cặp `PENDING` lớn.

#### Phương án B — Auto-merge toàn bộ ≥60% + thông báo cho Data Steward review sau

BE merge tất cả ≥60% tự động, nhưng cặp 60–89% được đánh dấu `NEEDS_REVIEW` trong tab "Hồ sơ liên kết". Data Steward review sau và có thể báo cáo sai.

Ưu điểm: Ít thủ công hơn, phù hợp hơn với D-01.

Nhược điểm: Rủi ro cao hơn — merge nhầm xảy ra trước khi review. Với COD risk score, một merge nhầm đã ảnh hưởng nghiệp vụ ngay lập tức trước khi được phát hiện.

#### Đề xuất chọn Phương án A

Lý do: VNPost có dữ liệu chất lượng không đồng đều (tên có dấu/không dấu, viết tắt, sai chính tả), nên vùng 60–89% có xác suất merge nhầm không nhỏ. Với COD risk score là quyết định tài chính trực tiếp, chi phí của một merge nhầm (khiếu nại, tranh chấp, mất tin tưởng) cao hơn chi phí của Data Steward review thêm vài chục cặp mỗi ngày.

**[A1]** Giả định rằng số lượng cặp `PENDING` (60–89%) mỗi ngày ở mức Data Steward có thể xử lý được (dưới 50 cặp/ngày trong giai đoạn đầu). Nếu sai (số lượng lớn hơn nhiều), cần xem xét thêm tính năng batch-approve cho nhóm ≥85% hoặc điều chỉnh ngưỡng lên 85% thay vì 60%.

---

## BL-02 — Audit Trail cho Merge Tự Động

### Vấn đề

Merge tự động ≥90% ảnh hưởng trực tiếp đến COD risk score — quyết định tài chính. Không có log giải thích được khi khách hàng khiếu nại hoặc cơ quan quản lý kiểm tra.

### Requirement Audit Log

#### Những gì phải ghi (bắt buộc với mọi loại merge)

| Trường | Mô tả | Ví dụ |
|---|---|---|
| `event_id` | ID duy nhất của sự kiện | MERGE-20260626-001 |
| `event_type` | Loại sự kiện | AUTO_MERGE / MANUAL_MERGE / MANUAL_DISMISS / UNMERGE |
| `timestamp` | Thời điểm xảy ra (UTC) | 2026-06-26T09:42:11Z |
| `uid_master` | UID được giữ lại | UID-0000123456 |
| `uid_merged` | UID bị hợp nhất vào master (nếu merge) | UID-0000123999 |
| `confidence_score` | Điểm tương đồng tại thời điểm quyết định | 94 |
| `match_keys` | Khóa trùng khớp kích hoạt phát hiện | PHONE, NAME_FUZZY |
| `trigger_source` | Nguồn phát hiện cặp trùng | CAS, MyVNPost |
| `decision_by` | Ai ra quyết định | SYSTEM (auto) hoặc user_id của Data Steward |
| `decision_reason` | Lý do (với manual) | "Xác nhận cùng người — SĐT + CCCD khớp" |
| `scores_before` | Snapshot điểm số trước merge | `{churn: 18, cod_risk: 22, fraud: 9}` |
| `scores_after` | Snapshot điểm số sau merge (nếu thay đổi) | `{churn: 18, cod_risk: 20, fraud: 9}` |
| `fields_merged` | Các trường dữ liệu được hợp nhất | name, phone, address_primary |
| `fields_kept_from` | Với mỗi trường conflict — giữ từ UID nào | `{name: uid_master, address: uid_merged}` |

#### Quy tắc audit log

- Log phải là **bất biến** — không được sửa hoặc xóa sau khi ghi. Chỉ có thể thêm log mới (ví dụ: UNMERGE thêm vào chuỗi log, không xóa log MERGE cũ)
- Log phải ghi **đồng bộ** với hành động merge — không được ghi sau hoặc ghi batch
- **[A2]** Giả định rằng log được giữ tối thiểu 5 năm theo yêu cầu lưu trữ hồ sơ của VNPost. Cần PO/client xác nhận thời hạn lưu trữ chính xác (câu hỏi mở OQ-01)

#### Ai được xem audit log

| Vai trò | Quyền xem | Phạm vi |
|---|---|---|
| Admin | Xem toàn bộ | Tất cả sự kiện merge |
| Data Steward | Xem toàn bộ | Tất cả sự kiện merge |
| CSKH đầy đủ | Xem tóm tắt | Chỉ các merge liên quan đến KH đang xem |
| CSKH cơ bản | Không xem | — |
| Marketing | Không xem | — |

#### Hiển thị trong UI

**Vị trí 1 — Tab "Hồ sơ liên kết" trong Customer 360 (đã chốt tại D-03)**

Bổ sung thêm cột và thông tin vào tab này:

```
Tab: Hồ sơ liên kết

[Bảng danh sách alias và merge]
Cột:  | UID cũ       | Nguồn    | Ngày merge | Loại merge    | Confidence | Người quyết định | Chi tiết |
Dòng: | UID-0000123999 | MyVNPost | 26/06/2026 | Tự động (≥90%) | 94%        | Hệ thống         | [Xem]    |

Khi click "Xem" → slide-in panel hiển thị:
  - Thông tin cặp hồ sơ tại thời điểm merge (snapshot)
  - Khóa trùng khớp
  - Điểm số trước/sau merge
  - Các trường được hợp nhất từ UID nào

[Nút: Báo cáo merge sai] (chỉ hiện với Data Steward và Admin)
```

**Vị trí 2 — Màn "Đối soát hồ sơ" (sidebar)**

Thêm tab thứ hai trong màn này:

```
Màn: Đối soát hồ sơ
Tab 1: Chờ review       (cặp PENDING 60–89%)
Tab 2: Lịch sử merge    (log các merge đã xử lý — MERGED, DISMISSED, UNMERGED)
```

Tab "Lịch sử merge" là audit log dạng bảng, có thể lọc theo: loại sự kiện, khoảng thời gian, người quyết định, UID liên quan.

---

## BL-03 — Flow xử lý khi phát hiện Merge Sai

### Vấn đề

Nếu BE merge nhầm 2 khách hàng khác nhau → COD risk score sai → ảnh hưởng nghiệp vụ. Sửa tay DB không phải phương án chấp nhận được vì không để lại audit trail và có thể gây lỗi toàn vẹn dữ liệu.

### Thiết kế: Flow "Báo cáo và Unmerge"

Không cấp quyền "Undo Merge" trực tiếp vì quá nguy hiểm (có thể làm mất dữ liệu đã hợp nhất hợp lệ). Thay vào đó, dùng flow **Báo cáo → Xác nhận → Unmerge có kiểm soát**.

#### Phân quyền

| Hành động | CSKH cơ bản | CSKH đầy đủ | Data Steward | Admin |
|---|---|---|---|---|
| Thấy tab "Hồ sơ liên kết" | Không | Có (read-only) | Có (read-only + Báo cáo) | Có (full) |
| Nhấn "Báo cáo merge sai" | Không | Không | Có | Có |
| Xem hàng chờ Unmerge | Không | Không | Có (chỉ xem) | Có |
| Phê duyệt Unmerge | Không | Không | Không | Có |
| Từ chối request Unmerge | Không | Không | Không | Có |

#### Flow 3 bước

```
Bước 1 — Data Steward phát hiện và báo cáo
   │
   ├─ Từ tab "Hồ sơ liên kết" trong Customer 360
   │    → Xem dòng merge nghi ngờ sai
   │    → Nhấn [Báo cáo merge sai]
   │    → Form điền:
   │        - Mô tả lý do nghi ngờ sai (bắt buộc)
   │        - Bằng chứng hỗ trợ: ghi chú hoặc UID tham chiếu (tùy chọn)
   │    → Xác nhận gửi
   │    → Hệ thống tạo request `UNMERGE_REQUESTED`
   │    → Gửi thông báo cho Admin
   │
   ↓
Bước 2 — Admin review request
   │
   ├─ Admin nhận thông báo (chuông cảnh báo hoặc email)
   │    → Vào màn "Đối soát hồ sơ" → Tab "Yêu cầu unmerge"
   │    → Xem: UID master, UID đã merge, lý do báo cáo, audit log gốc, điểm số hiện tại
   │    → Hai lựa chọn:
   │
   │    [Phê duyệt Unmerge]                    [Từ chối]
   │         │                                       │
   │         ↓                                       ↓
   │    Điền lý do phê duyệt              Điền lý do từ chối
   │    Xác nhận                           Thông báo cho Data Steward
   │         │
   │         ↓
Bước 3 — BE xử lý Unmerge
   │
   ├─ BE nhận tín hiệu Unmerge đã được phê duyệt
   │    → Tách UID master thành 2 UID riêng biệt
   │    → Phân chia lại dữ liệu (giao dịch, địa chỉ, scores) về đúng UID gốc
   │    → Tính lại COD risk score, churn score, fraud score cho cả 2 UID mới
   │    → Ghi audit log UNMERGE (xem mẫu bên dưới)
   │    → Cập nhật tab "Hồ sơ liên kết" — hiển thị trạng thái UNMERGED
   │    → Gửi thông báo thành công cho Data Steward và Admin
   │
   └─ Kết thúc: 2 UID độc lập hoạt động trở lại
```

#### Audit log cho Unmerge

```
event_type:     UNMERGE
event_id:       UNMERGE-20260626-001
timestamp:      2026-06-26T14:30:00Z
uid_master:     UID-0000123456         (UID được phân tách lại)
uid_restored:   UID-0000123999         (UID được tái tạo/khôi phục)
original_merge: MERGE-20260626-001     (link sang audit log merge gốc)
requested_by:   DS-user-001            (Data Steward đã báo cáo)
approved_by:    ADMIN-user-001
reason:         "Phạm Minh Tuấn (B2C) bị merge nhầm với Pham M. Tuan (shipper) — SĐT shared"
scores_recalculated: true
```

#### Edge Cases

| Tình huống | Hành vi hệ thống |
|---|---|
| Data Steward báo cáo sai một merge đúng | Admin từ chối request, ghi lý do, không Unmerge |
| UID master đã bị merge thêm với UID thứ 3 trước khi Unmerge được xử lý | BE hiển thị cảnh báo cho Admin: "UID này đã có merge phức tạp — cần review manual" — không tự động Unmerge, cần quy trình đặc biệt |
| Data Steward gửi báo cáo trùng cho cùng một merge | Hệ thống chỉ tạo một request, request thứ 2 được báo "Đã có yêu cầu đang xử lý" |
| Admin phê duyệt Unmerge nhưng phân chia dữ liệu không rõ ràng (giao dịch dùng chung) | BE ưu tiên giao dịch về UID nào tạo ra giao dịch đó. Giao dịch không thể phân tách được ghi vào cả 2 UID với flag `shared_transaction` để Data Steward xử lý thủ công sau |

---

## User Flow

### Flow 1 — Data Steward xử lý hàng chờ PENDING (60–89%)

Thực hiện: Hàng ngày — review cặp hồ sơ chờ xác nhận

1. Data Steward đăng nhập CDP, vào sidebar → **Đối soát hồ sơ**
2. Mặc định hiển thị **Tab "Chờ review"** — danh sách cặp `PENDING` sắp theo confidence score giảm dần
3. Click vào một cặp → mở màn **so sánh side-by-side** (read-only)
   - Trường trùng khớp hiển thị màu xanh
   - Trường khác nhau hiển thị màu vàng/đỏ
   - Hiển thị confidence score + match keys + nguồn phát hiện
   - Nếu có cảnh báo (ví dụ: SENDER vs RECEIVER, SĐT shared) → banner cảnh báo đỏ nổi bật
4. Data Steward quyết định:
   - **Nếu chắc chắn là cùng người:** nhấn [Xác nhận là cùng người] → điền lý do tùy chọn → xác nhận → BE thực hiện merge → trạng thái `MERGED` → quay về danh sách
   - **Nếu chắc chắn là khác người:** nhấn [Hai người khác nhau] → điền lý do tùy chọn → xác nhận → trạng thái `DISMISSED` → quay về danh sách
   - **Nếu chưa chắc:** nhấn [Để sau] → cặp vẫn ở `PENDING`, xuất hiện lại trong danh sách ngày hôm sau
5. Sau khi xử lý xong danh sách → Tab "Lịch sử merge" để xem lại các quyết định đã thực hiện

**Edge Cases:**
- Danh sách PENDING rỗng → hiển thị trạng thái "Không có hồ sơ chờ review" — không cần làm gì
- Cặp có cảnh báo (SENDER vs RECEIVER) → Data Steward phải cuộn xuống xem cảnh báo trước khi nút quyết định được kích hoạt (forced review)
- Data Steward không có quyền → màn "Đối soát hồ sơ" không hiển thị trong sidebar

---

### Flow 2 — Data Steward báo cáo merge sai

Thực hiện: Khi phát hiện merge nhầm trong quá trình xử lý CSKH hoặc kiểm tra định kỳ

1. Data Steward vào **Customer 360** của khách hàng nghi bị merge nhầm
2. Click tab **Hồ sơ liên kết** → xem danh sách merge đã thực hiện
3. Xác định dòng merge nghi ngờ → nhấn [Báo cáo merge sai]
4. Form hiện ra:
   - Trường bắt buộc: mô tả lý do nghi ngờ merge sai (ít nhất 20 ký tự)
   - Trường tùy chọn: ghi chú bổ sung
5. Nhấn [Gửi báo cáo] → xác nhận lần 2 "Bạn có chắc muốn báo cáo merge này là sai?" → xác nhận
6. Hệ thống tạo request → thông báo "Báo cáo đã được gửi đến Admin. Bạn sẽ nhận thông báo khi có cập nhật."
7. Data Steward có thể xem trạng thái request trong Tab "Lịch sử merge" → dòng merge được đánh dấu "Đang chờ xử lý"

**Edge Cases:**
- Data Steward gửi báo cáo cho merge đã có request đang xử lý → thông báo "Đã có yêu cầu xem xét cho merge này, đang chờ Admin xử lý"
- Data Steward không thể hủy report sau khi đã gửi — chỉ Admin có thể từ chối

---

### Flow 3 — Admin xử lý yêu cầu Unmerge

Thực hiện: Khi nhận được báo cáo từ Data Steward

1. Admin nhận thông báo (badge trên chuông cảnh báo header)
2. Vào **Đối soát hồ sơ** → Tab **Yêu cầu Unmerge**
3. Xem danh sách request đang chờ — cột: UID liên quan, người báo cáo, thời gian báo cáo, lý do, trạng thái
4. Click vào request cụ thể → xem chi tiết:
   - Thông tin 2 hồ sơ (snapshot tại thời điểm merge)
   - Audit log merge gốc (ai merge, confidence score bao nhiêu, khi nào)
   - Lý do Data Steward báo cáo
   - Tác động hiện tại: điểm số COD risk, churn đang bị ảnh hưởng
5. Admin quyết định:
   - **Nếu xác nhận merge sai:** nhấn [Phê duyệt Unmerge] → điền lý do → xác nhận → BE thực hiện Unmerge → thông báo cho Data Steward
   - **Nếu merge đúng:** nhấn [Từ chối] → điền lý do rõ ràng → xác nhận → thông báo từ chối cho Data Steward kèm lý do

**Edge Cases:**
- Admin phê duyệt nhưng merge phức tạp (UID đã merge thêm với UID thứ 3) → hệ thống hiển thị cảnh báo, yêu cầu Admin xác nhận thêm lần nữa trước khi BE xử lý
- Admin không xử lý request quá 5 ngày làm việc → hệ thống gửi nhắc nhở tự động (email hoặc thông báo trong app)

---

## Dependencies

- **BE phải implement state machine** trước khi wireframe và prototype có thể hiển thị đúng trạng thái — đặc biệt là trạng thái `PENDING`, `MERGED`, `DISMISSED`
- **Schema audit log** cần BE định nghĩa và confirm trước khi UI tab "Lịch sử merge" được thiết kế chi tiết
- **Bảng phân quyền** (ai thấy nút Báo cáo, ai thấy tab Yêu cầu Unmerge) cần được confirm với PO/client trước khi wireframe
- **Notification system** (chuông cảnh báo cho Admin) cần xác định cơ chế hiện có của CDP trước khi thiết kế — nếu chưa có thì cần thêm vào scope

---

## Trade-offs và Alternatives

| Hướng | Ưu điểm | Nhược điểm |
|---|---|---|
| **Đề xuất: PENDING cho 60–89%, Data Steward quyết định** | Giảm rủi ro merge nhầm; phù hợp với COD risk nhạy cảm; audit trail rõ ràng | Thêm công việc thủ công cho Data Steward; cần đủ năng lực Data Steward |
| Alternative 1: Auto-merge tất cả ≥60% | Ít thủ công hơn, phù hợp D-01 | Rủi ro merge nhầm cao hơn; merge sai trước khi review xảy ra |
| Alternative 2: Chỉ auto-merge ≥85%, bỏ qua 60–84% | Trung gian; giảm khối lượng review | 60–84% bị loại bỏ có thể miss nhiều cặp trùng thực sự |
| Alternative 3: Không cho phép Unmerge, chỉ tách thủ công qua DB | Đơn giản về BE | Không có audit trail; không có UI hỗ trợ; rủi ro lỗi toàn vẹn dữ liệu |

---

## Assumptions

| Mã | Assumption | Nếu sai — cần điều chỉnh |
|---|---|---|
| A1 | Số lượng cặp PENDING mỗi ngày dưới 50 cặp trong giai đoạn đầu (Data Steward có thể xử lý được) | Cần thêm tính năng batch-approve hoặc tăng ngưỡng lên 85% |
| A2 | Audit log giữ tối thiểu 5 năm theo yêu cầu lưu trữ của VNPost | Điều chỉnh storage requirement và chính sách lưu trữ |
| A3 | Notification cho Admin khi có request Unmerge dùng cơ chế badge trong app (không cần email riêng) | Cần thêm email notification nếu Admin không đăng nhập hàng ngày |
| A4 | Chỉ có một cấp phê duyệt Unmerge (Admin) — không cần cấp phê duyệt thứ hai từ lãnh đạo | Cần thêm cấp phê duyệt nếu Unmerge ảnh hưởng KHL lớn (yêu cầu pháp lý hoặc quy trình nội bộ VNPost) |
| A5 | Data Steward là vai trò riêng biệt, không phải CSKH cấp cao kiêm nhiệm | Cần đơn giản hóa luồng nếu không có Data Steward chuyên biệt |

---

## Open Questions — Cần PO/Client xác nhận

- [ ] **OQ-01**: Thời hạn lưu giữ audit log merge là bao nhiêu năm theo quy định nội bộ VNPost và NĐ 13? (Đang assume 5 năm)
- [ ] **OQ-02**: Ngưỡng số cụ thể cho vùng PENDING — VNPost muốn dùng 60% hay có ngưỡng khác? (Ví dụ: một số hệ thống dùng 70% là ngưỡng review)
- [ ] **OQ-03**: Notification cho Admin khi có request Unmerge mới — chỉ badge trong app hay cần email/SMS ngoài giờ?
- [ ] **OQ-04**: Nếu một UID đã bị merge nhiều lần (chuỗi merge), khi Unmerge cần tách đến đâu — chỉ tách merge gần nhất hay có thể tách toàn bộ chuỗi?
- [ ] **OQ-05**: Khi nhiều hệ thống nguồn cung cấp giá trị khác nhau cho cùng một trường (ví dụ: CRM nói ACTIVE, CAS nói INACTIVE; CRM phân loại "Cá nhân", Portal KHL phân loại "Doanh nghiệp"), CDP dùng rule ưu tiên nguồn nào để quyết định giá trị hiển thị trên Golden Record? Rule này do VNPost tự định nghĩa hay theo cấu hình Unomi? Cần xác nhận cho các trường: loại KH, group, status, loyalty tier.

---

## Risk Register

| Mã | Rủi ro | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| R1 | Số lượng cặp PENDING lớn → Data Steward không xử lý kịp → tồn đọng → hàng chờ phình to | H | M | Theo dõi KPI "thời gian trung bình xử lý PENDING" từ ngày đầu; thiết lập SLA cho Data Steward (ví dụ: xử lý trong 2 ngày làm việc) |
| R2 | Data Steward bấm nhầm "Xác nhận merge" cho cặp thực ra khác nhau | H | L | Bắt buộc hiển thị cảnh báo cho cặp có dấu hiệu rủi ro; cho phép báo cáo merge sai ngay sau khi merge để thu hồi |
| R3 | Admin không xử lý request Unmerge kịp thời → COD risk score sai kéo dài | H | M | Reminder tự động sau 5 ngày; escalation nếu không xử lý sau 10 ngày |
| R4 | Audit log bị tấn công hoặc thay đổi → không giải trình được trước cơ quan quản lý | H | L | Audit log cần lưu trên storage riêng biệt với quyền append-only; kiểm tra integrity định kỳ (phạm vi SA/Dev) |
| R5 | Unmerge không tách được dữ liệu chính xác → 2 UID sau unmerge vẫn sai | M | M | BE cần định nghĩa rõ logic phân chia dữ liệu trước khi implement; test case phải bao gồm các scenario phức tạp |

**Ưu tiên xử lý ngay:** R1 (cần định nghĩa SLA cho Data Steward ngay trong requirement), R3 (cần confirm cơ chế notification trước khi wireframe)

---

## Tóm tắt thay đổi UI cần thực hiện

Dựa trên 3 BLOCK issues đã giải quyết, các thay đổi UI cần thực hiện:

### Màn "Đối soát hồ sơ" (Identity Resolution — sidebar)

```
TRƯỚC:  Một tab duy nhất — danh sách cặp nghi ngờ trùng
        Nút Merge và Dismiss đã bị xóa (D-02 đã chốt)

SAU:    Tab 1: "Chờ review"       — cặp PENDING 60–89%
                                    Data Steward chọn: [Xác nhận merge] / [Hai người khác nhau] / [Để sau]
        Tab 2: "Lịch sử merge"   — audit log dạng bảng (MERGED, DISMISSED, UNMERGED)
                                    Lọc theo: loại sự kiện, thời gian, người quyết định
        Tab 3: "Yêu cầu Unmerge" — chỉ Admin thấy
                                    Danh sách request đang chờ xử lý
```

### Tab "Hồ sơ liên kết" trong Customer 360 (D-03 đã chốt, bổ sung thêm)

```
TRƯỚC:  Danh sách alias IDs, nguồn, ngày merge, confidence score

SAU:    Giữ nguyên + bổ sung:
        - Cột "Loại merge": Tự động (≥90%) / Xác nhận thủ công / Unmerged
        - Cột "Người quyết định": Hệ thống / [Tên Data Steward]
        - Nút [Xem chi tiết] → slide-in panel: snapshot hồ sơ tại thời điểm merge + các trường hợp nhất
        - Nút [Báo cáo merge sai] — chỉ hiện với Data Steward và Admin
        - Dòng UNMERGED được đánh dấu riêng biệt (màu xám + badge "Đã tách")
```

---

*Tài liệu này đủ để BA trình bày với PO/client xác nhận OQ-01 đến OQ-04, và đủ để wireframe agent thiết kế màn "Đối soát hồ sơ" và tab "Hồ sơ liên kết" theo đúng state machine và flow đã định nghĩa.*
