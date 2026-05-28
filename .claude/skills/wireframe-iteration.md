---
name: wireframe_iteration
description: Quản lý vòng lặp wireframe — gen, nhận feedback, cập nhật có mục tiêu, không tái tạo toàn bộ
tools: []
---

# Skill: Wireframe Iteration

## Mục tiêu

Duy trì vòng lặp gen wireframe → feedback → cập nhật hiệu quả, đảm bảo chỉ sửa đúng phần được yêu cầu.

## Các giai đoạn

### Giai đoạn 1 — GEN (Tạo mới)

Input:
- EPIC hoặc solution đã có
- (Optional) Danh sách màn hình cần tạo

Output:
- Wireframe text-based cho từng màn hình
- User flow tổng

### Giai đoạn 2 — FEEDBACK (Nhận phản hồi)

Khi nhận feedback từ user, phân loại ngay:

| Loại | Ví dụ | Xử lý |
|---|---|---|
| Thêm mới | "Thêm màn hình X" | Tạo thêm, không đụng màn hình cũ |
| Sửa nội dung | "Step 3 bổ sung thêm..." | Chỉ sửa đúng phần đó |
| Sửa hành vi | "Khi click X thì hiện Y" | Bổ sung vào mục User Actions |
| Xóa | "Bỏ trường Z đi" | Xóa đúng trường đó |
| Tái cấu trúc | "Tách màn hình A thành 2 màn" | Tách, giữ nguyên nội dung |

### Giai đoạn 3 — UPDATE (Cập nhật)

Rules bắt buộc:
- KHÔNG tái tạo toàn bộ wireframe
- Chỉ render lại màn hình / section có thay đổi
- Ghi rõ "(Updated)" sau tên màn hình được sửa
- Ghi rõ "(Giữ nguyên)" cho màn hình không thay đổi
- Sau khi update: hỏi "Còn phần nào cần điều chỉnh không?"

## Output Format mỗi vòng lặp

```
## Wireframe Update — Vòng [N]

### Screen X: [Tên màn hình] (Updated)
[Nội dung đã cập nhật]

### Screen Y: [Tên màn hình] (Giữ nguyên)

### Thay đổi trong vòng này:
- [Màn hình X] — [Mô tả thay đổi ngắn gọn]
```

## Rules

- Luôn đánh số vòng lặp (Vòng 1, Vòng 2...)
- Không thêm nội dung ngoài feedback của user
- Không tự suy diễn feedback — nếu không rõ → hỏi lại trước khi sửa
- Sau mỗi vòng update: tóm tắt thay đổi + hỏi confirm tiếp theo

## Failure Cases

- Tái tạo toàn bộ wireframe khi chỉ cần sửa 1 phần
- Tự thêm màn hình / tính năng không có trong feedback
- Không ghi rõ phần nào đã thay đổi
- Sửa sai màn hình do hiểu nhầm feedback
