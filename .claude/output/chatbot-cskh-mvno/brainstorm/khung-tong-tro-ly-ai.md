---
type: brainstorm
feature: chatbot-cskh-mvno
idea_slug: khung-tong-tro-ly-ai
status: draft
lang: vi
owner: "Jun"
created: 2026-10-06
updated: 2026-10-06
links:
  - .claude/input/Chatbot - CSKH - MVNO/TTr nang cao giai phap CSKH (callbot_tro ly ao AI) V2.docx
  - .claude/input/Chatbot - CSKH - MVNO/Mẫu cung cấp thông tin chatbot CSKH MVNO.xlsx
tags: [brainstorm, chatbot, cskh, mvno, khung-tong]
stale_reason: ""
changelog:
  - 2026-10-06 | /brainstorm | initial board từ tờ trình V2 (Phụ lục 02, 03) + Excel mẫu (còn trống) + phỏng vấn Jun
---

# Khung Tổng Trợ lý AI (Chatbot CSKH) — Brainstorm

> Feature: chatbot-cskh-mvno | Idea: khung-tong-tro-ly-ai
>
> **Phạm vi board này:** phần khung chung của Tổng Trợ lý AI (điều phối, xác thực, kênh, nguyên tắc xử lý, chuyển hotline) và luồng giao dịch mẫu "Đăng ký gói cước". Từng lĩnh vực (Viễn thông, Tài chính và thanh toán, ...) sẽ có board riêng, bắt đầu từ Viễn thông [A1].

## 1. Idea Seed

Trung tâm VTDĐ (viết tắt MVNO) muốn xây dựng **hệ thống Trợ lý AI đa lĩnh vực (Multi-Agent) trên Super App**. Trong đó Tổng Trợ lý AI là đầu mối tiếp nhận, hiểu và điều phối mọi yêu cầu của khách hàng sang các Trợ lý chuyên trách theo lĩnh vực.

Tờ trình V2 (Phụ lục 02) liệt kê 7 nhóm lĩnh vực: Viễn thông; Cá nhân và công việc; Tài chính và thanh toán; Giải trí; Sức khỏe và đời sống cá nhân; Du lịch và tiện ích đời sống; Khác (tra cứu pháp luật, thủ tục hành chính công, hỏi đáp thường ngày).

## 2. Context

**Nguồn input:**
- Tờ trình V2: chỉ có Phụ lục 02 và Phụ lục 03 (benchmark VietinBank, TPBank, MB, Viettel, VNPT, Vietnam Airlines, Vietjet; ghi chú là thông tin công khai, chưa được các đơn vị xác nhận). Không có thân tờ trình.
- Excel mẫu thu thập thông tin: 8 sheet, gần như trống. Các ví dụ trong file (định vị bưu gửi, phí vận chuyển, CAGR) là ví dụ mẫu, không phải dữ liệu của dự án.

**Mục tiêu (theo Jun, chưa có số liệu):**
1. Giảm nhân sự CSKH.
2. Tăng sự chủ động của khách hàng.
3. Giảm tần suất khách gọi lại trung tâm.
4. Thể hiện việc ứng dụng AI của công ty trong phát triển sản phẩm.

**Thứ tự ưu tiên:** nhóm vận hành (mục 1, 2, 3) ưu tiên hơn nhóm hình ảnh (mục 4).

**Tình trạng:** chưa có kế hoạch triển khai cụ thể, chưa có số liệu hiện tại (baseline), chưa biết số lượng người dùng, chưa biết tên hệ thống nghiệp vụ, chưa rõ đơn vị nào quản lý Super App. Bot là dự án mới, cần tích hợp vào Super App để lấy thông tin và sử dụng dữ liệu.

## 3. User Types (preliminary)

| Role | Mô tả | Truy cập | Trạng thái xác nhận |
|---|---|---|---|
| Thuê bao đã đăng nhập | Khách chính, dùng tra cứu cá nhân, giao dịch | Đăng nhập Super App; giao dịch có tiền cần thêm OTP gửi về số thuê bao [A3] | Giả định, chờ xác nhận |
| Khách chưa là thuê bao | Hỏi gói cước, chính sách chung | Không cần đăng nhập, chỉ dùng nội dung chung (FAQ, tư vấn gói) | Chờ xác nhận (OQ-12) |
| Quản trị nội dung bot | Người của trung tâm nạp và duy trì tri thức: FAQ, tài liệu sản phẩm, thuật ngữ; xem các câu bot trả lời sai hoặc không trả lời được | Cổng quản trị riêng | Chờ xác nhận (OQ-10) |
| Nhân viên CSKH | Xử lý cuộc gọi hotline | **Không dùng giao diện của bot** vì giai đoạn 1 chỉ đưa hotline, không chuyển chat sang người [A9] | Giả định |

**Entry point:** nút chatbot trong Super App. Không có callbot ở giai đoạn 1 [A2].

**Số lượng người dùng:** chưa biết (OQ-6).

## 4. Capabilities Breakdown

> Phân loại P0/P1/P2 dưới đây là **đề xuất của BA** dựa trên nguyên tắc "nhóm vận hành ưu tiên", chưa được trung tâm xác nhận (OQ-7).

### P0 — must have (khung chung)

- Tiếp nhận yêu cầu bằng chữ, ảnh, giọng nói (giọng nói qua ô chat, chuyển thành văn bản) [A2].
- Hiểu ý định, hỏi lại khi mơ hồ (tối đa 2 lần), điều phối sang Trợ lý đúng lĩnh vực [A5].
- Xác thực khi cần (OTP cho giao dịch có tiền) [A3].
- Bước tóm tắt và xác nhận bắt buộc trước mọi giao dịch (nguyên tắc cứng).
- Báo "chưa hỗ trợ" kèm thông tin hotline khi không xử lý được.
- Thông báo kết quả giao dịch qua chat, SMS và thông báo trong app.
- Trả lời lịch sự, ngắn gọn, đúng trọng tâm, xưng hô "tôi - bạn".

### P0 — lĩnh vực Viễn thông (đề xuất, sẽ đào sâu ở board riêng)

- Tra cứu thuê bao, gói cước, data, cước đang dùng.
- Đăng ký, gia hạn gói cước di động.
- Tư vấn, gợi ý gói phù hợp.
- Hướng dẫn xử lý sự cố di động thường gặp.
- Nạp thẻ điện thoại.

### P0 — lĩnh vực Tài chính và thanh toán (đề xuất)

- Thanh toán hóa đơn cước di động.

### P1 — should have

- Hủy dịch vụ, quản lý dịch vụ đang dùng.
- Nhắc lịch gia hạn gói và thanh toán hóa đơn.
- Tra cứu giao dịch.
- Hỏi đáp thường ngày dựa trên FAQ và tài liệu.
- Cổng quản trị nội dung bot ở mức tối thiểu (nạp FAQ, tài liệu, xem câu trả lời sai).

### P2 — nice to have

- Cá nhân và công việc (quản lý lịch, tóm tắt cuộc họp, soạn email, tạo slide, phân tích tài liệu).
- Tài chính nâng cao (phân tích chi tiêu, cảnh báo giao dịch bất thường, sản phẩm tài chính, tra số dư tài khoản liên kết).
- Giải trí, Sức khỏe và đời sống, Du lịch và tiện ích đời sống.
- Tra cứu pháp luật, thủ tục hành chính công.

> **Lưu ý:** Jun nói "có thể cần cả 7 lĩnh vực". Phân loại P2 ở trên chỉ là thứ tự ưu tiên triển khai, không loại bỏ phạm vi.

## 5. Core Flows

### 5.1 Luồng chung của Tổng Trợ lý AI

1. Khách mở chatbot trong Super App, gõ chữ, gửi ảnh hoặc nói.
2. Hệ thống đã biết khách là thuê bao nào nhờ khách đang đăng nhập Super App.
3. Tổng Trợ lý AI hiểu ý định và chuyển cho Trợ lý đúng lĩnh vực.
4. Nếu yêu cầu cần dữ liệu cá nhân (cước, gói đang dùng), hệ thống truy vấn hệ thống nghiệp vụ.
5. Bot trả lời ngắn gọn, đúng trọng tâm, kèm gợi ý hành động tiếp theo.
6. Nếu yêu cầu tạo thay đổi (đăng ký gói, nạp thẻ), bot hiển thị tóm tắt và khách xác nhận rồi mới thực hiện.
7. Bot báo kết quả và gửi thông báo qua Super App.

**Khi bot không xử lý được:** bot báo chưa hỗ trợ và đưa thông tin hotline, khách tự gọi để được giải quyết. Lối thoát nhẹ (ghi lại yêu cầu để nhân viên gọi lại) đang chờ quyết định (OQ-1).

### 5.2 Luồng mẫu "Đăng ký gói cước"

**Các bước:**
1. Khách nhắn "Đăng ký gói X cho tôi".
2. Bot hiểu ý định. Nếu mơ hồ thì hỏi lại, tối đa 2 lần; vẫn mơ hồ thì báo chưa hỗ trợ và đưa hotline.
3. Bot kiểm tra điều kiện: gói tồn tại, thuê bao đủ điều kiện, đủ tiền. Không đủ điều kiện thì báo lý do và gợi ý.
4. Bot hiển thị tóm tắt: tên gói, giá, thời hạn, số thuê bao áp dụng.
5. Khách bấm "Đồng ý" (không đồng ý thì hủy).
6. Hệ thống gửi OTP về số thuê bao; khách nhập OTP.
7. OTP đúng thì hệ thống gọi hệ thống nghiệp vụ để đăng ký gói; OTP sai tối đa 3 lần thì khóa giao dịch và đưa hotline.
8. Thành công: bot báo kết quả trong chat và gửi SMS xác nhận. Lỗi hoặc không phản hồi: bot báo chưa thực hiện được, không trừ tiền, đưa hotline.

**Sơ đồ:**

```
 KHÁCH                         BOT / HỆ THỐNG
   │
   │ "Đăng ký gói X cho tôi"
   ├──────────────────────────▶ Hiểu ý định
   │                              │
   │                       ┌──────┴───────┐
   │                  Rõ ý định?       Mơ hồ
   │                       │              │
   │                       │      Hỏi lại (tối đa 2 lần)
   │◀──────────────────────┼──────────────┤
   │                       │         Vẫn mơ hồ ──▶ Báo không xử lý được
   │                       │                       + đưa hotline  [KẾT THÚC]
   │                       ▼
   │                 Kiểm tra điều kiện (gói tồn tại,
   │                 thuê bao đủ điều kiện, đủ tiền)
   │                       │
   │                ┌──────┴───────┐
   │             Đủ ĐK          Không đủ ĐK ──▶ Báo lý do + gợi ý
   │                │                            [KẾT THÚC]
   │                ▼
   │◀──── Hiển thị tóm tắt: gói, giá, thời hạn, số áp dụng
   │
   │ Bấm "Đồng ý"  (không đồng ý ──▶ Hủy [KẾT THÚC])
   ├──────────────────────────▶ Gửi OTP về số thuê bao
   │◀────── Nhập OTP
   ├──────────────────────────▶ OTP đúng?
   │                       ┌──────┴────────┐
   │                    Đúng          Sai (tối đa 3 lần) ──▶ Khóa giao dịch
   │                      │                                  + đưa hotline
   │                      ▼
   │                 Gọi hệ thống nghiệp vụ đăng ký gói
   │                 [dịch vụ ngoài]
   │                      │
   │               ┌──────┴────────┐
   │           Thành công       Lỗi / không phản hồi
   │               │                │
   │◀── Báo thành công      ◀── Báo chưa thực hiện được,
   │    + SMS xác nhận           không trừ tiền, đưa hotline
```

**Áp dụng cho các giao dịch khác:** nạp thẻ, gia hạn, hủy gói, thanh toán hóa đơn theo cùng khuôn (tóm tắt, xác nhận, OTP, thực hiện, báo kết quả). Chi tiết từng giao dịch viết ở board lĩnh vực.

## 6. System Behavior Deep Dive

### 6.1 Decision Points

| ID | Flow | Khi nào | YES | NO |
|---|---|---|---|---|
| D1 | Chung | Bot hiểu rõ ý định khách? | Chuyển sang Trợ lý chuyên trách | Hỏi lại (tối đa 2 lần), vẫn mơ hồ thì báo chưa hỗ trợ và đưa hotline |
| D2 | Chung | Yêu cầu là tra cứu hay giao dịch? | Giao dịch: sang D4. | Tra cứu: trả lời ngay (chỉ cần đã đăng nhập) |
| D3 | Chung | Yêu cầu cần dữ liệu cá nhân mà khách chưa đăng nhập? | Mời khách đăng nhập (OQ-12) | Trả lời bằng nội dung chung |
| D4 | Giao dịch | Thuê bao và gói đủ điều kiện thực hiện? | Sang D5 | Báo lý do không đủ điều kiện, gợi ý lựa chọn khác |
| D5 | Giao dịch | Khách bấm "Đồng ý" trên màn hình tóm tắt trong 5 phút? | Gửi OTP | Hủy, không phát sinh giao dịch |
| D6 | Giao dịch | OTP đúng và còn hiệu lực (3 phút)? | Gọi hệ thống nghiệp vụ | Cho nhập lại (tối đa 3 lần), quá 3 lần thì khóa giao dịch và đưa hotline |
| D7 | Giao dịch | Hệ thống nghiệp vụ trả kết quả thành công? | Báo thành công, gửi SMS | Báo chưa thực hiện được, không trừ tiền, đưa hotline |

### 6.2 Scenario matrix

| Đối tượng | Loại yêu cầu | Quy tắc | Hành động | Kết quả |
|---|---|---|---|---|
| Thuê bao đã đăng nhập | Tra cứu (cước, gói đang dùng) | Chỉ cần đăng nhập | Truy vấn hệ thống nghiệp vụ | Bot trả lời ngay |
| Thuê bao đã đăng nhập | Giao dịch có tiền | Tóm tắt, xác nhận, OTP | Thực hiện sau khi xác thực | Báo kết quả, SMS xác nhận |
| Thuê bao đã đăng nhập | Ngoài khả năng bot | Không xử lý được | Báo chưa hỗ trợ | Đưa hotline |
| Khách chưa là thuê bao | FAQ, tư vấn gói | Nội dung chung | Trả lời từ tri thức | Bot trả lời (OQ-12) |
| Khách chưa là thuê bao | Cần dữ liệu cá nhân hoặc giao dịch | Cần là thuê bao | Mời đăng nhập hoặc đăng ký | Chờ xác nhận (OQ-12) |
| Quản trị nội dung bot | Nạp FAQ, tài liệu | Chỉ role quản trị | Cập nhật tri thức | Bot dùng nội dung mới (OQ-10) |

### 6.3 State transitions

**Giao dịch (ví dụ đăng ký gói) [A6]:**

| Entity | Từ | Sang | Trigger | Quay lại? |
|---|---|---|---|---|
| Giao dịch | Chờ xác nhận | Đang xử lý | Khách đồng ý và OTP đúng | Không |
| Giao dịch | Đang xử lý | Thành công | Hệ thống nghiệp vụ xác nhận | Không |
| Giao dịch | Đang xử lý | Thất bại | Hệ thống nghiệp vụ từ chối, lỗi hoặc không phản hồi | Khách có thể bắt đầu giao dịch mới |

**Các trường hợp kết thúc không tạo giao dịch:** khách không đồng ý, hết 5 phút chờ xác nhận, khách thoát giữa chừng chưa xác nhận, OTP sai quá 3 lần. Em giả định các trường hợp này không phát sinh giao dịch và không có trạng thái riêng; cần đối chiếu với trạng thái thật của hệ thống nghiệp vụ.

### 6.4 Interrupted transactions

| Tình huống | Trạng thái còn lại | Hệ thống xử lý | Khách thấy gì |
|---|---|---|---|
| Khách thoát app khi đang xác nhận, chưa bấm đồng ý | Chờ xác nhận | Giao dịch tự hủy sau 5 phút, không trừ tiền | Khi mở lại: giao dịch chưa được thực hiện |
| Khách thoát app sau khi đã nhập OTP đúng | Đang xử lý | Hệ thống vẫn hoàn tất, báo kết quả qua SMS và thông báo app | SMS hoặc thông báo kết quả |
| Hệ thống nghiệp vụ lỗi hoặc không phản hồi | Thất bại | Không trừ tiền, bot báo chưa thực hiện được | Thông báo lỗi và hotline |
| Đã trừ tiền nhưng chưa kích hoạt gói | Cần tra soát | Khách gọi hotline để tra soát; cơ chế hoàn tiền chưa có (OQ-9) | Thông báo liên hệ hotline |
| Khách bắt đầu giao dịch mới khi giao dịch cũ chưa xong | Chờ xác nhận | Hủy giao dịch cũ chưa xác nhận, tạo giao dịch mới; giao dịch đang xử lý thì không cho tạo mới | Thông báo chờ giao dịch trước hoàn tất |
| Hai thiết bị cùng đăng ký một gói | Đang xử lý | Chỉ một giao dịch được thực hiện, giao dịch còn lại báo đã được xử lý | Thông báo trùng yêu cầu |

## 7. Validation, Limits & Wording

### 7.1 Required info

- Giao dịch đăng ký gói: tên gói, số thuê bao áp dụng (mặc định là số đang đăng nhập), xác nhận của khách, OTP.
- Chi tiết thông tin bắt buộc của từng giao dịch khác viết ở board lĩnh vực.

### 7.2 Limits & quotas

| Mục | Giá trị | Nguồn |
|---|---|---|
| Số lần bot hỏi lại khi mơ hồ | Tối đa 2 lần | Giả định A5 |
| Hiệu lực OTP | 3 phút | Giả định A8 |
| Số lần nhập sai OTP | Tối đa 3 lần, quá thì khóa giao dịch | Giả định A8 |
| Thời gian chờ khách xác nhận trên màn hình tóm tắt | 5 phút | Giả định A8 |
| Giới hạn số giao dịch mỗi ngày hoặc giá trị mỗi giao dịch | Chưa có số | OQ-8 |

### 7.3 Wording (bản nháp, xưng hô "tôi - bạn")

**Thông báo lỗi**

| Tình huống | Nội dung |
|---|---|
| Không hiểu hoặc không hỗ trợ | "Xin lỗi, hiện tại tôi chưa hỗ trợ được yêu cầu này. Bạn vui lòng gọi hotline {số hotline} để được hỗ trợ." |
| Không đủ điều kiện đăng ký | "Số thuê bao {số thuê bao} chưa đủ điều kiện đăng ký gói {tên gói}: {lý do}." |
| OTP sai | "Mã OTP chưa đúng. Bạn vui lòng nhập lại. Bạn còn {n} lần thử." |
| OTP sai quá 3 lần | "Bạn đã nhập sai mã OTP 3 lần nên giao dịch đã bị khóa. Bạn vui lòng gọi hotline {số hotline} để được hỗ trợ." |
| OTP hết hiệu lực | "Mã OTP đã hết hiệu lực. Bạn có muốn nhận mã mới không?" |
| Quá 5 phút chưa xác nhận | "Phiên xác nhận đã hết hạn sau 5 phút. Giao dịch chưa được thực hiện." |
| Hệ thống lỗi | "Hệ thống đang gặp sự cố nên chưa thực hiện được giao dịch. Tài khoản của bạn chưa bị trừ tiền. Bạn vui lòng thử lại sau hoặc gọi hotline {số hotline}." |

**Thông báo thành công**

| Tình huống | Nội dung |
|---|---|
| Đăng ký gói thành công | "Bạn đã đăng ký thành công gói {tên gói}. Tin nhắn xác nhận đã được gửi đến số {số thuê bao}." |
| Khách hủy giao dịch | "Đã hủy yêu cầu đăng ký gói {tên gói}. Bạn chưa bị trừ tiền." |

**Thông báo thông tin**

| Tình huống | Nội dung |
|---|---|
| Tóm tắt trước khi xác nhận | "Bạn sắp đăng ký gói {tên gói}, giá {giá}, thời hạn {thời hạn}, áp dụng cho số {số thuê bao}. Bạn có đồng ý không?" |
| Đã gửi OTP | "Tôi đã gửi mã OTP đến số {số thuê bao}. Mã có hiệu lực trong 3 phút." |
| Hỏi lại khi mơ hồ | "Bạn muốn {lựa chọn A} hay {lựa chọn B}?" |
| Đang xử lý | "Yêu cầu của bạn đang được xử lý, vui lòng chờ trong giây lát." |

> Số hotline, danh sách tên gói và các tham số trong {} do trung tâm cung cấp (OQ-5 và board Viễn thông).

## 8. System Context (mức nghiệp vụ)

- **Thông tin cần lưu:** lịch sử hội thoại; lịch sử giao dịch do bot thực hiện (gói, thời điểm, trạng thái); nội dung tri thức (FAQ, tài liệu, thuật ngữ). Thời gian lưu chưa có (OQ trong mục 13).
- **Hệ thống bên ngoài:** hệ thống nghiệp vụ của trung tâm (tra cứu thuê bao, gói, cước; đăng ký gói; nạp thẻ; thanh toán), chưa biết tên và chưa rõ API (OQ-3); Super App (chưa rõ đơn vị quản lý, OQ-2); dịch vụ gửi OTP và SMS; dịch vụ chuyển giọng nói thành văn bản.
- **Thông báo:** SMS và thông báo trong app (Jun đã đồng ý "tạm thế").
- **Xử lý nền:** nhắc lịch gia hạn gói và thanh toán hóa đơn (P1).
- **Thời gian thực:** kết quả giao dịch trả ngay trong chat.

## 9. Edge Cases & Risks

### 9.1 Edge cases

- Mất kết nối giữa chừng: xem bảng 6.4.
- Hotline ngoài giờ làm việc: bot hoạt động liên tục nhưng khách chỉ được đưa số hotline; chưa biết hotline có hoạt động 24/7 không (OQ-11).
- Khách nêu nhiều yêu cầu trong một câu (ví dụ "đăng ký gói X và nạp 50 nghìn"): bot cần xử lý lần lượt từng giao dịch, mỗi giao dịch xác nhận riêng.
- Giọng nói vùng miền, nói nhỏ hoặc ồn; ảnh mờ: bot hỏi lại hoặc đề nghị nhập chữ; vượt số lần hỏi lại thì đưa hotline.
- Khách đổi ý giữa chừng sau khi đã xem tóm tắt: hủy được ở bước xác nhận, không hủy được khi đang xử lý.

### 9.2 Top rủi ro nghiệp vụ

| # | Rủi ro | Khả năng | Hậu quả nghiệp vụ | Cách phòng |
|---|---|---|---|---|
| R1 | Bot hiểu sai ý và thực hiện giao dịch ngoài ý muốn | Thỉnh thoảng | Khách bị trừ tiền, khiếu nại, tăng cuộc gọi hotline | Bước tóm tắt và xác nhận bắt buộc; OTP cho giao dịch có tiền |
| R2 | Phụ thuộc Super App và hệ thống nghiệp vụ chưa có đơn vị đầu mối và chưa rõ API | Thường | Không tích hợp được, chậm tiến độ toàn dự án | Chốt đầu mối và danh sách hệ thống sớm (OQ-2, OQ-3) |
| R3 | Không đạt mục tiêu giảm cuộc gọi: chưa có baseline và khi bot lỗi khách chỉ có hotline | Thường | Không chứng minh được hiệu quả; khách bực hơn khi phải chat rồi mới gọi | Thu số liệu baseline (OQ-5); cân nhắc lối thoát nhẹ (OQ-1) |
| R4 | Dữ liệu cá nhân (ảnh, giọng nói, hội thoại, dữ liệu cá nhân của thuê bao) bị lưu hoặc sử dụng chưa đúng quy định | Thỉnh thoảng | Rủi ro pháp lý và uy tín | Làm rõ yêu cầu về đồng ý của khách, thời gian lưu, nơi lưu ở board tiếp theo |

## 10. Success Criteria (preliminary)

Chưa có mục tiêu bằng số vì chưa có baseline (OQ-5). Các chỉ số đề xuất theo dõi:
- Tỷ lệ yêu cầu bot tự xử lý hoàn tất (không phải gọi hotline).
- Số cuộc gọi hotline thuộc các nhóm nghiệp vụ đã đưa lên bot (so với trước khi có bot).
- Tỷ lệ giao dịch bot bị khiếu nại vì hiểu sai ý.
- Tỷ lệ yêu cầu bot phải đưa hotline.
- Số lượt khách dùng bot trên số khách mở Super App.

## 11. Assumptions

- **[A1]** Chia thành một board khung và các board lĩnh vực, bắt đầu từ Viễn thông. Nếu sai, gộp thành một board duy nhất ở mức capability.
- **[A2]** Giọng nói chỉ nhận qua ô chat trong Super App, không có callbot ở giai đoạn 1. Nếu sai, cần mở rộng kênh và cách xác thực qua điện thoại.
- **[A3]** Tra cứu chỉ cần đăng nhập; mọi giao dịch có tiền cần thêm OTP gửi về số thuê bao. Nếu sai (có quy định khác về xác thực), điều chỉnh luồng 5.2 và bảng 7.2.
- **[A4]** Đăng ký gói trừ tài khoản chính của thuê bao; nạp thẻ và thanh toán hóa đơn qua phương thức đã liên kết trong Super App, có thể chuyển sang dịch vụ ngoài rồi quay lại. Nếu sai, điều chỉnh nguồn tiền và bảng 6.4.
- **[A5]** Bot hỏi lại tối đa 2 lần khi ý định mơ hồ. Nếu sai, điều chỉnh D1 và bảng 7.2.
- **[A6]** Giao dịch có 4 trạng thái: Chờ xác nhận, Đang xử lý, Thành công, Thất bại. Nếu khác trạng thái thật của hệ thống nghiệp vụ, điều chỉnh bảng 6.3.
- **[A7]** Sự cố giữa chừng xử lý theo bảng 6.4; trường hợp đã trừ tiền nhưng chưa kích hoạt do khách gọi hotline để tra soát. Nếu sai, điều chỉnh bảng 6.4 và OQ-9.
- **[A8]** OTP hiệu lực 3 phút; sai tối đa 3 lần; chờ xác nhận tối đa 5 phút. Nếu quy định hiện hành khác, điều chỉnh bảng 7.2.
- **[A9]** Nhân viên CSKH không dùng giao diện của bot (chỉ đưa hotline). Nếu có chuyển giao chat sang người, thêm role và luồng.

## 12. Open Questions

- [ ] OQ-1: Có lối thoát nhẹ (ghi lại yêu cầu, nhân viên gọi lại) thay vì chỉ đưa hotline khi bot không xử lý được không?
- [ ] OQ-2: Ai là đầu mối quản lý Super App? Bot nhúng vào Super App bằng cách nào?
- [ ] OQ-3: Tên hệ thống nghiệp vụ (billing, CRM, đăng ký gói)? API đã sẵn sàng chưa?
- [ ] OQ-4: Ai đề xuất, ai duyệt, có mốc thời gian nào không? Tờ trình V2 đang ở trạng thái nào?
- [ ] OQ-5: Số liệu hiện tại: số cuộc gọi mỗi ngày, số nhân sự CSKH, top lý do khách gọi? Chưa có thì ai cung cấp? Số hotline và giờ làm việc.
- [ ] OQ-6: Số thuê bao và số lượt chat dự kiến mỗi ngày?
- [ ] OQ-7: Giai đoạn 1 có chính thức cả 7 lĩnh vực không? BA đề xuất ưu tiên Viễn thông và Thanh toán.
- [ ] OQ-8: Giới hạn số giao dịch mỗi ngày hoặc giá trị tối đa mỗi giao dịch?
- [ ] OQ-9: Quy trình tra soát và hoàn tiền khi đã trừ tiền nhưng chưa kích hoạt gói?
- [ ] OQ-10: Role "Quản trị nội dung bot" có trong phạm vi dự án không?
- [ ] OQ-11: Hotline làm việc giờ nào, có 24/7 không?
- [ ] OQ-12: Khách chưa là thuê bao có được dùng bot không (FAQ, tư vấn gói)?

## 13. Danh sách câu hỏi cần làm rõ

### A. Đã chốt (không hỏi lại)

- Chủ đầu tư là Trung tâm VTDĐ; Jun là BA tiếp nhận yêu cầu.
- Ưu tiên nhóm vận hành hơn nhóm hình ảnh.
- Kênh giai đoạn 1: chat trong Super App, không có callbot.
- Bot không xử lý được thì báo "chưa hỗ trợ" và đưa hotline.
- Bot được tự thực hiện giao dịch nhưng bắt buộc có bước xác nhận rõ ràng.
- Xưng hô "tôi - bạn".
- Bot là dự án mới, tích hợp Super App để lấy dữ liệu.
- Thông báo qua SMS và trong app, có nhắc lịch.

### B. Đã hỏi, chưa có câu trả lời

Xem mục 12 (OQ-1 đến OQ-12). Ba câu đang **chặn** các board tiếp theo: OQ-2, OQ-3, OQ-7.

### C. Giả định BA chưa chắc, cần làm rõ với ban nghiệp vụ

Xem mục 11 (A1 đến A9). Cần ưu tiên xác nhận A3 (xác thực, có thể liên quan quy định về thuê bao và giao dịch, cần pháp chế hoặc bộ phận tuân thủ), A4 (nguồn tiền) và A7 (hoàn tiền).

### D. Sẽ hỏi ở các board sau

- **Viễn thông:** danh mục gói cước và nguồn cập nhật; mức độ bot được dùng thói quen sử dụng để gợi ý gói (dữ liệu cá nhân, khách có cần đồng ý không); các loại sự cố di động bot được tự xử lý; nạp thẻ bằng mã thẻ hay thanh toán điện tử.
- **Tài chính và thanh toán:** tài khoản liên kết là của ngân hàng hoặc ví nào; giới hạn pháp lý khi tra số dư; ngưỡng "giao dịch bất thường".
- **Dữ liệu cá nhân:** cách xin đồng ý khi nhận ảnh, giọng nói, dữ liệu sức khỏe; thời gian lưu lịch sử hội thoại; dữ liệu có bắt buộc lưu trong nước không.
- **Tri thức cho bot:** ai cung cấp FAQ, tài liệu, thuật ngữ (Excel mẫu hiện còn trống).
- **Lĩnh vực 2 đến 7:** mỗi lĩnh vực phụ thuộc đối tác ngoài (lịch, ngân hàng, hãng vé, dịch vụ công), cần xác nhận có hợp tác hay không.
- **Chất lượng:** định nghĩa "bot giải quyết được" để đo tỷ lệ tự xử lý; chỉ hỗ trợ tiếng Việt hay có thêm ngôn ngữ khác.

## 14. Next Steps

- Jun làm rõ 3 OQ đang chặn (OQ-2, OQ-3, OQ-7) với trung tâm VTDĐ.
- Sau khi Jun duyệt board này: chạy `ba-clarification-agent` (kế thừa các OQ còn giữ) hoặc tiếp tục brainstorm board lĩnh vực **Viễn thông**.
- Nếu muốn nắm domain MVNO và CSKH viễn thông trước: chạy `ba-research-agent`.
