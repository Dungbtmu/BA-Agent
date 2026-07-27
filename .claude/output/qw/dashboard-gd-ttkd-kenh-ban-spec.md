# Đặc tả chỉ số — Dashboard GĐ TTKD · Phần "Hiệu quả kênh bán"

> Dự án: **QW** · Màn: Kênh bán (GĐ Trung tâm Kinh doanh)
> Nguồn đối chiếu: giao diện mẫu `dashboard_gd_ttkd (16).html` + bảng data inventory (mục C · Kênh bán) + bảng đặc tả tổng của team
> Phạm vi bảng này: toàn bộ chỉ số phần **Hiệu quả kênh bán** (STT 17–23)
> Style: đồng bộ với bảng đặc tả tổng — 10 cột, gom mỗi khối UI 1 dòng, KPI/chart con liệt kê trong ô Công thức.

---

## Bảng đặc tả chỉ số

| STT | Màn | Tên chỉ số | Loại biểu đồ | Công thức tính / Cách xác định / Mô tả | Lưu ý khi hiển thị | Đơn vị tính | Dữ liệu nguồn | Tần suất lấy | Ghi chú |
|---|---|---|---|---|---|---|---|---|---|
| 17 | Hiệu quả kênh bán | Tổng điểm bán | KPI Card | Phản ánh quy mô mạng lưới điểm bán trong phạm vi đang xem.<br>**Tổng điểm bán** = Tổng số điểm bán, bao gồm: Bưu cục GD1, Bưu cục GD2, Bưu cục GD3 và Bưu điện Văn hoá xã (BĐVHX) | - Là con số quy mô nền, đóng vai trò mẫu số (Y) trong tỷ lệ "Điểm bán phát sinh doanh thu"<br>- Thay đổi theo bộ lọc đơn vị (TCT / Tỉnh / BĐX) | Điểm bán | TT ĐHKD cung cấp file điểm bán (từ điểm bán → BĐX → cấp tỉnh) | Theo file cập nhật | Nguồn dạng up file danh sách điểm bán |
| 18 | Hiệu quả kênh bán | Tổng LLBH | KPI Card | Phản ánh quy mô lực lượng bán hàng (LLBH) tại điểm bán.<br>**Tổng LLBH** = Tổng số nhân sự LLBH gắn với điểm bán trong phạm vi đang xem | - Là con số quy mô nền, đóng vai trò mẫu số (Y) trong tỷ lệ "Người bán phát sinh doanh thu"<br>- Xét LLBH gắn tại điểm bán | Người | Hệ thống HRM (danh sách LLBH gắn với điểm bán) | Theo cập nhật HRM | Chỉ tính LLBH gắn điểm bán, không gồm bán hàng chuyên trách không gắn điểm bán |
| 19 | Hiệu quả kênh bán | Chỉ tiêu kênh bán (cụm KPI Card) | KPI Card | Cụm 4 KPI Card đầu mục:<br>**1. Điểm bán phát sinh doanh thu** — hiển thị `X/Y`:<br>+ X = Số điểm bán có phát sinh doanh thu trong kỳ<br>+ Y = Tổng số điểm bán trong phạm vi đang xem (xem STT 17)<br>+ % có DT = X / Y × 100%<br>**2. %KH giao toàn kênh** = Tổng DT thực hiện toàn kênh / Tổng KH giao toàn kênh × 100%<br>**3. Người bán phát sinh doanh thu** — hiển thị `X/Y`:<br>+ X = Số người bán có phát sinh DT trong tháng<br>+ Y = Tổng số người bán trong phạm vi<br>+ Xác định người bán PSDT: từ Cas-report theo user người bán / mã HRM<br>**4. Khách mới qua kênh bán** = Số khách hàng lần đầu phát sinh giao dịch qua kênh bán trong kỳ; trong đó KHHH = số khách hàng hiện hữu phát sinh qua kênh bán | Mỗi KPI có delta và logic mũi tên tăng/giảm:<br>- Nếu % tỷ lệ > 0 (hoặc đạt ngưỡng) → mũi tên lên, màu xanh<br>- Nếu % tỷ lệ ≤ 0 (hoặc chưa đạt) → mũi tên xuống, màu đỏ, lấy giá trị tuyệt đối<br>Delta: KPI 1 ghi `% số điểm bán có DT`; KPI 3 ghi `% người bán có DT`; KPI 4 ghi `Trong đó X KHHH`<br>Phạm vi đổi theo bộ lọc đơn vị (TCT / Tỉnh / BĐX)<br>**<span style="color:red">Căn cứ công thức và các ngưỡng màu tương ứng — cần Ban chốt ngưỡng.</span>** | Điểm bán · % · Người · Khách hàng | Hệ thống Cas-report (khai thác Bảng S98 lưu trữ dưới data base) | Hàng ngày | Ngưỡng màu KPI để trống, chờ Ban chốt thống nhất với ngưỡng phần doanh thu |
| 20 | Hiệu quả kênh bán | Top điểm bán phát sinh doanh thu | Bar Chart | + **Column** = giá trị doanh thu thực hiện của từng điểm bán trong kỳ<br>+ **Trục X** = tên điểm bán · **Trục Y** = doanh thu thực hiện<br>+ Giá trị xếp hạng căn cứ doanh thu thực hiện của điểm bán và sắp xếp **từ cao xuống thấp**<br>+ Có ô nhập **số lượng N** (mặc định 5)<br>+ Khi drill xuống 1 BĐX cụ thể: chuyển sang **Combo Chart** — bar = DT tháng từng điểm bán, line nét đứt = KH giao từng điểm bán, % đạt KH = DT tháng / KH giao × 100% (quyết định màu thanh) | - Trường hợp bộ lọc TCT → xếp hạng theo Tỉnh; bộ lọc Tỉnh → xếp hạng theo BĐX; chọn 1 BĐX → chi tiết từng điểm bán<br>- Khi nhấn thanh bất kỳ → drill xuống đơn vị/điểm bán tương ứng<br>**<span style="color:red">Căn cứ công thức và các ngưỡng màu tương ứng — cần Ban chốt.</span>** | VNĐ (Triệu đồng, lấy 2 số sau dấu phẩy) | Hệ thống Cas-report (khai thác Bảng S98 lưu trữ dưới data base) | Hàng ngày | Dùng chung 1 khối biểu đồ với Bottom (STT 21), chuyển qua tab |
| 21 | Hiệu quả kênh bán | Bottom điểm bán phát sinh doanh thu | Bar Chart | + **Column** = giá trị doanh thu thực hiện của từng điểm bán trong kỳ<br>+ **Trục X** = tên điểm bán · **Trục Y** = doanh thu thực hiện<br>+ Giá trị xếp hạng căn cứ doanh thu thực hiện của điểm bán và sắp xếp **từ thấp lên cao**<br>+ Có ô nhập **số lượng N** (mặc định 5) | - Trường hợp bộ lọc TCT → xếp hạng theo Tỉnh; bộ lọc Tỉnh → xếp hạng theo BĐX; chọn 1 BĐX → chi tiết từng điểm bán<br>- Khi nhấn thanh bất kỳ → drill xuống đơn vị/điểm bán tương ứng<br>**<span style="color:red">Căn cứ công thức và các ngưỡng màu tương ứng — cần Ban chốt.</span>** | VNĐ (Triệu đồng, lấy 2 số sau dấu phẩy) | Hệ thống Cas-report (khai thác Bảng S98 lưu trữ dưới data base) | Hàng ngày | Dùng chung 1 khối biểu đồ với Top (STT 20), chuyển qua tab |
| 22 | Hiệu quả kênh bán | Hoạt động bán hàng | Funnel | Phễu 4 bước theo thứ tự thu hẹp dần: **Nhân sự → Hoạt động → Cơ hội chốt → Thành công**<br>+ **Tổng nhân sự** = Số nhân sự tham gia hoạt động bán trong phạm vi<br>+ **Hoạt động bán hàng** = Tổng số lượt hoạt động bán ghi nhận<br>+ **Cơ hội chốt thắng** = Số cơ hội có khả năng chốt<br>+ **Chốt thắng thành công** = Số cơ hội chốt thành công (phát sinh DT)<br>+ **Doanh thu chốt thắng** (chỉ số kèm) = Tổng DT kỳ vọng chốt thắng | - Mỗi tầng 1 màu; hover hiện giá trị từng tầng<br>- Ghi chú nguồn: hoạt động ghi nhận qua CRM (NVKD gắn điểm phục vụ/điểm bán + Bán hàng chuyên trách không gắn điểm bán); GĐ xã và nhân sự khác quản lý trên HRM | Số lượng (DT chốt thắng: Triệu đồng) | Hệ thống CRM (Báo cáo cơ hội / Báo cáo tổng hợp cơ hội) | Hàng ngày | Quy mô ~14.000 nhân sự tham gia hoạt động bán toàn TCT |
| 23 | Hiệu quả kênh bán | Người bán phát sinh doanh thu | Table | Danh sách nhân viên bán có doanh thu. Mỗi dòng gồm:<br>+ **%KH giao** = DT thực hiện / KH giao của người bán × 100%<br>+ **% Chuyển đổi** = Cơ hội PSDT / Hoạt động bán × 100%<br>Kèm 3 KPI tổng hợp phía trên bảng:<br>+ **Người bán PSDT** = Số người bán có DT / Tổng người bán (kèm % người bán có DT)<br>+ **%KH giao bình quân** = Trung bình cộng %KH giao của tất cả người bán trong phạm vi<br>+ **% Chuyển đổi bình quân** = Trung bình cộng (Cơ hội PSDT / Hoạt động bán) của các người bán | - Mở danh sách chi tiết qua nút **"Xem danh sách người bán →"**<br>- Bar Chart trong modal: 2 nhóm thanh (%KH giao · % Chuyển đổi) theo từng người bán<br>- Click dòng/thanh → drill người bán<br>- Tag hiển thị cấp đang xem: Toàn TCT / `Tỉnh · Tất cả BĐX` / `BĐX · [tên]`<br>**<span style="color:red">Căn cứ công thức và các ngưỡng màu tương ứng — cần Ban chốt.</span>** | Người · % | Hệ thống Cas-report (khai thác Bảng S98 lưu trữ dưới data base) | Hàng ngày | Xác định người bán PSDT theo user người bán / mã HRM trên Cas-report |

---

## Ghi chú & Assumptions

**[A1]** Bảng gồm **7 dòng STT 17–23**, phủ toàn bộ mục C · Kênh bán của bảng data inventory:
- STT 17–18: 2 chỉ số quy mô nền (Tổng điểm bán, Tổng LLBH) — đóng vai trò mẫu số cho các tỷ lệ PSDT
- STT 19: cụm KPI card tỷ lệ (Điểm bán PSDT, %KH giao, Người bán PSDT, Khách mới) — liệt kê con trong ô Công thức theo style ảnh mẫu STT 1
- STT 20–21: Top / Bottom điểm bán PSDT tách 2 dòng riêng như bảng doanh thu mẫu, dù UI dùng chung 1 khối biểu đồ có tab
- STT 22: Funnel hoạt động bán · STT 23: Table người bán PSDT

**[A2]** Đơn vị doanh thu chuẩn hóa là **Triệu đồng, lấy 2 số sau dấu phẩy** (đồng bộ bảng doanh thu). UI hiển thị rút gọn "312tr". Top/Bottom điểm bán để đơn vị **VNĐ** như bảng tổng.

**[A3]** Ngưỡng màu KPI (%KH giao, tỷ lệ đạt...) **để trống**, ghi chú đỏ "Căn cứ công thức và các ngưỡng màu tương ứng" — chờ Ban chốt để thống nhất với ngưỡng phần doanh thu (`<50% đỏ · 50%–<65% cam · >65% xanh`).

**[A4]** Logic mũi tên tăng/giảm áp dụng quy ước chung của dashboard: `% > 0 → mũi tên lên, xanh` · `% ≤ 0 → mũi tên xuống, đỏ, lấy giá trị tuyệt đối`. Với KPI dạng tỷ lệ đạt (X/Y), cần Ban xác nhận đánh giá theo **ngưỡng đạt** hay **biến động so kỳ trước** (xem OQ-4).

**[A5]** Cột "Dữ liệu nguồn" và "Tần suất lấy" điền theo bảng data inventory + giao diện mẫu. Hoạt động bán lấy từ CRM, các chỉ số DT/điểm bán/người bán lấy từ Cas-report (Bảng S98). Xác nhận lại nếu nguồn thực tế khác.

## Điểm cần xác nhận (Open Questions)

- [ ] OQ-1: Ngưỡng màu KPI phần Kênh bán — dùng chung ngưỡng doanh thu (`<50% đỏ · 50–<65% cam · >65% xanh`) hay ngưỡng riêng? (liên quan [A3])
- [ ] OQ-2: "Khách mới qua kênh bán" (KPI 4, STT 17) — kỳ tính là tháng hay khoảng thời gian theo bộ lọc? Mũi tên so kỳ trước tính theo tháng trước hay cùng kỳ năm trước?
- [ ] OQ-3: Tổng điểm bán (STT 17) và Tổng LLBH (STT 18) — hiển thị thành KPI card độc lập trên dashboard, hay chỉ là con số nền dùng làm mẫu số? (Bảng data inventory liệt kê là chỉ số nghiệp vụ độc lập)
- [ ] OQ-4: KPI dạng tỷ lệ đạt (Điểm bán PSDT, Người bán PSDT...) — mũi tên đánh giá theo **ngưỡng đạt** hay **biến động so kỳ trước** như chỉ số doanh thu?
- [ ] OQ-5: Combo Chart khi drill xuống 1 BĐX (STT 20–21) — có phải hành vi Ban mong muốn, hay chỉ hiển thị Bar Chart thuần theo doanh thu như Top/Bottom?
- [ ] OQ-6: "Doanh thu chốt thắng" trong Funnel (STT 22, dòng 20 data inventory = Tổng DT kỳ vọng chốt thắng) — hiển thị như tầng thứ 5 của phễu, hay chỉ là số liệu kèm tooltip?
