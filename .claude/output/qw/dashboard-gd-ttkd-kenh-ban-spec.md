# Đặc tả chỉ số — Dashboard GĐ TTKD · Phần "Hiệu quả kênh bán"

> Dự án: **QW** · Màn: Kênh bán (GĐ Trung tâm Kinh doanh)
> Nguồn đối chiếu: giao diện mẫu `dashboard_gd_ttkd (16).html` + bảng data inventory + dashboard thực tế
> Phạm vi bảng này: chỉ phần **Mục 3 — Hiệu quả kênh bán**

---

## Bảng đặc tả chỉ số

| STT | Màn | Tên chỉ số | Loại biểu đồ | Công thức tính / Cách xác định / Mô tả | Lưu ý khi hiển thị | Đơn vị tính |
|---|---|---|---|---|---|---|
| 17 | Hiệu quả kênh bán | Điểm bán phát sinh doanh thu | KPI Card | Giá trị hiển thị dạng `X/Y`:<br>+ **X** = Số điểm bán có phát sinh doanh thu trong kỳ<br>+ **Y** = Tổng số điểm bán trong phạm vi đang xem<br>+ **% có DT** = X / Y × 100% | Delta ghi `% số điểm bán có DT`<br>Logic mũi tên đánh giá theo **% có DT**:<br>- % có DT ≥ ngưỡng đạt (mặc định 85%) → mũi tên lên, màu xanh<br>- % có DT < ngưỡng đạt → mũi tên xuống, màu đỏ<br>Phạm vi thay đổi theo bộ lọc đơn vị (TCT / Tỉnh / BĐX) | Điểm bán |
| 18 | Hiệu quả kênh bán | %KH giao toàn kênh | KPI Card | Bình quân tỷ lệ hoàn thành kế hoạch giao của toàn bộ điểm bán:<br>**%KH giao** = Tổng DT thực hiện toàn kênh / Tổng KH giao toàn kênh × 100% | Delta ghi `Bình quân toàn đơn vị`<br>Logic mũi tên đánh giá theo **%KH giao**:<br>- %KH giao ≥ 100% → mũi tên lên, màu xanh (đạt/vượt kế hoạch)<br>- 90% ≤ %KH giao < 100% → màu vàng (gần đạt)<br>- %KH giao < 90% → mũi tên xuống, màu đỏ (chưa đạt) | % |
| 19 | Hiệu quả kênh bán | Người bán phát sinh doanh thu | KPI Card | Giá trị dạng `X/Y`:<br>+ **X** = Số người bán có phát sinh DT trong tháng<br>+ **Y** = Tổng số người bán trong phạm vi đang xem<br>+ Xác định người bán PSDT: từ hệ thống Cas-report theo user người bán / mã HRM<br>+ **% có DT** = X / Y × 100% | Delta ghi `Trong tháng`<br>Logic mũi tên đánh giá theo **% có DT**:<br>- % có DT > 85% → mũi tên lên, màu xanh<br>- % có DT ≤ 85% → mũi tên xuống, màu đỏ | Người |
| 20 | Hiệu quả kênh bán | Khách mới qua kênh bán | KPI Card | + **Tổng KH mới** = Số khách hàng lần đầu phát sinh giao dịch qua kênh bán trong kỳ<br>+ **Trong đó KHHH** = Số khách hàng hiện hữu (đã có trong hệ thống) phát sinh qua kênh bán | Delta ghi `Trong đó X KHHH`<br>Logic mũi tên đánh giá theo **biến động so với kỳ trước** (`% = (KH mới kỳ này - KH mới kỳ trước) / KH mới kỳ trước × 100%`):<br>- % > 0 → mũi tên lên, màu xanh<br>- % ≤ 0 → mũi tên xuống, màu đỏ, lấy giá trị tuyệt đối | Khách hàng |
| 21 | Hiệu quả kênh bán | Top điểm bán PSDT | Bar Chart (thanh ngang, có track %) | **Thanh (bar)**: mỗi đơn vị 1 thanh, giá trị = `Số điểm bán PSDT / Tổng số điểm bán` của đơn vị, kèm % lấp đầy<br>**Cách xác định đơn vị xếp hạng**: xem TCT → xếp theo **Tỉnh**; xem 1 Tỉnh → xếp theo **BĐX**<br>Sắp xếp theo số điểm bán PSDT **từ cao xuống thấp** | Có tab **Top/Bottom** + ô nhập **số lượng N** (mặc định 5)<br>Click thanh → drill xuống đơn vị tương ứng | Điểm bán |
| 22 | Hiệu quả kênh bán | Bottom điểm bán PSDT | Bar Chart (thanh ngang, có track %) | Cùng công thức STT 21, sắp xếp số điểm bán PSDT **từ thấp lên cao** | Chung khối biểu đồ với STT 21, chuyển qua tab **Bottom**<br>Click thanh → drill xuống đơn vị | Điểm bán |
| 23 | Hiệu quả kênh bán | Điểm bán phát sinh doanh thu (chi tiết) | Combo Chart (Bar + Line) | + **Column (bar)** = DT tháng thực hiện của từng điểm bán trong BĐX<br>+ **Line (nét đứt)** = KH giao của từng điểm bán<br>+ **% đạt KH** = DT tháng / KH giao × 100% (quyết định màu thanh)<br>+ **Trục X** = tên điểm bán · **Trục Y** = DT (triệu đồng) | **Chỉ hiển thị khi drill xuống 1 BĐX cụ thể** (thay biểu đồ Top/Bottom ở STT 21-22)<br>Màu thanh theo % đạt KH: ≥100% xanh, 70–99% vàng, <70% đỏ<br>Click thanh → drill điểm bán | Triệu đồng (lấy 2 số sau dấu phẩy) |
| 24 | Hiệu quả kênh bán | Hoạt động bán hàng | Funnel (Bar ngang 4 tầng) | Phễu 4 bước theo thứ tự thu hẹp dần:<br>**1. Tổng nhân sự** = Số nhân sự tham gia hoạt động bán trong phạm vi<br>**2. Hoạt động bán hàng** = Tổng số lượt hoạt động bán ghi nhận<br>**3. Cơ hội chốt thắng** = Số cơ hội có khả năng chốt<br>**4. Chốt thắng thành công** = Số cơ hội chốt thành công (phát sinh DT) | Ghi chú nguồn: hoạt động ghi nhận qua **CRM** (NVKD gắn điểm phục vụ/điểm bán + Bán hàng chuyên trách không gắn điểm bán); GĐ xã và nhân sự khác trên **HRM**<br>Mỗi tầng 1 màu, hover hiện giá trị | Số lượng |
| 25 | Hiệu quả kênh bán | Người bán PSDT (tổng hợp) | KPI Card | Giá trị `X/Y`:<br>+ **X** = Số người bán có DT · **Y** = Tổng người bán trong phạm vi<br>+ **% người bán có DT** = X / Y × 100% | Delta ghi `% người bán có DT`<br>Logic mũi tên đánh giá theo **% người bán có DT**:<br>- % > 85% → mũi tên lên, màu xanh<br>- % ≤ 85% → mũi tên xuống, màu đỏ<br>Tag hiển thị cấp đang xem: Toàn TCT / `Tỉnh · Tất cả BĐX` / `BĐX · [tên]` | Người |
| 26 | Hiệu quả kênh bán | %KH giao bình quân (người bán) | KPI Card | **%KH giao bình quân** = Trung bình cộng % hoàn thành KH giao của tất cả người bán trong phạm vi | Delta ghi `Trung bình toàn bộ người bán`<br>Logic mũi tên đánh giá theo **%KH giao bình quân**:<br>- ≥ 90% → mũi tên lên, màu xanh<br>- 70% ≤ % < 90% → màu vàng<br>- < 70% → mũi tên xuống, màu đỏ | % |
| 27 | Hiệu quả kênh bán | % Chuyển đổi bình quân (người bán) | KPI Card | **% Chuyển đổi bình quân** = Trung bình cộng tỷ lệ (Cơ hội PSDT / Hoạt động bán) của các người bán<br>Phản ánh hiệu suất biến hoạt động thành cơ hội có DT | Delta ghi `Cơ hội PSDT / hoạt động bán`<br>Logic mũi tên đánh giá theo **% chuyển đổi bình quân**:<br>- ≥ 30% → mũi tên lên, màu xanh<br>- 20% ≤ % < 30% → màu vàng<br>- < 20% → mũi tên xuống, màu đỏ | % |
| 28 | Hiệu quả kênh bán | Danh sách người bán chi tiết | Table + Bar Chart (trong modal) | Danh sách từng người bán, mỗi dòng gồm:<br>+ **%KH giao** = DT thực hiện / KH giao của người bán × 100%<br>+ **% Chuyển đổi** = Cơ hội PSDT / Hoạt động bán × 100%<br>+ Bar Chart trong modal: 2 nhóm thanh (%KH giao · % Chuyển đổi) theo từng người bán | Mở qua nút **"Xem danh sách người bán →"**<br>Màu thanh %KH giao theo ngưỡng: ≥100% xanh, 70–99% vàng, <70% đỏ<br>Click dòng/thanh → drill người bán | Người / % |

---

## Ghi chú & Assumptions

**[A1]** Top và Bottom điểm bán PSDT được tách thành **2 dòng riêng** (STT 21, 22) theo yêu cầu, dù trên UI dùng chung 1 khối biểu đồ với tab Top/Bottom.

**[A2]** Đơn vị doanh thu chuẩn hóa là **Triệu đồng, lấy 2 số sau dấu phẩy** (đồng bộ với bảng data inventory). Trên UI hiển thị rút gọn dạng "312tr".

**[A3]** Hai chỉ số nền **"Tổng điểm bán"** và **"Tổng LLBH"** (mẫu số trong X/Y của STT 17, 19) **chưa tách thành dòng riêng** — chỉ đóng vai trò thành phần Y. Bổ sung sau nếu cần hiển thị độc lập.

**[A4]** Các ngưỡng màu (90% / 85% / 30% / 20%...) lấy từ logic hiển thị của giao diện mẫu. Cần PO/team xác nhận lại ngưỡng nghiệp vụ chính thức nếu khác.

**[A5]** Logic mũi tên tăng/giảm — hai kiểu đánh giá:
- **Kiểu ngưỡng đạt (dùng cho phần lớn KPI Kênh bán):** so giá trị hiện tại với ngưỡng nghiệp vụ (85% / 90% / 100% / 30% / 20%...) → đạt: mũi tên lên, xanh; chưa đạt: mũi tên xuống, đỏ; vùng giữa (nếu có): màu vàng. Áp dụng cho STT 17, 18, 19, 25, 26, 27.
- **Kiểu biến động so kỳ (giống bảng doanh thu):** `% = (giá trị kỳ này − giá trị kỳ trước) / giá trị kỳ trước × 100%` → % > 0: mũi tên lên, xanh; % ≤ 0: mũi tên xuống, đỏ, lấy giá trị tuyệt đối. Áp dụng cho STT 20 (Khách mới qua kênh bán).

## Điểm cần xác nhận (Open Questions)

- [ ] OQ-1: Ngưỡng màu KPI (90%/85%/30%/20%) có đúng chuẩn nghiệp vụ QW không, hay chỉ là giá trị demo của prototype?
- [ ] OQ-2: "Khách mới qua kênh bán" (STT 20) — kỳ tính là tháng hay khoảng thời gian theo bộ lọc? Và mũi tên so với kỳ trước tính theo tháng trước hay cùng kỳ năm trước?
- [ ] OQ-3: Có cần hiển thị 2 chỉ số nền "Tổng điểm bán" / "Tổng LLBH" thành card độc lập không? (liên quan [A3])
- [ ] OQ-4: Các KPI dạng ngưỡng đạt (STT 17, 19, 25...) — mũi tên nên đánh giá theo **ngưỡng đạt** (như đang đặc tả) hay theo **biến động so kỳ trước** như chỉ số doanh thu? Cần PO thống nhất quy ước chung.
