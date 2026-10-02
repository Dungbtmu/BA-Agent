# Solution Design — Mô hình Độ ưu tiên Campaign mới (Priority Redesign)

**Dự án:** CVM (Customer Value Management)
**Nguồn:** `.claude/output/cvm/solution/clarification.md` (đã chốt toàn bộ với Jun)
**Baseline tham chiếu:** `urd-srs-v4.md` (V4.17) — II.6.8, UC-CAM-01/02/03/05/07, UC-PRIORITY-01, Screen 2, Screen 3, Screen Settings Tab 3
**Trạng thái:** Đã qua phản biện `ba-devil-advocate-agent` — kết quả PASS WITH CONDITIONS (0 BLOCK, 3 MAJOR, 5 MINOR), toàn bộ 8 điểm đã được áp dụng vào bản này. Còn OQ-4 tạm hoãn (chờ Jun làm việc với CNTT) — cần chốt trước khi chạy SYNC patch thật. Nếu OQ-4 không ảnh hưởng gì thêm, sẵn sàng chuyển sang SYNC.

---

## 1. Tổng quan thay đổi

### 1.1. So sánh mô hình cũ vs mô hình mới

| Khía cạnh | Mô hình CŨ (V4.17) | Mô hình MỚI (Solution này) |
|---|---|---|
| Đơn vị ưu tiên | 1 số nguyên toàn cục (1–9999) / campaign | N vị trí độc lập / campaign — mỗi vị trí gắn đúng 1 nhóm trigger |
| Nơi cấu hình | 3 nơi: Campaign List (inline), Campaign Builder (Section 1 STT 5), Priority Matrix | Duy nhất 1 nơi: màn "Độ ưu tiên" tại Cài đặt |
| Cách nhóm | Không nhóm — so trùng toàn cục theo số | Nhóm tự động theo trigger trùng (≥ 1 trigger chung) |
| Phạm vi tham gia | Mọi campaign Active (không phân loại) | Chỉ campaign Active + loại "Có thời hạn" mới vào bàn kéo-thả |
| Campaign "vận hành thường trực" | Không có khái niệm này — vẫn nhập priority như bình thường | Không tham gia kéo-thả; tiebreak bằng `created_at` khi cạnh tranh với nhau; luôn nhường campaign "Có thời hạn" cùng trigger |
| Validate trùng | Chặn cứng tại 4 nơi (List/Builder/Matrix/Duyệt), phát sinh 4 lần vá lỗ hổng (V4.11, V4.14, V4.15) | Không còn khái niệm "trùng" — vị trí trong nhóm là thứ tự tương đối (index), không phải số tự nhập nên không thể trùng |
| Sửa priority có cần duyệt lại? | Có — sửa tại Campaign List bắt buộc chuyển về Pending | Không — Admin sắp xếp tại Cài đặt áp dụng ngay, không đụng vòng đời campaign |
| Campaign mới Active | priority = max + 1 (tự xếp cuối) | Tự động thêm vào cuối mỗi nhóm trigger tương ứng |
| Trigger đổi khiến nhóm thay đổi | Không có cơ chế — vì priority không gắn trigger | Tự động thêm/gỡ khỏi nhóm, cuối nhóm khi mới thêm, giữ thứ tự tương đối khi gỡ |
| Field ở Campaign Builder | Có field "Độ ưu tiên" — ô nhập số tay (Section 1 STT 5) | Giữ lại vị trí field, nhưng đổi thành **hiển thị theo trạng thái** (không nhập tay) — xem Mục 3.1 |
| Phân loại chiến dịch | Không có | Mới: "Vận hành thường trực" / "Có thời hạn" — chọn tường minh lúc Tạo |

### 1.2. Vì sao chọn hướng này thay vì vá tiếp mô hình cũ

Mô hình cũ đã trải qua 4 lần vá lỗ hổng chặn trùng (V4.11, V4.14, V4.15) mà vẫn phát sinh case mới (Paused × Active) — dấu hiệu of lỗi thiết kế gốc: dùng 1 số tự do làm proxy cho một khái niệm vốn có tính cục bộ theo ngữ cảnh (trigger). Nhóm theo trigger loại bỏ tận gốc nhu cầu "so trùng" vì thứ tự trở thành index nội bộ của danh sách, không phải giá trị do người dùng nhập tay — không có validate range/blocking issue nào cần thêm nữa.

**Trade-off phải đánh đổi:** mất khả năng QTV tự đặt priority ngay lúc tạo campaign (phải chờ Admin sắp xếp sau khi Active); tăng 1 bước thao tác cho Admin (phải chủ động vào Cài đặt nếu muốn khác vị trí cuối). Đây là đánh đổi hợp lý vì bản chất nghiệp vụ: độ ưu tiên chỉ có ý nghĩa khi đặt cạnh các campaign cùng trigger, và việc đó chỉ Admin mới có đủ góc nhìn toàn cục để quyết — matching đúng với quyết định "chỉ Admin được sắp xếp" đã chốt.

### 1.3. Phân biệt thuật ngữ — tránh nhầm lẫn với "Priority trigger" đã có sẵn

**Cảnh báo quan trọng khi patch URD**: hệ thống đang có sẵn 1 cơ chế khác cũng dùng từ "ưu tiên" + "trigger" + "kéo-thả" — đó là **Priority trigger** (UC-CAM-02 bước 2b, Advanced mode): sắp xếp thứ tự xử lý **giữa nhiều trigger trong CÙNG 1 campaign** khi logic OR/AND khớp đồng thời nhiều điều kiện. Cơ chế này **giữ nguyên, không đổi** trong CR này.

Solution ở đây tạo ra 1 cơ chế **hoàn toàn khác** nhưng dễ bị gọi nhầm cùng tên: sắp xếp thứ tự xử lý **giữa nhiều campaign khác nhau cùng dùng 1 trigger**. Để tránh 2 khái niệm lẫn vào nhau khi patch URD (nhiều UC cùng dùng từ "ưu tiên" + "trigger") và khi đào tạo người dùng thật, dùng tên gọi phân biệt tường minh xuyên suốt:

| Tên gọi chính thức | Phạm vi | Nơi cấu hình | Thay đổi trong CR này |
|---|---|---|---|
| **Thứ tự trigger nội bộ** (Priority trigger — giữ tên cũ trong URD hiện tại) | Nhiều trigger trong 1 campaign | Campaign Builder, UC-CAM-02 bước 2b | Không đổi |
| **Nhóm ưu tiên liên-campaign** (tên mới cho cơ chế ở solution này) | Nhiều campaign cùng dùng 1 trigger | Cài đặt → Tab "Độ ưu tiên" | Toàn bộ Mục 2 của solution này |

Khi patch URD (II.6.8, UC-PRIORITY-01, Screen Settings Tab 3), luôn dùng cụm "Nhóm ưu tiên liên-campaign" hoặc "Độ ưu tiên liên-campaign" khi nói về cơ chế mới — không dùng trơn từ "độ ưu tiên trigger" vì dễ lẫn với Thứ tự trigger nội bộ đã có.

### 1.4. Phạm vi ảnh hưởng — UC/Screen sẽ bị patch (chi tiết ở Mục 6)

- **URD Section**: II.6.8 (Cross-campaign Priority) — viết lại cơ chế
- **UC-CAM-01** (Danh sách) — bỏ inline edit priority (không cho nhập tay nữa), đổi cột Ưu tiên thành hiển thị trạng thái/vị trí
- **UC-CAM-02** (Tạo) — đổi field Độ ưu tiên từ nhập tay sang hiển thị theo trạng thái, thêm chọn loại hình chiến dịch
- **UC-CAM-03** (Sửa Draft) — kế thừa thay đổi từ UC-CAM-02
- **UC-CAM-05** (Duyệt) — bỏ bước check trùng priority
- **UC-CAM-07** (Bật lại) — bổ sung rõ: chỉ nhánh dẫn tới Active thật (1a, hoặc 1c với `pre_pause_status`=Active) mới vào nhóm ngay; nhánh dẫn tới Pending (1b, hoặc 1c với `pre_pause_status`=Pending) phải chờ Duyệt lại qua UC-CAM-05 (xem Mục 2.3, 6)
- **UC-PRIORITY-01** — thiết kế lại hoàn toàn thành "UC Cấu hình Độ ưu tiên theo nhóm Trigger"
- **Screen 2** (Campaign List) — đổi cột Ưu tiên từ inline edit sang hiển thị, bỏ khả năng nhập tay
- **Screen 3** (Campaign Builder) — Section 1: đổi field Độ ưu tiên thành hiển thị theo trạng thái (không nhập tay), thêm Loại hình chiến dịch; Kênh & Lịch gửi: field Ngày kết thúc ẩn/khóa theo loại hình
- **Screen Settings Tab 3** — thiết kế lại hoàn toàn

---

## 2. Flow "Độ ưu tiên" mới tại Cài đặt (thay thế UC-PRIORITY-01)

### 2.1. Cấu trúc màn hình

```
Cài đặt → Tab "Độ ưu tiên" (đổi tên từ "Priority Matrix")
┌─────────────────────────────────────────────────────────┐
│ Filter: [ Chọn Trigger ▾ ]  (mặc định: hiện tất cả nhóm) │
├─────────────────────────────────────────────────────────┤
│ ▾ Nhóm Trigger: "SIM sắp hết hạn" (E01)                  │
│   ┌───────────────────────────────────────────────────┐ │
│   │ [≡] 1. Campaign "Chào mừng đại lễ"      Có thời hạn│ │
│   │ [≡] 2. Campaign "Ưu đãi gia hạn sớm"    Có thời hạn│ │
│   │ [≡] 3. Campaign "Nhắc gia hạn T-3"      Có thời hạn│ │
│   ├───────────────────────────────────────────────────┤ │
│   │ ℹ Ngoài ra còn có Vận hành thường trực dùng chung  │ │
│   │   trigger này: "Gia hạn tự động" (không xếp hạng)  │ │
│   └───────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│ ▾ Nhóm Trigger: "Data sắp hết" (E02)                     │
│   (chỉ có Vận hành thường trực — không có bàn kéo-thả,   │
│    hiển thị dạng ghi chú: "2 campaign vận hành thường    │
│    trực dùng chung trigger này, tự động theo thời gian   │
│    tạo — không cần sắp xếp")                             │
└─────────────────────────────────────────────────────────┘
```

- Mỗi nhóm = 1 khối (accordion hoặc card), tiêu đề là tên + mã trigger.
- Trong khối: nếu có ≥ 2 campaign "Có thời hạn" trùng trigger → hiện bảng kéo-thả (giống UI Priority Matrix cũ, bỏ ô nhập số tay — chỉ giữ kéo-thả để tránh tái lập vấn đề "số trùng").
- Nếu nhóm chỉ có 1 campaign "Có thời hạn" (không cạnh tranh với ai) → xem Mục 5.3 (vẫn hiển thị, dạng đơn giản, không có kéo-thả).
- Nếu nhóm chỉ toàn "Vận hành thường trực" → không có bàn kéo-thả, hiển thị dạng liệt kê thông tin kèm ghi chú tiebreak `created_at` (xem Mục 2.5).
- Campaign "Vận hành thường trực" trùng trigger với nhóm có "Có thời hạn" → hiển thị dưới dạng khối ghi chú phụ trong cùng nhóm (không phải dòng kéo-thả), đúng quyết định 3.3.
- Filter theo Trigger: dropdown chọn 1 trigger cụ thể → chỉ hiện đúng nhóm đó; mặc định hiện toàn bộ nhóm đang tồn tại.
- Filter theo Campaign (bổ sung — phục vụ link điều hướng từ Mục 2.7): chọn/truyền 1 campaign cụ thể → hiển thị đồng thời **tất cả nhóm trigger mà campaign đó tham gia** (có thể nhiều nhóm cùng lúc), mỗi nhóm vẫn hiện đầy đủ các campaign khác trong đó để Admin thấy toàn cảnh cạnh tranh. Đây là chế độ filter khác với filter theo Trigger (1 trigger → 1 nhóm); dùng khi Admin cần tra cứu nhanh "campaign X đang đứng đâu trong tất cả các nhóm nó thuộc về" — đúng kịch bản khi QTV/Admin bấm link từ Section 1 Builder hoặc Campaign List (Mục 2.7).

**[A1]** Giả định đơn vị nhóm hiển thị là "1 nhóm = 1 trigger cụ thể" (không gộp nhiều trigger vào 1 khối), vì mức khớp đã chốt là "trùng ít nhất 1 trigger" — một campaign dùng nhiều trigger sẽ xuất hiện lặp lại ở nhiều nhóm khác nhau (đúng bản chất "N vị trí độc lập theo từng nhóm" ở clarification mục 4). Nếu Jun muốn gộp hiển thị theo tổ hợp trigger thay vì từng trigger đơn lẻ, cần điều chỉnh lại UI này.

### 2.2. Logic nhóm tự động

- Hệ thống quét toàn bộ trigger đang được dùng bởi ít nhất 1 campaign Active (bất kể loại hình) → mỗi trigger tạo thành 1 nhóm tiềm năng.
- 1 campaign dùng N trigger (Advanced mode) → xuất hiện trong N nhóm khác nhau, độc lập vị trí ở từng nhóm.
- Trigger chỉ có 1 campaign Active duy nhất tham chiếu (loại "Có thời hạn") → nhóm đó không cần hiển thị kéo-thả (xem Mục 5.3) nhưng vẫn tồn tại về mặt dữ liệu (để sẵn sàng khi có campaign thứ 2 trùng trigger).
- Trigger hoàn toàn không có campaign Active nào dùng → không tạo nhóm (ẩn khỏi màn Cài đặt).

### 2.3. Cơ chế tự động thêm/gỡ khỏi nhóm

| Sự kiện | Hành vi |
|---|---|
| Campaign "Có thời hạn" chuyển Active (qua Duyệt hoặc Bật lại) | Tự động thêm vào cuối mỗi nhóm trigger tương ứng |
| Campaign "Có thời hạn" đang Active đổi trigger (thêm trigger mới) | Tự động thêm vào cuối nhóm trigger mới, **giữ nguyên** vị trí hiện có ở các nhóm trigger cũ không đổi |
| Campaign "Có thời hạn" đang Active bớt trigger (không còn trùng nhóm cũ) | Tự động gỡ khỏi đúng nhóm đó; các thành viên còn lại trong nhóm **giữ nguyên thứ tự tương đối**, không dồn số (ví dụ: vị trí 1,2,3,4 gỡ vị trí 2 → còn lại 1,3,4 giữ nguyên độ lệch, hiển thị lại thành 1,2,3 chỉ là số thứ tự hiển thị — thứ tự xử lý thực tế không đổi) |
| Campaign chuyển Paused (Kill Switch hoặc Bật lại có sửa) | Gỡ khỏi mọi nhóm đang tham gia — không giữ "vị trí ảo" trong lúc Paused (đúng quyết định mục 3.6 clarification) |
| Campaign Paused bấm [Bật] — theo đúng 3 nhánh của UC-CAM-07 | **Nhánh 1a (Bật thẳng Active)** hoặc **nhánh 1c với `pre_pause_status` = Active**: coi như "mới thêm" — tự động xếp **cuối** nhóm ngay, không khôi phục vị trí cũ trước khi Paused. **Nhánh 1b (param/filter bị sửa lúc Paused → về Pending)** hoặc **nhánh 1c với `pre_pause_status` = Pending**: campaign **chưa vào nhóm** — chỉ Active thật mới vào nhóm (Mục 2.2), nên phải chờ Admin duyệt lại qua UC-CAM-05 thành Active rồi mới được thêm vào cuối nhóm, đúng cơ chế Mục 4.2 |
| Campaign chuyển Ended (hết hạn hoặc bị xóa) | Gỡ khỏi mọi nhóm; xem xử lý chi tiết Mục 5.2 |
| Campaign đổi loại hình "Có thời hạn" → "Vận hành thường trực" | Gỡ khỏi bàn kéo-thả của mọi nhóm đang tham gia; xem Mục 5.1 |
| Campaign đổi loại hình "Vận hành thường trực" → "Có thời hạn" | Thêm vào cuối bàn kéo-thả của mọi nhóm trigger đang dùng; xem Mục 5.1 |

**Rule "luôn xếp cuối"**: áp dụng cho MỌI trường hợp một campaign bắt đầu tham gia bàn kéo-thả của 1 nhóm — dù là do mới Active, do đổi trigger, hay do đổi loại hình — không có ngoại lệ chèn giữa.

**[A2]** Đề xuất bổ sung UX (theo gợi ý mở trong clarification mục 3.3): thêm badge nhỏ "Mới" cạnh campaign vừa được tự động thêm vào nhóm trong 24 giờ gần nhất (dựa trên timestamp `added_to_group_at`, không phải `created_at` của campaign) — giúp Admin nhận biết cần rà soát mà không chặn xử lý. Đây là đề xuất không bắt buộc theo đúng tinh thần clarification — nếu Jun không cần, có thể bỏ badge này mà không ảnh hưởng logic cốt lõi.

### 2.4. Mô hình dữ liệu nghiệp vụ (mức khái niệm — không đi vào schema)

Mỗi campaign "Có thời hạn" có thể sở hữu **N vị trí ưu tiên độc lập**, mỗi vị trí gắn với đúng 1 nhóm trigger mà campaign đó tham gia:

```
Campaign "Chào mừng đại lễ" (Có thời hạn, dùng trigger E01 + E05):
  - Vị trí trong nhóm E01: #2
  - Vị trí trong nhóm E05: #1

Campaign "Ưu đãi cuối tuần" (Có thời hạn, chỉ dùng trigger E01):
  - Vị trí trong nhóm E01: #1
```

Về nghiệp vụ, đây tương đương một quan hệ "Campaign × Nhóm Trigger → Vị trí", không phải một thuộc tính đơn của campaign. Khi xử lý event thực tế theo trigger E01, hệ thống chỉ cần tra đúng vị trí của các campaign trong nhóm E01 — không liên quan đến vị trí của campaign đó ở nhóm E05.

**[A3]** Giả định phạm vi BA dừng ở mức mô hình khái niệm này — quyết định lưu trữ cụ thể (bảng riêng, cột JSON, hay cách khác) thuộc phạm vi SA/Dev, solution này chỉ đảm bảo nghiệp vụ được diễn giải đúng "N vị trí độc lập theo nhóm" để SA thiết kế schema tương ứng.

### 2.5. Case nhiều campaign "Vận hành thường trực" cùng trigger (không có "Có thời hạn" nào trong nhóm)

- Không xuất hiện trong bàn kéo-thả — nhóm này hiển thị dạng ghi chú liệt kê (xem khung UI Mục 2.1).
- Thứ tự xử lý hoàn toàn tự động: `created_at` sớm hơn thắng (tái dùng tiebreak cũ).
- Admin không thao tác gì với nhóm này — chỉ xem để biết đang có bao nhiêu campaign vận hành thường trực cạnh tranh cùng trigger.
- Nếu sau đó có 1 campaign "Có thời hạn" bắt đầu dùng trigger này → nhóm chuyển sang có bàn kéo-thả (chỉ chứa các campaign "Có thời hạn", các "Vận hành thường trực" lùi xuống khối ghi chú phụ, xử lý event vẫn theo rule ưu tiên "Có thời hạn trước" ở Mục 2.6).

### 2.6. Áp dụng thứ tự khi xử lý event thực tế (nhắc lại, không đổi so với clarification 3.4)

1. Khách hàng thỏa đồng thời campaign "Có thời hạn" và "Vận hành thường trực" cùng trùng trigger → campaign "Có thời hạn" luôn được xử lý trước, mặc định, không cấu hình.
2. Trong nhóm chỉ có "Có thời hạn" → theo đúng thứ tự Admin đã sắp tại Cài đặt.
3. Trong nhóm chỉ có "Vận hành thường trực" → theo `created_at` sớm hơn thắng.

### 2.7. Hiển thị cho QTV Marketing tại chính màn Campaign (thay thế risk UX mở — clarification mục 4)

**Bối cảnh**: QTV Marketing là người tạo/sửa campaign nhưng theo quyết định 3.3, chỉ Admin được sắp xếp thứ tự. Quyết định cuối (sau trao đổi với CNTT — Feedback "không cần bỏ field, chỉ cần cho chỉnh được ở ngoài"): **giữ nguyên vị trí field Độ ưu tiên tại Section 1 Campaign Builder và tại Campaign List**, nhưng đổi hoàn toàn hành vi — không còn là ô nhập tay, mà là 1 vùng hiển thị đổi nội dung theo đúng trạng thái thực tế của campaign. Đây là nguồn xem chính cho QTV, thay thế hoàn toàn ý tưởng badge ở Detail View đã cân nhắc trước đó.

**Lý do đổi hướng so với bản nháp trước**: nếu hiển thị 1 con số cụ thể tại field này, sẽ phát sinh mâu thuẫn dữ liệu — một campaign dùng nhiều trigger có N vị trí độc lập ở N nhóm khác nhau (Mục 2.4), không có "1 con số duy nhất" nào đại diện đúng cho tất cả; đồng thời Cài đặt chỉ liệt kê campaign đã Active, nên tại thời điểm Draft/Pending, không có dữ liệu thật nào để hiển thị hay dẫn link tới. Vì vậy nội dung field phải đổi theo đúng trạng thái:

| Trạng thái campaign | Hiển thị tại field "Độ ưu tiên" (Section 1 Builder + Campaign List) |
|---|---|
| **Draft / Pending** (chưa Active) | Dòng chữ thông tin tĩnh, không bấm được: *"Độ ưu tiên sẽ được thiết lập tại Cài đặt sau khi chiến dịch được Duyệt và Kích hoạt"* |
| **Active** | Link thật, dẫn đúng tới nhóm trigger của campaign tại Cài đặt: *"Xem/Điều chỉnh tại Cài đặt →"* — không hiện con số (tránh mâu thuẫn multi-nhóm nêu trên); bấm vào mở đúng khối nhóm trigger tương ứng (nếu nhiều nhóm, mở màn Cài đặt với filter sẵn theo campaign này) |
| **Paused** | Dòng chữ thông tin tĩnh: *"Tạm dừng — không tham gia xếp hạng ưu tiên"* (nhất quán quyết định 3.6 — Paused không thuộc nhóm nào) |
| **Ended** | Dòng chữ thông tin tĩnh: *"Đã kết thúc — không còn tham gia xếp hạng ưu tiên"* |
| **Vận hành thường trực** (bất kể trạng thái) | Dòng chữ thông tin tĩnh: *"Chiến dịch vận hành thường trực — không sử dụng cơ chế xếp hạng ưu tiên"* (loại này không bao giờ có vị trí trong bàn kéo-thả) |

Field không bao giờ cho nhập tay ở bất kỳ trạng thái nào — mọi thao tác sắp xếp vẫn tập trung duy nhất tại Cài đặt (giữ đúng nguyên tắc 1 nguồn ghi — Admin only). Field này chỉ đóng vai trò hiển thị thông tin đúng ngữ cảnh, kiêm lối tắt điều hướng khi đã có dữ liệu thật (Active).

**Ưu điểm so với hướng "bỏ hẳn field" ban đầu**: giữ đúng yêu cầu CNTT (không xóa field khỏi màn Tạo), QTV luôn thấy ngay tại màn mình quản lý mà không cần vào Cài đặt mới biết trạng thái ưu tiên của campaign, tránh hoàn toàn rủi ro hiển thị sai lệch khi 1 campaign thuộc nhiều nhóm trigger, và không gợi ý sai rằng đây là field nhập tay được (dòng chữ tĩnh/link rõ ràng khác hẳn style ô input).

### 2.8. Phương án migrate dữ liệu priority cũ

Dữ liệu hiện có: mỗi campaign Active đang mang 1 số `priority` rời rạc (1–9999, không gắn trigger).

**Đề xuất migrate 1 lần (one-time migration), theo logic:**

1. Với mỗi trigger đang được ≥ 1 campaign Active (loại "Có thời hạn" sau khi đã phân loại — xem Assumption A4 dưới) tham chiếu → tạo 1 nhóm.
2. Trong mỗi nhóm, sắp xếp các campaign theo đúng `priority` cũ tăng dần (số nhỏ = ưu tiên cao, giữ đúng thứ tự tương đối đã có) → gán thành vị trí index mới (#1, #2, #3...) cho nhóm đó. **Tie-break**: nếu 2 campaign có `priority` cũ bằng nhau (có thể xảy ra với dữ liệu lịch sử/race-condition cũ dù rule hiện tại chặn trùng khi cả hai Active), dùng `created_at` sớm hơn làm tiêu chí phụ để đảm bảo kết quả migrate xác định, tái lập được giữa các môi trường.
3. Nếu 1 campaign dùng nhiều trigger → áp dụng bước 2 độc lập cho từng nhóm (dùng cùng 1 `priority` cũ làm cơ sở sắp xếp ban đầu ở tất cả các nhóm nó tham gia — vì trước đây chỉ có 1 con số, không có gì tốt hơn để suy ra thứ tự ban đầu khác nhau cho từng nhóm).
4. Sau bước migrate, cột/field `priority` cũ được loại bỏ khỏi nghiệp vụ (không hiển thị, không dùng để tính toán) — chỉ giữ lại nếu SA/Dev cần cho mục đích audit lịch sử.

**[A4]** Đã chốt với Jun: tại thời điểm migrate, **toàn bộ campaign Active hiện có được phân loại là "Có thời hạn"** — không có ngoại lệ nào cần đánh dấu riêng là "Vận hành thường trực".

---

## 3. Flow Campaign Builder rút gọn (patch UC-CAM-02/03)

### 3.1. Đổi field Độ ưu tiên từ nhập tay sang hiển thị theo trạng thái

Giữ nguyên vị trí field "Độ ưu tiên" (STT 5 hiện tại) tại Section 1 — Thông tin Campaign, nhưng bỏ hoàn toàn khả năng nhập tay (xóa ô input + validate range 1–9999). Thay bằng vùng hiển thị đổi nội dung theo đúng trạng thái campaign — xem bảng chi tiết tại Mục 2.7. Thao tác sắp xếp thật sự vẫn diễn ra duy nhất tại Cài đặt (Mục 2).

### 3.2. Thêm lựa chọn loại hình chiến dịch

**Vị trí đặt trong luồng tạo:** ngay trong Section 1 — Thông tin Campaign, đặt **trước** field Thời gian hiệu lực (vì lựa chọn này quyết định field Ngày kết thúc có hiển thị hay không — đặt trước để QTV hiểu ngữ cảnh trước khi thấy field bị ảnh hưởng).

```
Section 1 — Thông tin Campaign
1. Tên campaign
2. Mã kịch bản (auto)
3. Mục tiêu
4. [MỚI] Loại hình chiến dịch: ( ) Vận hành thường trực   ( ) Có thời hạn
5. Thời gian hiệu lực (Từ ngày – Đến ngày) ← hành vi Đến ngày phụ thuộc STT 4
6. Người tạo (auto)
```

- Radio button, bắt buộc chọn, không có mặc định pre-select (buộc QTV chọn tường minh — đúng quyết định 3.1 "không suy ra ngầm").
- Khi tạo mới, nếu QTV chưa chọn → không tính là lỗi ngay (chưa chạm) nhưng là blocking issue khi Gửi duyệt, tương tự Tên campaign.

### 3.3. Ảnh hưởng đến field Ngày kết thúc (Kênh & Lịch gửi hiện đang mô tả tại Section 1 STT 4 — theo đúng cấu trúc hiện có, "Thời gian hiệu lực" nằm ở Section 1, không phải Kênh & Lịch gửi)

**[A5]** Ghi chú làm rõ vị trí: theo URD hiện tại, field "Đến ngày / Vô hạn" nằm ở **Section 1 STT 4** (Thời gian hiệu lực), không phải trong khối "Kênh & Lịch gửi" (khối đó chỉ có Thời gian gửi trong ngày + Blackout + Nhắc lại). Clarification mục 3.2 gọi chung nhóm "Kênh & Lịch gửi" theo nghĩa nghiệp vụ rộng (toàn bộ cấu hình liên quan đến thời gian/kênh), nhưng khi patch URD cụ thể, field Ngày kết thúc vẫn sửa tại đúng vị trí hiện có — Section 1 STT 4. Nếu Jun muốn dời hẳn field này sang khối Kênh & Lịch gửi luôn trong đợt patch này, cần nêu rõ — mặc định solution giữ nguyên vị trí Section 1, chỉ đổi hành vi hiển thị.

Hành vi cụ thể theo loại hình đã chọn ở STT 4:

| Loại hình | Ngày bắt đầu | Ngày kết thúc |
|---|---|---|
| Vận hành thường trực | Hiển thị, bắt buộc | **Ẩn hẳn khỏi form** (không phải disabled — ẩn); hệ thống tự gán = Vô hạn (tương đương checkbox "Vô hạn" đang tick sẵn và bị khóa ở mô hình cũ); campaign chạy đến khi bị Dừng chủ động (UC-CAM-06) |
| Có thời hạn | Hiển thị, bắt buộc | Hiển thị, **bắt buộc nhập** (bỏ hẳn checkbox "Vô hạn" — loại này luôn phải có ngày kết thúc cụ thể) |

- **Lần đầu chọn radio** (chưa có lựa chọn nào trước đó, null → "Vận hành thường trực" hoặc null → "Có thời hạn"): đơn thuần là show/hide field Ngày kết thúc theo bảng trên — không có dữ liệu nào để xóa vì field Ngày kết thúc chưa từng hiển thị/nhập trước đó.
- **Đổi ý sau khi đã chọn** từ "Có thời hạn" → "Vận hành thường trực" khi đang ở Draft (chưa Active): ẩn ngay field Ngày kết thúc, dữ liệu đã nhập trước đó (nếu có) bị xóa không cảnh báo (giống hành vi ẩn field khác khi đổi lựa chọn liên quan trong URD hiện có, ví dụ AND mode ẩn Audience Variant).
- **Đổi ý ngược lại** "Vận hành thường trực" → "Có thời hạn" khi đang Draft: hiện lại field Ngày kết thúc, bắt buộc nhập lại từ đầu (trống, không khôi phục giá trị cũ nếu có trước đó).
- Case đổi loại hình **sau khi đã Active** → xử lý riêng ở Mục 5.1 (phức tạp hơn, ảnh hưởng vị trí trong nhóm).

### 3.4. Cập nhật UC-CAM-01 (Danh sách) — đổi inline priority thành hiển thị trạng thái

- Giữ cột "Ưu tiên" (STT 7 hiện tại) nhưng bỏ hoàn toàn hành vi inline-edit gắn với nó (bước 5/5a/5a-i/5b/5c trong Hoạt động, nhánh Alternative sửa priority Draft) — nội dung cột đổi theo đúng bảng trạng thái ở Mục 2.7 (dòng chữ tĩnh hoặc link, không có ô nhập).
- Thêm mới cột **Loại hình chiến dịch** (chip nhỏ "Thường trực" / "Có thời hạn") cạnh cột Ưu tiên — giúp QTV/Admin phân biệt nhanh 2 loại ngay tại danh sách.
- Sắp xếp mặc định (hiện tại: Trạng thái → Ưu tiên): đã chốt đổi thành **Trạng thái → Ngày tạo (mới nhất trước)** — vì "Ưu tiên" không còn là 1 số đơn của campaign để sort toàn cục (có thể là N vị trí khác nhau theo nhóm, hoặc chỉ là dòng chữ tĩnh).

---

## 4. Flow Duyệt (UC-CAM-05) sau khi tách rời validate ưu tiên

### 4.1. Bỏ hẳn bước check trùng priority

- Xóa toàn bộ bước 3/3a/3b trong Hoạt động (check trùng độ ưu tiên khi bấm [Duyệt]) và nhánh Quy tắc nghiệp vụ tương ứng (V4.15).
- [Duyệt] quay về hành vi đơn giản: bấm → dialog xác nhận bình thường "Duyệt chiến dịch [Tên]? Chiến dịch sẽ chuyển sang trạng thái Đang chạy ngay" → [Hủy]/[Duyệt] → Active.
- [Từ chối] không đổi — vẫn là action Admin tự quyết định, không còn ràng buộc "chỉ là con đường sửa priority" (vì không còn gì để sửa liên quan priority tại Builder nữa).

### 4.2. Xử lý sau khi Active

- Ngay khi campaign chuyển Active (dù qua Duyệt hay qua Bật lại từ Paused): hệ thống tự động thêm campaign vào cuối mỗi nhóm trigger tương ứng (đúng rule Mục 2.3).
- Không có trạng thái "chờ xếp hạng" — ngay khi vào nhóm ở vị trí cuối, campaign xử lý event bình thường theo đúng vị trí đó, không bị chặn, không cần Admin xác nhận thêm bước nào.
- Nếu Admin muốn thay đổi vị trí, phải chủ động vào Cài đặt — đây là hành động tách rời hoàn toàn khỏi luồng Duyệt.

---

## 5. Edge cases

### 5.1. Campaign đổi loại hình sau khi đã Active

Về nguyên tắc, campaign Active không thể sửa trực tiếp (phải qua [Sửa] → Draft → gửi duyệt lại, theo hành vi hiện có của UC-CAM-03/UC-CAM-04). Loại hình chiến dịch là 1 field trong Section 1 nên tuân theo đúng luồng đó — không có "đổi loại hình trực tiếp trên campaign đang Active".

Vậy case thực tế là: **campaign đang Active/Paused, QTV bấm [Sửa] → về Draft → đổi loại hình → gửi duyệt lại → Admin duyệt → Active lại.**

| Bước | Xử lý vị trí ưu tiên |
|---|---|
| Campaign Active → [Sửa] → Draft | Gỡ ngay khỏi mọi nhóm đang tham gia (đúng hành vi chung: campaign không Active thì không có mặt trong bàn kéo-thả) — giống hệt xử lý khi Paused (Mục 2.3) |
| Đổi "Có thời hạn" → "Vận hành thường trực" tại Draft, gửi duyệt, Admin duyệt lại | Không quay lại bàn kéo-thả của bất kỳ nhóm nào (vì giờ là Vận hành thường trực) — tham gia tiebreak `created_at` nếu trùng trigger với các "Vận hành thường trực" khác |
| Đổi "Vận hành thường trực" → "Có thời hạn" tại Draft, gửi duyệt, Admin duyệt lại | Coi như campaign "Có thời hạn" mới Active lần đầu → tự động thêm vào **cuối** mỗi nhóm trigger tương ứng (rule "luôn xếp cuối", không có ưu tiên đặc biệt gì vì "đã từng Active trước đó") |

**[A6]** Giả định: vị trí ưu tiên cũ (trước khi vào [Sửa]) **không được khôi phục** dù đổi loại hình qua lại — mọi lần một campaign "Có thời hạn" (re-)xuất hiện trong bàn kéo-thả đều coi là lần thêm mới, xếp cuối. Điều này nhất quán với rule đã chốt ở clarification 3.3 (không có ngoại lệ chèn giữa) và đơn giản hóa logic (không cần lưu lịch sử vị trí cũ). Nếu Jun muốn giữ lại vị trí cũ trong 1 số trường hợp cụ thể, cần nêu rõ case đó.

### 5.2. Campaign bị xóa/Kết thúc (Ended) khi đang giữ vị trí trong nhóm

- Campaign không thể "xóa cứng" theo ràng buộc đã có ở C.2 (Toàn vẹn dữ liệu — Campaign chỉ đổi trạng thái, không xóa cứng) → case thực tế duy nhất là **chuyển Ended** (do hết `endDate` qua background job II.6.4, hoặc do Admin/QTV chủ động Dừng rồi campaign đó không bao giờ Bật lại — nhưng trạng thái đó vẫn là Paused, không tự thành Ended trừ qua hết hạn).
- Khi campaign chuyển Ended: gỡ khỏi mọi nhóm đang tham gia ngay lập tức (cùng cơ chế với chuyển Paused — Mục 2.3).
- Các thành viên còn lại trong (các) nhóm đó: **giữ nguyên thứ tự tương đối**, không dồn số cứng — đúng nguyên tắc đã áp dụng cho case gỡ do đổi trigger (Mục 2.3).
- Nếu nhóm chỉ còn lại 1 thành viên "Có thời hạn" sau khi gỡ: nhóm chuyển về dạng hiển thị đơn giản (không cần bàn kéo-thả) — xem Mục 5.3.
- Nếu nhóm không còn thành viên "Có thời hạn" nào (hết sạch) nhưng vẫn còn "Vận hành thường trực" dùng trigger đó: nhóm chuyển thành dạng "chỉ có Vận hành thường trực" (Mục 2.5).
- Nếu nhóm không còn campaign Active nào cả (không phân biệt loại): nhóm biến mất khỏi màn Cài đặt (đúng logic Mục 2.2 — chỉ hiện nhóm có ít nhất 1 campaign Active tham chiếu).

### 5.3. Campaign "Có thời hạn" không trùng trigger với bất kỳ campaign nào khác (nhóm chỉ có 1 thành viên)

- Vẫn hiển thị tại Cài đặt (không ẩn) — vì Admin cần biết đầy đủ campaign nào đang dùng trigger nào, kể cả khi chưa có cạnh tranh.
- Hiển thị dạng đơn giản, không có kéo-thả (kéo-thả với 1 dòng là vô nghĩa): ví dụ 1 dòng tĩnh "Campaign [Tên] — vị trí #1 (không có campaign **Có thời hạn** nào khác cạnh tranh cùng trigger này)" — cố ý ghi rõ "Có thời hạn" thay vì "không có campaign nào khác" chung chung, vì nhóm vẫn có thể có campaign Vận hành thường trực cùng trigger (hiển thị ở khối ghi chú phụ theo Mục 2.1), chỉ là chúng không tham gia cạnh tranh vị trí.
- Vị trí #1 mặc định, không cần Admin thao tác gì; nếu sau đó có thêm 1 campaign "Có thời hạn" khác cũng dùng trigger này → nhóm tự động chuyển sang dạng có bàn kéo-thả, campaign mới luôn vào vị trí cuối (tức #2), campaign hiện tại giữ #1.

---

## 6. Danh sách UC/Screen cần patch cụ thể (chuẩn bị cho SYNC)

| # | UC/Screen | Loại thay đổi | Tóm tắt nội dung cần sửa |
|---|---|---|---|
| 1 | **URD II.6.8** (Cross-campaign Priority) | REWRITE | Viết lại toàn bộ cơ chế: bỏ priority score toàn cục, thay bằng mô hình nhóm theo trigger + N vị trí độc lập/campaign; giữ nguyên phần tiebreak `created_at` (dùng lại cho case Vận hành thường trực); bổ sung rule "Có thời hạn luôn ưu tiên hơn Vận hành thường trực cùng trigger" |
| 2 | **UC-CAM-01** (Danh sách) | PATCH | Giữ cột Ưu tiên (STT 7) nhưng bỏ toàn bộ hành vi inline-edit (bước 5, 5a, 5a-i, 5b, 5c, Alternative sửa priority Draft) — đổi nội dung cột theo bảng trạng thái Mục 2.7; thêm cột mới Loại hình chiến dịch (chip); đổi tiêu chí sort mặc định (xem OQ-2) |
| 3 | **UC-CAM-02** (Tạo) | PATCH | Đổi field Độ ưu tiên ở bước 1 từ ô nhập tay sang vùng hiển thị theo trạng thái (Mục 2.7) + cập nhật Quy tắc nghiệp vụ liên quan; thêm bước chọn Loại hình chiến dịch (radio, bắt buộc) trước bước nhập Thời gian hiệu lực; cập nhật hành vi field Ngày kết thúc theo loại hình (ẩn = Vô hạn / bắt buộc); bỏ bước 7c (check trùng priority khi Lưu Nháp) và phần Exception liên quan priority |
| 4 | **UC-CAM-03** (Sửa Draft) | PATCH (kế thừa) | Không có thay đổi trực tiếp trong Hoạt động — kế thừa toàn bộ thay đổi từ UC-CAM-02 vì logic dùng chung; rà soát Quy tắc nghiệp vụ không còn tham chiếu priority nào sót lại |
| 5 | **UC-CAM-05** (Duyệt) | PATCH | Bỏ bước 3/3a/3b (check trùng priority) trong Hoạt động; bỏ dòng Quy tắc nghiệp vụ "Chặn trùng độ ưu tiên khi Phê duyệt" (V4.15); bổ sung 1 dòng: sau khi Active, hệ thống tự động thêm campaign vào cuối nhóm trigger tương ứng (dẫn tham chiếu sang II.6.8 mới) |
| 6 | **UC-CAM-07** (Bật lại) | RÀ SOÁT | Không có bước liên quan priority trực tiếp trong bản hiện tại — chỉ cần bổ sung 1 dòng: khi Bật lại thành công (Active), campaign coi như "mới thêm" vào nhóm, xếp cuối (dẫn tham chiếu II.6.8) |
| 7 | **UC-PRIORITY-01** | REWRITE | Đổi tên thành "Cấu hình Độ ưu tiên theo nhóm Trigger"; viết lại toàn bộ Hoạt động + Quy tắc nghiệp vụ theo Mục 2 của solution này (nhóm, filter, kéo-thả trong nhóm, tự động thêm/gỡ, không còn ô nhập số tay, không còn khái niệm trùng) |
| 8 | **Screen 2** (Campaign List) | PATCH | Giữ STT 7 (cột Ưu tiên) nhưng đổi từ inline edit sang hiển thị theo trạng thái (Mục 2.7); thêm cột Loại hình chiến dịch; cập nhật STT 10 (tiêu chí sort mặc định) |
| 9 | **Screen 3 — Section 1** (Campaign Builder) | PATCH | Đổi STT 5 (Độ ưu tiên) từ ô nhập tay sang vùng hiển thị theo trạng thái (Mục 2.7); thêm field mới "Loại hình chiến dịch" (radio, bắt buộc) trước STT 4 (Thời gian hiệu lực) — đánh lại số STT; cập nhật mô tả field Đến ngày (STT 4) theo loại hình đã chọn |
| 10 | **Screen 3 — Kênh & Lịch gửi** | RÀ SOÁT | Không có field priority ở đây theo cấu trúc hiện tại — chỉ cần xác nhận không có tham chiếu chéo nào đến priority cần dọn |
| 11 | **Screen Settings Tab 3** | REWRITE | Đổi tên tab từ "Priority Matrix" thành "Độ ưu tiên"; viết lại toàn bộ bảng đặc tả component theo cấu trúc nhóm + filter + kéo-thả trong nhóm (Mục 2.1); bỏ STT 3.3 (nhập số tay) và STT 3.4 (cảnh báo trùng) |
| 12 | **Screen 2B** (Campaign Detail View) | RÀ SOÁT | Rà soát Section 1 để đồng bộ cách hiển thị Độ ưu tiên với Mục 2.7 nếu màn này có hiển thị field này riêng biệt — không còn là patch bắt buộc theo P2 (đã thay bằng hiển thị tại chính Builder/List) |
| 13 | **Traceability Map** | UPDATE | REQ-CVM-020, REQ-CVM-054, REQ-CVM-061 cần viết lại hoàn toàn (bản chất nghiệp vụ đổi, không phải chỉnh nhỏ); rà soát reverse index UC-CAM-01/02/05/07, UC-PRIORITY-01 và các Screen liên quan |

**[A7]** Giả định về STT 10: các UC/Screen khác không nằm trong phạm vi clarification (UC-CAM-04, UC-CAM-06, UC-CAM-08, Screen 2B ngoài phần Section 1, Function Tree II.2, Permission Matrix II.3, RBAC II.4, Sequence Diagram II.5) không cần patch — vì không có tham chiếu trực tiếp đến priority theo rà soát URD hiện tại, ngoại trừ Screen 2B (đã liệt kê ở dòng 12) và UC-CAM-04 (Section 1 hiển thị "độ ưu tiên" — cần rà soát cùng đợt patch Screen 2B, gộp chung dòng 12). Bước impact-analysis ở SYNC cần grep lại toàn văn để xác nhận không sót.

---

## 7. Open Questions

- [x] ~~OQ-1 (cũ): Chọn phương án hiển thị read-only cho QTV Marketing~~ → **Resolved**: không dùng P1/P2/P3 nêu ban đầu — chốt phương án hiển thị trạng thái ngay tại field Độ ưu tiên gốc (Section 1 Builder + Campaign List), xem Mục 2.7.
- [x] **OQ-2**: Có campaign Active nào hiện tại trên hệ thống thực tế mang bản chất "vận hành thường trực" cần đánh dấu riêng khi migrate dữ liệu priority cũ không? → **Resolved: Không** — migrate 100% campaign Active hiện có thành "Có thời hạn" (đúng A4).
- [x] **OQ-3**: Tiêu chí sort phụ tại Campaign List sau khi bỏ sort theo Ưu tiên (trong cùng trạng thái) — Ngày tạo mới nhất trước, hay Tên A-Z? → **Resolved: Ngày tạo mới nhất trước**.
- [~] **OQ-4 (BLOCKER, tạm hoãn)**: CNTT đặt câu hỏi ranh giới phạm vi — nếu tin nhắn loại "Vận hành thường trực" (ví dụ campaign kích hoạt sim) thực chất do hệ thống **Core** tự gửi, CVM có cần giữ khái niệm "Vận hành thường trực" trong Campaign Builder nữa không, hay toàn bộ solution chỉ còn phục vụ loại "Có thời hạn" (bỏ hẳn radio chọn loại hình vì chỉ còn 1 loại). Jun xác nhận: tạm để lại, chưa ảnh hưởng nhiều, cần làm việc với CNTT trước khi patch URD chính thức — không chặn việc hoàn thiện solution doc ở bước này, nhưng **phải chốt trước khi chạy SYNC patch thật**.

---

## 8. Assumptions tổng hợp

- **[A1]** 1 nhóm hiển thị = 1 trigger cụ thể (không gộp theo tổ hợp trigger); nếu sai, cần thiết kế lại UI nhóm ở Mục 2.1.
- **[A2]** Badge "Mới" cho campaign vừa tự động thêm vào nhóm là đề xuất UX không bắt buộc; có thể bỏ mà không ảnh hưởng logic cốt lõi.
- **[A3]** Solution dừng ở mô hình dữ liệu khái niệm ("N vị trí độc lập theo nhóm trigger"); schema cụ thể là phạm vi SA/Dev.
- **[A4]** Toàn bộ campaign Active hiện có được coi là "Có thời hạn" khi migrate, trừ khi Jun cung cấp danh sách ngoại lệ cụ thể (xem OQ-2).
- **[A5]** Field Ngày kết thúc giữ nguyên vị trí Section 1 STT 4 khi patch, chỉ đổi hành vi hiển thị theo loại hình — không dời sang khối Kênh & Lịch gửi trừ khi Jun yêu cầu rõ.
- **[A6]** Vị trí ưu tiên cũ không được khôi phục khi campaign đổi loại hình qua lại hoặc Paused rồi Active lại — mọi lần (tái) xuất hiện trong bàn kéo-thả đều xếp cuối.
- **[A7]** Ngoài các UC/Screen liệt kê ở Mục 6, không có vị trí nào khác trong URD V4.17 cần patch cho thay đổi này — cần xác nhận lại bằng grep toàn văn ở bước impact-analysis.
- **[A8]** Field "Độ ưu tiên" tại Section 1 Builder và Campaign List không bao giờ cho nhập tay ở bất kỳ trạng thái campaign nào (Draft/Pending/Active/Paused/Ended) — chỉ hiển thị dòng chữ tĩnh hoặc link điều hướng theo đúng bảng trạng thái Mục 2.7; mọi thao tác ghi/sắp xếp vẫn tập trung 100% tại Cài đặt (Admin only), đúng tinh thần "1 nguồn ghi duy nhất" dù vẫn giữ field hiển thị tại 2 nơi theo yêu cầu CNTT.
- **[A9]** Đã xác nhận với Jun: không có loại campaign nào trong danh mục nghiệp vụ CVM bắt buộc phải được ưu tiên tuyệt đối ngay khi vừa Active — toàn bộ phụ thuộc vào việc Admin chủ động vào Cài đặt sắp xếp. Giữ nguyên thiết kế "luôn xếp cuối, không ngoại lệ" (Mục 2.3); không cần thêm cơ chế báo động/nhắc việc chủ động nào ngoài badge thụ động đã đề xuất ở A2. Rủi ro "cửa sổ chờ Admin" được chấp nhận như một đánh đổi vận hành đã biết trước.
- **[A10 — Risk vận hành, chấp nhận đánh đổi]** Cơ chế mới tập trung toàn bộ quyền ghi vào duy nhất 1 nơi (Cài đặt, Admin only) — khác với cơ chế cũ có 3 cửa ghi dự phòng (Campaign List, Campaign Builder, Priority Matrix). Nếu màn Cài đặt gặp sự cố kỹ thuật hoặc Admin bị thu hồi nhầm quyền truy cập, không còn cách nào khác để điều chỉnh thứ tự ưu tiên khẩn cấp (vì field hiển thị ở Builder/List theo A8 chỉ đọc, không ghi được). Đây là đánh đổi được chấp nhận có chủ đích (đổi lấy việc loại bỏ hoàn toàn khái niệm "trùng"), không cần giải pháp kỹ thuật ở bước BA — chỉ ghi nhận để Jun cân nhắc có cần đề xuất thêm 1 role "second Admin" dự phòng khi chuyển sang SA/Dev hay không.

---

## 9. Bước tiếp theo đề xuất

1. ~~Jun review solution, quyết định OQ-1/OQ-2/OQ-3~~ — Đã xong (xem Mục 7).
2. ~~Chạy `ba-devil-advocate-agent` phản biện~~ — Đã xong: PASS WITH CONDITIONS, 3 MAJOR + 5 MINOR đã được vá trực tiếp vào bản này (Mục 1.3 mới, Mục 2.1, 2.3, 2.8, 3.3, 5.3, Assumption A9/A10).
3. Jun chốt OQ-4 (ranh giới CVM/Core với CNTT) — điều kiện bắt buộc trước khi patch URD thật.
4. Chuyển sang chế độ SYNC: `change-handler` → `impact-analysis` (dựa trên Traceability Map + bảng Mục 6) → `artifact-patch` → `ba-qa-agent` → `ba-postcheck-agent`.
