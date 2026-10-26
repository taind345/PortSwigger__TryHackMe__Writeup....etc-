
-mô tả lab: lab cho sẵn `carlos:montoya`. Sau khi login thì nó bắt nhập mã 2FA. Nhưng logic check 2FA bị lỗi.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass
![[Pasted image 20261018194540.png]]
### hướng giải
- [ ] mình login `carlos:montoya`, sau đó quan sát gói tin này, mình thấy nó redirect sang `/login2` bắt nhập `verification-code`
- [ ] mình thử nhập bừa `1234`, mình thấy nó báo sai
- [ ] mình thử vào thẳng `/my-account` mà không thèm nhập 2FA, mình thấy nó cho vào luôn
- [ ] => nên mình hiểu là server nó chỉ check user/pass ở bước 1 là cho cookie logged-in rồi, bước 2 chỉ là hình thức, không verify gì cả
![[Pasted image 20261018194705.png]]
### giải
-bước 1: mình login bằng Burp:
```
POST /login HTTP/1.1
...
username=carlos&password=montoya
```
-response trả về:
```
HTTP/1.1 302 Found
Set-Cookie: session=SESSION_LOGGED_IN
Location: /login2
```

-bước 2: thay vì đi theo `/login2`, mình sửa URL thành `/my-account` rồi forward request đi, kèm cookie vừa nhận được:
```
GET /my-account HTTP/1.1
Cookie: session=SESSION_LOGGED_IN
```

-sau khi quan sát gói tin trả về này, mình thấy nó hiện luôn trang `My Account | carlos`, không hỏi 2FA nữa. Là solve lab.

> [!NOTE]
> -bài học của mình: 2FA mà check ở client hoặc không check ở trang `/my-account` thì coi như không có 2FA.
