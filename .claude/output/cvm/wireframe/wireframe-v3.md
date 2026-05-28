# Wireframe CVM System — v3
> Cập nhật: Bổ sung logic Trigger Payload — mỗi trigger có tham số riêng, QTV chọn trigger chính làm nguồn tham số
> Các screen không đổi: Dashboard, Campaign List, Trigger Management, Customer List, Customer 360, Report, Template Management

---

## Thay đổi so với v2

| v2 | v3 |
|---|---|
| Tham số động dùng chung, không phân biệt trigger | Mỗi trigger có payload riêng |
| Cột trái chỉ hiện tên trigger | Cột trái hiện tên trigger + payload tương ứng |
| Cột giữa show tất cả tham số | Cột giữa chỉ show tham số của trigger chính |
| Không có khái niệm trigger chính | QTV chọn 1 trigger chính làm nguồn tham số |

---

## Screen 3: Campaign Builder — Cột trái (Updated)

```
┌─────────────────────┐
│ TRIGGER             │
│                     │
│ Logic: [OR ▾]       │
│                     │
│ ┌─────────────────────────────────────┐
│ │ ★ SIM_ACTIVATE          [✕]        │  ← ★ = trigger chính
│ │   Payload:                          │
│ │   • ngay_kich_hoat (BSS)            │
│ │   • loai_sim (BSS)                  │
│ │   • ten_kh (BSS)                    │
│ │   • so_dien_thoai (BSS)             │
│ └─────────────────────────────────────┘
│                                        
│ ┌─────────────────────────────────────┐
│ │   TOP_UP_FAIL            [✕]        │
│ │   Payload:                          │
│ │   • so_lan_nap_sai (BSS)            │
│ │   • so_tien_nap (BSS)               │
│ │   • ten_kh (BSS)                    │
│ └─────────────────────────────────────┘
│                                        
│ [+ Thêm trigger]                       
│                                        
│ Trigger chính:                         
│ [SIM_ACTIVATE ▾]                       
│ ⓘ Tham số trong tin nhắn sẽ lấy       
│   từ trigger này                       
│                                        
│ ─────────────────── 
│ SEGMENT             
│ ...                 
└─────────────────────┘
```

**Hành vi cột trái:**

- Mỗi trigger được chọn → expand hiện danh sách payload (tên tham số + nguồn BSS/OCS)
- Badge `★` đánh dấu trigger đang là trigger chính
- Dropdown **"Trigger chính"** — QTV chọn 1 trong các trigger đã thêm
- Khi đổi trigger chính → tham số trong cột giữa cập nhật theo
- Nếu chỉ có 1 trigger → tự động là trigger chính, không cần chọn

---

## Screen 3: Campaign Builder — Cột giữa (Updated)

```
┌───────────────────────┐
│ KÊNH & TIN NHẮN       │
│                       │
│ ┌───────────────────┐ │
│ │⠿ 1  USSD     [✕] │ │
│ │ (kênh đầu)        │ │
│ │ ──────────────── │ │
│ │ ○ Chọn template   │ │
│ │ ● Soạn mới        │ │
│ │ ┌─────────────┐   │ │
│ │ │Xin chào     │   │ │
│ │ │{{ten_kh}},  │   │ │
│ │ │SIM của bạn  │   │ │
│ │ │đã kích hoạt │   │ │
│ │ └─────────────┘   │ │
│ │                   │ │
│ │ Tham số           │ │  ← chỉ tham số của trigger chính
│ │ (SIM_ACTIVATE):   │ │
│ │ [{{ten_kh}}      ]│ │
│ │ [{{loai_sim}}    ]│ │
│ │ [{{ngay_kich_hoat}}]│ │
│ │ [{{so_dien_thoai}}]│ │
│ └───────────────────┘ │
│                       │
│ ┌───────────────────┐ │
│ │⠿ 2 Zalo OA   [✕] │ │
│ │ Fallback: 5 phút  │ │
│ │ ──────────────── │ │
│ │ ● Chọn template   │ │
│ │ [Chào mừng SIM▾]  │ │
│ │                   │ │
│ │ Tham số           │ │
│ │ (SIM_ACTIVATE):   │ │
│ │ [{{ten_kh}}      ]│ │
│ │ [{{loai_sim}}    ]│ │
│ │ [{{ngay_kich_hoat}}]│ │
│ └───────────────────┘ │
└───────────────────────┘
```

**Hành vi cột giữa:**

- Mỗi kênh hiện label **"Tham số (tên_trigger_chính):"** phía trên chip tham số
- Chip tham số chỉ gồm payload của trigger chính
- Click chip → insert vào textarea tại vị trí con trỏ
- Khi QTV đổi trigger chính ở cột trái:
  - Chip tham số trong tất cả kênh cập nhật theo
  - Nội dung đã soạn giữ nguyên (không xóa)
  - Tham số không còn hợp lệ → highlight màu đỏ + tooltip "Tham số này không thuộc trigger chính hiện tại"

---

## Screen 3: Campaign Builder — Cột phải (Updated)

```
┌───────────────────────┐
│ PREVIEW               │
│ Kênh: [Zalo OA ▾]    │
│                       │
│ ┌───────────────────┐ │
│ │  Xin chào         │ │
│ │  Nguyễn Văn A,    │ │
│ │  SIM eSIM của bạn │ │
│ │  đã kích hoạt     │ │
│ │  ngày 01/04/2026. │ │
│ └───────────────────┘ │
│                       │
│ Tham số mẫu           │
│ (SIM_ACTIVATE):       │
│ ten_kh =              │
│ [Nguyễn Văn A    ]    │
│ loai_sim =            │
│ [eSIM             ]   │
│ ngay_kich_hoat =      │
│ [01/04/2026       ]   │
│ so_dien_thoai =       │
│ [0987 xxx 001     ]   │
└───────────────────────┘
```

**Hành vi cột phải:**
- Label tham số mẫu hiển thị tên trigger chính
- Chỉ show tham số của trigger chính để điền mẫu
- Khi đổi trigger chính → tham số mẫu cập nhật theo

---

## Ví dụ đầy đủ: Campaign có 2 trigger, trigger chính = SIM_ACTIVATE

```
TRIGGER đã chọn:
  ★ SIM_ACTIVATE  →  ten_kh, loai_sim, ngay_kich_hoat, so_dien_thoai
    TOP_UP_FAIL   →  so_lan_nap_sai, so_tien_nap, ten_kh

Trigger chính: SIM_ACTIVATE

→ Cột giữa (chip tham số): ten_kh | loai_sim | ngay_kich_hoat | so_dien_thoai
→ Cột phải (tham số mẫu): 4 fields trên
→ Nếu nội dung có {{so_lan_nap_sai}} → highlight đỏ vì không thuộc trigger chính
```

---

## Edge Cases

| Tình huống | Hành vi |
|---|---|
| Chỉ có 1 trigger | Tự động là trigger chính, không hiện dropdown chọn |
| Đổi trigger chính khi đã soạn tin | Giữ nội dung, highlight tham số không hợp lệ màu đỏ |
| Xóa trigger đang là trigger chính | Tự động chọn trigger còn lại làm trigger chính, cập nhật chip |
| Trigger chính bị Admin tắt sau khi campaign Active | Hiện cảnh báo trong Campaign Detail |
