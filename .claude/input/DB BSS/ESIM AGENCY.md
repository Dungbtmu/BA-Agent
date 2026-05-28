




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.3aa7b379-abfb-4aed-8d3a-060cb554883d.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Hệ thống Quản lý Đại lý eSIM**

**Phiên bản 1.0**
**\


**TÀI LIỆU THIẾT KẾ CƠ SỞ DỮ LIỆU**

**Hệ thống Quản lý Đại lý eSIM (esim\_agency)**

**Phiên bản:** 1.0\
**Ngày tạo:** August 2025\
**Schema:** esim\_agency

-----
**1. TỔNG QUAN HỆ THỐNG**

**1.1 Mục đích**

Hệ thống quản lý đại lý eSIM được thiết kế để quản lý toàn bộ quy trình kinh doanh của các đại lý phân phối eSIM, bao gồm:

- **Quản lý thông tin đại lý**: Đăng ký, phân loại và quản lý hồ sơ đại lý
- **Quản lý bảng giá**: Thiết lập và cập nhật giá sản phẩm theo từng đại lý
- **Quản lý đơn hàng**: Xử lý đơn hàng và theo dõi trạng thái
- **Quản lý tài chính**: Ví điện tử, giao dịch và tín dụng
- **Audit và báo cáo**: Theo dõi lịch sử thay đổi và báo cáo
-----
**2. SƠ ĐỒ QUAN HỆ ENTITIES**

**2.1 Relationships chính**

agency\_types (1) ── (N) agencies

`                            `│

`                            `├── (1) individual\_info

`                            `├── (1) organization\_info  

`                            `├── (N) agency\_contacts

`                            `├── (N) representatives

`                            `├── (N) agency\_wallets

`                            `├── (N) agency\_orders

`                            `└── (N) agency\_price\_lists

price\_groups (1) → (N) price\_lists (1) → (N) price\_list\_items

price\_groups (1) → (N) provider\_package\_price\_lists (1) → (N) provider\_package\_price\_items

agencies (1) → (N) agency\_orders (1) → (N) agency\_order\_items

agency\_wallets (1) → (N) agency\_transactions

-----
**3. MÔ TẢ CHI TIẾT CÁC BẢNG**

**3.1 NHÓM QUẢN LÝ ĐẠI LÝ (Agency Management)**

**3.1.1 Bảng agency\_types**

**Mục đích:** Phân loại các loại đại lý theo cấp độ và đặc tính

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã định danh duy nhất|NOT NULL, UNIQUE|
|name|varchar(100)|Tên loại đại lý|NOT NULL|
|description|text|Mô tả chi tiết|NULL|
|level|int2|Cấp độ đại lý (0,1,2,3...)|NOT NULL, DEFAULT 0|
|default\_credit\_limit|numeric(15,2)|Hạn mức tín dụng mặc định|DEFAULT 0|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_types\_is\_active ON (is\_active)

**3.1.2 Bảng agencies**

**Mục đích:** Lưu trữ thông tin chính của đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã đại lý duy nhất|NOT NULL, UNIQUE|
|name|varchar(200)|Tên đại lý|NOT NULL|
|agency\_type\_id|int4|ID loại đại lý|NOT NULL, FK → agency\_types(id)|
|entity\_type|varchar(50)|Loại thực thể|NOT NULL, CHECK IN ('INDIVIDUAL','ORGANIZATION')|
|business\_model|varchar(50)|Mô hình kinh doanh|NOT NULL|
|agency\_form|varchar(50)|Hình thức đại lý|NOT NULL|
|parent\_organization\_id|int4|ID tổ chức cha|FK → auth.organizations(id)|
|esim\_approval\_limit|int4|Giới hạn phê duyệt eSIM|NOT NULL, DEFAULT 20|
|price\_list\_id|int4|ID bảng giá áp dụng|FK → price\_lists(id)|
|provider\_package\_price\_list\_id|int4|ID bảng giá gói nhà cung cấp|FK → provider\_package\_price\_lists(id)|
|last\_price\_update|timestamptz|Lần cập nhật giá cuối|NULL|
|status|varchar(50)|Trạng thái đại lý|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'AGENCY'|
|user\_id|int4|ID user liên kết|FK → auth.users(id)|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agencies\_agency\_type\_id ON (agency\_type\_id)
- idx\_agencies\_entity\_type ON (entity\_type)
- idx\_agencies\_business\_model ON (business\_model)
- idx\_agencies\_agency\_form ON (agency\_form)
- idx\_agencies\_price\_list\_id ON (price\_list\_id)
- idx\_agencies\_status ON (status)

**Foreign Keys:**

- agency\_type\_id → agency\_types(id)
- parent\_organization\_id → auth.organizations(id)
- price\_list\_id → price\_lists(id)
- provider\_package\_price\_list\_id → provider\_package\_price\_lists(id)
- user\_id → auth.users(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.1.3 Bảng individual\_info**

**Mục đích:** Thông tin chi tiết đại lý cá nhân (entity\_type = 'INDIVIDUAL')

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, UNIQUE, FK → agencies(id) CASCADE|
|full\_name|varchar(255)|Họ tên đầy đủ|NOT NULL|
|id\_type|varchar(50)|Loại giấy tờ (CCCD/CMND/Passport)|NOT NULL|
|id\_number|varchar(50)|Số giấy tờ|NOT NULL|
|id\_issue\_place|varchar(255)|Nơi cấp|NOT NULL|
|id\_issue\_date|date|Ngày cấp|NOT NULL|
|id\_expiry\_date|date|Ngày hết hạn|NULL|
|nationality|varchar(100)|Quốc tịch|NOT NULL|
|date\_of\_birth|date|Ngày sinh|NOT NULL|
|gender|varchar(10)|Giới tính|NULL|
|address|text|Địa chỉ|NULL|
|phone|varchar(20)|Số điện thoại|NOT NULL|
|email|varchar(100)|Email|NOT NULL|
|face\_image\_id|int4|ID ảnh chân dung|FK → auth.media\_files(id)|
|id\_front\_image\_id|int4|ID ảnh mặt trước CCCD|FK → auth.media\_files(id)|
|id\_back\_image\_id|int4|ID ảnh mặt sau CCCD|FK → auth.media\_files(id)|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_individual\_info\_agency\_id ON (agency\_id)

**3.1.4 Bảng organization\_info**

**Mục đích:** Thông tin chi tiết đại lý tổ chức (entity\_type = 'ORGANIZATION')

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, UNIQUE, FK → agencies(id) CASCADE|
|organization\_name|varchar(255)|Tên tổ chức|NOT NULL|
|address|text|Địa chỉ|NOT NULL|
|tax\_code|varchar(50)|Mã số thuế|NOT NULL|
|phone|varchar(20)|Số điện thoại|NOT NULL|
|email|varchar(100)|Email|NOT NULL|
|fax|varchar(50)|Số fax|NULL|
|website|varchar(255)|Website|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_organization\_info\_agency\_id ON (agency\_id)

**3.1.5 Bảng agency\_contacts**

**Mục đích:** Danh bạ liên hệ của đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id) CASCADE|
|full\_name|varchar(255)|Họ tên người liên hệ|NOT NULL|
|position|varchar(100)|Chức vụ|NOT NULL|
|gender|varchar(10)|Giới tính|NULL|
|date\_of\_birth|date|Ngày sinh|NOT NULL|
|phone|varchar(20)|Số điện thoại|NOT NULL|
|email|varchar(100)|Email|NOT NULL|
|is\_primary|bool|Liên hệ chính|NOT NULL, DEFAULT false|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_contacts\_agency\_id ON (agency\_id)
- idx\_agency\_contacts\_is\_primary ON (is\_primary)

**3.1.6 Bảng representatives**

**Mục đích:** Thông tin người đại diện (cho tổ chức)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id) CASCADE|
|full\_name|varchar(255)|Họ tên đại diện|NOT NULL|
|position|varchar(100)|Chức vụ|NOT NULL|
|gender|varchar(10)|Giới tính|NULL|
|date\_of\_birth|date|Ngày sinh|NOT NULL|
|id\_type|varchar(50)|Loại giấy tờ|NOT NULL|
|id\_number|varchar(50)|Số giấy tờ|NOT NULL|
|id\_issue\_place|varchar(255)|Nơi cấp|NOT NULL|
|id\_issue\_date|date|Ngày cấp|NOT NULL|
|id\_expiry\_date|date|Ngày hết hạn|NULL|
|nationality|varchar(100)|Quốc tịch|NOT NULL|
|address|text|Địa chỉ|NOT NULL|
|phone|varchar(20)|Số điện thoại|NULL|
|email|varchar(100)|Email|NULL|
|face\_image\_id|int4|ID ảnh chân dung|FK → auth.media\_files(id)|
|id\_front\_image\_id|int4|ID ảnh mặt trước CCCD|FK → auth.media\_files(id)|
|id\_back\_image\_id|int4|ID ảnh mặt sau CCCD|FK → auth.media\_files(id)|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_representatives\_agency\_id ON (agency\_id)
-----
**3.2 NHÓM QUẢN LÝ GIÁ (Pricing Management)**

**3.2.1 Bảng price\_groups**

**Mục đích:** Nhóm bảng giá để phân loại và quản lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã nhóm giá|NOT NULL, UNIQUE|
|name|varchar(100)|Tên nhóm giá|NOT NULL|
|description|text|Mô tả|NULL|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_price\_groups\_is\_active ON (is\_active)

**3.2.2 Bảng price\_lists**

**Mục đích:** Bảng giá sản phẩm chính

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã bảng giá|NOT NULL, UNIQUE|
|name|varchar(100)|Tên bảng giá|NOT NULL|
|description|text|Mô tả|NULL|
|price\_group\_id|int4|ID nhóm giá|NOT NULL, FK → price\_groups(id)|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|effective\_date|timestamptz|Ngày hiệu lực|NOT NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'AGENCY\_PRICE\_LIST'|
|applicable\_entity\_types|\_varchar|Loại đại lý áp dụng|NOT NULL, DEFAULT ARRAY['INDIVIDUAL','ORGANIZATION']|
|applicable\_levels|\_int2|Cấp độ áp dụng|NOT NULL, DEFAULT ARRAY[1,2,3]|
|is\_default|bool|Bảng giá mặc định|NOT NULL, DEFAULT false|
|version|int2|Phiên bản|NOT NULL, DEFAULT 1|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_price\_lists\_price\_group\_id ON (price\_group\_id)
- idx\_price\_lists\_effective\_date ON (effective\_date)
- idx\_price\_lists\_expiry\_date ON (expiry\_date)
- idx\_price\_lists\_status ON (status)

**Foreign Keys:**

- price\_group\_id → price\_groups(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.2.3 Bảng price\_list\_items**

**Mục đích:** Chi tiết giá từng sản phẩm trong bảng giá

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → price\_lists(id) CASCADE|
|product\_id|int4|ID sản phẩm|NULL|
|variant\_id|int4|ID biến thể sản phẩm|NULL|
|cost\_price|numeric(15,2)|Giá vốn|NOT NULL|
|selling\_price|numeric(15,2)|Giá bán|NOT NULL|
|discount\_rate|numeric(5,2)|Tỷ lệ chiết khấu (%)|DEFAULT 0|
|commission\_rate|numeric(5,2)|Tỷ lệ hoa hồng (%)|DEFAULT 0|
|min\_quantity|int4|Số lượng tối thiểu|NOT NULL, DEFAULT 1|
|max\_quantity|int4|Số lượng tối đa|NULL|
|status|varchar(50)|Trạng thái|NOT NULL, DEFAULT 'ACTIVE'|
|is\_displayed|bool|Hiển thị trên giao diện|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- CHECK: (product\_id IS NOT NULL) OR (variant\_id IS NOT NULL)
- UNIQUE: (price\_list\_id, product\_id, variant\_id, min\_quantity)

**Indexes:**

- idx\_price\_list\_items\_price\_list\_id ON (price\_list\_id)
- idx\_price\_list\_items\_product\_id ON (product\_id)
- idx\_price\_list\_items\_variant\_id ON (variant\_id)
- idx\_price\_list\_items\_status ON (status)

**3.2.4 Bảng agency\_price\_lists**

**Mục đích:** Áp dụng bảng giá cho từng đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id)|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → price\_lists(id)|
|effective\_date|timestamptz|Ngày hiệu lực|NOT NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NOT NULL, FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (agency\_id, price\_list\_id)

**Indexes:**

- idx\_agency\_price\_lists\_agency\_id ON (agency\_id)
- idx\_agency\_price\_lists\_price\_list\_id ON (price\_list\_id)
- idx\_agency\_price\_lists\_effective\_date ON (effective\_date)
- idx\_agency\_price\_lists\_is\_active ON (is\_active)

**3.2.5 Bảng provider\_package\_price\_lists**

**Mục đích:** Bảng giá gói từ nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã bảng giá|NOT NULL, UNIQUE|
|name|varchar(100)|Tên bảng giá|NOT NULL|
|description|text|Mô tả|NULL|
|price\_group\_id|int4|ID nhóm giá|NOT NULL, FK → price\_groups(id)|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|effective\_date|timestamptz|Ngày hiệu lực|NOT NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'PROVIDER\_PACKAGE\_PRICE\_LIST'|
|applicable\_entity\_types|\_varchar|Loại đại lý áp dụng|NOT NULL, DEFAULT ARRAY['INDIVIDUAL','ORGANIZATION']|
|applicable\_levels|\_int2|Cấp độ áp dụng|NOT NULL, DEFAULT ARRAY[1,2,3]|
|is\_default|bool|Bảng giá mặc định|NOT NULL, DEFAULT false|
|version|int2|Phiên bản|NOT NULL, DEFAULT 1|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_provider\_package\_price\_lists\_price\_group\_id ON (price\_group\_id)
- idx\_provider\_package\_price\_lists\_effective\_date ON (effective\_date)
- idx\_provider\_package\_price\_lists\_expiry\_date ON (expiry\_date)
- idx\_provider\_package\_price\_lists\_status ON (status)

**3.2.6 Bảng provider\_package\_price\_items**

**Mục đích:** Chi tiết giá gói từ nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → provider\_package\_price\_lists(id) CASCADE|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|package\_id|int4|ID gói dịch vụ|NOT NULL, FK → travel.provider\_packages(id)|
|cost\_price|numeric(15,2)|Giá vốn|NOT NULL|
|selling\_price|numeric(15,2)|Giá bán|NOT NULL|
|discount\_rate|numeric(5,2)|Tỷ lệ chiết khấu (%)|DEFAULT 0|
|commission\_rate|numeric(5,2)|Tỷ lệ hoa hồng (%)|DEFAULT 0|
|min\_quantity|int4|Số lượng tối thiểu|NOT NULL, DEFAULT 1|
|max\_quantity|int4|Số lượng tối đa|NULL|
|status|varchar(50)|Trạng thái|NOT NULL, DEFAULT 'ACTIVE'|
|is\_displayed|bool|Hiển thị trên giao diện|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (price\_list\_id, provider\_id, package\_id, min\_quantity)

**Indexes:**

- idx\_provider\_package\_price\_items\_price\_list\_id ON (price\_list\_id)
- idx\_provider\_package\_price\_items\_provider\_id ON (provider\_id)
- idx\_provider\_package\_price\_items\_package\_id ON (package\_id)
- idx\_provider\_package\_price\_items\_status ON (status)

**3.2.7 Bảng provider\_profit\_configs**

**Mục đích:** Cấu hình lợi nhuận cho nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_id|int4|ID nhà cung cấp|FK → resource.providers(id)|
|min\_profit\_rate|numeric(5,2)|Tỷ lệ lợi nhuận tối thiểu (%)|NOT NULL|
|max\_profit\_rate|numeric(5,2)|Tỷ lệ lợi nhuận tối đa (%)|NOT NULL|
|default\_profit\_rate|numeric(5,2)|Tỷ lệ lợi nhuận mặc định (%)|NOT NULL|
|effective\_date|timestamptz|Ngày hiệu lực|NOT NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|status|varchar(20)|Trạng thái|NOT NULL, DEFAULT 'ACTIVE'|
|description|text|Mô tả|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- CHECK: min\_profit\_rate <= max\_profit\_rate
- CHECK: default\_profit\_rate >= min\_profit\_rate AND default\_profit\_rate <= max\_profit\_rate
- CHECK: status IN ('ACTIVE', 'INACTIVE')

**Indexes:**

- idx\_provider\_profit\_configs\_provider\_id ON (provider\_id)
- idx\_provider\_profit\_configs\_effective\_date ON (effective\_date)
- idx\_provider\_profit\_configs\_expiry\_date ON (expiry\_date)
- idx\_provider\_profit\_configs\_status ON (status)

**Triggers:**

- update\_provider\_profit\_configs\_modtime BEFORE UPDATE → auth.update\_modified\_column()
-----
**3.3 NHÓM GIAO DỊCH VÀ TÀI CHÍNH (Transaction & Finance)**

**3.3.1 Bảng currency\_exchange\_rates**

**Mục đích:** Quản lý tỷ giá hối đoái giữa các loại tiền tệ

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|from\_currency|varchar(3)|Tiền tệ nguồn (VND, USD...)|NOT NULL|
|to\_currency|varchar(3)|Tiền tệ đích|NOT NULL|
|rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL|
|effective\_date|date|Ngày hiệu lực|NOT NULL|
|expiry\_date|date|Ngày hết hạn|NULL|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (from\_currency, to\_currency, effective\_date)

**Indexes:**

- idx\_currency\_exchange\_rates\_currencies ON (from\_currency, to\_currency)
- idx\_currency\_exchange\_rates\_effective\_date ON (effective\_date)
- idx\_currency\_exchange\_rates\_is\_active ON (is\_active)

**3.3.2 Bảng agency\_wallets**

**Mục đích:** Quản lý ví điện tử và tài chính của đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id) CASCADE|
|balance|numeric(15,2)|Số dư hiện tại|NOT NULL, DEFAULT 0|
|debt\_amount|numeric(15,2)|Số nợ|NOT NULL, DEFAULT 0|
|credit\_limit|numeric(15,2)|Hạn mức tín dụng|NOT NULL, DEFAULT 0|
|available\_credit|numeric(15,2)|Tín dụng khả dụng|GENERATED ALWAYS AS (GREATEST(0, credit\_limit - debt\_amount)) STORED|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|status|varchar(50)|Trạng thái ví|NOT NULL, DEFAULT 'ACTIVE'|
|last\_transaction\_id|int4|ID giao dịch cuối cùng|FK → agency\_transactions(id)|
|last\_transaction\_date|timestamptz|Ngày giao dịch cuối|NULL|
|before\_balance|varchar|Số dư trước đó|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (agency\_id, currency)

**Indexes:**

- idx\_agency\_wallets\_agency\_id ON (agency\_id)
- idx\_agency\_wallets\_status ON (status)

**Đặc điểm:**

- available\_credit là computed column: credit\_limit - debt\_amount (không âm)
- Mỗi đại lý có thể có nhiều ví theo từng loại tiền tệ

**3.3.3 Bảng agency\_transactions**

**Mục đích:** Ghi nhận tất cả giao dịch tài chính của đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|transaction\_code|varchar(50)|Mã giao dịch duy nhất|NOT NULL, UNIQUE|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id)|
|wallet\_id|int4|ID ví|NOT NULL, FK → agency\_wallets(id)|
|transaction\_type|varchar(50)|Loại giao dịch|NOT NULL|
|amount|numeric(15,2)|Số tiền giao dịch|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|balance\_before|numeric(15,2)|Số dư trước giao dịch|NOT NULL|
|balance\_after|numeric(15,2)|Số dư sau giao dịch|NOT NULL|
|transaction\_date|timestamptz|Ngày giờ giao dịch|NOT NULL|
|reference\_id|int4|ID tham chiếu|NULL|
|reference\_type|varchar(50)|Loại tham chiếu|NULL|
|payment\_method|varchar(50)|Phương thức thanh toán|NULL|
|payment\_details|jsonb|Chi tiết thanh toán|NULL|
|status|varchar(50)|Trạng thái giao dịch|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'AGENCY\_TRANSACTION'|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_transactions\_agency\_id ON (agency\_id)
- idx\_agency\_transactions\_wallet\_id ON (wallet\_id)
- idx\_agency\_transactions\_transaction\_type ON (transaction\_type)
- idx\_agency\_transactions\_transaction\_date ON (transaction\_date)
- idx\_agency\_transactions\_reference\_id\_type ON (reference\_id, reference\_type)
- idx\_agency\_transactions\_status ON (status)

**Foreign Keys:**

- agency\_id → agencies(id)
- wallet\_id → agency\_wallets(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.3.4 Bảng agency\_orders**

**Mục đích:** Quản lý đơn hàng của đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_code|varchar(50)|Mã đơn hàng duy nhất|NOT NULL, UNIQUE|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id)|
|order\_name|varchar(255)|Tên đơn hàng|NOT NULL|
|order\_date|timestamptz|Ngày đặt hàng|NOT NULL|
|total\_quantity|int4|Tổng số lượng|NOT NULL|
|activated\_quantity|int4|Số lượng đã kích hoạt|NOT NULL, DEFAULT 0|
|total\_amount|numeric(15,2)|Tổng giá trị|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|payment\_status|varchar(50)|Trạng thái thanh toán|NOT NULL|
|status|varchar(50)|Trạng thái đơn hàng|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'AGENCY\_ORDER'|
|transaction\_id|int4|ID giao dịch thanh toán|FK → agency\_transactions(id)|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_orders\_agency\_id ON (agency\_id)
- idx\_agency\_orders\_order\_date ON (order\_date)
- idx\_agency\_orders\_payment\_status ON (payment\_status)
- idx\_agency\_orders\_status ON (status)
- idx\_agency\_orders\_transaction\_id ON (transaction\_id)

**Foreign Keys:**

- agency\_id → agencies(id)
- transaction\_id → agency\_transactions(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.3.5 Bảng agency\_order\_items**

**Mục đích:** Chi tiết sản phẩm trong đơn hàng

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_id|int4|ID đơn hàng|NOT NULL, FK → agency\_orders(id)|
|esim\_id|int4|ID eSIM|NOT NULL|
|status|varchar(50)|Trạng thái item|NOT NULL|
|price|numeric(15,2)|Giá sản phẩm|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|is\_activated|bool|Đã kích hoạt|NOT NULL, DEFAULT false|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expire\_date|timestamptz|Ngày hết hạn|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_order\_items\_order\_id ON (order\_id)
- idx\_agency\_order\_items\_esim\_id ON (esim\_id)
- idx\_agency\_order\_items\_status ON (status)
- idx\_agency\_order\_items\_is\_activated ON (is\_activated)
-----
**3.4 NHÓM LỊCH SỬ VÀ AUDIT (History & Audit)**

**3.4.1 Bảng price\_list\_history**

**Mục đích:** Theo dõi lịch sử thay đổi bảng giá

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → price\_lists(id)|
|previous\_version|int2|Phiên bản trước|NULL|
|new\_version|int2|Phiên bản mới|NOT NULL|
|change\_date|timestamptz|Ngày thay đổi|NOT NULL|
|change\_type|varchar(20)|Loại thay đổi|NOT NULL|
|change\_reason|text|Lý do thay đổi|NULL|
|price\_group\_id|int4|ID nhóm giá|NOT NULL, FK → price\_groups(id)|
|effective\_date|timestamptz|Ngày hiệu lực|NOT NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NOT NULL, FK → auth.users(id)|

**Indexes:**

- idx\_price\_list\_history\_price\_list\_id ON (price\_list\_id)
- idx\_price\_list\_history\_change\_date ON (change\_date)
- idx\_price\_list\_history\_version ON (new\_version)

**3.4.2 Bảng price\_list\_item\_history**

**Mục đích:** Lịch sử thay đổi chi tiết giá sản phẩm

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_history\_id|int4|ID lịch sử bảng giá|NOT NULL, FK → price\_list\_history(id)|
|price\_list\_item\_id|int4|ID item bảng giá|NULL|
|product\_id|int4|ID sản phẩm|NULL|
|variant\_id|int4|ID biến thể|NULL|
|change\_type|varchar(20)|Loại thay đổi (CREATE/UPDATE/DELETE)|NOT NULL|
|old\_cost\_price|numeric(15,2)|Giá vốn cũ|NULL|
|old\_selling\_price|numeric(15,2)|Giá bán cũ|NULL|
|old\_discount\_rate|numeric(5,2)|Tỷ lệ chiết khấu cũ|NULL|
|old\_commission\_rate|numeric(5,2)|Tỷ lệ hoa hồng cũ|NULL|
|old\_min\_quantity|int4|Số lượng tối thiểu cũ|NULL|
|old\_max\_quantity|int4|Số lượng tối đa cũ|NULL|
|old\_status|varchar(50)|Trạng thái cũ|NULL|
|old\_is\_displayed|bool|Hiển thị cũ|NULL|
|new\_cost\_price|numeric(15,2)|Giá vốn mới|NULL|
|new\_selling\_price|numeric(15,2)|Giá bán mới|NULL|
|new\_discount\_rate|numeric(5,2)|Tỷ lệ chiết khấu mới|NULL|
|new\_commission\_rate|numeric(5,2)|Tỷ lệ hoa hồng mới|NULL|
|new\_min\_quantity|int4|Số lượng tối thiểu mới|NULL|
|new\_max\_quantity|int4|Số lượng tối đa mới|NULL|
|new\_status|varchar(50)|Trạng thái mới|NULL|
|new\_is\_displayed|bool|Hiển thị mới|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NOT NULL, FK → auth.users(id)|

**Indexes:**

- idx\_price\_list\_item\_history\_history\_id ON (price\_list\_history\_id)
- idx\_price\_list\_item\_history\_item\_id ON (price\_list\_item\_id)
- idx\_price\_list\_item\_history\_product\_id ON (product\_id)
- idx\_price\_list\_item\_history\_variant\_id ON (variant\_id)

**3.4.3 Bảng agency\_price\_list\_history**

**Mục đích:** Lịch sử áp dụng bảng giá cho đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → agencies(id)|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → price\_lists(id)|
|previous\_price\_list\_id|int4|ID bảng giá trước đó|FK → price\_lists(id)|
|applied\_date|timestamptz|Ngày áp dụng|NOT NULL|
|description|text|Mô tả thay đổi|NULL|
|download\_url|varchar(255)|URL tải bảng giá|NULL|
|price\_list\_items|jsonb|Snapshot các items|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NOT NULL, FK → auth.users(id)|

**Indexes:**

- idx\_agency\_price\_list\_history\_agency\_id ON (agency\_id)
- idx\_agency\_price\_list\_history\_price\_list\_id ON (price\_list\_id)
- idx\_agency\_price\_list\_history\_applied\_date ON (applied\_date)
- idx\_agency\_price\_list\_history\_changed\_items ON (price\_list\_items) USING GIN

**3.4.4 Bảng provider\_package\_price\_list\_history**

**Mục đích:** Lịch sử thay đổi bảng giá gói nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → provider\_package\_price\_lists(id)|
|code|varchar(50)|Mã bảng giá|NULL|
|name|varchar(100)|Tên bảng giá|NULL|
|description|text|Mô tả|NULL|
|price\_group\_id|int4|ID nhóm giá|FK → price\_groups(id)|
|currency|varchar(3)|Loại tiền tệ|NULL|
|effective\_date|timestamptz|Ngày hiệu lực|NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|status|varchar(50)|Trạng thái|NULL|
|object\_type|varchar(100)|Loại đối tượng|NULL|
|applicable\_entity\_types|\_varchar|Loại đại lý áp dụng|NULL|
|applicable\_levels|\_int2|Cấp độ áp dụng|NULL|
|is\_default|bool|Bảng giá mặc định|NULL|
|version|int2|Phiên bản mới|NOT NULL|
|previous\_version|int2|Phiên bản trước|NULL|
|change\_date|timestamptz|Ngày thay đổi|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|change\_type|varchar(20)|Loại thay đổi|NOT NULL|
|change\_reason|text|Lý do thay đổi|NULL|
|old\_data|jsonb|Dữ liệu cũ|NULL|
|new\_data|jsonb|Dữ liệu mới|NULL|
|user\_id|int4|ID người dùng|NOT NULL, FK → auth.users(id)|
|ip\_address|varchar(50)|Địa chỉ IP|NULL|
|user\_agent|text|User agent|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Indexes:**

- idx\_provider\_package\_price\_list\_history\_price\_list\_id ON (price\_list\_id)
- idx\_provider\_package\_price\_list\_history\_change\_date ON (change\_date)
- idx\_provider\_package\_price\_list\_history\_change\_type ON (change\_type)
- idx\_provider\_package\_price\_list\_history\_code ON (code)
- idx\_provider\_package\_price\_list\_history\_effective\_date ON (effective\_date)
- idx\_provider\_package\_price\_list\_history\_price\_group\_id ON (price\_group\_id)
- idx\_provider\_package\_price\_list\_history\_status ON (status)
- idx\_provider\_package\_price\_list\_history\_user\_id ON (user\_id)
- idx\_provider\_package\_price\_list\_history\_version ON (version)
- idx\_provider\_package\_price\_list\_history\_old\_data ON (old\_data) USING GIN
- idx\_provider\_package\_price\_list\_history\_new\_data ON (new\_data) USING GIN
-----
**3.5 NHÓM CẤU HÌNH HỆ THỐNG (System Configuration)**

**3.5.1 Bảng agency\_system\_configs**

**Mục đích:** Lưu trữ các cấu hình hệ thống cho đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|config\_key|varchar(100)|Khóa cấu hình|NOT NULL, UNIQUE|
|config\_value|text|Giá trị cấu hình|NOT NULL|
|data\_type|varchar(20)|Kiểu dữ liệu (string/int/bool/json)|NOT NULL|
|description|text|Mô tả cấu hình|NULL|
|for\_agency\_types|\_int4|Áp dụng cho các loại đại lý|NULL|
|for\_agencies|\_int4|Áp dụng cho các đại lý cụ thể|NULL|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_system\_configs\_config\_key ON (config\_key)
- idx\_agency\_system\_configs\_for\_agencies ON (for\_agencies) USING GIN
- idx\_agency\_system\_configs\_is\_active ON (is\_active)
-----
**4. RELATIONSHIPS VÀ FOREIGN KEYS**

**4.1 Relationships chính**

**Agency Management Relationships**

agency\_types (1) → (N) agencies

agencies (1) → (0..1) individual\_info [entity\_type = 'INDIVIDUAL']

agencies (1) → (0..1) organization\_info [entity\_type = 'ORGANIZATION']

agencies (1) → (N) agency\_contacts

agencies (1) → (N) representatives

agencies (1) → (N) agency\_wallets

agencies (1) → (N) agency\_orders

agencies (1) → (N) agency\_price\_lists

**Pricing Relationships**

price\_groups (1) → (N) price\_lists

price\_lists (1) → (N) price\_list\_items

price\_groups (1) → (N) provider\_package\_price\_lists

provider\_package\_price\_lists (1) → (N) provider\_package\_price\_items

**Financial Relationships**

agencies (1) → (N) agency\_wallets

agency\_wallets (1) → (N) agency\_transactions

agencies (1) → (N) agency\_orders

agency\_orders (1) → (N) agency\_order\_items

**History Relationships**

price\_lists (1) → (N) price\_list\_history

price\_list\_history (1) → (N) price\_list\_item\_history

agencies (1) → (N) agency\_price\_list\_history

provider\_package\_price\_lists (1) → (N) provider\_package\_price\_list\_history

**4.2 Cross-schema References**

esim\_agency.agencies → auth.users(id)

esim\_agency.agencies → auth.organizations(id)

esim\_agency.agencies → auth.state\_definitions(object\_type, status)

esim\_agency.individual\_info → auth.media\_files(id)

esim\_agency.provider\_package\_price\_items → resource.providers(id)

esim\_agency.provider\_package\_price\_items → travel.provider\_packages(id)

-----
**5. BUSINESS RULES VÀ CONSTRAINTS**

**5.1 Data Constraints**

**Entity Type Validation**

- agencies.entity\_type chỉ nhận giá trị 'INDIVIDUAL' hoặc 'ORGANIZATION'
- Khi entity\_type = 'INDIVIDUAL' thì phải có bản ghi trong individual\_info
- Khi entity\_type = 'ORGANIZATION' thì phải có bản ghi trong organization\_info

**Price List Items Validation**

- Trong price\_list\_items phải có ít nhất product\_id hoặc variant\_id
- selling\_price phải >= cost\_price
- min\_quantity phải > 0
- max\_quantity phải >= min\_quantity (nếu có)

**Financial Constraints**

- agency\_wallets.available\_credit = MAX(0, credit\_limit - debt\_amount)
- agency\_transactions.balance\_after = balance\_before ± amount
- Mỗi đại lý chỉ có một ví cho mỗi loại tiền tệ

**Profit Configuration Constraints**

- min\_profit\_rate <= max\_profit\_rate
- min\_profit\_rate <= default\_profit\_rate <= max\_profit\_rate

**5.2 Unique Constraints**

|**Bảng**|**Unique Constraint**|
| :- | :- |
|agency\_types|code|
|agencies|code|
|individual\_info|agency\_id|
|organization\_info|agency\_id|
|price\_groups|code|
|price\_lists|code|
|provider\_package\_price\_lists|code|
|agency\_wallets|(agency\_id, currency)|
|agency\_transactions|transaction\_code|
|agency\_orders|order\_code|
|agency\_price\_lists|(agency\_id, price\_list\_id)|
|price\_list\_items|(price\_list\_id, product\_id, variant\_id, min\_quantity)|
|provider\_package\_price\_items|(price\_list\_id, provider\_id, package\_id, min\_quantity)|
|currency\_exchange\_rates|(from\_currency, to\_currency, effective\_date)|
|agency\_system\_configs|config\_key|

**5.3 Cascade Rules**

**DELETE CASCADE**

- agencies → individual\_info, organization\_info, agency\_contacts, representatives, agency\_wallets
- price\_lists → price\_list\_items
- provider\_package\_price\_lists → provider\_package\_price\_items

**RESTRICT (No Cascade)**

- agency\_types → agencies (không được xóa loại đại lý khi còn đại lý sử dụng)
- price\_groups → price\_lists (không được xóa nhóm giá khi còn bảng giá)
- agencies → agency\_orders (không được xóa đại lý khi còn đơn hàng)
-----
**6. INDEXING STRATEGY**

**6.1 Performance Indexes**

**Primary Indexes (Automatically Created)**

- Tất cả các PRIMARY KEY
- Tất cả các UNIQUE constraints

**Foreign Key Indexes**

-- Agency Management

CREATE INDEX idx\_agencies\_agency\_type\_id ON agencies(agency\_type\_id);

CREATE INDEX idx\_agencies\_price\_list\_id ON agencies(price\_list\_id);

CREATE INDEX idx\_agency\_contacts\_agency\_id ON agency\_contacts(agency\_id);

CREATE INDEX idx\_individual\_info\_agency\_id ON individual\_info(agency\_id);

CREATE INDEX idx\_organization\_info\_agency\_id ON organization\_info(agency\_id);

CREATE INDEX idx\_representatives\_agency\_id ON representatives(agency\_id);

-- Pricing

CREATE INDEX idx\_price\_lists\_price\_group\_id ON price\_lists(price\_group\_id);

CREATE INDEX idx\_price\_list\_items\_price\_list\_id ON price\_list\_items(price\_list\_id);

CREATE INDEX idx\_provider\_package\_price\_items\_price\_list\_id ON provider\_package\_price\_items(price\_list\_id);

-- Financial

CREATE INDEX idx\_agency\_wallets\_agency\_id ON agency\_wallets(agency\_id);

CREATE INDEX idx\_agency\_transactions\_agency\_id ON agency\_transactions(agency\_id);

CREATE INDEX idx\_agency\_transactions\_wallet\_id ON agency\_transactions(wallet\_id);

CREATE INDEX idx\_agency\_orders\_agency\_id ON agency\_orders(agency\_id);

CREATE INDEX idx\_agency\_order\_items\_order\_id ON agency\_order\_items(order\_id);

**Business Logic Indexes**

-- Status và Active flags

CREATE INDEX idx\_agency\_types\_is\_active ON agency\_types(is\_active);

CREATE INDEX idx\_agencies\_status ON agencies(status);

CREATE INDEX idx\_price\_groups\_is\_active ON price\_groups(is\_active);

CREATE INDEX idx\_price\_lists\_status ON price\_lists(status);

CREATE INDEX idx\_agency\_wallets\_status ON agency\_wallets(status);

-- Date ranges cho performance queries

CREATE INDEX idx\_price\_lists\_effective\_date ON price\_lists(effective\_date);

CREATE INDEX idx\_price\_lists\_expiry\_date ON price\_lists(expiry\_date);

CREATE INDEX idx\_agency\_transactions\_transaction\_date ON agency\_transactions(transaction\_date);

CREATE INDEX idx\_agency\_orders\_order\_date ON agency\_orders(order\_date);

-- Business filtering

CREATE INDEX idx\_agencies\_entity\_type ON agencies(entity\_type);

CREATE INDEX idx\_agencies\_business\_model ON agencies(business\_model);

CREATE INDEX idx\_agency\_contacts\_is\_primary ON agency\_contacts(is\_primary);

CREATE INDEX idx\_agency\_order\_items\_is\_activated ON agency\_order\_items(is\_activated);

**Composite Indexes**

-- Currency exchange rates lookup

CREATE INDEX idx\_currency\_exchange\_rates\_currencies ON currency\_exchange\_rates(from\_currency, to\_currency);

-- Transaction reference lookup

CREATE INDEX idx\_agency\_transactions\_reference\_id\_type ON agency\_transactions(reference\_id, reference\_type);

-- Multi-column searches

CREATE INDEX idx\_agency\_price\_lists\_agency\_effective ON agency\_price\_lists(agency\_id, effective\_date);

CREATE INDEX idx\_price\_list\_items\_product\_status ON price\_list\_items(product\_id, status);

**JSONB Indexes (GIN)**

-- History data search

CREATE INDEX idx\_agency\_price\_list\_history\_changed\_items ON agency\_price\_list\_history USING GIN (price\_list\_items);

CREATE INDEX idx\_provider\_package\_price\_list\_history\_old\_data ON provider\_package\_price\_list\_history USING GIN (old\_data);

CREATE INDEX idx\_provider\_package\_price\_list\_history\_new\_data ON provider\_package\_price\_list\_history USING GIN (new\_data);

-- Array columns

CREATE INDEX idx\_agency\_system\_configs\_for\_agencies ON agency\_system\_configs USING GIN (for\_agencies);

**6.2 Partitioning Strategy**

**Time-based Partitioning (Khuyến nghị cho production)**

-- Partition transaction tables theo tháng

CREATE TABLE agency\_transactions\_y2025m01 PARTITION OF agency\_transactions

FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

-- Partition history tables theo năm

CREATE TABLE price\_list\_history\_2025 PARTITION OF price\_list\_history

FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');


