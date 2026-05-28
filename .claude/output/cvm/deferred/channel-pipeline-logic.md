# Channel Pipeline Logic — Deferred (không hiển thị trên UI)

> Phần này được tách ra khỏi wireframe v8 (Section 5) theo quyết định ngày 2026-05-11.  
> Logic pipeline kênh sẽ được định nghĩa nội bộ bởi BA/Dev, không expose lên giao diện QTV.  
> Giữ lại tại đây để tái sử dụng khi cần thiết.

---

## Pipeline kênh gửi (Fallback Chain)

Thứ tự ưu tiên kênh mặc định (rẻ trước, SMS cuối):

```
Push  →  Zalo OA  →  SMS
```

Các kênh chưa tham gia mặc định: Banner, Email, USSD.

### Cấu hình per kênh trong pipeline

| Kênh | Thời gian chờ trước khi fallback | Điều kiện dừng fallback |
|---|---|---|
| Push | 30 phút | Clicked |
| Zalo OA | 30 phút | Delivered |
| SMS | Kênh cuối, không fallback tiếp | — |

### Điều kiện dừng toàn pipeline

- Mặc định: **Clicked** (KH đã click → không gửi tiếp kênh sau)
- Tùy chọn: Delivered / Opened / Clicked
- Tùy chọn bổ sung: `Không gửi SMS nếu KH đã Clicked ở kênh trước`

### Hành vi pipeline

- Kéo-thả để đổi thứ tự kênh (reorder)
- Click card kênh → expand inline để chỉnh "Chờ" (số phút) và "Dừng khi"
- Icon `[×]` khi hover → xóa kênh khỏi pipeline
- Kênh bị xóa khỏi Message Matrix → tự động gỡ khỏi pipeline (có confirm)
- Kênh thêm mới ở Message Matrix → xuất hiện vào vùng "Chưa tham gia"

### Wireframe mock-up gốc (từ v8)

```
┌────────────────────────────────────────────────────────────────────┐
│ PIPELINE KÊNH GỬI  (kéo-thả để đổi thứ tự)                        │
│                                                                    │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐         │
│  │ [≡] Push    │  →   │ [≡] Zalo OA │  →   │ [≡] SMS     │         │
│  │ Chờ: [30] p │      │ Chờ: [30] p │      │ Kênh cuối   │         │
│  │ Dừng khi:   │      │ Dừng khi:   │      │ ~2,400 SMS  │         │
│  │ [Clicked ▾] │      │[Delivered▾] │      │             │         │
│  └─────────────┘      └─────────────┘      └─────────────┘         │
│                                                                    │
│  Kênh chưa tham gia:  [Banner]  [Email]  [USSD]                    │
│                       [+ Thêm vào pipeline]                        │
│                                                                    │
│  ☑ Không gửi SMS nếu KH đã Clicked ở kênh trước                    │
│  Điều kiện dừng fallback toàn pipeline:                            │
│  ○ Delivered   ○ Opened   ● Clicked                                │
└────────────────────────────────────────────────────────────────────┘
```

---

## Đặt lịch gửi theo kênh (per-channel schedule)

Logic này vẫn còn trên UI tại Section 5 (accordion "Đặt lịch gửi theo kênh").

Nếu bật lịch riêng cho kênh → kênh đó chỉ gửi vào khung giờ được chọn, dù trigger kích hoạt bất kỳ lúc nào. Độc lập với thời gian gửi đã cấu hình ở Section 4.
