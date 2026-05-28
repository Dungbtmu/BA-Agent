




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.34ef7c1b-0fbf-4adc-ba3a-7b82decdc8e4.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Hệ thống Quản lý tài nguyên**

**Phiên bản 1.0**
**\


**1. TỔNG QUAN HỆ THỐNG**

Hệ thống Resource được thiết kế để quản lý tài nguyên viễn thông, bao gồm:

- **MSISDN Management**: Quản lý số điện thoại và pattern
- **SIM Management**: Quản lý SIM vật lý và eSIM
- **Number Pattern Management**: Quản lý mẫu số và pricing
- **Provider Management**: Quản lý nhà cung cấp
- **Batch Processing**: Xử lý hàng loạt import/export tài nguyên
- **State Management**: Quản lý trạng thái lifecycle của tài nguyên
- **Price Management**: Quản lý giá theo pattern và thời gian

**2. SƠ ĐỒ QUAN HỆ ENTITIES**

**2.1 Relationships chính**

providers (1) → (N) sim\_batches providers (1) → (N) number\_range\_batches number\_patterns (1) → (N) msisdns number\_patterns (1) → (N) number\_pattern\_prices sim\_batches (1) → (N) sims number\_range\_batches (1) → (N) msisdns number\_pattern\_groups (1) → (N) number\_pattern\_subgroups number\_pattern\_subgroups (N) ← → (N) number\_patterns number\_pattern\_tags (N) ← → (N) number\_patterns

**3. MÔ TẢ CHI TIẾT CÁC BẢNG**

**3.1 NHÓM QUẢN LÝ NHÀ CUNG CẤP (Provider Management)**

**3.1.1 Bảng providers**

**Mục đích:** Quản lý thông tin nhà cung cấp SIM và MSISDN

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|provider\_code|varchar(20)|Mã nhà cung cấp|NOT NULL, UNIQUE|
|name|varchar(100)|Tên nhà cung cấp|NOT NULL|
|contact\_person|varchar(100)|Người liên hệ|NULL|
|phone|varchar(20)|Số điện thoại|NULL|
|email|varchar(100)|Email|NULL|
|address|text|Địa chỉ|NULL|
|website|varchar(255)|Website|NULL|
|description|text|Mô tả|NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_providers\_provider\_code ON (provider\_code)
- idx\_providers\_is\_active ON (is\_active)

**3.2 NHÓM QUẢN LÝ PATTERN SỐ (Number Pattern Management)**

**3.2.1 Bảng number\_patterns**

**Mục đích:** Định nghĩa các mẫu số điện thoại

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|pattern\_code|varchar(10)|Mã pattern|NOT NULL, UNIQUE|
|name|varchar(200)|Tên pattern|NOT NULL|
|format\_pattern|varchar(200)|Pattern hiển thị|NULL|
|regexp\_pattern|text|Pattern regex|NOT NULL|
|additional\_condition|text|Điều kiện bổ sung|NULL|
|sql\_pattern|text|Pattern SQL|NULL|
|priority\_level|int4|Mức độ ưu tiên|NOT NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|FK → auth.users(id)|
|updated\_by|int4|Người cập nhật|FK → auth.users(id)|

**Indexes:**

- idx\_number\_patterns\_pattern\_code ON (pattern\_code)
- idx\_number\_patterns\_is\_active ON (is\_active)
- idx\_number\_patterns\_priority\_level ON (priority\_level)

**Foreign Keys:**

- created\_by → auth.users(id)
- updated\_by → auth.users(id)

**3.2.2 Bảng number\_pattern\_groups**

**Mục đích:** Phân nhóm các pattern số

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|group\_code|varchar(20)|Mã nhóm|NOT NULL, UNIQUE|
|name|varchar(100)|Tên nhóm|NOT NULL|
|description|text|Mô tả nhóm|NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_number\_pattern\_groups\_group\_code ON (group\_code)
- idx\_number\_pattern\_groups\_is\_active ON (is\_active)

**3.2.3 Bảng number\_pattern\_subgroups**

**Mục đích:** Phân nhóm con cho pattern số

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|subgroup\_code|varchar(20)|Mã nhóm con|NOT NULL, UNIQUE|
|name|varchar(100)|Tên nhóm con|NOT NULL|
|group\_id|int4|ID nhóm cha|NOT NULL, FK → number\_pattern\_groups(id)|
|description|text|Mô tả|NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_number\_pattern\_subgroups\_subgroup\_code ON (subgroup\_code)
- idx\_number\_pattern\_subgroups\_group\_id ON (group\_id)
- idx\_number\_pattern\_subgroups\_is\_active ON (is\_active)

**Foreign Keys:**

- group\_id → number\_pattern\_groups(id)

**3.2.4 Bảng number\_pattern\_tags**

**Mục đích:** Gắn tag cho pattern số

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|tag\_code|varchar(20)|Mã tag|NOT NULL, UNIQUE|
|name|varchar(100)|Tên tag|NOT NULL|
|description|text|Mô tả tag|NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_number\_pattern\_tags\_tag\_code ON (tag\_code)
- idx\_number\_pattern\_tags\_is\_active ON (is\_active)

**3.3 NHÓM QUAN HỆ PATTERN (Pattern Relations)**

**3.3.1 Bảng number\_pattern\_subgroup\_relations**

**Mục đích:** Quan hệ N-N giữa pattern và subgroup

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|pattern\_id|int4|ID pattern|NOT NULL, FK → number\_patterns(id)|
|subgroup\_id|int4|ID subgroup|NOT NULL, FK → number\_pattern\_subgroups(id)|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|

**Constraints:**

- UNIQUE: (pattern\_id, subgroup\_id)

**Indexes:**

- idx\_number\_pattern\_subgroup\_relations\_pattern\_id ON (pattern\_id)
- idx\_number\_pattern\_subgroup\_relations\_subgroup\_id ON (subgroup\_id)

**Foreign Keys:**

- pattern\_id → number\_patterns(id)
- subgroup\_id → number\_pattern\_subgroups(id)

**3.3.2 Bảng number\_pattern\_tag\_relations**

**Mục đích:** Quan hệ N-N giữa pattern và tag

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|pattern\_id|int4|ID pattern|NOT NULL, FK → number\_patterns(id)|
|tag\_id|int4|ID tag|NOT NULL, FK → number\_pattern\_tags(id)|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|

**Constraints:**

- UNIQUE: (pattern\_id, tag\_id)

**Indexes:**

- idx\_number\_pattern\_tag\_relations\_pattern\_id ON (pattern\_id)
- idx\_number\_pattern\_tag\_relations\_tag\_id ON (tag\_id)

**Foreign Keys:**

- pattern\_id → number\_patterns(id)
- tag\_id → number\_pattern\_tags(id)

**3.4 NHÓM QUẢN LÝ GIÁ (Price Management)**

**3.4.1 Bảng number\_pattern\_price\_lists**

**Mục đích:** Quản lý bảng giá theo thời gian

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|list\_code|varchar(50)|Mã bảng giá|NOT NULL, UNIQUE|
|name|varchar(100)|Tên bảng giá|NOT NULL|
|description|text|Mô tả|NULL|
|effective\_from|timestamptz|Ngày hiệu lực|NOT NULL|
|effective\_to|timestamptz|Ngày hết hiệu lực|NULL|
|is\_active|bool|Trạng thái hoạt động|DEFAULT true|
|parent\_id|int4|ID bảng giá cha|FK → number\_pattern\_price\_lists(id)|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'PRICE\_LIST\_APPROVAL'|
|status|varchar(100)|Trạng thái approval|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_number\_pattern\_price\_lists\_effective\_from ON (effective\_from)
- idx\_number\_pattern\_price\_lists\_effective\_to ON (effective\_to)
- idx\_number\_pattern\_price\_lists\_is\_active ON (is\_active)
- idx\_number\_pattern\_price\_lists\_object\_type\_status ON (object\_type, status)
- idx\_number\_pattern\_price\_lists\_parent\_id ON (parent\_id)

**Foreign Keys:**

- parent\_id → number\_pattern\_price\_lists(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.4.2 Bảng number\_pattern\_prices**

**Mục đích:** Chi tiết giá cho từng pattern trong bảng giá

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|price\_list\_id|int4|ID bảng giá|NOT NULL, FK → number\_pattern\_price\_lists(id)|
|pattern\_id|int4|ID pattern|NOT NULL, FK → number\_patterns(id)|
|base\_price|numeric(12,2)|Giá gốc|NOT NULL|
|discount\_percentage|numeric(5,2)|Phần trăm giảm giá|DEFAULT 0|
|final\_price|numeric(12,2)|Giá cuối cùng|NOT NULL|
|min\_price|numeric(12,2)|Giá tối thiểu|NULL|
|max\_price|numeric(12,2)|Giá tối đa|NULL|
|currency|varchar(3)|Đơn vị tiền tệ|DEFAULT 'VND'|
|notes|text|Ghi chú|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Constraints:**

- UNIQUE: (price\_list\_id, pattern\_id)

**Indexes:**

- idx\_number\_pattern\_prices\_price\_list\_id ON (price\_list\_id)
- idx\_number\_pattern\_prices\_pattern\_id ON (pattern\_id)

**Foreign Keys:**

- price\_list\_id → number\_pattern\_price\_lists(id)
- pattern\_id → number\_patterns(id)

**3.5 NHÓM QUẢN LÝ MSISDN (MSISDN Management)**

**3.5.1 Bảng msisdns**

**Mục đích:** Quản lý số điện thoại

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|msisdn|varchar(20)|Số điện thoại|NOT NULL, UNIQUE|
|pattern\_id|int4|ID pattern|FK → number\_patterns(id)|
|range\_batch\_id|int4|ID batch range|FK → number\_range\_batches(id)|
|sim\_id|int4|ID SIM|NULL|
|customer\_id|int4|ID khách hàng|NULL|
|last\_status\_change|timestamptz|Thời gian đổi trạng thái cuối|DEFAULT CURRENT\_TIMESTAMP|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|expiry\_date|timestamptz|Ngày hết hạn|NULL|
|dealer\_id|int4|ID dealer|NULL|
|price|numeric(12,2)|Giá bán|NULL|
|reservation\_id|int4|ID reservation|NULL|
|reservation\_expiry|timestamptz|Hết hạn reservation|NULL|
|is\_ported|bool|Đã chuyển mạng|DEFAULT false|
|port\_in\_date|timestamptz|Ngày chuyển mạng vào|NULL|
|port\_out\_date|timestamptz|Ngày chuyển mạng ra|NULL|
|notes|text|Ghi chú|NULL|
|object\_type|varchar(100)|Loại đối tượng|CHECK = 'MSISDN\_LIFECYCLE'|
|status|varchar(100)|Trạng thái lifecycle|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Constraints:**

- CHECK: object\_type = 'MSISDN\_LIFECYCLE'

**Indexes:**

- idx\_msisdns\_msisdn ON (msisdn)
- idx\_msisdns\_pattern\_id ON (pattern\_id)
- idx\_msisdns\_sim\_id ON (sim\_id)
- idx\_msisdns\_customer\_id ON (customer\_id)
- idx\_msisdns\_dealer\_id ON (dealer\_id)
- idx\_msisdns\_status ON (status)
- idx\_msisdns\_object\_type\_status ON (object\_type, status)
- idx\_msisdns\_is\_ported ON (is\_ported)

**Foreign Keys:**

- pattern\_id → number\_patterns(id)
- range\_batch\_id → number\_range\_batches(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.6 NHÓM QUẢN LÝ RANGE BATCH (Range Batch Management)**

**3.6.1 Bảng number\_range\_batches**

**Mục đích:** Quản lý batch import dải số

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_code|varchar(50)|Mã batch|NOT NULL, UNIQUE|
|allocated\_numbers|int4|Số lượng đã phân bổ|DEFAULT 0|
|total\_numbers|int4|Tổng số lượng|NULL|
|description|text|Mô tả batch|NULL|
|provider\_id|int4|ID nhà cung cấp|FK → providers(id)|
|declaration\_date|timestamptz|Ngày khai báo|NULL|
|declaration\_reason|text|Lý do khai báo|NULL|
|submitted\_by|int4|Người submit|NULL|
|submitted\_at|timestamptz|Thời gian submit|NULL|
|approved\_by|int4|Người duyệt|NULL|
|approved\_at|timestamptz|Thời gian duyệt|NULL|
|rejected\_by|int4|Người từ chối|NULL|
|rejected\_at|timestamptz|Thời gian từ chối|NULL|
|rejected\_reason|text|Lý do từ chối|NULL|
|imported\_by|int4|Người import|NULL|
|imported\_at|timestamptz|Thời gian import|NULL|
|activated\_by|int4|Người kích hoạt|NULL|
|activated\_at|timestamptz|Thời gian kích hoạt|NULL|
|completed\_by|int4|Người hoàn thành|NULL|
|completed\_at|timestamptz|Thời gian hoàn thành|NULL|
|object\_type|varchar(100)|Loại đối tượng|CHECK = 'NUMBER\_RANGE\_APPROVAL'|
|status|varchar(100)|Trạng thái approval|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Constraints:**

- CHECK: object\_type = 'NUMBER\_RANGE\_APPROVAL'

**Indexes:**

- idx\_number\_range\_batches\_batch\_code ON (batch\_code)
- idx\_number\_range\_batches\_provider\_id ON (provider\_id)
- idx\_number\_range\_batches\_status ON (status)
- idx\_number\_range\_batches\_object\_type\_status ON (object\_type, status)

**Foreign Keys:**

- provider\_id → providers(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.6.2 Bảng number\_range\_batch\_details**

**Mục đích:** Chi tiết dải số trong batch

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → number\_range\_batches(id)|
|prefix|varchar(10)|Đầu số|NOT NULL|
|start\_range|varchar(20)|Số đầu dải|NOT NULL|
|end\_range|varchar(20)|Số cuối dải|NOT NULL|
|total\_numbers|int4|Tổng số trong dải|NULL|
|description|text|Mô tả dải số|NULL|
|status|varchar(20)|Trạng thái|DEFAULT 'PENDING'|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_number\_range\_batch\_details\_batch\_id ON (batch\_id)
- idx\_number\_range\_batch\_details\_prefix ON (prefix)
- idx\_number\_range\_batch\_details\_status ON (status)

**Foreign Keys:**

- batch\_id → number\_range\_batches(id)

**3.6.3 Bảng number\_range\_batch\_attachments**

**Mục đích:** File đính kèm cho batch

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → number\_range\_batches(id)|
|media\_id|int4|ID file|NOT NULL, FK → auth.media\_files(id)|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|

**Indexes:**

- idx\_number\_range\_batch\_attachments\_batch\_id ON (batch\_id)
- idx\_number\_range\_batch\_attachments\_media\_id ON (media\_id)

**Foreign Keys:**

- batch\_id → number\_range\_batches(id)
- media\_id → auth.media\_files(id)

**3.7 NHÓM QUẢN LÝ SIM (SIM Management)**

**3.7.1 Bảng sims**

**Mục đích:** Quản lý SIM vật lý và eSIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|iccid|varchar(30)|ICCID của SIM|NOT NULL, UNIQUE|
|imsi|varchar(20)|IMSI của SIM|UNIQUE|
|batch\_id|int4|ID batch SIM|FK → sim\_batches(id)|
|sim\_type|varchar(20)|Loại SIM|NOT NULL|
|pin1|varchar(100)|PIN1|NULL|
|puk1|varchar(100)|PUK1|NULL|
|pin2|varchar(100)|PIN2|NULL|
|puk2|varchar(100)|PUK2|NULL|
|adm1|varchar(100)|ADM1|NULL|
|eki|varchar(64)|EKI|NULL|
|msisdn\_id|int4|ID MSISDN|NULL|
|activation\_date|timestamptz|Ngày kích hoạt|NULL|
|deactivation\_date|timestamptz|Ngày vô hiệu hóa|NULL|
|last\_status\_change|timestamptz|Thời gian đổi trạng thái cuối|DEFAULT CURRENT\_TIMESTAMP|
|warehouse\_id|int4|ID kho|NULL|
|customer\_id|int4|ID khách hàng|NULL|
|dealer\_id|int4|ID dealer|NULL|
|is\_esim|bool|Là eSIM|DEFAULT false|
|lpa\_string|text|LPA string (eSIM)|NULL|
|matching\_id|varchar(100)|Matching ID (eSIM)|NULL|
|activation\_code|text|Mã kích hoạt (eSIM)|NULL|
|profile\_type|varchar(100)|Loại profile (eSIM)|NULL|
|esim\_state|varchar(50)|Trạng thái eSIM|NULL|
|notes|text|Ghi chú|NULL|
|object\_type|varchar(100)|Loại đối tượng|CHECK = 'SIM\_LIFECYCLE'|
|status|varchar(100)|Trạng thái lifecycle|NULL|
|created\_at|timestamp|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamp|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Constraints:**

- CHECK: object\_type = 'SIM\_LIFECYCLE'

**Indexes:**

- idx\_sims\_iccid ON (iccid)
- idx\_sims\_imsi ON (imsi)
- idx\_sims\_batch\_id ON (batch\_id)
- idx\_sims\_msisdn\_id ON (msisdn\_id)
- idx\_sims\_customer\_id ON (customer\_id)
- idx\_sims\_dealer\_id ON (dealer\_id)
- idx\_sims\_warehouse\_id ON (warehouse\_id)
- idx\_sims\_is\_esim ON (is\_esim)
- idx\_sims\_matching\_id ON (matching\_id)
- idx\_sims\_esim\_state ON (esim\_state)
- idx\_sims\_object\_type\_status ON (object\_type, status)

**Foreign Keys:**

- batch\_id → sim\_batches(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.8 NHÓM QUẢN LÝ SIM BATCH (SIM Batch Management)**

**3.8.1 Bảng sim\_batches**

**Mục đích:** Quản lý batch import SIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_code|varchar(50)|Mã batch|NOT NULL, UNIQUE|
|sim\_type|varchar(50)|Loại SIM|NULL|
|artwork|varchar(100)|Thiết kế SIM|NULL|
|profile|varchar(50)|Profile SIM|NULL|
|batch\_number|varchar(20)|Số lô sản xuất|NULL|
|sim\_reference|varchar(50)|Tham chiếu SIM|NULL|
|sim\_size|varchar(20)|Kích thước SIM|NULL|
|transport\_key|varchar(50)|Transport key|NULL|
|smsc|varchar(50)|SMSC|NULL|
|op\_index|varchar(20)|OP index|NULL|
|quantity|int4|Số lượng|DEFAULT 0|
|manufacture\_date|date|Ngày sản xuất|NULL|
|expiry\_date|date|Ngày hết hạn|NULL|
|description|text|Mô tả batch|NULL|
|provider\_id|int4|ID nhà cung cấp|FK → providers(id)|
|graph\_reference|varchar(100)|Tham chiếu đồ thị|NULL|
|is\_esim|bool|Là eSIM batch|DEFAULT false|
|customer\_name|varchar(100)|Tên khách hàng|NULL|
|po\_ref\_number|varchar(50)|Số PO|NULL|
|location|varchar(100)|Địa điểm|NULL|
|submitted\_by|int4|Người submit|NULL|
|submitted\_at|timestamptz|Thời gian submit|NULL|
|approved\_by|int4|Người duyệt|NULL|
|approved\_at|timestamptz|Thời gian duyệt|NULL|
|rejected\_by|int4|Người từ chối|NULL|
|rejected\_at|timestamptz|Thời gian từ chối|NULL|
|rejected\_reason|text|Lý do từ chối|NULL|
|imported\_by|int4|Người import|NULL|
|imported\_at|timestamptz|Thời gian import|NULL|
|completed\_by|int4|Người hoàn thành|NULL|
|completed\_at|timestamptz|Thời gian hoàn thành|NULL|
|object\_type|varchar(100)|Loại đối tượng|CHECK = 'SIM\_BATCH\_APPROVAL'|
|status|varchar(100|||
**Constraints:**

- CHECK: object\_type = 'SIM\_BATCH\_APPROVAL'

**Indexes:**

- idx\_sim\_batches\_batch\_code ON (batch\_code)
- idx\_sim\_batches\_provider\_id ON (provider\_id)
- idx\_sim\_batches\_is\_esim ON (is\_esim)
- idx\_sim\_batches\_object\_type\_status ON (object\_type, status)

**Foreign Keys:**

- provider\_id → providers(id)
- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.8.2 Bảng sim\_batch\_details**

**Mục đích:** Chi tiết SIM trong batch

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → sim\_batches(id)|
|iccid|varchar(20)|ICCID|NOT NULL|
|imsi|varchar(20)|IMSI|NOT NULL|
|pin1|varchar(100)|PIN1|NOT NULL|
|puk1|varchar(100)|PUK1|NOT NULL|
|pin2|varchar(100)|PIN2|NOT NULL|
|puk2|varchar(100)|PUK2|NOT NULL|
|adm1|varchar(100)|ADM1|NOT NULL|
|eki|varchar(64)|EKI|NOT NULL|
|kic01|varchar(64)|KIC01|NULL|
|kid01|varchar(64)|KID01|NULL|
|kik01|varchar(64)|KIK01|NULL|
|kic02|varchar(64)|KIC02|NULL|
|kid02|varchar(64)|KID02|NULL|
|kik02|varchar(64)|KIK02|NULL|
|lpa\_string|text|LPA string (eSIM)|NULL|
|matching\_id|varchar(100)|Matching ID (eSIM)|NULL|
|activation\_code|text|Mã kích hoạt (eSIM)|NULL|
|profile\_type|varchar(100)|Loại profile (eSIM)|NULL|
|esim\_state|varchar(50)|Trạng thái eSIM|DEFAULT 'RELEASED'|
|status|varchar(20)|Trạng thái|DEFAULT 'PENDING'|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Constraints:**

- UNIQUE: (batch\_id, iccid)

**Indexes:**

- idx\_sim\_batch\_details\_batch\_id ON (batch\_id)
- idx\_sim\_batch\_details\_iccid ON (iccid)
- idx\_sim\_batch\_details\_imsi ON (imsi)
- idx\_sim\_batch\_details\_matching\_id ON (matching\_id)
- idx\_sim\_batch\_details\_status ON (status)

**Foreign Keys:**

- batch\_id → sim\_batches(id)

**3.8.3 Bảng sim\_batch\_attachments**

**Mục đích:** File đính kèm cho SIM batch

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → sim\_batches(id)|
|media\_id|int4|ID file|NOT NULL, FK → auth.media\_files(id)|
|attachment\_type|varchar(50)|Loại file đính kèm|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|

**Indexes:**

- idx\_sim\_batch\_attachments\_batch\_id ON (batch\_id)
- idx\_sim\_batch\_attachments\_media\_id ON (media\_id)
- idx\_sim\_batch\_attachments\_attachment\_type ON (attachment\_type)

**Foreign Keys:**

- batch\_id → sim\_batches(id)
- media\_id → auth.media\_files(id)

**3.9 NHÓM QUẢN LÝ THAY ĐỔI TRẠNG THÁI (State Change Management)**

**3.9.1 Bảng msisdn\_state\_change\_batches**

**Mục đích:** Quản lý batch thay đổi trạng thái MSISDN

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_code|varchar(50)|Mã batch|NOT NULL, UNIQUE|
|from\_status|varchar(100)|Trạng thái cũ|NOT NULL|
|to\_status|varchar(100)|Trạng thái mới|NOT NULL|
|description|text|Mô tả batch|NULL|
|total\_msisdns|int4|Tổng số MSISDN|DEFAULT 0|
|processed\_msisdns|int4|Số đã xử lý|DEFAULT 0|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'MSISDN\_STATE\_CHANGE\_APPROVAL'|
|status|varchar(100)|Trạng thái approval|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_msisdn\_state\_change\_batches\_batch\_code ON (batch\_code)
- idx\_msisdn\_state\_change\_batches\_from\_status ON (from\_status)
- idx\_msisdn\_state\_change\_batches\_to\_status ON (to\_status)
- idx\_msisdn\_state\_change\_batches\_object\_type\_status ON (object\_type, status)

**Foreign Keys:**

- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.9.2 Bảng msisdn\_state\_change\_items**

**Mục đích:** Chi tiết MSISDN trong batch thay đổi trạng thái

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → msisdn\_state\_change\_batches(id)|
|msisdn\_id|int4|ID MSISDN|NOT NULL, FK → msisdns(id)|
|is\_processed|bool|Đã xử lý|DEFAULT false|
|processed\_at|timestamptz|Thời gian xử lý|NULL|

**Constraints:**

- UNIQUE: (batch\_id, msisdn\_id)

**Indexes:**

- idx\_msisdn\_state\_change\_items\_batch\_id ON (batch\_id)
- idx\_msisdn\_state\_change\_items\_msisdn\_id ON (msisdn\_id)
- idx\_msisdn\_state\_change\_items\_is\_processed ON (is\_processed)

**Foreign Keys:**

- batch\_id → msisdn\_state\_change\_batches(id)
- msisdn\_id → msisdns(id)

**3.9.3 Bảng sim\_state\_change\_batches**

**Mục đích:** Quản lý batch thay đổi trạng thái SIM

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_code|varchar(50)|Mã batch|NOT NULL, UNIQUE|
|from\_status|varchar(100)|Trạng thái cũ|NOT NULL|
|to\_status|varchar(100)|Trạng thái mới|NOT NULL|
|description|text|Mô tả batch|NULL|
|total\_sims|int4|Tổng số SIM|DEFAULT 0|
|processed\_sims|int4|Số đã xử lý|DEFAULT 0|
|object\_type|varchar(100)|Loại đối tượng|DEFAULT 'SIM\_STATE\_CHANGE\_APPROVAL'|
|status|varchar(100)|Trạng thái approval|NULL|
|created\_at|timestamptz|Thời gian tạo|DEFAULT CURRENT\_TIMESTAMP|
|updated\_at|timestamptz|Thời gian cập nhật|DEFAULT CURRENT\_TIMESTAMP|
|created\_by|int4|Người tạo|NULL|
|updated\_by|int4|Người cập nhật|NULL|

**Indexes:**

- idx\_sim\_state\_change\_batches\_batch\_code ON (batch\_code)
- idx\_sim\_state\_change\_batches\_from\_status ON (from\_status)
- idx\_sim\_state\_change\_batches\_to\_status ON (to\_status)
- idx\_sim\_state\_change\_batches\_object\_type\_status ON (object\_type, status)

**Foreign Keys:**

- (object\_type, status) → auth.state\_definitions(object\_type, status)

**3.9.4 Bảng sim\_state\_change\_items**

**Mục đích:** Chi tiết SIM trong batch thay đổi trạng thái

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|**Ràng buộc**|
| :- | :- | :- | :- |
|id|serial4|Khóa chính|PRIMARY KEY|
|batch\_id|int4|ID batch|NOT NULL, FK → sim\_state\_change\_batches(id)|
|sim\_id|int4|ID SIM|NOT NULL, FK → sims(id)|
|is\_processed|bool|Đã xử lý|DEFAULT false|
|processed\_at|timestamptz|Thời gian xử lý|NULL|

**Constraints:**

- UNIQUE: (batch\_id, sim\_id)

**Indexes:**

- idx\_sim\_state\_change\_items\_batch\_id ON (batch\_id)
- idx\_sim\_state\_change\_items\_sim\_id ON (sim\_id)
- idx\_sim\_state\_change\_items\_is\_processed ON (is\_processed)

**Foreign Keys:**

- batch\_id → sim\_state\_change\_batches(id)
- sim\_id → sims(id)

**4. RELATIONSHIPS VÀ FOREIGN KEYS**

**4.1 Cross-schema References**

**To auth schema:**

- number\_patterns → auth.users(id) (created\_by, updated\_by)
- number\_pattern\_price\_lists → auth.state\_definitions(object\_type, status)
- number\_range\_batches → auth.state\_definitions(object\_type, status)
- sim\_batches → auth.state\_definitions(object\_type, status)
- msisdns → auth.state\_definitions(object\_type, status)
- sims → auth.state\_definitions(object\_type, status)
- sim\_batch\_attachments → auth.media\_files(id)
- number\_range\_batch\_attachments → auth.media\_files(id)

**5. BUSINESS RULES VÀ CONSTRAINTS**

**5.1 Data Constraints**

**Number Pattern Management:**

- pattern\_code phải unique toàn hệ thống
- regexp\_pattern phải là valid regex
- priority\_level càng thấp thì độ ưu tiên càng cao
- format\_pattern dùng để hiển thị số theo format đẹp

**MSISDN Lifecycle:**

- msisdn phải unique toàn hệ thống
- Chỉ được gắn với 1 SIM tại 1 thời điểm
- reservation\_expiry phải > current\_time khi có reservation
- is\_ported = true khi có port\_in\_date hoặc port\_out\_date

**SIM Lifecycle:**

- iccid và imsi phải unique toàn hệ thống
- eSIM có thêm các trường: lpa\_string, matching\_id, activation\_code
- SIM vật lý có đầy đủ PIN/PUK codes
- Chỉ được gắn với 1 MSISDN tại 1 thời điểm

**Price Management:**

- effective\_from <= effective\_to nếu có effective\_to
- final\_price = base\_price \* (1 - discount\_percentage/100)
- min\_price <= final\_price <= max\_price nếu có giới hạn

**Batch Processing:**

- batch\_code phải unique trong từng loại batch
- total\_numbers/total\_sims/total\_msisdns >= processed count
- Approval workflow: DRAFT → SUBMITTED → APPROVED/REJECTED → IMPORTED → COMPLETED

**5.2 Unique Constraints Summary**

|**Bảng**|**Unique Constraint**|
| :- | :- |
|providers|provider\_code|
|number\_patterns|pattern\_code|
|number\_pattern\_groups|group\_code|
|number\_pattern\_subgroups|subgroup\_code|
|number\_pattern\_tags|tag\_code|
|number\_pattern\_price\_lists|list\_code|
|number\_pattern\_prices|(price\_list\_id, pattern\_id)|
|number\_pattern\_subgroup\_relations|(pattern\_id, subgroup\_id)|
|number\_pattern\_tag\_relations|(pattern\_id, tag\_id)|
|msisdns|msisdn|
|number\_range\_batches|batch\_code|
|sims|iccid, imsi|
|sim\_batches|batch\_code|
|sim\_batch\_details|(batch\_id, iccid)|
|msisdn\_state\_change\_batches|batch\_code|
|msisdn\_state\_change\_items|(batch\_id, msisdn\_id)|
|sim\_state\_change\_batches|batch\_code|
|sim\_state\_change\_items|(batch\_id, sim\_id)|

**5.3 State Management**

**MSISDN States:**

- AVAILABLE: Có sẵn để bán
- RESERVED: Đã được reserve
- SOLD: Đã bán
- ACTIVATED: Đã kích hoạt
- SUSPENDED: Tạm ngưng
- TERMINATED: Đã chấm dứt
- PORTED\_OUT: Đã chuyển mạng ra
- PORTED\_IN: Đã chuyển mạng vào

**SIM States:**

- MANUFACTURED: Đã sản xuất
- IMPORTED: Đã import vào hệ thống
- AVAILABLE: Có sẵn để bán
- ALLOCATED: Đã phân bổ
- ACTIVATED: Đã kích hoạt
- SUSPENDED: Tạm ngưng
- TERMINATED: Đã vô hiệu hóa
- DAMAGED: Hư hỏng
- LOST: Mất

**eSIM States:**

- RELEASED: Đã phát hành
- DOWNLOADED: Đã download
- INSTALLED: Đã cài đặt
- ENABLED: Đã kích hoạt
- DISABLED: Đã vô hiệu hóa
- DELETED: Đã xóa

**6. INDEXING STRATEGY**

**6.1 Performance Indexes**

**MSISDN Lookup:**

sql

*-- High frequency searches*

CREATE INDEX idx\_msisdns\_status\_pattern ON msisdns(status, pattern\_id) 

WHERE status = 'AVAILABLE';

CREATE INDEX idx\_msisdns\_customer\_status ON msisdns(customer\_id, status) 

WHERE customer\_id IS NOT NULL;

**SIM Lookup:**

sql

*-- SIM availability check*

CREATE INDEX idx\_sims\_status\_type ON sims(status, sim\_type) 

WHERE status = 'AVAILABLE';

*-- eSIM specific queries*

CREATE INDEX idx\_sims\_esim\_state ON sims(is\_esim, esim\_state) 

WHERE is\_esim = true;

**Price Lookup:**

sql

*-- Active price lookup*

CREATE INDEX idx\_pattern\_prices\_active ON number\_pattern\_prices p

JOIN number\_pattern\_price\_lists l ON p.price\_list\_id = l.id

WHERE l.is\_active = true AND l.effective\_from <= CURRENT\_TIMESTAMP 

AND (l.effective\_to IS NULL OR l.effective\_to >= CURRENT\_TIMESTAMP);

**6.2 Composite Indexes**

sql

*-- Pattern matching queries*

CREATE INDEX idx\_patterns\_active\_priority ON number\_patterns(is\_active, priority\_level) 

WHERE is\_active = true;

*-- Batch processing queries*

CREATE INDEX idx\_batch\_details\_status\_batch ON sim\_batch\_details(status, batch\_id);

CREATE INDEX idx\_range\_batch\_prefix\_status ON number\_range\_batch\_details(prefix, status);

**7. WORKFLOW PROCESSING**

**7.1 Number Range Import Workflow**

1. **Create Batch**: Tạo number\_range\_batches với status = 'DRAFT'
1. **Add Details**: Thêm number\_range\_batch\_details với các dải số
1. **Submit**: Chuyển status = 'SUBMITTED' để chờ duyệt
1. **Approval**: Duyệt hoặc từ chối batch
1. **Import**: Generate MSISDNs từ các dải số đã duyệt
1. **Complete**: Đánh dấu hoàn thành

**7.2 SIM Batch Import Workflow**

1. **Create Batch**: Tạo sim\_batches với status = 'DRAFT'
1. **Upload Details**: Import sim\_batch\_details từ file CSV/Excel
1. **Validation**: Kiểm tra duplicate ICCID/IMSI
1. **Submit**: Submit để chờ duyệt
1. **Approval**: Duyệt batch
1. **Import**: Tạo records trong bảng sims
1. **Complete**: Hoàn thành import

**7.3 State Change Workflow**

1. **Create Change Batch**: Tạo state change batch
1. **Add Items**: Thêm danh sách MSISDNs/SIMs cần đổi trạng thái
1. **Submit**: Submit để chờ duyệt
1. **Approval**: Duyệt thay đổi
1. **Execute**: Thực hiện thay đổi trạng thái từng item
1. **Complete**: Hoàn thành batch

**7.4 Pattern Price Management**

1. **Create Price List**: Tạo bảng giá mới
1. **Define Prices**: Thiết lập giá cho từng pattern
1. **Set Effective Date**: Đặt ngày hiệu lực
1. **Approval**: Duyệt bảng giá
1. **Activation**: Kích hoạt bảng giá vào ngày hiệu lực
1. **Deactivation**: Vô hiệu hóa bảng giá cũ




