# Review Report — URD/SRS CVM v3.22

Phạm vi review: đợt cập nhật V3.22 (13/07/2026) — cơ chế "điều kiện lọc phân khúc theo trigger" (Filter field), policy `FILTER_INVALID`, và traceability nội bộ xuống các ma trận II.2/II.3/II.4.

File review: `.claude/output/cvm/urd/urd-srs-v3.md`

---

## CRITICAL — Phải sửa trước khi tiếp tục

**[CR-01]** Permission Matrix II.3 phủ nhận quyền QTV "Xem chi tiết trigger" — trái với quyết định V3.22 "QTV được XEM Nhóm C"
- Vị trí: II.3 Ma trận phân quyền, dòng "Thêm / Xem chi tiết / Sửa sự kiện kích hoạt | Admin HT = X | QTV Marketing = –" (dòng ~545)
- Vấn đề: Dòng này gộp "Xem chi tiết" chung với "Thêm/Sửa" rồi gán QTV = `–` (không được). Nhưng UC-TRG-02 ("Tác nhân: Admin (xem + sửa), QTV Marketing (chỉ đọc)"), Screen 5B (Modal Xem chi tiết Trigger cho QTV, có đủ Nhóm A/B/C), và quyết định mới "QTV được XEM Nhóm C điều kiện lọc" đều khẳng định QTV CÓ quyền xem chi tiết trigger (bao gồm Nhóm C read-only). Ma trận phân quyền — nguồn chân lý về quyền — lại ghi ngược. Dev/Tester đọc ma trận này sẽ implement/test sai: chặn QTV mở modal chi tiết, phá vỡ chính luồng tra cứu Filter field mà V3.22 vừa thêm.
- Khuyến nghị: Tách dòng thành 2 quyền riêng: (1) "Xem chi tiết sự kiện kích hoạt" → Admin HT = X, QTV = X (hoặc (X) đọc-only); (2) "Khai báo / Sửa trigger + tham số + điều kiện lọc" → Admin HT = X, QTV = `–`. Đảm bảo khớp UC-TRG-02 và Screen 5B. Quay lại bước [7] urd-srs-agent, chỉ sửa II.3.

**[CR-02]** RBAC Matrix II.4 ghi Admin chỉ có VIEW trên Trigger — trái với toàn bộ UC-TRG-03/04/05 và mô tả role Admin
- Vị trí: II.4.3, dòng "3. Trigger | Trigger catalog (chỉ đọc) | ADMIN_HT = VIEW | QTV_MKT = VIEW" (dòng ~596)
- Vấn đề: RBAC gán ADMIN_HT chỉ `VIEW` trên trigger, kèm nhãn "(chỉ đọc)". Điều này mâu thuẫn trực tiếp với: II.4.1 mô tả role ADMIN_HT "khai báo trigger mới và quản lý tham số đầu ra của trigger" (dòng 571); UC-TRG-03 (Admin CREATE trigger); UC-TRG-04 (Admin CREATE/DELETE tham số); UC-TRG-05 (Admin CREATE/DELETE điều kiện lọc). Đây là xung đột nội bộ nghiêm trọng: cùng một tài liệu vừa nói Admin quản lý trigger, vừa nói Admin chỉ được xem. RBAC là ma trận Dev dùng để cấu hình phân quyền backend — nếu để nguyên, backend sẽ chặn Admin tạo/sửa trigger, vô hiệu hóa cả 3 use case mới.
- Khuyến nghị: Sửa dòng Trigger thành: ADMIN_HT = `VIEW, CREATE, UPDATE, DELETE` (áp cho cả trigger, tham số đầu ra, và điều kiện lọc phân khúc); QTV_MKT = `VIEW`. Bỏ nhãn "(chỉ đọc)" trên đối tượng Trigger. Cân nhắc tách rõ "Trigger + Tham số + Điều kiện lọc" trong cột đối tượng để không mơ hồ.

**[CR-03]** Ghi chú II.3 vẫn khẳng định "Trigger catalog chỉ đọc với mọi role — quản lý qua Dev/SA deployment" (mô hình V3.0 đã bị thay thế)
- Vị trí: II.3 phần "Ghi chú", gạch đầu dòng thứ 2 (dòng ~559): "Trigger catalog chỉ đọc với mọi role trên UI — quản lý trigger là trách nhiệm của Dev/SA qua deployment, không phơi thao tác này ra giao diện"
- Vấn đề: Đây là mô tả của mô hình cũ (V3.0) đã bị đảo ngược từ V3.3 (Admin khai báo trigger trực tiếp trên UI) và mở rộng thêm ở V3.22. Câu ghi chú này khẳng định điều ngược hoàn toàn với I.2 (dòng 97), UC-TRG-03/04/05, và Screen Admin Tab Trigger. Người đọc gặp phải sẽ hiểu sai toàn bộ mô hình quản lý trigger.
- Khuyến nghị: Viết lại ghi chú: "Trigger do Admin Hệ thống khai báo và quản lý trực tiếp trên UI (trigger, tham số đầu ra, điều kiện lọc phân khúc); QTV Marketing chỉ tra cứu (đọc). Riêng thay đổi payload ở tầng deployment vẫn kích hoạt policy PARAM_INVALID (xem Khối 3)."

---

## MAJOR — Ảnh hưởng đáng kể, nên sửa

**[MA-01]** Function Tree II.2 (Khối 3) còn cấu trúc và tên cũ — thiếu chức năng Admin và điều kiện lọc, còn dấu vết mô hình 3-group đã xóa
- Vị trí: II.2 khối cây, dòng 453–455:
  - Dòng 453: "Khối 3: **Tra cứu** Sự kiện kích hoạt" (heading UC ở dòng 1227 là "**Quản lý** Sự kiện kích hoạt"; phần diễn giải dòng 496 cũng là "Quản lý")
  - Dòng 454: "Xem danh sách sự kiện kích hoạt **(nhóm theo loại)**" — cụm "(nhóm theo loại)" là mô hình 3 collapsible group Realtime/Near Realtime/Offline đã bị bỏ ở V3.13 (chuyển sang bảng phẳng + filter chip)
  - Dòng 454–455: cây chỉ liệt kê 2 chức năng lá (Xem danh sách, Xem chi tiết); thiếu "Khai báo trigger mới (Admin)", "Thêm/Xóa tham số đầu ra (Admin)", "Thêm/Xóa điều kiện lọc phân khúc (Admin)"
- Vấn đề: Function Tree là gốc traceability (Business Function → Permission → Use Case → Giao diện). Cây thiếu 3 chức năng Admin trong khi UC-TRG-03/04/05 và Screen Admin đã đặc tả đầy đủ → đứt mạch traceability, khó rà soát đủ UC. Phần diễn giải (dòng 499) tuy đã cập nhật "Khai báo trigger mới (Admin), Thêm/xóa tham số đầu ra (Admin)" nhưng vẫn thiếu UC-TRG-05 (điều kiện lọc) và không khớp với phần cây phía trên.
- Khuyến nghị: Đồng bộ Khối 3 trong cây II.2: đổi "Tra cứu" → "Quản lý"; bỏ "(nhóm theo loại)"; bổ sung các lá "Khai báo trigger mới (Admin)", "Thêm/Xóa tham số đầu ra (Admin)", "Thêm/Xóa điều kiện lọc phân khúc (Admin)". Cập nhật câu diễn giải dòng 499 thêm chức năng điều kiện lọc phân khúc để khớp với cây.

**[MA-02]** Permission Matrix II.3 còn dòng phantom "Bật / Tắt sự kiện kích hoạt" — chức năng không tồn tại trong bất kỳ UC hay Function Tree nào
- Vị trí: II.3, dòng "Bật / Tắt sự kiện kích hoạt | Admin HT = X | QTV = –" (dòng ~546)
- Vấn đề: Use case Bật/Tắt trigger (UC-TRG-04 cũ) đã bị xóa từ V3.0. Hiện không có UC nào, không có node nào trong Function Tree, không có nút UI nào cho "Bật/Tắt trigger". Dòng này là quyền treo (dead permission) — Dev có thể hiểu nhầm cần build chức năng bật/tắt trigger. Ngoài ra ID "UC-TRG-04" hiện đã được tái sử dụng cho "Thêm/Xóa tham số đầu ra", càng dễ gây nhầm.
- Khuyến nghị: Xóa dòng "Bật / Tắt sự kiện kích hoạt" khỏi II.3. Thay bằng dòng phản ánh đúng chức năng hiện tại, ví dụ "Khai báo trigger / Quản lý tham số + điều kiện lọc | Admin HT = X | QTV = –" (đồng bộ với cách tách ở CR-01).

**[MA-03]** Ba ma trận (II.2/II.3/II.4) hoàn toàn không phản ánh chức năng "Quản lý điều kiện lọc phân khúc" (UC-TRG-05) — chức năng lõi của V3.22
- Vị trí: II.2 (thiếu node), II.3 (thiếu dòng), II.4 (không tách quyền cho filter field)
- Vấn đề: V3.22 thêm hẳn một use case mới (UC-TRG-05) và một policy mới (FILTER_INVALID) xoay quanh điều kiện lọc phân khúc, nhưng cả 3 ma trận cấu trúc/phân quyền đều không nhắc đến nó. Đây là hệ quả gộp của CR-01, CR-02, MA-01, MA-02 — nêu riêng để đảm bảo khi patch không bỏ sót góc "điều kiện lọc" (dễ chỉ sửa phần "tham số" mà quên "điều kiện lọc").
- Khuyến nghị: Khi sửa CR-01/CR-02/MA-01/MA-02, đảm bảo mỗi ma trận đều thể hiện rõ quyền Admin trên điều kiện lọc phân khúc và quyền QTV chỉ đọc điều kiện lọc. Kiểm tra chéo lại 4 mắt xích: Function Tree có node → Permission có dòng → RBAC có quyền → UC-TRG-05 + Screen 5B/T-DETAIL có màn.

---

## MINOR — Cải thiện chất lượng

**[MI-01]** Value-input rendering chưa mô tả cho 2/8 kiểu dữ liệu: "Đúng-Sai" (boolean) và "Ngày giờ" (datetime)
- Vị trí: UC-CAM-02 Quy tắc nghiệp vụ "Toán tử và giá trị điều kiện lọc" (dòng ~1056); Screen 3 Section 3 STT 5 (dòng ~1903) — cả hai chỉ liệt kê: "Danh mục → dropdown; Số/Chuỗi → nhập tay; Ngày → date picker; BETWEEN → 2 ô; IS NULL/IS NOT NULL → không ô"
- Vấn đề: Admin khai báo được 8 kiểu (Danh mục / Chuỗi / Số nguyên / Số thập phân / Số thực / Đúng-Sai / Ngày / Ngày giờ — xem UC-TRG-05 bước 3, T-FILTER, N6a) nhưng quy tắc render ô giá trị bên Campaign Builder chỉ bao 4–5 kiểu. Với kiểu "Đúng-Sai" QTV nhập giá trị bằng gì (toggle? dropdown Có/Không?) và "Ngày giờ" dùng datetime picker hay date picker — chưa mô tả. Dev phải tự quyết; Tester thiếu cơ sở test đủ 8 kiểu.
- Khuyến nghị: Bổ sung vào cả 2 vị trí: "kiểu Đúng-Sai → dropdown/toggle Có-Không (True/False); kiểu Ngày giờ → datetime picker (ngày + giờ)". Đảm bảo phủ đủ 8 kiểu đã khai báo.

**[MI-02]** Permission Matrix ký hiệu `(X)` cho "QTV xem danh sách trigger" mang nghĩa hơi lệch
- Vị trí: II.3, dòng "Xem danh sách sự kiện kích hoạt | Admin HT = X | QTV = (X)" (dòng ~544)
- Vấn đề: Theo quy ước dòng 527, `(X)` = "Được xem/tổng hợp toàn hệ thống (read-only)" — vốn dùng cho trường hợp Admin xem tổng hợp dữ liệu vận hành. Với QTV xem danh sách trigger, đây là thao tác tra cứu bình thường (read), không phải "tổng hợp toàn hệ thống". Dùng `(X)` ở đây làm nhòe ngữ nghĩa ký hiệu. (Đánh MINOR vì không gây sai chức năng, chỉ lệch quy ước.)
- Khuyến nghị: Cân nhắc đổi QTV thành `X` cho "Xem danh sách" và "Xem chi tiết" trigger (thao tác đọc thuần), giữ `(X)` đúng nghĩa "tổng hợp toàn hệ thống"; hoặc bổ sung chú thích làm rõ vì sao dùng `(X)` cho trigger.

---

## Tóm tắt

- CRITICAL: 3 issues (CR-01, CR-02, CR-03)
- MAJOR: 3 issues (MA-01, MA-02, MA-03)
- MINOR: 2 issues (MI-01, MI-02)

**Đánh giá theo nhóm trọng tâm:**

- **Nhóm 1 — Nhất quán cơ chế "điều kiện lọc phân khúc theo trigger":** PASS (chỉ 1 MINOR về render 2 kiểu dữ liệu). Toán tử khai báo thẳng, 8 kiểu dữ liệu, render giá trị (enum/nhập tay/BETWEEN/IS NULL), group-theo-trigger, badge trigger nguồn, và techName tự sinh (Admin không nhập) đều mô tả nhất quán xuyên UC-TRG-02/03/05, UC-CAM-02, Screen 3 S3 STT 5, Screen 5B Nhóm C, Screen T-DETAIL/T-NEW, và I.3. Vai trò params (chèn nội dung) vs filter field (lọc audience) được phân biệt rõ ở I.3.5b, UC-TRG-04 vs UC-TRG-05, và N6 ghi chú.
- **Nhóm 2 — Policy FILTER_INVALID:** PASS. Song song hoàn toàn với PARAM_INVALID về điều kiện áp cờ (2 điều kiện đồng thời, thêm mới không kích hoạt), xử lý theo trạng thái (Active/Pending → Paused + cờ; Draft/Paused/Ended → chỉ gắn cờ), và cách resume (khóa [Bật], chỉ qua [Sửa]→Draft→duyệt lại). Được liệt kê ĐẦY ĐỦ ở mọi nơi cần: UC-CAM-03 (banner), UC-CAM-07 (tiền điều kiện + tooltip [Bật]), UC-CAM-08 (blocking issue [Gửi duyệt]), Screen 2 STT 8, Screen 2B STT 3, Screen 3 Header Fixed STT 6+7. Không phát hiện chỗ nào tham chiếu PARAM_INVALID mà quên FILTER_INVALID.
- **Nhóm 3 — Traceability nội bộ:** FAIL (6 issue: 3 CRITICAL + 3 MAJOR). Nhóm C điều kiện lọc và cả các chức năng Admin (khai báo trigger, quản lý tham số) KHÔNG được phản ánh xuống II.2 Function Tree, II.3 Permission Matrix, II.4 RBAC Matrix. Ba ma trận này còn giữ mô hình cũ "trigger read-only / quản lý qua deployment" (V3.0), thậm chí phủ nhận quyền QTV xem chi tiết (CR-01) và giới hạn Admin chỉ VIEW (CR-02) — mâu thuẫn trực tiếp với tầng UC và Screen vốn đã đúng.

**Đánh giá tổng thể: Cần sửa trước.** Tầng đặc tả chi tiết (UC + Screen + thuật ngữ + policy) của V3.22 rất chắc và nhất quán. Nhưng tầng ma trận tổng thể (II.2/II.3/II.4) chưa được đồng bộ theo — còn xung đột nội bộ ở mức CRITICAL. Cần quay lại bước [7] urd-srs-agent, chỉ sửa 3 section II.2/II.3/II.4 (không đụng phần UC/Screen đã đạt), sau đó chạy lại review để đóng CRITICAL trước khi sang [9] postcheck.

---
*Review bởi ba-qa-agent · v3.22 · 13/07/2026*
