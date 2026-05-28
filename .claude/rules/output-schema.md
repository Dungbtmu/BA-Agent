# Output Schema — Tiêu chuẩn định dạng tài liệu

**Output chính của hệ thống là URD/SRS** — chuẩn định dạng URD/SRS xem trong `.claude/skills/urd-srs-structure.md`.

Epic, User Story, AC là artifact phụ trợ — chỉ tạo khi BA yêu cầu rõ. Schema dưới đây áp dụng khi các artifact đó được tạo ra.

Tất cả đầu ra phải tuân theo schema trong thư mục `schemas/`. Tham chiếu schema gốc tại đó khi cần chi tiết đầy đủ.

## Epic *(optional — chỉ khi BA yêu cầu)*

- **ID**: `EPIC-nnn` (ví dụ: `EPIC-001`)
- **Chỉ số thành công**: tối thiểu 3, phải đo lường được — sai: "cải thiện trải nghiệm"; đúng: "Tỉ lệ đăng ký thành công > 95%"
- **Scope**: phải xác định rõ cả In Scope lẫn Out of Scope
- **Phạm vi nghiệp vụ**: Epic chỉ chứa nội dung nghiệp vụ. KHÔNG đưa các mục technical (Architecture, Data Sources, API, Event...) — đó là phạm vi của SA/Dev

## User Story *(optional — chỉ khi BA yêu cầu)*

- **Định dạng**: `As a [vai trò], I want [hành động], So that [lợi ích]`
- **ID**: `STORY-nnn-nnn` (ví dụ: `STORY-001-002`)
- **Acceptance Criteria**: theo Gherkin (Given / When / Then), tối thiểu 3 kịch bản (happy path, alternative, error)
- **Edge Cases**: tối thiểu 2 trường hợp ngoại lệ có mô tả hành vi hệ thống
- **Dependencies**: liệt kê rõ Blocked By / Blocks

## Wireframe

- **ID**: `WF-nnn-nnn` (ví dụ: `WF-001-002`)
- Nếu có User Story → phủ 100% acceptance criteria tương ứng; nếu không có Story → bám sát solution/user flow từ Phase 1
- Mỗi component phải đặc tả đầy đủ: type, label, purpose, states (default, hover, error...), position
- Mọi tương tác người dùng phải có: trigger, process, success outcome, error handling
- Navigation flow phải rõ: Previous / Next / Alternative paths

## Chuỗi truy vết (Traceability)

Tùy vào output nào đã được tạo ra:

```
Đủ backlog:   PRD → Epic → Story → Wireframe → URD/SRS
Không backlog: PRD/Input → Solution → Wireframe → URD/SRS
```

- Nếu có Story → Wireframe phải liên kết về Story cha
- Nếu không có Story → Wireframe liên kết về chức năng/user flow trong solution
- **Không được có liên kết bị đứt** — kiểm tra trước khi trả kết quả

## Nghiêm cấm

- Dùng "TBD", "N/A", hoặc placeholder trong nội dung hoàn chỉnh
- Chỉ số mơ hồ: "cải thiện", "tốt hơn", "nhanh hơn" mà không có con số cụ thể
- Actor không nhất quán: không lúc "Giáo viên" lúc "Teacher" lúc "Thầy/Cô"
- Thuật ngữ nghiệp vụ biến thể: không lúc "Đăng ký" lúc "Ghi danh" cho cùng một khái niệm
