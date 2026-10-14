![[jwt-kid-path.png]]
-mô tả lab: server dùng `kid` trong header làm **đường dẫn file** để đọc key verify. Mình trỏ nó về file rỗng `/dev/null` rồi ký bằng secret rỗng.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal

### hướng giải
- [ ] mình login rồi decode JWT, mình thấy header kiểu `{"kid":"key-xxxx","alg":"HS256"}`
- [ ] mình thử sửa `kid` thành `key-xxxx' OR ...` thì không ăn (không phải SQL), nhưng thử `../../../../dev/null` thì response khác
- [ ] sau khi quan sát hành vi này, mình đoán code server kiểu `open("/keys/" + kid).read()` làm secret. `/dev/null` đọc ra rỗng -> secret = `""`
- [ ] => mình ký token admin bằng secret rỗng + `kid` trỏ `/dev/null` là khớp

### giải
-bước 1: trong JWT Editor, mình sửa header:
```json
{"kid":"../../../../../../dev/null","alg":"HS256"}
```
-số lượng `../` mình tăng dần tới khi qua (thử 4-8 cái, xem response hết báo invalid key là được).
![[jwt-kid-null.png]]

-bước 2: payload `{"iss":"portswigger","sub":"administrator","exp":9999999999}`, ký HS256 với secret **rỗng** (`""`).

-bước 3: thay cookie, GET `/admin`:
```
Cookie: session=TOKEN_MOI
-> 200 admin panel, xóa carlos là solve
```
-nếu `/dev/null` không ăn (một số bản lab), mình thử trỏ về file có nội dung mình biết/bắt được, vd file static của web, rồi ký bằng nội dung file đó. Nguyên lý y hệt.

> [!NOTE]
> -bài học của mình: `kid` phải whitelist trong 1 set key cố định, không bao giờ ghép path. Gặp `kid` nhìn như filename/random-id là mình thử traversal ngay.
