




![A logo of a company&#x0A;&#x0A;Description automatically generated](Aspose.Words.51d6f343-bdf7-47d0-a14b-0d1b63987c28.001.jpeg)

**HỆ THỐNG BSS**

**TÀI LIỆU THIẾT KẾ**

**Content Management System**

**Phiên bản 1.0**
**\


**1. Tổng quan Schema CMS**

Schema cms được thiết kế để quản lý nội dung đa ngôn ngữ trong hệ thống CMS. Bao gồm các chức năng chính:

- **Multi-language Support**: Hỗ trợ đa ngôn ngữ với translation tables
- **Dynamic Content Structure**: Cấu trúc nội dung linh hoạt với JSON fields
- **SEO Optimization**: Meta tags, canonical URLs, Open Graph
- **Workflow Management**: Quản lý quy trình phê duyệt nội dung
- **Category & Tag System**: Phân loại và gắn tag nội dung
- **Menu Management**: Quản lý menu đa cấp

**2. Sơ đồ quan hệ Schema CMS**

Languages (1) ←→ (N) All Translation Tables

`    `↓

Organizations (1) ←→ (N) Content Categories

`    `↓                          ↓

Menus (1) ←→ (N) Menu Items    Dynamic Contents

`    `↓              ↓               ↓

Menu Translations  Dynamic Content Categories/Tags

`    `↓              ↓               ↓

Content Structure Types ←→ News ←→ News Translations

**3. Mô tả chi tiết các bảng**

**3.1 Quản lý Ngôn ngữ**

**cms.languages**

**Mục đích**: Quản lý các ngôn ngữ được hỗ trợ trong hệ thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|code|varchar(10)|Mã ngôn ngữ (ISO 639-1: en, vi, fr...)|
|name|varchar(100)|Tên ngôn ngữ (English, Vietnamese...)|
|native\_name|varchar(100)|Tên bản địa (English, Tiếng Việt...)|
|flag\_icon|varchar(50)|Icon cờ quốc gia|
|is\_rtl|bool|Ngôn ngữ từ phải sang trái|
|is\_active|bool|Trạng thái hoạt động|
|display\_order|int4|Thứ tự hiển thị|

**Đặc điểm**:

- Code là unique constraint
- Hỗ trợ RTL languages (Arabic, Hebrew...)

**3.2 Quản lý Danh mục Nội dung**

**cms.content\_categories**

**Mục đích**: Phân loại nội dung theo cấu trúc cây phân cấp

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|parent\_id|int4|Danh mục cha (self-reference)|
|name|varchar(200)|Tên danh mục|
|slug|varchar(255)|URL slug (unique per org)|
|description|text|Mô tả danh mục|
|image|varchar(255)|Hình ảnh đại diện|
|meta\_title|varchar(255)|SEO meta title|
|meta\_description|text|SEO meta description|
|meta\_keywords|varchar(255)|SEO keywords|
|display\_order|int4|Thứ tự hiển thị|

**Ràng buộc**: Unique constraint trên (organization\_id, slug)

**cms.content\_category\_translations**

**Mục đích**: Bản dịch đa ngôn ngữ cho danh mục

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|category\_id|int4|ID danh mục gốc|
|language\_id|int4|ID ngôn ngữ|
|name|varchar(200)|Tên danh mục đã dịch|
|slug|varchar(255)|URL slug đã dịch|
|description|text|Mô tả đã dịch|
|meta\_title|varchar(255)|Meta title đã dịch|

**3.3 Quản lý Cấu trúc Nội dung Động**

**cms.content\_structure\_types**

**Mục đích**: Định nghĩa các loại cấu trúc nội dung linh hoạt

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|name|varchar(200)|Tên loại cấu trúc|
|code|varchar(100)|Mã định danh (unique per org)|
|description|text|Mô tả|
|icon|varchar(100)|Icon hiển thị|
|fields|jsonb|Định nghĩa các trường dữ liệu|
|layout|jsonb|Cấu hình layout hiển thị|

**Đặc điểm**:

- fields chứa JSON schema để define dynamic fields
- layout chứa thông tin về cách hiển thị fields

**Ví dụ fields JSON**:

json

{

`  `"title": {"type": "text", "required": true},

`  `"content": {"type": "richtext", "required": true},

`  `"gallery": {"type": "image\_array", "required": false},

`  `"price": {"type": "number", "required": false}

}

**cms.dynamic\_contents**

**Mục đích**: Lưu trữ nội dung động theo structure types

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|structure\_type\_id|int4|ID loại cấu trúc|
|title|varchar(255)|Tiêu đề nội dung|
|slug|varchar(255)|URL slug|
|content\_data|jsonb|Dữ liệu nội dung thực tế|
|thumbnail|varchar(255)|Ảnh thumbnail|
|author\_id|int4|ID tác giả|
|is\_featured|bool|Nội dung nổi bật|
|view\_count|int4|Số lượt xem|
|published\_at|timestamp|Thời gian xuất bản|
|meta\_title|varchar(255)|SEO meta title|
|meta\_description|text|SEO meta description|
|canonical\_url|varchar(255)|Canonical URL|
|og\_title|varchar(255)|Open Graph title|
|og\_description|text|Open Graph description|
|og\_image|jsonb|Open Graph images|
|workflow\_status|varchar(100)|Trạng thái workflow|
|object\_type|varchar(100)|Loại object cho workflow|

**Đặc điểm**:

- content\_data chứa dữ liệu thực tế theo structure type
- Tích hợp với workflow system từ auth schema
- Comprehensive SEO support

**3.4 Quản lý Tin tức**

**cms.news**

**Mục đích**: Quản lý tin tức/bài viết truyền thống

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|category\_id|int4|ID danh mục|
|title|varchar(255)|Tiêu đề bài viết|
|slug|varchar(255)|URL slug (unique per org)|
|summary|text|Tóm tắt|
|content|text|Nội dung đầy đủ|
|excerpt|text|Đoạn trích|
|thumbnail|varchar(255)|Ảnh thumbnail|
|featured\_image|varchar(255)|Ảnh nổi bật|
|author\_id|int4|ID tác giả|
|published\_at|timestamp|Thời gian xuất bản|
|is\_featured|bool|Tin nổi bật|
|view\_count|int4|Số lượt xem|
|status|varchar(20)|Trạng thái: draft/published|
|custom\_fields|jsonb|Các trường tùy chỉnh|
|workflow\_status|varchar(100)|Trạng thái workflow|
|content\_type\_id|int4|ID loại nội dung|

**SEO Fields**:

- meta\_title, meta\_description, meta\_keywords
- canonical\_url, meta\_robots
- og\_title, og\_description, og\_image

**cms.news\_translations**

**Mục đích**: Bản dịch đa ngôn ngữ cho tin tức

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|news\_id|int4|ID tin tức gốc|
|language\_id|int4|ID ngôn ngữ|
|title|varchar(255)|Tiêu đề đã dịch|
|slug|varchar(255)|URL slug đã dịch|
|summary|text|Tóm tắt đã dịch|
|content|text|Nội dung đã dịch|
|custom\_fields|jsonb|Custom fields đã dịch|

**3.5 Hệ thống Tags**

**cms.tags**

**Mục đích**: Quản lý tags để gắn cho nội dung

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|name|varchar(100)|Tên tag|
|slug|varchar(150)|URL slug (unique per org)|

**cms.tag\_translations**

**Mục đích**: Bản dịch đa ngôn ngữ cho tags

**Many-to-Many Relationships:**

- cms.news\_tags: Liên kết news với tags
- cms.dynamic\_content\_tags: Liên kết dynamic content với tags
- cms.dynamic\_content\_categories: Liên kết dynamic content với categories

**3.6 Quản lý Menu**

**cms.menus**

**Mục đích**: Định nghĩa các menu trong website

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|organization\_id|int4|ID tổ chức|
|name|varchar(100)|Tên menu|
|code|varchar(50)|Mã menu (unique per org)|
|description|text|Mô tả menu|

**cms.menu\_items**

**Mục đích**: Các item trong menu (cấu trúc cây)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|id|serial4|Khóa chính|
|menu\_id|int4|ID menu|
|parent\_id|int4|Menu item cha|
|title|varchar(100)|Tiêu đề menu item|
|url|varchar(512)|URL liên kết|
|target|varchar(20)|Target (\_self/\_blank...)|
|icon|varchar(100)|Icon hiển thị|
|css\_class|varchar(100)|CSS class|
|position|int4|Vị trí trong menu|

**Translation Tables:**

- cms.menu\_translations: Dịch tên và mô tả menu
- cms.menu\_item\_translations: Dịch title và URL của menu items

**3.7 Bảng phụ trợ**

**cms.banners**

**Mục đích**: Quản lý banner (cấu trúc đơn giản)

|**Cột**|**Kiểu dữ liệu**|**Mô tả**|
| :- | :- | :- |
|banner\_img\_link|text|Link ảnh banner|

**4. Tính năng nổi bật**

**4.1 Multi-language Architecture**

- **Translation Pattern**: Mỗi entity chính có bảng translation tương ứng
- **Fallback Mechanism**: Có thể fallback về ngôn ngữ mặc định
- **RTL Support**: Hỗ trợ ngôn ngữ viết từ phải sang trái

**4.2 Dynamic Content System**

- **JSON Schema**: Flexible field definitions
- **Type Safety**: Validation dựa trên structure types
- **Layout Configuration**: Customizable display layouts

**4.3 SEO Optimization**

- **Complete Meta Tags**: title, description, keywords
- **Open Graph**: Facebook/social media optimization
- **Canonical URLs**: Tránh duplicate content
- **Meta Robots**: Control search engine indexing

**4.4 Workflow Integration**

- **State Management**: Tích hợp với auth.state\_definitions
- **Approval Process**: Support workflow approval
- **Version Control**: Track changes through workflow

**5. Indexes và Performance**

**Key Indexes:**

- idx\_dynamic\_contents\_organization: Tìm theo tổ chức
- idx\_dynamic\_contents\_slug: Tìm theo URL slug
- idx\_dynamic\_contents\_workflow\_status: Tìm theo trạng thái
- idx\_news\_category: Tìm tin tức theo danh mục
- Language-specific indexes cho translation tables




