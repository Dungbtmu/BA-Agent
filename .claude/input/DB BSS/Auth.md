




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.6547bd92-c5f6-473b-81a3-5c23044362a5.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Hệ thống Xác thực và Phân quyền (Auth Schema)**

**Phiên bản 1.0**
**\


**1. Tổng quan**

Hệ thống cơ sở dữ liệu được thiết kế để quản lý xác thực, phân quyền và audit trong một môi trường đa hệ thống. Schema auth bao gồm các chức năng chính:

- Quản lý người dùng và vai trò (Users & Roles)
- Hệ thống phân quyền chi tiết (Permissions)
- Quản lý phạm vi dữ liệu (Data Scopes)
- Quản lý đa hệ thống (Multi-system)
- Audit và logging
- Quản lý trạng thái (State Management)
- Quản lý tổ chức (Organizations)

**2. Sơ đồ quan hệ các bảng chính**

Users (1) ←→ (N) User\_Roles (N) ←→ (1) Roles

`  `↓                                      ↓

User\_Systems (N) ←→ (1) Systems ←→ (N) Role\_Permissions

`  `↓                    ↓                  ↓

User\_Data\_Scopes    Menu\_Items        Permissions

`  `↓                    ↓                  ↓

Data\_Scopes      System\_Configs     Permission\_Types

**3. Mô tả chi tiết các bảng**

**3.1 Quản lý Người dùng**

**auth.users**

**Mục đích**: Lưu trữ thông tin người dùng cơ bản

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|code|varchar(20)|Mã người dùng (unique)|
|full\_name|varchar(64)|Họ tên đầy đủ|
|email|varchar(64)|Email (unique)|
|password|varchar(255)|Mật khẩu đã mã hóa|
|status|varchar(20)|Trạng thái: Active/Lock/Inactive|
|role\_id|int4|Vai trò mặc định|
|last\_system\_id|int4|Hệ thống truy cập cuối|
|failed\_login\_attempts|int2|Số lần đăng nhập thất bại|

**Ràng buộc**:

- Email và code phải unique
- Status chỉ được phép: Active, Lock, Inactive
- Gender chỉ được phép: Nam, Nữ, Khác

**auth.user\_sessions**

**Mục đích**: Quản lý phiên đăng nhập

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|session\_token|varchar(255)|Token phiên (unique)|
|refresh\_token|varchar(255)|Token làm mới|
|expires\_at|timestamp|Thời gian hết hạn|
|ip\_address|varchar(50)|Địa chỉ IP|
|user\_agent|text|Thông tin trình duyệt|

**3.2 Hệ thống Phân quyền**

**auth.roles**

**Mục đích**: Định nghĩa các vai trò trong hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|name|varchar(50)|Tên vai trò|
|code|varchar(50)|Mã vai trò (unique)|
|role\_level|int2|Cấp độ vai trò|
|status|varchar(20)|Trạng thái|

**auth.permissions**

**Mục đích**: Định nghĩa các quyền trong hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|name|varchar(100)|Tên quyền|
|code|varchar(50)|Mã quyền|
|permission\_type\_id|int4|Loại quyền|
|system\_id|int4|Hệ thống sở hữu|

**auth.role\_permissions**

**Mục đích**: Ánh xạ quyền cho vai trò với các hành động cụ thể

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|role\_id|int4|ID vai trò|
|permission\_id|int4|ID quyền|
|system\_id|int4|ID hệ thống|
|can\_create|bool|Quyền tạo mới|
|can\_read|bool|Quyền đọc|
|can\_update|bool|Quyền cập nhật|
|can\_delete|bool|Quyền xóa|
|can\_approve|bool|Quyền phê duyệt|
|can\_export|bool|Quyền xuất dữ liệu|
|can\_import|bool|Quyền nhập dữ liệu|

**3.3 Quản lý Phạm vi Dữ liệu**

**auth.data\_scopes**

**Mục đích**: Định nghĩa các phạm vi dữ liệu (data scope)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|code|varchar(50)|Mã phạm vi (unique)|
|name|varchar(100)|Tên phạm vi|
|scope\_type|varchar(50)|Loại phạm vi|

**auth.user\_data\_scopes & auth.role\_data\_scopes**

**Mục đích**: Ánh xạ phạm vi dữ liệu cho người dùng và vai trò

- Cho phép giới hạn dữ liệu mà người dùng có thể truy cập
- Có thể thiết lập theo vai trò hoặc cá nhân
- scope\_value: Giá trị cụ thể của phạm vi

**3.4 Quản lý Đa Hệ thống**

**auth.systems**

**Mục đích**: Quản lý các hệ thống trong môi trường đa hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|code|varchar(50)|Mã hệ thống (unique)|
|name|varchar(100)|Tên hệ thống|
|base\_url|varchar(255)|URL cơ sở|
|api\_base\_url|varchar(255)|URL API|
|db\_host|varchar(255)|Host database|
|connection\_string|text|Chuỗi kết nối|

**auth.system\_configs**

**Mục đích**: Lưu trữ cấu hình cho từng hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|config\_key|varchar(100)|Khóa cấu hình|
|config\_value|text|Giá trị cấu hình|
|data\_type|varchar(20)|Kiểu dữ liệu|
|is\_encrypted|bool|Có mã hóa không|

**3.5 Quản lý Menu và Giao diện**

**auth.menu\_items**

**Mục đích**: Quản lý cấu trúc menu cho từng hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|code|varchar(50)|Mã menu|
|name|varchar(200)|Tên hiển thị|
|parent\_id|int4|Menu cha (self-reference)|
|route\_path|varchar(200)|Đường dẫn|
|display\_order|int4|Thứ tự hiển thị|

**3.6 Hệ thống Audit và Logging**

**auth.access\_logs**

**Mục đích**: Ghi lại các lần truy cập tài nguyên

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|user\_id|int4|ID người dùng|
|resource\_type|varchar(50)|Loại tài nguyên|
|resource\_name|varchar(200)|Tên tài nguyên|
|method|varchar(10)|Phương thức HTTP|
|response\_code|int4|Mã phản hồi|
|execution\_time|int4|Thời gian thực thi|

**auth.change\_logs**

**Mục đích**: Ghi lại các thay đổi dữ liệu

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|table\_name|varchar(100)|Tên bảng|
|record\_id|varchar(100)|ID bản ghi|
|action|varchar(20)|Hành động (INSERT/UPDATE/DELETE)|
|old\_values|jsonb|Giá trị cũ|
|new\_values|jsonb|Giá trị mới|

**auth.login\_logs**

**Mục đích**: Ghi lại các lần đăng nhập

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|email|varchar(64)|Email đăng nhập|
|status|varchar(20)|Trạng thái (SUCCESS/FAILED)|
|reason|text|Lý do thất bại|

**3.7 Quản lý Trạng thái**

**auth.state\_definitions**

**Mục đích**: Định nghĩa các trạng thái và chuyển đổi

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|object\_type|varchar(100)|Loại đối tượng|
|status|varchar(100)|Mã trạng thái|
|state\_name|varchar(255)|Tên trạng thái|
|possible\_next\_statuses|\_text|Các trạng thái tiếp theo|
|is\_initial|bool|Trạng thái khởi tạo|
|is\_terminal|bool|Trạng thái kết thúc|

**auth.state\_history**

**Mục đích**: Lịch sử chuyển đổi trạng thái

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|object\_type|varchar(100)|Loại đối tượng|
|entity\_id|varchar(50)|ID thực thể|
|from\_status|varchar(100)|Trạng thái cũ|
|to\_status|varchar(100)|Trạng thái mới|
|transition\_comment|text|Ghi chú chuyển đổi|

**3.8 Quản lý Tổ chức**

**auth.organizations**

**Mục đích**: Quản lý cấu trúc tổ chức

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|code|varchar(50)|Mã tổ chức (unique)|
|name|varchar(100)|Tên tổ chức|
|parent\_id|int4|Tổ chức cha|
|address|text|Địa chỉ|

**auth.user\_organizations**

**Mục đích**: Ánh xạ người dùng với tổ chức

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|user\_id|int4|ID người dùng|
|organization\_id|int4|ID tổ chức|
|is\_primary|bool|Tổ chức chính|
|position|varchar(100)|Chức vụ|

**3.9 Quản lý Tác vụ Định kỳ**

**auth.scheduled\_tasks**

**Mục đích**: Quản lý các tác vụ được lên lịch

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|task\_name|varchar(100)|Tên tác vụ|
|cron\_expression|varchar(100)|Biểu thức cron|
|last\_run\_status|varchar(20)|Trạng thái chạy cuối|

**4. Chỉ mục (Indexes)**

Hệ thống sử dụng nhiều chỉ mục để tối ưu hiệu suất:

**Chỉ mục thường dùng:**

- idx\_users\_status: Tìm kiếm theo trạng thái người dùng
- idx\_access\_logs\_created\_at: Tìm kiếm log theo thời gian
- idx\_role\_permissions\_role\_id: Tìm quyền theo vai trò
- idx\_user\_sessions\_expires\_at: Quản lý phiên hết hạn

**5. Triggers và Functions**

**Triggers tự động cập nhật thời gian:**

- update\_users\_modtime: Cập nhật updated\_at cho bảng users
- update\_roles\_modtime: Cập nhật updated\_at cho bảng roles
- update\_systems\_modtime: Cập nhật updated\_at cho bảng systems

**Triggers đặc biệt:**

- state\_history\_notify\_trigger: Thông báo khi có thay đổi trạng thái

**6. Bảo mật**

**Mã hóa:**

- Mật khẩu được hash trước khi lưu
- Hỗ trợ mã hóa cấu hình nhạy cảm (is\_encrypted trong system\_configs)

**Audit Trail:**

- Ghi lại tất cả thay đổi dữ liệu quan trọng
- Lưu trữ thông tin IP, User Agent
- Tracking lịch sử đăng nhập


