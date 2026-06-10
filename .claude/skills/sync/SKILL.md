---
name: sync
description: Orchestrator chế độ SYNC — đồng bộ artifact khi requirement thay đổi. Gồm 4 bước tuần tự: change-handler (parse thay đổi) → impact-analysis (tra Traceability Map) → artifact-patch (patch đúng phần bị ảnh hưởng) → verify. Yêu cầu Traceability Map đã tồn tại. Trigger khi BA mô tả thay đổi requirement, PO cập nhật file mới, hoặc cần đồng bộ artifact sau khi scope thay đổi.
tools: []
---

# Skill: SYNC Orchestrator

Điều phối toàn bộ luồng đồng bộ artifact khi requirement thay đổi. Chỉ patch đúng phần bị ảnh hưởng — không chạy lại toàn bộ pipeline.

> Quy tắc chung (IT-BA framing, no-re-ask, assumption, approval gate): xem `.claude/rules/ba-conventions.md`.

---

## Điều kiện tiên quyết

Traceability Map phải tồn tại tại `.claude/output/[tên_dự_án]/traceability-map.md` — nếu chưa có, không chạy SYNC. Yêu cầu BA chạy pipeline GENERATE đến bước [T] trước.

---

## 4 Sub-skill và thứ tự thực hiện

| Bước | Sub-skill | Mục tiêu | Khi nào dừng checkpoint |
|---|---|---|---|
| S1 | [`change-handler.md`](references/change-handler.md) | Parse trigger (BA mô tả / file PO mới) → Delta Summary | Luôn dừng để BA confirm Delta Summary trước khi tiếp tục |
| S2 | [`impact-analysis.md`](references/impact-analysis.md) | Tra Traceability Map → tính artifact bị ảnh hưởng → Impact Report | Dừng nếu có REMOVE requirement hoặc ambiguity MAJOR |
| S3 | [`artifact-patch.md`](references/artifact-patch.md) | Patch đúng phần bị ảnh hưởng theo thứ tự → Patch Summary | Dừng nếu patch có thể gây conflict với section khác |
| S4 | verify | Gọi `ba-qa-agent` verify không có conflict mới, sau đó `ba-postcheck-agent` audit cấu trúc | Báo cáo kết quả cuối |

---

## Quy trình

```
Trigger (BA mô tả / file PO mới)
   ↓
[S1] change-handler → Delta Summary → BA confirm
   ↓
[S2] impact-analysis → Impact Report
   ├─ REMOVE requirement hoặc ambiguity MAJOR → dừng, hỏi BA
   └─ OK → tiếp tục
   ↓
[S3] artifact-patch → patch theo thứ tự artifact → Patch Summary
   ↓
[S4] ba-qa-agent → ba-postcheck-agent → báo cáo xong
```

---

## Nguyên tắc

- Chỉ patch artifact có trong Impact Report — không tự mở rộng scope
- Thứ tự patch: URD/SRS trước → Wireframe → Backlog (nếu có)
- Mỗi patch phải ghi rõ REQ ID liên quan để giữ traceability
- Không xóa nội dung cũ nếu chưa có approval — mark `[DEPRECATED: ...]` thay thế
