![[idor-userid.png]]
-mô tả lab: `/my-account?id=wiener` lấy account theo id mình gửi. Đổi id là xem được người khác.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter

### hướng giải
- [ ] mình login `wiener:peter` vào `/my-account?id=wiener`, bình thường
- [ ] mình đổi thành `/my-account?id=carlos`, sau đó quan sát gói tin trả về, mình thấy nó hiện luôn account carlos kèm API key
- [ ] => không có check id có phải của session không. Mình copy API key submit là solve

### giải
```
GET /my-account?id=wiener -> 200 trang wiener
GET /my-account?id=carlos -> 200 trang carlos, body có API key: SECRET...
```
![[idor-userid-carlos.png]]

> [!NOTE]
> -cách mình test IDOR chuẩn: 2 account 2 browser, lấy id của A đem sang session B request. 200 + dữ liệu người khác là dính.
> -fix: lấy id từ session server, đừng lấy từ query client.
