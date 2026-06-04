# Output Schema — Danh sách artifact và tiêu chuẩn tối thiểu

Tiêu chuẩn chất lượng chi tiết từng section xem trong các skill files tương ứng.

---

## Artifact và ID format

| Artifact | ID format | Ví dụ | Optional? |
|---|---|---|---|
| Epic | `EPIC-nnn` | `EPIC-001` | Có — chỉ khi BA yêu cầu |
| User Story | `STORY-nnn-nnn` | `STORY-001-002` | Có — chỉ khi BA yêu cầu |
| Wireframe | `WF-nnn-nnn` | `WF-001-002` | Có — chỉ khi BA yêu cầu |
| URD/SRS | `urd-srs-v[N].md` | `urd-srs-v1.md` | Không — output chính |
| Process Summary | `process-summary.md` | — | Tự động sau postcheck |

---

## Cấu trúc section URD/SRS

```
I.   Giới thiệu (I.1 Mục đích · I.2 Phạm vi · I.3 Thuật ngữ · I.4 Kiến trúc tổng thể)
II.  Yêu cầu tổng thể (II.1 Workflow · II.2 Function Tree · II.3 Permission · II.4 RBAC · II.5 Sequence)
III. Đặc tả Use Case
IV.  Giao diện chức năng
C.   Yêu cầu phi chức năng
```

Không được bỏ section — nếu thiếu thông tin thì ghi `[Cần xác nhận: ...]`, không bỏ trắng.

Tiêu chuẩn chất lượng từng section: xem `urd-review-checklist.md`.
Thứ tự viết và traceability nội bộ: xem `urd-srs-structure.md`.
Đặc tả giao diện (bảng component 6 cột, validation): xem `urd-screen-spec.md`.

---

## Traceability chain

```
Đủ backlog:    PRD → Epic → Story → Wireframe → URD/SRS → Process Summary
Không backlog: PRD/Input → Solution → Wireframe → URD/SRS → Process Summary
```

Traceability nội bộ URD/SRS:
```
Business Function → Permission Matrix → Use Case → Giao diện
```

Kiểm tra traceability tự động: xem `document-integrity-check.md`.

---

## Nghiêm cấm

- Dùng "TBD" hoặc placeholder trong nội dung hoàn chỉnh (trừ `[Cần xác nhận: ...]` trong URD/SRS)
- Chỉ số mơ hồ không đo được: "cải thiện", "tốt hơn", "nhanh hơn"
- Actor hoặc thuật ngữ nghiệp vụ không nhất quán xuyên suốt tài liệu
