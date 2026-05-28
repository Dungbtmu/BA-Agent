




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.eefed4cf-6c6a-42de-85ca-cb0270ae4ef5.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Thiết kế Schema CDR (Call Detail Record)**

**Phiên bản 1.0**
**\


**1. Tổng quan Schema CDR**

Schema cdr được thiết kế để quản lý và xử lý dữ liệu Call Detail Record trong hệ thống telecom. Bao gồm các chức năng chính:

- **ETL Pipeline**: Thu thập, xử lý và load dữ liệu từ nhiều nguồn
- **Multi-format Support**: Hỗ trợ CSV, ZIP, GZ files
- **Role-based Access**: Phân quyền chi tiết theo từng operation
- **Search & Export**: Tìm kiếm linh hoạt và xuất báo cáo
- **Audit Trail**: Theo dõi đầy đủ các hoạt động

**2. Sơ đồ quan hệ Schema CDR**

Data\_Sources (1) ←→ (N) Data\_Configurations

`       `↓                        ↓

Operation\_Logs              Column\_Definitions

`       `↓                        ↓

Processed\_Files          Searchable\_Fields

`       `↓                        ↓

Export\_Logs              Search\_Logs

**3. Mô tả chi tiết các bảng**

**3.1 Quản lý Nguồn Dữ liệu**

**cdr.data\_sources**

**Mục đích**: Quản lý các nguồn dữ liệu CDR (SFTP, FTP, Local)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|name|varchar(100)|Tên nguồn dữ liệu|
|connection\_type|varchar(20)|Loại kết nối: SFTP/FTP/LOCAL|
|server\_address|varchar(255)|Địa chỉ server|
|port|int4|Cổng kết nối|
|username|varchar(100)|Tên đăng nhập|
|password|varchar(255)|Mật khẩu (được mã hóa)|
|root\_directory|varchar(255)|Thư mục gốc|
|is\_active|bool|Trạng thái hoạt động|

**Ràng buộc**: connection\_type chỉ được phép: SFTP, FTP, LOCAL

**3.2 Cấu hình Xử lý Dữ liệu**

**cdr.data\_configurations**

**Mục đích**: Cấu hình chi tiết cho từng loại dữ liệu CDR

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|data\_source\_id|int4|ID nguồn dữ liệu|
|name|varchar(100)|Tên cấu hình (unique)|
|file\_pattern|varchar(255)|Pattern tìm file (regex)|
|directory\_path|varchar(255)|Đường dẫn thư mục|
|done\_directory|varchar(255)|Thư mục lưu file đã xử lý|
|file\_format|varchar(20)|Định dạng: CSV/ZIP/GZ|
|table\_name|varchar(100)|Tên bảng đích (unique)|
|csv\_delimiter|bpchar(1)|Ký tự phân tách CSV (mặc định: ',')|
|csv\_quote|bpchar(1)|Ký tự quote (mặc định: '"')|
|csv\_escape|bpchar(1)|Ký tự escape (mặc định: '')|
|csv\_header|bool|File có header không|
|csv\_encoding|varchar(20)|Encoding file (mặc định: UTF8)|
|csv\_import\_columns|\_text|Danh sách cột cần import|
|view\_access\_roles|\_text|Roles được xem dữ liệu|
|export\_excel\_roles|\_text|Roles được export Excel|
|export\_csv\_roles|\_text|Roles được export CSV|
|admin\_roles|\_text|Roles quản trị|
|fetch\_schedule|varchar(50)|Lịch trình tự động (cron expression)|
|display\_columns|\_text|Cột hiển thị mặc định|

**Đặc điểm**:

- Role-based access control cho từng operation
- Flexible CSV parsing configuration
- Automatic scheduling support

**3.3 Metadata Management**

**cdr.column\_definitions**

**Mục đích**: Định nghĩa cấu trúc cột cho từng configuration

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|config\_id|int4|ID cấu hình|
|column\_name|varchar(100)|Tên cột trong database|
|data\_type|varchar(50)|Kiểu dữ liệu PostgreSQL|
|csv\_column|varchar(100)|Tên cột trong file CSV|
|is\_required|bool|Cột bắt buộc|
|column\_order|int4|Thứ tự cột|
|default\_value|text|Giá trị mặc định|

**Ràng buộc**: Unique constraint trên (config\_id, column\_name)

**cdr.searchable\_fields**

**Mục đích**: Định nghĩa các trường có thể tìm kiếm

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|config\_id|int4|ID cấu hình|
|field\_name|varchar(100)|Tên trường|
|search\_type|varchar(20)|exact/like/range/in|
|display\_name|varchar(100)|Tên hiển thị cho user|
|is\_default|bool|Trường tìm kiếm mặc định|
|display\_order|int4|Thứ tự hiển thị|

**3.4 File Processing Tracking**

**cdr.processed\_files**

**Mục đích**: Theo dõi file đã xử lý để tránh duplicate processing

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|data\_source\_id|int4|ID nguồn dữ liệu|
|configuration\_id|int4|ID cấu hình|
|file\_name|varchar(255)|Tên file|
|file\_path|text|Đường dẫn đầy đủ|
|processed\_at|timestamp|Thời gian xử lý|
|batch\_id|int4|ID batch xử lý|
|records\_processed|int4|Số bản ghi đã xử lý|

**Ràng buộc**: Unique constraint trên (configuration\_id, file\_name)

**3.5 Audit và Logging**

**cdr.operation\_logs**

**Mục đích**: Ghi lại tất cả các operation ETL

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|data\_source\_id|int4|ID nguồn dữ liệu|
|configuration\_id|int4|ID cấu hình|
|operation\_type|varchar(50)|Loại operation (FETCH/PROCESS/LOAD)|
|status|varchar(20)|SUCCESS/FAILED/RUNNING|
|message|text|Thông điệp log chi tiết|
|started\_at|timestamptz|Thời gian bắt đầu|
|completed\_at|timestamptz|Thời gian kết thúc|
|file\_name|varchar(255)|Tên file đang xử lý|
|records\_processed|int4|Số bản ghi đã xử lý|
|copy\_command|text|Lệnh COPY đã thực hiện|
|user\_id|int4|ID người dùng (nếu manual)|
|client\_ip|varchar(50)|IP client|

**cdr.search\_logs**

**Mục đích**: Audit các hoạt động tìm kiếm

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|configuration\_id|int4|ID cấu hình|
|user\_id|int4|ID người dùng|
|user\_role|varchar(50)|Role của người dùng|
|search\_criteria|text|Điều kiện tìm kiếm (JSON)|
|executed\_at|timestamptz|Thời gian thực hiện|
|execution\_time|int4|Thời gian thực thi (ms)|
|record\_count|int4|Số bản ghi trả về|
|exported\_format|varchar(10)|Định dạng export (nếu có)|
|client\_ip|varchar(50)|IP client|

**cdr.export\_logs**

**Mục đích**: Audit các hoạt động export dữ liệu

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|configuration\_id|int4|ID cấu hình|
|user\_id|int4|ID người dùng|
|user\_role|varchar(50)|Role của người dùng|
|export\_format|varchar(20)|CSV/EXCEL/PDF|
|search\_criteria|text|Điều kiện export|
|started\_at|timestamptz|Thời gian bắt đầu|
|completed\_at|timestamptz|Thời gian hoàn thành|
|execution\_time|int4|Thời gian thực hiện|
|record\_count|int4|Số bản ghi export|
|file\_size|int8|Kích thước file (bytes)|
|output\_file\_path|text|Đường dẫn file output|
|status|varchar(20)|Trạng thái export|
|error\_message|text|Thông báo lỗi (nếu có)|

**4. Indexes và Performance**

**Indexes quan trọng:**

- idx\_export\_logs\_config: Tìm kiếm theo configuration
- idx\_export\_logs\_date: Tìm kiếm theo thời gian
- idx\_export\_logs\_user: Tìm kiếm theo user
- idx\_operation\_logs\_status: Tìm kiếm theo trạng thái
- idx\_search\_logs\_date: Tìm kiếm search logs theo thời gian

**5. Workflow ETL**

**Quy trình xử lý dữ liệu:**

1. **Fetch**: Scheduler tự động fetch file từ data sources
1. **Validate**: Kiểm tra file pattern và format
1. **Process**: Parse và validate dữ liệu theo column\_definitions
1. **Load**: Insert dữ liệu vào target table
1. **Archive**: Di chuyển file đã xử lý vào done\_directory
1. **Log**: Ghi lại toàn bộ quá trình vào operation\_logs

**Role-based Access Control:**

- **view\_access\_roles**: Quyền xem dữ liệu
- **export\_excel\_roles**: Quyền export Excel
- **export\_csv\_roles**: Quyền export CSV
- **admin\_roles**: Quyền quản trị (config, logs)

**6. Security và Best Practices**

**Bảo mật:**

- Mật khẩu data sources được mã hóa
- Audit trail đầy đủ cho mọi hoạt động
- Role-based access control chi tiết
- IP tracking cho audit

**Performance:**

- Partition tables theo thời gian cho logs
- Index optimization cho search queries
- Batch processing cho large datasets
- Connection pooling cho data sources

**Maintenance:**

- Regular cleanup cho processed files
- Archive old logs theo retention policy
- Monitor disk space cho temp files
- Health check cho data source connections


