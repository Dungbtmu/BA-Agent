# Audit Report — urd-srs-v4.md (CVM)

**Thời điểm audit:** 21/08/2026
**Tài liệu được audit:** `.claude/output/CVM/urd/urd-srs-v4.md`
**Bối cảnh:** SYNC patch v3 → v4 (Change Set 2026-08-21, 14 thay đổi) + 2 fix CRITICAL từ QA (S151 workflow, tham chiếu "Section 3") — **chưa chạy lại ba-qa-agent để xác nhận nội dung nghiệp vụ**. Audit này chỉ kiểm tra cấu trúc/metadata, không thay thế bước QA còn thiếu.

---

### Checklist kết quả

| # | Kiểm tra | Kết quả | Ghi chú |
|---|---|---|---|
| 1 | Section Completeness | PASS | I, II (II.1–II.5 + II.6/II.7 bổ sung), III, IV, C đều đầy đủ, đúng thứ tự. 17 mục `[Cần xác nhận]` — tất cả đã có mặt trong danh sách "Mục cần xác nhận" cuối tài liệu (12 mục còn mở, liệt kê rõ vị trí) → hợp lệ theo quy tắc WARN trong URD đang làm việc |
| 2 | Traceability Audit | FAIL (nội bộ URD) — nhưng không phát sinh từ patch | Function Tree lá: 36 · Permission Matrix: 26 entries · RBAC: 26 dòng · UC: 29 · Screen headings chính: 21. Gap chính: **UC-BL-04 (Blacklist toàn hệ thống) không có mặt trong Function Tree (II.2), Permission Matrix (II.3), RBAC Matrix (II.4)** dù đã có UC-BL-04 đầy đủ ở Section III và Screen 6A STT 13 ở Section IV |
| 3 | Naming Consistency | PASS | UC ID, Screen ID nhất quán xuyên III và IV. Không phát hiện actor/function/screen bị gọi tên khác nhau giữa các section |
| 4 | Version Hygiene | PASS | File name `urd-srs-v4.md` đúng format. Version log cuối bảng "CÁC THAY ĐỔI" (dòng 56) ghi `V4.0`, ngày `21/08/2026`. Footer cuối file ghi `Phiên bản: V4.0 \| Ngày: 21/08/2026` — khớp. Version history không nhảy số (V1.0 → ... → V4.0 tuần tự, có cả các sub-version V3.x hợp lệ) |
| 5 | Placeholder Detection | FAIL — pre-existing, không phát sinh từ patch | `N/A` xuất hiện 4 lần trong cột **Mô tả** của bảng 4 cột (STT/Tên/Định dạng/Mô tả) tại Screen 1 Dashboard (dòng 1841 khu vực Row... thực tế offset 472-476 khi đếm từ đầu Section IV) và Screen 9 Report Tab 2 (dòng 2295, 2297) — vi phạm quy tắc "N/A cấm ở cột Mô tả". Không tìm thấy TBD/TODO/dấu `...` bỏ lửng/template placeholder chưa điền |

---

### Chi tiết Kiểm tra 2 — Traceability Audit

**Đếm chi tiết:**
```
Function Tree (lá):        36
  Khối 1: 8 · Khối 2: 5 · Khối 3: 5 · Khối 4: 4 · Khối 5: 2 · Khối 6: 7 · Khối 7: 5
Permission Matrix (entries): 26  → Delta so với Function Tree: -10
  (một số dòng Permission Matrix gộp nhiều chức năng lá — ví dụ "Tạo / Xem chi tiết / Sửa mẫu tin nhắn"
  gộp 3 lá Function Tree thành 1 dòng — pattern đã tồn tại từ v1, không phải lỗi phát sinh từ patch v3→v4)
RBAC Matrix (dòng):          26  → tương ứng 1-1 với Permission Matrix, không lệch thêm
Use Case (UC IDs):           29  → nhiều hơn Permission Matrix entries — hợp lý (1 chức năng có thể có nhiều UC)
Màn hình (Section IV headings chính): 21 (Screen 1, 2, 2B, 3, 4A, 4B, 4C, 5A, 5B, 6A, 6B, 6C, 7, 8, 9,
  Screen Admin chính, Tab Trigger Admin ×3, Screen Settings) → đủ để cover 29 UC (nhiều UC dùng chung 1 màn hình,
  ví dụ UC-CAM-02/03 cùng dùng Screen 3)
```

**Gap cụ thể phát hiện — không đối xứng giữa 3 ma trận tổng thể và UC/Screen mới thêm ở v4:**

REQ-CVM-067 (Blacklist toàn hệ thống, patch #9 trong Change Set 2026-08-21) đã được thêm đầy đủ ở:
- Section III: `UC-BL-04: Thêm thủ công số vào Blacklist toàn hệ thống` (dòng ~1468)
- Section IV: Screen 6A STT 13 "Nút [+ Thêm vào Blacklist toàn hệ thống]" (chỉ Admin thấy)

Nhưng **chưa được thêm** vào:
- II.2 Function Tree — Khối 4 chỉ có 4 chức năng lá cũ (Xem danh sách / Thêm thủ công / Upload CSV / Xóa), không có "Thêm vào Blacklist toàn hệ thống"
- II.3 Permission Matrix — Khối 4 "Quản lý Danh sách chặn" chỉ có 4 dòng cũ, không có dòng riêng cho quyền Blacklist toàn hệ thống (theo UC-BL-04, chỉ Admin CREATE/DELETE, QTV không thấy)
- II.4 RBAC Matrix — Khối 4 "Blacklist CVM" ghi chung 1 dòng `VIEW, DELETE | VIEW, CREATE, DELETE` — không phân biệt quyền Blacklist theo campaign (QTV có CREATE) và Blacklist toàn hệ thống (chỉ Admin có CREATE, theo đúng UC-BL-04 đã đặc tả)

Đây là đúng loại lỗi mà chính tài liệu này đã tự phát hiện và sửa ở V3.23 ("đồng bộ 3 ma trận tổng thể với tầng UC/Screen"). Change Set 2026-08-21 patch #9 chỉ chạm Section III/IV, chưa lan sang II.2/II.3/II.4.

---

### Chi tiết Kiểm tra 5 — Placeholder Detection

| Vị trí | Bảng | Nội dung |
|---|---|---|
| Screen 1 — Dashboard, Tab/Row hiển thị KPI kênh | Bảng 4 cột (STT/Tên/Định dạng/Mô tả) | `"kênh không đo được → hiển thị N/A"` — 3 lần liên tiếp (Tỉ lệ mở, CTR, Tỉ lệ chuyển đổi) |
| Screen 1 — Biểu đồ hiệu suất theo kênh | Bảng 4 cột | `"Kênh không đo được metric → bar = 0 + label N/A"` |
| Screen 9 — Report Tab 2 (Tương tác), dòng 2295, 2297 | Bảng 4 cột | Cùng pattern N/A trong cột Mô tả |

Đây là bảng dạng 4 cột (không có cột Bắt buộc/Mặc định — vì là bảng hiển thị KPI/report, không phải input component), nên N/A chỉ có thể nằm ở cột Mô tả. Về mặt nghiệp vụ, N/A ở đây mô tả **giá trị UI thực sự hiển thị cho user** (kênh SMS/USSD không đo được Open Rate/CTR nên UI show "N/A") — không phải placeholder chưa điền. Tuy nhiên theo đúng nghĩa đen của quy tắc Kiểm tra 5 (N/A bị cấm tuyệt đối ở cột Mô tả bất kỳ component nào trong Section IV), đây vẫn được flag để BA quyết định — không tự diễn giải ngoại lệ.

**Ghi chú:** Không phải lỗi phát sinh từ SYNC patch v3→v4 — nội dung này đã tồn tại từ các version trước (không nằm trong danh sách thay đổi V4.0). Không liên quan đến 2 CRITICAL đã sửa.

---

### Danh sách lỗi cần sửa

**[PC-01]** Blacklist toàn hệ thống (UC-BL-04, REQ-CVM-067) chưa được đồng bộ vào 3 ma trận tổng thể
- Kiểm tra: Traceability Audit
- Vị trí: II.2 Function Tree (Khối 4), II.3 Permission Matrix (Khối 4), II.4 RBAC Matrix (Khối 4 — dòng "Blacklist CVM")
- Hành động cần làm: Bổ sung chức năng lá "Thêm/Xóa Blacklist toàn hệ thống" vào Function Tree; thêm dòng Permission Matrix phân biệt quyền Blacklist theo campaign (QTV CREATE) vs Blacklist toàn hệ thống (chỉ Admin CREATE/DELETE, QTV chỉ VIEW); tách dòng RBAC Matrix Khối 4 tương ứng — nhất quán với cách tài liệu đã tự sửa ở V3.23 cho trường hợp tương tự (Trigger)

**[PC-02]** N/A xuất hiện trong cột Mô tả của bảng component Section IV (Screen 1, Screen 9)
- Kiểm tra: Placeholder Detection
- Vị trí: Screen 1 — bảng ROW/Tab hiệu suất kênh (3 dòng); Screen 9 — Tab 2 Tương tác, STT 2.1–2.3 và 2.5
- Hành động cần làm: BA xác nhận có coi đây là vi phạm quy tắc cứng "cấm N/A ở cột Mô tả" hay là ngoại lệ hợp lệ (N/A ở đây mô tả giá trị UI thực tế hiển thị cho end-user khi kênh không đo được metric, không phải placeholder bỏ trống). Nếu cần sửa: đổi cách diễn đạt, ví dụ thay `→ hiển thị "N/A"` bằng `→ hiển thị dấu gạch ngang "–" kèm chú thích "Không đo được ở kênh này"` để tránh trùng ký hiệu N/A dùng cho placeholder. Đây là vấn đề pre-existing từ trước v4, không phát sinh từ SYNC patch lần này.

---

### Xác nhận riêng — 2 điểm QA đã tự sửa tay

- **S151 (Workflow Diagram, node kích hoạt lại)**: đã kiểm tra dòng 379, 398–400 — mô tả đúng cả 2 nhánh "Không đổi param/lọc → về Đang chạy" và "Có đổi param/lọc → về Chờ duyệt", khớp với UC-CAM-07 patch #3 trong Change Set. Không còn sai logic.
- **5 chỗ tham chiếu "Section 3"**: đã quét toàn bộ 6 lần xuất hiện "Section 3" ngoài bảng version log lịch sử — tất cả đều là mô tả hợp lệ (Section 3 = Audience, vẫn tồn tại như 1 section; chỉ sub-mục "Điều kiện lọc" đã dời sang Section 4, và các câu văn đều ghi rõ "đã chuyển từ Section 3 xuống Section 4"). Không phát hiện tham chiếu chết còn sót.

**Lưu ý ngoài phạm vi audit:** Nội dung PC-02 (cách diễn đạt N/A cho metric không đo được) mang tính nghiệp vụ nhẹ — nếu BA muốn đánh giá sâu hơn cách trình bày UI cho case này, nên đưa vào lượt chạy `ba-qa-agent` sắp tới (vốn đang chờ chạy lại sau khi tự sửa 2 CRITICAL).

---

### Phán quyết

**NEEDS FIX**

Phát hiện 1 lỗi traceability (PC-01 — gap 3 ma trận tổng thể với UC-BL-04) cần sửa trước khi handoff, và 1 lỗi placeholder (PC-02) cần BA quyết định có sửa hay chấp nhận là ngoại lệ hợp lệ. Không có mục `[Cần xác nhận]` critical nào chưa xử lý — toàn bộ 12 mục còn mở đều thuộc loại non-critical (đã liệt kê rõ ràng ở cuối tài liệu, PO/bảo mật cần xác nhận nhưng không chặn cấu trúc).

Ngoài phạm vi 5 kiểm tra postcheck: tài liệu vẫn **chưa qua ba-qa-agent xác nhận lại** sau khi tự sửa 2 CRITICAL (S151 + Section 3 refs) — đề nghị chạy `ba-qa-agent` trước khi coi văn bản này là chốt để chạy lại `ba-postcheck-agent` lần cuối.

Sau khi sửa PC-01 (và quyết định PC-02), chạy lại `ba-postcheck-agent` để xác nhận READY FOR HANDOFF.
