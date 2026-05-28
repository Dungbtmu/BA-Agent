




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.29fbe97d-1651-44f2-9600-aeee8ea4fb25.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Hệ thống Portal eSIM Portal**

**Phiên bản 1.0**
**\


**1. TỔNG QUAN HỆ THỐNG**

**1.1 Mục đích**

Hệ thống Portal eSIM được thiết kế để cung cấp giao diện khách hàng cho các đại lý bán eSIM, bao gồm:

- **Customer Management**: Quản lý khách hàng của đại lý
- **Portal Configuration**: Cấu hình giao diện portal theo từng đại lý
- **Order Processing**: Xử lý đơn hàng từ khách hàng qua portal
- **Payment Management**: Quản lý thanh toán và giao dịch
- **Support System**: Hệ thống hỗ trợ khách hàng
- **eSIM Delivery**: Giao hàng và kích hoạt eSIM
- **Discount Management**: Quản lý mã giảm giá
-----
**2. SƠ ĐỒ QUAN HỆ ENTITIES**

**2.1 Relationships chính**

esim\_agency.agencies (1) → (1) agency\_portal\_configs

esim\_agency.agencies (1) → (N) customers

customers (1) → (N) customer\_orders

customer\_orders (1) → (N) customer\_order\_items

customer\_order\_items (1) → (N) customer\_order\_item\_esims

customer\_orders (1) → (N) payment\_transactions

customers (1) → (N) support\_tickets

support\_tickets (1) → (N) support\_ticket\_messages

agencies (1) → (N) discount\_codes

discount\_codes (1) → (N) discount\_code\_usage

agencies (1) → (N) esim\_replacement\_requests

-----
**3. MÔ TẢ CHI TIẾT CÁC BẢNG**

**3.1 NHÓM CẤU HÌNH PORTAL (Portal Configuration)**

**3.1.1 Bảng agency\_portal\_configs**

**Mục đích:** Cấu hình giao diện portal cho từng đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, UNIQUE, FK → esim\_agency.agencies(id)|
|portal\_name|varchar(200)|Tên portal|NOT NULL|
|logo\_url|text|URL logo|NULL|
|favicon\_url|text|URL favicon|NULL|
|primary\_color|varchar(20)|Màu chính|DEFAULT '#3B82F6'|
|secondary\_color|varchar(20)|Màu phụ|DEFAULT '#1E3A8A'|
|hero\_image\_url|text|URL ảnh hero|NULL|
|header\_text|varchar(500)|Text header|NULL|
|footer\_text|varchar(500)|Text footer|NULL|
|custom\_css|text|CSS tùy chỉnh|NULL|
|custom\_domain|varchar(255)|Tên miền riêng|NULL|
|is\_custom\_domain\_active|bool|Kích hoạt domain riêng|DEFAULT false|
|meta\_title|varchar(255)|SEO title|NULL|
|meta\_description|text|SEO description|NULL|
|google\_analytics\_id|varchar(50)|Google Analytics ID|NULL|
|seo\_keywords|text|Từ khóa SEO|NULL|
|contact\_email|varchar(100)|Email liên hệ|NULL|
|contact\_phone|varchar(20)|Số điện thoại liên hệ|NULL|
|social\_links|jsonb|Links mạng xã hội|NULL|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|status|varchar(50)|Trạng thái|DEFAULT 'ACTIVE'|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'PORTAL\_CONFIG'|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_agency\_portal\_configs\_agency\_id ON (agency\_id)
- idx\_agency\_portal\_configs\_status ON (status)

**Foreign Keys:**

- agency\_id → esim\_agency.agencies(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.1.2 Bảng discount\_codes**

**Mục đích:** Quản lý mã giảm giá cho từng đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|code|varchar(50)|Mã giảm giá|NOT NULL|
|name|varchar(100)|Tên mã giảm giá|NOT NULL|
|description|text|Mô tả|NULL|
|discount\_type|varchar(20)|Loại giảm giá|NOT NULL|
|discount\_value|numeric(15,2)|Giá trị giảm|NOT NULL|
|min\_order\_amount|numeric(15,2)|Giá trị đơn hàng tối thiểu|NOT NULL, DEFAULT 0|
|max\_discount\_amount|numeric(15,2)|Số tiền giảm tối đa|NULL|
|valid\_from|timestamptz|Ngày bắt đầu|NOT NULL|
|valid\_to|timestamptz|Ngày kết thúc|NOT NULL|
|usage\_limit|int4|Giới hạn sử dụng|NULL|
|usage\_count|int4|Số lần đã sử dụng|NOT NULL, DEFAULT 0|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|applicable\_products|\_int4|Sản phẩm áp dụng|NULL|
|applicable\_variants|\_int4|Biến thể áp dụng|NULL|
|applicable\_regions|\_int4|Khu vực áp dụng|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- CHECK: discount\_type IN ('PERCENTAGE', 'FIXED\_AMOUNT')
- UNIQUE: (agency\_id, code)

**Indexes:**

- idx\_discount\_codes\_agency\_id ON (agency\_id)
- idx\_discount\_codes\_is\_active ON (is\_active)
- idx\_discount\_codes\_valid\_period ON (valid\_from, valid\_to)
-----
**3.2 NHÓM QUẢN LÝ KHÁCH HÀNG (Customer Management)**

**3.2.1 Bảng customers**

**Mục đích:** Quản lý thông tin khách hàng của từng đại lý

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|full\_name|varchar(255)|Họ tên đầy đủ|NOT NULL|
|email|varchar(100)|Email|NOT NULL|
|phone|varchar(20)|Số điện thoại|NULL|
|country\_code|varchar(5)|Mã quốc gia|NULL|
|password\_hash|varchar(255)|Hash mật khẩu|NULL|
|is\_guest|bool|Khách vãng lai|NOT NULL, DEFAULT false|
|status|varchar(50)|Trạng thái|DEFAULT 'ACTIVE'|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'PORTAL\_CUSTOMER'|
|last\_login\_at|timestamptz|Lần đăng nhập cuối|NULL|
|last\_login\_ip|varchar(50)|IP đăng nhập cuối|NULL|
|user\_agent|text|User agent|NULL|
|avatar\_url|text|URL avatar|NULL|
|address|text|Địa chỉ|NULL|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (agency\_id, email)

**Indexes:**

- idx\_customers\_agency\_id ON (agency\_id)
- idx\_customers\_email ON (email)
- idx\_customers\_status ON (status)

**Foreign Keys:**

- agency\_id → esim\_agency.agencies(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)
-----
**3.3 NHÓM XỬ LÝ ĐÚN HÀNG (Order Processing)**

**3.3.1 Bảng order\_requests**

**Mục đích:** Log request đặt hàng từ API/Portal

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|bigserial|Khóa chính|PRIMARY KEY|
|request\_data|jsonb|Dữ liệu request gốc|NOT NULL|
|customer\_id|int8|ID khách hàng|NULL|
|customer\_name|varchar(255)|Tên khách hàng|NULL|
|customer\_email|varchar(255)|Email khách hàng|NULL|
|package\_id|varchar(50)|ID gói dịch vụ|NULL|
|quantity|int4|Số lượng|NULL|
|price|numeric(15,2)|Giá|NULL|
|currency|varchar(10)|Loại tiền tệ|NULL|
|payment\_method|varchar(50)|Phương thức thanh toán|NULL|
|agent\_code|varchar(100)|Mã đại lý|NULL|
|order\_code|varchar(100)|Mã đơn hàng|NULL|
|status|varchar(20)|Trạng thái|DEFAULT 'PENDING'|
|accept\_language|varchar(10)|Ngôn ngữ|DEFAULT 'vi'|
|client\_ip|varchar(45)|IP client|NULL|
|user\_agent|text|User agent|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|processing\_started\_at|timestamptz|Bắt đầu xử lý|NULL|
|processing\_finished\_at|timestamptz|Hoàn thành xử lý|NULL|
|error\_message|text|Thông báo lỗi|NULL|
|error\_code|varchar(50)|Mã lỗi|NULL|
|steps|jsonb|Các bước xử lý|NULL|

**Indexes:**

- idx\_order\_requests\_client\_ip ON (client\_ip)
- idx\_order\_requests\_created\_at ON (created\_at)
- idx\_order\_requests\_customer\_email ON (customer\_email)
- idx\_order\_requests\_order\_code ON (order\_code)
- idx\_order\_requests\_request\_data ON (request\_data) USING GIN
- idx\_order\_requests\_status ON (status)

**Triggers:**

- tr\_order\_requests\_updated\_at BEFORE UPDATE → update\_order\_requests\_updated\_at()

**3.3.2 Bảng customer\_orders**

**Mục đích:** Đơn hàng từ khách hàng qua portal

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_code|varchar(50)|Mã đơn hàng|NOT NULL, UNIQUE|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|customer\_id|int4|ID khách hàng|NOT NULL, FK → customers(id)|
|order\_date|timestamptz|Ngày đặt hàng|NOT NULL|
|total\_amount|numeric(15,2)|Tổng tiền hàng|NOT NULL|
|discount\_amount|numeric(15,2)|Số tiền giảm giá|NOT NULL, DEFAULT 0|
|tax\_amount|numeric(15,2)|Tiền thuế|NOT NULL, DEFAULT 0|
|final\_amount|numeric(15,2)|Số tiền cuối cùng|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|payment\_method|varchar(50)|Phương thức thanh toán|NULL|
|payment\_reference|varchar(100)|Mã tham chiếu thanh toán|NULL|
|payment\_date|timestamptz|Ngày thanh toán|NULL|
|status|varchar(50)|Trạng thái đơn hàng|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'CUSTOMER\_ORDER'|
|agency\_order\_id|int4|ID đơn hàng đại lý|FK → esim\_agency.agency\_orders(id)|
|notes|text|Ghi chú|NULL|
|source|varchar(50)|Nguồn đơn hàng|NOT NULL, DEFAULT 'PORTAL'|
|ip\_address|varchar(50)|Địa chỉ IP|NULL|
|user\_agent|text|User agent|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_customer\_orders\_agency\_id ON (agency\_id)
- idx\_customer\_orders\_customer\_id ON (customer\_id)
- idx\_customer\_orders\_order\_date ON (order\_date)
- idx\_customer\_orders\_status ON (status)
- idx\_customer\_orders\_agency\_order\_id ON (agency\_order\_id)

**Foreign Keys:**

- agency\_id → esim\_agency.agencies(id)
- customer\_id → customers(id)
- agency\_order\_id → esim\_agency.agency\_orders(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.3.3 Bảng customer\_order\_items**

**Mục đích:** Chi tiết sản phẩm trong đơn hàng khách

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_id|int4|ID đơn hàng|NOT NULL, FK → customer\_orders(id)|
|esim\_id|int4|ID eSIM|NULL|
|product\_id|int4|ID sản phẩm|NOT NULL, FK → inventory.products(id)|
|variant\_id|int4|ID biến thể|FK → inventory.product\_variants(id)|
|provider\_package\_id|int4|ID gói nhà cung cấp|FK → travel.provider\_packages(id)|
|quantity|int4|Số lượng|NOT NULL, DEFAULT 1|
|price|numeric(15,2)|Giá sản phẩm|NOT NULL|
|discount\_amount|numeric(15,2)|Số tiền giảm giá|NOT NULL, DEFAULT 0|
|tax\_amount|numeric(15,2)|Tiền thuế|NOT NULL, DEFAULT 0|
|final\_price|numeric(15,2)|Giá cuối cùng|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|status|varchar(50)|Trạng thái item|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'CUSTOMER\_ORDER\_ITEM'|
|agency\_order\_item\_id|int4|ID item đơn hàng đại lý|FK → esim\_agency.agency\_order\_items(id)|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_customer\_order\_items\_order\_id ON (order\_id)
- idx\_customer\_order\_items\_esim\_id ON (esim\_id)
- idx\_customer\_order\_items\_product\_id ON (product\_id)
- idx\_customer\_order\_items\_variant\_id ON (variant\_id)
- idx\_customer\_order\_items\_status ON (status)
- idx\_customer\_order\_items\_agency\_order\_item\_id ON (agency\_order\_item\_id)

**Foreign Keys:**

- order\_id → customer\_orders(id)
- product\_id → inventory.products(id)
- variant\_id → inventory.product\_variants(id)
- agency\_order\_item\_id → esim\_agency.agency\_order\_items(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.3.4 Bảng discount\_code\_usage**

**Mục đích:** Theo dõi việc sử dụng mã giảm giá

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|discount\_id|int4|ID mã giảm giá|NOT NULL, FK → discount\_codes(id)|
|order\_id|int4|ID đơn hàng|NOT NULL, FK → customer\_orders(id)|
|customer\_id|int4|ID khách hàng|NOT NULL, FK → customers(id)|
|discount\_amount|numeric(15,2)|Số tiền được giảm|NOT NULL|
|created\_at|timestamptz|Thời gian sử dụng|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Constraints:**

- UNIQUE: (discount\_id, order\_id)

**Indexes:**

- idx\_discount\_code\_usage\_discount\_id ON (discount\_id)
- idx\_discount\_code\_usage\_order\_id ON (order\_id)
- idx\_discount\_code\_usage\_customer\_id ON (customer\_id)
-----
**3.4 NHÓM QUẢN LÝ THANH TOÁN (Payment Management)**

**3.4.1 Bảng payment\_transactions**

**Mục đích:** Giao dịch thanh toán từ khách hàng

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|transaction\_code|varchar(50)|Mã giao dịch|NOT NULL, UNIQUE|
|order\_id|int4|ID đơn hàng|NOT NULL, FK → customer\_orders(id)|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|customer\_id|int4|ID khách hàng|NOT NULL, FK → customers(id)|
|amount|numeric(15,2)|Số tiền giao dịch|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'VND'|
|exchange\_rate|numeric(15,6)|Tỷ giá quy đổi|NOT NULL, DEFAULT 1|
|transaction\_type|varchar(20)|Loại giao dịch|NOT NULL|
|payment\_method|varchar(50)|Phương thức thanh toán|NOT NULL|
|transaction\_reference|varchar(100)|Mã tham chiếu|NULL|
|payment\_details|jsonb|Chi tiết thanh toán|NULL|
|status|varchar(50)|Trạng thái giao dịch|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'PAYMENT\_TRANSACTION'|
|transaction\_date|timestamptz|Ngày giao dịch|NOT NULL|
|notes|text|Ghi chú|NULL|
|ip\_address|varchar(50)|Địa chỉ IP|NULL|
|user\_agent|text|User agent|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_payment\_transactions\_order\_id ON (order\_id)
- idx\_payment\_transactions\_agency\_id ON (agency\_id)
- idx\_payment\_transactions\_customer\_id ON (customer\_id)
- idx\_payment\_transactions\_status ON (status)
- idx\_payment\_transactions\_transaction\_date ON (transaction\_date)
- idx\_payment\_transactions\_transaction\_type ON (transaction\_type)

**Foreign Keys:**

- order\_id → customer\_orders(id)
- agency\_id → esim\_agency.agencies(id)
- customer\_id → customers(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)
-----
**3.5 NHÓM GIAO HÀNG ESIM (eSIM Delivery)**

**3.5.1 Bảng customer\_order\_item\_esims**

**Mục đích:** Quản lý eSIM được giao cho từng order item

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_item\_id|int4|ID order item|NOT NULL, FK → customer\_order\_items(id)|
|esim\_id|int4|ID eSIM|NULL|
|iccid|varchar(30)|ICCID của eSIM|NULL|
|activation\_code|varchar(100)|Mã kích hoạt|NULL|
|qrcode\_url|text|URL QR code|NULL|
|provider\_id|int4|ID nhà cung cấp|FK → resource.providers(id)|
|provider\_package\_id|int4|ID gói nhà cung cấp|FK → travel.provider\_packages(id)|
|package\_info|jsonb|Thông tin gói|NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|status|varchar(50)|Trạng thái eSIM|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ORDER\_ITEM\_ESIM'|
|api\_request\_id|varchar(100)|ID request API|NULL|
|api\_request\_data|jsonb|Dữ liệu request API|NULL|
|api\_response\_data|jsonb|Dữ liệu response API|NULL|
|api\_error\_message|text|Thông báo lỗi API|NULL|
|api\_retry\_count|int2|Số lần retry API|NOT NULL, DEFAULT 0|
|request\_date|timestamptz|Ngày request|NULL|
|response\_date|timestamptz|Ngày response|NULL|
|delivery\_date|timestamptz|Ngày giao hàng|NULL|
|delivery\_method|varchar(50)|Phương thức giao hàng|DEFAULT 'EMAIL'|
|delivery\_status|varchar(50)|Trạng thái giao hàng|DEFAULT 'PENDING'|
|delivery\_reference|varchar(100)|Mã tham chiếu giao hàng|NULL|
|delivery\_error\_message|text|Thông báo lỗi giao hàng|NULL|
|delivery\_retry\_count|int2|Số lần retry giao hàng|NOT NULL, DEFAULT 0|
|delivery\_retry\_max|int2|Số lần retry tối đa|NOT NULL, DEFAULT 3|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (order\_item\_id, esim\_id)

**Indexes:**

- idx\_customer\_order\_item\_esims\_order\_item\_id ON (order\_item\_id)
- idx\_customer\_order\_item\_esims\_esim\_id ON (esim\_id)
- idx\_customer\_order\_item\_esims\_iccid ON (iccid)
- idx\_customer\_order\_item\_esims\_status ON (status)
- idx\_customer\_order\_item\_esims\_delivery\_status ON (delivery\_status)
- idx\_customer\_order\_item\_esims\_api\_retry\_count ON (api\_retry\_count)
- idx\_customer\_order\_item\_esims\_delivery\_retry\_count ON (delivery\_retry\_count)

**Foreign Keys:**

- order\_item\_id → customer\_order\_items(id)
- provider\_id → resource.providers(id)
- provider\_package\_id → travel.provider\_packages(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.5.2 Bảng order\_item\_esim\_api\_logs**

**Mục đích:** Log các request API cho eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_item\_esim\_id|int4|ID order item eSIM|NOT NULL, FK → customer\_order\_item\_esims(id)|
|request\_type|varchar(50)|Loại request|NOT NULL|
|request\_data|jsonb|Dữ liệu request|NOT NULL|
|response\_data|jsonb|Dữ liệu response|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|error\_message|text|Thông báo lỗi|NULL|
|retry\_number|int2|Số lần retry|NOT NULL, DEFAULT 0|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Indexes:**

- idx\_order\_item\_esim\_api\_logs\_order\_item\_esim\_id ON (order\_item\_esim\_id)
- idx\_order\_item\_esim\_api\_logs\_status ON (status)
- idx\_order\_item\_esim\_api\_logs\_created\_at ON (created\_at)

**3.5.3 Bảng order\_item\_esim\_delivery\_logs**

**Mục đích:** Log quá trình giao hàng eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_item\_esim\_id|int4|ID order item eSIM|NOT NULL, FK → customer\_order\_item\_esims(id)|
|delivery\_method|varchar(50)|Phương thức giao hàng|NOT NULL|
|recipient|varchar(255)|Người nhận|NOT NULL|
|content\_data|jsonb|Dữ liệu nội dung|NOT NULL|
|status|varchar(50)|Trạng thái giao hàng|NOT NULL|
|error\_message|text|Thông báo lỗi|NULL|
|retry\_number|int2|Số lần retry|NOT NULL, DEFAULT 0|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Indexes:**

- idx\_order\_item\_esim\_delivery\_logs\_order\_item\_esim\_id ON (order\_item\_esim\_id)
- idx\_order\_item\_esim\_delivery\_logs\_status ON (status)
- idx\_order\_item\_esim\_delivery\_logs\_created\_at ON (created\_at)
-----
**3.6 NHÓM HỖ TRỢ KHÁCH HÀNG (Support System)**

**3.6.1 Bảng support\_tickets**

**Mục đích:** Quản lý ticket hỗ trợ khách hàng

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|ticket\_code|varchar(50)|Mã ticket|NOT NULL, UNIQUE|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|customer\_id|int4|ID khách hàng|FK → customers(id)|
|subject|varchar(255)|Tiêu đề|NOT NULL|
|description|text|Mô tả vấn đề|NOT NULL|
|priority|varchar(20)|Mức độ ưu tiên|DEFAULT 'MEDIUM'|
|category|varchar(50)|Danh mục|NOT NULL|
|assigned\_to|int4|Người được giao|FK → auth.users(id)|
|status|varchar(50)|Trạng thái ticket|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'SUPPORT\_TICKET'|
|related\_order\_id|int4|ID đơn hàng liên quan|FK → customer\_orders(id)|
|related\_esim\_id|int4|ID eSIM liên quan|NULL|
|attachment\_urls|jsonb|URLs file đính kèm|NULL|
|closed\_at|timestamptz|Thời gian đóng|NULL|
|resolution\_note|text|Ghi chú giải quyết|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_support\_tickets\_agency\_id ON (agency\_id)
- idx\_support\_tickets\_customer\_id ON (customer\_id)
- idx\_support\_tickets\_assigned\_to ON (assigned\_to)
- idx\_support\_tickets\_status ON (status)
- idx\_support\_tickets\_priority ON (priority)
- idx\_support\_tickets\_category ON (category)
- idx\_support\_tickets\_related\_order\_id ON (related\_order\_id)
- idx\_support\_tickets\_related\_esim\_id ON (related\_esim\_id)

**Foreign Keys:**

- agency\_id → esim\_agency.agencies(id)
- customer\_id → customers(id)
- assigned\_to → auth.users(id)
- related\_order\_id → customer\_orders(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.6.2 Bảng support\_ticket\_messages**

**Mục đích:** Tin nhắn trao đổi trong ticket

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|ticket\_id|int4|ID ticket|NOT NULL, FK → support\_tickets(id)|
|sender\_type|varchar(20)|Loại người gửi|NOT NULL|
|sender\_id|int4|ID người gửi|NOT NULL|
|message|text|Nội dung tin nhắn|NOT NULL|
|attachment\_urls|jsonb|URLs file đính kèm|NULL|
|is\_internal|bool|Tin nhắn nội bộ|NOT NULL, DEFAULT false|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Constraints:**

- CHECK: sender\_type IN ('CUSTOMER', 'AGENT', 'STAFF')

**Indexes:**

- idx\_support\_ticket\_messages\_ticket\_id ON (ticket\_id)
- idx\_support\_ticket\_messages\_sender ON (sender\_type, sender\_id)

**Foreign Keys:**

- ticket\_id → support\_tickets(id)

**3.6.3 Bảng esim\_replacement\_requests**

**Mục đích:** Yêu cầu thay thế eSIM lỗi

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|request\_code|varchar(50)|Mã yêu cầu|NOT NULL, UNIQUE|
|agency\_id|int4|ID đại lý|NOT NULL, FK → esim\_agency.agencies(id)|
|customer\_id|int4|ID khách hàng|FK → customers(id)|
|original\_esim\_id|int4|ID eSIM gốc|NOT NULL|
|replacement\_esim\_id|int4|ID eSIM thay thế|NULL|
|order\_item\_id|int4|ID order item|FK → customer\_order\_items(id)|
|issue\_type|varchar(50)|Loại vấn đề|NOT NULL|
|issue\_description|text|Mô tả vấn đề|NOT NULL|
|issue\_reported\_date|timestamptz|Ngày báo cáo|NOT NULL|
|priority|varchar(20)|Mức độ ưu tiên|NOT NULL, DEFAULT 'MEDIUM'|
|status|varchar(50)|Trạng thái yêu cầu|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ESIM\_REPLACEMENT\_REQUEST'|
|related\_ticket\_id|int4|ID ticket liên quan|FK → support\_tickets(id)|
|provider\_id|int4|ID nhà cung cấp|FK → resource.providers(id)|
|provider\_package\_id|int4|ID gói nhà cung cấp|FK → travel.provider\_packages(id)|
|approved\_by|int4|Người phê duyệt|FK → auth.users(id)|
|approved\_at|timestamptz|Thời gian phê duyệt|NULL|
|processed\_by|int4|Người xử lý|FK → auth.users(id)|
|processed\_at|timestamptz|Thời gian xử lý|NULL|
|completed\_at|timestamptz|Thời gian hoàn thành|NULL|
|resolution\_note|text|Ghi chú giải quyết|NULL|
|refund\_amount|numeric(15,2)|Số tiền hoàn lại|NULL|
|refund\_currency|varchar(3)|Loại tiền hoàn lại|DEFAULT 'VND'|
|refund\_transaction\_id|int4|ID giao dịch hoàn tiền|FK → payment\_transactions(id)|
|is\_compensated|bool|Đã bồi thường|NOT NULL, DEFAULT false|
|compensation\_details|jsonb|Chi tiết bồi thường|NULL|
|source|varchar(20)|Nguồn yêu cầu|NOT NULL, DEFAULT 'AGENCY\_REQUEST'|
|detected\_by|varchar(50)|Phát hiện bởi|NULL|
|end\_user\_affected|bool|Ảnh hưởng người dùng|NOT NULL, DEFAULT true|
|internal\_note|text|Ghi chú nội bộ|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_esim\_replacement\_requests\_agency\_id ON (agency\_id)
- idx\_esim\_replacement\_requests\_customer\_id ON (customer\_id)
- idx\_esim\_replacement\_requests\_original\_esim\_id ON (original\_esim\_id)
- idx\_esim\_replacement\_requests\_replacement\_esim\_id ON (replacement\_esim\_id)
- idx\_esim\_replacement\_requests\_order\_item\_id ON (order\_item\_id)
- idx\_esim\_replacement\_requests\_issue\_type ON (issue\_type)
- idx\_esim\_replacement\_requests\_status ON (status)
- idx\_esim\_replacement\_requests\_issue\_reported\_date ON (issue\_reported\_date)

**Foreign Keys:**

- agency\_id → esim\_agency.agencies(id)
- customer\_id → customers(id)
- order\_item\_id → customer\_order\_items(id)
- related\_ticket\_id → support\_tickets(id)
- provider\_id → resource.providers(id)
- provider\_package\_id → travel.provider\_packages(id)
- approved\_by → auth.users(id)
- processed\_by → auth.users(id)
- refund\_transaction\_id → payment\_transactions(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.6.4 Bảng esim\_replacement\_attachments**

**Mục đích:** File đính kèm cho yêu cầu thay thế eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|replacement\_request\_id|int4|ID yêu cầu thay thế|NOT NULL, FK → esim\_replacement\_requests(id)|
|file\_name|varchar(255)|Tên file|NOT NULL|
|file\_type|varchar(50)|Loại file|NOT NULL|
|file\_url|text|URL file|NOT NULL|
|file\_size|int4|Kích thước file|NULL|
|attachment\_type|varchar(50)|Loại đính kèm|NOT NULL|
|uploaded\_by|int4|Người upload|NOT NULL, FK → auth.users(id)|
|description|text|Mô tả|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|

**Indexes:**

- idx\_esim\_replacement\_attachments\_request\_id ON (replacement\_request\_id)
- idx\_esim\_replacement\_attachments\_type ON (attachment\_type)

**Foreign Keys:**

- replacement\_request\_id → esim\_replacement\_requests(id)
- uploaded\_by → auth.users(id)
-----
**4. RELATIONSHIPS VÀ FOREIGN KEYS**

**4.1 Relationships chính**

**Portal Configuration**

esim\_agency.agencies (1) → (1) agency\_portal\_configs

esim\_agency.agencies (1) → (N) customers

esim\_agency.agencies (1) → (N) discount\_codes

**Order Processing Flow**

customers (1) → (N) customer\_orders

customer\_orders (1) → (N) customer\_order\_items

customer\_order\_items (1) → (N) customer\_order\_item\_esims

customer\_orders (1) → (N) payment\_transactions

discount\_codes (1) → (N) discount\_code\_usage

**Support System**

customers (1) → (N) support\_tickets

support\_tickets (1) → (N) support\_ticket\_messages

agencies (1) → (N) esim\_replacement\_requests

esim\_replacement\_requests (1) → (N) esim\_replacement\_attachments

**Logging & Monitoring**

customer\_order\_item\_esims (1) → (N) order\_item\_esim\_api\_logs

customer\_order\_item\_esims (1) → (N) order\_item\_esim\_delivery\_logs

**4.2 Cross-schema References**

**To esim\_agency**

agency\_portal\_configs → esim\_agency.agencies(id)

customers → esim\_agency.agencies(id)

customer\_orders → esim\_agency.agencies(id)

customer\_orders → esim\_agency.agency\_orders(id)

customer\_order\_items → esim\_agency.agency\_order\_items(id)

discount\_codes → esim\_agency.agencies(id)

payment\_transactions → esim\_agency.agencies(id)

support\_tickets → esim\_agency.agencies(id)

esim\_replacement\_requests → esim\_agency.agencies(id)

**To other schemas**

customer\_order\_items → inventory.products(id)

customer\_order\_items → inventory.product\_variants(id)

customer\_order\_item\_esims → resource.providers(id)

customer\_order\_item\_esims → travel.provider\_packages(id)

esim\_replacement\_requests → resource.providers(id)

esim\_replacement\_requests → travel.provider\_packages(id)

**To auth schema**

agency\_portal\_configs → auth.state\_definitions(object\_type, status)

customers → auth.state\_definitions(object\_type, status)

customer\_orders → auth.state\_definitions(object\_type, status)

customer\_order\_items → auth.state\_definitions(object\_type, status)

customer\_order\_item\_esims → auth.state\_definitions(object\_type, status)

payment\_transactions → auth.state\_definitions(object\_type, status)

support\_tickets → auth.state\_definitions(object\_type, status)

esim\_replacement\_requests → auth.state\_definitions(object\_type, status)

esim\_replacement\_attachments → auth.users(id)

-----
**5. BUSINESS RULES VÀ CONSTRAINTS**

**5.1 Data Constraints**

**Portal Configuration**

- Mỗi đại lý chỉ có một cấu hình portal (agency\_portal\_configs.agency\_id UNIQUE)
- Custom domain phải unique across system
- Color codes phải theo format hex (#RRGGBB)

**Customer Management**

- Email khách hàng phải unique trong phạm vi đại lý (customers.agency\_id, email UNIQUE)
- Guest customers có thể không có password\_hash
- Registered customers phải có password\_hash

**Order Processing**

- customer\_orders.final\_amount = total\_amount - discount\_amount + tax\_amount
- customer\_order\_items.final\_price = price - discount\_amount + tax\_amount
- Order code phải unique toàn hệ thống
- Mỗi order item chỉ có thể link đến một eSIM (customer\_order\_item\_esims.order\_item\_id, esim\_id UNIQUE)

**Discount Management**

- discount\_codes.code phải unique trong phạm vi đại lý
- usage\_count không được vượt quá usage\_limit
- Discount chỉ áp dụng trong khoảng valid\_from và valid\_to
- discount\_type chỉ nhận 'PERCENTAGE' hoặc 'FIXED\_AMOUNT'

**Payment Constraints**

- Transaction code phải unique toàn hệ thống
- Amount phải > 0 cho transaction type PAYMENT
- Currency code phải theo ISO 4217

**eSIM Delivery**

- delivery\_retry\_count không được vượt quá delivery\_retry\_max
- api\_retry\_count tăng dần mỗi lần retry
- Status transitions phải tuân theo workflow định sẵn

**Support System**

- Ticket code phải unique toàn hệ thống
- sender\_type chỉ nhận 'CUSTOMER', 'AGENT', 'STAFF'
- Internal messages chỉ visible cho staff/agents

**5.2 Unique Constraints Summary**

|**Bảng**|**Unique Constraint**|
| :- | :- |
|agency\_portal\_configs|agency\_id|
|customers|(agency\_id, email)|
|customer\_orders|order\_code|
|customer\_order\_item\_esims|(order\_item\_id, esim\_id)|
|discount\_codes|(agency\_id, code)|
|discount\_code\_usage|(discount\_id, order\_id)|
|payment\_transactions|transaction\_code|
|support\_tickets|ticket\_code|
|esim\_replacement\_requests|request\_code|

**5.3 Cascade Rules**

**DELETE CASCADE**

- customers → không cascade (preserve order history)
- customer\_orders → customer\_order\_items (soft delete recommended)
- customer\_order\_items → customer\_order\_item\_esims
- discount\_codes → discount\_code\_usage
- support\_tickets → support\_ticket\_messages
- esim\_replacement\_requests → esim\_replacement\_attachments

**RESTRICT (No Cascade)**

- agencies → tất cả bảng portal (không được xóa agency khi còn data)
- customer\_orders → payment\_transactions (preserve financial records)
-----
**6. INDEXING STRATEGY**

**6.1 Performance Indexes**

**Business Logic Indexes**

-- Portal filtering

CREATE INDEX idx\_agency\_portal\_configs\_status ON agency\_portal\_configs(status);

CREATE INDEX idx\_customers\_status ON customers(status);

-- Order processing

CREATE INDEX idx\_customer\_orders\_order\_date ON customer\_orders(order\_date);

CREATE INDEX idx\_customer\_orders\_status ON customer\_orders(status);

CREATE INDEX idx\_customer\_order\_items\_status ON customer\_order\_items(status);

-- Payment tracking

CREATE INDEX idx\_payment\_transactions\_transaction\_date ON payment\_transactions(transaction\_date);

CREATE INDEX idx\_payment\_transactions\_transaction\_type ON payment\_transactions(transaction\_type);

-- Support system

CREATE INDEX idx\_support\_tickets\_priority ON support\_tickets(priority);

CREATE INDEX idx\_support\_tickets\_category ON support\_tickets(category);

-- eSIM delivery monitoring

CREATE INDEX idx\_customer\_order\_item\_esims\_delivery\_status ON customer\_order\_item\_esims(delivery\_status);

CREATE INDEX idx\_customer\_order\_item\_esims\_api\_retry\_count ON customer\_order\_item\_esims(api\_retry\_count);

**Time-based Indexes**

-- Monitoring và reporting

CREATE INDEX idx\_order\_item\_esim\_api\_logs\_created\_at ON order\_item\_esim\_api\_logs(created\_at);

CREATE INDEX idx\_order\_item\_esim\_delivery\_logs\_created\_at ON order\_item\_esim\_delivery\_logs(created\_at);

-- Discount code validity

CREATE INDEX idx\_discount\_codes\_valid\_period ON discount\_codes(valid\_from, valid\_to);

**JSONB Indexes (GIN)**

-- Request data search

CREATE INDEX idx\_order\_requests\_request\_data ON order\_requests USING GIN (request\_data);

-- Social links search

CREATE INDEX idx\_agency\_portal\_configs\_social\_links ON agency\_portal\_configs USING GIN (social\_links);

-- Payment details search

CREATE INDEX idx\_payment\_transactions\_payment\_details ON payment\_transactions USING GIN (payment\_details);

**Array Indexes (GIN)**

-- Discount applicability

CREATE INDEX idx\_discount\_codes\_applicable\_products ON discount\_codes USING GIN (applicable\_products);

CREATE INDEX idx\_discount\_codes\_applicable\_variants ON discount\_codes USING GIN (applicable\_variants);

CREATE INDEX idx\_discount\_codes\_applicable\_regions ON discount\_codes USING GIN (applicable\_regions);

**6.2 Composite Indexes**

**Agency-based Queries**

-- Customer lookup per agency

CREATE INDEX idx\_customers\_agency\_status ON customers(agency\_id, status);

-- Order lookup per agency

CREATE INDEX idx\_customer\_orders\_agency\_date ON customer\_orders(agency\_id, order\_date);

-- Support tickets per agency

CREATE INDEX idx\_support\_tickets\_agency\_status ON support\_tickets(agency\_id, status);

**Cross-reference Lookups**

-- Order to agency order mapping

CREATE INDEX idx\_customer\_orders\_agency\_order ON customer\_orders(agency\_order\_id);

-- Item to agency item mapping  

CREATE INDEX idx\_customer\_order\_items\_agency\_item ON customer\_order\_items(agency\_order\_item\_id);

-----
**7. WORKFLOW VÀ BUSINESS PROCESSES**

**7.1 Order Processing Workflow**

**Customer Order Flow**

1\. Customer places order → customer\_orders (PENDING)

2\. Payment processing → payment\_transactions

3\. Order confirmation → customer\_orders (CONFIRMED)

4\. Create agency order → link to esim\_agency.agency\_orders

5\. eSIM provisioning → customer\_order\_item\_esims

6\. eSIM delivery → delivery\_logs

7\. Order completion → customer\_orders (COMPLETED)

**eSIM Delivery Process**

1\. API Request → order\_item\_esim\_api\_logs

2\. eSIM allocation → customer\_order\_item\_esims

3\. QR code generation → qrcode\_url

4\. Email/SMS delivery → order\_item\_esim\_delivery\_logs

5\. Delivery confirmation → delivery\_status = 'DELIVERED'

**7.2 Support Workflow**

**Ticket Lifecycle**

1\. Customer creates ticket → support\_tickets (OPEN)

2\. Agent assignment → assigned\_to

3\. Message exchange → support\_ticket\_messages

4\. Issue resolution → resolution\_note

5\. Ticket closure → support\_tickets (CLOSED)

**Replacement Request Process**

1\. Issue reported → esim\_replacement\_requests (PENDING)

2\. Investigation → status = 'INVESTIGATING'

3\. Approval process → approved\_by, approved\_at

4\. New eSIM provision → replacement\_esim\_id

5\. Delivery → delivery process

6\. Completion → status = 'COMPLETED'

**7.3 Discount Management**

**Discount Application**

-- Validate discount code

SELECT \* FROM discount\_codes 

WHERE agency\_id = ? AND code = ? 

`  `AND is\_active = true

`  `AND CURRENT\_TIMESTAMP BETWEEN valid\_from AND valid\_to

`  `AND (usage\_limit IS NULL OR usage\_count < usage\_limit);

-- Apply discount

INSERT INTO discount\_code\_usage (discount\_id, order\_id, customer\_id, discount\_amount);

UPDATE discount\_codes SET usage\_count = usage\_count + 1 WHERE id = ?;


