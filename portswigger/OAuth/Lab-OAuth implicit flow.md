![[oauth-implicit.png]]
-mô tả lab: client dùng OAuth implicit flow nhưng đoạn cuối nó POST `/authenticate` với `email` do client gửi. Mình sửa email thành của carlos là login vào account carlos.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow
> -blog có nút "Login with social media", account carlos@carlos-montoya.net

### hướng giải
- [ ] mình bấm login social, đi hết luồng OAuth bằng account của mình, tới bước cuối client POST `/authenticate` với `email`, `username`, `token`
- [ ] sau đó quan sát gói tin này, mình thấy server tin luôn `email` client gửi, không verify lại với provider
- [ ] => mình sửa `email` thành `carlos@carlos-montoya.net` rồi gửi lại là nó cho login vào carlos

### giải
-bước 1: mình đi luồng OAuth thật 1 lần bằng account mình, bắt request cuối:
```
POST /authenticate HTTP/1.1
...
email=wiener@...&username=wiener&token=TOKEN_XIN&...
-> 302 /my-account?id=wiener
```

-bước 2: mình sửa email rồi forward:
```
POST /authenticate HTTP/1.1
...
email=carlos@carlos-montoya.net&username=wiener&token=TOKEN_XIN&...
-> 302 /my-account?id=carlos
```
![[oauth-implicit-email.png]]
-sau khi quan sát gói tin trả về này, mình thấy nó set session carlos luôn dù token là của mình. Mình GET `/my-account?id=carlos` là solve.

> [!NOTE]
> -bài học của mình: identity phải lấy từ server-to-server với provider (dùng token đổi ở bước 3), không bao giờ tin field email/username client gửi lên.
