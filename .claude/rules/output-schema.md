# Output Schema — Tiêu chuẩn định dạng tài liệu

Epic, User Story, AC là artifact phụ trợ — chỉ tạo khi BA yêu cầu rõ. Schema bên dưới áp dụng khi các artifact đó được tạo ra.

---

## URD/SRS *(output chính)*

Cấu trúc chi tiết và thứ tự viết từng section xem trong `.claude/skills/urd-srs-structure.md`.

### Cấu trúc section bắt buộc

```
I.   Giới thiệu (I.1 Mục đích · I.2 Phạm vi · I.3 Thuật ngữ · I.4 Kiến trúc tổng thể)
II.  Yêu cầu tổng thể (II.1 Workflow · II.2 Function Tree · II.3 Permission · II.4 RBAC · II.5 Sequence)
III. Đặc tả Use Case
IV.  Giao diện chức năng
C.   Yêu cầu phi chức năng
```

Không được bỏ section — nếu thiếu thông tin thì ghi `[Cần xác nhận: ...]`, không bỏ trắng.

### Tiêu chuẩn chất lượng từng section

**Section I — Giới thiệu**
- I.2 Phạm vi: phải ghi rõ cả In Scope **và** Out of Scope
- I.3 Thuật ngữ: mọi từ chuyên ngành dùng trong tài liệu phải có định nghĩa tại đây
- I.4 Kiến trúc: vẽ Mermaid block diagram, diễn giải từng lớp — không chỉ dán sơ đồ

**Section II — Workflow & Sơ đồ**
- II.1 Workflow: swimlane đủ lane cho tất cả actor; đủ nhánh điều kiện (từ chối, lỗi, ngoại lệ)
- II.2 Function Tree: nhóm theo nghiệp vụ (không nhóm theo màn hình); tên dùng động từ + đối tượng
- II.3–II.4 Permission & RBAC: tất cả chức năng trong Function Tree phải có trong matrix; không có quyền conflict (vừa tạo vừa duyệt)
- II.5 Sequence: đủ `alt/else` cho nhánh hợp lệ và lỗi; response hệ thống sau mỗi action được mô tả

**Section III — Use Case**
- Mỗi chức năng lá trong Function Tree có ít nhất 1 UC
- Pre-condition, Alternative Flow, Exception đầy đủ — đây là nguồn test case ngoài happy path
- Business Rules phải cụ thể và testable — "email hợp lệ" không đủ, phải ghi điều kiện rõ
- Post-condition ghi rõ dữ liệu nào được tạo / cập nhật / xóa
- Thông báo lỗi trong Exception phải cụ thể — "Dữ liệu không hợp lệ" không đạt

**Section IV — Giao diện**
- Mỗi UC có ít nhất 1 màn hình tương ứng
- Bảng component đủ 4 cột: Định dạng · Bắt buộc · Mặc định · Mô tả
- Validation rule đủ từ cơ bản (bắt buộc, định dạng, độ dài) đến nâng cao (ràng buộc liên field, điều kiện nghiệp vụ)
- Thông báo lỗi cụ thể — "Ngày kết thúc phải sau ngày bắt đầu", không phải "Ngày không hợp lệ"
- Log requirement ghi cho các thao tác Sửa / Xóa / Duyệt

**Section C — Phi chức năng**
- Mỗi yêu cầu có con số hoặc tiêu chí đo được — "nhanh" không đạt; "< 2 giây" đạt
- Phải đề cập bảo mật: xác thực, phân quyền, mã hóa, log thao tác nhạy cảm
- Nếu có tích hợp hệ thống ngoài → mô tả xử lý lỗi tích hợp

### Traceability nội bộ URD/SRS

```
Business Function → Permission Matrix → Use Case → Giao diện
```

Trước khi output, phải đảm bảo:
- Mọi chức năng trong Function Tree có trong Permission Matrix
- Mọi chức năng trong Permission Matrix có ít nhất 1 UC
- Mọi UC có ít nhất 1 màn hình giao diện
- Tên Actor nhất quán xuyên suốt (Workflow, UC, RBAC, Giao diện)
- Tên chức năng nhất quán (Function Tree, Permission Matrix, UC)

---

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
