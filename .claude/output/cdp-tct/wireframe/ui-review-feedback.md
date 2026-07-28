# UI Review Feedback — prototype-v2.html
**Ngày:** 29/06/2026
**Nguồn:** ui-feedback-agent (tự động)

---

## CRITICAL

**F-01 — Tab "Lịch sử gộp hồ sơ" trùng nội dung với tab "Hồ sơ liên kết"**
Tab "Hồ sơ liên kết" có section "Lịch sử merge hồ sơ" hiển thị cùng dữ liệu.
Đề xuất: xóa bảng merge history khỏi "Hồ sơ liên kết", chỉ giữ Alias IDs + nút báo cáo.
⚠️ **Cần BA confirm** trước khi sửa — có thể là thiết kế có chủ đích cho Data Steward.

**F-02 — Cross-reference trong tab "Nhật ký" là text tĩnh, không click được**
Đề xuất: đổi thành button chuyển sang tab `mergehist` với `setTab('mergehist')`.

**F-03 — Mũi tên expand/collapse accordion quá nhỏ và màu xám nhạt**
Đề xuất: tăng kích thước + tương phản, thêm text "Xem chi tiết" / "Thu gọn".

---

## MAJOR

**F-04 — Card grid + bảng so sánh trong accordion hiển thị cùng 7 field — dư thừa**
Đề xuất: chỉ giữ bảng so sánh (có cột Trạng thái rõ hơn), bỏ card grid.
⚠️ **Cần confirm về IA trước** khi sửa.

**F-05 — Customer 360 có 10–11 tab ngang, không có cơ chế phân nhóm**
Đề xuất: nhóm tab (Hồ sơ / Giao dịch / Dữ liệu & Tuân thủ / Định danh & Merge) hoặc gộp 3 tab merge-related thành 1 tab "Định danh" với sub-tab.
⚠️ **Cần confirm về IA trước** khi sửa.

**F-06 — Badge "⚠️ Có sai khác" khó nhận ra trong row dày đặc**
Đề xuất: thêm `border-left: 4px solid #d97706` cho accordion card có conflict.

---

## MINOR

**F-07** — Cột Trạng thái hiện "—" khi thiếu dữ liệu — đổi thành badge "Thiếu" màu xám.

**F-08** — "Người xác nhận: SYSTEM" → đổi label thành "Xử lý bởi", dịch "SYSTEM" → "Hệ thống tự động".

**F-09** — Segment Builder dropdown dùng snake_case tiếng Anh — Việt hóa label (`churn_score` → "Điểm rời bỏ"...).

**F-10** — Badge COD status trong tab "GD & COD" vẫn là `DISCREPANCY`/`MATCHED` — không nhất quán với tab Doanh nghiệp.

---

## Thứ tự thực hiện (sau khi BA confirm)

1. Hỏi BA về F-01 và F-04, F-05 (liên quan IA)
2. Sửa React trực tiếp (không cần wireframe): F-02, F-03, F-06, F-07, F-08, F-09, F-10
3. Sửa wireframe trước, rồi React sau: F-04, F-05
4. Sau khi BA xác nhận F-01: xóa bảng merge history khỏi tab "Hồ sơ liên kết"
