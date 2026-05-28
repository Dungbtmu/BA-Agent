




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.364ffea1-0a6a-48c2-84df-3596c51baa5b.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Hệ thống Quản lý Travel eSIM**

**Phiên bản 1.0**
**\


**1. TỔNG QUAN HỆ THỐNG**

**1.1 Mục đích**

Hệ thống Travel được thiết kế để quản lý toàn bộ quy trình liên quan đến eSIM du lịch, bao gồm:

- **Provider Management**: Quản lý nhà cung cấp và gói dịch vụ
- **eSIM Lifecycle**: Quản lý vòng đời eSIM từ provision đến activation
- **Regional Management**: Quản lý vùng địa lý và quốc gia
- **Package Management**: Quản lý gói dữ liệu và mapping với sản phẩm
- **Order Processing**: Xử lý đơn hàng từ partner và hệ thống ngoài
- **API Integration**: Tích hợp với các nhà cung cấp thông qua API

**2. SƠ ĐỒ QUAN HỆ ENTITIES**

**2.1 Relationships chính**

resource.providers (1) → (N) provider\_packages

provider\_packages (1) → (N) product\_variant\_package\_mappings

inventory.product\_variants (1) → (N) product\_variant\_package\_mappings

regions (1) → (N) provider\_packages

regions (1) → (N) esims

inventory.warehouses (1) → (N) esims

resource.providers (1) → (N) esim\_provider\_orders

esim\_provider\_orders (1) → (N) esim\_provider\_order\_items

travel\_orders (1) → (N) travel\_order\_items

inventory.product\_variants (1) → (N) esim\_variant\_details

-----
**3. MÔ TẢ CHI TIẾT CÁC BẢNG**

**3.1 NHÓM QUẢN LÝ VÙNG ĐỊA LÝ (Regional Management)**

**3.1.1 Bảng regions**

**Mục đích:** Quản lý vùng địa lý (quốc gia, khu vực, toàn cầu)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã vùng|NOT NULL, UNIQUE|
|name|varchar(100)|Tên vùng|NOT NULL|
|type|varchar(50)|Loại vùng|NOT NULL|
|iso\_code|varchar(10)|Mã ISO|NULL|
|country\_codes|\_text|Danh sách mã quốc gia|NULL|
|continent|varchar(50)|Châu lục|NULL|
|description|text|Mô tả|NULL|
|airalo\_data|jsonb|Dữ liệu từ Airalo|NULL|
|flag|varchar(100)|URL cờ|NULL|
|avaiable\_skyboss|bool|Có sẵn Skyboss|DEFAULT false|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- CHECK: type IN ('COUNTRY', 'REGIONAL', 'GLOBAL')

**3.1.2 Bảng md\_countries**

**Mục đích:** Master data các quốc gia

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|code|varchar(8)|Mã quốc gia|NOT NULL, PRIMARY KEY|
|capital|varchar(64)|Thủ đô|NULL|
|continent|varchar(16)|Châu lục|NULL|
|flag|varchar(64)|URL cờ|NULL|
|banner|varchar(64)|URL banner|NULL|
|iso|varchar(20)|Mã ISO|NULL|
|name|varchar(64)|Tên quốc gia|NULL|

**3.1.3 Bảng compatible\_devices**

**Mục đích:** Danh sách thiết bị tương thích

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|int4|Khóa chính|PRIMARY KEY|
|brand|varchar|Thương hiệu|NULL|
|name|varchar|Tên thiết bị|NULL|
|model|varchar|Model|NULL|
|os|varchar|Hệ điều hành|NULL|
|note|text|Ghi chú (EN)|NULL|
|note\_vi|text|Ghi chú (VI)|NULL|
|is\_travel\_esims|bool|Hỗ trợ travel eSIM|NULL|

-----
**3.2 NHÓM QUẢN LÝ NHÀ CUNG CẤP (Provider Management)**

**3.2.1 Bảng provider\_packages**

**Mục đích:** Gói dịch vụ từ nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|package\_identifier|varchar(100)|Mã gói từ provider|NOT NULL|
|name|varchar(255)|Tên gói|NOT NULL|
|region\_id|int4|ID vùng|FK → regions(id)|
|price|numeric(15,2)|Giá gói|NOT NULL|
|currency|varchar(10)|Loại tiền tệ|NOT NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|package\_type|varchar(50)|Loại gói|NULL|
|geographic\_type|varchar(20)|Loại địa lý|NULL|
|operator|varchar(255)|Nhà mạng|NULL|
|original\_data|jsonb|Dữ liệu gốc từ API|NOT NULL|
|countries|jsonb|Danh sách quốc gia|NULL|
|version|varchar(8)|Phiên bản|NOT NULL|
|status|varchar(100)|Trạng thái|NOT NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'TRAVEL\_ESIM'|
|description|text|Mô tả|NULL|
|extras|jsonb|Thông tin bổ sung|NULL|
|is\_topup|bool|Là gói topup|DEFAULT false|
|net\_price|numeric(15,2)|Giá net|NULL|
|is\_vja|bool|Dành cho VietJet Air|DEFAULT false|
|vja\_country\_code|varchar(10)|Mã quốc gia VJA|NULL|
|vja\_package\_id|varchar(20)|Mã gói VJA|NULL|
|vja\_slug|varchar(20)|Slug VJA|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (provider\_id, package\_identifier)

**Foreign Keys:**

- provider\_id → resource.providers(id)
- region\_id → regions(id)

**3.2.2 Bảng provider\_packages\_history**

**Mục đích:** Lịch sử thay đổi gói dịch vụ

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|package\_id|int4|ID gói|NOT NULL, FK → provider\_packages(id)|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|package\_identifier|varchar(100)|Mã gói|NOT NULL|
|name|varchar(255)|Tên gói|NOT NULL|
|price|numeric(15,2)|Giá gói|NOT NULL|
|currency|varchar(10)|Loại tiền tệ|NOT NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|package\_type|varchar(50)|Loại gói|NULL|
|geographic\_type|varchar(20)|Loại địa lý|NULL|
|operator|varchar(255)|Nhà mạng|NULL|
|original\_data|jsonb|Dữ liệu gốc|NOT NULL|
|countries|jsonb|Danh sách quốc gia|NULL|
|version|varchar(8)|Phiên bản|NOT NULL|
|previous\_status|varchar(100)|Trạng thái trước|NULL|
|new\_status|varchar(100)|Trạng thái mới|NOT NULL|
|change\_reason|text|Lý do thay đổi|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|

**Foreign Keys:**

- package\_id → provider\_packages(id)
- provider\_id → resource.providers(id)

**3.2.3 Bảng provider\_packages\_temp**

**Mục đích:** Bảng tạm cho việc sync gói từ API

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|package\_identifier|varchar(100)|Mã gói|NOT NULL|
|name|varchar(255)|Tên gói|NOT NULL|
|price|numeric(15,2)|Giá gói|NOT NULL|
|currency|varchar(10)|Loại tiền tệ|NOT NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|package\_type|varchar(50)|Loại gói|NULL|
|geographic\_type|varchar(20)|Loại địa lý|NULL|
|operator|varchar(255)|Nhà mạng|NULL|
|original\_data|jsonb|Dữ liệu gốc|NOT NULL|
|countries|jsonb|Danh sách quốc gia|NULL|
|region\_id|int4|ID vùng|FK → regions(id)|
|is\_topup|bool|Là gói topup|DEFAULT false|
|net\_price|numeric(15,2)|Giá net|NULL|
|fetched\_at|timestamptz|Thời gian fetch|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|has\_changes|bool|Có thay đổi|DEFAULT false|
|change\_comments|text|Nhận xét thay đổi|NULL|
|is\_updated|bool|Đã cập nhật|DEFAULT false|

**Constraints:**

- UNIQUE: (provider\_id, package\_identifier)

**3.2.4 Bảng provider\_api\_configs**

**Mục đích:** Cấu hình API cho từng nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|config\_key|varchar(100)|Khóa cấu hình|NOT NULL|
|config\_value|text|Giá trị cấu hình|NULL|
|description|text|Mô tả|NULL|
|is\_encrypted|bool|Có mã hóa|NOT NULL, DEFAULT false|
|environment|varchar(20)|Môi trường|NOT NULL, DEFAULT 'production'|
|expired\_at|timestamptz|Ngày hết hạn|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (provider\_id, config\_key, environment)

**Foreign Keys:**

- provider\_id → resource.providers(id)

**3.2.5 Bảng product\_variant\_package\_mappings**

**Mục đích:** Ánh xạ giữa biến thể sản phẩm và gói provider

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|variant\_id|int4|ID biến thể sản phẩm|NOT NULL, FK → inventory.product\_variants(id)|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|package\_id|int4|ID gói|NOT NULL, FK → provider\_packages(id)|
|priority|int4|Thứ tự ưu tiên|NOT NULL, DEFAULT 0|
|is\_active|bool|Trạng thái hoạt động|NOT NULL, DEFAULT true|
|is\_topup|bool|Là mapping topup|NOT NULL, DEFAULT false|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (variant\_id, provider\_id, package\_id)

**Foreign Keys:**

- variant\_id → inventory.product\_variants(id)
- provider\_id → resource.providers(id)
- package\_id → provider\_packages(id)
-----
**3.3 NHÓM QUẢN LÝ CHI TIẾT SẢN PHẨM (Product Details)**

**3.3.1 Bảng esim\_variant\_details**

**Mục đích:** Chi tiết kỹ thuật của biến thể eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|variant\_id|int4|ID biến thể|NOT NULL, UNIQUE, FK → inventory.product\_variants(id) CASCADE|
|region\_id|int4|ID vùng|FK → regions(id)|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|NOT NULL, DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NOT NULL|
|max\_speed|varchar(20)|Tốc độ tối đa|NULL|
|network\_providers|text|Nhà mạng|NULL|
|fair\_usage\_policy|text|Chính sách sử dụng|NULL|
|auto\_renewal|bool|Tự động gia hạn|NOT NULL, DEFAULT false|
|additional\_features|jsonb|Tính năng bổ sung|NULL|
|technical\_instructions|text|Hướng dẫn kỹ thuật|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_esim\_variant\_details\_variant\_id ON (variant\_id)
- idx\_esim\_variant\_details\_data\_validity ON (data\_amount, validity\_days)

**Foreign Keys:**

- variant\_id → inventory.product\_variants(id) ON DELETE CASCADE
- region\_id → regions(id)
-----
**3.4 NHÓM QUẢN LÝ ESIM (eSIM Lifecycle)**

**3.4.1 Bảng esims**

**Mục đích:** Quản lý eSIM chính (optimized version)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|iccid|varchar(30)|ICCID|NOT NULL, UNIQUE|
|msisdn|varchar(20)|Số điện thoại|NULL|
|product\_id|int4|ID sản phẩm|NOT NULL, FK → inventory.products(id)|
|variant\_id|int4|ID biến thể|FK → inventory.product\_variants(id)|
|region\_id|int4|ID vùng|NOT NULL, FK → regions(id)|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|warehouse\_id|int4|ID kho|NOT NULL, FK → inventory.warehouses(id)|
|qrcode|varchar(255)|Mã QR|NOT NULL|
|qrcode\_url|text|URL QR code|NOT NULL|
|installation\_data|jsonb|Dữ liệu cài đặt|NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|net\_price|numeric(10,2)|Giá net|NULL|
|currency|varchar(3)|Loại tiền tệ|DEFAULT 'USD'|
|exchange\_rate|numeric(15,6)|Tỷ giá|DEFAULT 1|
|technical\_data|jsonb|Dữ liệu kỹ thuật|NULL|
|sharing\_data|jsonb|Dữ liệu chia sẻ|NULL|
|package\_data|jsonb|Dữ liệu gói|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ESIM'|
|recycled|bool|Đã tái chế|DEFAULT false|
|is\_roaming|bool|Roaming|DEFAULT false|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expire\_date|timestamptz|Ngày hết hạn|NULL|
|start\_date|timestamptz|Ngày bắt đầu|NULL|
|end\_date|timestamptz|Ngày kết thúc|NULL|
|last\_status\_change|timestamptz|Lần đổi trạng thái cuối|DEFAULT CURRENT\_TIMESTAMP|
|recycled\_date|timestamptz|Ngày tái chế|NULL|
|inventory\_item\_id|int4|ID inventory item|NULL|
|batch\_id|int4|ID batch|NULL|
|mapping\_id|int4|ID mapping|FK → product\_variant\_package\_mappings(id)|
|provider\_order\_id|int4|ID đơn hàng provider|NULL|
|provider\_order\_item\_id|int4|ID item đơn hàng|NULL|
|order\_code|varchar(50)|Mã đơn hàng|NULL|
|confirmation\_code|varchar(100)|Mã xác nhận|NULL|
|voucher\_code|varchar(100)|Mã voucher|NULL|
|airalo\_code|varchar(100)|Mã Airalo|NULL|
|direct\_apple\_installation\_url|varchar|URL cài đặt Apple|NULL|
|brand\_settings\_name|varchar|Tên brand settings|NULL|
|sharing|jsonb|Dữ liệu chia sẻ|NULL|
|available\_topups|jsonb|Topup có sẵn|NULL|
|imsis|jsonb|Danh sách IMSI|NULL|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_esims\_optimized\_product\_id ON (product\_id)
- idx\_esims\_optimized\_variant\_id ON (variant\_id)
- idx\_esims\_optimized\_region\_id ON (region\_id)
- idx\_esims\_optimized\_provider\_id ON (provider\_id)
- idx\_esims\_optimized\_warehouse\_id ON (warehouse\_id)
- idx\_esims\_optimized\_status ON (status)
- idx\_esims\_optimized\_mapping\_id ON (mapping\_id)
- idx\_esims\_optimized\_recycled ON (recycled)
- idx\_esims\_optimized\_is\_roaming ON (is\_roaming)
- idx\_esims\_optimized\_activation\_date ON (activation\_date)
- idx\_esims\_optimized\_expire\_date ON (expire\_date)
- idx\_esims\_optimized\_created\_at ON (created\_at)

**Foreign Keys:**

- product\_id → inventory.products(id)
- variant\_id → inventory.product\_variants(id)
- region\_id → regions(id)
- provider\_id → resource.providers(id)
- warehouse\_id → inventory.warehouses(id)
- mapping\_id → product\_variant\_package\_mappings(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.4.2 Bảng partner\_esims**

**Mục đích:** eSIM từ partner (raw data từ API)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|partner\_sim\_id|varchar(100)|ID SIM từ partner|NULL|
|sim\_type|varchar(50)|Loại SIM|NOT NULL, DEFAULT 'prepaid'|
|iccid|varchar(30)|ICCID|NULL|
|msisdn|varchar(30)|MSISDN|NULL|
|qrcode|text|Mã QR|NOT NULL|
|qrcode\_url|text|URL QR code|NULL|
|activation\_code|varchar(200)|Mã kích hoạt|NULL|
|lpa|varchar(100)|LPA|NULL|
|matching\_id|varchar(100)|ID matching|NULL|
|direct\_apple\_installation\_url|text|URL cài đặt Apple|NULL|
|status|varchar(50)|Trạng thái|NOT NULL, DEFAULT 'PROVISIONED'|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ORDER\_ITEM\_ESIM'|
|apn\_type|varchar(50)|Loại APN|NULL|
|apn\_value|varchar(100)|Giá trị APN|NULL|
|apn\_data|jsonb|Dữ liệu APN|NULL|
|region\_id|int4|ID vùng|FK → regions(id)|
|package\_id|int4|ID gói|FK → provider\_packages(id)|
|package\_identifier|varchar|Mã gói|NULL|
|package\_data|jsonb|Dữ liệu gói|NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expire\_date|timestamptz|Ngày hết hạn|NULL|
|start\_date|timestamptz|Ngày bắt đầu|NULL|
|end\_date|timestamptz|Ngày kết thúc|NULL|
|recycled\_date|timestamptz|Ngày tái chế|NULL|
|is\_roaming|bool|Roaming|DEFAULT false|
|recycled|bool|Đã tái chế|DEFAULT false|
|brand\_settings\_name|varchar(100)|Tên brand settings|NULL|
|imsis|jsonb|Danh sách IMSI|NULL|
|sharing|jsonb|Dữ liệu chia sẻ|NULL|
|available\_topups|jsonb|Topup có sẵn|NULL|
|confirmation\_code|varchar(100)|Mã xác nhận|NULL|
|voucher\_code|varchar(100)|Mã voucher|NULL|
|airalo\_code|varchar(100)|Mã Airalo|NULL|
|raw\_data|jsonb|Dữ liệu thô|NULL|
|notes|text|Ghi chú|NULL|
|technical\_instructions|text|Hướng dẫn kỹ thuật|NULL|
|last\_synced\_at|timestamptz|Lần sync cuối|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- UNIQUE: (iccid, qrcode)

**Indexes:**

- idx\_partner\_esims\_provider\_id ON (provider\_id)
- idx\_partner\_esims\_iccid ON (iccid)
- idx\_partner\_esims\_msisdn ON (msisdn)
- idx\_partner\_esims\_status ON (status)
- idx\_partner\_esims\_sim\_type ON (sim\_type)
- idx\_partner\_esims\_region\_id ON (region\_id)
- idx\_partner\_esims\_package\_id ON (package\_id)
- idx\_partner\_esims\_activation\_date ON (activation\_date)
- idx\_partner\_esims\_expire\_date ON (expire\_date)

**Foreign Keys:**

- provider\_id → resource.providers(id)
- region\_id → regions(id)
- package\_id → provider\_packages(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.4.3 Bảng travel\_esims**

**Mục đích:** eSIM legacy system (tương thích ngược)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|product\_id|int4|ID sản phẩm|NOT NULL|
|region\_id|int4|ID vùng|FK → regions(id)|
|iccid|varchar(30)|ICCID|NOT NULL, UNIQUE|
|msisdn|varchar(20)|MSISDN|NULL|
|lpa|varchar(255)|LPA|NOT NULL|
|qrcode|text|Mã QR|NOT NULL|
|qrcode\_url|text|URL QR code|NOT NULL|
|direct\_apple\_installation\_url|text|URL cài đặt Apple|NULL|
|brand\_settings\_name|varchar(100)|Tên brand settings|NULL|
|imsis|jsonb|Danh sách IMSI|NULL|
|sharing|jsonb|Dữ liệu chia sẻ|NULL|
|package\_data|jsonb|Dữ liệu gói|NULL|
|available\_topups|jsonb|Topup có sẵn|NULL|
|status|varchar(50)|Trạng thái|DEFAULT 'NOT\_ACTIVE'|
|provider\_id|int4|ID nhà cung cấp|DEFAULT 2|
|provider\_order\_id|int4|ID đơn hàng provider|NULL|
|provider\_order\_item\_id|int4|ID item đơn hàng|NULL|
|order\_code|varchar(50)|Mã đơn hàng|NULL|
|batch\_id|int4|ID batch|NULL|
|warehouse\_id|int4|ID kho|NULL|
|inventory\_item\_id|int4|ID inventory item|NULL|
|last\_status\_change|timestamptz|Lần đổi trạng thái cuối|DEFAULT CURRENT\_TIMESTAMP|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expire\_date|timestamptz|Ngày hết hạn|NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|start\_date|timestamptz|Ngày bắt đầu|NULL|
|end\_date|timestamptz|Ngày kết thúc|NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ESIM'|
|notes|text|Ghi chú|NULL|
|technical\_instructions|text|Hướng dẫn kỹ thuật|NULL|
|package\_id|varchar(30)|ID gói|NULL|
|order\_item\_id|int4|ID order item|NULL|
|matching\_id|varchar(100)|ID matching|NULL|
|email|varchar|Email|NULL|
|variant\_id|int4|ID biến thể|NULL|
|mapping\_id|int4|ID mapping|NULL|
|ordertype|varchar(100)|Loại đơn hàng|DEFAULT 'WZ'|
|price|numeric(15,2)|Giá|NULL|
|data\_remaining|int4|Data còn lại|NULL|
|currency|varchar(5)|Loại tiền tệ|DEFAULT 'VND'|
|provider\_order\_code|varchar(20)|Mã đơn hàng provider|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**3.4.4 Bảng travel\_esims\_topup**

**Mục đích:** Topup cho travel eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|product\_id|int4|ID sản phẩm|NOT NULL|
|status|varchar(50)|Trạng thái|NULL|
|provider\_id|int4|ID nhà cung cấp|DEFAULT 2|
|provider\_order\_id|int4|ID đơn hàng provider|NULL|
|provider\_order\_item\_id|int4|ID item đơn hàng|NULL|
|order\_code|varchar(50)|Mã đơn hàng|NULL|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expire\_date|timestamptz|Ngày hết hạn|NULL|
|data\_amount|numeric(10,2)|Dung lượng data|NULL|
|data\_unit|varchar(5)|Đơn vị data|DEFAULT 'GB'|
|validity\_days|int4|Số ngày hiệu lực|NULL|
|start\_date|timestamptz|Ngày bắt đầu|NULL|
|end\_date|timestamptz|Ngày kết thúc|NULL|
|notes|text|Ghi chú|NULL|
|technical\_instructions|text|Hướng dẫn kỹ thuật|NULL|
|package\_id|varchar|ID gói|NULL|
|order\_item\_id|int4|ID order item|NULL|
|email|varchar|Email|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

-----
**3.5 NHÓM QUẢN LÝ ĐƠN HÀNG PROVIDER (Provider Orders)**

**3.5.1 Bảng esim\_provider\_orders**

**Mục đích:** Đơn hàng gửi tới nhà cung cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|code|varchar(50)|Mã đơn hàng|NOT NULL, UNIQUE|
|order\_type|varchar(50)|Loại đơn hàng|NOT NULL|
|provider\_id|int4|ID nhà cung cấp|NOT NULL, FK → resource.providers(id)|
|quantity|int4|Số lượng|NOT NULL, DEFAULT 1|
|package\_id|int4|ID gói|NOT NULL, FK → provider\_packages(id)|
|price|numeric(15,2)|Giá|NOT NULL|
|currency|varchar(3)|Loại tiền tệ|NOT NULL, DEFAULT 'USD'|
|description|text|Mô tả|NULL|
|status|varchar(50)|Trạng thái|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ESIM\_PROVIDER\_ORDER'|
|provider\_order\_id|varchar(100)|ID đơn hàng từ provider|NULL|
|provider\_response|jsonb|Response từ provider|NULL|
|esim\_type|varchar(50)|Loại eSIM|NOT NULL, DEFAULT 'Prepaid'|
|reference\_number|varchar(100)|Số tham chiếu|NULL|
|region\_id|int4|ID vùng|FK → regions(id)|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Constraints:**

- CHECK: order\_type IN ('initial', 'topup')

**Foreign Keys:**

- provider\_id → resource.providers(id)
- package\_id → provider\_packages(id)
- region\_id → regions(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.5.2 Bảng esim\_provider\_order\_items**

**Mục đích:** Chi tiết items trong đơn hàng provider

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_id|int4|ID đơn hàng|NOT NULL, FK → esim\_provider\_orders(id)|
|iccid|varchar(100)|ICCID|NULL|
|package\_id|int4|ID gói|NOT NULL, FK → provider\_packages(id)|
|status|varchar(50)|Trạng thái|NOT NULL|
|object\_type|varchar(100)|Loại đối tượng|NOT NULL, DEFAULT 'ESIM\_PROVIDER\_ORDER\_ITEM'|
|provider\_item\_id|varchar(100)|ID item từ provider|NULL|
|provider\_response|jsonb|Response từ provider|NULL|
|inventory\_transaction\_id|int4|ID giao dịch inventory|FK → inventory.inventory\_transactions(id)|
|esim\_id|int4|ID eSIM|NULL|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|NOT NULL, DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Foreign Keys:**

- order\_id → esim\_provider\_orders(id)
- package\_id → provider\_packages(id)
- inventory\_transaction\_id → inventory.inventory\_transactions(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)
-----
**3.6 NHÓM QUẢN LÝ ĐƠN HÀNG TRAVEL (Travel Orders)**

**3.6.1 Bảng travel\_orders**

**Mục đích:** Đơn hàng từ hệ thống bên ngoài

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|order\_code|varchar(50)|Mã đơn hàng|NOT NULL|
|source\_system|varchar(50)|Hệ thống nguồn|NOT NULL|
|customer\_email|varchar(100)|Email khách hàng|NOT NULL|
|customer\_name|varchar(100)|Tên khách hàng|NULL|
|customer\_phone|varchar(20)|SĐT khách hàng|NULL|
|total\_amount|numeric(15,2)|Tổng tiền|NULL|
|currency|varchar(3)|Loại tiền tệ|DEFAULT 'VND'|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'TRAVEL\_ORDER'|
|source\_created\_at|timestamp|Thời gian tạo từ nguồn|NULL|
|source|varchar(20)|Nguồn|NULL|
|agent\_code|varchar(20)|Mã đại lý|NULL|
|departure\_code|varchar(5)|Mã điểm đi|NULL|
|destination\_code|varchar(5)|Mã điểm đến|NULL|
|country\_code|varchar(5)|Mã quốc gia|NULL|
|skyjoy\_code|varchar(20)|Mã SkyJoy|NULL|
|created\_at|timestamp|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamp|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|varchar|Người tạo|NULL|
|updated\_by|varchar|Người cập nhật|NULL|

**Constraints:**

- UNIQUE: (order\_code, source\_system)

**Indexes:**

- idx\_travel\_orders\_order\_code ON (order\_code)
- idx\_travel\_orders\_email ON (customer\_email)
- idx\_travel\_orders\_source ON (source\_system, order\_code)

**3.6.2 Bảng travel\_order\_items**

**Mục đích:** Chi tiết sản phẩm trong đơn hàng travel

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|travel\_order\_id|int4|ID đơn hàng travel|NOT NULL, FK → travel\_orders(id)|
|order\_item\_id|int4|ID item từ nguồn|NULL|
|order\_code|varchar(50)|Mã đơn hàng|NOT NULL|
|product\_id|int4|ID sản phẩm|NOT NULL|
|package\_id|varchar(100)|ID gói|NULL|
|quantity|int4|Số lượng|NOT NULL, DEFAULT 1|
|price|numeric(15,2)|Giá|NULL|
|currency|varchar(3)|Loại tiền tệ|DEFAULT 'VND'|
|provision\_status|varchar(50)|Trạng thái provision|DEFAULT 'PENDING'|
|retry\_count|int4|Số lần retry|DEFAULT 0|
|max\_retry\_count|int4|Số retry tối đa|DEFAULT 3|
|last\_error\_message|text|Thông báo lỗi cuối|NULL|
|last\_retry\_at|timestamp|Lần retry cuối|NULL|
|provisioned\_count|int4|Số đã provision|DEFAULT 0|
|failed\_count|int4|Số thất bại|DEFAULT 0|
|api\_request\_data|jsonb|Dữ liệu request API|NULL|
|api\_response\_data|jsonb|Dữ liệu response API|NULL|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'TRAVEL\_ORDER\_ITEM'|
|created\_at|timestamp|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamp|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|

**Constraints:**

- UNIQUE: (order\_item\_id, order\_code)

**Indexes:**

- idx\_travel\_order\_items\_travel\_order ON (travel\_order\_id)
- idx\_travel\_order\_items\_order\_code ON (order\_code)
- idx\_travel\_order\_items\_provision\_status ON (provision\_status)
- idx\_travel\_order\_items\_source ON (order\_code, order\_item\_id)

**Foreign Keys:**

- travel\_order\_id → travel\_orders(id)
-----
**3.7 NHÓM HỖ TRỢ VÀ TIỆN ÍCH (Support & Utilities)**

**3.7.1 Bảng travel\_log**

**Mục đích:** Log hoạt động của hệ thống travel

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|ref\_id|varchar(100)|ID tham chiếu|NULL|
|sub\_ref\_id|varchar(100)|ID tham chiếu phụ|NULL|
|log\_type|varchar(100)|Loại log|NULL|
|log\_date|timestamp|Ngày log|DEFAULT CURRENT\_TIMESTAMP|
|content|text|Nội dung log|NULL|

**4. RELATIONSHIPS VÀ FOREIGN KEYS**

**4.1 Relationships chính**

**Provider & Package Hierarchy**

resource.providers (1) → (N) provider\_packages

resource.providers (1) → (N) provider\_api\_configs

provider\_packages (1) → (N) product\_variant\_package\_mappings

provider\_packages (1) → (N) esim\_provider\_orders

**Product Integration**

inventory.products (1) → (N) esims

inventory.product\_variants (1) → (N) esims

inventory.product\_variants (1) → (N) esim\_variant\_details

inventory.product\_variants (1) → (N) product\_variant\_package\_mappings

**Regional Management**

regions (1) → (N) provider\_packages

regions (1) → (N) esims

regions (1) → (N) partner\_esims

regions (1) → (N) esim\_variant\_details

**Order Processing**

travel\_orders (1) → (N) travel\_order\_items

esim\_provider\_orders (1) → (N) esim\_provider\_order\_items

**4.2 Cross-schema References**

**To inventory schema**

esims → inventory.products(id)

esims → inventory.product\_variants(id)

esims → inventory.warehouses(id)

product\_variant\_package\_mappings → inventory.product\_variants(id)

esim\_variant\_details → inventory.product\_variants(id)

esim\_provider\_order\_items → inventory.inventory\_transactions(id)

**To resource schema**

provider\_packages → resource.providers(id)

provider\_api\_configs → resource.providers(id)

product\_variant\_package\_mappings → resource.providers(id)

esims → resource.providers(id)

partner\_esims → resource.providers(id)

esim\_provider\_orders → resource.providers(id)

**To auth schema**

esims → auth.state\_definitions(object\_type, status)

partner\_esims → auth.state\_definitions(object\_type, status)

esim\_provider\_orders → auth.state\_definitions(object\_type, status)

esim\_provider\_order\_items → auth.state\_definitions(object\_type, status)

-----
**5. BUSINESS RULES VÀ CONSTRAINTS**

**5.1 Data Constraints**

**Regional Management**

- regions.type chỉ nhận: COUNTRY, REGIONAL, GLOBAL
- Country codes lưu dưới dạng array text
- ISO codes phải tuân theo chuẩn quốc tế

**Provider Packages**

- provider\_packages.package\_identifier phải unique trong phạm vi provider
- Version format: semantic versioning (x.x.x.x)
- Original data phải lưu trữ full response từ API

**eSIM Lifecycle**

- ICCID phải unique toàn hệ thống
- Status transitions phải tuân theo workflow định sẵn
- Recycled eSIMs có thể tái sử dụng với recycled\_date

**Package Mappings**

- Mỗi variant có thể map với nhiều packages từ nhiều providers
- Priority xác định thứ tự ưu tiên khi chọn package
- is\_topup flag phân biệt initial package và topup package

**Order Processing**

- esim\_provider\_orders.order\_type chỉ nhận: initial, topup
- Travel orders phải unique theo (order\_code, source\_system)
- Retry mechanism với max\_retry\_count

**5.2 Unique Constraints Summary**

|**Bảng**|**Unique Constraint**|
| :- | :- |
|regions|code|
|md\_countries|code|
|provider\_packages|(provider\_id, package\_identifier)|
|provider\_packages\_temp|(provider\_id, package\_identifier)|
|provider\_api\_configs|(provider\_id, config\_key, environment)|
|product\_variant\_package\_mappings|(variant\_id, provider\_id, package\_id)|
|esim\_variant\_details|variant\_id|
|esims|iccid|
|partner\_esims|(iccid, qrcode)|
|travel\_esims|iccid|
|esim\_provider\_orders|code|
|travel\_orders|(order\_code, source\_system)|
|travel\_order\_items|(order\_item\_id, order\_code)|

**5.3 Cascade Rules**

**DELETE CASCADE**

- inventory.product\_variants → esim\_variant\_details
- esim\_provider\_orders → esim\_provider\_order\_items
- travel\_orders → travel\_order\_items

**RESTRICT (No Cascade)**

- resource.providers → tất cả bảng liên quan (preserve data integrity)
- regions → esims, provider\_packages (không được xóa region khi còn dữ liệu)
-----
**6. INDEXING STRATEGY**

**6.1 Performance Indexes**

**eSIM Operations**

sql

*-- eSIM status và lifecycle*

CREATE INDEX idx\_esims\_status\_warehouse ON esims(status, warehouse\_id);

CREATE INDEX idx\_esims\_provider\_region ON esims(provider\_id, region\_id);

CREATE INDEX idx\_esims\_available ON esims(status, recycled) WHERE status = 'AVAILABLE';

*-- Partner eSIM tracking*

CREATE INDEX idx\_partner\_esims\_provider\_status ON partner\_esims(provider\_id, status);

CREATE INDEX idx\_partner\_esims\_package\_region ON partner\_esims(package\_id, region\_id);

**Package Management**

sql

*-- Package lookups*

CREATE INDEX idx\_provider\_packages\_provider\_active ON provider\_packages(provider\_id, is\_active);

CREATE INDEX idx\_provider\_packages\_region\_type ON provider\_packages(region\_id, package\_type);

*-- Mapping efficiency*

CREATE INDEX idx\_variant\_package\_mappings\_variant\_active ON product\_variant\_package\_mappings(variant\_id, is\_active);

CREATE INDEX idx\_variant\_package\_mappings\_priority ON product\_variant\_package\_mappings(variant\_id, priority);

**Order Processing**

sql

*-- Travel order tracking*

CREATE INDEX idx\_travel\_order\_items\_provision\_retry ON travel\_order\_items(provision\_status, retry\_count);

CREATE INDEX idx\_travel\_orders\_source\_date ON travel\_orders(source\_system, created\_at);

*-- Provider order monitoring*

CREATE INDEX idx\_esim\_provider\_orders\_provider\_status ON esim\_provider\_orders(provider\_id, status);

**6.2 JSONB Indexes (GIN)**

sql

*-- Package data search*

CREATE INDEX idx\_provider\_packages\_countries ON provider\_packages USING GIN (countries);

CREATE INDEX idx\_provider\_packages\_original\_data ON provider\_packages USING GIN (original\_data);

*-- eSIM technical data*

CREATE INDEX idx\_esims\_technical\_data ON esims USING GIN (technical\_data);

CREATE INDEX idx\_partner\_esims\_raw\_data ON partner\_esims USING GIN (raw\_data);

*-- Order API data*

CREATE INDEX idx\_travel\_order\_items\_api\_data ON travel\_order\_items USING GIN (api\_request\_data);



